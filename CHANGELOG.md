# Changelog

All notable changes to superflow will be documented in this file.

## [2.0.0] - 2026-03-21

### Changed
- **BREAKING: Replace hardcoded Codex dependency with provider-agnostic cross-model review** — the skill no longer requires Codex specifically. Any CLI-based LLM (Codex, Gemini CLI, Aider, etc.) can serve as the secondary reviewer
- **Two-tier review strategy:**
  - **Tier 1 (cross-model):** when a secondary provider is available, dispatches Claude + secondary provider in parallel for truly independent reviews — different models, different biases, different blind spots
  - **Tier 2 (split-focus fallback):** when no secondary provider is available, dispatches two Claude agents with different review focus areas (correctness vs architecture, spec-fit vs user-scenarios)
- **Auto-detection at startup:** checks for `codex`, `gemini`, `aider` — uses first available, falls back silently to Tier 2
- Updated all prompt templates with both cross-model and split-focus invocation patterns
- Secondary provider is now optional but recommended (was effectively required before)

## [1.2.0] - 2026-03-21

### Added
- **ultrathink reasoning**: spec review, plan review, and product acceptance prompts now use `ultrathink` for extended thinking, regardless of user's default reasoning effort
- **Codex in brainstorming**: Codex dispatched as Product Expert during Phase 1 brainstorming (parallel with Claude conversation) — two AI models produce more diverse ideas
- **Recommended launch section**: `claude --dangerously-skip-permissions` for autonomous execution, reasoning effort guidance (high/max + ultrathink)
- **Model strategy table**: detailed per-task model and reasoning recommendations (Opus for planning/review, Sonnet for implementation)

## [1.1.0] - 2026-03-21

### Fixed
- Codex CLI invocation: updated from `codex --approval-mode full-auto --quiet -p` to `codex exec --full-auto` (new Codex CLI API)
- macOS compatibility: use `gtimeout` from coreutils instead of `timeout`
- PR base strategy: all PRs now target `main` to prevent auto-close on squash merge
- Superpowers attribution: corrected to community project (obra/superpowers), not official Anthropic

### Added
- Product Acceptance Review enforcement: marked as NON-NEGOTIABLE with 6-step checklist
- Mandatory self-reminder loop: sprint completion checklist before PR creation
- Checkpoint re-read after each sprint completion

## [1.0.0] - 2026-03-19

### Added
- Initial release
- Two-phase workflow: collaborative product discovery + autonomous execution
- PR-per-sprint with git worktrees
- Dual-model reviews (Claude + Codex)
- Product acceptance review stage
- Context drift prevention
- 5 prompt templates (implementer, spec-reviewer, code-quality-reviewer, product-reviewer, testing-guidelines)
