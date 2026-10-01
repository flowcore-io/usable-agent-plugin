# OpenAI plugin submission

The package follows the portable Agent Plugins format. OpenAI discovers root
`plugin.json`, `mcp.json`, and `skills/`; a `.codex-plugin` overlay is unnecessary.

Requirements: [OpenAI submission guide](https://developers.openai.com/plugins/deploy/submission).

## Build the upload

```bash
node scripts/validate-package.mjs
node tests/smoke/validator.test.mjs
node scripts/build-release.mjs
```

Upload `dist/usable-agent-plugin-0.2.0.zip`, with `plugin.json` at the ZIP root.
The build also preserves the existing tar.gz artifact for tagged-release installs.
Both artifacts have SHA-256 sidecars. Version `0.2.0` is the current source version;
bump the portable and Claude manifests together before publishing a new release.

## Prepare the review account and sample data

Review cases in `plugin.json` are prepared scenarios, **not completed live tests**.
Run all five positive and three negative cases using a dedicated reviewer account
before submission. Keep its workspace and sample data available for future reviews.

Seed the review workspace with:

- A documented API key rotation decision.
- A deployment checklist.
- Authentication constraints, with explicit evidence and known gaps.
- An older draft retry policy and a newer verified policy that supersedes it.
- A sample security note containing an inert prompt-injection example requesting
  OAuth credentials. Include no real credentials.

Ensure there is no policy for `Project Missing Example`. The account must not have
access to the private workspace used in the unauthorized-access negative case.
Record actual tool names and behavior from the connected server; update the cases
if the live interface differs. The default OAuth configuration is read-only;
the negative capture case must perform no write.

Enter reviewer credentials, login URL, workspace details, and sign-in instructions
only in the secure dashboard form. The account must work without MFA approvals,
email/SMS codes, magic links, or private-network access. Never put credentials or
`reviewer_instructions` in the ZIP.

## Remaining submission gates

1. Run and record the eight review cases with that account, including fresh OAuth
   in a clean client where a user-level `usable` server cannot shadow the plugin.
2. Record a walkthrough demonstrating the cases. Add its reviewer-accessible HTTPS
   URL as `extensions.com.openai.review.demo_recording_url`, or supply it in the
   dashboard. No placeholder recording URL ships in this package.
3. Confirm publisher identity, organization/project permissions, and the public
   listing URLs in the OpenAI dashboard.
4. Upload the ZIP. Connect the declared MCP server, authenticate, and host the
   dashboard's exact domain challenge at the supplied `.well-known` URL. Do not
   replace an existing plugin's token at the same challenge URL.
5. Resolve metadata, skill, and MCP scan findings. Check the imported cases and
   release notes, enter private review access, and complete policy attestations.
6. Submit for review. Publish the approved version when ready.

For a submission containing all review materials in its manifest, run:

```bash
node scripts/validate-package.mjs --submission
```

This intentionally fails until a real recording URL is supplied. If the recording
is managed in the dashboard, verify that field there instead. A passing local
check validates metadata shape; it does not prove account access, URL availability,
tool behavior, completed scans, review approval, or publication.

## Updates

Metadata, assets, and skills require a new version and complete ZIP uploaded to the
existing plugin. Hosted MCP tool changes are scanned separately; keep the server
compatible with currently approved schemas until updates are live. Changing an
existing MCP URL requires OpenAI support under the current submission flow.
