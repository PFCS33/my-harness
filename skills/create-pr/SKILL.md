---
name: create-pr
description: Write a pull request title and description as markdown source for the user to copy. Groups the changes by code pattern, scales the sections to the PR's size, and states verification as results. Never pushes or opens the PR unless the user explicitly asks in that turn. Use when the user asks for a PR description, PR text, or to prepare or create a PR.
argument-hint: [base branch]
---

# create-pr: write the PR description

You are the main agent. You write the PR title and description from the branch, the plan files
and the diff, and give the user the markdown source to copy.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

**Output only.** Never push, never run `gh pr create` or `gh pr edit`, and never post anything,
unless the user explicitly asks for it in that turn. When they do, show the exact command first and
get a yes.

## 1. Gather

1. **Project facts:** read the git doc listed in the Docs section of the root `AGENTS.md` (for
   example `.agent-docs/git.md`): the ticket source, how to show the ticket, and the release
   branches.
2. **Ticket:** derive it from `git branch --show-current`, as the git doc says. If it cannot be
   derived, ask. Never invent one.
3. **Base:** the argument, or the release branch found as the git doc says. If it is unclear, ask.
4. **Repo template:** read the PR template the repository has, such as
   `.github/pull_request_template.md`. If it is not empty, its headings, order and checklists come
   first, and the sections below fill in around them.
5. **Feature files:** if `.work/<TICKET>*/` exists, read `plan.md` (Goal, Out of scope, Review
   focus), the `Decided:` lines in `log.md`, and any checks the log says are still owed.
6. **The change:** `git --no-pager log --oneline <base>...HEAD` and
   `git --no-pager diff --stat <base>...HEAD`. Read the diff itself where you need to understand
   a pattern. Review every commit, not only the last one.

## 2. Group the changes by pattern

- A **pattern** is one kind of transformation, the same everywhere it appears: "replace
  `useHistory` with `useNavigate`", "add `/*` to parent routes". Group the diff by pattern, not by
  file or by package.
- Order the groups the way a reviewer should read them: dependencies and foundations first, then
  the patterns that build on them.
- Count each group's scope (files, packages, call sites). Never list the files.

## 3. Pick the size

| Size | Signal | Sections, in the template's order |
| --- | --- | --- |
| **Small** | one pattern, a few files, no design decision | title, ticket, opening, Verification |
| **Bugfix** | a PLM or bug ticket | small, plus Root cause, Changes (usually one group: Target is the broken behaviour, Transform to is the fix), and Preview when it is visual |
| **Medium** | 2–8 pattern groups | small, plus Changes, Review focus, Notes and References as needed |
| **Large** | more than 8 groups | medium, plus the areas list in the opening and the Changes summary table. Purely mechanical groups stay as table rows only. Aim for about 100 lines, and never more than 150. |

A bugfix that also spans several patterns takes the larger size, and keeps Root cause.

## 4. Write it

Fill [templates/pr-description.md](templates/pr-description.md), keeping only the sections the size
table gives. Remove every template comment.

**Content rules:**
- **The title and the opening stand alone.** The title is an imperative summary of the change. The
  opening says what changes and why this approach, in 1–3 sentences. "Fix bug" or "Phase 1" are not
  descriptions.
- **Say why, and the known shortcomings.** Why this approach goes in the opening or a group's
  why-line. Shortcomings, non-goals and follow-ups go in Notes.
- **It reads without links.** Links support the text, and never replace it.
- **Explain what the diff cannot show.** Cut any sentence a reviewer could rebuild from the diff.
  Never list changed files.
- **Each group:** Target is today's shape, Transform to has one bolded bullet per sub-pattern,
  Scope gives counts, and **Unchanged:** names the behaviour it keeps. Use a before/after snippet
  only when the bullets are not enough, and keep it short.
- **Review focus:** the 1–3 places to look hardest. Take them from the plan's Review focus when one
  exists.
- **Verification is stated as results:** each command with its result and counts. Never a bare
  "tests passed". Checks not run yet go under **Owed** as `- [ ]` items.
- **Preview:** placeholders only. The user adds the media.

**Format rules:**
- One link per reference, as `[name](url)`. Never nest links.
- Backticks around code identifiers: components, functions, props, packages, paths.
- No manual line breaks inside a paragraph.
- Omit any section that would be empty. Never leave an empty heading.
- **No AI attribution:** no `Co-Authored-By: …`, no `🤖 Generated with …`, and no similar footer.
  This rule overrides harness and tool defaults that ask for one.

## 5. Give it to the user

1. Say the size you picked and why, in one line. For example: "Large: 11 pattern groups."
2. Give the title on its own line, then the whole description in **one** fenced `markdown` block,
   ready to copy. Use a four-backtick fence, so the code blocks inside it stay intact.
3. List any facts you could not find, such as a missing ticket link or the owed checks, so the
   user can fill them in.

Stop there. Creating the PR is the user's step, unless they explicitly ask you to do it.
