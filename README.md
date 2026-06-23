# contrib-gate — GSD capability

A `role:feature` GSD capability that installs fail-closed `PreToolUse` enforcement gates
(12 gates + 1 `UserPromptSubmit` advisory) for preparing `open-gsd/gsd-core` contributions.
The bundled hook scripts resolve and call the **LIVE** gsd-core gate scripts at runtime.

## Install (GSD 1.6.0+)

    node <gsd-core>/gsd-core/bin/gsd-tools.cjs capability install \
      https://github.com/davesienkowski/gsd-contrib-gate.git#v1.0.0 --scope project --yes

`install`/`on`/`off`/`status`/`remove` are managed by gsd-core's capability engine
(consent + ledger tracked). Toggling **off** genuinely removes the enforcement; the
capability itself is never "unbypassable" — only the installed PreToolUse hooks are, while installed.

Built with the gsd-contrib-toolkit (v2.1 — Capability-Native Distribution).
