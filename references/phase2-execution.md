# Phase 2: Autonomous Execution (ZERO INTERACTION)

After user approves, execute continuously until all sprints are complete.
Never ask, never pause, never summarize between sprints.

## Execution Architecture

**CRITICAL: The orchestrator NEVER writes code directly.**

The main agent (you) only:
- Creates worktrees and branches
- Dispatches subagents for implementation
- Runs tests and reviews
- Creates PRs and merges

All implementation happens in subagents with isolated context windows.

## Per-Sprint Flow

```
For each Sprint N:
|
+-- 1. Re-read this file (references/phase2-execution.md)
|
+-- 2. Create git worktree:
|      git worktree add .worktrees/sprint-N feat/<feature>-sprint-N
|      (Verify .worktrees/ is in .gitignore)
|
+-- 3. Run baseline tests in worktree
|
+-- 4. For each task (maximize parallel agents):
|      +-- Dispatch implementer via Agent tool:
|          - mode: bypassPermissions
|          - model: sonnet (for mechanical tasks)
|          - Include: plan excerpt, file paths, codebase context
|          - Use prompts/implementer.md template
|      +-- On completion: dispatch spec reviewer (background)
|      +-- On spec pass: dispatch code quality reviewer (background)
|      +-- On quality pass: VERIFY — run tests, read output
|      +-- Mark task complete only with verification evidence
|
+-- 5. After all tasks: PRODUCT ACCEPTANCE REVIEW
|      +-- Run full test suite — paste output as evidence
|      +-- Launch Claude Agent reviewer (background)
|      +-- Launch Codex reviewer (background, gtimeout on macOS)
|      +-- Wait for both. Fix issues if any. Re-test.
|      +-- Only create PR after acceptance
|
+-- 6. Push branch, create PR targeting main
|
+-- 7. Clean up worktree:
|      git worktree remove .worktrees/sprint-N
|
+-- 8. If Telegram MCP connected: send progress update
|
+-- 9. Start next sprint immediately
```

## Agent Dispatch Rules

**Implementation agents:**
- Use `run_in_background: true` for independent tasks
- Use `mode: bypassPermissions` for all implementation agents
- Use `model: sonnet` for mechanical tasks (1-2 files, clear spec)
- Default model for complex integration tasks
- Maximum concurrent agents: limited only by task independence

**Review optimization:**
- Simple tasks (1-2 files, <50 lines): spec review only
- Medium tasks (2-5 files): spec review + Claude code quality
- Complex tasks (5+ files): full review cycle (spec + Claude + Codex + product)

## Git Worktree Rules

```bash
# Setup (once per session)
grep -q '.worktrees/' .gitignore 2>/dev/null || echo '.worktrees/' >> .gitignore

# Per sprint
git worktree add .worktrees/sprint-N feat/<feature>-sprint-N
# Work happens inside .worktrees/sprint-N/
# After PR creation:
git worktree remove .worktrees/sprint-N
```

**Fallback:** If worktrees unavailable, use regular branch checkout.

## Systematic Debugging

When a test fails:
1. **Investigate** — read failure output, identify the failing assertion
2. **Identify pattern** — data issue, logic error, integration mismatch?
3. **Form hypothesis** — state what's wrong before touching code
4. **Implement fix** — targeted fix based on hypothesis
5. **Verify** — run specific test, then full suite

If unfixable after 2 attempts → BLOCKED with evidence, continue.

## PR Creation

**All PRs target `main`.** Never target other sprint branches.

```bash
git push -u origin feat/<feature>-sprint-N
gh pr create --base main --title "Sprint N: <scope>" --body "..."
```

## Handling Failures

- **Test failure:** Debug systematically. If unfixable → note in PR, continue.
- **Build failure:** Pre-existing → ignore. New → fix. Unfixable → note, continue.
- **Agent blocked:** Re-dispatch with more context. 2 fails → implement manually.
- **NEVER** stop to ask the user. Accumulate issues, report at end.

## Completion Report

```
## SuperFlow Complete

### PRs Created
1. #NNN — Sprint 1: [scope]
2. #NNN — Sprint 2: [scope]

### Verification Evidence
- Full test suite: X/Y passing
- All PRs passed product acceptance review

### Known Issues
- [issues with diagnostic evidence]

### Merge Order
1. Merge #NNN first
2. Then #NNN (depends on #1)
```

## Telegram Progress (when MCP connected)

At these points, send a short message via `mcp__plugin_telegram_telegram__reply`:
- Sprint started: "Sprint N/M started: [scope]"
- PR created: "PR #NNN created: [title]"
- Error: "Sprint N blocked: [brief reason]"
- Done: Full completion report
