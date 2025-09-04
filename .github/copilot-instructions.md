# Copilot Coding Agent Instructions

Status: Repository scaffold (Python project not yet initialized). This file gives high-level AI contribution rules. Language/tooling specifics (Python, etc.) live in language style guide files referenced from `LLM.md` and must NOT be duplicated here.

## Mandatory Read First
Before writing or editing code you MUST read root `LLM.md`. That file lists active language guides (e.g., `src/LLM.md` for Python) which you must also read before touching related files. If guidance conflicts: technical style (language guide) > orchestration (`LLM.md`) > this file. When introducing or modifying a convention, update the relevant language guide and (if process changes) `LLM.md` in the same PR.

## Scope of This File
This document focuses on: AI workflow guardrails, change discipline, and when to update the style guide. It intentionally omits concrete commands already captured in `src/LLM.md`.

## Core AI Workflow
1. Read `LLM.md` (always re-check if repo recently changed) plus any language guides it references.
2. If editing files in a subdirectory containing its own `LLM.md`, read that before changes.
3. Search for existing patterns before adding new modules or dependencies.
3. Keep edits minimal & focused; separate unrelated concerns into distinct PRs.
4. Add or update tests for every behavioral change (happy path + edge case minimum) following locations described in the style guide.
5. Run lint + tests locally prior to completion (commands in style guide).

## Dependency & Tooling Governance
-- Do not introduce alternative version managers, env managers, or formatters beyond those allowed in the applicable language guide(s).
- Any new third-party library requires: brief rationale in PR description + test coverage + style guide update if it changes conventions.

## Structural Changes
-- When adding new top-level packages, update the appropriate language guide future sections list.
- If refactoring cross-cutting utilities (logging, config, error handling), ensure backward compatibility or note breakage explicitly.

## Configuration & Secrets
- Never commit secrets; extend `.env.example` when adding required environment variables. Reference only through central config module (pattern defined in style guide future section).

## Documentation Hygiene
- Keep this file under ~60 lines; remove stale guidance proactively.
-- Any style or command drift detected: fix the source of truth language guide first, then adjust this file only if meta references change.
 - When referencing repository files in comments, PRs, or AI-generated text: never list files or exact paths that are matched by `.gitignore`. If the ignored files are relevant, reference the ignore pattern or directory and note that contents are ignored (do not expose or enumerate them).

## Quality Gates (Summarized)
- Lint clean, formatted code.
- Tests pass (unit + integration if touched components have integration coverage).
- No unused or unjustified dependencies.
- Commits reference introduced conventions when applicable.

## Escalation / Ambiguity Handling
If a task requires a new convention (e.g., background job runner abstraction) and no precedent exists: (a) implement minimal viable pattern, (b) document it in the appropriate language guide (e.g., `src/LLM.md`), (c) reference the new section in the PR description.

## Quick Checklist for Any PR
-- [ ] Followed relevant language guide(s) referenced by `LLM.md`.
- [ ] Updated/added tests.
- [ ] Ran lint & format.
-- [ ] Updated applicable language guide(s) if conventions changed.
- [ ] No secret leakage; `.env.example` updated if needed.

End of instructions. Operational language/tooling details live in language guide files referenced from `LLM.md`.
