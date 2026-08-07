---
name: usable-knowledge-capture
description: Capture a verified outcome back into Usable so the next person or agent does not rediscover it. Use after fixing a non-obvious bug, making an architectural decision, finishing an incident investigation, or discovering a constraint that cost real time. Requires the outcome to be verified first, prefers updating an existing item over creating a duplicate, and always asks before writing.
---

# Usable knowledge capture

Knowledge that stays in a chat transcript is lost. This skill writes verified outcomes into
Usable so the next person does not pay the same debugging cost twice.

## Preconditions

Do not write anything until all four hold:

1. **The outcome is verified.** Tests passed, the behavior was observed, the fix is confirmed
   in reality — not predicted. An unverified guess written into durable knowledge is worse
   than no knowledge at all, because it will be trusted later.
2. **Write tooling is available and authorized.** If the Usable MCP server is missing or
   unauthenticated, say so and offer the content for the user to save manually.
3. **The user has confirmed.** Show what you intend to write and where, and wait for a yes.
4. **It is worth keeping.** See the bar below.

## What is worth capturing

Capture when the answer was expensive to find and would be expensive to find again:

- a bug whose root cause was not obvious from the symptom
- an architectural decision, including the options rejected and why
- a constraint discovered the hard way — an API limit, a schema quirk, an ordering
  requirement
- an incident: what broke, why, how it was diagnosed, what fixed it
- a repeatable procedure that took several attempts to get right
- a hypothesis that measurement killed; negative results save the next person a day

Do not capture:

- routine changes obvious from the diff
- restatements of public documentation
- anything unverified
- transient state ("the build is currently red")
- content containing secrets, personal data, or customer data

## Update before you create

Search for existing coverage first. Duplicates fragment the corpus and cause exactly the
conflicting-guidance problem the retrieval workflow has to spend effort resolving.

- Same topic, still accurate, new detail → update it.
- Same topic, now wrong → update it and state what changed and when.
- Genuinely new topic → create, and reference related items.

Prefer one good item that gets maintained over three partial items that rot.

## What to include

- **Title** — specific enough to recognize in a search result. "Badge print claim returns
  500 because FOR UPDATE cannot be used with LEFT JOIN" beats "Fixed printing bug".
- **Context** — the symptom as it was first observed, and where.
- **Root cause** — the actual mechanism, not just the change made.
- **Resolution** — what fixed it, and why that works.
- **Verification** — how you know. Name the tests, commands, or observations.
- **Metadata** — repository, branch, version, or release, when known.
- **Residual risk** — what remains untested or uncertain.
- **Tags** — repository and domain tags so the item is findable later.

Write for a reader who has none of the current conversation's context.

## Before writing: redact

Strip access tokens, API keys, passwords, connection strings with embedded credentials,
internal-only URLs, personal data, customer data, and raw logs containing any of those.
Describe the shape of a value instead of reproducing it.

## Confirmation

Always show the plan and wait:

> I'd like to record this as a solution titled "<title>" in <workspace>, tagged
> `<tags>`. It covers the root cause, the fix, and the verification steps. Save it?

If write tooling is unavailable, say so and hand the content over instead:

> I can't write to Usable — no authorized write tool is configured. Here is the content if
> you'd like to save it yourself: ...

Never write silently, and never treat an earlier "yes" in the conversation as blanket
approval for later writes.
