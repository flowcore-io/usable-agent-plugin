# Claude Code

Verified 2026-08-11 against **Claude Code 2.1.227** on macOS. The marketplace registers and the
plugin installs and enables. See
[`../../docs/supported-clients.md`](../../docs/supported-clients.md) for exactly what is and is
not verified.

## How Claude Code installs plugins

Claude Code reads its own manifests, separate from the Agent Plugins ones:

| File | Purpose |
|---|---|
| `.claude-plugin/marketplace.json` | Marketplace catalogue, at the repository root |
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.mcp.json` | MCP servers — note the leading dot, and `type: "http"` rather than `streamable-http` |
| `skills/` | Shared with Agent Plugins, no duplication needed |

This repository is both the marketplace and the single plugin it lists, so the marketplace
entry points at `./`.

## Install from GitHub

```bash
claude plugin marketplace add flowcore-io/usable-agent-plugin
claude plugin install usable@usable
```

## Install from a local clone

```bash
git clone https://github.com/flowcore-io/usable-agent-plugin.git
cd usable-agent-plugin
claude plugin marketplace add ./
claude plugin install usable@usable
```

Use `./`, not `.`. A bare dot is rejected with
`Invalid marketplace source format. Try: owner/repo, https://..., or ./path`.

The `<plugin>@<marketplace>` form is required, and here both are named `usable`, so the
command reads a little oddly but is correct.

## Verify

```bash
claude plugin list
```

Expect:

```
usable@usable
  Version: 0.1.0
  Scope: user
  Status: enabled
```

If the install summary says `Run /reload-plugins to activate.`, run that inside Claude Code.

Then confirm the skills reached the model — ask it inside a session:

> Which skills do you have available?

Plugin skills are namespaced, so expect `usable:usable-knowledge-workflow` and
`usable:usable-knowledge-capture`.

## MCP server

The plugin declares the Usable MCP server in `.mcp.json` with the public OAuth client ID
`mcp_oauth_client` and an explicit read-only scope list. The explicit list prevents Claude Code
from requesting every scope advertised by the server, which Keycloak rejects with
`invalid_scope`. Claude Code performs the OAuth flow and stores tokens itself; no credential ships
in this package.

**A user-level server named `usable` will shadow it.** If you already have one, the plugin's
declaration is silently ignored. Plugin-provided servers appear as `plugin:usable:usable` in
`claude mcp list`; a bare `usable` entry is your own config, not this plugin's.

```bash
claude mcp list | grep usable
```

To let the plugin's declaration take effect:

```bash
claude mcp remove usable
```

Worth checking while you are there: if your existing entry uses a static
`Authorization: Bearer ...` header, that is a long-lived credential sitting in
`~/.claude.json`. The OAuth flow is the better path, and removing the entry lets the plugin
provide it.

## Try it

From a repository whose decisions are recorded in Usable, ask something the team has already
decided. A working install cites retrieved sources. An install that cannot reach Usable says so
explicitly rather than answering confidently from general knowledge — that degraded-mode
honesty is itself the thing being tested.

## Uninstall

```bash
claude plugin uninstall usable@usable
claude plugin marketplace remove usable
```

Removing files does not revoke your OAuth grant. To fully disconnect, revoke it in your Usable
account settings and clear the stored credentials from Claude Code.
