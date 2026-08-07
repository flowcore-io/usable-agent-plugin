# Changelog

All notable changes to this project are documented here.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

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

### Notes
- Verified against production: the MCP endpoint returns `401` with a `WWW-Authenticate`
  header carrying `resource_metadata`; protected-resource metadata (RFC 9728) and
  authorization-server metadata (RFC 8414) both resolve; PKCE `S256` and dynamic client
  registration are supported.
- Recorded deviation: the path-suffixed protected-resource URL
  (`/.well-known/oauth-protected-resource/api/mcp`) returns HTML rather than JSON or `404`.
  Clients that follow the `WWW-Authenticate` header are unaffected.
- No client has completed acceptance testing, so no client is listed as supported.

[Unreleased]: https://github.com/flowcore-io/usable-agent-plugin/commits/main
