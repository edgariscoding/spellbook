---
name: erecto
description: >-
  The Structure-Erecting Charm for a new repo — run ONCE to stand up a project's
  working setup. Use when setting up a new repo, scaffolding a CLAUDE.md, or
  onboarding a project to the design→build→review workflow ("set up this repo",
  "create a CLAUDE.md for this project", "scaffold the working setup"). It
  interviews you about how the project works, then writes its CLAUDE.md (a fixed
  workflow-invariants block + a filled project-constants block that lumos reads),
  materializes a GitHub issue template, and adds a gitignore entry for review
  reports. Ships no baked-in ids or copy — it asks. Safe by default: it detects
  and never blind-overwrites an existing CLAUDE.md, issue template, or gitignore.
  Not for an already-scaffolded repo — this is a one-time setup, not day-to-day work.
---

# Erecto — the Structure-Erecting Charm for a new repo

Erecto stands up a repo's **working setup** so the rest of the workflow (**lumos**
day to day, **legilimens** to design, **revelio** to review) has what it needs. Run
it **once per repo**. It interviews, then writes files — and only local files; it
never touches external state (the board, plugin installs, your global config);
instead it *reminds* you to run those steps yourself.

> **Run once.** Erecto is setup, not day-to-day work. If a repo already has a
> `CLAUDE.md` written this way, don't re-run Erecto to make a change — edit the
> file, or use **lumos**. Erecto detects an existing setup and offers to update in
> place rather than clobber it.

---

## Step 1 — Interview: gather the project constants

Ask these one topic at a time. **Detect and propose** where you can (read
`package.json`, project files, CI config) so the user confirms rather than types.
**Pre-fill nothing that is personal** — no board ids, no house copy voice; those
are asked, so the skill stays generic.

1. **Identity** — the project name and a one-line "what this is."
2. **Stack** — language/framework(s). Detect from the repo (e.g. `package.json`,
   `*.xcodeproj`, `go.mod`) and confirm.
3. **Verify commands** — the exact typecheck / lint / test commands that must pass
   before a commit (0 errors AND 0 warnings). Detect from the `scripts` block / CI
   and confirm; if a category doesn't exist yet (no linter, no tests), record that
   honestly rather than inventing one.
4. **Release / deploy flow** — how the project ships: nothing (CI auto-deploys on
   merge), a CI pipeline, a manual runbook, an App-Store release, etc. If it points
   to a heavier project extension (a `release/` doc, a deploy script), name it —
   lumos's ship step will defer to it.
5. **Work tracking** — does the project use GitHub issues + a project board? If so,
   ask for the **board number, the project-id, the Status field id and its option
   ids**, and confirm the `gh project item-edit` card-move command shape. Do **not**
   pre-fill these — paste them from an existing repo if you have one.
6. **Tech-doc source** — the documentation source to consult over memory for
   platform/API questions (e.g. an MDN MCP for web/JS, an Apple-docs MCP for Swift,
   or none).
7. **Copy voice / domain terms** — *generic question:* does this project have a
   voice or glossary that should govern its user-facing writing? **No** → write no
   copy section. **Yes** → ask for the source(s) as links or file paths, read them,
   and **distill a self-contained section** into this repo's `CLAUDE.md`. Never
   leave a live cross-repo link — copy the durable rules in, so they can't rot or
   404 when the source moves.

---

## Step 2 — Write the files (local only)

> **Run Step 3's existence checks first.** Never write or overwrite any file below
> before confirming it doesn't already exist (or backing it up) — the safety rules
> in Step 3 gate every write here.

### `CLAUDE.md`

Compose it from three parts, in order:

1. **The invariants block** — paste `assets/invariants-block.md` **verbatim**. It
   is identical in every repo and must not be edited per-project.
2. **A `## Project constants` block** — filled from the interview:

   ```markdown
   ## Project constants

   - **What this is:** <one-line>
   - **Stack:** <language/framework>
   - **Verify (all pass — 0 errors, 0 warnings):** `<typecheck && lint && test>`
     <note any category that doesn't exist yet>
   - **Release / deploy:** <flow; name the extension doc it defers to, if any>
   - **Tech-doc source:** <MDN MCP / Apple-docs MCP / none>
   - **Work tracking:** GitHub issues (from the interview). If the project uses a
     board, add its number and a card-move command; if not, note "no board":
     ```bash
     <gh project item-edit with this repo's board/field/option ids — omit if no board>
     ```
   - **Copy voice / terms:** <distilled, self-contained rules — or "none">
   ```

3. **A `## Project-specific notes` stub** — a short section for anything a repo
   needs that the constants don't cover (release/deploy extensions lumos defers to,
   gotchas, runbooks the repo points to). Seed it with a one-line placeholder-free
   note or leave a single "None yet." line.

### `.github/ISSUE_TEMPLATE/task.md`

Materialize `assets/task.md` into the repo (it drives GitHub's new-issue chooser on
the default branch, and `gh issue create`).

### `.gitignore`

Append `docs/reviews/` if it isn't already ignored — that's where **revelio** writes
its local report, which must never be committed.

---

## Step 3 — Safety: never blind-overwrite

Before writing any file, check whether it exists:

- **`CLAUDE.md` exists** → don't clobber. If it already has the invariants +
  constants shape, update those sections in place. Otherwise, show the user the
  proposed file and offer to back the old one up (`CLAUDE.md.bak`) before replacing.
- **`.github/ISSUE_TEMPLATE/task.md` exists** → leave it; mention the difference.
- **`.gitignore`** → append only if the `docs/reviews/` line is absent; never
  rewrite the file.

---

## Step 4 — Remind (never execute) the external-state steps

Erecto only writes local files. Print these as a copy-paste checklist for the user
to run — do **not** perform them:

- **Global baseline** — is `~/.claude/CLAUDE.md` importing the spellbook baseline
  (`@.../spellbook/home/CLAUDE.md`)? If not, print the one-time install line.
- **Plugins** — are `lumos`, `legilimens`, and `revelio` installed? If not, print
  the `/plugin install <name>@spellbook` lines.
- **Board** — if the project uses a board, print the `gh project item-add` line to
  add this repo's issues to it.

---

## Credit

Part of the same workflow family as **lumos** (the spine), **legilimens** (design),
and **revelio** (review), whose lineage traces to the `superpowers` skills by
[Jesse Vincent](https://github.com/obra/superpowers) (MIT). Erecto is the one-time
setup step that writes the `CLAUDE.md` those skills read.
