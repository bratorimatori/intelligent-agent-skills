---
name: verify-gate
description: Run the repository's definition of done before handing work back. Use when finishing a change, before opening a PR, or when asked whether something is ready. Triggers on - is this ready, can I ship it, run the checks, done with this.
---

# The verify gate

A change is done when the gate exits 0. Not before. This applies to every
change, whether a human or an agent wrote it: no pull request, no "ready for
review", no hand-back until it is green.

The gate is a script, not a policy. It exits non-zero when the repository is
broken, and that exit code is the whole rule.

## Set it up once

Create `scripts/verify.sh` running three stages in this order, cheapest first:

| Stage | Catches |
|---|---|
| lint | errors, plus any *increase* in warnings over a committed baseline |
| typecheck | type errors, faster and with better messages than the build |
| build | anything that only breaks when the app is actually compiled |

Two details that matter more than they look:

**Baseline the warnings, do not zero them.** Most real repos have hundreds of
pre-existing warnings. Demanding zero means the gate is disabled on day one.
Commit the current count to `scripts/verify-baseline.json` and fail only on an
increase. That number goes down over time and never up.

**Give typecheck its own config that excludes build output.** A stale build
directory makes the typechecker report missing modules for files you just
renamed: a failure with nothing wrong behind it. The build stage covers those
types anyway.

If the build reads a database or an API, point it at an unroutable address for
the gate run. The build then needs no credentials, runs on a fresh clone, and
cannot touch production. Say so out loud, because it also means anything that
only breaks on real data will pass.

## Running it

Run the full gate before handing back. A fast subset (lint plus typecheck) is
for the edit-check loop while you work, and is not a hand-back.

## What you may not do to make it pass

The gate measures the change. Changing the gate to fit the change defeats it.

- No suppression comments at the call site. If a rule is wrong for a whole class
  of file, say so in the config with a comment explaining why.
- No `any` casts or equivalent to silence a type error, and no loosening the
  compiler config.
- No deleting, commenting out, or error-suppressing the failing call site.
- No raising the warning baseline to absorb new warnings.
- No quietly dropping part of the change so the rest gets through. If the scope
  shrinks, that is a thing you say out loud, not a thing you do silently.

## When it fails and you cannot fix it

Hand back the real output. The actual stage output, in full, not a summary and
not "the build failed". Say what you tried and what you think the cause is, and
leave the change where it is.

A red gate reported honestly is a useful result. A green claim over a red gate
is the one outcome this whole thing exists to prevent.

## Say what the gate does not prove

If there is no test suite, the gate proves the code builds, not that it behaves.
Say that plainly in the hand-back and describe what you checked by hand instead.
