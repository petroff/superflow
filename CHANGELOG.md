# Changelog

All notable changes to superflow will be documented in this file.

## [1.3.0] - 2026-03-22

### Fixed
- Codex detection: replaced `which codex` with `codex --version 2>/dev/null` smoke test (binary can exist without API keys)
- Heredoc templates in code-quality-reviewer.md and product-reviewer.md: changed `<<'PROMPT'` (quoted, blocks variable expansion) to `<<PROMPT` (unquoted, allows `$(git diff ...)` to expand)

### Added
- Provider-agnostic review fallback: when Codex is unavailable, dispatch TWO Claude agents with split focus (technical + product) instead of skipping reviews
- macOS timeout fallback: `perl -e 'alarm N; exec @ARGV'` as universal fallback when neither `timeout` nor `gtimeout` is available. All timeout references now use `$TIMEOUT_CMD` variable
- Timeout Helper section in SKILL.md with startup detection snippet (gtimeout > timeout > perl fallback)
- `mkdir -p` for spec and plan directories before writing files (prevents failure on first run)

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
