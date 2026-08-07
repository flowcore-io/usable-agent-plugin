# Knowledge lifecycle

Stored knowledge is a snapshot of what was true when someone wrote it down. Some of it stays
true for years; some of it is wrong within a week. Judging which is which is part of the
retrieval job.

## How different kinds of knowledge age

Fast-decaying — verify against the code before relying on it:

- exact file paths, function names, line numbers
- version numbers, dependency pins, image tags
- environment variable names and endpoint URLs
- "current" architecture descriptions

Slow-decaying — usually still valid:

- why a decision was made, and what was rejected
- incident root causes and the reasoning behind the fix
- domain rules and business constraints
- security boundaries and threat models

The distinction is roughly: *facts about the current state* decay, *reasoning about
tradeoffs* does not.

## Freshness heuristics

- Under 90 days: treat as current, but still confirm specifics against the code.
- 90 days to a year: treat the reasoning as valid and re-check every concrete detail.
- Over a year: treat as historical context. Say so if you rely on it.

Dates alone are not enough. A three-year-old item about a module nobody has touched may be
more accurate than a three-week-old item about a module being actively rewritten.

## Signals an item is stale

- it references files, functions, or services that no longer exist
- it describes a library version well behind what is installed
- a newer item covers the same ground and disagrees
- it is explicitly marked superseded, abandoned, or deprecated
- its branch or release tag has long since merged or shipped

When you spot these, report the staleness. That is useful output in itself, and it is often
the reason the user's expectation and the code have drifted apart.

## Supersession

Items sometimes explicitly replace one another. Follow the chain forward to the newest one
before acting, and prefer whichever item is marked as current. If an item says another item
supersedes it, do not act on the superseded one just because it ranked higher in search.

## When knowledge and code disagree

Code is the ground truth about behavior. Stored knowledge is the ground truth about
intent — why the code is the way it is.

If they disagree about behavior, believe the code. If they disagree about intent, the stored
decision usually still explains a constraint you would otherwise break by "fixing" the code.
Report the mismatch either way.
