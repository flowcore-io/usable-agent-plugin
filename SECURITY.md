# Security policy

## Reporting a vulnerability

**Do not open a public issue.**

Use GitHub private vulnerability reporting on this repository
(Security → Report a vulnerability), or email **security@usable.dev**.

Please include the affected version or commit, what an attacker can achieve, reproduction
steps, and any suggested fix. Redact real credentials — describe the shape of a value rather
than including it.

We aim to acknowledge within 3 business days, give an initial assessment within 10, and
coordinate disclosure once a fix is available. We will credit you unless you prefer otherwise.

## In scope

- credentials, private identifiers, or customer data present in the package or Git history
- skill instructions that can be manipulated into unauthorized actions
- prompt injection paths that survive the documented mitigations
- release, tagging, or CI weaknesses allowing artifact tampering
- an incorrect or malicious MCP endpoint in `mcp.json`
- misleading security documentation

## Out of scope

- vulnerabilities in the Usable service itself — report to security@usable.dev separately
- vulnerabilities in agent clients — report to that client's vendor
- model behavior that ignores instructions absent a concrete exploit path; prompt text is
  documented as not being an enforcement boundary
- missing hardening that the Agent Plugins specification does not define and that we document
  as absent (see [`docs/threat-model.md`](docs/threat-model.md))

## What this package guarantees

- no executable code, dependencies, or install hooks in the release archive
- no credentials anywhere in the repository or artifacts
- exactly two network destinations: `usable.dev` and `auth.flowcore.io`
- releases built from a path allowlist, from protected tags, with published SHA-256 checksums

What it does **not** guarantee is listed plainly in the threat model. Agent Plugins 1.0.0
provides no trust model, and schema conformance is not a safety property.

## Verifying an artifact

```bash
shasum -a 256 -c usable-agent-plugin-<version>.tar.gz.sha256
```

On mismatch: do not extract, and report it. Also confirm that `mcp.json` points at
`https://usable.dev/api/mcp` and nowhere else.

## If a release is compromised

Maintainer procedure:

1. Delete the affected release and tag; keep the audit trail.
2. Publish a security advisory naming affected versions.
3. Rotate any implicated CI credentials.
4. Publish a patched release with a fresh checksum.
5. Update `CHANGELOG.md` and `docs/supported-clients.md`.

User procedure:

1. Uninstall the affected version.
2. Revoke the OAuth grant in your Usable account settings.
3. Clear stored credentials from your client.
4. Install the patched release and verify its checksum.
5. Review your Usable workspace for unexpected writes.

## Reducing your own exposure

- grant `fragments.read` and `workspace.read` only, unless you need the capture skill
- never grant `fragments.delete` — no shipped skill requires it
- pin an exact version and verify checksums
- read `skills/**/*.md` before enabling; it is all plain Markdown
- keep your client's tool-approval prompts enabled
