---
name: review-code
description: Run a code review. Start the reviewer agent, plus the design-reviewer agent when the change is large or structural, check every finding and design proposal against the code, and return one verified result. Called by review-pr (pr mode), dispatch (stage mode) and wrapup (feature mode), which format the result for their own report. The user can also call it as `/review-code diff [base]` to review a branch and its uncommitted work before pushing, and then it reports directly.
argument-hint: <mode> [base] [--design | --no-design]
---

# review-code: run the reviewers, verify, return one result

You are the main agent. The `reviewer` and `design-reviewer` agents do the reviewing. You decide
which ones run, check their work, and hand one verified result back to the skill that called
you. The caller owns what happens next: the Decision table, the PR comments or the wrapup report.
In `diff` mode the user is the caller, and you report to them yourself (step 6).

**Subagents:** start them, wait for them and continue them with the entries in your adapter's
"Running the loop" (`harness/adapters/<tool>.md`). Read it before the first one.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

## 1. Inputs

The caller gives:
- the **mode**: `stage`, `feature`, `pr` or `diff`;
- the inputs that mode needs, as listed in the Input table of `harness/agents/reviewer.md`;
- in stage mode, the round: round 0, or "re-review r";
- the **tier**: balanced by default, deep when the caller says so;
- optionally `--design` or `--no-design`;
- optionally "verify only", when a stage's review block already exists (a resumed `dispatch`):
  skip steps 2 and 3, and verify the latest review block.

**`diff` mode**, when the user calls you directly:
- **Base:** the `base` argument. Without one, find the branch's release branch as the project's
  git doc describes, and use `origin/<release branch>`. If it is still unclear, ask. Never guess.
- **Diff:** everything since `git merge-base <base> HEAD`: commits, uncommitted changes and
  untracked files, minus the install changes.
- **Spec:** the user's description of the change, plus the commit messages since the merge-base.
  Without a description, ask "What is this change for?" before you start.
- **Tier:** balanced, or deep if the user asks.

## 2. Decide on the design pass

- **Stage mode:** never. A stage follows its approved brief, and design choices belong to
  `handoff`.
- **`--no-design`:** off. **`--design`:** on.
- **Otherwise,** read `git --no-pager diff --stat` for the mode's diff. The design pass is **on**
  when any of these is true:
  - 200 or more changed lines of hand-written code. Do not count lockfiles, generated files,
    translation files, snapshots, or files that are only renamed or deleted;
  - the diff adds a new hook, component, util, context or store;
  - the diff changes a shared module that 2 or more packages use (see `AGENTS.md` for where
    shared code lives);
  - the diff moves or splits files and also edits them.
- It is **off** otherwise, for example for a pure rename, a dependency bump, or only config,
  docs or test changes. Off skips the architectural review only. It never narrows the
  correctness or impact review, because a small config or dependency change can still change
  behaviour.
- Write the decision as one line, for the caller's report. For example: "Design pass: on, 340
  hand-written lines and a new `useDeviceFilters` hook" or "Design pass: off, rename only, no
  trigger hit".

## 3. Run the reviewers

1. Start the `reviewer` agent at the tier. Its prompt is "Mode: <mode>." plus the mode's inputs,
   for example "Mode: stage. Handoff: <path>." or "Mode: stage, re-review r. Handoff: <path>."
   Pass only the mode and its inputs. Do not add your own reading of the change, such as
   "mechanical" or "low risk". The reviewer forms its own view.
2. If the design pass is on, start the `design-reviewer` agent **at the same time**, at the same
   tier, with the same prompt.
3. Wait for both.

In stage mode the reviewer writes its Review block into the handoff. In the other modes both
agents return their results in their message.

## 4. Verify

The reviewers can be wrong too.

**For every finding and every "Could not verify" item:**
1. Open the code at `file:line`. Confirm the problem is real.
2. Judge the proposed fix:
   - Is it correct?
   - Is it the cleanest option?
   - Does it reuse existing code?
   - Does it stay in the scope of the change?

   Replace it with a better fix if you have one.

**For every design proposal:**
1. Open the code. Confirm that today's shape is as described.
2. Check that the proposed shape really removes what it claims, and that its cost is realistic.
3. Drop it if it conflicts with a `Decided:` line, if the benefit is not shown, or if it is only
   taste.

**For the coverage behind the verdict, including an APPROVE with no findings:**
1. Read the reviewer's Checked line. Name the assumption the verdict rests on, for example
   "behaviour is unchanged".
2. For each changed name, run one targeted search for consumers the reviewer did not trace. Open
   any that could change behaviour.
3. Challenge the assumption. Do not just rerun the reviewer's checks.

Put what you find under Missed, and what you could not check under Could not verify.

Then:
- Drop what is false, with a one-line reason.
- When a cleanliness finding and a design proposal cover the same code, keep the proposal and
  drop the finding as covered.
- Note any problem you saw that both reviewers missed.

## 5. Return the result

Hand the caller one result in this shape:

- **Design pass:** the line from step 2.
- **Verdict:** the reviewer's verdict, after your checks. Design proposals never change it.
- **Findings:** each with severity, area, `file:line`, the problem and the final fix.
- **Design proposals:** each with its fields from `harness/agents/design-reviewer.md`, after your
  checks. Leave this out when the design pass was off.
- **Dropped:** each rejected finding or proposal, with its reason.
- **Checked:** the consumers and boundaries traced, by the reviewer and by your own check.
- **Could not verify:** what is still open.
- **Missed:** problems you found that the reviewers did not.

Then go back to the caller's next step. In `diff` mode, go to step 6.

## 6. Report to the user (diff mode only)

1. **Summary:** what the change does in 2–3 lines, the base and merge-base, and the design-pass
   line.
2. **Findings:** a table with severity, area, `file:line`, the problem and the fix. Critical
   findings come first.
3. **Design proposals:** each with today's shape, the proposed shape, what it removes, its cost,
   and `now` or `follow-up`.
4. **Dropped**, **Checked**, **Could not verify** and **Missed**.
5. **Verdict:** the reviewer's verdict. Design proposals never change it.

Edit nothing, commit nothing and post nothing. The user decides what to fix.
