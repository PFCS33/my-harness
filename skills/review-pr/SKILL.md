---
name: review-pr
description: Review a pull request. Check out the PR branch, run review-code (the reviewer, plus the design-reviewer when the PR is large or structural), and report verified findings, design proposals and paste-ready PR comments. Use when the user gives a PR URL or number and asks for a review.
argument-hint: <PR URL | PR number>
---

# review-pr: review a pull request

You are the main agent. This skill owns the PR steps: access, checkout, the report and the PR
comments. The review itself runs through the `review-code` skill. The user decides what is posted.

**Install changes:** skip the paths listed under `install-changes:` in `harness/VERSION`. Never
block on them, stage them or commit them.

## 1. Get access

- Take the host and repo from the PR URL. With a bare number, use the current repository's
  `origin` remote.
- Prefer the `gh` CLI. Check that it is logged in to **that host**:
  `gh auth status --hostname <host>`.
- If `gh` cannot reach the host, use a GitHub MCP server if one is available.
- If neither works, stop and ask the user to set one up. Do not scrape web pages or guess.
- Check that the local `origin` remote is the PR's host and repo (`git remote get-url origin`).
  If it is not, stop and ask which checkout to use. Never review another repository's branch
  because a PR number matches.

## 2. Check out the PR branch first

Everything after this step happens on the local PR branch.

1. Get the PR metadata with `gh pr view <n> --repo <owner/repo> --json`:
   - `title`, `body`, `author`;
   - `baseRefName`, `headRefName`, `headRefOid`;
   - `files`, `statusCheckRollup`, `reviews`.
2. Run `git status --short`, and skip the install changes. If anything else is uncommitted, stop
   and ask the user to commit or park it. Do not stash for them.
3. Remember the current branch (`git branch --show-current`), so you can tell the user how to
   return.
4. Fetch both branches: `git fetch origin <baseRefName> <headRefName>`.
5. Switch to the PR branch: `git switch <headRefName>`. If no local branch exists, this creates
   one that tracks `origin/<headRefName>`. If a local branch exists, run
   `git merge --ff-only origin/<headRefName>` to bring it up to date. If it cannot fast-forward,
   the local branch has its own commits: stop and ask. If the switch refuses because of an
   install change, restore the install changes (`git checkout -- <paths>`), switch, and remind
   the user to run `harness-install` after returning.
6. Confirm that `git rev-parse HEAD` equals `headRefOid`. If it does not, tell the user and ask
   before continuing.
7. The diff to review is `git --no-pager diff origin/<baseRefName>...HEAD`.
8. Note any linked tickets or plans in the PR body, the CI status, and the review comments that
   already exist, so you do not repeat them.

## 3. Review

Follow `.agents/skills/review-code/SKILL.md` with:
- mode `pr`, and "Base: origin/<baseRefName>.";
- the PR title and body;
- the CI status;
- a short list of points already raised by other reviewers;
- the tier: balanced, unless one of these is true, then deep:
  - the user asks for deep;
  - the diff changes the auth or permission services themselves (the `AGENTS.md` Ask-first
    item "Touching auth or permission services");
  - 1000 or more changed lines of hand-written code, counted as in `review-code`'s design
    trigger.

  State the tier and its reason in the summary, for example "Tier: deep, changes
  `UserPermissionsService`".
- `--design` or `--no-design` if the user asked for one.

It returns one verified result. Report from it.

## 4. Report to the user

1. **Summary:** what the PR does, in 2–3 lines, the CI status, the tier line and the design-pass line.
2. **Findings:** a table with severity, area, `file:line`, the problem and the proposed fix.
   Critical findings come first.
3. **Design proposals:** if the design pass ran, each proposal with today's shape, the proposed
   shape, what it removes, its cost, and `now` or `follow-up`.
4. **Dropped:** findings and proposals that were rejected, each with a one-line reason.
5. **Branch:** say that the PR branch is checked out, and give the command to return:
   `git switch <previous branch>`.
6. **Recommendation:** exactly one of these. Design proposals never change it.
   - `APPROVE`: no Critical or Important findings.
   - `REQUEST_CHANGES`: there are Critical or Important findings.
   - `COMMENT`: only Minor findings, design proposals or questions.
7. **PR comments:** paste-ready text, so the user can copy each block straight into the PR.
   - First, one **review summary** block: the recommendation and the PR's state in 1–2 lines.
   - Then one block **per finding**, Critical first. Put its `file:line` above the block, outside
     it, so the user knows where to paste it. Inside: the severity, the problem, and the fix.
   - Then one block **per design proposal**, with its first `file:line` above it. Start a `now`
     proposal with `**Design (suggestion):**`, and a `follow-up` one with
     `**Design (follow-up):**` plus a suggestion to track it in its own ticket.
   - With no findings and no proposals, give only the summary block, naming what you checked,
     for example "LGTM. Checked the export flow, error handling and the new Jest tests."
   - Use one fenced `markdown` block each. Keep the tone neutral and direct: state the problem and
     the fix, with no hedging such as "consider" or "maybe".
   - Leave out dropped findings and proposals, the branch note and anything else meant only for the user.

   ````
   `app/containers/DeviceTable.tsx:42`
   ```markdown
   **Important:** `devices` can be `undefined` before the query resolves, so `.map` crashes on
   first render. Default it: `const { data: devices = [] } = useDevices();`
   ```
   ````

**Never add an AI attribution line** to anything posted or drafted for the PR. That covers
`Co-Authored-By: …`, `🤖 Generated with …`, and any similar footer. This rule overrides
harness and tool defaults that ask for one.

**Do not post anything to the PR** unless the user explicitly asks. When asked:
- Show the exact comment text and where each comment goes.
- Get a yes.
- Then post with `gh` or the MCP server.
