# Constitution backbone

This is a generic outline for a project's `AGENTS.md` and its on-demand agent docs. For a new
repository, `harness-install` builds `constitution/<key>/` from it, in the harness repo. It fills
it from that repository's existing AI files (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`,
`.clinerules`), its README and its docs.

## The rule: always loaded vs on demand

`AGENTS.md` is loaded in **every** session. Agent docs in `.agent-docs/` are opened only when a
task needs them. For each fact, ask:

- **Is it needed in most sessions?** Put it in `AGENTS.md`.
- **Is it needed only for some tasks?** Put it in an agent doc, and add one line for it in
  `AGENTS.md` §6.
- **Would an agent make a mistake without it?** If not, delete it.

Targets:
- `AGENTS.md`: about 100–150 lines. Under 300 is a hard limit.
- Each agent doc: under about 80 lines, on one topic.

## How to fill it

1. Collect the team's existing agent files and docs. Treat them as raw material, not as truth.
2. Check every fact against the repository: `package.json`, configs, folders, source. Drop
   anything you cannot verify.
3. Give each fact one home. If a human doc already explains it well, link to that doc.
4. Leave out:
   - **workflow and process**, because the harness skills own it;
   - **generic advice** a good model already follows;
   - **anything a linter or formatter enforces**;
   - **personal preferences**, which belong in your tool's personal instructions file
     (`~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`);
   - **volatile state**, which belongs in `.work/` or the ticket.
5. Leave tool-specific instruction files, such as `CLAUDE.md`, to each adapter's owned paths in
   `adapters/`. Codex and most other tools read `AGENTS.md` directly.

## `AGENTS.md` sections

Start with a **2–3 line intro**, with no heading: what the product is, its main stack, and how
the file is loaded. Then use these sections in this order.

### 1. Project map

- **Contains:**
  - top-level folders and packages, and what lives in each;
  - naming quirks, such as folder name ≠ package name;
  - prefixes or IDs agents must use;
  - the standard package layout;
  - where new code goes.
- **Not:** file-by-file descriptions, or anything `ls` makes obvious.

### 2. Commands

- **Contains:**
  - the pinned runtime and package-manager versions;
  - a table of the commands agents run most, with exact flags: install, start, scoped typecheck,
    lint, test, format;
  - which commands are slow, and the scoped alternative;
  - environment and config sources.
- **Not:** human setup steps. Link the README.

### 3. Conventions

- **Contains:**
  - rules a model would not guess: framework versions and allowed APIs, the component library
    order, styling, state ownership, the data layer, feature flags, forms, permissions,
    accessibility, comment policy;
  - **one real code example** that shows the shape to follow.
- **Not:** generic best practices, or rules the linter or formatter enforces.

### 4. Testing

- **Contains:**
  - the test types and when each one is required;
  - where specs live;
  - shared helpers;
  - known traps;
  - **the smallest meaningful check** for a typical change.
- **Not:** test philosophy.

### 5. Boundaries

Three tiers:
- **Always:** actions to take every time, such as running the smallest check.
- **Ask first:** risky but sometimes needed. The `handoff` skill copies this tier into every
  stage's stop conditions.
- **Never:** irreversible, expensive or forbidden actions, including the never-commit paths.

**Not:** rules that are already stated above. Keep one line per item.

### 6. Docs (read when)

- **Contains:** a table with the columns "Read this…" and "…when". Agent docs come first, then
  human docs, submodule guides and repository skills.
- **Not:** summaries of those docs.

## Agent docs (`.agent-docs/`)

Create only the docs this repository needs. Common ones:

| Doc                   | Contains                                                                   | Read by                          |
| --------------------- | -------------------------------------------------------------------------- | -------------------------------- |
| `git.md`              | Branch model and naming; **commit facts**: ticket source, valid prefixes, no-ticket fallback, scope hints; PR expectations | `commit-msg` skill, PR work |
| `review-checklist.md` | Project review items, grouped by area, each one checkable against a diff   | `reviewer` agent, via `review-code` |
| `<topic>.md`          | One architecture mechanic that causes real bugs, such as module federation, i18n or code generation | Tasks touching that topic |
| `troubleshooting.md`  | A table of symptom → likely cause → fix                                     | When something breaks           |

The commit **format** lives in the `commit-msg` skill, not in `git.md`. `git.md` only states
this repository's facts, plus a format override when the repository needs one.
