---
description: Implement the approved task list, one step at a time
---

> Wywołanie: `np. /implement ZFD-12345` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]


# Implement

Ticket: `(wartość podana po komendzie w czacie)` (if empty, infer from the current branch).

## Gate

Read `specs/<TICKET-ID>/.status`. Proceed only if it reads `tasks:approved`.
If it does not, say so and stop.

## Rules

Work through `tasks.md` in order. After each task, report:

```
Claim:   what the task was supposed to achieve
Command: the exact command run
Exit:    exit code
Verdict: PASS | FAIL
```

**FAIL means stop.** Show the diagnosis and wait. Do not iterate silently until
something passes — a failing verification is information, and quietly fixing it
discards that information.

Mark a task `[x]` only after seeing its command pass.

Never create a file that is not in `design.md`'s list. If you need one, that is a
divergence: stop, describe it, wait for approval, update `design.md` first.

Pause after each logical group of tasks rather than after every single one — I
want to see progress in meaningful chunks, not a message every two minutes.

## Do not

- commit, push, or open a pull request unless asked
- change `.status`
- add dependencies without saying so first
