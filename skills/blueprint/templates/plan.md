---
feature: <TICKET> — <short title>
status: active | done
created: <YYYY-MM-DD>
---

# <TICKET> — <short title>

<!--
Master plan (blueprint). Written by the main agent with `/blueprint`, approved by the user.
Stage boxes here are the only record of whether a stage is done: tick them here, nowhere else.
The stage file's `status:` tracks execution.
Keep it at the level of direction: stage targets and order. Implementation details are
grilled per stage in `/handoff`; park them under "Deferred to handoff".
-->

## Goal

<One or two sentences: what the user can do when this is finished, and why it matters.>

## Context

<What exists today, what the research found. Link sources and `file:line` references,
do not paste code.>

- <finding> — <source / file:line>

## Out of scope

- <What we are deliberately NOT doing in this feature.>

## Constraints

- <Exact values every stage must respect: versions, naming, flags, APIs, permissions.>

## Review focus

<Up to five failure modes that the goal implies but no single stage's tests will catch.
The final review in `/wrapup` checks these.>

- <risk>

## Stages

<!--
Ordered by dependency. Each stage must be reviewable on its own: a reviewer could reject it
while approving its neighbours. `/handoff` may revise this list (add, split, reorder) with the
user's approval; each revision is recorded in log.md.
Mark `- [x]` only after the stage is reviewed and committed.
-->

- [ ] **01 — <name>**
  - Target: <what this stage delivers, not how>
  - Depends on: <none | stage NN>
  - Done when: <observable result>
  - Deferred to handoff: <open detail questions, e.g. where state lives, empty-state behaviour>
  - Hints: <optional; unconfirmed leads from research — handoff's scout decides>
- [ ] **02 — <name>** · Type: spike
  - Target: <the question that must be answered, e.g. "does the API support X?">
  - Depends on:
  - Done when: the question is answered with evidence; plan reviewed at the next handoff
  - Deferred to handoff:
  - Hints:

## Open questions

- [ ] <question for the user> — <who answers / where the answer was recorded>
