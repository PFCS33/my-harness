---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases. Also used, with a scope, by blueprint (direction) and handoff (stage details).
---

<!--
Adapted from mattpocock/skills (skills/productivity/grilling), MIT License, Copyright (c) 2026 Matt Pocock.
https://github.com/mattpocock/skills
Changes: added rework-risk ordering, scoped grilling, the "enough" early stop, the hand-back to a calling skill, and "batches" for "rounds".
-->

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

**Subagents:** start them, wait for them and continue them with the entries in your adapter's
"Running the loop" (`harness/adapters/<tool>.md`). Read it before the first one.

Work the tree in **batches**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one batch: number each question and give your recommended answer. Then wait for the user's answers before the next batch.

Within a batch, order the questions by **rework risk**: first the ones whose wrong answer would
force the most work to be redone.

Format a batch like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each batch the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next batch. A question whose answer depends on another question still open in this batch belongs to a _later_ batch, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), start a `researcher` subagent to find it; don't ask the user for anything you could look up yourself. A running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the subagent to report. Prepare the rest of the frontier while it runs. Ask those questions now if your adapter allows asking the user while it runs; otherwise ask them after it returns. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

## Scoped grilling

A calling skill may give you a scope:

- **In scope**: the branches to explore.
- **Parked**: questions deferred to a later layer. Never ask them. If a new question falls here,
  add it to the parked list.
- **Settled**: decisions inherited from earlier work. Never ask them again. Build on them.
  When a question's options touch a settled decision, show it in the question, for example
  "You chose X in stage 02 (D-03)", so the user can spot a conflict.
- **Push-up**: a question whose answer would change a *higher* layer, such as the plan the
  caller is working inside. Do not answer it yourself. Flag it to the caller with the reason.
- **Stopping rule**: when the session is done. It replaces "every branch visited". The frontier
  is empty once every in-scope branch meets the caller's stopping rule.

## Ending early

If the user says "enough", "stop grilling" or similar, stop asking. List the branches that are still open and the answer you will assume for each, and ask the user to confirm those assumptions.

## When another skill called you

If a skill such as `blueprint` or `handoff` called you, finish with:
- a short **settled decisions** list: each decision, the chosen answer, and the reason in one line;
- the assumptions the user confirmed;
- the **parked** questions;
- the **pushed-up** questions.

Then return control to that skill's next step instead of acting yourself.
