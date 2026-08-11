# Client configuration examples

A client gets a directory here once its steps have actually been run. Publishing a
configuration snippet we have not executed would be a support claim we cannot back — the
point of the compatibility matrix is that support is asserted from evidence, not from a
client advertising Agent Plugins compatibility.

- [`codex/`](codex/) — verified 2026-08-07 on Codex CLI 0.146.0. Loads the package and both
  skills; MCP and OAuth steps still outstanding.
- [`claude-code/`](claude-code/) — verified 2026-08-11 on Claude Code 2.1.227. Marketplace
  registers and the plugin installs and enables; skills-reaching-the-model and MCP steps still
  outstanding.

Still to test: Cursor, Warp / Oz, Opencode.

Each entry records the install commands, how to verify, the tested client version, and any
deviations found. See [`../docs/supported-clients.md`](../docs/supported-clients.md) for the
five acceptance steps.

If you have run the acceptance steps, please
[file a compatibility report](https://github.com/flowcore-io/usable-agent-plugin/issues/new?template=compatibility_report.yml) —
including failures.

Note: examples are repository-only. They are excluded from release archives.
