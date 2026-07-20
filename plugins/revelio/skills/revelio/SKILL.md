---
name: revelio
description: >-
  Reveal hidden issues in code with a scope-disciplined, multi-lens review.
  Reviews a branch diff, staged changes, specific files/folders, an open GitHub
  PR, or pasted code. Applies Security, Architecture, and Code Quality lenses
  plus technology experts for Swift/SwiftUI/UIKit, Safari web extensions
  (TypeScript/JavaScript), .NET/C#, Python, and Bash — consulting installed
  Apple skills where available. Produces severity-rated findings and a verdict,
  applies the fixes that hold up, writes a local report under ./docs/reviews/, and
  always ends by printing two summary tables into the chat — what the review found,
  and what was done about each finding — followed by the report path. Runs a single
  pass or a multi-round review loop, chosen from how critical, complex, and broad
  the change is. Use when asked to review code, a branch, or a PR, or before
  opening a GitHub PR.
---

# Revelio — the Revealing Charm for your code

Reveal what's hidden: bugs, vulnerabilities, design problems, and quality issues
in changed code. Pragmatic and scope-disciplined — review the change, flag
pre-existing problems separately, and never block on noise.

This is a **solo, GitHub-oriented, pre-PR review**. It reads code, applies the
fixes that hold up, and reports findings; it does not create PRs, post comments,
set approval status, or notify anyone.

> **Completion gate — do this before you stop, on every run, even when invoked
> mid-task by another agent.** Print the summary tables directly into the chat and
> end with the report path. **Table 1 (what the review found) always; Table 2 (what
> was done about each finding) whenever fixes were applied.** The review is not
> complete until they are in the conversation — the fan-out returning is not the
> end. Never `git add`, commit, or push the report; it is a local, gitignored
> artifact. Exact table shapes are in Phase 6 and Loop mode.

---

## Choosing the mode

Revelio runs in one of two modes. **Pick the mode from the change itself** — its
criticality, complexity, and breadth — not from a fixed default.

| Mode | What it does |
| - | - |
| **Single pass** | One fan-out, one report. Reviews, then applies the fixes that hold up. |
| **Loop** | Rounds: review → apply the fixes that hold up → fresh, blind re-review → repeat until convergence. Catches defects in the fixes themselves. |

Both modes apply fixes; the loop adds the re-review rounds. See **Applying fixes**.

### Use loop when any of these hold

- **Core / backbone** — auth, persistence / migrations, concurrency / async /
  cancellation / state machines, security-sensitive paths, widely-imported shared
  infrastructure, public API / protocols / interfaces, build / release / CI config.
- **Correctness-critical** — a bug there means data loss, a crash, a security hole,
  or silently wrong behavior.
- **Broad** — ≥2 technology specialists triggered on a non-trivial change, or a
  large diff (≥~8 files or ≥~300 net lines).

### Otherwise use single pass

Changes confined to one surface, small-to-moderate in size, in non-core logic: UI /
layout / copy, docs, comments, tests-only, config value tweaks, dependency bumps —
and the ordinary middle ground of everyday feature work.

**Tie-break:** when genuinely uncertain and the change is not clearly core,
critical, or broad, prefer single pass. Loop costs 2–3× the tokens and wall-clock,
so reserve it for changes that clearly meet a trigger above.

**Overrides and announcement.** An explicit request always wins over the heuristic:
"single pass" / "just one pass" forces single; "review loop", "review, fix, and
re-review", "loop the review" force loop. Before dispatching reviewers, state the
chosen mode and the one-line reason, e.g. `Loop mode — the change reworks the
cancellation path in shared async infrastructure across 6 files.`

Single pass is Phases 1–6 below, run once. The loop wraps those phases: each
round is a fan-out (Phases 3–5) that reads the *current* code in place, and the
orchestrator fixes validated findings between rounds. Loop mechanics are in
**Loop mode** at the end.

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

## Applying fixes

After validation (Phase 5), **apply the fixes that hold up** — in both modes. This
is the same orchestrator role the loop uses between rounds; a single pass simply
does it once and does not re-review.

- Fix only **validated** findings. A finding that did not survive Phase 5 is
  recorded as `disproved`, never as "fixed".
- **Reviewers never fix their own findings** — only the orchestrating session
  applies fixes, which is what keeps the next loop round's reviewers fresh and
  blind.
- Record a **disposition** for every in-scope C/H/M finding, and carry it into the
  report and Table 2:

| Disposition | Meaning |
| - | - |
| `fixed` | Validated and corrected in this run. |
| `deferred` | Real, but intentionally not fixed now — state why in the note. |
| `disproved` | Did not survive Phase 5 validation; not a real defect. |
| `open` | Unaddressed when the run stopped. |

**Review-only override.** If the request is to *only* review — "just review",
"review only", "don't touch my code", "tell me what's wrong" — skip fixing
entirely. Every finding's disposition is `open`, you print **Table 1 only**, and
you say in one line that no fixes were applied because a review-only run was
requested. Fixing is otherwise standard in both modes.

---

## Severity and verdict

| Level | Criteria | Effect on verdict |
| - | - | - |
| 🚨 CRITICAL | Security holes, data loss, breaking changes | Blocks |
| 🔴 HIGH | Bugs, significant design issues | Blocks |
| 🟠 MEDIUM | Code quality, maintainability | Blocks |
| 🟡 LOW | Style, minor improvements | Optional |
| 🔵 INFO | Observations | Awareness |

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

### Finish with the summary tables (single-pass completion contract)

Revelio is often invoked by another agent mid-task, so **every run ends with the
summary tables printed directly in the chat.** This is a hard gate, not a courtesy:
an agent that dispatched reviewers, read their findings, and moved on without
printing these has not finished the review. Print a header line, both tables, then
the report path.

Header line:

```
Code review · single pass · {✅ Approved | ⚠️ Changes requested | 🚫 Blocked}
```

**Table 1 — what the review found.** Every in-scope C/H/M finding, highest severity
first. LOW/INFO collapse to a counts line beneath it.

| Sev | Location | Finding |
| - | - | - |
| 🔴 | `Sources/Sync/RetreatEngine.swift:212` | retreat() step size untested |
| 🟠 | `Sources/Sync/ClampTests.swift:88` | clamp test traps on a negative index |

`🟡 3 low · 🔵 2 info (in report)`

**Table 2 — what was done.** One row per in-scope C/H/M finding, same order,
carrying the disposition recorded in **Applying fixes**.

| Sev | Finding | Disposition | Note |
| - | - | - | - |
| 🔴 | retreat() step size untested | fixed | added hop-2 discriminating test |
| 🟠 | clamp test traps on a negative index | deferred | pre-existing; separate PR |

Omit Table 2 **only** on a review-only run — instead say in one line that no fixes
were applied because review-only was requested.

End with the explicit report path so the caller can re-open the full report:

```
Report: ./docs/reviews/2026-06-23-review-42.md
```

No round-overview table for a single pass. Do not commit, push, open or comment
on a PR, or notify anyone. Stop here.

For loop mode, the completion contract is the **loop summary** in Loop mode below.

---

## Loop mode

Selected per **Choosing the mode**. The invoking agent is the **orchestrator**: it
dispatches a review round, validates and applies the fixes that hold up,
dispatches a fresh round over the fixed code, and repeats until a round stops
surfacing substance. Why loop: the fixes are themselves unreviewed code — a fix
can introduce a false comment, a vacuous test, or a fresh bug — so an independent
second pass catches those and a third confirms convergence.

Each round runs Phases 3–5 (fan-out → consolidate → validate) and appends to the
report (Phase 6). Scope (Phase 1–2) is set once, at the start of the loop.

### How a round reads the code

Reviewers are `Agent`-tool sub-agents that read the branch **in place, from the
same working directory** — no independent checkout. Each round therefore sees the
*current* code, including the previous round's fixes, automatically. (Worktree
isolation is for agents that mutate files in parallel; review is read-only, so it
is not used.)

### Reviewers stay blind to prior rounds

Each round's reviewers receive only the diff (or files) plus the specific files to
read — **never the report, never the prior findings**. Blindness is the point: a
reviewer who has read "R1 fixed X" tends to confirm rather than re-scrutinize, and
inherits the prior round's blind spots. Blind reviewers are exactly what let later
rounds find defects *in the fixes*.

### Narrow scope each round

Later rounds review the **surface changed since the previous round** (the fixes),
not a full re-run of every lens. The fan-out shrinks as the change settles — in
the reference run it went 5 reviewers → 3 → 2.

### The report is the orchestrator's cross-round ledger

The on-disk report (`./docs/reviews/<date>-review-<name>.md`) is the
orchestrator's durable memory of what was found / fixed / deferred / disproved,
and it survives context summarization mid-loop. Between rounds:

1. Read your own report.
2. Inject a curated **"known/deferred — do not re-report"** list into each fresh
   reviewer's prompt, so a deferred finding is not re-flagged as new every round.
   (This — not handing over the report — is what prevents re-litigation while
   keeping reviewers blind.)
3. Append a `## Review Round {N}` section (per Phase 6) and update the top-level
   verdict.

It stays a **local, uncommitted artifact** — never `git add`ed — for the whole
loop (see Phase 6).

### Guardrails

- **No manufactured findings.** Every later-round reviewer's prompt must include:
  "report ONLY what you can substantiate; if the change is sound, say so plainly —
  do not invent findings to appear thorough." Without this, later rounds fabricate
  noise to justify their own existence.
- **Adversarial before fixing.** Validate findings (Phase 5) before applying a
  fix. A finding that does not hold up is recorded as `disproved`, not fixed.
- **Only the orchestrator fixes, and only between rounds.** A reviewer never fixes
  its own findings. Because the next round's reviewers are fresh and blind,
  orchestrator fixes never compromise reviewer independence.

### Convergence and the round cap

- After a round, if it produced any **new in-scope finding at MEDIUM or above** (a
  re-report of a known/deferred item does not count), fix them and run another
  round.
- **Stop** when a round returns only LOW/INFO in scope, or nothing new — converged.
- **Default cap: 3 rounds.** You MAY run a **4th** if round 3 still surfaced
  substantive MEDIUM+ and another round looks productive — but you must state that
  judgment in the stop-reason line. Never silently exceed the cap.

### Loop completion contract — the conversation summary

At the end of the loop, print the same header + two tables as Phase 6, with the
loop's per-round overview leading Table 1. Same hard gate: the loop is not finished
until these are in the chat.

Header line:

```
Code review · {N} rounds · {✅ Approved | ⚠️ Changes requested | 🚫 Blocked}
```

**Table 1 — what the review found.** Lead with the round overview (one row per
round; the Reviewers column lists that round's lenses):

| Round | Reviewers | 🚨 | 🔴 | 🟠 | 🟡 | 🔵 | Verdict |
| - | - | - | - | - | - | - | - |
| R1 | Security · Architecture · Code Quality · Swift · Test Quality | 0 | 1 | 5 | 3 | 2 | ⚠️ Changes requested |
| R2 | Correctness · Code Quality · Test Rigor | 0 | 0 | 1 | 3 | 1 | ⚠️ Changes requested |
| R3 | Comment Accuracy · Test Rigor | 0 | 0 | 0 | 2 | 2 | ✅ Approved |

Then the findings themselves, each tagged with the round it surfaced in, so "a
later round caught a defect in an earlier round's *fix*" is visible at a glance:

| Sev | Round | Location | Finding |
| - | - | - | - |
| 🔴 | R1 | `Sources/Sync/RetreatEngine.swift:212` | retreat() step size untested |
| 🟠 | R2 | `Sources/Sync/ClampTests.swift:88` | docstring claims coverage the test lacks |
| 🟠 | R3 | `Sources/Sync/Index.swift:44` | off-by-one symptom named is unreachable |

`🟡 7 low · 🔵 5 info (in report)`

**Table 2 — what was done.** One row per in-scope C/H/M finding, keeping the round
tag so the fix history stays legible:

| Sev | Round | Finding | Disposition | Note |
| - | - | - | - | - |
| 🔴 | R1 | retreat() step size untested | fixed | added hop-2 discriminating test |
| 🟠 | R2 | docstring claims coverage the test lacks | fixed | rewrote docstring to match |
| 🟠 | R3 | off-by-one symptom named is unreachable | disproved | guard makes the branch dead |

Then the **stop-reason** line and the report path:

```
Stopped after R3: round returned only low/info — converged.
Report: ./docs/reviews/2026-07-19-review-271.md
```

The **stop-reason** line states why the loop ended: "converged" (only LOW/INFO or
nothing new), "hit the 3-round cap", or — for a 4th round — the explicit judgment
that justified taking it.

Do not commit, push, open or comment on a PR, or notify anyone. Stop here.
