# Usable Agent Plugin

A portable [Agent Plugins 1.0.0](https://agent-plugins.org/) package that gives coding agents
a knowledge-first workflow backed by [Usable](https://usable.dev): search what the team
already decided before implementing, retrieve complete sources, separate evidence from
assumptions, and verify before claiming success.

> **Status: pre-release (`0.1.0`), not yet released.**
> Codex is partially verified — the package loads and both skills reach the model. No client
> has completed all five acceptance steps, so none is listed as fully supported. See
> [`docs/supported-clients.md`](docs/supported-clients.md).

## Contents

| Component | What it does |
|---|---|
| [`usable-knowledge-workflow`](skills/usable-knowledge-workflow/SKILL.md) | Read-first loop: search Usable, fetch complete sources, rank by verification and freshness, write a knowledge receipt, verify before claiming done |
| [`usable-knowledge-capture`](skills/usable-knowledge-capture/SKILL.md) | Write verified outcomes back to Usable — gated on verification, deduplication, and explicit user confirmation |
| [`mcp.json`](mcp.json) | Declares the hosted Usable MCP server at `https://usable.dev/api/mcp` over `streamable-http` |

No credentials ship in this package, anywhere. The **release archive** contains only Markdown
and JSON — no executable code, no dependencies, no install hooks. A **marketplace or Git
install** materialises the repository, so `scripts/` and `tests/` are present in the installed
copy; the client does not execute them, but they are there.

All instruction text is in `skills/`. Read it before enabling.

## Install

**Codex** (verified — loads the package and both skills):

```bash
codex plugin marketplace add flowcore-io/usable-agent-plugin --ref main
codex plugin add usable@usable
```

Confirm the model can see them:

```bash
codex exec "List the names of every skill available to you, one per line, then stop."
# expect: usable:usable-knowledge-workflow / usable:usable-knowledge-capture
```

Other clients, tagged releases with checksum verification, upgrade, rollback, and uninstall:
[`docs/installation.md`](docs/installation.md) and [`examples/codex/README.md`](examples/codex/README.md).

## Authentication

Nothing to configure by hand. The plugin declares a URL; your client performs OAuth 2.1 with
PKCE and stores the tokens itself.

For a read-only install, grant `fragments.read` and `workspace.read`. Add
`fragments.create` and `fragments.update` only if you want the capture skill to work.

Details and the verified discovery chain: [`docs/authentication.md`](docs/authentication.md).

**Never add a token, API key, client secret, or static `Authorization` header to `mcp.json`.**
CI rejects it.

## Documentation

- [Installation](docs/installation.md) — install, upgrade, rollback, uninstall
- [Authentication](docs/authentication.md) — OAuth 2.1 discovery chain, scopes, revocation
- [Supported clients](docs/supported-clients.md) — compatibility matrix and known deviations
- [Permissions and data flow](docs/permissions-and-data-flow.md) — for security review
- [Threat model](docs/threat-model.md) — threats, mitigations, residual risk
- [Troubleshooting](docs/troubleshooting.md) — diagnosis by symptom

## Validate locally

```bash
node scripts/validate-package.mjs
```

Checks manifest and MCP schema conformance, skill frontmatter and layout, path containment and
symlink safety, and credential patterns. Requires Node 20+, no dependencies.

## Trust boundary

Agent Plugins 1.0.0 standardizes package structure and loading. It does **not** define
installation, registries, trust, permissions, sandboxing, signatures, portable secrets,
updates, or rollback.

Schema validity therefore implies nothing about safety — for this package or any other.
Evaluate contents, not conformance. The strongest control available to you is the set of OAuth
scopes you actually grant; prompt instructions are not an enforcement boundary.

## Open decisions

Tracked from the PRD, unresolved:

1. Repository name — provisionally `flowcore-io/usable-agent-plugin`
2. Launch client targets — two must pass acceptance before `1.0.0`
3. ~~License~~ — resolved: MIT
4. Whether `1.0.0` ships read-only, or includes verified writeback
5. Headless authorization profile — `client_credentials` and `device_code` are advertised but neither is wired up or tested
6. Public data-processing, retention, and revocation disclosure wording
7. Whether releases need signed attestations in addition to tags and checksums
8. Ownership of client compatibility testing as client plugin loaders change

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Skill wording changes agent behavior, so those
changes get reviewed as carefully as code. Security issues go through
[`SECURITY.md`](SECURITY.md), not public issues.

## License

[MIT](LICENSE)
