## What changes

<!-- Describe the behavior change, not just the diff. -->

## Why

<!-- What problem does this solve? Link an issue if there is one. -->

## Type

- [ ] Skill instruction change (**changes agent behavior**)
- [ ] Documentation
- [ ] Client compatibility result
- [ ] Validation or CI
- [ ] Manifest or MCP configuration
- [ ] Other

## Checks

- [ ] `node scripts/validate-package.mjs` passes
- [ ] `node tests/smoke/validator.test.mjs` passes
- [ ] `CHANGELOG.md` updated under `## Unreleased`
- [ ] Conventional Commit message

## Security

- [ ] No credentials, tokens, API keys, or `Authorization` headers added anywhere
- [ ] No private workspace IDs, tenant IDs, internal URLs, or customer data
- [ ] `mcp.json` remains URL-only — no `headers`, no `env`
- [ ] No executable code added to the release payload

## For skill changes only

**Expected behavior change:**
<!-- What will the agent do differently? -->

**How you tested it:**
<!-- Which client, which prompts, what you observed. "Reads better" is not
     sufficient justification for altering agent behavior. -->

**Degraded mode still honest:**
- [ ] With Usable unavailable, the agent still reports that it could not reach Usable rather than fabricating an answer

## For compatibility results only

- Client and exact version:
- OS and version:
- Plugin version:
- Acceptance steps passed (of the five in `docs/supported-clients.md`):
- Verbatim error text for any failure:
