---
description: Review a branch's changes against the base — findings by severity, with mechanism and impact
---

> Wywołanie: `np. /review feature/ZFD-12345 develop service-a` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Bash", "Read", "Glob", "Grep"]


# Review

Arguments: `(wartość podana po komendzie w czacie)` — branch to review, optionally a base branch (default
`develop`) and a service directory to scope to.

If the branch is not given, review the current branch. If the working tree is
dirty, say so and ask whether to review committed changes only.

## 1. Establish scope

```bash
git fetch origin
git log --oneline <base>..<branch>
git diff <base>...<branch> --stat
```

Note the three-dot form: it diffs against the merge base, so changes that landed
on `<base>` after the branch was cut do not pollute the review.

Report before reading any code: how many commits, how many files, roughly how
many lines changed. If the diff exceeds ~800 lines, say so and ask whether to
review it whole or split by area — a review that large produces worse findings
per line and both of you should know that upfront.

## 2. Understand the intent

Read the commit messages and, if the branch name carries a ticket ID, say so and
ask for the ticket if you cannot fetch it. **You cannot review code well without
knowing what it was supposed to do** — a change can be flawless and still be
wrong.

If no intent is available, state that explicitly and review only for internal
consistency and defects, not for fitness to purpose.

## 3. Read the changed code in context

Do not review the diff alone. For each changed file, read enough of the
surrounding code to know:

- what the class or module is for
- what called it before and what calls it now
- whether the change follows or breaks the conventions already in that file

A diff hides the most expensive category of defect: code that is correct in
isolation and wrong in context.

## 4. Findings

Go file by file, top to bottom, the way a person reads. Do not group thematically
— grouping makes you skip lines.

Every finding has three parts, in this order, and is useless without all three:

1. **What you see** — the specific line or block.
2. **What it does in production** — the mechanism and its consequence. Not "this
   is risky", but "two concurrent requests both pass this check and both
   increment, so the limit is exceeded by one".
3. **What you would change** — or, when there is more than one reasonable fix,
   name the problem and let the author choose.

### Severity

Assign deliberately, not by spreading findings across the scale. **A review with
zero blockers is a normal outcome.** Do not promote the most prominent nit to
fill a category.

- **Blocker** — must not merge. Data loss, security hole, broken contract with
  another service, a requirement not met. If you have more than one blocker, say
  so plainly: more than one usually means the problem is upstream of this PR
  (unclear requirements, no design review) and a conversation beats three red
  flags.
- **Major** — should be fixed before merge. Defect under a realistic but
  non-obvious condition, missing error handling, a decision that will be
  expensive to reverse later.
- **Minor** — worth fixing, does not block. Readability, a missing test for a
  case that is covered elsewhere, a name that misleads.
- **Nit** — optional polish. Style, formatting, preference. Keep these few and
  grouped at the end; a dozen nits bury the findings that matter. If a linter
  could catch it, say that instead of listing it.
- **Question** — you do not understand the intent and need the author to explain
  before you can judge. This is a legitimate finding, not a failure to review.

State severity per finding. Never soften a blocker's language or inflate a nit's.

## 5. Checklist — yours, not the output format

Hold this while reading. Afterwards, verify each category was consciously
considered. Do not structure the report around it.

1. Concurrency and shared state — mutable field on a singleton, check-then-act,
   race between read and write
2. Transactions — boundary too wide, external call inside a transaction,
   self-invocation defeating the proxy, missing transaction on a modifying query
3. Resources — unclosed streams, executors without shutdown, unbounded
   collections, connections held across a slow call
4. Data access — N+1, query in a loop, unbounded fetch, missing index for a new
   query pattern
5. Types and precision — floating point for money, rounding, timezone handling,
   nullable primitives
6. Layering — entity leaking past the adapter, business rule in the controller,
   domain importing the framework
7. API contract — breaking change to an existing consumer, wrong HTTP method or
   status, missing idempotency, verb in the path
8. Validation — unvalidated input, null reaching a dereference, boundary off by
   one
9. Error handling — swallowed exception, catch too broad, missing timeout,
   missing retry, retry without backoff, error surfaced to the caller with
   internal detail
10. Configuration — value hardcoded that belongs in properties, secret in code
11. Testability — static clock instead of injected, no seam for substitution
12. Tests — does the assertion test what the name claims, are the boundaries
    covered, does a new branch have a test at all

## 6. Close honestly

Three sections, all required:

- **Categories where you found nothing** — name them. "Nothing found" without a
  list is indistinguishable from "did not look".
- **What you could not assess** — and why. Missing context, an external service
  you cannot see, a requirement you were not given.
- **What is done well** — if something is. One line. It is not flattery; it tells
  the author which of their instincts to keep.

Never write "otherwise looks good". Write what you looked at and what you could
not.

## 7. Verdict

One of: **approve**, **approve with comments**, **request changes**. Plus one
sentence on what "done" looks like if it is the third.

Do not post anything, do not comment on any platform, do not push. Output the
review here.
