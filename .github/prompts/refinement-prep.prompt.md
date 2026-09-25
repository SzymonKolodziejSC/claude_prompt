---
description: Prepare brief for a refinement session — fetch tickets, explore codebase, create summary
---

> Wywołanie: `np. /refinement-prep 1` — argumenty wpisz w oknie czatu zaraz po nazwie komendy.
> Odpowiednik `allowed-tools` z Claude Code (do ręcznej weryfikacji, IntelliJ Copilot nie ma jeszcze pełnej kontroli nad narzędziami per-prompt): ["Agent", "Bash", "Read", "Write", "Edit", "mcp__atlassian__*"]


# Refinement Prep

Group: `(wartość podana po komendzie w czacie)` (if empty, ask which group: 1 or 2).

Output: `refinement-sprint-<SPRINT>-group-<GROUP>.md` in workspace root.

## 1. Fetch tickets

Query Jira for tickets in Refinement status for the group:

```
Sprint = "Refinement Group <GROUP>" AND status = "Refinement"
```

For each ticket, fetch full details including comments.

## 2. Parallel exploration

For **each ticket**, spawn an Explore agent to investigate:

```
Ticket ZFD-XXXXX: "<title>"

Find:
1. Which services/repos are affected (search origin/develop)
2. Key files/classes that would change
3. Is this BE, FE, or both?
4. Any related code patterns or existing implementations

Search terms from ticket description. Report in 5-10 lines max.
```

Run all Explore agents **in parallel** (one Agent call per ticket, all in same message).

## 3. Analyze each ticket

For each ticket, determine:

| Check | Question |
|-------|----------|
| **Area** | BE / FE / Both / Infra |
| **Services** | Which repos touched |
| **Scope** | Small (1-2 files) / Medium (1 service) / Large (multi-service) |
| **Clear?** | Are requirements clear enough to estimate? |
| **Dependencies** | Blocked by other tickets? |
| **Risk** | Any technical unknowns? |

Note any **comments** on the ticket that clarify requirements or raise concerns.

## 4. Create brief

Write the output file with this structure:

```markdown
# Refinement: Sprint XX.YY — Group N

**Date:** YYYY-MM-DD  
**Tickets:** X

---

## Quick Reference

| # | Ticket | Title | Area | Scope |
|---|--------|-------|------|-------|
| 1 | [ZFD-XXXXX](#1-zfd-xxxxx) | Short title | BE | Medium |
| 2 | [ZFD-YYYYY](#2-zfd-yyyyy) | Short title | FE | Small |

---

## 1. ZFD-XXXXX

**Title:** Full title  
**Area:** BE | **Scope:** Medium | **Clear:** Yes/No

**What:** 1-2 sentences explaining what this ticket does, in plain language.

**Where:**
- `service-name` — `ClassName.java`, `OtherClass.java`

**Notes:** Key comment or concern (if any). Skip if nothing notable.

**Questions:**
- Any unclear points to raise in refinement

---

## 2. ZFD-YYYYY
...
```

## Format rules

- **Brief is mandatory.** Each ticket section: max 10-15 lines. No walls of text.
- **Plain language.** "What" should be readable by anyone, not just the implementer.
- **Clickable links.** Table links to sections, section headers link to Jira.
- **Skip empty sections.** No "Notes: none" — just omit.
- **No estimates.** This is prep, not planning poker.

## 5. Report

When done, show:
- File path
- Ticket count
- Any tickets that are unclear or risky (need extra discussion)
