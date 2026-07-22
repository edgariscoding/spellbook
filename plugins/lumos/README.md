# lumos

> _Lumos_ — the Wand-Lighting Charm. Lights the way through the work.

Lumos is the **workflow spine**: the generic, project-agnostic method for taking a
change from scope to ship. It sits between the other charms — **legilimens** designs
the change, **revelio** reviews it before it ships, and Lumos is everything around
and between.

It reads everything project-specific (verify commands, board ids, tech-doc source,
release flow) from the repo's own `CLAUDE.md`, so the same skill works in every
repo. It **names** legilimens and revelio at their steps; it does not orchestrate
them.

## Triggers

Starting or resuming a non-trivial change, deciding how to approach a task, or "what's
the right next step here." Skip it for trivial one-line edits.

## The spine

| Step | What happens |
| - | - |
| **1 · Agree scope** | Never auto-start. Settle what's being done and its shape; decompose multi-part work into ordered units and design only the first. |
| **2 · Design** | Reach an agreed spec with **legilimens**, in plan mode. Front-load decisions; no silent guesses. |
| **3 · Implement** | One logical unit per commit, matching the repo's patterns. |
| **4 · Verify green** | Typecheck + lint + tests, 0 errors AND 0 warnings. Extend tests for new logic/fixes. |
| **5 · Review** | Run **revelio** over the diff before a PR; validate findings adversarially. |
| **6 · Document & track** | Issue-body discipline (no placeholders); move the board card. |
| **7 · Ship** | Branch → PR, then the repo's declared release/deploy flow. |

## What Lumos is not

Not a release/deploy pipeline and not a project's build mechanics. A repo's heavy
release machinery is a project extension the repo's `CLAUDE.md` points to, and
Lumos's ship step defers to it.

## Credit

The design→build→review discipline is adapted from the `superpowers` skills by
[Jesse Vincent](https://github.com/obra/superpowers) (MIT). Lumos rewrites it as a
repo-agnostic spine that reads project specifics from a `CLAUDE.md`.
