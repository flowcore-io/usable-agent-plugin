# Codex

Verified 2026-08-07 against **Codex CLI 0.146.0** on macOS. Step 1 of the five acceptance
steps passes; see [`../../docs/supported-clients.md`](../../docs/supported-clients.md) for
exactly what is and is not verified.

## How Codex installs plugins

Codex does not install from a tarball. It reads *marketplaces*, and a marketplace source can
be a local path, `owner/repo`, or a Git URL. A marketplace is a directory containing
`.agents/plugins/marketplace.json`, which lists the plugins it offers and where each one
lives.

This repository is therefore both the marketplace and the plugin it offers — the descriptor
at `.agents/plugins/marketplace.json` points at `./`, the repo root, which is where
`plugin.json` lives.

## Install from GitHub

```bash
codex plugin marketplace add flowcore-io/usable-agent-plugin --ref main
codex plugin add usable@usable
```

Pin a released version once tags exist:

```bash
codex plugin marketplace add flowcore-io/usable-agent-plugin --ref v0.1.0
```

## Install from a local clone

Useful while developing — the marketplace points at your working tree.

```bash
git clone https://github.com/flowcore-io/usable-agent-plugin.git
cd usable-agent-plugin
codex plugin marketplace add .
codex plugin add usable@usable
```

The `<plugin>@<marketplace>` form is required. Plain `codex plugin add usable` fails with
`plugin requires --marketplace unless passed as <plugin>@<marketplace>`, because the plugin
and marketplace share the name `usable`.

## Verify

```bash
codex plugin list
```

Expect `usable@usable` with status `installed, enabled`.

Then confirm the model can actually see the skills:

```bash
codex exec --skip-git-repo-check "List the names of every skill available to you, one per line, then stop."
```

Expect these two entries, namespaced by plugin name:

```
usable:usable-knowledge-workflow
usable:usable-knowledge-capture
```

If they are absent, the plugin loaded but the skills did not — check
[`../../docs/troubleshooting.md`](../../docs/troubleshooting.md).

## Try it

From inside a repository whose decisions are recorded in Usable:

```bash
codex exec "Search Usable for how we handle authentication in this service, then summarise what you found with sources."
```

A working install cites retrieved sources. An install where Usable is unreachable says so
explicitly rather than answering confidently from general knowledge — that degraded-mode
honesty is itself the thing being tested.

## MCP caveat

If `~/.codex/config.toml` already contains an `[mcp_servers.usable]` entry, it will shadow the
plugin's declaration and you cannot tell the two apart from `codex mcp list`. To test the
plugin's own MCP configuration, remove or rename the user-level entry first.

## Uninstall

```bash
codex plugin remove usable@usable
codex plugin marketplace remove usable
```

Removing files does not revoke your OAuth grant. To fully disconnect, also revoke the grant in
your Usable account settings and clear the stored credentials from Codex.
