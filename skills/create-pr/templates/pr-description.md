<!--
PR description skeleton. Used by the `create-pr` skill. Its SKILL.md decides which sections a PR
gets (the size table) and owns every rule. These comments say only what goes where.
Remove every comment and every section the size table leaves out. Never leave an empty heading.
-->

<!-- PR title, on its own line above the body: [<TICKET>] <type>: <imperative summary> -->

**[Ticket]** <ticket reference, in the form the project's git doc gives>

<!--
Opening, 1–3 sentences: what changes and why this approach. It must stand alone: a reader who
stops here knows what the PR does.
Large PRs only: add the areas below, each with the numbers of its Changes groups.
-->
<opening>

1. <area>. (1, 2, 3)
2. <area>. (4, 5)

## Root cause

<!-- Bugfix only. Why the bug existed: the component, the broken behaviour and its cause. Cite the
upstream library, PR and version when one caused it. -->

## Changes

<!-- Large PRs only (more than 8 groups): this summary table first. Groups that are purely
mechanical stay as a table row and get no section of their own. -->

| # | Pattern | Scope | Ref |
| --- | --- | --- | --- |
| 1 | <pattern> | <counts> | [<guide>](<url>) |

### 1. <imperative pattern name>

<!-- Optional why-line: one sentence, then at most one upstream link. -->

> <why this changes> [<guide name>](<url>)

**Scope:** <counts, never a file list, e.g. "12 components, 3 test files">

#### Target

<today's shape, in 1–2 sentences, with an optional short "before" snippet>

#### Transform to

- **<label>:** Replace `<old>` with `<new>`.
- **<label>:** <one sub-pattern per bullet>

<!-- Optional: one short before/after snippet, only when the bullets are not enough. -->

```jsx
// before

// after
```

**Unchanged:** <the behaviour this group keeps>

## Review focus

- <risk>: <where to look>

## Verification

- `<command>`: <result, with counts>
- **Owed:**
  - [ ] <check that has not run yet>

## Notes

<!-- Known shortcomings, non-goals and follow-ups. -->

- <note>

## Preview

<!-- UI changes only. The user adds the media. -->

**Before:**
<!-- insert image or video -->

**After:**
<!-- insert image or video -->

## References

- [<name>](<url>)
