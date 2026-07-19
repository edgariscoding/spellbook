# revelio

> _Revelio_ — the Revealing Charm. Makes the concealed visible.

A scope-disciplined, multi-lens code review for solo development on GitHub. It
reveals the issues hiding in a change — bugs, vulnerabilities, design problems,
and quality gaps — rates them by severity, and renders a verdict, **before** you
open a pull request. It reviews code and reports; it never creates PRs, posts
comments, sets approval status, or notifies anyone.

## Triggers

"review my branch", "review my changes", "review PR #N", "review staged changes",
"review these files", "review this code", or "review before I open a PR".

## Modes

- **Single pass** (default) — one fan-out, one report, one summary. Review-only:
  reports findings and never fixes. This is what the triggers above run.
- **Loop** (opt-in) — review → apply the fixes that hold up → fresh, blind
  re-review → repeat until a round surfaces only LOW/INFO (converged), capped at
  3 rounds (a 4th only with stated justification). Fixes code between rounds.
  Triggered explicitly by **"review loop"**, **"review, fix, and re-review"**, or
  **"loop the review"**.

Each loop round dispatches fresh reviewers that are **blind to prior rounds** —
they see only the current code, never the earlier findings — which is what lets a
later round catch a defect in an earlier round's *fix*. The loop costs 2–3× the
tokens and wall-clock, so single pass stays the default.

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
- Ends with a chat summary that finishes with the report path, so an autonomous
  agent gets the result inline and can re-open the full report. A single pass
  prints the verdict, severity counts, and top blocking findings; a loop prints a
  round-overview table, the C/H/M findings tagged by round and disposition, and a
  stop-reason.

## Verdict

| In-scope severity | Verdict |
| - | - |
| Any CRITICAL | 🚫 BLOCKED |
| Any HIGH or MEDIUM | ⚠️ CHANGES REQUESTED |
| Only LOW / INFO | ✅ APPROVED |

Pre-existing issues in unchanged code are noted as tech debt but never affect the
verdict.
