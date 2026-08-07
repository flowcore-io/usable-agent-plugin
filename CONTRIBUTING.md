# Contributing

## Before you start

Read [`docs/threat-model.md`](docs/threat-model.md) and the
[trust boundary section](README.md#trust-boundary) of the README. This package is public,
contains no credentials, and ships instructions that change how agents behave. Both facts
constrain what can be merged.

## What belongs here

Welcome:

- clarity fixes to skill instructions
- client compatibility results, including failures
- installation guides for a newly validated client
- validation and CI improvements
- documentation corrections

Not accepted:

- credentials, tokens, API keys, or static `Authorization` headers anywhere
- private workspace IDs, internal URLs, customer names, or real user data
- executable code in the release payload — the archive stays Markdown and JSON
- forks or redefinitions of the Agent Plugins specification; upstream contributions go to
  [`agentplugins/agent-plugins-spec`](https://github.com/agentplugins/agent-plugins-spec)
- large monolithic skills; prefer small composable ones with references
- claims of client support without acceptance evidence

## Setup

```bash
git clone https://github.com/flowcore-io/usable-agent-plugin.git
cd usable-agent-plugin
node scripts/validate-package.mjs
```

Node 20+. No dependencies to install.

## Writing skills

Skill instructions are the product. They are read by a model under time pressure, so they must
be unambiguous.

Requirements:

- `SKILL.md` in a directory that is an immediate child of `skills/`
- frontmatter with `name` (matching the directory) and `description`
- the `description` must state *what it does* **and** *when to use it* — this is how clients
  decide whether to load it
- keep `SKILL.md` focused; move depth into `references/`
- use concrete examples, and show the bad version alongside the good one
- state what the agent must **not** do, not only what it should
- never hard-code workspace IDs, tenant identifiers, or internal URLs

When you change wording, explain in the PR what behavior you expect to change and how you
tested it. "Reads better" is not sufficient justification for altering agent behavior.

## Changing `mcp.json`

Highest-scrutiny file in the repository. It must stay URL-only.

- no `headers`, no `env`, no token, no client secret
- transport stays `streamable-http` for the hosted server
- a URL change requires re-verifying the full OAuth discovery chain and updating
  [`docs/authentication.md`](docs/authentication.md)

## Reporting compatibility results

Include the client name and exact version, OS and version, plugin version, which of the five
acceptance steps in [`docs/supported-clients.md`](docs/supported-clients.md) passed, and
verbatim error text for failures. Negative results are valuable — a documented ❌ is more
useful than an optimistic ⏳.

## Pull requests

1. Branch from `main`.
2. Run `node scripts/validate-package.mjs` and make it pass.
3. Add a `CHANGELOG.md` entry under `## Unreleased`.
4. Use [Conventional Commits](https://www.conventionalcommits.org/) — `feat:`, `fix:`,
   `docs:`, `chore:`, `ci:`.
5. Describe the behavior change, not just the diff.

CI blocks: invalid manifests, invalid skill frontmatter, unsafe paths or symlinks, credential
patterns, broken links, and release-archive drift.

Skill, `mcp.json`, release workflow, and security documentation changes require CODEOWNERS
review.

## Versioning

SemVer, with the manifest version tracking the tag.

- **patch** — typos, clarifications that do not change behavior
- **minor** — new skills, or behavior changes that are additive
- **major** — removing a skill, renaming one, or changing the MCP endpoint

Skill wording that meaningfully changes agent behavior is at least a minor bump, even though
no code changed.

## Releasing

Maintainers only. The release workflow enforces each of these, so a step skipped here shows up
as a failed release rather than a bad artifact.

1. Rename the `## Unreleased` section in `CHANGELOG.md` to the version being released, e.g.
   `## 0.1.0`. The workflow fails if no section matches the tag.
2. Set the matching `version` in `plugin.json`. The workflow fails on any tag/manifest
   mismatch.
3. Open a PR with both changes and get it reviewed.
4. After merge, tag and push:
   ```bash
   git tag v0.1.0
   git push origin v0.1.0
   ```
5. The workflow re-runs validation and the self-tests, builds the allowlisted archive,
   verifies the checksum, asserts the archive contains no executable code, and publishes to
   GitHub Releases. `0.x` versions are marked prerelease automatically.

Releases publish only from tags on `flowcore-io/usable-agent-plugin`, never from a fork or a
pull-request context. At least two maintainers must be able to publish and revoke releases.

## Security

Never open a public issue for a vulnerability. Follow [`SECURITY.md`](SECURITY.md).
