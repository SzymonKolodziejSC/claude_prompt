---
description: Break an approved design into verifiable implementation steps
---

> Wywołanie: `np. /tasks ZFD-12345` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Read", "Write", "Glob", "Grep", "Bash"]


# Tasks

Ticket: `(wartość podana po komendzie w czacie)` (if empty, infer from the current branch).

## Gate

Read `specs/<TICKET-ID>/.status`. Proceed only if it reads `design:approved`.
If it does not, say so and stop.

## Produce `specs/<TICKET-ID>/tasks.md`

Break the design into steps ordered so that each one only depends on what already
exists by the time it starts. Every step gets:

- a checkbox `- [ ]` and an ID (`T001`, `T002`, …)
- the files it creates or changes, drawn only from `design.md`'s file list
- which requirement(s) it realises
- **the exact command that verifies it**, ending in an exit code

A step whose completion cannot be verified by a command is not a step — either
give it a real verification or fold it into one that has one.

Do not split artificially. If the work is genuinely one step, say so and stop
rather than manufacturing a list.

Finish with a traceability check: confirm every REQ in `requirements.md` appears
in at least one task, and name any that do not.

## Finish

Set `.status` to `tasks:draft`.
