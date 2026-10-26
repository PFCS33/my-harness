---
name: blueprint
description: Start a new feature. Grill the direction (goal, scope, stage targets and order, solvability), explore only the details the direction depends on, park the rest for handoff, and write the master plan and log under .work/<TICKET>/. For the start of a new feature, or when the user asks for a master plan. Runs only when the user invokes it by name; otherwise suggest it.
argument-hint: <ticket or feature name> <goal>
disable-model-invocation: true
---

# blueprint: set the direction and write the master plan

Run only when the user invokes this skill by name.

You are the main agent and the planner. This skill produces `.work/<TICKET>/plan.md` and
`.work/<TICKET>/log.md`. It writes no code. Work in this order and do not skip the gates.

Blueprint decides **direction first, details on demand**. Settling every implementation detail
up front is impossible and wasteful, so most details are grilled later, one stage at a time, in
`handoff`. A detail is explored here only when the direction depends on it.

## 1. Set up

1. Find the ticket. Use the argument, or take it from the branch name
   (`git branch --show-current`, usually `<TICKET>-<description>`). If neither works, ask.
2. If `.work/<TICKET>/` already exists, stop and suggest `/pickup` instead.
3. Run `git check-ignore -q .work/` from the repository root. If `.work/` is not ignored, the
   harness is not installed correctly: stop, and ask the user to run `harness-install`. Do not
   edit ignore files yourself.

## 2. Grill the direction

Run the `grilling` skill with this scope.

**In scope.** Settle these:
- the goal, who benefits, and how "done" will be checked;
- what is out of scope, and the hard constraints;
- **how many stages, and the target of each one**: what it delivers, not how;
- **the order of the stages**: dependencies first, and risky or foundational work early;
- **solvability**: following the stages in order plausibly reaches the goal.

**Details: the relevance test.** For every detail question, such as component or API shape,
where state lives, data flow, edge cases, test design, files or patterns, ask:

> **Would a different answer change the plan?** That means the stage count, a stage target, the
> order, solvability, the constraints, or what is out of scope.

- **Yes:** it is a *direction-relevant detail*. Explore it now, as deep as the decision needs.
  - Facts first: send a scout or a researcher. Ask the user only when it is a real choice.
  - Name the plan item it protects, for example "column support in `DeviceTable` decides
    whether stage 02 exists". If you cannot name one, the answer to the test was really no.
  - Record the answer as a `D-NN` decision. The affected stage inherits it, so `handoff` never
    asks it again.
  - If even a deep look cannot settle it, make it a **spike stage** (below).
- **No:** park it. Write it under its stage's `Deferred to handoff:` line.

**Stopping rule.** Grilling is done when all of these are true:
- every stage has a target, a dependency and an observable "done when";
- the order has a one-line justification;
- every unknown that could break the whole plan is either resolved by research or turned into a
  **spike stage** (below);
- every detail settled early names the plan item it protects;
- every other detail question is parked, not answered.

Challenge the request when there is a simpler or cleaner route, and say why.

**Research depth: feasibility, plus targeted dives.** For facts, `grilling` starts `researcher`
agents, at most 3 at a time:
- **scout** mode: "does this part of the code exist, and roughly where?";
- **research** mode: "is this approach or library viable?";
- a **targeted dive**, as deep as needed, but only for a direction-relevant detail.

Leave general file mapping to `handoff`. If research happens to find a likely pattern or file,
record it as a stage `Hints:` line. It is unconfirmed, and `handoff` decides.
Separate verified facts from assumptions. Unverified items go under Open questions.

**Spike stages.** Use one when an unknown could break the plan and research cannot settle it, for
example "does the backend API support X?". A spike stage:
- is marked `Type: spike`;
- has the question as its target;
- has "the question is answered, with evidence" as its done-when;
- comes before the stages that depend on the answer.

After a spike, the next `handoff` reviews the plan with the user first.

Grilling ends with the settled decisions (including details settled early), the parked
questions for each stage, and any spikes.

**Tiny-work exit.** If grilling shows that the whole feature fits in **one stage**, say so and
offer to skip the full blueprint. If the user agrees, that is gate 0. Skip step 3, and in step 4
write a minimal `plan.md`: the Goal, Out of scope and the one stage, with no Review focus. Then
continue with `/handoff 01`.

## 3. Agree the outline (GATE 0)

Show the user, in chat, before writing any file:
- the goal in one sentence and the out-of-scope list;
- stages as a numbered list: name, one-line target, dependency, and deferred questions. Mark
  spikes.
- one line on why this order reaches the goal;
- details settled early: each with its `D-NN`, its answer, and the plan item it protects;
- the main risks (candidates for Review focus);
- open questions, and any tradeoff as 2–3 options with your recommendation.

Stage sizing:
- A stage is one reviewable, committable unit. Usually 1–8 files.
- A reviewer could reject it while approving its neighbours.

**Stop and wait** for the user to approve or change the outline.

## 4. Write the files

1. Create `.work/<TICKET>/plan.md` from [templates/plan.md](templates/plan.md). Set
   `status: active`.
2. Create `.work/<TICKET>/log.md` from [templates/log.md](templates/log.md). Add the first entry:
   `Gate 0: plan approved`, the key decisions as `Decided: D-NN …` lines, and
   `Next: /handoff 01`.
3. Create the empty folder `.work/<TICKET>/stages/`.

## 5. Self-check before presenting

- No placeholders ("TBD", "handle edge cases", "etc.") anywhere.
- Every stage has a target, a dependency and an observable "done when".
- The stages, done in order, reach the Goal. The order has a justification.
- The plan stays at target level. There are no file lists, step lists or code, and no guessed
  details. Open details are under `Deferred to handoff`, and leads are under `Hints`.
- Review focus has 1–5 concrete risks. Each one is a failure mode, not a topic.
- Every open question is listed with who answers it.

Then give the user the file path and a 5-line summary. Next step: `/handoff`.
