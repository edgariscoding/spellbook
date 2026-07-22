# erecto

> _Erecto_ — the Structure-Erecting Charm. Stands up a new repo's working setup.

Run Erecto **once** in a new repo. It interviews you about how the project works,
then writes the working setup the rest of the workflow depends on:

- **`CLAUDE.md`** — a fixed workflow-invariants block (points at **lumos**, and at
  **legilimens** / **revelio** for design and review) plus a filled
  **project-constants** block (stack, verify commands, release/deploy flow,
  work-tracking ids, tech-doc source, optional copy voice) that lumos reads.
- **`.github/ISSUE_TEMPLATE/task.md`** — the issue template (Summary / Technical
  detail / Decisions / Done when / Out of scope / Notes / links), driving GitHub's
  new-issue chooser.
- **`.gitignore`** — a `docs/reviews/` entry for revelio's local reports.

Erecto ships **no baked-in ids or copy** — it asks, so it works in any repo. It only
writes local files; anything external (installing plugins, adding the repo to a
board, wiring the global baseline import) is printed as a checklist for you to run.

## Triggers

Setting up a new repo, scaffolding a `CLAUDE.md`, or onboarding a project to the
workflow. It's a one-time setup — for day-to-day work in an already-scaffolded repo,
use **lumos**.

## Safe by default

Erecto detects an existing `CLAUDE.md`, issue template, or `.gitignore` entry and
updates in place or backs up — it never blind-overwrites.

## Credit

Part of the workflow family (**lumos** · **legilimens** · **revelio**), whose
lineage traces to the `superpowers` skills by
[Jesse Vincent](https://github.com/obra/superpowers) (MIT).
