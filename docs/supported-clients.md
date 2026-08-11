# Supported clients

## Support policy

A client is listed as **supported** only after it has passed acceptance testing:

1. the package loads and both skills are discoverable
2. the `usable` MCP server connects and completes OAuth
3. a knowledge retrieval succeeds and the agent cites its sources
4. with MCP unavailable, skills still load and the agent reports degraded mode instead of
   fabricating results
5. install, upgrade, rollback, and uninstall have each been performed

Until then a client is **untested**, and any listed steps are provisional. We do not claim
support based on a client advertising Agent Plugins compatibility.

## Matrix

Legend: ✅ verified · ⚠️ partial · ❌ not supported · ⏳ untested

| Client | Plugin loading | Agent Skills | Remote MCP | OAuth 2.1 + PKCE | OS tested | Last tested | Status |
|---|---|---|---|---|---|---|---|
| Codex CLI 0.146.0 | ✅ | ✅ | ⏳ | ⏳ | macOS | 2026-08-07 | Partial (1 of 5) |
| Claude Code 2.1.227 | ✅ | ⏳ | ⏳ | ⏳ | macOS | 2026-08-11 | Partial (1 of 5) |
| Warp / Oz | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |
| Cursor | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |
| Opencode | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |

Launch targets are [open decision #2](../README.md#open-decisions). Two clients must reach
✅ across all columns before `1.0.0`.

Agent Plugins 1.0.0 was published on 2026-08-06 with launch support announced for ChatGPT and
Codex, Cursor, GitHub Copilot, Kiro, and VS Code. Claude Code is not among them and uses its
own manifest format, so this package ships both sets of manifests — see
[Supporting two formats](#supporting-two-formats).

## Supporting two formats

The two clients disagree on filenames and on one transport identifier, so both are shipped.
Only `skills/` is genuinely shared.

| Concern | Agent Plugins | Claude Code |
|---|---|---|
| Plugin manifest | `plugin.json` | `.claude-plugin/plugin.json` |
| Marketplace catalogue | `.agents/plugins/marketplace.json` (Codex) | `.claude-plugin/marketplace.json` |
| MCP config | `mcp.json` | `.mcp.json` |
| Remote transport | `streamable-http` | `http` |
| Skills | `skills/` | `skills/` |

The duplication is a drift risk, so CI guards it: the validator asserts the two MCP documents
declare the same servers pointing at the same URLs, and that the two plugin manifests agree on
name and version. Editing one and forgetting the other fails the build.

This mirrors what other multi-client plugins do — Slack's official Claude Code plugin ships
`.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/` and `.agents/` side by side.

## Per-client notes

Each section gets filled in during acceptance testing with the plugin directory path, the
tested client version, how skills are enabled, how MCP is configured and removed, and any
deviations found.

### Codex
Partially verified on 2026-08-07 against Codex CLI 0.146.0 on macOS.

**Step 1 passed.** The package loads and both skills are exposed to the model, namespaced by
plugin name:

```
usable:usable-knowledge-workflow
usable:usable-knowledge-capture
```

Codex installs plugins from *marketplaces* rather than from archives, so this repository also
ships a marketplace descriptor at `.agents/plugins/marketplace.json`. See
[`../examples/codex/README.md`](../examples/codex/README.md) for exact steps.

**Step 2 inconclusive, not passed.** The MCP entry could not be attributed to the plugin: the
test machine already had an identical `[mcp_servers.usable]` entry in `~/.codex/config.toml`
pointing at the same URL, so `codex mcp list` showed the server both with the plugin installed
and after removing it. Re-test on a machine with no pre-existing `usable` MCP server before
marking this ✅. This also flags a real collision risk: a plugin server keyed `usable` may be
shadowed by, or conflict with, a user-level server of the same name.

**Steps 3–5 untested.**

Note on packaging: a local or Git marketplace materialises the repository as the plugin root,
so `scripts/`, `tests/`, and `.github/` are present in the installed copy. They are not
executed by the client, but the "Markdown and JSON only" property holds strictly for the
release archive, not for a marketplace install.

### Claude Code
Partially verified on 2026-08-11 against Claude Code 2.1.227 on macOS.

**Step 1 passed.** The marketplace registers and the plugin installs and enables:

```
usable@usable   Version: 0.1.0   Scope: user   Status: ✔ enabled
```

See [`../examples/claude-code/README.md`](../examples/claude-code/README.md) for exact steps.
Note `claude plugin marketplace add ./` — a bare `.` is rejected.

**Skills not yet confirmed reaching the model.** The package installs, but we have not yet
observed `usable:usable-knowledge-workflow` and `usable:usable-knowledge-capture` offered in a
session. Claude Code exposes no CLI listing of available skills, so this needs an interactive
check.

**Step 2 inconclusive, and for an identified reason.** The test machine had a user-level
`[mcpServers.usable]` entry in `~/.claude.json` pointing at the same URL, which shadows the
plugin's declaration — no `plugin:usable:usable` server appeared, while other plugins' servers
did show under that prefix. This is direct evidence of the collision risk previously recorded
as a hypothesis for Codex. Re-test on a machine with no pre-existing `usable` server.

**Steps 3–5 untested.**

### Warp / Oz
Untested. Supports MCP servers and skills natively; needs verification of whether an Agent
Plugins package is loaded as a unit or whether components must be registered individually.

### Cursor
Untested. Needs verification of MCP transport support for `streamable-http` and of the
interactive OAuth flow.

### Opencode
Untested. Needs verification of plugin loading, skill discovery, and OAuth handling.

## Known ecosystem deviations

**Path-suffixed protected resource metadata returned HTML. Fixed 2026-08-07.**
`https://usable.dev/.well-known/oauth-protected-resource/api/mcp` used to serve the site's
HTML rather than JSON, so clients that build that URL by convention — appending the resource
path, per RFC 9728 §3.1 — instead of reading the `resource_metadata` parameter from the
`WWW-Authenticate` header failed discovery. Both URLs now return the same JSON document, and
unmatched paths under `.well-known` return a JSON `404` rather than the app shell. No client
workaround is needed.

**`plain` PKCE is advertised.** The authorization server lists both `S256` and `plain` in
`code_challenge_methods_supported`. Clients must use `S256`.

## Reporting a result

Open an issue with the client name and exact version, OS and version, plugin version, which
of the five acceptance steps passed, and the verbatim error text for any failure. Include
whether skills still loaded when MCP was unavailable — component-level failure isolation is
the behavior most likely to differ between clients.
