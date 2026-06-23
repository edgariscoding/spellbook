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

## Review targets

- **Branch diff** — `git diff <base>...HEAD` (auto-detects the default branch).
- **GitHub PR** — `gh pr diff <N>` for an open PR.
- **Staged changes** — `git diff --cached`.
- **Files / folder** — reviews whole files.
- **Ad-hoc** — pasted code.

## What it looks at

Three universal lenses on every review — **Security**, **Architecture**, and
**Code Quality** — plus technology experts that activate based on what's in the
diff:

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

- Writes a report to `./docs/reviews/<YYYY-MM-DD>-review-<name>.md`.
- The report is a **local artifact** — not committed or pushed. Add
  `docs/reviews/` to your `.gitignore`.
- Ends with a chat summary (verdict, severity counts, top blocking findings) that
  finishes with the report path, so an autonomous agent gets the result inline and
  can re-open the full report.

## Verdict

| In-scope severity | Verdict |
| - | - |
| Any CRITICAL | 🚫 BLOCKED |
| Any HIGH or MEDIUM | ⚠️ CHANGES REQUESTED |
| Only LOW / INFO | ✅ APPROVED |

Pre-existing issues in unchanged code are noted as tech debt but never affect the
verdict.
