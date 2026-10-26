---
name: dispatch
description: Run an approved stage handoff. The executor subagent codes it, review-code runs the reviewer and checks its findings, the main agent fills the Decision table and reports to the user (gate 2), runs any fix round, and proposes the commits (gate 3). Use when the user approves a handoff and asks to run, execute or start the stage, answers a gate-2 or gate-3 question, or once the stage is committed.
argument-hint: [stage number]
---

# dispatch: execute → review → decide → commit

You are the main agent and the orchestrator. Subagents do the coding and the reviewing. You make
the decisions they cannot make, and you check their work.

**Subagents:** start them, wait for them and continue them with the entries in your adapter's
"Running the loop" (`harness/adapters/<tool>.md`). Read it before the first one.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

**Rounds.** A stage runs in rounds. Round 0 is the first run, and round r is fix round r. **At
most 2 fix rounds:** a fix round is not offered at review 2. Each round leaves three records, in
this order:
1. an **executor block**: the `## Report` in round 0, then a `**Fix round r:**` block;
2. a **review block**: the `## Review` in round 0, then a `**Re-review r:**` block;
3. a **`Gate 2` line** for review r, in `log.md`.

**The stage file.** You own the front matter. The executor writes only its Report blocks, and
the reviewer only its Review blocks. Keep the original `base-sha` and `ignore-in-diff`.

## Resume

Start here when the stage's status is already `active` or `done`, for example in a new session.

- **`done`:** if `git --no-pager log --oneline <base-sha>..HEAD` shows no commits, go to G.2.
  Otherwise go to G.4.
- **`active`:** take the latest round r: the highest round with any record, or round r+1 when the
  `Gate 2` line for review r says "fix round". Walk its three records in order, and stop at the
  first one that is missing or unfinished:
  1. **Executor block:** missing → restart the executor (below). Status `NEEDS_CONTEXT` or
     `BLOCKED` → B.
  2. **Review block:** missing → C in round 0, or F.3 in a fix round.
  3. **`Gate 2` line:** missing → `review-code` with "verify only" (the review block exists), then
     D and E. Present → follow its outcome (F).

**Restarting the executor.** Run `git status --short`, and list the stage's changes since
`base-sha`. Ask the user to keep them (the new executor continues from them) or to discard them.
In a fix round, discarding also discards round 0's work: say so. To discard, show the exact
files, and after a yes, restore them with `git checkout -- <files>` and delete the files the
stage created. Then start the executor as in A.4, or F.2 in a fix round.

## A. Start the executor (round 0)

1. Load the handoff: the stage in the argument; else a stage file in `.work/<TICKET>/stages/`
   that is `active`, or `done` but not ticked in `plan.md`, which you resume (Resume); else the
   latest `approved` one. If its status is `draft`, stop and send the user to `/handoff`.
2. Run `git status --short`. Skip the install changes, and sort the rest into two groups:
   - **Text files** (`*.md`, `*.txt`, other docs and notes) do not block. Write their paths into
     the front matter as `ignore-in-diff: [<paths>]`, so the reviewer skips them.
   - **Anything else** (source, tests, config, lockfiles, assets) blocks: it would mix into the
     stage diff. Ask the user to commit or park it first. Do not stash or commit for them.
3. Write `base-sha: <git rev-parse HEAD>` and `status: active` into the front matter.
4. Start the `executor` agent at the `executor-tier` from the front matter.
   - Prompt: the handoff path and "Implement the Brief and fill the Report section."
   - Add nothing else. The file is the whole brief.
5. Tell the user "Executor (<tier>) running stage NN." Wait for it.

## B. Executor finished

Read its short status and its latest executor block.

- **DONE** or **DONE_WITH_CONCERNS**: go to C in round 0, or F.3 in a fix round. Carry any
  concerns into your report at gate 2.
- **NEEDS_CONTEXT** or **BLOCKED**: decide who answers.
  - **You answer** only if all of these are true:
    - the answer is already in `plan.md`, `log.md`, a stage Decision, `AGENTS.md` or one of its
      agent docs;
    - **or** it is reversible, inside the stage scope, and needs no new architecture, dependency
      or behaviour choice.

    Then:
    1. Continue the same executor with the answer. If it is gone, restart the executor (Resume).
    2. Record the ruling as a `Decided:` line in `log.md`.
    3. Wait for it.
  - **Otherwise ask the user.** Give the blocker as Expected / Found / Why it matters, plus 2–3
    options with your recommendation. Stop.
  - If a fast-tier executor blocks a **second** time in this stage, set
    `executor-tier: balanced`, record it as a `Decided:` line, and restart the executor (Resume)
    at that tier.

## C. Review (round 0)

Follow `.agents/skills/review-code/SKILL.md` with mode `stage` and the handoff path. The design
pass never runs in stage mode. It returns one verified result.

## D. Fill the Decision table

From the verified result of `review-code`, fill the `## Decision` table, one row per finding and
"Could not verify" item in the latest review block:
- Accept: `yes`, `no` or `changed`.
- Reason: one line.
- Final fix instruction: precise enough for the executor.

Carry the problems that the reviewer missed into your report at gate 2.

## E. Report to the user (GATE 2)

Keep it short and scannable:
- **Execution:**
  - status;
  - files changed;
  - checks run with pass or fail;
  - deviations and concerns.
- **Review:**
  - the verdict;
  - counts by severity;
  - the Decision table, with your reasoning where you disagreed with the reviewer;
  - the problems the reviewer missed.
- **Outcomes:** the user picks one.
  - **Fix round:** the executor applies the accepted fixes, then the reviewer checks them. At
    review 2, say "fix round not available: the limit is reached".
  - **Fix directly:** you apply the accepted fixes yourself, and show the diff. No re-review.
  - **Move to a new stage:** the accepted findings become a new stage in `plan.md`.
  - **No fixes:** close the stage as it is. Each rejected finding keeps its reason in the
    Decision table.
- **Questions:**
  - anything where the right fix depends on the user's preference or a product decision;
  - give options with a recommendation.

**Stop.** Wait for the user to pick an outcome, and to confirm, change or reject the decisions.

## F. Apply the gate-2 outcome

1. Update the Decision table with the user's choices. Append the `Gate 2` line to `log.md`:
   `Gate 2: stage NN, review r — fix round: #… | fixed directly: #… | moved to stage NN | no fixes`.
2. **Fix round** (round r+1):
   - If any accepted finding is Critical and the executor was fast tier, set
     `executor-tier: balanced`, record it as a `Decided:` line, and start a **new** executor at
     that tier. A continued agent keeps its tier.
   - Otherwise continue the same executor if it is available, or start a new one at the recorded
     tier.
   - Prompt: "Fix round r+1. Apply only the fixes in the Decision table of <path>."
   - Wait for it, then go to B.
3. **Re-review** (after B, in a fix round): follow `review-code` with mode `stage`, round
   "re-review r+1" and the handoff path. Then return to D and E, with a shorter report.
4. **Fix directly:** apply the accepted fixes yourself, inside the stage scope. Show the user the
   diff. Then go to G.
5. **Move to a new stage:** add a stage after this one to `plan.md`, with the accepted findings as
   its target, and record it as a `Decided:` line. Then go to G.
6. **No fixes:** go to G.

## G. Close the stage (GATE 3)

1. Set `status: done`.
2. Run the `commit-msg` skill for this stage's changes. It shows the commit plan, and lists what
   it leaves out on purpose: the install changes and the `ignore-in-diff` paths.
3. **Stop.** Wait for the user to approve the plan, or to say they will commit themselves.
4. Once the stage is committed:
   - list its commits with `git --no-pager log --oneline <base-sha>..HEAD`;
   - tick the stage in `plan.md`;
   - append a log entry with `Gate 3: stage NN — commits <sha>, …`, the decisions, and
     `Next: /handoff NN+1`, or `Next: /wrapup` after the last stage.
