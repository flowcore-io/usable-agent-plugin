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

| Client | Plugin loading | Agent Skills | `streamable-http` MCP | OAuth 2.1 + PKCE | OS tested | Last tested | Status |
|---|---|---|---|---|---|---|---|
| Claude Code | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |
| Codex | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |
| Warp / Oz | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |
| Cursor | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |
| Opencode | ⏳ | ⏳ | ⏳ | ⏳ | — | — | Untested |

Launch targets are [open decision #2](../README.md#open-decisions). Two clients must reach
✅ across all columns before `1.0.0`.

## Per-client notes

Each section gets filled in during acceptance testing with the plugin directory path, the
tested client version, how skills are enabled, how MCP is configured and removed, and any
deviations found.

### Claude Code
Untested. Reads project skills from `.claude/skills/` only; a plugin package placed elsewhere
may not be discovered. Needs verification of whether the plugin loader picks up `skills/`
from an Agent Plugins package directly.

### Codex
Untested. Needs verification of plugin directory location and whether `mcp.json` from the
package is merged into the client's MCP configuration or must be declared separately.

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
