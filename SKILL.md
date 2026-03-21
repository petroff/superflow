---
name: superflow
description: "Use when user says 'superflow', 'суперфлоу', or asks for full dev workflow. Two phases: (1) collaborative Product Discovery with multi-expert brainstorming, (2) fully autonomous execution with PR-per-sprint, git worktrees, Codex reviews, max parallelism, and verification discipline."
---

# SuperFlow — Product-to-Production Workflow

Two distinct phases: collaborative discovery, then autonomous execution.

```
Phase 1: COLLABORATIVE (with user)
  Product Discovery → Multi-Expert Brainstorming → Spec → Plan

Phase 2: AUTONOMOUS (no stops until done)
  Sprint 1 (worktree) → PR #1
  Sprint 2 (worktree) → PR #2
  ...
  Sprint N (worktree) → PR #N
  → Report to user: "Done. N PRs ready for review."
```

---

## CRITICAL RULES

### Rule 1: NEVER pause during autonomous execution
After user says "go" / "ok" / "давай" on the approved plan — ZERO stops, ZERO questions, ZERO "should I continue?". Execute all sprints, create all PRs, report when fully done.

### Rule 2: Use Codex when available
At startup, detect if Codex CLI is installed. If yes — use it for all parallel reviews (spec, plan, code quality, product acceptance). If not — proceed with Claude-only, no warnings. Codex is a bonus for dual-provider coverage, not a hard requirement.

### Rule 3: PR per sprint
Each sprint/logical chunk = separate git branch + PR. Never accumulate 20 commits in one PR. Smaller PRs are easier to review, safer to merge, and can be deployed independently.

### Rule 4: Maximum parallelism
Always dispatch the maximum number of independent agents simultaneously. If 5 tasks are independent — launch 5 agents. Never serialize independent work.

### Rule 5: Proactive product thinking
Don't just ask questions — PROPOSE ideas. "Have you considered X?" is better than "What do you want?". Research best practices, suggest features from analogous products, challenge assumptions.

### Rule 6: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
Before marking ANY task complete: run tests, read output, THEN claim done.
Before creating ANY PR: run full test suite, verify clean output.

**Red flags that indicate rule violation:**
- "should work" — NOT evidence
- "probably passes" — NOT evidence
- "seems fine" — NOT evidence
- "I believe this is correct" — NOT evidence

**Evidence** = actual command output showing pass/success, pasted in full.

---

## Rationalization Prevention

Agents rationalize skipping quality steps. Recognize and reject these:

| Rationalization | Reality |
|---|---|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing about your code. |
| "Skip review, it's trivial" | Trivial changes break production. Review anyway. |
| "One big PR is fine" | 20-commit PRs don't get reviewed. Split into sprints. |
| "Codex isn't needed" | If available, use it. Different models catch different bugs. |
| "Let me ask the user" | After plan approval, execute silently. |
| "This task is too hard" | Escalate with BLOCKED, don't silently produce bad work. |
| "Tests are green so it works" | Green tests prove tests pass, not that the feature works. Verify behavior. |

---

## Mode Detection

At the start, detect which mode applies based on context:

**Enhancement mode** (existing codebase):
- Deeper codebase exploration: read key files, understand patterns, find reusable code
- Step 1 (Context Exploration) is heavier — understand what exists before proposing changes
- Brainstorming references existing architecture: "The current data model already has X, we can extend it"
- Plan preserves backward compatibility

**Greenfield mode** (new project / new major subsystem):
- Focus on stack selection, project structure, CI/CD setup
- Research phase includes framework comparison, starter templates, infrastructure options
- Brainstorming is more open-ended — no existing patterns to follow
- First sprint often includes project scaffolding + CI/CD

The mode is auto-detected from context (existing code = enhancement, empty repo = greenfield). No need to ask the user.

---

## Phase 1: Product Discovery (COLLABORATIVE)

This phase is a conversation with the user. Take your time here — rushing discovery leads to building the wrong thing.

### Step 1: Context Exploration

Before any questions, silently explore:
- Read CLAUDE.md, project docs, recent git history
- Understand current architecture, data model, existing features
- Identify gaps between what exists and what the user is describing

### Step 2: Best Practices Research

Before brainstorming, launch parallel research agents to gather external context:

**What to research (dispatch as background agents):**
- Best practices in the domain (e.g., "financial analytics dashboard best practices")
- Existing libraries/packages that solve parts of the problem (don't reinvent wheels)
- MCP servers that could provide useful integrations
- How analogous products solve the same problem (competitors, open-source alternatives)
- Relevant design patterns, data models, algorithms

**Format:** Each research agent returns a brief summary. Orchestrator synthesizes into context for brainstorming. Share key findings with user.

**This is NOT optional.** Even for seemingly simple features, 5 minutes of research prevents hours of reinventing.

### Step 3: Multi-Expert Brainstorming

**Format:** Freeflow conversation with product focus. No rigid checklist — adapt to the task context. But ensure sufficient depth before moving to design.

**Product vision focus (most important):**
- Understand the WHY and FOR WHOM before the WHAT and HOW
- Pull the user's vision: what does "done" look like? What experience do they want?
- Ask for references: apps, screenshots, examples, competitors — anything that shows intent
- Understand scope ambition: personal tool vs service for others? MVP vs long-term?

**Rhythm: questions -> proposals -> questions**
Don't just ask. After gathering enough context (3-5 questions), make concrete product proposals based on what you've heard + research findings. Let the user react — "yes / no / interesting but..." — this generates new requirements you wouldn't have found by asking.

Example cycle:
1. Ask 3-4 questions to understand the vision
2. Present 2-3 product ideas/suggestions based on answers + research
3. User reacts -> new requirements emerge
4. Ask 1-2 follow-up questions on the new requirements
5. Repeat until the picture is clear

**Depth calibration:** The number of cycles depends on task complexity. A simple feature may need 1 cycle (5 min). A major system overhaul may need 3-4 cycles (20 min). The sign that you're done: you can describe back to the user what they want to build, and they say "yes, exactly."

**Three expert lenses** (weave into the freeflow, not separate blocks):
- **Product lens:** "Users of similar apps typically expect X — should we include that?"
- **Architecture lens:** "The current data model already supports X, so we can leverage that"
- **Domain lens:** "In [domain] apps, best practice for X is Y"

**One question at a time.** Combine questions with insights when natural, but don't overload.

### Step 4: Approaches + Recommendation

Propose 2-3 approaches with trade-offs. Lead with your recommendation and explain why. Be opinionated.

### Step 5: Design Presentation

Present design section by section. Scale depth to complexity.

### Step 6: Spec Document

Write spec to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`.

### Step 7: Spec Review (Claude + Codex PARALLEL)

Launch BOTH in parallel. Use prompt template from `prompts/spec-reviewer.md`.

**Calibration:** Only flag issues that would cause real problems during planning or implementation. Check: Completeness, Consistency, Clarity, Scope, YAGNI. Do NOT flag style or formatting issues in specs.

Fix issues from both reviewers. Re-review if NEEDS_REVISION.

### Step 8: Implementation Plan

Write plan to `docs/superpowers/plans/YYYY-MM-DD-<topic>.md`.

**Plan structure:**
- Break into sprints (logical chunks, each deployable independently)
- Each sprint = 3-8 tasks

**Bite-sized task granularity (CRITICAL):**
Each step = 2-5 minutes of work. Break tasks down to atomic operations:
- "Write the failing test for X behavior" — one step
- "Run it to verify failure" — one step
- "Implement minimal code to pass" — one step
- "Run tests to verify pass" — one step
- "Commit" — one step

A medium feature plan should have 20-30+ steps, NOT 5-10 large tasks. If a step takes more than 5 minutes, it's too big — split it.

**Each task has:** files, steps, code, tests, commit message
**Tasks within a sprint can be parallelized where independent**

### Step 9: Plan Review (Claude + Codex PARALLEL)

Same pattern as spec review. Both must APPROVE.

### Step 10: User Approval

Present the plan summary to user. This is the LAST interaction point before autonomous execution. Show:
- Sprint breakdown with scope
- Estimated PR count
- "After approval, I'll execute all sprints autonomously and report when done."

User says "go" -> Phase 2 begins. No more questions.

---

## Phase 2: Autonomous Execution (ZERO INTERACTION)

After user approves, execute continuously until all sprints are complete. Never ask, never pause, never summarize between sprints.

### Per-Sprint Flow

```
For each Sprint N:
|
+-- 1. Create git worktree for isolation:
|      git worktree add .worktrees/sprint-N feat/<feature>-sprint-N
|      (Verify .worktrees/ is in .gitignore — add if missing)
|
+-- 2. Run baseline tests in worktree before starting work
|      (If baseline fails, note pre-existing failures and continue)
|
+-- 3. For each task (maximize parallel agents):
|      +-- Dispatch implementer (see prompts/implementer.md)
|      +-- On completion: spec review (see prompts/spec-reviewer.md)
|      +-- On spec pass: code quality review (see prompts/code-quality-reviewer.md)
|      +-- On quality pass: VERIFY — run tests, read output, confirm pass
|      +-- Mark task complete only with verification evidence
|
+-- 4. After all tasks in sprint: PRODUCT ACCEPTANCE REVIEW
|      +-- Run full test suite — paste output as evidence
|      +-- Launch Product Acceptance (Claude + Codex PARALLEL)
|      |      (see prompts/product-reviewer.md)
|      +-- Fix any product issues found (auto, no user interaction)
|      +-- Run tests AGAIN after fixes — verify still green
|      +-- Push branch, create PR with sprint summary + acceptance results
|
+-- 5. Clean up worktree:
|      git worktree remove .worktrees/sprint-N
|
+-- 6. Start next sprint immediately (next worktree from updated base)
```

### Git Worktree Rules

Worktrees provide sprint isolation — each sprint works in its own directory.

**Setup (once per session):**
```bash
# Ensure .worktrees/ is gitignored
grep -q '.worktrees/' .gitignore 2>/dev/null || echo '.worktrees/' >> .gitignore
```

**Per sprint:**
```bash
# Create worktree for sprint
git worktree add .worktrees/sprint-N feat/<feature>-sprint-N

# Work happens inside .worktrees/sprint-N/
# All file paths in agent dispatches use this directory

# After PR creation, clean up
git worktree remove .worktrees/sprint-N
```

**Fallback:** If worktrees are unavailable (e.g., bare repo, permission issues), fall back to regular branch checkout. Note this in the completion report.

### Agent Dispatch Rules

**Implementation agents:**
- Use `run_in_background: true` for independent tasks
- Use `mode: bypassPermissions` for all implementation agents
- Use `model: sonnet` for mechanical tasks (1-2 files, clear spec)
- Use default model for complex integration tasks
- Maximum concurrent agents: limited only by task independence
- All agents use prompt from `prompts/implementer.md`

**Review optimization:**
- For simple tasks (1-2 files, <50 lines changed): spec review only, skip code quality review
- For medium tasks (2-5 files): spec review + Claude code quality (skip Codex)
- For complex tasks (5+ files, new architecture): full review cycle (spec + Claude + Codex + product)
- Product review: only for user-facing changes (UI, bot responses, API behavior)

### Systematic Debugging

When a test fails during execution, follow this protocol — never do "try random fixes until tests pass":

1. **Investigate** — read the failure output, identify the failing assertion
2. **Identify pattern** — is it a data issue, logic error, integration mismatch, or environment problem?
3. **Form hypothesis** — state what you think is wrong and why, before touching code
4. **Diagnostic instrumentation** — if unclear, add logging at component boundaries to trace data flow
5. **Implement fix** — targeted fix based on the hypothesis
6. **Verify** — run the specific failing test, confirm it passes, then run full suite

If unfixable after 2 targeted attempts, mark as BLOCKED with diagnostic evidence and continue.

### Codex for Code Quality Review

MANDATORY for complex tasks. Launch in parallel with Claude reviewer.
Use prompt template from `prompts/code-quality-reviewer.md`.

```bash
codex --approval-mode full-auto --quiet \
  -p "PROMPT_CONTENT" 2>&1
```

### Product Acceptance Review (MANDATORY per sprint)

After all tasks pass code review, run Product Acceptance. This is the **most important review stage** — it verifies that what was built matches the spec INTENT, not just the technical requirements. Code can be technically clean but productively wrong.

Launch Claude + Codex product reviewers IN PARALLEL using `prompts/product-reviewer.md`.

**Merge results from both.** If either says NEEDS_FIXES -> fix autonomously -> re-review. Only create PR after BOTH accept.

### PR Creation Pattern

Each sprint creates its own PR:

```bash
git push -u origin feat/<feature>-sprint-N
gh pr create --title "Sprint N: <scope>" --body "..."
```

PR body format:
```markdown
## Sprint N: [Scope Description]

### Changes
- [bullet points of what was built]

### Verification Evidence
- Test suite: X/Y passing (full output reviewed)
- Product acceptance: ACCEPTED by Claude + Codex

### Dependencies
- Depends on: PR #NNN (Sprint N-1) — merge that first
- or: Independent — can merge standalone
```

### Handling Failures During Autonomous Execution

**Test failure:** Investigate root cause using systematic debugging protocol. Fix in-place, don't stop sprint. If unfixable after 2 targeted attempts, note in PR description with diagnostic evidence and continue.

**Build failure:** Check if it's pre-existing (ignore) or new (fix). If new and unfixable, note and continue.

**Agent blocked:** Re-dispatch with more context or different model. If still blocked after 2 attempts, implement manually and continue.

**NEVER** stop the sprint pipeline to ask the user. Accumulate issues and report them all at the end.

### Completion Report

When ALL sprints are done, report to user:

```
## SuperFlow Complete

### PRs Created
1. #NNN — Sprint 1: [scope] — [status: ready for review]
2. #NNN — Sprint 2: [scope] — [status: ready for review]

### Verification Evidence
- Full test suite: X/Y passing
- New tests added: Z
- All PRs passed product acceptance review

### Known Issues
- [any accumulated issues with diagnostic evidence]

### Merge Order
1. Merge PR #NNN first (no dependencies)
2. Then PR #NNN (depends on #1)
```

---

## Codex Integration Rules

### When Codex is MANDATORY
- Spec review (parallel with Claude)
- Plan review (parallel with Claude)
- Code quality review for complex tasks (parallel with Claude)

### When Codex is OPTIONAL
- Implementation of isolated tasks (alternative to Claude subagent)
- Quick checks (lint, type-check output analysis)

### Codex Invocation Pattern

Always via Bash tool:
```bash
codex --approval-mode full-auto --quiet -p "PROMPT" 2>&1
```

Key rules:
- Self-contained prompt (Codex can't ask questions)
- Include ALL context (file paths, project conventions, constraints)
- Always `git diff` after Codex implementations to verify scope
- Timeout: `timeout 300 codex ...` for long tasks

### Codex Detection at Startup

At the very beginning of the session (Step 1), run:
```bash
which codex 2>/dev/null && codex --version 2>/dev/null
```

If codex is found -> set `CODEX_AVAILABLE=true` and use for all parallel reviews.
If codex is NOT found -> set `CODEX_AVAILABLE=false`, skip silently, use Claude-only.

**Never fail or warn the user about missing Codex.** It's a bonus, not a requirement.

---

## Red Flags — Things That Must NEVER Happen

1. **Pausing to ask "should I continue?"** — After plan approval, execute silently
2. **Claiming done without test output** — Paste evidence or it didn't happen
3. **Skipping the "verify fail" step in TDD** — A test that never failed proves nothing
4. **One giant PR** — Always PR per sprint
5. **Skipping Codex reviews** — Always try Codex first if available
6. **2 agents when 5 could run** — Maximize parallelism
7. **Only asking questions, no suggestions** — Always propose ideas alongside questions
8. **Committing directly to main** — Always feature branches
9. **Random fix attempts** — Investigate before fixing. Hypothesis before code.
10. **"Should work" without evidence** — Run it. Read output. Then claim.

---

## Documentation Update (after all sprints)

After the last sprint's PR is created, before the completion report:

1. **Update CLAUDE.md** — add new modules, files, conventions introduced by this work
2. **Update README** (if project has one) — add new features, API endpoints, commands
3. **Update BACKLOG.md** (if exists) — mark completed items, add new items discovered
4. **Commit documentation updates** to the last sprint's branch

---

## Integration with Superpowers

SuperFlow uses these superpowers skills internally:
- `superpowers:brainstorming` — base flow, extended with multi-expert lenses
- `superpowers:writing-plans` — plan structure, bite-sized granularity
- `superpowers:subagent-driven-development` — agent dispatch patterns
- `superpowers:test-driven-development` — TDD within tasks (see `prompts/implementer.md`)
- `superpowers:verification-before-completion` — Rule 6, evidence-based completion
- `superpowers:using-git-worktrees` — sprint isolation via worktrees
- `superpowers:debugging` — systematic debugging protocol

SuperFlow OVERRIDES these superpowers behaviors:
- Brainstorming: adds proactive product suggestions (not just questions)
- Plan execution: PR per sprint (not one PR for everything)
- Plan granularity: 2-5 minute steps, 20-30+ per feature (not 5-10 big tasks)
- Review: mandatory Codex parallel review (not Claude-only)
- Completion: requires verification evidence (not self-reported status)
- Pacing: zero pauses after plan approval (not "check with user between tasks")
