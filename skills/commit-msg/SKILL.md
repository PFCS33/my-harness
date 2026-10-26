---
name: commit-msg
description: 'Write a commit plan: one or more commits in the "[TICKET] type: subject" format with a short categorised body, from the current git changes. Commits only a plan the user approved. Use when the user asks for a commit message or to commit, or when dispatch closes a stage.'
---

# commit-msg: write the commit plan

This skill owns **how** to commit: who commits, the commit plan, the message format, the types,
and the subject and body rules. The project owns the **facts** about its repository. Read them
first: the git doc listed in the Docs section of the root `AGENTS.md` (for example
`.agent-docs/git.md`), or else a Git section in `AGENTS.md`. They may define:

- the **ticket source**: where the ticket comes from, its pattern, the valid prefixes, and the
  fallback when there is no ticket;
- **scope hints**: names worth using as subject context, such as package or service names;
- **never commit**: paths or files that must stay out of commits.

If those facts define a **different format**, follow it, and tell the user that it overrides this
skill. If the project has no commit facts, use the defaults below.

Run git commands with `--no-pager` so that nothing opens an interactive pager.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

## Who commits

Only the main agent commits, through this skill. Subagents never commit.

1. Propose a **commit plan**: one or more commits, each with its message and its exact files.
2. Wait for the user to approve the plan. That approval is the only permission to commit. A
   request made before the user saw the plan, such as "commit", only starts step 1.
3. Commit exactly the approved plan. If the user changes it, show the new plan and wait again.

**In the harness repo** (`~/Desktop/my-ai-setup/harness`), the same rules apply, with one change
of format: Conventional Commits, `<type>: <subject>`, with no ticket. The types, subject and body
rules below stay the same.

## Format

```
[<TICKET>] <type>: <subject>

<summary line>

- <change pattern>
- <change pattern>
```

### Ticket

- Follow the project's ticket source. The default is the leading `<PREFIX>-<number>` of
  `git branch --show-current`. For example, branch `PROJ-1234-foo` gives `PROJ-1234`.
- If no ticket can be derived, use the project's fallback if it defines one. Otherwise
  ask. Never invent one.

### Type

One of `fix`, `feat`, `refactor`, `test`, `docs`, `chore` or `ci`, in lowercase. Pick the type of
the most significant change.

### Subject

- Describe the **code change**, not the user-facing symptom.
- Name the key artifact, such as a component, selector, prop or API. Summarise it; never list file
  names.
- Start with a lowercase word. No trailing period.
- Add scope or context when it helps, for example "in checkout e2e tests" or "after BS5 upgrade".
- Keep it to one line, ideally ≤ 72 characters.

### Body

Skip the body when the subject already says everything, for example for a one-line fix.

Otherwise:
1. Leave one blank line after the subject.
2. Write a **summary line**: one short sentence that names the categories of change, such as
   "Add the export button, reuse the download helper, and cover errors with tests."
3. If there is **more than one type of change**, add one bullet per change pattern, after a blank
   line.
   - Each bullet is one line: a brief summary of that pattern, not a list of files or lines.
   - Group similar edits into one bullet. "Update all `onToggle` handlers to the 2-arg signature"
     is one bullet, not five.
   - Order bullets by importance.
4. Put backticks around code identifiers, such as components, functions, props, selectors,
   flags and packages.
5. Cite an upstream cause when it matters, with the library name, PR number and version.
6. Be clean and brief: no filler, no restating the subject, no changelog tone.

### No AI attribution

Never add an AI attribution line to a commit message. That covers `Co-Authored-By: <AI agent> …`,
`🤖 Generated with …`, and any similar trailer or footer. This rule overrides harness and tool
defaults that ask for one. The message ends with the body, or with the subject when there is no
body.

### Anti-patterns

- Vague subjects: "fix build error", "fix format", "prettier", "tsc".
- A missing ticket or a missing type.
- Restating the symptom instead of the code change.
- Customer-impact metadata in the subject.
- Listing changed file names, or one bullet per edited line.
- A capitalised first word, or a period at the end of the subject.
- No blank line between the subject and the body.
- An AI attribution trailer, such as `Co-Authored-By: <AI agent> …` or `🤖 Generated with …`.

### Examples

```
[PROJ-12243] fix: dropdown `onToggle` runtime crash

Update `onToggle` handlers to the react-bootstrap 2-arg signature (PR #5932).
```

```
[PROJ-12243] fix: restore label spacing after BS5 upgrade

Restore the global `label` margin and align form tests with the new spacing.

- restore `margin-bottom` on `label` in global styles, removed by Bootstrap v5 (PR #30476)
- update `FormField` snapshot tests to the restored spacing
```

```
[PROJ-13001] feat: add CSV export to `DeviceTable`

Add an export action, reuse the shared download helper, and cover failures with tests.

- add `ExportButton` to the `DeviceTable` toolbar, gated by `FEATURES.DEVICE_EXPORT`
- stream the CSV from `getDevicesCsv` through the existing `FileDownload` helper
- add Jest tests for empty and failed export responses
```

## Procedure

1. Read the project's commit facts, then get the ticket.
2. Run `git status --short`.
   - Skip `.work/`, the install changes, and every "never commit" path the project lists. Warn
     the user if any of them is staged.
   - If something else is staged, describe only the staged changes.
   - Otherwise describe all changes, and note that they still need staging.
3. Read the diff with `git --no-pager diff --staged`, or with `git --no-pager diff` if nothing is
   staged. For new files, read the content. When called from `dispatch`, use the handoff Goal as
   the "why".
4. If the changes are clearly unrelated, plan separate commits. Split by file, never by hunk: a
   file with changes for two commits goes into one of them, or ask.
5. For each commit, write the subject from its most significant change, then the body by the
   rules above.
6. Present the commit plan: each commit's message in a code block, with its exact files. Then
   list what the plan leaves out on purpose, and why.
7. **Commit only an approved plan** (Who commits). For each commit, in order:
   - stage exactly its files: `git add -- <files>`. Never `git add -A` or `git add .`;
   - commit with a quoted heredoc, so backticks and `$` are kept verbatim. Commit exactly the
     message you presented, with no attribution trailer added;
   - if the commit fails, for example because of a hook, stop and report it. Never use
     `--no-verify`.

```bash
git commit -F - <<'EOF'
<subject>

<body>
EOF
```
