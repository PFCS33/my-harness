# harness

## What this is

A light, files-only harness for a **plan → execute → review** loop with AI coding agents.

- A smart main model plans.
- A cheap executor subagent codes.
- A reviewer subagent checks the work.
- Per-feature memory lives in plain markdown.

It needs no framework and no scripts. The skills and agent prompts are shared by every tool. What
differs per tool (models, running subagents, the paths it reads) lives in one adapter per tool, in
`adapters/`. Claude Code and Codex are supported, and one install sets up both.

### The one-way rule

- The **harness repo** (`~/Desktop/my-ai-setup/harness`) is the only place where harness files
  change. Every change is made and committed here, on `main`.
- `harness-install` exports `main` into a project and wires it up. Every run overwrites the
  **project copy** (`harness/`) and everything wired from it: the skill and agent links and
  copies, root `AGENTS.md` and `.agent-docs/`. Never edit them in a project: edits there are lost
  on the next install. This includes lessons from `wrapup`.
- The only way back is **team-file sync**. When the team changes a tracked file that the install
  replaces or overlaps, such as `AGENTS.md`, `harness-install` stops and shows the change. You
  decide what goes into the harness repo. It is committed there first, and then installed.

**Two repos, two commit rules.**
- **Harness repo:** harness files change only here. They are committed on `main`, and only after
  you approve the commit plan.
- **Project repo:** nothing the harness installs or changes is ever committed here. That covers
  `harness/`, the links and copies, and the team files the install changed: root `AGENTS.md`,
  `CLAUDE.md`, and deleted team skills. The only commits in a project are stage work, made through
  `commit-msg` after gate 3.

### Terms

| Term | Meaning |
| --- | --- |
| **harness repo** | `~/Desktop/my-ai-setup/harness`, the single source of truth (SOT) |
| **project copy** | `harness/` in a project: a read-only export of the harness repo's `main` |
| **team files** | Tracked project files that the install changes or that overlap the harness: root `AGENTS.md`, `CLAUDE.md`, team skills and agents. Each one is recorded in `constitution/<repo>/TEAM_FILES.md` |
| **install changes** | The team files the install changed in a project. They show in `git status`, are listed in `harness/VERSION`, and are never committed. Every skill skips them |

## The loop

### Diagram and roles

```
/blueprint ──► /handoff ──► /dispatch ─────────────────────────────► next /handoff … ──► /wrapup
 GATE 0         GATE 1       executor → reviewer → GATE 2 → GATE 3                         final review
 the plan       the brief                          findings  commits                       lessons, archive

/pickup — "where am I?" at the start of any session
```

| Role | Who | Tier | Effort |
| --- | --- | --- | --- |
| Planner and orchestrator | Main agent | Your choice for the session | Your choice |
| Executor | `executor` agent | fast by default. balanced when the `handoff` tier rules or `dispatch` say so. | medium |
| Reviewer | `reviewer` agent | balanced. deep for the final whole-feature review in `/wrapup`, and in `/review-pr` only on request, auth or permission service changes, or 1000+ hand-written lines. | high (Codex balanced: medium) |
| Design reviewer | `design-reviewer` agent, only when the `review-code` design trigger fires (never in stage reviews) | same tier as the reviewer it runs with | high (Codex balanced: medium) |
| Scout and researcher | `researcher` agent | balanced | medium |

The **tier** picks the model, and can change per stage. The **effort** is fixed per role, except
where an adapter says otherwise. Each tool's adapter maps the tiers to its own models. Subagents
run alongside the main agent. Each adapter's "Running the loop" says how to start, wait for and
continue them.

### Gates

| Gate | You approve | Skill | Log line |
| --- | --- | --- | --- |
| 0 | the plan outline, or the tiny-work plan | `blueprint` | `Gate 0: plan approved` |
| 1 | a stage brief | `handoff` | `Gate 1: stage NN brief approved` |
| 2 | what happens to the review findings | `dispatch` | `Gate 2: stage NN, review r — <outcome>` |
| 3 | the stage's commit plan | `dispatch` | `Gate 3: stage NN — commits <sha>, …` |

The `Gate N` line in `log.md` is the only record of an approval.

### Daily use

```
/pickup                          new session: where are we, and what is next?
/blueprint PROJ-123 <goal>       once per feature: grill the direction, approve the plan (gate 0)
"plan stage 2"                   → handoff: scout, grill the details, approve the brief (gate 1)
"run it"                         → dispatch: executor, then reviewer; decide the findings (gate 2)
"commit" … "yes"                 → commit-msg proposes the commits, you approve them (gate 3),
                                   they are made, and the stage is ticked
/wrapup                          after the last stage: final review, lessons, archive
```

Any time: "grill me on <idea>" runs `grilling`, and "review PR 123" runs `review-pr`.

`/name` means *invoke the skill `name`*. Your tool's adapter gives the exact syntax.

## Layout

```
harness/
  README.md                       this file
  research/                       harness repo only: not exported to projects (.gitattributes)
    harness-landscape.md          research notes behind the design
    superpowers-deep-dive.md      deep dive into obra/superpowers
    review-split-design.md        design note for review-code and design-reviewer
  constitution/
    backbone.md                   generic outline for a project AGENTS.md + agent docs
    <repo>/AGENTS.md              one project's AGENTS.md (copied to the project root)
    <repo>/agent-docs/*.md        on-demand docs (linked as .agent-docs/)
    <repo>/TEAM_FILES.md          every team file: its git id and decision (replace, delete, keep)
    <repo>/TEAM_AGENTS_AUDIT.md   every team AGENTS.md fact: taken (where) or skipped (why)
  skills/
    blueprint/                    master plan (+ templates/plan.md, templates/log.md)
    handoff/                      stage brief (+ templates/handoff.md)
    dispatch/                     execute → review → decide → commit
    pickup/                       resume a feature
    wrapup/                       final review, lessons, archive
    grilling/                     scoped design-tree interview
    commit-msg/                   commit messages and commit plans
    create-pr/                    PR title and description (+ templates/pr-description.md)
    review-code/                  run the reviewers, verify, return one result
    review-pr/                    pull request review
  agents/
    executor.md  reviewer.md  design-reviewer.md  researcher.md   role prompts, shared by every tool
    codex/*.toml                  Codex agent definitions, one per role and tier, pointing to the prompts
  adapters/
    claude-code.md                what is specific to Claude Code
    codex.md                      what is specific to Codex
  installer/
    harness-install/              the install, update and uninstall procedure (linked at user level, never into projects)
  VERSION                         project copy only: written by harness-install (version, date, install changes)
```

## Per-feature memory

Feature state lives in `.work/<TICKET>/` at the project root, **outside** `harness/`, so
installs never touch it. It is git-ignored.

| File | Written by | Rule |
| --- | --- | --- |
| `plan.md` | `blueprint`; revised by `handoff` and `dispatch` with your approval | The only record of whether a stage is **done**: stage checkboxes. Holds direction: targets, order, deferred questions, hints. |
| `log.md` | every skill | Add-only. Holds decisions as `Decided: D-NN` lines and approvals as `Gate N` lines. `Next:` says how the next session starts. |
| `stages/NN-<slug>.md` | `handoff` → `executor` → `reviewer` → main agent | One stage, one file, with Brief / Report / Review / Decision sections. Each part has one author. Its `status:` tracks progress. |

Finished features move to `.work/_archive/<TICKET>/`.

## Skills and agents

| Name | Trigger | Purpose |
| --- | --- | --- |
| `blueprint` | `/blueprint` only | Grill the direction and write `plan.md` and `log.md` (gate 0) |
| `handoff` | `/handoff`, or "plan the next stage" | Scout, grill the stage details, revise the plan if needed, write the brief, pick the executor tier (gate 1) |
| `dispatch` | `/dispatch`, or "run it" | Executor, then `review-code`. Fill the Decision table (gate 2), run fix rounds, propose the commits (gate 3) |
| `pickup` | `/pickup`, or "where are we?" | Read-only status report from files and git |
| `wrapup` | `/wrapup` only | Final deep-tier review, lessons, archive |
| `grilling` | "grill me", or called by blueprint and handoff | Design-tree interview in batches, with a scope |
| `commit-msg` | "commit message", "commit" | `[TICKET] type: subject` with a short categorised body. Commits only a commit plan you approved. |
| `create-pr` | "PR description", "prepare the PR" | Write the PR title and description, grouped by code pattern and scaled to the PR's size, as markdown to copy. Opens the PR only when you ask |
| `review-code` | called by dispatch, wrapup and review-pr; or `/review-code diff [base]` before you push | Decide on the design pass, run `reviewer` (and `design-reviewer`) in parallel, verify, return one result |
| `review-pr` | "review PR <n>" | Check out the PR branch, run `review-code`, report findings, design proposals and PR comments |
| `executor` (agent) | dispatch only | Implements one approved brief and writes the Report |
| `reviewer` (agent) | review-code | Spec, correctness, cleanliness and tests, in stage, feature, pr or diff mode. Writes the Review. |
| `design-reviewer` (agent) | review-code, when its design trigger fires | At most 3 redesigns that delete complexity, with benefit, cost, and now or follow-up, in feature, pr or diff mode |
| `researcher` (agent) | blueprint, handoff | Read-only scout or web research, returning `file:line` and URLs |

**Callers pass parameters. The agent's prompt owns the method.** Each agent file defines its
modes, the inputs each mode needs, and its output. A skill's prompt gives only the mode and those
inputs.

## Install, update, uninstall

1. **Link the installer once**, for each tool you use. It is a link, so it never goes out of
   date. If something else is already at that path, check it before you replace it.
   ```bash
   mkdir -p ~/.claude/skills ~/.agents/skills
   ln -s ~/Desktop/my-ai-setup/harness/installer/harness-install ~/.claude/skills/harness-install
   ln -s ~/Desktop/my-ai-setup/harness/installer/harness-install ~/.agents/skills/harness-install
   ```
   Claude Code reads `~/.claude/skills/`. Codex reads `~/.agents/skills/`.
2. **Install or update:** in a project, from either tool, invoke
   `harness-install [target-repo] [constitution-key]`. Install and update are the same run. The
   harness repo must be on a clean `main`.
3. **A new repository:** use a new key. The installer builds `constitution/<key>/` with you, in
   the harness repo, before it installs.
4. **Before `git pull`** in a project: if the pull refuses because of an install change, restore
   the install changes listed in `harness/VERSION` with `git checkout -- <paths>`, pull, then run
   `harness-install`. Nothing is lost: the install writes them again, and asks about any team
   change.
5. **Uninstall:** invoke `harness-install uninstall`.

To change the harness, edit it in the harness repo, commit on `main`, then run `harness-install`
in each project.

## Adding a tool

Write `adapters/<tool>.md` with the same sections as the existing adapters:

1. **What `<tool>` reads.**
2. **Tiers:** tier → model, and how effort is set.
3. **Running the loop**, with five entries: Start an agent, Wait for it, Ask the user while it
   runs, Continue an agent, Invoke a skill.
4. **Limits**, if the tool has any.
5. **Owned paths:** a table of Path | The install writes | Ours when.
6. **Install notes:** only what the table cannot say.
7. **Verify:** File checks, and a ready-to-paste Live check prompt.

## Credits

- `skills/grilling` is adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT).
- The design is based on [research/harness-landscape.md](research/harness-landscape.md) (in the
  harness repo only), which draws on Anthropic's guidance, superpowers, HumanLayer, GSD and
  wayfinder.
