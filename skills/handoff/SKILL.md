---
name: handoff
description: Plan one stage in detail. Scout the code, grill the stage's implementation details, revise the master plan if gaps appear, write the executor brief, pick the executor tier, and stop for user approval (gate 1). Use when the user asks to plan, prepare or write the handoff for the next (or a named) stage, after blueprint or after the previous stage is committed.
argument-hint: [stage number]
---

# handoff: plan one stage for the executor

You are the main agent. `blueprint` set the direction. `handoff` owns the **details** of one stage.
You write `.work/<TICKET>/stages/NN-<slug>.md`, the only brief the executor will see. A weak model
must be able to follow it without guessing.

**Subagents:** start them, wait for them and continue them with the entries in your adapter's
"Running the loop" (`harness/adapters/<tool>.md`). Read it before the first one.

## 1. Load context

1. Find the feature folder: `.work/<TICKET>/` from the branch ticket. If unclear, list `.work/`
   and ask.
2. Read `plan.md`. Pick the stage named in the argument, or the first unticked stage. If an
   earlier stage is not ticked, ask before skipping it. Note the stage's Target, Done when,
   `Deferred to handoff` questions and `Hints`.
3. Read the last 3 entries of `log.md`.
4. Read the `## Brief` and `## Decision` sections of the previous stage files. Skip their
   `## Report` and `## Review` sections unless the Decision points to them.
5. Collect the **settled decisions** that this stage inherits: the plan's Constraints and Out of
   scope, `Decided:` lines in the log, and earlier stage Decisions. These are never asked again.
6. If the previous stage was a **spike**, start with step 4 (the plan check), using the spike's
   answer, before you grill.

## 2. Scout

Send one `researcher` subagent in **scout** mode. Give it:
- the stage target;
- the `Hints`;
- the deferred questions;
- the relevant settled decisions.

Start it at its default tier, and wait for it.

When it returns:
- The scout confirms or replaces each Hint. If it **rejects** a Hint, add a log line now:
  `Decided: D-NN use <X>, not hint <Y> — because <reason>`. Do not edit the Hint in `plan.md`.
- Open only the `pattern` excerpts and any "Watch out" item a decision depends on.
- If the scout is unsure about locations, resolve it with a targeted second scout or a quick
  look yourself.
- Never guess paths.

## 3. Grill the stage details

Run the `grilling` skill with this scope.

**In scope:** this stage's implementation subtree. Start the frontier from:
- the `Deferred to handoff` questions;
- choices the scout surfaced, such as which pattern, which reuse, or the shape of the change;
- component or API shape, where state lives, the data flow;
- edge-case and error behaviour;
- the test approach.

**Settled:** the inherited decisions from step 1.5. Build on them, and never re-ask them.

**Parked:** details that belong to later stages. Add them to those stages' `Deferred to handoff`
lines.

**Push-up test: HOW vs WHETHER.** If a question clarifies **how** to build this stage's target,
grill it here. If it asks **whether** to add, drop or change a capability, the plan's scope or a
stage target, push it up to step 4. Being too big also goes to step 4.

**Stopping rule:** every Brief step can be written so that the executor needs **no judgement**,
except for items the user explicitly hands over as **agent's discretion**.
For a mechanical stage this may take a single round: "These are the assumptions I will write
into the brief. Confirm?". Never skip that round. It is how silent assumptions get checked.

**Agent's discretion.** A choice may be left to the executor only if all of these are true:
- the user agrees;
- it is low risk, such as a local name or the order of test cases;
- it cannot change behaviour, a public API or a shared module.

Anything else must be decided now.

The facts are your job. Use more scouts, not questions to the user.

## 4. Plan check (push-up)

Skip this step if grilling pushed nothing up and the stage is not too big.

A stage is **too big** if any of these is true:
- more than about 8 files change;
- more than about 3 decisions were needed;
- it mixes unrelated concerns.

When this step applies:
1. Pause before writing the brief.
2. Propose a concrete change to `plan.md`: which stages to add, split (`NN-a`, `NN-b`), reorder or
   retarget, and why. Show it as before → after.
3. **Stop and wait** for the user's approval.
4. Update `plan.md`, and record the revision as a `Decided:` line in `log.md`.
5. Continue with the current stage. If it was split, continue with its first piece.

## 5. Write the handoff

Copy [templates/handoff.md](templates/handoff.md) to `.work/<TICKET>/stages/NN-<slug>.md`.
- Fill only the front matter and the `## Brief` section. Leave the other section comments as they are.
- Set `status: draft`. Leave `base-sha` empty, because `/dispatch` fills it.
- Give each settled grilling answer the next decision ID, and append it to `log.md` as a
  `Decided: D-NN …` line with its reason. In the Brief, put the answer into Constraints or Steps
  and cite the ID, for example "(D-04)". Do not restate the reason.
- Put delegated choices under "Agent's discretion". Leave it empty if there are none.

Writing rules:
- Goal: one sentence. Context: link `file:line`, and paste only signatures the executor must match.
- Files:
  - `modify` and `create` rows use exact paths and are the expected scope.
  - `pattern` rows hold the scout-confirmed references, 1–2 of them. This is the binding home of
    the pattern, not `plan.md`.
- Steps:
  - One decision per step, in order.
  - Name every edge case instead of writing "handle edge cases".
  - Name the existing function to reuse instead of describing a new one.
- Out of scope: list the tempting neighbours the executor must not touch.
- Acceptance: automated checks are real commands that exist in the project. Check
  `package.json` or `AGENTS.md`. Each one has its expected result.
- Copy the **Ask first** items from the `AGENTS.md` Boundaries section, or any project stop
  conditions, into "Stop and report back if".

## 6. Choose the executor tier

The tier picks the executor's model, and each adapter maps the tiers to its models. Default is
**fast**. Use **balanced** if **any** of these is true:

| Use balanced when… | Why |
| --- | --- |
| The `modify` / `create` list is not complete or certain | A fast model is weak at finding where code goes |
| More than 3 different edits, or more than 8 files edited by hand | A fast model loses track across many changes. The same exact before/after edit in several files counts as one edit, and one scripted command (for example a `perl -pi` rename) counts as one edit for any number of files |
| No existing pattern to copy, or the pattern must be adapted, not copied | Needs design judgement |
| New state, async, caching or error-flow logic | Subtle bugs that a fast model misses |
| Config, lockfile, shared-module or build changes | Wide blast radius. The same exact edit repeated in several packages does not count |
| Tests must be designed, not just added in an existing shape | Needs judgement about what to assert |
| Any step still needs interpretation | Ambiguity is where a fast model goes wrong |
| "Agent's discretion" is not empty | A fast model makes poor judgement calls |

- Never use the deep tier as executor. If a stage seems to need it, split the stage (step 4) or
  do it yourself and tell the user.
- Write the tier in the front matter.
- In the summary, give a one-line reason, such as "fast: 2 files, copies `X` exactly".

## 7. Self-check

- [ ] No placeholders. Every step can be done without asking a question.
- [ ] Every `modify` path exists, and every `create` path's folder exists.
- [ ] Every automated check command exists.
- [ ] Out of scope is not empty.
- [ ] Every settled grilling answer appears in the Brief with its `D-NN`, and the log has the reason.
- [ ] Every "Agent's discretion" item was agreed by the user and passes the low-risk rule.
- [ ] The stage matches its Target in `plan.md`. If not, go back to step 4.

## 8. Present (GATE 1)

Show the user:
- the file path;
- the goal;
- the Files table;
- a summary of the steps;
- the executor tier and its reason;
- the decisions settled in grilling, by ID;
- the "Agent's discretion" items, if any;
- any remaining questions, as options with your recommendation.

**Stop.** Do not start execution.

When the user approves:
- Set `status: approved`.
- Append a log entry with `Gate 1: stage NN brief approved`.
- Tell the user to run `/dispatch`.

If they ask for changes, edit and present again.
