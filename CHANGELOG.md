# Changelog

All notable changes to SuperFlow will be documented in this file.

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
