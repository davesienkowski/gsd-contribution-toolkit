# contrib-gate

A `role:feature` GSD capability that bundles the **gsd-core contribution + maintainer-review toolkit** as an installable, toggleable plugin for **GSD 1.6.0+** (ADR-1244 capability ecosystem).

Installing it writes **12 `PreToolUse` gates + 1 `UserPromptSubmit` advisory** into your gsd-core `.claude/settings.json`, ships two skills and five commands, and adds one advisory `plan:pre` contribution. The bundled hook scripts **resolve and call the LIVE gsd-core gate scripts at runtime** — they don't reimplement gsd-core policy.

> Built and generated from the gsd-contrib-toolkit (v2.1 — Capability-Native Distribution). The bundle here is a published distribution artifact; the dev source of truth is the toolkit's canonical `hooks/`.

## Requirements

- **GSD 1.6.0+** (`engines.gsd: ">=1.6.0"`) — needs the capability install engine + runtime registry overlay.
- **A local `open-gsd/gsd-core` checkout**, resolvable from `$GSD_CORE_ROOT`, `~/repos/gsd-core`, or `~/gsd-core`. The gates shell out to gsd-core's own mechanizable scripts (`lint:ci`, `policy-invariants`, identity-drift, etc.), so a checkout must be present at runtime.

## Install

From your gsd-core checkout (so `--scope project` targets it):

```bash
node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability install \
  https://github.com/davesienkowski/gsd-contrib-gate.git#v1.0.0 \
  --scope project --yes --shared-file .claude/settings.json
```

- `--yes` grants consent — the capability ships executable surfaces (the 13 hooks), so the install discloses them and aborts without consent.
- `--shared-file .claude/settings.json` is **required to actually apply the gates** into settings.json; without it the install records the ledger + overlay but writes no hooks.
- `--scope project` installs into this checkout (`.gsd/capabilities/contrib-gate/` + a `.gsd-capabilities.json` ledger). Use `--scope global` for `~/.gsd/...`.

Pin a specific release with `#v1.0.0` (a tag) or `#sha:<40-hex>` (an exact commit).

## Manage

```bash
node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability status   --scope project
node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability disable  contrib-gate --scope project
node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability enable   contrib-gate --scope project
node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability update   contrib-gate --scope project --yes
node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability remove   contrib-gate --scope project
```

`remove` strips exactly the ledger-recorded files and shared-file fragments — it never deletes shared files wholesale.

## What the gates cover

12 `PreToolUse` gates fire on contribution-shaped tool calls:

| Gate | Guards |
|------|--------|
| `gh-issue-create` / `gh-pr-create` / `gh-edit` | well-formed, intake-passing issues/PRs |
| `issue-dedupe` | duplicate-issue detection before filing |
| `policy-invariants` | the LIVE gsd-core mechanizable POLICY-02 checks on commit / pr-create |
| `lint-ci-marker` | the `lint:ci` stamp before push |
| `git-commit-convention` | commit-message convention |
| `containment` | edits/pushes confined to the intended worktree |
| `freshness` | acting against a current base |
| `githooks-seal` | git hooks not tampered |
| `scan-gate` | secret / sensitive-content scan |
| `binlib-edit` | no hand-edits to generated `bin/lib/*.cjs` |

Plus a `UserPromptSubmit` advisory (`protocol-reminder`) and one advisory `plan:pre` contribution (gated by the default-off `workflow.gsd_contrib_enforcement` config flag).

## Honesty & scope

This matters — please read it as written, don't over-read it:

- The capability's **GSD-loop surfaces are advisory**: `gates[]` is **empty**, and the one `plan:pre` contribution only fires inside a GSD command. They never see an arbitrary tool call.
- The **harness-boundary enforcement** comes from the 12 `PreToolUse` hooks **once installed into `settings.json`** — those fire on matching tool calls while installed. That is a property of installed PreToolUse hooks, **not an inherent property of this (toggleable) capability**.
- It is therefore **fully removable**: `disable`/`remove` genuinely takes the enforcement away. **Do not call this capability "unbypassable."**
- Deliberate bypass accountability rides on the existing per-worktree, append-only `GSD_CONTRIB_OVERRIDE` receipt (a logged reason-string, never a silent default) — this capability adds no new receipt mechanism.

## Provenance

- **Version:** 1.0.0
- **Source toolkit:** gsd-contrib-toolkit (private), v2.1 milestone
- The bundle is generated from canonical `hooks/` with a `--check` drift gate and validated against the LIVE gsd-core capability validators before publish.
