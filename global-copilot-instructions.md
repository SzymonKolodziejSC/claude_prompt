# Workspace

Multi-repository workspace. Each subdirectory is a separate service with its own
git repository and its own `CLAUDE.md` where one exists — read it before touching
that service.

## Working documents

Planning documents live in `specs/<TICKET-ID>/` at the workspace root, **never**
inside a service repository. They are personal working notes, not team artifacts,
and must never be committed to any service repo.

## Deciding how much process a task needs

Not every ticket needs a full specification. Assess before starting, and say which
route you are taking and why.

**Full cycle** (`requirements` → `design` → `tasks` → implementation) when any of
these hold:

- changes a contract between services — API, event, message schema
- touches more than one repository
- requires a database migration
- involves non-obvious concurrency or transaction boundaries
- requirements are ambiguous, incomplete, or internally contradictory
- there is more than one reasonable way to build it

**Direct implementation** when none of them hold: implement, then write a short
`specs/<TICKET-ID>/notes.md` recording what changed and why.

Story points are not the criterion — a two-point change to a shared API contract
carries more risk than a five-point refactor inside one service.

**When unsure which route applies, ask. Do not guess.**

## The status gate

`specs/<TICKET-ID>/.status` holds one line, and it is the only signal that
authorises the next phase:

```
requirements:draft → requirements:approved → design:draft
→ design:approved → tasks:draft → tasks:approved
```

Implementation is authorised only at `tasks:approved`. A file existing on disk is
not approval. **Never advance `.status` yourself** — only the human does that.

Direct-implementation tasks skip the gate entirely.

## Hard constraints

- Never implement a requirement that is not in `requirements.md`.
- Never create a file not listed or implied in `design.md`.
- Never change a database schema without updating `design.md` first.
- Never mark a task done without running its verification command and seeing it pass.
- Never commit or push without being asked.
- Never guess when a requirement is ambiguous — stop and ask.

## Divergence protocol

If implementation must deviate from `design.md`: stop, describe the conflict, wait
for approval, update `design.md` before writing any code. Never patch code to hide
a specification error — a failing verification means the spec was wrong.

## Task verification

For every completed task, report:

```
Claim:   what the task was supposed to achieve
Command: the exact command run to verify it
Exit:    exit code
Verdict: PASS | FAIL
```

FAIL means stop and show the diagnosis. It does not mean "fix it quietly until
it passes."

### Changing a shared library

A change to `ziflow-webapp-commons`, `zibot-common` or any other artifact other repositories
depend on is not verified until its consumers are verified. Before pushing, always:

1. `mvn clean install` in the library — **with tests**, not `-DskipTests`. Installing it
   locally is what lets the consumers compile against the change at all.
2. `mvn clean install` in every repository that consumes it, and run their test suites.

A shared DTO looks harmless and is not. Adding a field changes the Lombok `@AllArgsConstructor`,
so every positional construction anywhere breaks — and in Groovy specs that break appears at
**runtime**, not compile time, so a green compile proves nothing. The cost of skipping this is
a broken `develop` and an hour of other people's work.

### Checkstyle does not run locally

`maven-checkstyle-plugin` sits in `pluginManagement` only and its ruleset comes from CI, so a
green `mvn clean install` says nothing about import order, spacing or naming. Those surface first
in Sonar, after the push.

There is no single house convention to follow: the three repositories group imports differently —
`ziflow-webapp-commons` keeps `com.*` and `lombok` in one group with `java.*` separate, while
`review-ai` and `ziflow-backend` separate `com.*` from `lombok`/`org.springframework`. So when
adding an import, copy the grouping from a neighbouring file **in that repository**, and never
introduce a blank line inside an existing group.

## Git

- Branch from `develop`, always, unless told otherwise.
- Branch naming: `feature/<TICKET-ID>`, e.g. `feature/ZFD-12345`.
- Always `git fetch` and start from the current `develop`, never a stale local copy.
- **Read code from `origin/develop`, not the working tree.** Repositories here sit on old
  feature branches and go stale by dozens of commits, so a bare `grep` over checked-out
  files reports whatever branch that repo happens to be on. Use `git -C <repo> grep
  <pattern> origin/develop` and `git show origin/develop:<path>`. Fetch first, and say
  which repositories were behind before quoting any code. A finding read from a stale
  tree is worse than no finding, because it looks like evidence.
- Never commit, push, or open a PR unless asked.
- Commit messages are **one line only** — `<TICKET-ID> <what changed>`:

  ```
  ZFD-38950 Preserve the default section flag when duplicating a checklist template
  ```

  No body, no bullet list, no explanation of the mechanism, no `Co-Authored-By` or
  other trailers. Nothing below the subject line, ever. The reasoning belongs in
  `specs/<TICKET-ID>/` — not in `git log`.
- **Pull request descriptions are short too.** A sentence or two saying what changed, in the
  same loose register as everything else written for people. No summary of the investigation,
  no measurement tables, no rationale sections, no generated-with footer. Whoever reviews the
  code reads the diff; whoever wants the reasoning opens `specs/<TICKET-ID>/`.

## Language

Code, comments, commit messages and planning documents in English.

### Tone for things other people read

Jira comments, PR comments and anything written to a person in a ticket or chat go out
**short, concrete and unpolished** — plain sentences, no formal punctuation, no tidy grammar,
the way a developer types in a hurry. No headings, no bullet-point theatre, no closing summary.
Write it as a note to a colleague, not a report.

**Short means short.** A handful of lines. Say the finding and what you need back; drop the
reasoning, the test transcript and the background — whoever wants that opens `specs/`. If it
looks like a wall of text, it is one.

This covers only prose addressed to people. It does **not** change code comments, commit
messages or the documents under `specs/` — those stay as specified above and in the Git
section.
