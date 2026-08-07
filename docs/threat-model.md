# Threat model

## Scope

Covers the plugin package, its distribution, and its effect on agent behavior. Does not cover
the security of the Usable service itself, of the agent client, or of the model provider.

## The most important thing on this page

**Agent Plugins 1.0.0 provides no trust model.** It specifies no permissions framework, no
sandbox, no signature standard, no portable secrets mechanism, and no dependency or lifecycle
audit requirements.

Schema conformance therefore says nothing about safety. A package can be perfectly valid and
entirely malicious. Evaluate this plugin — and every other one — on its contents, not on its
validity.

## Assets

| Asset | Why it matters |
|---|---|
| The user's Usable OAuth token | Grants API access at the scopes granted |
| Workspace knowledge | May contain proprietary decisions and internal details |
| The user's repository and local environment | Reachable by the agent, though not by this plugin |
| Release artifact integrity | A compromised artifact reaches every installer |
| Agent behavior | Instructions shape actions the agent takes on the user's behalf |

## Trust boundaries

```
User ──trusts──▶ Agent client ──loads──▶ This plugin (instructions only, no code)
                      │
                      ├──authenticates──▶ auth.flowcore.io   (OAuth 2.1 + PKCE)
                      └──calls──────────▶ usable.dev/api/mcp (bearer token)
                                                 │
                                                 ▼
                                    Returns knowledge = UNTRUSTED DATA
```

The critical boundary is the last one. Content returned by the MCP server crosses into the
model's context, and must never be treated as instructions.

## Threats

### T1 — Prompt injection via retrieved knowledge

**Risk: high.** Anyone who can write to a workspace, or to a repository, issue, or log the
agent reads, can plant text shaped like instructions.

Mitigations: an explicit data-not-instructions rule in the workflow skill and in
`references/security-boundaries.md`; a rule that self-attested exemption flags confer no
authority; least-privilege scopes; the client's tool-approval prompts.

Residual: instructions are not an enforcement boundary. A confused model may still comply.
Read-only scopes are what actually bound the damage.

### T2 — Credential leakage into the public package

**Risk: critical if it happened; low likelihood.**

Mitigations: `mcp.json` is URL-only by design and documented as such; CI fails on credential
patterns in `mcp.json`, `plugin.json`, and skills; GitHub secret scanning with push
protection is enabled; the release archive is built from a path allowlist.

Residual: `CODEOWNERS` assigns reviewers for MCP configuration but approval is not enforced —
required approvals are zero while the maintainer team has one member. The automated checks
above are the real control here; human review is currently advisory. See `CONTRIBUTING.md`.

### T3 — Supply chain compromise of a release

**Risk: high impact, low likelihood.**

Mitigations: releases publish only from tags on this repository; the manifest version must
equal the tag; Actions are pinned to commit SHAs; SHA-256 checksums are published; no
publishing from fork or pull-request contexts; `main` forbids force pushes and deletions;
revocation procedure in `SECURITY.md`.

Residual: no signed attestations yet — [open decision #7](../README.md#open-decisions). Tags
themselves are not protected, so a maintainer could move one; releases are immutable once
published, and checksums are the artifact-level guarantee.

### T4 — Over-broad scope grant

**Risk: medium.** A user who grants write or delete scopes for convenience enables actions
the skills only discourage.

Mitigations: per-scope documentation with an explicit read-only recommendation;
`fragments.delete` documented as never required.

### T5 — Sensitive content written into knowledge

**Risk: medium.** The capture skill exists to write things down, and could write down
something it should not.

Mitigations: mandatory redaction rules; verification precondition; user confirmation on every
write; no blanket approval carried across a conversation.

Residual: the user has to actually read what they approve.

### T6 — Malicious fork or typosquat

**Risk: medium.** A lookalike repository could ship a plugin that exfiltrates context to a
different endpoint.

Mitigations: the canonical repository is `flowcore-io/usable-agent-plugin`; the only endpoints
this plugin contacts are `usable.dev` and `auth.flowcore.io`, documented in
`permissions-and-data-flow.md`. Verify the `url` in `mcp.json` before installing anything
claiming to be this plugin.

### T7 — Stale or wrong knowledge treated as authoritative

**Risk: medium.** Confidently retrieved bad guidance is more dangerous than no guidance.

Mitigations: freshness heuristics and staleness signals in `references/knowledge-lifecycle.md`;
a rule that code beats stored knowledge on questions of behavior; required reporting of
conflicts rather than silent resolution.

### T8 — False success claims

**Risk: medium.** An agent that says "done" without verifying transfers unearned confidence
to the user.

Mitigations: the verified/reported/assumed distinction in
`references/evidence-and-verification.md`; a requirement to state partial verification
explicitly; planned golden tests asserting degraded-mode honesty.

## Non-threats

- **Code execution from the package.** The release archive contains only Markdown and JSON.
- **Local file exfiltration by the plugin.** It has no filesystem access. The agent does, via
  the client, which is a client-level concern.
- **Telemetry.** None.

## Reporting

Use the private path in `SECURITY.md`. Do not open a public issue for a vulnerability.
