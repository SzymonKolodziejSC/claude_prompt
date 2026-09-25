---
description: Verify the whole ticket against its requirements before handing off
---

> Wywołanie: `np. /wrap-up ZFD-12345` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Read", "Glob", "Grep", "Bash"]


# Wrap up

Ticket: `(wartość podana po komendzie w czacie)` (if empty, infer from the current branch).

## 1. Full verification

Run the complete test suite for every affected service, not just the tests you
wrote. Report the result per service.

## 2. Requirements audit

Go through `requirements.md` requirement by requirement. For each one:

- name the code that realises it
- name the test that verifies it
- if no test covers it, say so plainly as a gap

Do not assume the code is right because it compiles and tests pass — check that
the assertions actually test the requirement's substance, not just that nothing
threw. Pay particular attention to boundary cases: is there a test for the value
exactly at a threshold, not only comfortably either side of it?

## 3. Scope audit

The other direction: is there anything in the diff that no requirement asked for?
Name it. Unrequested scope is as much a finding as missing scope.

## 4. Self-review

Read the full diff as if reviewing someone else's pull request. Report findings
in the form: what you see → what it will do in production → what you would change.

Check specifically for: state shared across threads, resources left unclosed,
money handled as floating point, entities leaking across layer boundaries,
queries inside loops, missing input validation, swallowed exceptions, values
hardcoded that belong in configuration, and time read from a static clock rather
than an injected one.

Close honestly: name the categories where you found nothing, and separately the
ones you cannot judge without context you do not have.

## 5. Summary

One paragraph I could paste into the Jira ticket: what changed, in which services,
and anything the reviewer should look at first.

Do not commit or push.
