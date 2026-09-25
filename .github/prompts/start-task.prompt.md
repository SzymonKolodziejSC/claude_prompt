---
description: Start work on a Jira ticket — gather context, set up the branch, draft requirements
---

> Wywołanie: `np. /start-task ZFD-12345` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Bash", "Read", "Write", "Glob", "Grep", "mcp__atlassian__*"]


# Start task

Ticket: `(wartość podana po komendzie w czacie)` (if empty, ask which ticket).

Work through the steps below in order. Stop at any point where information is
missing rather than filling the gap with an assumption.

## 1. Read the ticket

Fetch the ticket from Jira. Read the description, acceptance criteria, comments,
attachments and linked issues — comments frequently contain the decisions that
never made it into the description.

## 2. Gather external context

Scan the ticket for links you cannot open yourself:

- **Slack threads** — ask me to paste the relevant conversation. Discussions
  there usually carry the reasoning behind a requirement.
- **Figma / designs** — this is backend work, so judge whether it actually
  matters. If the ticket concerns an API contract that a design depends on, ask
  for it. If it is purely internal, say you are skipping it and why.
- **Confluence or other Jira tickets** — fetch these yourself via the Atlassian
  tools.

Ask for everything you need in **one message**, not one question at a time.

## 3. Identify affected services and code paths

**Fetch before you read anything.** Working trees in this workspace sit on old
feature branches and go stale by dozens of commits. Start with:

```bash
for r in */; do [ -d "$r/.git" ] || continue
  git -C "$r" fetch --quiet origin
  printf "%-24s %-30s behind: %s\n" "$r" "$(git -C "$r" branch --show-current)" \
    "$(git -C "$r" rev-list --count HEAD..origin/develop)"
done
```

Report which repositories were behind and by how much, before quoting any code.

**Then search `origin/develop`, not the working tree.** `git -C <repo> grep <pattern>
origin/develop` and `git show origin/develop:<path>`, never a bare `grep` over the
checked-out files — those show whatever branch the repo happens to sit on. A finding
read from a stale tree is worse than no finding, because it looks like evidence.

If a conclusion did come from a working tree, say so and re-run it against
`origin/develop` before putting it in the report.

Work out which repositories in this workspace the ticket touches. Search for the
relevant classes, endpoints, or events by name rather than assuming from the
ticket title. State what you found, and say explicitly if a service you expected
to be involved turns out not to be.

**For features involving data mapping or transformation:**

1. Search for ALL places where the target type is constructed (e.g., `Deadline.builder()`,
   `new FooDto(`), not just the method name you expect.
2. Trace the flow from the API endpoint down — the same endpoint may have multiple
   code paths (e.g., create vs update, different trigger conditions).
3. List every code path explicitly and state which ones the ticket affects.

A common mistake: finding one place that handles the data and assuming it is the
only one. If you find `ReviewGroupAssembler.buildDeadline()`, also check whether
`ProofAssembler` builds deadlines inline somewhere else.

## 4. Set up the branch

In each affected repository:

```bash
git -C <repo> fetch origin
git -C <repo> checkout -b feature/<TICKET-ID> origin/develop
```

Branch from `origin/develop` directly rather than checking out local `develop` and
pulling — local `develop` may itself be behind, and this leaves it untouched. Print
the commit you branched from so the base is on the record.

Only branch repositories you have **confirmed** are affected. If a repository's
involvement is still unknown, say so and leave it alone rather than creating a
branch on a guess.

Stop before doing this if the working tree is dirty — show me `git status` and
ask how to proceed. Never stash or discard anything on your own.

## 5. Decide how much process this needs

Apply the criteria in the workspace `CLAUDE.md`. State the route you are taking
and the reason. If it is genuinely borderline, ask instead of deciding.

**Direct implementation** → stop here. Report the plan in a couple of sentences
and wait for my go-ahead.

**Full cycle** → continue to step 6.

## 6. Draft requirements

Create `specs/<TICKET-ID>/requirements.md` at the workspace root, containing:

1. **Business context** — two or three sentences on what breaks without this.
2. **Acceptance criteria** in EARS form (`WHEN <trigger> THEN <system response>`),
   numbered `REQ-001`, `REQ-002`, … Each one must be verifiable by a test.
3. **Assumptions** — every gap you filled yourself, with the reason. If the
   ticket gives no numbers for scale, latency, volume or thresholds, propose
   concrete values and mark them as assumptions rather than leaving them vague.
4. **Constraints** — what is deliberately out of scope.
5. **Open questions** — what remains unresolved and who can answer it.

Then create `specs/<TICKET-ID>/.status` containing `requirements:draft`.

## 7. Report back

Summarise: which services are affected, which branch you created, which route
you chose and why, and every ambiguity or conflict you found in the ticket.

**Do not write any production code.** Wait for me to review the requirements and
advance `.status`.
