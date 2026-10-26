---
name: wrapup
description: Close a finished feature. Run a final whole-feature review at the deep tier against the plan's Review focus, propose lessons worth keeping, and archive the feature folder. For when every stage is committed. Runs only when the user invokes it by name; otherwise suggest it.
argument-hint: [ticket]
disable-model-invocation: true
---

# wrapup: final review, lessons, archive

Run only when the user invokes this skill by name.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

## 1. Check readiness

- Every stage in `.work/<TICKET>/plan.md` is ticked.
  - If not, list the open stages.
  - Ask whether to finish them or drop them. Mark dropped stages in the plan, and record each one
    as a `Decided:` line in the log.
- `git status --short` is clean, apart from the install changes. If anything else is
  uncommitted, ask the user to commit or park it first.
- Find the feature's base: `base-sha` in the first stage file's front matter. If it is missing,
  ask the user for the feature base. Never guess `HEAD` or `HEAD~1`.

## 2. Final review (deep tier)

1. Follow `.agents/skills/review-code/SKILL.md` with mode `feature`, the deep tier, the plan
   `.work/<TICKET>/plan.md` and the first stage's `base-sha`. It decides on the design pass and
   returns one verified result.
2. Report to the user:
   - the verdict, and the design-pass line;
   - each Review focus risk: covered or not, with evidence;
   - the findings you accept, with your fixes;
   - the design proposals, each with its benefit, its cost, and `now` or `follow-up`;
   - questions.

   **Stop.**
3. The user picks what to act on:
   - accepted fixes and accepted `now` proposals go into `plan.md` as a new stage, "NN — Final
     review fixes", recorded as a `Decided:` line. The normal loop runs from there: `/handoff`,
     then `/dispatch`;
   - each accepted `follow-up` proposal is recorded as a `Decided:` line and proposed to the user
     as its own piece of work after this feature;
   - each rejected proposal is recorded as a `Decided:` line with its reason, so it does not come
     back.

   Run `/wrapup` again afterwards. The final review runs once more only if the user asks.

## 3. Lessons (optional, user approves each one)

1. Look through:
   - the `Decided:` lines in `log.md`;
   - repeated findings across the stage reviews;
   - blockers.
2. Propose **at most 3** durable lessons. A durable lesson is a rule that would have prevented a
   real problem in future features. For each one give:
   - the lesson, in one line;
   - where it should live, always in the **harness repo**, never in the project:
     - `constitution/<repo>/AGENTS.md` (name the section) or one of its agent docs;
     - a harness skill or agent file;
     - or nowhere, if it is one-off.
   - the exact text to add.
3. Apply only the lessons the user approves, in the harness repo. Commit them there with the
   `commit-msg` skill, after the user approves the commit plan. Record each one as a `Decided:`
   line. Then tell the user to run `harness-install`, which brings them into the project.

## 4. Archive

1. Set `status: done` in the `plan.md` front matter.
2. Append a final log entry with the summary, the commits, and the lessons applied.
3. Ask the user before you move `.work/<TICKET>/` to `.work/_archive/<TICKET>/`.
