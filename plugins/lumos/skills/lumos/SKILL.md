---
name: lumos
description: >-
  The Wand-Lighting Charm for your workflow — lights the way through a change from
  scope to ship. Use when starting or resuming a non-trivial change, deciding how
  to approach a task, or asking "what's the right next step here." Walks the spine:
  agree scope and decompose → design with legilimens in plan mode → implement one
  unit per commit → verify green (0 errors AND 0 warnings) → review with revelio
  before a PR → document and track → ship via the repo's declared release flow.
  Names legilimens and revelio at their steps and carries the reusable disciplines
  (issue authoring, board hygiene, comment durability, the resume contract), reading
  project specifics from the repo's CLAUDE.md. Not a release/deploy or build-mechanics
  skill — it defers those to the repo. Skip for trivial one-line edits.
---

# Lumos — the Wand-Lighting Charm for your workflow

Lumos lights the way through a change end to end. It is the **spine** that sits
between the other charms: **legilimens** designs the change, **revelio** reviews it
before it ships, and Lumos is everything around and between — how work is scoped,
built, verified, documented, and shipped. Lumos *names* those two at their steps; it
does not wrap or invoke them — each stays a standalone skill you run yourself.

Lumos is **generic and project-agnostic**. Everything specific to a repo — the
verify commands, the board ids, the tech-doc source, the release/deploy flow —
lives in that repo's `CLAUDE.md` (its *Project constants* block, written by
**erecto**). Lumos reads those; it never hard-codes them.

> **What Lumos is NOT.** It is not a release/deploy pipeline and not a project's
> build mechanics. Where a repo has heavy release machinery (a version-stamping
> step, a release runbook, a deploy script), that lives as a *project extension*
> the repo's `CLAUDE.md` points to, and Lumos's ship step **defers** to it — it
> does not absorb or replace it.

---

## When to run

Reach for Lumos when **starting or resuming a non-trivial change**, when deciding
how to approach a task, or whenever you're unsure what the right next step is.
Skip it for trivial one-line edits (a typo, an obvious tweak) — there's no spine
to walk for those.

---

## The pipeline

Walk these in order. Each step names the gate it must clear before the next.

| # | Step | What it means |
| - | - | - |
| 1 | **Agree scope first** | Never auto-start. Settle *what* is being done and its shape. |
| 2 | **Design** | Turn the idea into an agreed spec with **legilimens**, in plan mode. |
| 3 | **Implement** | One logical unit per commit; follow the patterns already in the repo. |
| 4 | **Verify green** | Typecheck + lint + tests, **0 errors AND 0 warnings**. |
| 5 | **Review** | Run **revelio** over the diff before opening a PR. |
| 6 | **Document & track** | Issue body, board card, commit message. |
| 7 | **Ship** | Branch → PR (or the repo's own merge convention), then its declared release flow. |

### 1 · Agree scope first

Open by agreeing scope — never auto-start on "work on X." Decide the **shape**:

- **One cohesive change** → one unit, one review, one PR. The default.
- **Several independently-shippable pieces** → decompose into ordered units, each
  leaving the project working, and **design only the first** (the rest earn their
  own design when you reach them). Record the order where the work is tracked.

If the request rests on something false — a file, symbol, or setting that doesn't
exist or doesn't say what it's assumed to — surface that before planning.

### 2 · Design with legilimens, in plan mode

For anything non-trivial, use **legilimens** to draw the design out one question at
a time and land an agreed spec, then plan in plan mode. Front-load the decisions;
never bake in a silent guess. legilimens ends at an approved spec and hands off —
that's the boundary between design and build.

### 3 · Implement — one unit per commit

Build the smallest coherent unit, matching the surrounding style. Keep commits
scoped to one logical change. If you spot unrelated dead code, mention it — don't
delete it. Honor any commit-message convention the repo's `CLAUDE.md` records
(e.g. a docs-only / CI-skip prefix).

### 4 · Verify green

Run the repo's verify commands (from its `CLAUDE.md` *Project constants*) —
typecheck, lint, and tests — and require **0 errors AND 0 warnings**. A newly
introduced warning is a regression: fix it before committing. Extend the tests
whenever you add logic or fix a bug (skip only when there is genuinely nothing
unit-testable). Smoke-test the change in the running app when it's exercisable.

### 5 · Review with revelio

Before opening a PR, run **revelio** over the diff (or the built changeset). Be
adversarial about the report, not credulous — validate each finding against the
code, fix the ones that hold up, and note them. revelio's report is a local,
gitignored artifact; never commit or cite it by filename — cite the finding.

### 6 · Document & track

- **Issue authoring** — an issue body carries **Summary / Technical detail /
  Decisions (the road not taken + why) / Done when (each criterion names its
  evidence) / Out of scope / Notes / links**. **No placeholders**: name the errors, the cases,
  the assertions; an open decision may stay open if you say so and say who
  decides. Closed issues are read as prior art, so *Decisions* is what stops a
  future session re-proposing a rejected approach.
- **Board hygiene** — move the issue's card as status changes. The board never
  moves cards on its own (ids + the move command are in the repo's `CLAUDE.md`).

### 7 · Ship

Work on a branch (branch first if you're on the default branch); open a PR, or use
the repo's own merge convention. Then follow the repo's **declared release/deploy
flow** — a project constant. It may be "nothing: CI auto-deploys on merge," or a
heavy runbook the repo points to. Lumos ships by following what's declared; it
does not invent a release process.

---

## Reusable disciplines

These hold across every step, not just one:

- **Code-comment durability.** Comments carry *durable* context — the why, the
  gotcha, the non-obvious. Describe what the code *is*, not what it replaced; a
  comment pointing at a renamed or deleted identifier is dead weight. Never embed
  ephemeral ids (review finding numbers, task/step numbers, tool or report file
  names) — they mean nothing once the review or plan is gone. Cite only durable
  refs (issue numbers, error-tracker ids).
- **Tech-doc lookup.** For platform/API questions, consult the repo's declared
  documentation source (its `CLAUDE.md` names it — e.g. an MDN or Apple-docs MCP)
  rather than recalling from memory.
- **Resume contract.** To pick up work mid-stream, re-orient from the tracker:
  read the issue (body **and** comments), find its branch, check out the
  worktree, read any closed sub-units' handoff notes, and continue at the next
  open unit. The tracker is the source of truth — don't re-derive it.

---

## Credit

The design→build→review discipline is adapted from the `superpowers` skills by
Jesse Vincent (MIT). Lumos rewrites it as a repo-agnostic spine that reads project
specifics from a `CLAUDE.md` and names **legilimens** and **revelio** at the design
and review steps, rather than embedding those steps itself.
