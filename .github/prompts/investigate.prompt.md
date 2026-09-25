---
description: Investigate a bug — hypothesis log that survives context compaction
---

> Wywołanie: `np. /investigate ZFD-12345` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "Agent", "mcp__atlassian__*"]


# Investigate

Ticket: `(wartość podana po komendzie w czacie)` (if empty, ask which ticket).

A bug is not a feature. There is no `requirements.md` to write, because what the
system *should* do is already known — what is unknown is why it doesn't. The
output of this command is not a plan, it is a **hypothesis log**: what you
suspected, what you tested, what you learned.

## Why this file exists

Context compaction is a separate model call that does **not** read `CLAUDE.md`
and does not read this command. Anything not written to disk is gone when the
window compacts. On a bug that spans sessions, that means retesting the same
hypothesis twice — which happens more often than it sounds.

So: **write to the file as you go, not at the end.** After every hypothesis is
resolved. Not once at the finish.

## 1. Set up the log

Create `specs/<TICKET-ID>/investigation.md` with these sections. Fill what you
can now, leave the rest as headings.

```markdown
# <TICKET-ID> — <one-line symptom>

## Signature
- Symptom, in observable terms (timings, error message, status code)
- Exact error text and stack trace, verbatim — not paraphrased
- Affected services and files
- Reproduction: steps, or "not reproduced yet"
- Environment where it occurs, and where it doesn't

## Known good vs known bad
What works, what doesn't, and the narrowest difference between them.

## Hypotheses
(append-only — never delete a rejected hypothesis)

## Root cause
(empty until proven)

## Fix
(empty until root cause is proven)

## Open threads
Things noticed but not chased. Note them or they are lost.
```

## 2. Establish the signature before theorising

Fill in `Signature` first. Resist proposing causes until it is complete —
a hypothesis formed before the symptom is pinned down anchors the whole
investigation on a guess.

Specifically nail down:

- **What exactly is slow, failing, or wrong** — in numbers, not adjectives.
  "Takes over a minute" is a start; "p50 is 62s, p99 is 71s, over 40 executions"
  is a signature.
- **The narrowest known-good comparison.** If the same operation is fast via one
  path and slow via another, that difference *is* the investigation. Say what
  differs between the two paths, exhaustively.
- **First and last known-good time**, if it used to work. A deploy, config change
  or traffic shift in that window is a prime suspect.
- **Whether it reproduces**, and under what conditions. An intermittent bug and a
  deterministic one need different approaches.

If you cannot get this data yourself — production logs, metrics, a staging
environment — say exactly what you need and stop. Do not substitute plausible
reasoning for missing evidence.

## 3. Hypotheses, one at a time

Each hypothesis gets an entry, appended to the log the moment it is resolved:

```markdown
### H1: <specific, falsifiable claim>
**Predicts:** if true, we would observe X
**Tested by:** the exact command, query, log filter or code path examined
**Result:** what was actually observed
**Verdict:** CONFIRMED | REJECTED | INCONCLUSIVE
**Learned:** what this tells us regardless of verdict
```

Rules:

- A hypothesis must be **falsifiable**. "SQS is slow" is not — there is no
  observation that refutes it. "Each SQS hop adds ~1s of polling latency, so four
  hops account for 4s of the 60s" is, and it is also immediately checkable.
- State the prediction **before** testing. Otherwise every result confirms
  whatever you already believed.
- **INCONCLUSIVE is a real verdict.** Recording "could not test, no access to
  staging" is worth more than a guess dressed as a finding.
- **Never delete a rejected hypothesis.** The rejections are why the log exists —
  they are what stops the next session repeating them.
- Order by cost: cheap and likely first. Reading a config file beats reproducing
  on staging beats adding instrumentation and waiting for traffic.

Use a subagent for wide searches across services — it keeps raw file dumps out of
the main context. Bring back the conclusion, not the transcript.

## 4. Narrow before you go deep

Before reading implementation, cut the search space:

- Which hop is slow? Instrument or time the boundaries before theorising about
  what happens inside any of them.
- Does it happen for all inputs or specific ones? Compare a fast case to a slow
  case with the same code path.
- Is it load-dependent? A queue that drains fine at 10/s and collapses at 100/s
  points at contention, not logic.

Measuring where the time goes beats reading code that might contain the answer.
State the measurement, then read only the code it implicates.

## 5. Root cause, not a plausible cause

A root cause is proven when you can **explain the full mechanism** and, ideally,
**turn the symptom on and off** on demand.

Before writing anything into `## Root cause`, answer:

- Does this explain the *magnitude*? A 3s explanation for a 60s symptom is not
  the root cause — it is a contributing factor. Say which.
- Does it explain why the fast path is fast?
- Does it explain the timing — why now, why this tenant, why this flow?
- What would I expect to see if I were wrong?

If any answer is unclear, it stays a hypothesis. Write it as such.

## 6. Fix — only after root cause

Once root cause is proven, `## Fix` gets:

- What to change and why that addresses the mechanism, not the symptom
- Blast radius: which services, what could regress
- **A regression test.** A bug fixed without one comes back at the next refactor
  and nobody knows why the code was written that way. The test is also the best
  documentation of the bug that will exist.
- Whether this warrants the full `/design` cycle — if the fix changes a contract,
  touches multiple services, or has more than one reasonable shape, it does.
  Say so and stop rather than implementing.

## 7. Every session

Start by reading `investigation.md` — including rejected hypotheses. End by
updating it, even if the session produced nothing but rejections. Especially
then.

When context compacts mid-session, the file is what remains. Treat it as the
source of truth, not your memory of the conversation.
