# AI Agent Harness Landscape — Research Notes

Research date: 2026-10-01. Purpose: reference material for designing a light-weight personal
harness built around a **plan → execute → review** loop with persistent per-feature memory.

Confidence markers: **[verified]** = primary source read or behaviour observed directly;
**[secondary]** = from summaries / third-party blogs; **[unverified]** = claim seen but not
confirmed — check before relying on it.

---

## 1. Cross-session memory (context engineering)

### 1.1 What the main sources do

| Source | Files | Who writes / when | Weight |
| --- | --- | --- | --- |
| Anthropic — [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | `claude-progress.txt` (log), `feature_list.json` (features with a `passes` flag), `init.sh`, git history | First session creates; every session reads at start, appends at end; frequent commits | Light |
| Anthropic — [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | "Structured note-taking" (`NOTES.md`, todo list) | Agent, during work | Light, no schema |
| Claude Code [memory docs](https://code.claude.com/docs/en/memory) | `CLAUDE.md` (<200 lines), `CLAUDE.local.md` (git-ignored), auto memory in `~/.claude/projects/…/memory/` | Human / agent | Built-in |
| HumanLayer — [Advanced Context Engineering](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md) | `thoughts/` dir: research doc, plan doc, handoff doc | Agent writes, human reviews plan | Medium |
| Manus — Context Engineering (original 403; [secondary](https://tianpan.co/blog/2026/03/02/context-engineering-lessons-from-manus)) | `todo.md` that the agent keeps **rewriting** ("recitation") | Agent, continuously | Light |
| Cline — [Memory Bank](https://docs.cline.bot/prompting/cline-memory-bank) | 6 files (brief, product, activeContext, systemPatterns, techContext, progress) | Agent reads all every session; manual update | Heavy |
| GitHub [spec-kit](https://github.com/github/spec-kit) | constitution + per feature `spec.md`, `plan.md`, `tasks.md` | Slash-command pipeline | Heavy |
| [Kiro specs](https://kiro.dev/docs/specs/) | `requirements.md`, `design.md`, `tasks.md` | IDE-driven | Heavy |
| [GSD](https://github.com/gsd-build/get-shit-done) (archived → gsd-core) | `.planning/` with `PROJECT`, `ROADMAP`, `STATE.md` ("living memory"), per-phase PLAN/SUMMARY | Orchestrator | Heavy |
| Every — [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | `docs/plans/<date>-<name>-plan.md`, `docs/solutions/` (lessons) | Agent; "compound" step promotes lessons | Medium |
| Community `HANDOFF.md` ([example](https://github.com/AnastasiyaW/claude-code-config/blob/main/rules/session-handoff.md)) | One overwritten file, or `handoffs/<timestamp>.md` | Agent at session end | Light |

### 1.2 Key quotes / facts

- Anthropic chose **JSON** for the feature list because the model is "less likely to
  inappropriately change or overwrite JSON files compared to Markdown files". [verified]
- Claude Code docs: memory should hold what is *not* derivable from code or git. [verified]
- Cline Memory Bank criticism: large payload every session, token cost
  ([discussion #2979](https://github.com/cline/cline/discussions/2979)). [verified]
- `activeContext.md` vs `progress.md` overlap is **structural** — both answer "where am I".

### 1.3 What practitioners converge on

1. **One plan file with a checkbox status list** — rewritten in place (Manus todo, Anthropic
   feature list, spec-kit/Kiro `tasks.md`, HumanLayer plan checkboxes).
2. **One append-only log** — dated entries: done, decisions + why, blockers, next step
   (Anthropic progress file, GSD STATE decisions, HANDOFF.md). Often a section of the plan file.
3. **Git as the third layer** — small commits, agent reads `git log` at session start.
4. **Stable knowledge stays out** — architecture/stack/conventions live in AGENTS.md and code,
   not in per-feature memory. (Memory Bank's brief/patterns/tech files duplicate AGENTS.md.)
5. Optional **"compound" step** — after a feature, promote durable lessons to AGENTS.md / skills.

### 1.4 Pitfalls

- **Stale state**: if updating is optional, it lags. Make the update part of a skill step.
- **Two status sources drift**: keep exactly one place that says "where am I".
- **Token cost**: everything loaded at session start costs context — keep files short; archive
  the feature folder when merged.
- **Overwrite vs append**: rewrite the checklist, append only to the log.
- **Commit vs git-ignore**: ephemeral state is usually local; durable lessons are committed.
- **Portability**: only plain repo files work across Claude Code, Codex and Cline.

---

## 2. Plan → execute → review harnesses

### 2.1 obra/superpowers — closest match ([repo](https://github.com/obra/superpowers)) [verified]

Pure-markdown skills: brainstorming → writing-plans → subagent-driven-development → code review.
Full breakdown: [superpowers-deep-dive.md](superpowers-deep-dive.md).

**Plan format** ([writing-plans](https://raw.githubusercontent.com/obra/superpowers/main/skills/writing-plans/SKILL.md))
- Header: one-sentence Goal, short Architecture, Tech Stack, **Global Constraints**, **Review
  Focus** (~5 failure modes the spec implies but no task's tests cover).
- Per task: Files (create/modify/test, line ranges), Interfaces (exact signatures), Steps.
- **"No placeholders"** — "handle edge cases" / TBD is a plan failure.
- Task boundary = "where a reviewer could reject one task while approving its neighbour".
- Planner self-review before handoff; plan must be "a plan, not a transcript".

**Subagent-driven development** ([skill](https://raw.githubusercontent.com/obra/superpowers/main/skills/subagent-driven-development/SKILL.md))
- Fresh implementer per task, given a **brief file**, never session history.
- Status vocabulary: `DONE` / `DONE_WITH_CONCERNS` / `NEEDS_CONTEXT` / `BLOCKED`. Short status
  line; details go into a **report file**.
- Model choice per task: cheapest for mechanical 1–2-file tasks, standard for integration, best
  for architecture and final review. Warning: cheap models often need 2–3× the turns.
- Bounded fix loop (max 5 rounds: resume same implementer, then a stronger fresh one, then the
  controller decides).
- `progress.md` **ledger** survives compaction: "trust the ledger and git log over recollection".
- Final whole-branch review on the strongest model.

**Executor stop conditions** ([executing-plans](https://raw.githubusercontent.com/obra/superpowers/main/skills/executing-plans/SKILL.md)):
irreversible/destructive ops, security-sensitive actions, side effects outside the worktree
(push/publish), fundamentally broken plan. Everything else: decide, log the ruling, continue.

**Implementer prompt**: ask questions before starting; "It is always OK to stop and say 'this is
too hard for me.' Bad work is worse than no work."; must not spawn subagents; show evidence.

**Reviewer prompt**: inputs = brief + constraints + report + diff (base/head SHA). "Do not trust
the report." Gate 1 spec compliance (**Missing / Extra / Misunderstood**), gate 2 code quality.
Severity Critical / Important / Minor, each with a defined action. Extra bucket: "cannot verify
from diff". **Receiving review**: verify before applying, push back with evidence, no
performative agreement.

### 2.2 HumanLayer `.claude/commands` ([dir](https://github.com/humanlayer/humanlayer/tree/main/.claude/commands)) [verified]

- `create_plan`: parallel research subagents → agree outline with user → write plan. Template:
  current state, desired end state, key discoveries, **"What we're NOT doing"**, phases with
  success criteria split into **Automated** vs **Manual** verification. No open questions allowed.
- `implement_plan`: plan checkboxes are the progress state; on mismatch stop with
  **"Expected / Found / Why this matters"**.
- `validate_plan`: compare git diff vs plan → deviations report.
- `create_handoff` / `resume_handoff`: session-to-session handoff (tasks, 2–3 critical refs,
  recent changes, learnings, artifacts, action items). Prefers `file:line` refs over code blocks.

### 2.3 Others (borrow ideas only)

- **Compound Engineering**: loop Plan → Work → Assess → **Compound**; "~80% is in plan and review".
- **spec-kit**: "constitution" concept, what-before-how. Pipeline too heavy for small changes.
- **Agent OS**: standards as small files loaded on demand.
- **GSD**: fresh context per task. Big command surface; archived.

### 2.4 Anthropic guidance

- [Claude Code best practices](https://code.claude.com/docs/en/best-practices): explore → plan →
  implement → commit; skip planning if the diff fits in one sentence; self-contained spec with
  files, out-of-scope and end-to-end verification; **adversarial review in a fresh subagent**.
  Caveat: a reviewer asked to find gaps always finds some — tell it to flag only gaps affecting
  correctness or stated requirements. Demand evidence, not claims. Keep CLAUDE.md short.
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents):
  orchestrator-workers and evaluator-optimizer patterns; start simple, add complexity only when
  it demonstrably helps.

### 2.5 Planner / executor model splitting

- [Aider architect/editor](https://aider.chat/2024/09/26/architect.html): strong model designs,
  second model edits; pairing beat solo baselines.
- Claude Code `opusplan`: Opus in plan mode, Sonnet in execution
  ([model-config](https://code.claude.com/docs/en/model-config)).
- Cline Plan/Act: separate models per mode ([docs](https://docs.cline.bot/features/plan-and-act)).

---

## 3. Tool mechanics (Claude Code / Codex / Cline)

### 3.1 Custom agents

- **Claude Code** [verified, [docs](https://code.claude.com/docs/en/sub-agents)]:
  `.claude/agents/<name>.md` (project) or `~/.claude/agents/` (user). Frontmatter: `name`,
  `description` (required); `tools`, `model` (`haiku`/`sonnet`/`opus`/`inherit`/full ID),
  `maxTurns`, `permissionMode`, `skills`, `isolation: worktree`, etc. Model resolution: per-call
  override → frontmatter → `CLAUDE_CODE_SUBAGENT_MODEL` → session model.
- **Codex CLI** [secondary — verify against official Codex docs]: custom roles in
  `.codex/config.toml` `[agents.<role>]` pointing to per-role TOML (model, reasoning, sandbox,
  instructions); `max_depth` limits nesting. Format differs from Claude Code → one role prompt
  in markdown + thin per-tool wrappers is the portable approach.
- **Cline**: no native subagent-definition file comparable to Claude Code [secondary].

### 3.2 Skills

- `SKILL.md` (front matter `name` + `description`) is a cross-tool open standard [verified in
  practice: this repo's `.agents/skills/` is read by Codex/Cline natively and by Claude Code via
  the `.claude/skills` symlink].
- Claude Code skills support extra front matter (e.g. `disable-model-invocation`, `allowed-tools`,
  `context: fork`, `model`) — other tools ignore unknown keys [secondary for non-Claude tools].
- Skills are instructions loaded into the *current* agent; subagents are separate contexts. A
  skill can instruct the main agent to spawn a subagent.

### 3.3 The key question: can a finished subagent wake the idle main agent?

**Claude Code — yes.** [verified by direct observation in the session that produced this file]
- Background subagents (the Agent tool's default) run while the main agent's turn **ends**.
  When a subagent finishes, the harness delivers a **task notification** that re-invokes the main
  agent automatically — no user message, no polling.
- While idle, the main agent consumes **no tokens**. Tokens are spent only when the notification
  arrives and the main agent reads the result.
- Foreground (blocking) subagent: the main agent also spends nothing while waiting — only the
  subagent's final message is added to its context. Difference: the user cannot chat with the
  main agent until it returns.
- Polling (sleep loops, repeatedly checking status) is the only pattern that wastes tokens —
  don't do it. Some third-party posts report large polling costs
  ([issue #90443](https://github.com/anthropics/claude-code/issues/90443)) [unverified numbers].
- Implication: a full **execute → review** chain can run without the user as middle-man; the
  main agent only needs to stop where *you* want a decision gate.

**Codex CLI — uncertain.** [unverified] Multi-agent support exists, but whether a background
child re-invokes an idle parent is not confirmed. Safe fallback: blocking subagent call (main
waits, costs nothing while waiting) or the user relays.

**Cline — no native mechanism** [secondary]; the user relays or uses Plan/Act switching.

### 3.4 Nesting and user interaction

- Claude Code: subagents cannot ask the user directly; they return to the parent, which asks.
  Spawn depth is limited (configurable). Best practice (superpowers): executors must not spawn
  reviewers.
- Codex: `max_depth` setting [secondary].

### 3.5 Portability — one source of truth

- `AGENTS.md` at root: read natively by Codex, Cline and 20+ tools; Claude Code needs
  `CLAUDE.md` with `@AGENTS.md` (Claude Code reads `AGENTS.md` only when no `CLAUDE.md` exists).
- Skills in `.agents/skills/` + symlink `.claude/skills -> ../.agents/skills` (already in repo).
- Agent definitions are **not** portable (Claude `.md` vs Codex TOML) — keep role prompts in
  shared markdown, wrap per tool.

---

## 4. Ideas worth borrowing

1. File-based brief + report: handoff file = executor brief; executor appends report; reviewer
   appends review. Never paste session history into prompts.
2. Status vocabulary `DONE` / `DONE_WITH_CONCERNS` / `NEEDS_CONTEXT` / `BLOCKED` + short list of
   hard stop conditions; everything else the executor decides and logs.
3. One status source (plan checklist) + one append-only log.
4. Handoff template: goal, context, constraints, files, interfaces, steps, **out of scope**,
   acceptance criteria (automated vs manual), verification commands, stop conditions, report.
5. Reviewer: fresh context, distrust the report, spec gate (Missing/Extra/Misunderstood) then
   quality gate, severities with actions, `file:line`, only correctness/requirement findings.
6. Bounded fix loop (~2–3 rounds), then escalate model or return to planner.
7. Blocker format "Expected / Found / Why it matters".
8. "No placeholders" planner self-check before dispatch.
9. Explicit `model:` per agent; pick executor model per stage complexity.
10. Optional "compound" step: promote lessons into AGENTS.md / skills after a feature.

## 5. Things to avoid (too heavy)

- Full pipelines: spec-kit, BMAD, AI-DLC, GSD, Kiro-style 3-doc specs for every change.
- Mandatory TDD / worktree / brainstorm gates on every task.
- Large skill catalogues (Compound Engineering: 36 skills) and parallel reviewer fleets for
  small stages.
- Plans containing full function bodies.
- Long always-loaded rule files — move workflow detail into skills that load on demand.
- Executors spawning reviewers; reviewers chasing nits.

---

## 6. Layered grilling (plan level vs stage level)

Research date: 2026-10-05. Question: which harnesses interview at a high level first (direction)
and then again per phase/stage (implementation details), with a path back to revise the plan?
Sources were read as raw files; gaps are marked [unverified].

### 6.1 Who layers the questioning

| Harness | Layering | Deferral | Feedback to the plan |
| --- | --- | --- | --- |
| **GSD** — `new-project` → `discuss-phase` ([workflow](https://raw.githubusercontent.com/gsd-build/get-shit-done/HEAD/get-shit-done/workflows/discuss-phase.md)) | Yes: project questioning → ROADMAP; per-phase interview → `{phase}-CONTEXT.md` | Phase boundary is "FIXED"; scope creep → "Deferred Ideas: Don't lose it, don't act on it" | Weak: backlog note only [no formal loop verified] |
| **mattpocock `wayfinder`** ([skill](https://raw.githubusercontent.com/mattpocock/skills/HEAD/skills/engineering/wayfinder/SKILL.md)) | Yes, different shape: a map of decision tickets, one resolved per session | "Not yet specified" fog; ticket only "when the question is already sharp" | Strong: each answer may add, graduate, update or delete tickets |
| **superpowers** `brainstorming` → `writing-plans` | Partly: decompose into sub-projects first, then brainstorm each | "Don't spend questions refining details of a project that needs to be decomposed first" | One-way ratchet: hidden complexity upgrades the path |
| **HumanLayer** `create_plan` / `iterate_plan` | Both layers in one command | Research first: "Only ask questions that you genuinely cannot answer through code investigation" | `iterate_plan`: add, split or rescope phases, user-driven |
| **spec-kit** `/clarify` → `/plan` | Spec → plan, not per stage | `/clarify` excludes "plan-level execution details (unless blocking correctness)"; ≤5 questions, ordered by rework risk | Unknowns marked NEEDS CLARIFICATION → research tasks |

### 6.2 Best match: GSD, compared with ours

| Aspect | GSD | Ours |
| --- | --- | --- |
| Top layer | `new-project`: deep questioning → roadmap | `blueprint`: direction |
| Detail layer | `discuss-phase` per phase | `handoff` per stage |
| Detail-layer rule | "Discussion clarifies HOW to implement what's scoped, never WHETHER to add new capabilities" | HOW vs WHETHER push-up test (adopted) |
| Inherited decisions | Skipped; shown as "You chose X in Phase 5" | Settled list; shown in questions (adopted) |
| Delegated choices | "Claude's Discretion" section | "Agent's discretion" in the Brief (adopted, low-risk only, forces sonnet) |
| Decision record | CONTEXT.md with D-01, D-02… | `log.md` with D-NN IDs (adopted) |
| Revise the plan | Weak | Push-up → proposed revision → user approves |
| Weight | Heavy (modes, flags, checkpoint JSON, SDK) | Light |

### 6.3 Adopted into our skills

1. HOW vs WHETHER test for park vs push-up (GSD).
2. Show inherited decisions inside questions (GSD).
3. Order each grilling round by rework risk (spec-kit).
4. "Agent's discretion" list, low-risk only (GSD).
5. Tiny-work exit: one-stage features skip the full blueprint (wayfinder).
6. Decision IDs `D-NN`, one home in `log.md` (GSD, wayfinder's "a decision lives in exactly one place").

Not adopted for now: wayfinder's unsliced "fog" list in `plan.md`. Revisit after real use.
Avoid: GSD modes and SDK, tracker-backed maps, superpowers' per-change approval gates, BMAD personas.
