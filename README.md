# contribution-toolkit

A `role:feature` GSD capability that bundles the **gsd-core contribution + maintainer-review toolkit** as an installable, toggleable plugin for **GSD 1.6.0+** (ADR-1244 capability ecosystem).

This release is **self-contained**: a remote install delivers the whole working surface — **13 hooks (12 `PreToolUse` gates + 1 `UserPromptSubmit` advisory)**, **2 skills**, and **5 commands** — straight from the published tree. (The earlier `…-gate` v1.0 release described skills and commands it did not actually ship in the bundle; that claim is now TRUE.) The bundled hook scripts **resolve and call the LIVE gsd-core gate scripts at runtime** — they don't reimplement gsd-core policy.

> Built and generated from the gsd-contrib-toolkit (v2.2 — self-contained capability distribution). The bundle here is a published distribution artifact; the dev source of truth is the toolkit's canonical `hooks/`/`skills/`/`commands/`.

## Requirements

- **GSD 1.6.0+** (`engines.gsd: ">=1.6.0"`) — needs the capability install engine + runtime registry overlay.
- **A local `open-gsd/gsd-core` checkout**, resolvable from `$GSD_CORE_ROOT`, `~/repos/gsd-core`, or `~/gsd-core`. The gates shell out to gsd-core's own mechanizable scripts (`lint:ci`, `policy-invariants`, identity-drift, etc.), so a checkout must be present at runtime.

## What the bundle delivers

The remote install is **self-contained** — it lays down the full surface, not a hooks-only artifact:

- **13 hooks** — 12 fail-closed `PreToolUse` gates + 1 advisory `UserPromptSubmit` reminder (`protocol-reminder`). The gates resolve and call the LIVE gsd-core scripts at runtime.
- **2 skills** — `gsd-core-contribution` (the contribution knowledge, including the stamp → marker → gate → scan loop) and `maintainer-review-sweep` (backs the maintainer assists).
- **5 commands** — `gsd-submit`, `gsd-review-sweep`, `gsd-triage-assist`, `gsd-release-preflight`, `gsd-ruleset-drift`.

These names/counts are exactly the `hooks[]`, `skills[]`, and the bundled `commands/` tree of `capability.json` (version 2.0.0) — no invented surface.

## Install

From your gsd-core checkout (so `--scope project` targets it):

```bash
node <gsd-core>/bin/gsd-tools.cjs capability install \
  https://github.com/davesienkowski/gsd-contribution-toolkit.git#v2.1.0 \
  --scope project --yes --shared-file .claude/settings.json
```

- `--yes` grants consent — the capability ships executable surfaces (the 13 hooks), so the install discloses them and aborts without consent.
- `--shared-file .claude/settings.json` is **required to actually apply the hooks** into settings.json; without it the install records the ledger + overlay but writes no hooks.
- `--scope project` installs into this checkout (`.gsd/capabilities/contribution-toolkit/` + a `.gsd-capabilities.json` ledger), keeping the enforcement project-scoped to the gsd-core checkout (never `~/.claude`). Use `--scope global` for `~/.gsd/...`.

Pin a specific release with `#v2.1.0` (a tag) or `#sha:<40-hex>` (an exact commit).

> The public repo was renamed to `gsd-contribution-toolkit` from its earlier `…-gate` name; GitHub redirects the old URL, so an existing `#v1.0.0` install does not hard-break.

### How the commands are delivered (the honest mechanism)

The toolkit install engine lays the command `.md`s into the runtime commands directory — mirroring `install.sh`'s symlink semantics, just copied from the published tree rather than symlinked from a local repo. The bundled hook scripts resolve and call the **LIVE gsd-core gate scripts** at runtime (no policy reimplementation). The skills are delivered the same way, from the bundle.

## Manage

```bash
node <gsd-core>/bin/gsd-tools.cjs capability status   --scope project
node <gsd-core>/bin/gsd-tools.cjs capability disable  contribution-toolkit --scope project
node <gsd-core>/bin/gsd-tools.cjs capability enable   contribution-toolkit --scope project
node <gsd-core>/bin/gsd-tools.cjs capability update   contribution-toolkit --scope project --yes
node <gsd-core>/bin/gsd-tools.cjs capability remove   contribution-toolkit --scope project
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

## Recovery offramp (FLOW-01)

When a contribution gate **denies** an action — or the `gsd-core-contribution` skill surfaces a real blocking issue mid-run — you are offered a GSD-native recovery choice rather than a dead-stop: **fix inline with `/gsd-quick`** for a trivial correction, or **route the issue through the GSD pipeline** (`/gsd-debug`, or `/gsd-discuss-phase`→`/gsd-plan-phase`→`/gsd-execute-phase`) as a tracked, resumable work item — then return to the submission once it is green.

This offramp is **advisory only**: the deny stays **fail-closed and unbypassable**, it NEVER suggests bypassing the gate or abusing `GSD_CONTRIB_OVERRIDE` to dodge a real failure, and no gate is weakened. (Surfaced from the contribution skill + `gsd-submit`/`gsd-review-sweep` commands.)

## Per-runtime behavior

This capability is delivered per-runtime, and what it enforces depends on the runtime:

- **Claude Code** — the full surface installs and the **12 `PreToolUse` gates enforce** at the harness boundary (once wired into `settings.json` via `--shared-file`). Skills + commands are delivered into the runtime dirs.
- **Other runtimes (Codex, OpenCode, …)** — the **2 skills are delivered cross-runtime** via the native `skills[]` contribution (the install engine copy-converts them into the runtime dialect), but those runtimes have **no `PreToolUse`-deny surface**, so the toolkit runs **advisory-only** there — its guidance is advice, not a hard block. Each skill carries an explicit advisory-only note to that effect.

In short: enforcement is a Claude-harness property; everywhere else the toolkit is advisory.

## Honesty & scope

This matters — please read it as written, don't over-read it:

- The capability's **GSD-loop surfaces are advisory**: it declares an empty gates[] array (`gates[]` is **empty**), and the one `plan:pre` contribution only fires inside a GSD command. They never see an arbitrary tool call. The capability does **not** reach the harness tool-call boundary — a direct issue/PR/push typed outside a GSD command never crosses a loop point, so the capability never sees it.
- The **harness-boundary enforcement** comes from the 12 `PreToolUse` hooks **once installed into `settings.json`** — those fire on matching tool calls while installed. That is a property of the **separate, personal installed PreToolUse hooks**, **not an inherent property of this (toggleable) capability**. Do not read this capability as "unbypassable."
- It is therefore **fully removable**: `disable`/`remove` genuinely takes the enforcement away (the hooks leave `settings.json` — the hooks *are* the enforcement).
- Deliberate bypass accountability rides on the existing per-worktree, append-only `GSD_CONTRIB_OVERRIDE` receipt (a logged reason-string, never a silent default) — this capability adds no new receipt mechanism.

## Provenance

- **Version:** 2.1.0
- **Source toolkit:** gsd-contrib-toolkit (private), v2.3 milestone
- The bundle is generated from canonical `hooks/`/`skills/`/`commands/` with a `--check` drift gate and validated against the LIVE gsd-core capability validators (tri-surface declared==shipped parity) before publish.
