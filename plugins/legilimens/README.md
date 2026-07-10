# legilimens

> _Legilimens_ — the Mind-Reading Charm. Draws out what is already there.

The design already exists in your head, in fragments. Legilimens draws it out one
question at a time instead of guessing at it, writing up a large plan, and asking
you to accept or deny it.

A plan presented for accept-or-deny is a failure mode, even when the plan is
good: you can't see which assumptions were silently made, so you can't correct
the one that matters. This skill asks first, and lets each answer reshape the
next question.

It ends at an approved spec and hands off to plan mode. **It never writes code.**

## Triggers

"let's build X", "I want to add X", "brainstorm X", "help me design X", or
starting work on an issue whose shape isn't settled yet.

## The phases

| Phase | What happens |
| - | - |
| **1 · Orient** | Read `CLAUDE.md`, the code the change touches, the issue if one is named. Stop once you can name the touched code and the biggest unknown — two to four files. Never assert a repo fact inferred from a filename. |
| **2 · Scope and shape** | Size it before spending questions on detail. Decompose anything spanning multiple shippable pieces; brainstorm only the first. State the work shape rather than asking about it. |
| **3 · Ask** | **One question per message.** Discrete choices where they stand alone, prose where they need rationale. Never ask what the repo answers. Recommend, don't survey. Push back. Stop at three to six. |
| **4 · Approaches** | 2-3 genuinely different ones — different boundaries, not variations on a theme. Honest costs. What each forecloses. Lead with the recommendation. Say what YAGNI cuts. |
| **5 · Design** | Presented section by section, approval after **each** — not the whole thing then one ask. Scaled to complexity. |
| **6 · Spec** | Written where the repo keeps work: a GitHub issue body, a design doc, or chat. Self-reviewed for placeholders, contradictions, ambiguity, scope, and captured decisions. Then a review gate. |
| **7 · Hand off** | Names the next step and stops before it. |

## The hard gate

No code, no scaffolding, no branch, no implementation skill until a design is
presented and approved. This holds for **every** task, regardless of perceived
simplicity — "simple" tasks are where an unexamined assumption survives to
implementation and wastes the work. The design may be three sentences, but it
gets presented.

The escape hatch is yours, not the agent's, and it's narrow:

| You said | Hatch |
| - | - |
| "just do it", "skip the design", "no questions, go" | **Open** — it complies, and states its assumptions. |
| "quick one", "tiny", "should take 30 seconds", "just knock it out" | **Closed** — a size claim is a premise, not an instruction. |

Even when the hatch opens, orientation still runs. If the request rests on
something false — a symbol or string that doesn't exist, or doesn't say what you
think — that gets surfaced instead of built.

## Where the spec lands

| Repo tracks work in… | Spec goes to… |
| - | - |
| GitHub issues | The issue body, matching the repo's template. Rewrites are accompanied by a comment recording the pivot. |
| Design docs | A doc in the established directory and format. |
| Neither | Chat, with a question about where it should live. |

The repo's own conventions win. Nothing is committed unless the repo's workflow
calls for it.

## Credit

The Socratic core — one question at a time, alternatives before design, sections
approved incrementally, and the hard gate — is adapted from the `brainstorming`
skill in [obra/superpowers](https://github.com/obra/superpowers) by Jesse
Vincent, MIT licensed. Legilimens rewrites it to terminate at a spec in the
repo's own tracker and hand off to plan mode, rather than at a committed
`docs/superpowers/specs/` document and a `writing-plans` skill.
