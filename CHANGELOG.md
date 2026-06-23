# Changelog

## v1.0.0 — 2026-06-23

Initial published distribution of the `contrib-gate` GSD capability.

- 12 `PreToolUse` gates + 1 `UserPromptSubmit` advisory (the 13 `hooks[]`).
- 2 skills (`gsd-core-contribution`, `maintainer-review-sweep`) + 5 prose-disclosed commands.
- One advisory `plan:pre` contribution; default-off `workflow.gsd_contrib_enforcement` flag; `gates: []`.
- Bundled hook scripts resolve + call the LIVE gsd-core gate scripts at runtime (no policy reimplementation).
- Installable via the gsd-core git capability adapter (GSD 1.6.0+).
