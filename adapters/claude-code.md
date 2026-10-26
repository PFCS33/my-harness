# Adapter: Claude Code

How the harness runs in Claude Code. The skills and agent prompts are shared by every tool. This
file holds only what is specific to Claude Code.

## 1. What Claude Code reads

| File | Purpose |
| --- | --- |
| `CLAUDE.md` | Contains only `@AGENTS.md`, which imports root `AGENTS.md`. Claude Code does not read `AGENTS.md` on its own. |
| `.claude/skills` | A link to `.agents/skills`, so Claude Code sees the shared skills. |
| `.claude/agents/<name>.md` | Links to `harness/agents/<name>.md`. Their YAML frontmatter holds Claude Code's agent settings: `tools`, the default `model`, and `effort`. |

## 2. Tiers

| Tier | Model |
| --- | --- |
| fast | `haiku` |
| balanced | `sonnet` |
| deep | `opus` |

- The frontmatter `model` of each agent is its role's default tier.
- **Effort** comes from the frontmatter `effort` of each agent. Never pass it per call.
- The `CLAUDE_CODE_EFFORT_LEVEL` environment variable overrides every effort setting. Leave it
  unset.

## 3. Running the loop

| Entry | How |
| --- | --- |
| **Start an agent** | Start it in the background. Pass the tier's model as the `model` of the agent call. It wins over the frontmatter `model`. |
| **Wait for it** | Do any independent work first, then end your turn with one short line saying what runs. Claude Code notifies you when it finishes, and an idle main agent costs no tokens. Never sleep or check on it. |
| **Ask the user while it runs** | Allowed. |
| **Continue an agent** | Send it a message, so it keeps its context. It keeps its model, so the same tier. To change tier, start a new agent. |
| **Invoke a skill** | `/name`, for example `/handoff 02`. |

## 4. Owned paths

| Path | The install writes | Ours when |
| --- | --- | --- |
| `.claude/agents/<name>.md`, one per `harness/agents/<name>.md` | a link to `../../harness/agents/<name>.md` | it is a link into `harness/` |
| `.claude/skills` | a link to `../.agents/skills` | it is that link |
| `CLAUDE.md` | the single line `@AGENTS.md` | its content is exactly `@AGENTS.md` |

## 5. Install notes

- **`.claude/skills` is a real folder** (not a link, and not a file): leave it. Link each harness
  skill inside it instead, as `.claude/skills/<name>` → `../../harness/skills/<name>`. These links
  are owned paths too: ours when they are a link into `harness/`.
- **Git tracks `.claude/skills` as a plain file** (mode `100644`, not `120000`): the team's link
  was stored as text, and Claude Code cannot follow it. It is a team file. Its usual decision is
  **replace**, which shows in `git status` as ` T .claude/skills`.

## 6. Verify

**File checks.**
- Every owned link resolves (`test -e <link>`).
- `CLAUDE.md` is exactly `@AGENTS.md`, unless its team decision is **keep**.

**Live check.** Paste this into a new Claude Code session in the project:

```text
Harness live check. Do not edit anything.
1. Every folder in harness/skills/ appears in your skill list.
2. Every harness/agents/<name>.md is an available agent type.
Report one line per check: pass or fail, and what you found.
```
