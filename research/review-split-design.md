# Review split — design note

Date: 2026-10-19. Status: approved and implemented, including `diff` mode (see the last
section).

## Problem

The review only asked "is this diff correct and minimal?". Three things in the wording caused
that:

- `agents/reviewer.md` had no design check and required every fix to be "small and clean";
- `review-pr` §4 dropped fixes that were not "within this PR's scope";
- the severity scale had no slot for a bigger idea.

The PR workflow and "how to review" were also mixed in one skill, and "check every finding" was
written twice (`dispatch` §D and `review-pr` §4).

## Decisions

- One general skill, **`review-code`**, owns orchestration: it starts the reviewers, checks their
  findings and merges the result. Two agents own the rubrics: **`reviewer`** (correctness) and
  **`design-reviewer`** (design).
- No separate `design-review` skill. A trigger inside `review-code` decides when the design pass
  runs, and `--design` / `--no-design` override it.
- The name `review-code` avoids clashing with Claude Code's built-in `code-review` (alias
  `/review`) and Codex's built-in `/review`.
- `review-pr`, `dispatch` and `wrapup` all call `review-code`. `review-pr` keeps only the PR steps.
- The design pass never runs in stage mode. A stage is bound by its approved brief, and design
  choices belong to `handoff` grilling.

## Files and references

```
                 user: /review-pr <PR>
                          │
   dispatch (stage) ──┐   │
   wrapup (feature) ──┼───┴──► skills/review-code/SKILL.md
                      │            │  reads: harness/adapters/<tool>.md ("Running the loop")
                      │            │  decides: design trigger
                      │            ├──starts──► agents/reviewer.md
                      │            │              reads: AGENTS.md, .agent-docs/review-checklist.md
                      │            └──starts──► agents/design-reviewer.md   (only if the trigger fires)
                      │                           reads: AGENTS.md (§1, §3), plan.md + log.md Decided: lines
                      ▼
          the caller formats the result (Decision table at gate 2, PR comments, wrapup report)
```

- Callers point to `review-code` by path (`.agents/skills/review-code/SKILL.md`), which works in
  both Claude Code and Codex. `review-code` is not request-only, so other skills can use it.
- The main agent relies only on each agent's **Output** section, never on its rubric.

### One owner per rule

| Rule | Owner |
| --- | --- |
| Severity scale | `agents/reviewer.md` |
| Design proposal fields and guards | `agents/design-reviewer.md` |
| Design trigger | `skills/review-code/SKILL.md` |
| Verifying findings and proposals | `skills/review-code/SKILL.md` |
| Decision table, gate 2 | `skills/dispatch/SKILL.md` |
| PR comment format and posting | `skills/review-pr/SKILL.md` |

### What each caller does with design proposals

| Caller | `now` proposals | `follow-up` proposals |
| --- | --- | --- |
| `review-pr` | a paste-ready **Design (suggestion)** comment | a comment suggesting a follow-up ticket |
| `wrapup` | the user picks; accepted ones join "NN — Final review fixes" | `Decided:` line, proposed as a separate session |
| `dispatch` | never fires | never fires |

## Design trigger (from Compound Engineering's `maintainability` persona)

- Never in stage mode.
- In pr and feature mode, it fires when any of these is true:
  - 200+ changed hand-written lines, not counting lockfiles, generated files, translations,
    snapshots, and pure renames or deletions;
  - a new hook, component, util, context or store;
  - a change to a shared module used by 2+ packages;
  - files moved or split, with edits;
  - `--design`.
- Skipped for mechanical-only diffs, or with `--no-design`. The reason is stated in one line.

## `diff` mode (added after the first version)

- **Why:** work done outside the plan loop (a quick fix, a CI fix, a merge resolution) had no
  harness review before the push. The built-in Claude `code-review` and Codex `/review` check a
  local diff, but not the project checklist or the design pass.
- **Entry point:** `/review-code diff [base]`. No new skill, because it would only wrap
  `review-code`.
- **Diff:** everything since the merge-base with `base`, including uncommitted and untracked
  files. It is meant for "before I push".
- **Base:** the `base` argument, or the release branch from the project's git doc. Ask when it is
  unclear.
- **Spec:** the user's description plus the commit messages. Ask for a description if there is
  none.
- **Design pass:** the same trigger as pr mode.
- **Report:** `review-code` reports to the user itself. There are no PR comments and nothing is
  posted.

## Sources

- obra/superpowers `code-reviewer.md`, `task-reviewer-prompt.md`:
  https://github.com/obra/superpowers
- Compound Engineering `ce-code-review` and its `maintainability` persona:
  https://raw.githubusercontent.com/EveryInc/compound-engineering-plugin/main/docs/guides/ce-code-review.md
- Claude Code `/simplify`, `/code-review`: https://code.claude.com/docs/en/commands
- Claude Code `code-review` plugin:
  https://raw.githubusercontent.com/anthropics/claude-code/main/plugins/code-review/commands/code-review.md
- Codex review rubric:
  https://raw.githubusercontent.com/openai/codex/main/codex-rs/prompts/templates/review/rubric.md
- Google eng-practices: https://google.github.io/eng-practices/review/reviewer/looking-for.html
