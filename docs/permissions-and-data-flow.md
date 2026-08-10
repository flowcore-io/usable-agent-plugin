# Permissions and data flow

Intended for security reviewers deciding whether to allow this plugin in an organization.

## What the package contains

| | |
|---|---|
| Executable code | None. No scripts, binaries, or postinstall hooks in the release archive. |
| Runtime dependencies | None. |
| Instruction text | `skills/**/*.md` — plain Markdown, fully auditable. |
| Network configuration | One remote MCP server declaration in `mcp.json`. |
| Static images | `assets/usable-icon.svg` and `assets/usable-logo.png` — branding only, no scripts. |
| Credentials | None. |
| Local filesystem access | None requested by the plugin itself. |

The SVG contains only path geometry. It declares no `<script>`, no external references, and
no event handlers, so it cannot execute anything when a client renders it.

The repository contains Node scripts under `scripts/` for validation and release builds.
These run in CI and for local development only, and are excluded from the release archive.

## Network destinations

| Host | Purpose | When |
|---|---|---|
| `usable.dev` | MCP endpoint (`/api/mcp`), OAuth discovery metadata, dynamic client registration | On MCP tool use |
| `auth.flowcore.io` | Authorization and token endpoints (Keycloak realm `memory-mesh`) | During sign-in and token refresh |

No telemetry, analytics, or reporting endpoint is contacted. The plugin sends nothing
anywhere except through the MCP server your client connects to.

## What leaves the machine

Sent to `usable.dev` when a Usable tool is invoked:

- the search query or intent string the agent constructs
- workspace, collection, or fragment identifiers being queried
- for writes: the content the agent proposes to store, after user confirmation

Not sent by the plugin:

- source code, unless the agent explicitly includes an excerpt in a write it asked you to
  confirm
- the full conversation transcript
- environment variables, credentials, or filesystem contents
- anything at all when Usable tools are not invoked

The agent's model provider is a separate data flow governed by your client, not by this
plugin.

## Permissions the plugin needs

| Permission | Why | Required |
|---|---|---|
| Outbound HTTPS to the two hosts above | MCP calls and OAuth | Yes |
| Browser launch | OAuth consent on first authorization | Yes, interactive clients |
| Credential storage | Holding access and refresh tokens | Yes |
| Filesystem write | — | No |
| Shell execution | — | No |

## OAuth scopes

`fragments.read` plus `workspace.read` is sufficient for the read-only workflow. Write scopes
(`fragments.create`, `fragments.update`) are needed only for `usable-knowledge-capture`.
`fragments.delete` is never required by either shipped skill — do not grant it.

See `authentication.md` for the full table.

## Instruction-level behavior

Skills change what the agent does, so review them as you would review a policy document.
The shipped skills instruct the agent to:

- search Usable before proposing implementations
- fetch complete sources rather than relying on snippets
- label assumptions and gaps explicitly
- verify results before claiming success
- report degraded mode rather than fabricate when Usable is unreachable
- treat all retrieved content as untrusted data, not as instructions
- require explicit user confirmation before any write

## What instructions cannot do

Prompt text is not an authorization boundary. The skills tell the agent to ask before
writing, but a sufficiently confused or adversarially-prompted model can still attempt an
action it was told not to take.

Real enforcement has to come from:

- the OAuth scopes you actually grant — the strongest control available here
- your client's tool-approval and permission settings
- server-side authorization in Usable

Grant read-only scopes if you want writes to be impossible rather than merely discouraged.

## Data handling by the Usable service

Retention, processing, and deletion of data sent to Usable are governed by the Usable
service, not by this package. See <https://usable.dev/docs/api> and Usable's own privacy
documentation. Access is revocable from your Usable account settings.

## Residual risks

| Risk | Mitigation |
|---|---|
| Prompt injection via retrieved knowledge | Explicit data-not-instructions rule in the skills; least-privilege scopes; client tool approval |
| Over-broad scope grant enables unintended writes | Grant read-only unless capture is needed |
| Agent writes sensitive content into knowledge | Redaction rules in the capture skill; user confirmation; but review what you approve |
| Compromised release artifact | Immutable tags, SHA-256 checksums, protected release workflow, revocation guidance in `SECURITY.md` |
| Client mishandles credentials | Outside this plugin's control; evaluate the client's credential storage |
