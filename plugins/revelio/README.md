# revelio

> _Revelio_ — the Revealing Charm. Makes the concealed visible.

A scope-disciplined, multi-lens code review for solo development on GitHub. It
reveals the issues hiding in a change — bugs, vulnerabilities, design problems,
and quality gaps — rates them by severity, and renders a verdict, **before** you
open a pull request. It reviews code, applies the fixes that hold up, and reports;
it never creates PRs, posts comments, sets approval status, or notifies anyone.

## Triggers

"review my branch", "review my changes", "review PR #N", "review staged changes",
"review these files", "review this code", or "review before I open a PR".

## Modes

The mode is chosen **from the change itself** — its criticality, complexity, and
breadth — and announced with a one-line reason before reviewers are dispatched.

- **Single pass** — one fan-out, one report. Reviews, then applies the fixes that
  hold up. Used for changes confined to one surface, small-to-moderate in size, in
  non-core logic: UI/layout/copy, docs, tests-only, config tweaks, dependency
  bumps, and ordinary everyday feature work.
- **Loop** — review → apply the fixes that hold up → fresh, blind re-review →
  repeat until a round surfaces only LOW/INFO (converged), capped at 3 rounds (a
  4th only with stated justification). Used when the change is **core** (auth,
  persistence/migrations, concurrency/cancellation/state machines, security paths,
  shared infrastructure, public API, build/CI), **correctness-critical** (a bug
  means data loss, a crash, a security hole, or silently wrong behavior), or
  **broad** (≥2 technology specialists on a non-trivial change, or ≥~8 files /
  ≥~300 net lines).

When a change is borderline and not clearly core, critical, or broad, single pass
wins — the loop costs 2–3× the tokens and wall-clock, so it is reserved for changes
that earn it. An explicit request overrides the heuristic in either direction:
**"single pass"** forces one; **"review loop"**, **"review, fix, and re-review"**,
or **"loop the review"** force the loop.

Each loop round dispatches fresh reviewers that are **blind to prior rounds** —
they see only the current code, never the earlier findings — which is what lets a
later round catch a defect in an earlier round's *fix*.

**Both modes apply fixes.** Reviewers never fix their own findings; only the
orchestrating session does, which is what keeps later rounds' reviewers
independent. Ask to **"just review"** / **"review only"** to skip fixing entirely.

## Review targets

- **Branch diff** — `git diff <base>...HEAD` (auto-detects the default branch).
- **GitHub PR** — `gh pr diff <N>` for an open PR.
- **Staged changes** — `git diff --cached`.
- **Files / folder** — reviews whole files.
- **Ad-hoc** — pasted code.

## What it looks at

Every review **fans out into independent, parallel sub-agents** — one per lens,
plus one per technology found in the diff. Each reviews in a fresh context (no
memory of having written the code, so no self-review bias) and the results are
consolidated, deduplicated, and validated into a single report. Three universal
lenses run on every review — **Security**, **Architecture**, and **Code
Quality** — plus technology experts that activate based on what's in the diff:

| In the diff | Lens | Consults |
| - | - | - |
| Swift / SwiftUI / UIKit | Swift / Apple | `swiftui-specialist`, `swiftui-whats-new-27`, `uikit-app-modernization` |
| Xcode settings, entitlements, C interop | Apple security / C | `audit-xcode-security-settings`, `c-bounds-safety` |
| Tests (XCTest / Swift Testing / Vitest) | Test quality | `test-modernizer` |
| TypeScript/JavaScript, Safari web extension (`browser.*`, `manifest`, content/background/popup) | Safari web extension / Web | Claude's web knowledge + MDN MCP if available |
| C# / .NET / EF | .NET / C# | Claude's built-in .NET knowledge |
| Python | Python | Claude's built-in knowledge |
| Bash / shell | Shell | shell-safety best practices |

The Apple lenses consult the Xcode-bundled skills directly, so the review reflects
current platform guidance rather than memory.

## Output

- Writes a report to `./docs/reviews/<YYYY-MM-DD>-review-<name>.md`. In loop mode
  each round appends a `## Review Round {N}` section to the same report.
- The report is a **local artifact** — not committed or pushed. Add
  `docs/reviews/` to your `.gitignore`.
- Ends with **two summary tables printed into the chat**, so an autonomous agent
  gets the result inline and can re-open the full report:
  - **Table 1 — what the review found:** every in-scope CRITICAL/HIGH/MEDIUM
    finding with its location; LOW/INFO as counts. In loop mode this is led by a
    round-overview table and each finding carries the round it surfaced in.
  - **Table 2 — what was done:** each of those findings marked `fixed`,
    `deferred`, `disproved`, or `open`, with a one-line note. Omitted only on a
    review-only run.
- Both tables are a **completion gate** — the review isn't done until they're in
  the conversation — and the summary ends with the explicit report path.

## Verdict

| In-scope severity | Verdict |
| - | - |
| Any CRITICAL | 🚫 BLOCKED |
| Any HIGH or MEDIUM | ⚠️ CHANGES REQUESTED |
| Only LOW / INFO | ✅ APPROVED |

Pre-existing issues in unchanged code are noted as tech debt but never affect the
verdict.
