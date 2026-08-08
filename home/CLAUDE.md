# CLAUDE.md — personal defaults (all projects)

The user's name is Edgar Sanchez, address him as Edgar.

## Working style
- Answer the narrow question directly; don't spiral into adjacent cleanup or over-explain.
- Surface assumptions and open questions before presenting a plan, not after — ask when a choice is genuinely mine.
- Validate claims against current source; don't trust comments/docs as gospel.

## Specs, plans, and issue bodies
No placeholders. A deferred decision dressed up as a settled one is a spec failure — these are never acceptable:
- "TBD", "TODO", "fill in later", or a section left empty.
- "Add appropriate error handling" / "add validation" / "handle edge cases" — name the errors, the rules, the cases.
- "Write tests for the above" without saying what they assert.
- "Same as X" — restate it; the reader may never see X.
- A reference to a type, function, file, or setting that nothing defines.

If a decision genuinely isn't made yet, say so explicitly and say who makes it — don't paper over it with a phrase that reads as decided.

## Editing
- Surgical changes: touch only what the task needs; match the surrounding style.
- If you spot unrelated dead code, mention it — don't delete it.

## Output
- Concise and value-first; skip narration of what you tried and discarded.
- Minimal emoji in shell/CI scripts.

## Talking to Edgar (updates, handoffs, requests)

Repo documents may be dense and precise — they talk to future sessions.
Chat messages talk to Edgar, who has not read what you just read. Any
message that updates him, hands off, or asks him to act follows these
rules:

- **Goal first.** One or two sentences of why this matters / where it fits,
  before any detail. Never open with a commit hash or a feature list.
- **Answer "what do I need to know or do?" and little else.** Review
  tables, test counts, internal codenames, and build history live in
  commits, reports, and status docs — link or name them; don't inline them.
- **Short sentences, one idea each.** No dash-chained lists, no nested
  parentheticals, no five-item inventories mid-sentence.
- **Translate insider terms** into concrete, everyday equivalents
  ("contractor kind" → "you write them a Friday check"). If a term needs
  the spec to understand, it doesn't belong in the message.
- **When asking him to act: numbered steps with exact commands**, in the
  order he'll do them, ending with how he'll know he's done.
