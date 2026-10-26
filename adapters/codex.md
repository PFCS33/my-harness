# Adapter: Codex

How the harness runs in Codex. The skills and agent prompts are shared by every tool. This file
holds only what is specific to Codex.

## 1. What Codex reads

| File | Purpose |
| --- | --- |
| `AGENTS.md` | Read directly. Codex ignores `CLAUDE.md`. |
| `.agents/skills/` | The shared skills, read directly. |
| `.codex/agents/<name>.toml` | **Copies** of `harness/agents/codex/<name>.toml`, because Codex does not load agent TOMLs that are links. Each TOML tells the agent to read the shared prompt in `harness/agents/<role>.md`, and to ignore that file's YAML frontmatter, which holds Claude Code settings. Codex loads them only in a project it trusts. |

## 2. Tiers

A TOML that sets `model` ignores a per-call `model` override, without any error. So each tier of
a role is its own agent, and **the agent name picks the tier**:

| Role | fast | balanced | deep |
| --- | --- | --- | --- |
| executor | `executor` | `executor-balanced` | — |
| reviewer | — | `reviewer` | `reviewer-deep` |
| design-reviewer | — | `design-reviewer` | `design-reviewer-deep` |
| researcher | — | `researcher` | — |

| Tier | Model |
| --- | --- |
| fast | `gpt-6-luna` |
| balanced | `gpt-6.1-sol` |
| deep | `gpt-6-astra` |

- **Effort** is fixed per agent in each TOML (`model_reasoning_effort`). Never pass it per call.
- The balanced `reviewer` and `design-reviewer` use `medium`, not the `high` of their Claude Code
  frontmatter, because `gpt-6.1-sol` sits at about Opus's level. Their deep variants keep `high`.
- To change a model, edit the TOMLs in `harness/agents/codex/` and the table above, nothing else.

## 3. Running the loop

| Entry | How |
| --- | --- |
| **Start an agent** | Spawn the agent named for that tier in the table above, as `agent_type`, with `fork_turns: "none"`. Give it only the prompt the skill says. Never pass `model`. |
| **Wait for it** | Do any independent work first, then call `wait_agent` with the longest timeout allowed. It also returns early: on an intermediate message, on user input, or at the timeout. If the agent is still running, handle the event and call `wait_agent` again. Never treat `timed_out: false` as "finished". Never sleep or list agents in a loop. |
| **Ask the user while it runs** | Not allowed. Never end your turn while an agent runs, and do not count on being resumed when it finishes. Finish the waits first, then ask. |
| **Continue an agent** | `followup_task`, so it keeps its context. It keeps the same agent, so the same tier. To change tier, start a new agent of the other name. |
| **Invoke a skill** | `$name`, for example `$handoff 02`, or name the skill in words. Skill files write `/name`: when you show a command to the user, write `$name`. |

## 4. Limits

- Codex ignores `disable-model-invocation`. Skills that run only on request have an
  `agents/openai.yaml` with `policy.allow_implicit_invocation: false`, which Codex enforces. Their
  body says so too.
- Claude Code's tool allowlists (`tools:` in the agent frontmatter) do not apply in Codex.
  Children inherit the parent's live permissions, so a sandbox setting in a TOML may be
  overridden. Read-only roles are kept read-only by their prompt. Verify isolation in your
  launch mode before you rely on it.
- The researcher's **research** mode needs web search enabled in Codex.

## 5. Owned paths

| Path | The install writes | Ours when |
| --- | --- | --- |
| `.codex/agents/<name>.toml`, one per `harness/agents/codex/<name>.toml` | a copy of `harness/agents/codex/<name>.toml` | it is a regular file whose first line starts with `# Codex agent:`, or a link into `harness/` (left by an older install) |

## 6. Install notes

- **Trust.** Codex loads project agents only in a project it trusts. If this project is not
  trusted yet, ask the user to trust it in Codex. Never change trust for any other project.

## 7. Verify

**File checks.**
- Each `.codex/agents/<name>.toml` is a regular file, identical to its source (`cmp`).

**Live check.** Paste this into a new Codex session in the project. A listed agent type is not
proof that the agent starts, so the check spawns each one:

```text
Harness live check. Do not edit anything.
1. Every folder in harness/skills/ appears in your skill list (team skills may appear too).
   Those that have agents/openai.yaml are request-only.
2. For each harness/agents/codex/<name>.toml: spawn agent_type <name> with fork_turns "none"
   and the prompt "Reply with your agent name only." Wait for each final answer.
3. In each child's rollout file under ~/.codex/sessions/, check that agent_role, model and
   effort match its TOML.
4. Spawn agent_type researcher with the prompt "Mode: research. Find the official Codex docs
   page about subagents. Reply with its URL only." It must answer with a working URL that it
   reached with its web tools.
Report one line per check: pass or fail, and what you found.
```
