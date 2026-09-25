---
description: Produce design.md from approved requirements
---

> Wywołanie: `np. /design ZFD-12345` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Read", "Write", "Glob", "Grep", "Bash"]


# Design

Ticket: `(wartość podana po komendzie w czacie)` (if empty, infer from the current branch).

## Gate

Read `specs/<TICKET-ID>/.status`. Proceed only if it reads `requirements:approved`.
If it does not, say so and stop.

## Before designing

Read `specs/<TICKET-ID>/requirements.md` in full, plus the `CLAUDE.md` of every
affected service. Existing architectural rules in those files are not up for
renegotiation here — if a requirement cannot be met without breaking one, that is
a conflict: stop and report it.

Read the existing code in the areas you will touch. Design against what is there,
not against what you would have built from scratch.

## Produce `specs/<TICKET-ID>/design.md`

Include:

1. **Files to create or change**, per repository. This list is binding —
   implementation may not create a file that is not on it.
2. **Contracts** — request/response shapes, event schemas, method signatures for
   anything crossing a service or module boundary. Include error responses, not
   just the happy path.
3. **Data model changes** — tables, columns, indexes, migrations. Name the
   migration tool and file convention used by that service.
4. **Concurrency and transactions**, where relevant — where the boundary sits,
   what guarantees it provides, and what mechanism enforces them. Be explicit
   about whether a guarantee rests on a database constraint, a lock, or
   application logic.
5. **REQ → design element mapping** — every requirement traced to the thing that
   realises it. A requirement with no entry is a gap.
6. **Test names to write**, no code. Name the boundary cases explicitly.
7. **Decisions not dictated by the requirements** — every judgment call you made
   to fill a gap, with the alternatives you rejected and why.

## Finish

Set `.status` to `design:draft`. List every decision from section 7 in your
reply — I want to see them without opening the file.

**Do not write any production code.**
