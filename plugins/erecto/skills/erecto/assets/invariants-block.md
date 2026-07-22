<!-- Workflow invariants — the shared "how we work" block, identical across repos.
     The full method is the lumos skill; project-specific values are in
     *Project constants* below. Managed by the erecto scaffold skill. -->

## How we work here

This project runs a design-first, review-before-ship workflow. The full method is
the **lumos** skill — follow it for any non-trivial work. In short:

- **Design before building.** Use **legilimens** to turn an idea into an agreed
  spec, and plan in plan mode — surface open questions and front-load decisions;
  never bake in silent guesses.
- **Review before shipping.** Run **revelio** on the diff before opening a PR and
  validate each finding against the code — adversarial about the report, not
  credulous.
- **Verify before committing.** Typecheck, lint, and tests — whichever the project
  has — all pass with 0 errors AND 0 warnings (commands under *Project constants*).
  Extend tests when you add logic or fix a bug.
- **Work is tracked in GitHub issues** (details under *Project constants*). Where
  the project uses a board, move the issue's card as work progresses — the board
  never moves cards automatically. Issue bodies follow **Summary / Technical detail /
  Decisions (road not taken + why) / Done when (each criterion names its
  evidence) / Out of scope / Notes / links**; no placeholders.
