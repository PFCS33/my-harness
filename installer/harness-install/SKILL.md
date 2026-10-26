---
name: harness-install
description: Install, update or uninstall the harness in one git repository, from the harness repo's main branch. Sets up the skills, agents, project AGENTS.md and agent docs for every supported tool, and syncs team-file changes into the harness repo first. For setting up a project, or bringing it to the newest main. Runs only when the user invokes it by name; otherwise suggest it.
argument-hint: '[uninstall] [target-repo-path] [constitution-key]'
disable-model-invocation: true
---

# harness-install: install, update or uninstall the harness

Run only when the user invokes this skill by name.

- **Harness repo:** `~/Desktop/my-ai-setup/harness`, called `<source>` below. It is the only
  place where harness files change. Projects always get its `main` branch, never another branch
  or uncommitted work.
- **Project copy:** `<target>/harness/`. It is read-only: every run overwrites it, and
  everything wired from it.
- Install and update are the same run: steps 1–8. Uninstall has its own section.
- Each tool's adapter in `harness/adapters/` adds its own owned paths, install notes and checks.
  One run sets up every tool, whichever tool runs it.

## Rules

- **Never commit, push or stash in the project repo.** Nothing the harness installs or changes
  is ever committed there.
- **In the harness repo, commit only a commit plan the user approved**, through the `commit-msg`
  skill (`<source>/skills/commit-msg/SKILL.md`). Never push or stash.
- Never touch `<target>/.work/`.
- Ask before writing anything into the harness repo.
- If a command or a path is missing, stop and tell the user. Do not work around it.
- If a sandbox blocks a write, request approval for it. Never work around the sandbox.

## Path rules

Every path the install writes is listed in an **owned-paths table**: the core table below, and
one table in each adapter. Each table row gives the path, what the install writes there, and
when an existing path counts as ours.

**Whose is a path?** Check in this order:
1. **Team's:** git tracks it in the project (`git -C <target> ls-files -- <path>` prints it).
2. **Ours:** it matches the "Ours when" column of its row.
3. **The user's:** anything else.

| What is at the path | Install | Uninstall |
| --- | --- | --- |
| Nothing | Create it | — |
| Ours | Make it exactly what its row says. Skip it if it already is | Remove it |
| Team's | Apply its decision in `TEAM_FILES.md` (step 4) | Restore it, as an install change |
| The user's | Stop and ask | Leave it |

- **Make it exactly** means: remove the old path, then create the new one. Never write through a
  link: `cp` onto a link writes into the link's target.
- **Team decisions:**
  - **replace:** write what its row says (the change stays local);
  - **delete:** `rm -rf <path>`;
  - **keep:** leave it.

  The change shows in `git status`, and it is never committed.
- Every owned path has its own line in `<target>/.git/info/exclude`, anchored to the project
  root, such as `/AGENTS.md` or `/.agents/skills/handoff`. An unanchored line would hide that
  name in every folder. Add the line if it is missing, and replace an older unanchored or folder
  line for an owned path with these lines. A line has no effect on a tracked path, so adding it is
  always safe.

**Core owned paths:**

| Path | The install writes | Ours when |
| --- | --- | --- |
| `harness/` | the export of `main` (step 5) | it contains `VERSION` |
| `.agents/skills/<name>`, one per folder in `harness/skills/` | a link to `../../harness/skills/<name>` | it is a link into `harness/` |
| `.agent-docs` | a link to `harness/constitution/<key>/agent-docs` | it is a link into `harness/` |
| `AGENTS.md` | a copy of `harness/constitution/<key>/AGENTS.md` | it is identical to the installed version's copy (`git -C <source> show <installed>:constitution/<key>/AGENTS.md`) |
| `.work/` | nothing but its exclude line. The install never touches `.work/` | — |

## 1. Check

1. **Source:** `<source>` must exist and contain `README.md`, `skills/`, `agents/` and
   `adapters/`.
2. **The source must be on a clean `main`.** Both must hold:
   - `git -C <source> branch --show-current` prints `main`;
   - `git -C <source> status --short` prints nothing.

   If not, stop. Tell the user what you found, and ask them to switch the harness repo to
   `main`, committing or parking their changes first. This also makes sure that this installer,
   which is read from the harness repo, is the `main` version.
3. **Mode:** if the first argument is `uninstall`, drop it from the arguments and go to
   **Uninstall** after the next two checks.
4. **Target:** the first argument, or the current directory. Run
   `git -C <target> rev-parse --show-toplevel` and use that root. Stop if it is not a git
   repository.
5. **Key:** the second argument. If it is missing, list `<source>/constitution/*/` and ask which
   one, or ask for a new key (step 3).

## 2. Confirm

- `<target>/harness/` does not exist: this is an **install**. Show the version and the key, and
  continue.
- It exists: this is an **update**. Read the installed commit from the `version:` line of
  `<target>/harness/VERSION`, called `<installed>` below. Show it, the new version
  (`git -C <source> rev-parse --short main`), and `git -C <source> log --oneline <installed>..main`.

**Overwrite list (update only).** Export `<installed>` to a temporary folder (`mktemp -d`), then
compare:

```bash
git -C <source> archive <installed> | tar -x -C <tmp>
diff -rq -x VERSION -x .DS_Store <tmp> <target>/harness
```

Every difference is an edit made in the project copy, and the update will overwrite it. Add every
owned copy that differs from its source in `<tmp>` (`cmp`), such as an adapter's agent copies.
If `VERSION` has no commit, compare with `main` instead, and say that every difference may be a
project edit.

Ask: "Update from `<installed>` to `<version>`? These project edits will be lost: …" To keep an
edit, the user stops here, makes the change in the harness repo, and runs the installer again.
If the user says no, stop.

## 3. Constitution

- `<source>/constitution/<key>/AGENTS.md` exists: continue.
- It is missing: this is a new repository. Offer to build `constitution/<key>/` **in the harness
  repo**. It is a review-heavy task, so never do it silently.
  1. Build `AGENTS.md` and `agent-docs/` from `<source>/constitution/backbone.md`, using the
     project's AI files (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.clinerules`), its README and
     its docs as raw material.
  2. Review the result with the user.
  3. Continue with step 4. The team `AGENTS.md` has no record yet, so step 4 runs the full
     coverage audit. Steps 3 and 4 are committed together, at the end of step 4.

## 4. Team-file sync

**Team files** are the tracked project files that the install changes, or that overlap the
harness:
- every tracked path in an owned-paths table, such as root `AGENTS.md` and `CLAUDE.md`;
- every tracked entry in a folder that an owned-paths table writes into: the team skills in
  `.agents/skills/` (`git -C <target> ls-tree -d --name-only HEAD .agents/skills/`), and the team
  agents in each adapter's agent folder.

`<source>/constitution/<key>/TEAM_FILES.md` records each one: its path, its git id, and the
decision. Compare each recorded id with `git -C <target> rev-parse HEAD:<path>`:

- **Same id:** apply the recorded decision (Path rules). Ask nothing.
- **Recorded, but gone from `HEAD`:** the team removed it. Drop its row.
- **New, or a different id:** decide it again with the user.
  - **Root `AGENTS.md`.** Its decision is always **replace**. What needs deciding are its facts.
    - If the recorded blob exists in the project (`git -C <target> cat-file -e <old>`), show
      only what the team changed: `git -C <target> diff <old> <new>`.
    - Otherwise, run the **coverage audit**: every fact in `git -C <target> show HEAD:AGENTS.md`.
      Facts already listed in `TEAM_AGENTS_AUDIT.md` are not asked again.
    - For each new or changed fact, the user picks **take** (and where it goes: the harness
      `AGENTS.md` section, or which agent doc) or **skip** (and why: easy to find, covered by a
      linked doc, rarely needed, or replaced by the harness). Verify each taken fact against the
      code.
  - **A team skill or agent.** Show the full list, harness and team. Flag overlaps with the
    harness: the same name, the same trigger, or the same job. Suggest **keep** or **delete**,
    with a reason.
    - A team skill or agent with the **same name** as a harness one must be deleted. If the user
      wants to keep it, stop the install: the team version would silently change the workflow.
    - Before a delete, offer to port the project facts it has, and the harness repo lacks, into
      `constitution/<key>/`.
  - **Any other team file**, such as `CLAUDE.md`: show its content, or the team's diff. Suggest
    **replace** or **keep**, and offer to port any project facts it has.
- **Write** every decision in the harness repo, in `constitution/<key>/`:
  - `TEAM_FILES.md`: each team file's path, new git id, decision and reason;
  - `TEAM_AGENTS_AUDIT.md`: each `AGENTS.md` fact, taken (where) or skipped (why);
  - the taken and ported facts, in `AGENTS.md` and the agent docs.

  Show the result as a diff, and get the user's approval.
- **Commit** in the harness repo: propose a commit plan with `commit-msg`, and commit after the
  user approves it. If the user declines, stop: the export takes only what is committed on `main`.

## 5. Export

```bash
rm -rf <target>/harness
mkdir -p <target>/harness
git -C <source> archive main | tar -x -C <target>/harness
```

Only committed files are exported, so the harness repo's local tool settings (`.claude/`,
`.codex/`) never reach a project.

## 6. Wire

1. Apply the Path rules to every row of the core table, then of each owned-paths table in
   `harness/adapters/`. Then follow each adapter's **Install notes**.
2. Remove our paths whose source is gone: our links and copies in the folders the tables write
   into, with no matching source in `harness/`.
3. If the user declines a step of an adapter, finish the other steps, and report that adapter as
   **incomplete**. Never report it as verified.
4. Write `<target>/harness/VERSION`:

   ```
   version: <version from step 2>
   date: <today>
   install-changes:
     - <each team path this install replaced or deleted, one per line>
   ```

   The **install changes** are the lines this install leaves in `git status`. Every workflow
   skill skips them, and they are never committed.

## 7. Verify

**File checks.** Run them on every install:
- Every link in an owned-paths table resolves: `test -e <link>`.
- Root `AGENTS.md`, `harness/constitution/<key>/AGENTS.md` and
  `<source>/constitution/<key>/AGENTS.md` are identical (`cmp`).
- `git -C <target> status --short` shows no path from an owned-paths table, except the install
  changes listed in `VERSION`. A path that shows up is missing its exclude line, or was changed
  without being recorded: stop and report it.
- Each adapter's **File checks**.

**Live checks.** A tool needs one when step 6 changed a path that tool loads: one of its
adapter's owned paths, or a core skill link. Take the prompt from the adapter's **Live check**.
Otherwise the tool's live check is "not needed: nothing it loads changed".

## 8. Report

Report in at most 12 lines, then add the needed live-check prompts below them:
- install or update, and the versions (installed → new);
- the target and the key;
- team-file sync: the decisions, and the harness repo commit, or "no team changes";
- the paths created, replaced or removed, per owner (core, then each adapter);
- the install changes written to `VERSION`;
- the file checks, per owner; any adapter that is incomplete;
- the live check per tool: "needed: paste the prompt below into a new session of that tool in
  this project", or "not needed". A failed live check means: fix it in the harness repo, then run
  the installer again.

## Uninstall

1. Remove every path that is ours, in each adapter's owned-paths table and then in the core
   table, except `harness/`.
2. Restore every install change listed in `harness/VERSION`, the deleted team skills included:
   `git checkout -- <path>`.
3. Remove the exclude lines the install added, and every folder the install created that is now
   empty.
4. Ask whether to delete `harness/`. Never touch `.work/`: the user keeps or archives it.
5. Report what was removed and restored, in at most 8 lines.
