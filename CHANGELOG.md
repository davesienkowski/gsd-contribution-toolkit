# Changelog

## v2.1.1 — 2026-06-23

**Docs release** — the bundle logic is byte-identical to v2.1.0 (hooks + skills + commands + manifest surface unchanged; only the version string, README, and docs change).

- **Single authoritative README:** rewritten to GitHub best practices with a proper title, a capabilities-by-role section (codeowner / maintainer / contributor / collaborator), the remote capability-install flow, an architecture/honesty section, and a "For reviewers" section for gsd-core maintainers.
- **Reviewer doc set added** under `docs/`: `cross-runtime-delivery-model.md` (architecture), `reuse-and-methodology.md` (reuse map + methodology alignment), and `upstream-feature-requests.md` (two capability-framework asks for gsd-core maintainers, each citing this toolkit as the reference implementation).

## v2.1.0 — 2026-06-23

Cross-runtime honesty + on/off full-surface toggle (from the v2.3 source milestone). **MINOR** bump (additive content; no enforcement-surface change — the 13 hooks + 5 commands are byte-identical to v2.0.0).

- **Per-runtime delivery made explicit:** the 2 skills now carry an "Advisory-only on non-Claude runtimes" section. On Claude the 12 `PreToolUse` gates enforce at the harness boundary; on other runtimes (Codex, OpenCode, …) the skills are delivered via the native `skills[]` contribution but run **advisory-only** (no `PreToolUse`-deny surface exists there). README gains a "Per-runtime behavior" section.
- **Manifest title** cased to "GSD Contribution Toolkit".
- **Honesty unchanged:** `gates[]` still empty; the harness-boundary property belongs to the separate personal `PreToolUse` hooks, not this capability; `disable`/`remove` genuinely removes the enforcement; `GSD_CONTRIB_OVERRIDE` stays logged.
- Install via the gsd-core git capability adapter from `#v2.1.0` (`--scope project --yes --shared-file .claude/settings.json`).

## v2.0.0 — 2026-06-23

Rename + self-contained distribution. **MAJOR** bump (rename of the capability AND new shipped surfaces).

- **Renamed** the capability `contribution-gate` → `contribution-toolkit` (the public repo was renamed `gsd-contribution-gate` → `gsd-contribution-toolkit`; GitHub redirects the old URL so an existing `#v1.0.0` install does not hard-break).
- **Self-contained bundle** — the install now actually delivers the full surface from the published tree: **13 hooks** (12 `PreToolUse` gates + 1 `UserPromptSubmit` advisory) + **2 skills** (`gsd-core-contribution`, `maintainer-review-sweep`) + **5 commands** (`gsd-submit`, `gsd-review-sweep`, `gsd-triage-assist`, `gsd-release-preflight`, `gsd-ruleset-drift`). The v1.0 README/CHANGELOG claimed it "ships skills/commands" while the v1.0 bundle shipped NEITHER (hooks-only); that claim is now TRUE — skills + commands are shipped in-bundle and delivered by the install engine into the runtime commands/skills dirs.
- **FLOW-01 recovery offramp** documented: on a gate deny, the contribution skill + `gsd-submit`/`gsd-review-sweep` offer a GSD-native recovery (`/gsd-quick` inline fix, or route through `/gsd-debug` / discuss→plan→execute) — advisory only, never bypassing the fail-closed deny.
- **Honesty unchanged:** `gates[]` is still **empty**; the capability is advisory-only at GSD-loop points and does NOT reach the harness tool-call boundary; the unbypassable `PreToolUse` property belongs to the SEPARATE personal hooks, not this capability; the `GSD_CONTRIB_OVERRIDE` override stays logged (per-worktree, reason-string), never silent — no new mechanism.
- Install via the gsd-core git capability adapter from `#v2.0.0` (`--scope project --yes --shared-file .claude/settings.json`).

## v1.0.0 — 2026-06-23

Initial published distribution of the `contribution-gate` GSD capability.

- 12 `PreToolUse` gates + 1 `UserPromptSubmit` advisory (the 13 `hooks[]`).
- 2 skills (`gsd-core-contribution`, `maintainer-review-sweep`) + 5 prose-disclosed commands.
- One advisory `plan:pre` contribution; default-off `workflow.gsd_contrib_enforcement` flag; `gates: []`.
- Bundled hook scripts resolve + call the LIVE gsd-core gate scripts at runtime (no policy reimplementation).
- Installable via the gsd-core git capability adapter (GSD 1.6.0+).
