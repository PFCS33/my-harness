---
stage: <NN> — <name>
feature: <TICKET>
executor-tier: fast | balanced
base-sha: <git rev-parse HEAD before execution starts>
ignore-in-diff: [] # pre-existing text-file changes the reviewer skips (set by dispatch)
status: draft | approved | active | done
---

# Stage <NN> — <name>

<!--
One file per stage, four sections, four authors:
  Brief     → main agent (`/handoff`), approved by the user (GATE 1)
  Report    → executor subagent
  Review    → reviewer subagent
  Decision  → main agent, after checking the review; user confirms (GATE 2)
The front matter belongs to the main agent. Each author edits only its own section.
The executor reads only this file. Everything it needs must be here or linked from here.
-->

## Brief

### Goal

<One sentence: what this stage delivers.>

### Context

<Where this stage sits in the master plan. Interfaces or results from earlier stages that
this stage relies on. Link `file:line`, do not paste large code.>

### Constraints

- <Exact rules: naming, versions, patterns to follow, things that must not change. Cite decisions as (D-NN).>

### Files

<!--
modify / create rows are the expected scope and must use exact paths.
pattern rows are 1–2 reference files to copy the shape from, not a reading list.
-->

| Action  | Path     | Notes                                 |
| ------- | -------- | ------------------------------------- |
| modify  | `<path>` | <what and where, line range if known> |
| create  | `<path>` | <purpose>                             |
| pattern | `<path>` | <what to copy from it>                |

The executor may touch an unlisted file only when a listed step needs it, and must list it
under "Deviations". Needing more than 2 unlisted files means the brief is wrong: stop and report.

### Steps

<!-- One decision per step. No "handle edge cases" or "TBD" — name the edge case. -->

1. <step>
2. <step>

### Out of scope

- <What the executor must not touch, even if it looks related.>

### Agent's discretion

<!-- Low-risk choices the user explicitly left to the executor. Usually empty. -->

- <none | choice, e.g. "name of the local helper">

### Acceptance criteria

- Automated:
  - [ ] `<command>` → <expected result>
- Manual (the user checks):
  - [ ] <behaviour to look at>

### Stop and report back if

- Any step would need a destructive or irreversible action (delete data, force push, reset).
- A change touches security, auth or permissions beyond what the steps say.
- The work needs files or behaviour listed under "Out of scope", or more than 2 unlisted files.
- The code does not match what this brief describes (the plan is wrong).

## Report

<!--
Written by the executor. Replace this comment.
Status is exactly one of: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
Fix round r adds a `**Fix round r:**` block below the report, starting with its own Status line.
-->

**Status:**

**Done:**
- <what was implemented, step by step>

**Files changed:**
- `<path>` — <one line>

**Evidence:**
```
<verification commands and their real output (trimmed)>
```

**Deviations from the brief:**
- <none | what and why>

**Discretion choices:**
- <none | item → what you chose>

**Concerns / blocker:**
<!-- For NEEDS_CONTEXT / BLOCKED use: Expected / Found / Why it matters -->

## Review

<!--
Written by the reviewer. Replace this comment.
Verdict is exactly one of: APPROVE | CHANGES_REQUESTED
Re-review r adds a `**Re-review r:**` block below the review, starting with its own Verdict line.
-->

**Verdict:**

**Spec check:** Missing: <…> · Extra: <…> · Misunderstood: <…>

**Findings:**

| # | Severity | Area | Location | Problem | Proposed fix |
| - | -------- | ---- | -------- | ------- | ------------ |
| 1 | Critical / Important / Minor | correctness / spec / cleanliness / tests | `file:line` | <why it matters> | <concrete fix> |

**Could not verify from the diff:**
- <item for the main agent to check>

## Decision

<!--
Written by the main agent after checking every finding against the code, then confirmed
by the user. Replace this comment.
-->

| # | Accept? | Reason | Fix to apply |
| - | ------- | ------ | ------------ |
| 1 | yes / no / changed | <why> | <final fix instruction> |

**Questions for the user:**
- <question>
