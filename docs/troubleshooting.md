# Troubleshooting

## The plugin does not load

Validate the package first — this catches most structural problems:

```bash
node scripts/validate-package.mjs
```

Then check:

- `plugin.json` is at the package root, not nested one directory deeper. Extracting a tarball
  often creates an extra wrapper directory.
- the client's plugin directory is the one you actually extracted into
- the client was restarted after installation
- the client supports Agent Plugins 1.0.0 at all — see `supported-clients.md`

## Skills do not appear

- each skill is an immediate child of `skills/`, and contains a `SKILL.md`
- frontmatter has both `name` and `description`, and `name` matches the directory name
- the client may require skills to be enabled explicitly after discovery
- some clients only read skills from their own conventional path rather than from a plugin
  package; Claude Code, for example, reads `.claude/skills/`. Check the per-client notes.

## MCP server will not connect

**401 that never resolves.** The client is not completing OAuth. Confirm it supports
interactive OAuth 2.1 for MCP; some clients only support static headers, which this plugin
deliberately does not provide.

**Claude Code shows `invalid_scope` with a long list of scopes.** Upgrade the plugin to a
version whose `.mcp.json` explicitly declares the read-only OAuth scopes. Then remove the old
OAuth grant or cached MCP credentials and authenticate again; existing sessions do not gain the
correct scope set automatically.

**Discovery fails.** Verify the chain by hand:

```bash
curl -sS -i -X POST https://usable.dev/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"probe","version":"0"}}}'
```

Expect `401` with:

```
www-authenticate: Bearer resource_metadata="https://usable.dev/.well-known/oauth-protected-resource"
```

Then:

```bash
curl -sS https://usable.dev/.well-known/oauth-protected-resource
curl -sS https://usable.dev/.well-known/oauth-protected-resource/api/mcp
curl -sS https://usable.dev/.well-known/oauth-authorization-server
```

All three must return JSON, and the first two must return the same document — a client is
free to reach the metadata either by reading `resource_metadata` from the challenge header or
by constructing the path-suffixed URL per RFC 9728 §3.1.

**Browser never opens.** Headless or remote environments cannot complete the interactive
flow. Do not work around this by adding a static credential to `mcp.json`. See the headless
section of `authentication.md`.

**Transport rejected.** The endpoint is `streamable-http`. Clients that only support `stdio`
cannot use the hosted server.

## The agent answers without searching

Most likely the skill is not loaded, or the client is not surfacing it to the model. Check
that the skill appears in the client's skill list, and ask directly: "which skills do you
have available?"

If the skill is loaded and the agent still skips retrieval, the model may not consider the
task to warrant it. Ask explicitly: "search Usable first, then answer."

## The agent claims to have searched but cites nothing

This is the failure mode the skill is written to prevent, and it is worth reporting as an
issue. Correct behavior with MCP unavailable is an explicit statement that Usable could not
be reached — not a confident answer with no sources.

Include the client name and version, the prompt, and the response in your report.

## Searches return nothing useful

- confirm you are querying the right workspace
- use a descriptive natural-language intent rather than keywords; these tools are semantic
- drop repository tag filters if coverage looks thin
- the workspace may genuinely not contain the knowledge — a correct empty result

## Writes fail

- the granted token may lack `fragments.create` and `fragments.update`; re-authorize with the
  needed scopes
- the capture skill requires user confirmation by design; it will not write silently
- verify the outcome first — the skill refuses to capture unverified results

## Checksum mismatch on download

Do not extract the archive. Re-download; if it still mismatches, treat it as a potentially
compromised artifact and report it per `SECURITY.md`.

## Filing a useful issue

Include the client name and exact version, OS, plugin version, output of
`node scripts/validate-package.mjs`, and the verbatim error text. Redact tokens and
workspace identifiers you consider sensitive.
