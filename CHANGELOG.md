# Changelog

All notable changes to this project are documented here.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

### Added
- Usable brand assets: `assets/usable-icon.svg` (composer icon) and `assets/usable-logo.png`
  (600×600 logo), taken from the Usable brand kit.
- Codex interface metadata under the `extensions["com.openai"]` namespace in `plugin.json`:
  display name, short and long descriptions, developer name, category, website, privacy and
  terms URLs, brand colour `#347cbf`, icon and logo paths, and a default prompt.
- Validation of client-extension file references: paths must be plugin-relative, must stay
  inside the plugin root, and must exist. Three new self-tests cover a missing asset, a path
  escaping the root, and a namespace without a reverse domain.

### Changed
- `assets/` is now included in the release archive allowlist.
- `homepage` and author URL now use the canonical `https://www.usable.dev`, which is where
  the apex domain redirects.

### Notes
- Codex surfaced the plugin with a generic icon and "Website: Unavailable" because it reads
  presentation metadata from an `interface` object, not from the Agent Plugins `homepage`
  field. **Whether Codex reads that object from the inline `com.openai` extension has not been
  verified visually** — the CLI exposes no way to inspect resolved interface metadata. If the
  icon and website still do not render, the fallback is a `.codex-plugin/plugin.json` overlay,
  which must be added carefully because a malformed overlay could disturb skill discovery that
  currently works.

## 0.1.0 — 2026-08-07

First prerelease. No client has completed all five acceptance steps, so no client is listed
as fully supported; Codex loads the package and both skills.

### Added
- Initial package scaffold conforming to Agent Plugins `1.0.0`.
- `plugin.json` manifest, MIT licensed.
- `usable-knowledge-workflow` skill — read-first retrieval loop with knowledge receipts,
  freshness ranking, verification requirements, and explicit degraded mode.
- Reference material for the workflow skill: evidence and verification, knowledge lifecycle,
  and security boundaries.
- `usable-knowledge-capture` skill — verified writeback, gated on deduplication, redaction,
  and user confirmation.
- `mcp.json` declaring the hosted Usable MCP server at `https://usable.dev/api/mcp` over
  `streamable-http`, URL-only with no credentials.
- Documentation: installation, authentication, supported clients, permissions and data flow,
  threat model, troubleshooting.
- `scripts/validate-package.mjs` — schema, skill, path-containment, symlink, and credential
  checks with no dependencies.
- `scripts/build-release.mjs` — allowlisted release archive with SHA-256 checksum.
- CI: pull-request validation and tag-triggered release workflows.
- Governance: `SECURITY.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, CODEOWNERS, issue and
  pull-request templates.
- Codex marketplace descriptor at `.agents/plugins/marketplace.json`. Codex installs plugins
  from marketplaces rather than archives, so the package was not installable there without it.
- Verified Codex install guide at `examples/codex/README.md`.

### Verified
- OAuth discovery against production: `POST /api/mcp` returns `401` with a `WWW-Authenticate`
  header carrying `resource_metadata`; RFC 9728 protected-resource metadata and RFC 8414
  authorization-server metadata both resolve; PKCE `S256` and dynamic client registration are
  supported. Both the origin-level and RFC 9728 path-suffixed metadata URLs return the same
  JSON document, and unmatched `.well-known` paths return a JSON `404`.
- Codex CLI 0.146.0 on macOS: the package installs and both skills reach the model as
  `usable:usable-knowledge-workflow` and `usable:usable-knowledge-capture`.
- Reproducible release archive: two consecutive builds are byte-identical, under both bsdtar
  and GNU tar.

### Known limitations
- No client has completed all five acceptance steps, so none is listed as fully supported.
  Codex passes step 1 only.
- The Codex MCP step is **inconclusive, not passed**: the test machine already had an
  identical `[mcp_servers.usable]` entry in `~/.codex/config.toml`, so the entry survived
  removing the plugin and cannot be attributed to it. A plugin server keyed `usable` may also
  collide with a user-level server of the same name.
- Claude Code is untested and expected to need a different format — it uses
  `.claude-plugin/plugin.json` and was not part of the Agent Plugins launch lineup.
- Headless authorization is not wired up. `client_credentials` and `device_code` are
  advertised by the authorization server but neither is implemented or tested here.
- The metadata advertises `resource: https://usable.dev` while the endpoint served is
  `https://usable.dev/api/mcp`. Whether the authorization server accepts the endpoint URL as a
  resource indicator, and whether token audiences validate at the endpoint, is untested.
- No signed release attestations yet; releases carry immutable tags and SHA-256 checksums only.
- The release archive contains only Markdown and JSON. A marketplace or Git install
  materialises the repository, so `scripts/` and `tests/` are present in the installed copy;
  the client does not execute them.

[0.1.0]: https://github.com/flowcore-io/usable-agent-plugin/releases/tag/v0.1.0
