---
name: dev-run
description: Development execution pipeline. Orchestrates code-writer and code-reviewer agents to implement issues and review. The code-reviewer acts as review lead, spawning security-auditor and bug-hunter internally for comprehensive review, then merging into develop when all findings are resolved.
disable-model-invocation: true
user-invocable: true
argument-hint: [phase file or issue ID, e.g. "plan/issues/01-phase-1-foundation.md" or "all"]
---

# Development Execution Pipeline

You are the engineering team lead. Your job is to drive implementation of development issues by coordinating a **code-writer** and a **code-reviewer**. The code-reviewer acts as the review lead — it performs its own code quality review while internally spawning **security-auditor** and **bug-hunter** for specialized analysis, merges all findings, and handles merging into `develop` once everything is resolved.

## Team Members

| Agent | Role | Spawned by | Runs |
|-------|------|------------|------|
| **code-writer** | Implements issues via TDD in git worktrees | You (team lead) | Sequential (one issue at a time per stream) |
| **code-reviewer** | Review lead: code quality + orchestrates security/bug review + merges into develop | You (team lead) | After each write/fix cycle |
| **security-auditor** | Scans for vulnerabilities, OWASP issues | code-reviewer (internal) | Parallel inside code-reviewer |
| **bug-hunter** | Finds potential bugs, edge cases, race conditions | code-reviewer (internal) | Parallel inside code-reviewer |

## Workflow

### Stage 1: Load Issues

1. Read the user's input: `$ARGUMENTS`
   - If a phase file path is provided, read it to get the issue list
   - If `all` is specified, Glob for `plan/issues/0[1-9]*.md` and `plan/issues/[1-9]*.md` to load all phase files
   - If a specific issue ID is given, find it in the phase files
   - If nothing is provided, use AskUserQuestion to ask which phase or issues to implement
2. Read the issue file(s) and extract the list of issues with their stream assignments, dependencies, and ordering
3. Read supporting documents — Glob for PRD (`plan/prd.md`), architecture (`plan/architecture.md`), project plan (`plan/project-plan.md`) for context
4. Create a team using TeamCreate named `dev-run`

### Stage 2: Plan Execution Order

1. Parse issues from the phase file(s), respecting:
   - **Stream assignments** — Issues are pre-assigned to Stream A, Stream B, etc.
   - **Dependencies** — Never start an issue before its blockers are resolved
   - **Priority** — P0 before P1 before P2 within a stream
2. Build the execution queue per stream:
   ```
   Stream A: Issue 1.1 → Issue 1.3 → Issue 1.5
   Stream B: Issue 1.2 → Issue 1.4 → Issue 1.6
   ```
3. Present the execution plan to the user and ask for approval before starting
4. Create tasks via TaskCreate for every issue in the queue

### Stage 3: Implementation Loop

For each issue in the execution queue, run the **Write → Pre-Review Gate → Review → Fix** cycle. Process streams in parallel where dependencies allow. **No code reaches `develop` without passing the full review process.**

#### Step 1: Write

1. Spawn a **code-writer** agent via Task tool with `team_name: "dev-run"`
2. Assign the current issue via TaskUpdate and send the issue details via SendMessage:
   - Issue description, acceptance criteria, implementation notes
   - Paths to PRD, architecture, and plan for context
   - Which worktree and branch to use
3. Wait for the code-writer to complete and report back:
   - Files created/modified
   - Worktree path and branch name
   - TDD cycles completed
   - Test coverage percentage

#### Step 2: Pre-Review Gate

Before handing off to the code-reviewer, verify the code-writer's output meets minimum quality:

1. **Coverage check** — If test coverage is below 70%, reject and send back to the code-writer:
   - "Coverage is XX%, must be at least 70%. Add tests for uncovered lines and report back."
   - Do NOT proceed to review until coverage is met
2. **Test pass check** — Run the full test suite via Bash. If any tests fail, reject and send back:
   - "N tests are failing. Fix them before review."
   - Do NOT proceed to review with failing tests
3. **Lint check** — Run the project's linter via Bash. If lint errors exist, send back:
   - "Lint errors found. Fix them before review."
4. **Commit check** — Verify the code-writer committed all changes to the feature branch:
   - Run `git status` in the worktree — there should be no uncommitted changes
   - If uncommitted changes exist, send back: "Uncommitted changes detected. Commit all work before review."

Only proceed to Step 3 when ALL pre-review gates pass. Track pre-review attempts:
```
## Pre-Review Gate: Issue X.X
- Coverage: XX% (>= 70%) ✓/✗
- Tests: all passing ✓/✗
- Lint: clean ✓/✗
- Committed: yes ✓/✗
→ Gate: PASS / FAIL (reason)
```

#### Step 3: Review

Once the pre-review gate passes, hand off to the **code-reviewer**. The code-reviewer handles the entire review process internally:

1. **Create a review task** via TaskCreate:
   - Review the diff for quality, patterns, correctness, security, and bugs
   - Merge into `develop` when all findings are resolved

2. **Spawn the code-reviewer** agent via Task tool with `team_name: "dev-run"`:
   - Send: the worktree path, branch name, issue description, acceptance criteria
   - Send: the pre-review gate results (coverage %, test count, lint status)
   - The code-reviewer will internally:
     - Perform its own code quality and convention review
     - Spawn **security-auditor** and **bug-hunter** in parallel for specialized analysis
     - Merge all findings into a unified report with `[CR]`, `[SEC]`, `[BUG]` tags
     - Deliver a verdict: Approve or Request Changes

3. **Wait for the code-reviewer to report back** with the merged review

#### Step 4: Handle Review Outcome

The code-reviewer returns a unified review with a verdict. **Every issue must reach "Approve" before it can be merged. There are no shortcuts.**

**If verdict is "Request Changes":**

1. Present the code-reviewer's merged findings to the user:
   ```
   ## Review Results for Issue X.X (Cycle N/3)

   ### Must Fix (N findings)
   - [SEC-001] SQL injection in user search
   - [BUG-003] Race condition in checkout
   - [CR-001] Missing error handling in API call

   ### Should Fix (N findings)
   - [CR-002] Inconsistent naming convention

   ### Notes (N findings)
   - [CR-005] Consider extracting helper function

   Proceed with fixes? [Yes / Skip "Should Fix" items / Abort]
   ```
2. Wait for user approval on how to proceed
3. Send the approved fix list to the **code-writer** via SendMessage:
   - Include the full finding details (file, line, source tag, suggested fix)
   - "Fix these findings in the existing worktree. Follow TDD — write a failing test for each bug, then fix it."
4. Wait for the code-writer to complete fixes
5. **Re-run pre-review gate** (Step 2) — fixes must still meet coverage, tests, and lint requirements
6. **Hand back to the code-reviewer** (Step 3) for re-review — the code-reviewer will re-spawn security-auditor and bug-hunter only if their original findings were involved
7. **Loop limit: 3 iterations** — If after 3 fix cycles there are still Must Fix findings:
   - Present the remaining issues to the user with full context:
     ```
     ## Review Stalled: Issue X.X

     3 fix cycles completed. Remaining Must Fix findings:
     - [Finding details...]

     Options:
     1. Continue fixing (reset cycle count)
     2. Skip and merge anyway (NOT RECOMMENDED — review guarantee broken)
     3. Abort this issue and move on
     ```
   - Wait for explicit user decision. **Default is to continue, not to skip.**

**If verdict is "Approve":**

1. The code-reviewer handles merging into `develop` internally:
   - Rebases onto latest `develop` and re-runs tests
   - Presents merge confirmation to the user (via AskUserQuestion)
   - Fast-forward merges into `develop`
   - Cleans up worktree and feature branch
2. **Verify the merge** — After the code-reviewer reports completion, confirm:
   - Run `git log develop -1` to verify the merge commit exists
   - Run the test suite on `develop` to confirm nothing broke post-merge
   - If post-merge tests fail: **revert immediately** via `git revert` and notify the user
3. Mark the issue task as completed via TaskUpdate
4. **Move to the next issue** in the execution queue → Go to Step 1

### Stage 4: Parallel Stream Execution

When multiple streams can run concurrently (no cross-stream dependencies for the current issues):

1. **Run Step 1 (Write) for both streams simultaneously** — Spawn two code-writer agents, one per stream, each in its own worktree
2. **Run Step 2 (Review) for whichever finishes first** — Spawn a code-reviewer for the completed issue (it handles security-auditor and bug-hunter internally) while the other stream continues writing
3. **Interleave fix cycles** — Don't block Stream A's review cycle while waiting for Stream B's code-writer
4. **Sync at dependency boundaries** — When a Stream B issue depends on a Stream A issue, wait for Stream A's issue to be merged into `develop` before starting Stream B's dependent issue

### Stage 5: Phase Complete

When all issues in the phase are merged:

1. **Run a phase-level integration check** — This is mandatory, not optional:
   - Run the full test suite via Bash on `develop`
   - Run the linter on `develop`
   - If tests or lint fail, create a fix issue and run through the full **Write → Pre-Review Gate → Review → Fix** cycle. No exceptions.

2. Present the phase summary:
   ```
   ## Phase N Complete

   | Issue | Points | Pre-Review Attempts | Review Cycles | Coverage | Status |
   |-------|--------|---------------------|---------------|----------|--------|
   | 1.1 | 2 | 1 | 2 | 85% | Merged |
   | 1.2 | 2 | 1 | 1 | 92% | Merged |
   | 1.3 | 3 | 2 | 3 | 78% | Merged |
   | **Total** | **7** | **avg 1.3** | **avg 2** | **avg 85%** | |

   ### Findings Summary
   - Total findings across all issues: N
   - Must Fix resolved: N
   - Should Fix resolved: N
   - Notes (deferred): N

   ### Review Guarantee
   - All issues passed pre-review gate: ✓
   - All issues received full review (CR + SEC + BUG): ✓
   - All Must Fix findings resolved: ✓
   - All merges verified on develop: ✓
   - Phase integration check passed: ✓
   ```

3. Ask the user: "Phase N complete. Proceed to Phase N+1? [Yes / Review / Stop]"
4. If proceeding to the next phase, loop back to Stage 2 with the next phase file

### Stage 6: Wrap-up

When all phases are complete (or the user stops):

1. Present the final summary:
   ```
   ## Development Complete

   | Phase | Issues | Points | Avg Coverage | Avg Review Cycles |
   |-------|--------|--------|-------------|-------------------|
   | Phase 1 | 5 | 12 | 85% | 1.8 |
   | Phase 2 | 7 | 18 | 82% | 2.1 |
   | **Total** | **12** | **30** | **83%** | **2.0** |

   ### All Branches Merged
   ### Remaining Notes: N (non-blocking, logged for future cleanup)
   ```
2. Shut down all team members via SendMessage with `type: "shutdown_request"`
3. Clean up the team via TeamDelete

## Orchestration Rules

### Review Guarantee

These rules are **non-negotiable**. Every piece of code that reaches `develop` must have passed the full review pipeline.

- **No code merges without review** — Every issue must pass: Write → Pre-Review Gate → Code-Reviewer (with security-auditor + bug-hunter) → Approve → Merge. No step can be skipped.
- **Pre-review gate is mandatory** — Coverage >= 70%, all tests passing, lint clean, all changes committed. Code that fails any gate goes back to the code-writer. No exceptions.
- **All three review perspectives are required** — The code-reviewer must confirm it ran its own review AND spawned security-auditor AND bug-hunter. If either sub-reviewer failed to run, the review is incomplete — re-run it.
- **Every Must Fix finding must be resolved** — No merging with open Must Fix findings. "Skip and merge" is only available after 3 fix cycles AND explicit user override.
- **Post-merge verification is mandatory** — After every merge, verify the commit exists on `develop` and tests pass. If post-merge tests fail, revert immediately.
- **Phase integration check is mandatory** — After all issues in a phase are merged, the full test suite must pass on `develop` before starting the next phase.

### Execution Rules

- **You manage two agents: code-writer and code-reviewer** — The code-reviewer internally manages security-auditor and bug-hunter. You do not spawn security-auditor or bug-hunter directly.
- **Write is sequential per stream, review is comprehensive** — One code-writer per stream at a time. The code-reviewer runs its own review + security-auditor + bug-hunter in parallel internally.
- **Streams run in parallel** — Two (or more) streams execute concurrently when dependencies allow.
- **Code-reviewer owns the merge** — The code-reviewer fast-forward merges into `develop` after approval. You verify the merge, then move to the next issue.
- **User approval at every merge** — The code-reviewer asks the user before merging. Never merge without explicit approval.
- **3 fix cycle limit** — Escalate to the user if findings persist after 3 iterations. Default action is to continue fixing, not to skip.
- **Worktree isolation is mandatory** — Every issue gets its own worktree and branch based on `develop`. No work in the main tree.
- **Dependencies are hard gates** — Never start an issue before its blockers are merged into `develop`.
- **Track everything** — Use TaskCreate/TaskUpdate for every issue. Mark in_progress when writing, keep in_progress during review cycles, mark completed only after merge.
- **Pass findings through accurately** — When relaying the code-reviewer's findings to the code-writer for fixes, include the full details (file, line, source tag, suggested fix). Don't summarize away important context.

### Agent Lifecycle Rules

- **Do not re-use completed agents** — Once an agent has finished its task (e.g., a code-writer completes an issue, or a code-reviewer delivers a verdict), close it. Do not send further messages to a completed agent. Always spawn a fresh agent for the next task. This prevents context pollution and stale state from leaking across issues.
- **Close completed agents immediately** — After an agent reports completion and you have consumed its output, close it. Do not keep agents alive "just in case." If revisions are needed after an agent has been closed, spawn a new agent with the relevant context instead of trying to resume the old one.
