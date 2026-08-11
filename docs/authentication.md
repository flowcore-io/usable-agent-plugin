# Authentication

The plugin ships **no credentials**. `mcp.json` contains a URL and a transport type, and
nothing else. Your agent client obtains and stores tokens itself using OAuth 2.1.

## Why it works this way

Agent Plugins 1.0.0 defines no portable secrets mechanism. Any credential committed to a
public package would be shared by every installer, could not be rotated per user, and would
grant the same access to everyone. So the plugin declares only the endpoint, and delegates
authentication entirely to the client.

The practical consequence: **never add a bearer token, API key, client secret, static
`Authorization` header, or credential-bearing environment variable to `mcp.json`.** A
pull request that does so will be rejected, and CI is configured to fail on it.

## The discovery chain

A compliant client performs these steps on first connection. All values below were verified
against production.

**1. Connect unauthenticated, receive a challenge**

```
POST https://usable.dev/api/mcp
→ 401
   www-authenticate: Bearer resource_metadata="https://usable.dev/.well-known/oauth-protected-resource"
```

**2. Fetch protected resource metadata (RFC 9728)**

```
GET https://usable.dev/.well-known/oauth-protected-resource
```

```json
{
  "resource": "https://usable.dev",
  "authorization_servers": ["https://usable.dev"],
  "as_uri": "https://usable.dev/.well-known/oauth-authorization-server",
  "resource_documentation": "https://usable.dev/docs/api",
  "scopes_supported": ["openid", "profile", "email", "offline_access", "fragments.read", "..."]
}
```

**3. Fetch authorization server metadata (RFC 8414)**

```
GET https://usable.dev/.well-known/oauth-authorization-server
```

| Field | Value |
|---|---|
| `issuer` | `https://auth.flowcore.io/realms/memory-mesh` |
| `authorization_endpoint` | `https://auth.flowcore.io/realms/memory-mesh/protocol/openid-connect/auth` |
| `token_endpoint` | `https://auth.flowcore.io/realms/memory-mesh/protocol/openid-connect/token` |
| `registration_endpoint` | `https://usable.dev/api/oauth/register` |
| `code_challenge_methods_supported` | `S256`, `plain` |
| `grant_types_supported` | `authorization_code`, `refresh_token`, `client_credentials`, `device_code` |
| `token_endpoint_auth_methods_supported` | `client_secret_basic`, `client_secret_post`, `none` |

Dynamic client registration is available, so a client can obtain its own ID. Public clients may
register with `none` and use PKCE.

### The default client ID

Usable's default MCP configuration uses the public client ID **`mcp_oauth_client`**. Clients
that do not implement dynamic registration, or that prefer a fixed ID, should use it:

```bash
# Claude Code
claude mcp add --transport http usable https://usable.dev/api/mcp --client-id mcp_oauth_client

# Codex
codex mcp add usable --url https://usable.dev/api/mcp --oauth-client-id mcp_oauth_client
```

An OAuth `client_id` is a public identifier, not a secret — it is safe in a shared config and
in this repository. This package declares it for Claude Code in `.mcp.json` under
`oauth.clientId`, which is the field Claude Code reads.

A client **secret** is a different thing entirely and must never be packaged. CI enforces the
distinction: the validator permits only `clientId`, `callbackPort`, and `scopes` inside an
`oauth` block and fails on anything else, so a secret cannot slip in by being unrecognised.

The Agent Plugins `mcp.json` stays URL-only. Its schema does not define an `oauth` field, and
inventing one risks rejection by a strict client.

**4. Authorize with PKCE**

Authorization code flow, `code_challenge_method=S256`, including the MCP resource indicator
for `https://usable.dev`. A browser window opens for sign-in and consent.

**5. Store, refresh, retry**

The client stores the access and refresh tokens in its own credential store and retries the
MCP connection with a bearer token. `offline_access` is available for refresh.

## Scopes

Request the narrowest set the task needs.

| Scope | Grants |
|---|---|
| `fragments.read` | Search and read knowledge. **Sufficient for `usable-knowledge-workflow`.** |
| `fragments.create` | Create knowledge. Needed by `usable-knowledge-capture`. |
| `fragments.update` | Update existing knowledge. Needed by `usable-knowledge-capture`. |
| `fragments.delete` | Delete knowledge. Not required by either shipped skill. |
| `workspace.read` | List workspaces and fragment types. |
| `offline_access` | Refresh tokens without re-authenticating. |

A read-only install needs `fragments.read` and `workspace.read`. Grant write scopes only if
you intend to use the capture skill.

## Headless and CI environments

The interactive flow needs a browser. For non-interactive contexts the authorization server
also advertises `client_credentials` and `device_code`.

Neither is wired into this plugin, and neither is covered by the compatibility matrix yet.
Do **not** work around a headless client by committing a shared static credential into
`mcp.json` — that defeats per-user authorization and rotation. Track this in
[open decision #5](../README.md#open-decisions).

## A user-level server of the same name shadows this one

If your client already has an MCP server named `usable` at user or project scope, that entry
takes precedence and the plugin's declaration is silently ignored. Confirmed on both Claude
Code and Codex.

Two consequences worth knowing:

- You cannot tell from `claude mcp list` or `codex mcp list` whether a `usable` server came
  from this plugin or from your own config. On Claude Code a plugin-provided server appears as
  `plugin:usable:usable`; a bare `usable` is yours, not the plugin's.
- If your existing entry authenticates with a static bearer token rather than OAuth, you keep
  that behaviour — and none of this package's credential hygiene applies to it. A long-lived
  token in a client config is worth replacing with the OAuth flow.

To let the plugin's declaration take effect, remove or rename the user-level entry:

```bash
claude mcp remove usable      # or: codex mcp remove usable
```

## Revoking access

Revoke the client's grant in your Usable account settings, then remove the stored
credentials from the client (see the client's own credential management). Revocation is
immediate for new requests; already-issued access tokens remain valid until they expire.

## Both metadata URLs work

Either discovery strategy reaches the same document. Reading the `resource_metadata`
parameter out of the `WWW-Authenticate` challenge gives
`https://usable.dev/.well-known/oauth-protected-resource`; constructing the URL by the
RFC 9728 §3.1 rule — inserting `/.well-known/oauth-protected-resource` between host and
resource path — gives `https://usable.dev/.well-known/oauth-protected-resource/api/mcp`.
Both return the same JSON.

The path-suffixed URL previously served the site's HTML, which broke discovery for clients
that prefer the RFC construction over the challenge header. Fixed server-side on 2026-08-07.
