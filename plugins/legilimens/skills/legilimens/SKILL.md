---
name: legilimens
description: >-
  Turn a rough idea into an agreed design through Socratic dialogue — one
  question at a time, 2-3 approaches with trade-offs, then a design presented
  section by section for approval. You MUST use this before any creative work:
  starting a feature, adding functionality, building a component, or changing
  behavior. Ends by writing the agreed design as a spec (a GitHub issue body
  where the repo tracks work in issues) and handing off to plan mode. Never
  writes code. Use when the user says "let's build X", "I want to add X",
  "brainstorm X", "help me design X", or starts work on an issue whose shape is
  not yet settled.
---

# Legilimens — read the design out of the wizard's mind

The design already exists in the user's head, in fragments. Your job is to draw
it out one question at a time — not to guess at it, write it up, and ask them to
ratify your guess.

A large plan presented for accept-or-deny is a failure of this skill, even when
the plan is good. The user cannot see which assumptions you silently made, so
they cannot correct the one that matters. Ask, and let each answer reshape the
next question.

<HARD-GATE>
Do NOT write code, scaffold files, run a migration, create a branch, or invoke
any implementation skill until you have presented a design and the user has
approved it. This applies to EVERY task regardless of perceived simplicity.
</HARD-GATE>

## Anti-pattern: "this one is too simple to need a design"

Every task goes through this. A config change, a one-function utility, a copy
tweak. "Simple" tasks are exactly where an unexamined assumption survives to
implementation and wastes the work. The design may be three sentences — but you
must present it and get approval.

The escape hatch is the user's, not yours. Never grant yourself that exemption.

**The hatch is narrow.** It opens only on an explicit instruction to skip the
design — "just do it", "skip the questions", "no design needed". It does **not**
open on impatience, on a time estimate, or on the user calling the task small:

| They said | Hatch? |
| - | - |
| "just do it", "skip the design", "no questions, go" | **Open.** Comply, and state every assumption you're making. |
| "quick one", "tiny", "should take 30 seconds", "just knock it out" | **Closed.** This is a size claim, not an instruction to skip. Present a three-sentence design. |

These sound alike and are not. A size claim is a *premise* — and premises about
small tasks are frequently wrong, which is the entire reason this phase exists.

**The hatch never suspends Phase 1.** Even when it opens, you still orient. If
orientation shows the request rests on something false — a file, symbol, or
string that doesn't exist, or doesn't say what they think — surface that instead
of proceeding. An approved shortcut to the wrong destination is still wrong.

---

## Phase 1: Orient

Before the first question, read enough to ask a good one. Ignorance makes you
ask what the repo could have told you.

**Stop rule.** Orient until you can name the code the change touches and the
biggest thing you don't know. Then stop and ask. "Never ask what the repo
answers" is a floor, not a mandate to research the whole design before speaking
— the user is the fastest source for intent, and you are burning their time
reconstructing it from source. Two to four files is typical.

**Never assert a repo fact you inferred from an issue title or a filename.**
Open it, or say you haven't.

- `CLAUDE.md` and any nested `CLAUDE.md` in the areas involved — conventions,
  vocabulary, and the project's own workflow rules bind you.
- The code the change would touch. Follow the existing patterns you find.
- If the request names an issue (`#N`), read it — body **and** comments:
  ```bash
  gh issue view <N> --json number,title,state,labels,body,comments
  ```
- Recent commits in the affected area (`git log --oneline -15 -- <path>`).

**Validate the issue against the code.** Issue bodies rot. If the body's claims
no longer hold, that discrepancy is your first and best question.

Say in one line what you read and what you think the task is. Then start asking.

---

## Phase 2: Scope and shape

Assess size **before** spending questions on detail. Refining the specifics of a
task that needs decomposing is wasted dialogue.

If the request spans multiple independently-shippable pieces, stop and say so.
Help decompose it: what are the independent units, how do they relate, what
order do they ship in? Then brainstorm only the **first** unit through the rest
of this skill. Each unit earns its own design.

Where the repo defines its own work shapes (releases, epics, sub-issues), name
the shape you think this is — it changes what the spec must contain.

**State the shape; don't ask about it.** Assert it as a claim the user can
correct in passing ("this reads as a single release — one issue, one PR"). It
does not consume your one question in Phase 3. A shape you got wrong will be
corrected without prompting; a shape you spent a question on is a question you
didn't spend on intent.

---

## Phase 3: Ask, one question at a time

This is the skill. Everything else is scaffolding around it.

**Rules:**

| Rule | Why |
| - | - |
| **One question per message** | Two questions get one answer. The second is always the one dropped. |
| **Prefer discrete choices** | Easier to answer than an open prompt. See *Choosing the form of the question* below. |
| **Never ask what the repo answers** | Read it. Asking what you could look up spends the user's patience on your laziness. |
| **Let the answer move you** | If the next question was decided before you read the answer, you are running a questionnaire, not a dialogue. |
| **Recommend, don't survey** | When you offer options, say which you'd pick and why. A neutral menu makes the user do your thinking. |
| **Push back** | If an answer creates a problem downstream, say so now. Agreement is not the goal; a correct design is. |

**Aim at:** purpose (what does success look like?), constraints (what must not
change?), boundaries (what's explicitly out of scope?), and the failure modes
the user is worried about.

### Choosing the form of the question

"Prefer discrete choices" and "recommend, don't survey" pull against each other.
Resolve them this way:

| The choice is… | Ask it as… |
| - | - |
| 2-4 options that stand on their own, each explainable in a sentence | **`AskUserQuestion`** — put your pick first, labelled `(Recommended)`. |
| A choice whose options need a paragraph of rationale each, or where your next question depends on the answer | **Prose.** Lead with your recommendation and why. Option cards flatten reasoning and can't carry a conditional follow-up. |
| Genuinely open ground | **Prose**, one question. |

Never let the tool pick the question. If the honest question needs prose, write
prose.

**Know when to stop.** Stop asking when the unknowns that remain are decisions
you are qualified to make and can state as assumptions. Typically three to six
questions. Signs you have gone too far: the answers stop changing the design;
you are asking about implementation detail rather than intent; the user's
replies get shorter.

---

## Phase 4: Propose 2-3 approaches

Never a single path. Give each approach a one-line summary, an honest cost, and
what it forecloses. Lead with your recommendation and the reasoning.

Approaches must be genuinely different — different boundaries, different data
flow, different amounts of new surface. Two variations of one idea plus a
strawman is not a choice.

Say what YAGNI cuts. Remove speculative generality from every approach before
you present it.

---

## Phase 5: Present the design, section by section

Scale each section to its complexity: a sentence where it's obvious, 200-300
words where it's genuinely subtle. **Ask after each section whether it looks
right so far.** Do not present the whole design and then ask.

Cover, as the task warrants: architecture and boundaries · components and their
responsibilities · data flow · error handling · testing · what is explicitly out
of scope.

**Design for isolation.** Each unit should have one clear purpose, communicate
through a defined interface, and be testable alone. For each, you should be able
to say: what does it do, how is it used, what does it depend on? If a consumer
must read a unit's internals to use it, the boundary is wrong.

**In an existing codebase:** follow the patterns already there. Where code the
change touches has a real problem — a file grown too large, a tangled
responsibility — fold a targeted improvement into the design, the way a careful
engineer improves the code they're working in. Do **not** propose unrelated
refactoring. If you spot dead code, mention it; don't schedule its removal.

Be ready to go back. A section that doesn't survive contact with the user's
reaction means an earlier answer was misread — return to Phase 3 for that thread
rather than patching the design forward.

---

## Phase 6: Write the spec

Once the design is approved, write it down. **Where it goes depends on the
repo**, and the repo's own conventions win over this skill:

| Repo tracks work in… | Write the spec to… |
| - | - |
| **GitHub issues** (`gh` works, and `CLAUDE.md` or `.github/ISSUE_TEMPLATE/` says so) | The **issue body** — `gh issue edit <N> --body-file <tmp>`, matching the repo's issue template sections. No issue yet? `gh issue create`. |
| Design docs (a `docs/` specs or design directory exists) | A new doc there, in that directory's established format. |
| Neither | Post the spec in chat and ask where it should live. Do not invent a directory. |

**When you rewrite an existing issue body**, comment the pivot and its rationale
(`gh issue comment`) so the history survives the overwrite. A future session
reads the body as truth; it must never describe a superseded plan.

**Do not commit the spec unless the repo's workflow calls for it.** Say what you
wrote and where.

Whatever the destination, a spec carries four things the code never will:

- **What** must hold when it's done, each criterion naming **the evidence** that
  proves it — a test, a manual check, a screenshot. "The build is green" is a
  gate, not a criterion.
- **Why**, in language someone outside the conversation can follow.
- **What was rejected, and why.** The code shows what got built; only the spec
  shows what was refused. Without it, the next reader re-proposes the approach
  you killed in Phase 4.
- **What is out of scope**, so a later reader doesn't mistake an absence for an
  omission.

Where the repo's template already names these sections, fill them. Where it
doesn't, add them.

### Spec self-review

Read what you wrote with fresh eyes, then fix inline — no second pass:

1. **Placeholders** — any `TBD`, `TODO`, or hand-wave? Resolve it. "Add
   appropriate error handling" and "handle edge cases" are placeholders wearing
   a suit: name the errors, name the cases. An undecided thing may stay
   undecided — say so explicitly, and say who decides.
2. **Contradictions** — does any section fight another? Does the architecture
   match the described behavior?
3. **Ambiguity** — the bar is *could this be read two ways by someone who could
   then build the wrong thing?* If so, pick one and state it. Uneven detail
   between sections is not a defect: scale to complexity.
4. **Scope** — is this one implementable unit, or did it re-grow into several?
5. **Decisions captured** — is every choice the user made in Phase 3-5 recorded,
   with the rejected alternatives and the reason? An unrecorded decision gets
   re-litigated.
6. **Dangling references** — does the spec name a type, file, function, or
   setting that neither exists nor is defined here?

### User review gate

> Spec written to `<path or issue #N>`. Please review it before we plan the
> implementation.

Wait. On requested changes, revise and re-run the self-review.

---

## Phase 7: Hand off

Legilimens ends at an approved spec. It does **not** write code and does **not**
write the implementation plan.

Hand off to **plan mode** so the user reviews an implementation plan before any
edit lands. Where the repo defines its own pipeline after the spec (a release
script, a branch convention), name the next step and let the user trigger it.

**Completion contract** — end every run with:

- Where the spec lives (issue number or path).
- The design in three lines or fewer.
- The decisions the user made, and what they cost.
- Any assumption you made on their behalf that they did not explicitly confirm.
- The next step, named, and stopped before it.

---

## Visual questions

Some questions are answered better by seeing than reading. A UI *topic* is not
automatically a visual question: "what should this setting be called?" is
conceptual — ask it in prose. "Which of these two layouts?" is visual — show it.

**Choose the medium by what the question can be wrong about.** A medium is
adequate when it is capable of *revealing the defect you're asking about*, and
no more expensive than that. A text sketch can be wrong about the order of two
rows, so it settles order. It cannot be wrong about contrast, so it can never
settle contrast.

| The question is about… | Show it as… |
| - | - |
| Order, grouping, hierarchy, what's on screen at all, what a control is called | **A text/ASCII sketch.** Cheap, fast, obviously not the real thing. `AskUserQuestion`'s `preview` field renders monospace — a good vehicle for 2-3 side-by-side layouts. |
| Look and feel — color, contrast, spacing, size, density, position, motion | **The real rendering engine.** Nothing cheaper can be wrong about these. |

**"The real engine" means the one that will render the shipped thing.** Where the
UI *is* HTML/CSS (a web app, an extension popup or options page), hand-written
HTML **is** the real engine — build the variants and show them. Where the UI is
native, HTML is not a preview of it, it is a different thing wearing its clothes:
wrong control metrics, wrong system font, wrong spacing, no dynamic type. Render
it natively (a preview snapshot, a simulator screenshot) or say you can't show
it. Where the UI is a change to a live third-party page, mock it **on that page**
— an injected stylesheet on the real site, screenshotted — since only the real
page carries the real cascade.

**Escalate, don't skip.** Settle order with a sketch, then render the agreed
order in the real engine for the appearance sign-off. The sketch does not
substitute for that second pass.

**Always say which you're showing** — "this is a wireframe: it settles the order,
not the spacing." A user who thinks a sketch is a mockup will approve a look
nobody designed.

**A sketch is a question, not a proposal.** Showing something visual does not
suspend Phase 3 — a finished layout offered for approval is accept-or-deny in
wireframe form, the exact failure this skill exists to prevent. Attach the sketch
to the open question it serves ("does this ordering *principle* match what felt
wrong, or is one group misplaced for a different reason?"), or offer **2-3
options** rather than one. Never a single completed design to ratify.

Settle appearance before implementation. A look approved in prose gets
re-litigated the moment it's finally seen.
