# Installation

> **Status: pre-release.** No client has completed acceptance testing yet, so no client is
> listed as supported. See `supported-clients.md`. The steps below describe the intended
> installation path and are expected to change as each client is validated.

Agent Plugins 1.0.0 standardizes package *structure and loading*. It deliberately does not
standardize installation, registries, updates, or rollback. There is therefore no single
install command that works everywhere — each client needs its own instructions, which is why
per-client documentation is a release blocker rather than optional polish.

## What you are installing

| Component | Effect |
|---|---|
| `skills/usable-knowledge-workflow` | Instructions telling the agent to search Usable before implementing, and to verify before claiming success |
| `skills/usable-knowledge-capture` | Instructions for writing verified outcomes back to Usable, gated on user confirmation |
| `mcp.json` | Declares one remote MCP server at `https://usable.dev/api/mcp` over `streamable-http` |

Skills are plain Markdown. Read them before enabling them — all instruction text is in
`skills/`, and the package contains no executable code.

## Prerequisites

- An agent client that loads Agent Plugins 1.0.0 packages
- A Usable account
- Outbound HTTPS to `usable.dev` and `auth.flowcore.io`
- A browser on the same machine, for the OAuth consent step

## Install from a tagged release

Pin a version. Do not track `main` for anything you rely on.

```bash
VERSION=0.1.0

curl -fsSLO "https://github.com/flowcore-io/usable-agent-plugin/releases/download/v${VERSION}/usable-agent-plugin-${VERSION}.tar.gz"
curl -fsSLO "https://github.com/flowcore-io/usable-agent-plugin/releases/download/v${VERSION}/usable-agent-plugin-${VERSION}.tar.gz.sha256"

shasum -a 256 -c "usable-agent-plugin-${VERSION}.tar.gz.sha256"
```

Verify the checksum before extracting. Then extract into your client's plugin directory —
the exact location is client-specific and will be documented per client in
`supported-clients.md` as each is validated.

```bash
tar -xzf "usable-agent-plugin-${VERSION}.tar.gz"
```

## Install from a clone

Useful for evaluation and development. Note that a clone contains repository-only files
(`tests/`, `scripts/`, `.github/`) which are excluded from release archives.

```bash
git clone https://github.com/flowcore-io/usable-agent-plugin.git
cd usable-agent-plugin
git checkout v0.1.0
node scripts/validate-package.mjs
```

## Authenticate

The plugin ships no credentials. On first use of a Usable tool, your client should run the
OAuth 2.1 flow and open a browser for consent. See `authentication.md` for the full
discovery chain and the scopes to grant.

## Confirm it works

1. The client lists `usable-knowledge-workflow` and `usable-knowledge-capture` among its
   skills.
2. The client lists a `usable` MCP server, connected.
3. Ask a question about a documented decision in your workspace. A correct install cites
   retrieved sources. An unauthenticated install says it could not reach Usable — it does
   not invent an answer.

If step 3 produces confident detail with no sources cited, treat the install as broken and
check `troubleshooting.md`.

## Upgrade

```bash
# fetch, verify checksum, extract over the old version, restart the client
```

Read `CHANGELOG.md` first. Skill wording changes agent behavior, so review the diff for
anything you depend on.

## Rollback

Re-install the previous tag. Because releases are immutable and checksummed, rollback is
just installing an older archive. Nothing in the plugin holds state, so there is no
migration to reverse.

## Uninstall

1. Remove the plugin directory from your client's plugin location.
2. Remove the `usable` MCP server from the client's configuration, if it persists it
   separately.
3. Revoke the OAuth grant in your Usable account settings.
4. Clear the stored credentials from the client's credential store.

Steps 3 and 4 matter: removing files does not revoke a token.
