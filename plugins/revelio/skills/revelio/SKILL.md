---
name: revelio
description: >-
  Reveal hidden issues in code with a scope-disciplined, multi-lens review.
  Reviews a branch diff, staged changes, specific files/folders, an open GitHub
  PR, or pasted code. Applies Security, Architecture, and Code Quality lenses
  plus technology experts for Swift/SwiftUI/UIKit, Safari web extensions
  (TypeScript/JavaScript), .NET/C#, Python, and Bash — consulting installed
  Apple skills where available. Produces severity-rated findings and a verdict,
  writes them to a local report under ./docs/reviews/, and prints a summary that
  ends with the report path. Use when asked to review code, a branch, or a PR,
  or before opening a GitHub PR.
---

# Revelio — the Revealing Charm for your code

Reveal what's hidden: bugs, vulnerabilities, design problems, and quality issues
in changed code. Pragmatic and scope-disciplined — review the change, flag
pre-existing problems separately, and never block on noise.

This is a **solo, GitHub-oriented, pre-PR review**. It reads code and reports
findings; it does not create PRs, post comments, set approval status, or notify
anyone.

---

## Phase 1: Scope Analysis

Pick the review type from the request:

| Request | Type | What to review |
| - | - | - |
| "review my branch" / "review my changes" | **Branch diff** | `git diff <base>...HEAD` |
| "review PR #N" / a GitHub PR URL | **PR review** | `gh pr diff <N>` (metadata via `gh pr view`) |
| "review staged changes" | **Staged diff** | `git diff --cached` |
| "review these files: …" / "review the src/X folder" | **Files / folder** | Read and review the whole files |
| "review this code" + pasted code | **Ad-hoc** | Review the provided code |

### Branch / staged reviews

Detect the default branch instead of assuming `main`:

```bash
base=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's@^origin/@@')
base=${base:-main}
git diff "$base"...HEAD      # branch diff
git diff --cached            # staged changes
```

Confirm the current branch is the intended target. If HEAD is on the default
branch with an empty diff, stop and ask what to review.

### GitHub PR reviews

```bash
gh pr view <N> --json number,title,headRefName,baseRefName,url,additions,deletions,changedFiles
gh pr diff <N>
```

Review the PR diff. If `gh` is unavailable or unauthenticated, say so and offer
to review the local branch instead.

### Files / folder / ad-hoc

Read the files directly and review the full content (no diff context).

### Review name (for the report filename)

1. GitHub issue number in the branch (e.g. `42-fix-sync`) → use `42`.
2. Otherwise slugify the branch name (e.g. `fix/cache-leak` → `fix-cache-leak`).
3. On the default branch / no branch → ask for a short descriptive name.

---

## Phase 2: Scope Discipline

### Diff-based reviews (branch / staged / PR)

**In scope** — issues in the changed code:

- Lines added or modified, functions containing changes, new imports/dependencies.
- Report **all** severities (CRITICAL → INFO) for awareness.
- The **verdict** is driven by the highest in-scope severity.

**Out of scope** — pre-existing issues in unchanged code:

- Note them for awareness (tech debt). Do **not** let them affect the verdict.

### Files / folder / ad-hoc reviews

Everything provided is in scope. Lead with security and architecture; don't
nitpick style on pre-existing code unless asked.

---

## Phase 3: Review Execution

**Always fan out into independent, parallel reviewers — never review a diff inline
as the primary agent.** Dispatch one sub-agent per lens (and per detected
technology) in a single message with multiple Task calls. **Never** set
`run_in_background: true` (permissions won't work in the background).

Why fan out every time: each sub-agent reviews in a **fresh context** with no
memory of having written the code. In autonomous mode the same session that
authored the change is biased toward "looks correct" and tends to confirm rather
than scrutinize; independent reviewers don't carry that bias, and a dedicated
context per lens forces a deeper, more skeptical pass than one agent juggling
every concern at once. The cost is wall-clock time and tokens — that is expected
and worth it. (The only inline exceptions: a pasted snippet too small to diff, or
when `git`/`gh` is unavailable.)

### Reviewers to dispatch

**Three universal lenses — always dispatch all three:**

- **Security** — injection, auth, secrets, unsafe input handling, data exposure.
- **Architecture** — boundaries, coupling/cohesion, SOLID, correctness, performance.
- **Code Quality** — readability, maintainability, DRY, error handling, tests.

**Technology specialists — one sub-agent per technology present in the diff:**

Match the diff against the signals below and dispatch a specialist for each
technology that appears. In each specialist's prompt, instruct it to **load the
listed skill(s) via the Skill tool first** (the installed Apple skills are the
expert references — consult them, don't rely on memory), then review. If a listed
skill isn't available to the sub-agent, fall back to built-in knowledge of that
framework.

| Detected in the diff | Lens | Consult |
| - | - | - |
| `.swift`; SwiftUI (`View`, `@State`, `@Observable`, `@Binding`); UIKit | **Swift / Apple** | `swiftui-specialist`, `swiftui-whats-new-27`, `uikit-app-modernization` |
| Xcode build settings, entitlements, C/C++/ObjC interop, bounds annotations | **Apple security / C** | `audit-xcode-security-settings`, `c-bounds-safety` |
| Test files (XCTest, Swift Testing, Vitest) | **Test quality** | `test-modernizer` (Swift); general test-design otherwise |
| `.ts` / `.js` / `.html` / `.css`; `browser.*` / `chrome.*`; `manifest*`; content/background/popup scripts; `declarativeNetRequest`; `MutationObserver` | **Safari web extension / Web** | No installed skill — use Claude's web knowledge; if an MDN MCP (`mcp__mdn__*`) is available, use it for Web/WebExtensions API + compatibility. See checklist below. |
| `.cs`; EF Core / EF6; ASP.NET | **.NET / C#** | Claude's built-in .NET knowledge (see checklist below) |
| `.py` | **Python** | Claude's built-in knowledge — typing, error handling, security |
| `.sh` / shell | **Shell** | Quoting, `set -euo pipefail`, error handling, injection |

**Safari web extension checklist** (Safari diverges from Chrome/Firefox in ways
that look like bugs): MV3 background is a **non-persistent script** Safari
suspends aggressively — don't flag a `setInterval` keep-alive as dead code;
native messaging is **connectionless** one-shot request/response; content scripts
run in an **isolated world** (can't hook the page's `history.pushState` — DOM
`MutationObserver` is the navigation signal); prefer `browser.*` promises over
growing the `chrome.*` surface; validate `declarativeNetRequest` rule correctness.
Plus the usual: XSS via `innerHTML`/`document.write`, message-origin validation,
`storage` schema/versioning, and strict TypeScript typing (no implicit `any`).

**.NET / C# checklist**: `async`/`await` correctness (no `async void`, no
sync-over-async deadlocks), `IDisposable`/`using` discipline, DI lifetimes
(captive dependencies), EF tracking vs `AsNoTracking`, N+1 queries, transaction
scope, and nullable-reference-type annotations.

### The sub-agent prompt

Give every dispatched reviewer: the diff (or, for file/folder reviews, the changed
files), its lens or specialty, and these instructions — review only the change
against the Phase 2 scope rules; you did **not** write this code, so review it
skeptically and assume nothing is correct until verified; (specialists only) load
your skill(s) via the Skill tool first; use full file paths from the repository
root; return the findings block and nothing else.

Each sub-agent returns findings in this structure:

```yaml
expert: Security|Architecture|Code Quality|Swift|Web|...
findings:
  - id: "CRITICAL-001"
    severity: CRITICAL|HIGH|MEDIUM|LOW|INFO
    in_scope: true|false
    title: "Brief issue title"
    location: "full/path/from/repo/root.ext:123"
    description: |
      What the issue is and why it matters.
    recommendation: |
      How to fix it, with a code example if helpful.
```

Always use full file paths from the repository root, not abbreviated names.

---

## Phase 4: Consolidation

Collect every sub-agent's findings, then:

1. **Merge duplicates** — the same issue from multiple lenses becomes one finding.
2. **Tag domains** — add `domains: [Security, Swift]` to merged findings.
3. **Keep the highest severity** when lenses disagree.

---

## Phase 5: Finding Validation

Be adversarial before publishing. Validate every CRITICAL / HIGH / MEDIUM:

- Is it actually exploitable / broken, or only theoretical?
- Does full context change the severity? Downgrade an over-harsh call.
- Dismiss false positives with a one-line rationale rather than listing them.

**Never publish a CRITICAL / HIGH / MEDIUM without a second look.**

---

## Severity and verdict

| Level | Criteria | Effect on verdict |
| - | - | - |
| 🔴 CRITICAL | Security holes, data loss, breaking changes | Blocks |
| 🟠 HIGH | Bugs, significant design issues | Blocks |
| 🟡 MEDIUM | Code quality, maintainability | Blocks |
| 🟢 LOW | Style, minor improvements | Optional |
| ℹ️ INFO | Observations | Awareness |

| Condition (in-scope) | Verdict |
| - | - |
| Any CRITICAL | 🚫 BLOCKED |
| Any HIGH or MEDIUM | ⚠️ CHANGES REQUESTED |
| Only LOW / INFO | ✅ APPROVED |

---

## Phase 6: Output

### Write the report

Path: `./docs/reviews/<YYYY-MM-DD>-review-<name>.md` (UTC date, `<name>` from
Phase 1). Use `assets/review-report-template.md`.

- **New file** → write the full report from the template.
- **File exists** (a re-review) → do **not** overwrite. Count existing
  `## Review Round` headings, append the next round before the footer, and update
  the top-level **Verdict** to the latest round's.

This report is a **local artifact**: do **not** `git add`, commit, or push it. If
`docs/reviews/` is not gitignored in the target repo, mention it and offer to add
the line — don't silently edit the consumer's `.gitignore`.

### Finish with a summary (this is the completion contract)

Revelio is often invoked by another agent mid-task, so end every run with a
compact, machine-readable summary in the chat:

- The **verdict** and finding counts by severity (in-scope).
- The top blocking findings (CRITICAL / HIGH / MEDIUM), one line each:
  `SEVERITY · location · title`.
- The **explicit report path** so the caller can re-open the full report, e.g.
  `Full report: ./docs/reviews/2026-06-23-review-42.md`.

Do not commit, push, open or comment on a PR, or notify anyone. Stop here.
