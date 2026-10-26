# <TICKET> — log

<!--
Append-only changelog for the feature. Newest entry at the BOTTOM. Never rewrite old entries; add a
new entry that corrects them instead. Stage completion lives in plan.md, and stage progress in the
stage file, not here.
Keep each entry short: 3–8 bullets. Write one entry per session or per milestone. An entry's title
is only a label: the lines below record what happened.

Two kinds of line, and only these two record anything:
- `Decided: D-NN <choice> — because <reason>`: every choice, by the user or the main agent.
  IDs run D-01, D-02, … across the whole feature. Briefs and plans cite the ID and never restate
  the reason. A later choice that overrides an earlier one says so: "D-07 replaces D-03".
- `Gate N: <what> — <result>`: the user's approval at gate N (README, "Gates"). It is the only
  record of an approval. A filled table or a `status:` value never counts as one.
-->

## <YYYY-MM-DD> — <what happened, e.g. "Stage 02 reviewed">

- Done: <what changed, link stage file / commit sha>
- Decided: D-<NN> <choice> — because <reason>
- Gate <N>: <what> — <result>
- Blocked: <none | what, waiting on whom>
- Next: <the exact first action for the next session>
