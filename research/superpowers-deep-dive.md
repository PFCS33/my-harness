# obra/superpowers — Deep Dive

Source: https://github.com/obra/superpowers, `main` branch, plugin version 6.4.2. Read
2026-10-04 via GitHub tree API + raw files. Quoted phrases are from the files named. Items not
read are listed at the end.

---

## 1. Repo structure and bootstrap

- `skills/` (14 skills), `hooks/`, `docs/`, `scripts/`, `tests/`, `assets/`, plus one manifest
  dotdir per harness (`.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.opencode/`,
  `.pi/extensions/`, …), `GEMINI.md`, `AGENTS.md`.
- **No `agents/` and no `commands/` directory.** Every subagent is a general-purpose dispatch
  filled from a prompt template (`implementer-prompt.md`, `task-reviewer-prompt.md`,
  `re-review-prompt.md`, `code-reviewer.md`, `spec-document-reviewer-prompt.md`). The model is set
  per dispatch: `model: [MODEL — REQUIRED …]`.
- **Bootstrap (Claude Code)**: `hooks/hooks.json` registers a `SessionStart` hook with matcher
  `startup|clear|compact`. `hooks/session-start` injects `skills/using-superpowers/SKILL.md`
  wrapped in `<EXTREMELY_IMPORTANT>You have superpowers…`. The `compact` matcher re-injects it
  after every compaction.
- `using-superpowers` rules: "If you think there is even a 1% chance a skill might apply … you
  ABSOLUTELY MUST invoke the skill"; announce "Using [skill] to [purpose]"; one todo per checklist
  item; `<SUBAGENT-STOP>` block so subagents ignore it; user instructions (CLAUDE.md) override
  skills. Per-harness tool maps in `skills/using-superpowers/references/*-tools.md`.
- **Codex**: `.codex-plugin/plugin.json` → `"skills": "./skills/"`, `"hooks": {}` (no
  session-start hook; native skill discovery). Subagents need `[features] multi_agent = true`;
  spawn via `spawn_agent {fork_turns:"none"}`, resume fix rounds via `followup_task`, set `model`
  and `reasoning_effort` per spawn, backstop `default_subagent_model` in `[agents]`
  (`references/codex-tools.md`).
- Porting guide `docs/porting-to-a-new-harness.md`: "skills name actions, not tools"; "never edit
  the user's files".

## 2. Skills by phase

| Phase | Skill | Purpose |
| --- | --- | --- |
| Meta | `using-superpowers` | Bootstrap: find and invoke skills before acting |
| Meta | `writing-skills` | TDD applied to skill documents |
| Meta | `diagnosing-superpowers` | Post-mortem of a bad session from transcripts |
| Design | `brainstorming` | Classify (spike / bounded / architectural), one question at a time, spec |
| Setup | `using-git-worktrees` | Isolated workspace + clean test baseline |
| Plan | `writing-plans` | Bite-sized TDD tasks with exact paths, interfaces, tests |
| Execute | `subagent-driven-development` (SDD) | Fresh implementer + task review per task, final review |
| Execute | `executing-plans` | Inline execution in-session, one final fresh review |
| Execute | `dispatching-parallel-agents` | One agent per independent problem, all in one response |
| Execute | `test-driven-development` | "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST" |
| Execute | `systematic-debugging` | "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST" |
| Quality | `requesting-code-review` | Dispatch reviewer with `code-reviewer.md` |
| Quality | `receiving-code-review` | Verify before acting, no performative agreement |
| Quality | `verification-before-completion` | "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE" |
| Finish | `finishing-a-development-branch` | Green suite → merge / PR / keep menu → cleanup |

## 3. End-to-end workflow

| # | Step | Output | Gates |
| --- | --- | --- | --- |
| 1 | `brainstorming` | Architectural: `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` (committed). Bounded: design in chat. Spike: recommendation only | Path said aloud ("When in doubt … take the heavier one"); design approved section by section; spec self-review; user review gate; next skill is only `writing-plans` |
| 2 | `using-git-worktrees` | `.worktrees/<branch>` (must pass `git check-ignore`) | Consent; ask if baseline tests fail |
| 3 | `writing-plans` | `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` | Self-review (coverage, placeholders, type consistency, proportion); user reviews and picks Subagent-driven or Native |
| 4a | SDD | Git-ignored workspace `.superpowers/sdd/<plan>/`: `progress.md`, `task-N-brief.md`, `task-N-report.md`, `review-<b7>..<h7>.diff` | Runs continuously; only 4 stop conditions |
| 4b | `executing-plans` | Same ledger; `scripts/task-start`, `scripts/task-done PLAN N BASE -- <test cmd>` (writes ledger line only if tests pass) | Same |
| 5 | Final review | `review-package PLAN MERGE_BASE HEAD` + `code-reviewer.md` on most capable model | One fix dispatch + one scoped re-review, then adjudicate |
| 6 | `finishing-a-development-branch` | — | Tests must pass; menu "1. Merge back … 2. Push and create a Pull Request 3. Keep the branch as-is"; discard only if user types `discard` |

SDD controller per task: record BASE (`git rev-parse HEAD`) → `scripts/task-brief PLAN N` →
dispatch implementer → `scripts/review-package PLAN BASE HEAD` → dispatch task reviewer → fix loop.
Controller never fixes code itself and never runs implementers in parallel; small same-shape tasks
may be batched.

## 4. Templates

**Plan header** (`writing-plans/SKILL.md`)
```
# [Feature Name] Implementation Plan
> For agentic workers: REQUIRED SUB-SKILL: superpowers:subagent-driven-development … checkbox (- [ ]) syntax
**Goal:** / **Architecture:** / **Tech Stack:** / **Spec:** [path]
## Global Constraints   — exact values copied verbatim from spec
## Review Focus         — five untested failure modes most likely to bite
```

**Plan task**: `### Task N: [Component]`; `**Files:**` Create / Modify (`path:123-145`) / Test;
`**Interfaces:**` Consumes / Produces; 5 checkbox steps (write failing test → run, Expected FAIL →
implement → run, Expected PASS → commit). "A plan longer than the code it describes has written
the code instead."

**Implementer** (`implementer-prompt.md`): read brief first; ask questions now; "You Do Not
Dispatch Subagents"; "When You're in Over Your Head … Bad work is worse than no work"; self-review
(completeness, quality, YAGNI, testing); full report to `[REPORT_FILE]` with TDD evidence; returns
≤15 lines: `**Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT`, commits, test
summary, concerns, report path.

**Task reviewer** (`task-reviewer-prompt.md`): inputs brief + `[GLOBAL_CONSTRAINTS]` + report +
`[DIFF_FILE]`; "Do Not Trust the Report"; read-only, no re-running tests, no subagents. Output:
`### Spec Compliance` (✅ / ❌ / ⚠️ Cannot verify from diff), Strengths, Issues Critical /
Important / Minor with file:line, `**Task quality:** Approved | Needs fixes`. Plan-mandated
defects → "Important, labeled plan-mandated".

**Re-reviewer** (`re-review-prompt.md`): per finding ADDRESSED / NOT ADDRESSED ("'Attempted' is not
addressed"); New Breakage in the Fix Diff; Out-of-Scope Observations (non-blocking).

**Final reviewer** (`requesting-code-review/code-reviewer.md`): "Senior Code Reviewer"; "a spec's
silence is not permission"; required "Declined to judge" list; `**Ready to merge?** Yes | No |
With fixes`.

**Ledger** (`progress.md`), first line `# SDD ledger — plan: <plan path>`:
```
Task <N>: complete (commits <base7>..<head7>, review clean)
Task <N>: fix round <R>/5 (<X> addressed, <Y> open — …; commits a7..b7)
Task <N>: minor (deferred): …
Task <N>: parked — <finding> — Ruling: …
Ruling: <what> — <why> — <what it costs if wrong>
Final: fixed <finding> — <test> RED→GREEN, suite N/N
```
Final message must list "Rulings I made" and "Deferred minors" in full.

## 5. Memory, stop conditions, fix loop, models

- **Memory**: only git + spec/plan docs + ledger. "Conversation memory does not survive
  compaction"; "trust the ledger and `git log` over your own recollection". A task is done iff the
  ledger has `Task N: complete`. Ledger naming a different plan is ignored. Workspace is in the
  working tree (not `.git/`, which Claude Code blocks) with a self-ignoring `.gitignore`.
- **Files, not pasted context**: "a real session's dispatch hit 42k chars of which 99% was pasted
  history."
- **Stop conditions** (SDD and executing-plans): "an irreversible or destructive operation; a
  security-sensitive action; a side effect outside this worktree that norms say you ask about
  first (a merge, a push to a shared branch, a publish); and a plan so broken that every path
  forward is a guess." Everything else: "Rulings, not stalls."
- **Fix loop** (spec `docs/superpowers/specs/2026-07-15-sdd-fix-loop-redesign-design.md`): trigger
  = spec ❌, any Critical/Important, or confirmed ⚠️. Minors → ledger only. Max 5 rounds: 1–3
  resume same implementer, 4–5 fresh implementer ≥1 model tier higher ("A prior implementer
  attempted this task [N] times; you own it now."). Scoped re-review each round. At the cap,
  adjudicate each finding (park with ruling or rule). "Adjudicate only at the cap."
- **BLOCKED**: add context, stronger model, split task, or rule a plan fix. "Never … force the same
  model to retry without changes."
- **Models**: "least powerful model that can handle each role". Mechanical (1–2 files, full spec)
  → cheap; integration → standard; architecture + final review → most capable. "Turn count beats
  token price" — mid-tier is the floor for reviewers and prose-spec implementers. "Always specify
  the model explicitly."
- **Waiting**: bounded 5–10 minute waits, never short polling.
- Opt-in variant (`claude-code-tools.md`): one mid-tier orchestrator subagent runs the whole SDD
  loop one level down.

## 6. writing-skills

- "Writing skills IS Test-Driven Development applied to process documentation." RED = run a
  pressure scenario on a subagent *without* the skill, record its rationalizations; GREEN = minimal
  skill fixing them; REFACTOR = close loopholes, rationalization table, Red Flags list.
- Pressure scenarios combine ≥3 pressures (time, sunk cost, authority, exhaustion…), force an
  A/B/C choice, use real paths.
- **Description = triggers only** ("Use when…"), never a workflow summary — a workflow-summarising
  description made agents do one review instead of two.
- Word budgets: <150 (getting-started), <200 (frequently loaded), <500 (others).
- "Match the Form to the Failure": discipline failure → prohibition + rationalization table;
  wrong-shaped output → positive recipe; omitted element → REQUIRED template slot; conditional
  behaviour → observable predicate.

## 7. Borrow / skip for our harness

**Borrow**
- Status vocabulary + ≤15-line executor return; full detail in a file.
- The four stop conditions; "rulings, not stalls" with `what — why — cost if wrong`.
- Reviewer: fresh context, distrust the report, spec gate before quality gate, severities with
  actions, ⚠️ cannot-verify bucket, read-only.
- Fix loop with model escalation (shorter cap, e.g. 2 + 1 escalated round).
- "Ledger beats memory": one status source that defines done.
- Pass artifacts as file paths, never pasted history.
- Explicit model per dispatch; "turn count beats token price".
- Skill descriptions = triggers only; small word budgets.

**Skip**
- "1% chance → MUST invoke" bootstrap and session-start injection hook.
- Mandatory TDD, worktree and brainstorm gates on every task.
- Per-harness plugin manifests and helper scripts.
- Five files per task — one handoff file (brief → report → review) holds the same information.

## Not read

Skimmed: `test-driven-development`, `systematic-debugging` support files, `diagnosing-superpowers`
prompts, `docs/testing.md`. Not opened: `brainstorming/visual-companion.md` and scripts, non-Claude
/Codex `references/*-tools.md`, OpenCode/Pi/Hermes plugin code, `persuasion-principles.md`,
`RELEASE-NOTES.md`, most of `docs/superpowers/{specs,plans}`, `tests/`. Older releases may have had
`agents/` and `commands/` — not checked in git history.
