---
name: pickup
description: Resume a feature in a fresh session. Read the plan, log, current stage and git state, then report where things stand and the next action. Use when the user asks where we are, what is next, or to continue or resume a feature, typically at the start of a session.
argument-hint: [ticket]
---

# pickup: where am I?

Rebuild context from files, not from memory. This skill is read-only: it reports, it never fixes.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

1. **Find the feature.**
   - Use the argument or the branch ticket to find `.work/<TICKET>/`.
   - If that does not exist, list `.work/` (ignore `_archive/`) and ask which feature.
2. **Read**, in this order, and nothing more:
   - `plan.md`: the goal, stage checkboxes and open questions;
   - the last 3 entries of `log.md`, especially the latest `Next:`;
   - the current stage file: the first stage that is not ticked. Read its front matter `status`.
     If it is `active` or `done`, find the next step with the Resume section of the `dispatch`
     skill, and read only the blocks that step needs.
   - `git status --short`, `git log --oneline -10` and `git branch --show-current`.
3. **Check for drift.** Note it in the report; do not fix it.
   - Branch ticket ≠ feature ticket.
   - Uncommitted changes that no stage explains.
   - A stage that is `done` but not ticked: suggest `/dispatch NN`, which resumes at G.2 or G.4.
   - A stage that is `active` with no running subagent. The subagent died with the old session.
     Report the resume step.
4. **Report**, in at most 12 lines. Write the suggested command in your tool's syntax: the
   "Invoke a skill" entry of your adapter (`harness/adapters/<tool>.md`).

```
Feature: <TICKET> — <title>          Stages: <done>/<total>
Current: stage NN — <name> — status <status>
Last log: <date> — <title>
Next (from log): <next action>
Git: <branch>, <clean | N uncommitted files>
Drift: <none | list>
Open questions: <none | list>
Suggested command: </handoff NN | /dispatch NN (resume at X) | /wrapup>
```

Then stop and wait for the user.
