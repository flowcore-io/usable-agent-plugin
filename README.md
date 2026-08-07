# Usable Agent Plugin

A portable [Agent Plugins 1.0.0](https://agent-plugins.org/) package that gives coding agents
a knowledge-first workflow backed by [Usable](https://usable.dev): search what the team
already decided before implementing, retrieve complete sources, separate evidence from
assumptions, and verify before claiming success.

> **Status: pre-release (`0.1.0`), not yet released.**
> No client has completed acceptance testing, so no client is listed as supported yet. See
> [`docs/supported-clients.md`](docs/supported-clients.md).

## Contents

| Component | What it does |
|---|---|
| [`usable-knowledge-workflow`](skills/usable-knowledge-workflow/SKILL.md) | Read-first loop: search Usable, fetch complete sources, rank by verification and freshness, write a knowledge receipt, verify before claiming done |
| [`usable-knowledge-capture`](skills/usable-knowledge-capture/SKILL.md) | Write verified outcomes back to Usable — gated on verification, deduplication, and explicit user confirmation |
| [`mcp.json`](mcp.json) | Declares the hosted Usable MCP server at `https://usable.dev/api/mcp` over `streamable-http` |

The package contains **no executable code and no credentials** — only Markdown instructions
and two JSON files. Read `skills/` before enabling it.

## Install

```bash
VERSION=0.1.0
curl -fsSLO "https://github.com/flowcore-io/usable-agent-plugin/releases/download/v${VERSION}/usable-agent-plugin-${VERSION}.tar.gz"
curl -fsSLO "https://github.com/flowcore-io/usable-agent-plugin/releases/download/v${VERSION}/usable-agent-plugin-${VERSION}.tar.gz.sha256"
shasum -a 256 -c "usable-agent-plugin-${VERSION}.tar.gz.sha256"
tar -xzf "usable-agent-plugin-${VERSION}.tar.gz"
```

Extract into your client's plugin directory, then restart the client. Full steps, upgrade,
rollback, and uninstall: [`docs/installation.md`](docs/installation.md).

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
