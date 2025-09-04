# Copilot Coding Agent Instructions

Concise guardrails for AI contributions in this repository. Technical Python specifics live in `src/LLM.md`; do not duplicate—reference instead.

## Read Order (ALWAYS)
1. `src/LLM.md` (Python style/tooling)
2. Root `LLM.md` (orchestration + remembered rules)
3. Local directory `LLM.md` (if present)
4. Relevant `README.md` (ensure directory tree freshness & alignment)

If conflicting guidance: `src/LLM.md` > local `LLM.md` > root `LLM.md` > this file.

## Repository Shape (Current)
Minimal scaffold: single package `src/copilot_agents/` (empty `__init__.py`). Style & workflow rules already established; future modules (logging, config) to follow documented placeholders in `src/LLM.md` (see Future Sections).

## Core Workflow (Happy Path)
1. Gather context (search/read) before changing anything.
2. Plan small, single-purpose change (avoid bundling unrelated edits).
3. Edit via patch tool; never commit/push unless explicitly requested by user (see remembered rule).
4. Run: `uv sync` (if deps changed) -> `uv run ruff check . --fix` -> `uv run ruff format .` -> `uv run pytest -q`.
5. Add/update tests for any new public API (happy + edge case).
6. Prepare Conventional Commit message; await explicit commit instruction.

## Conventions (Project-Specific)
- Single-source layout: ALL importable code under `src/`.
- Dependency management: `uv` only (no pip/poetry/pip-tools). Never hand-edit `uv.lock`.
- Env loading: `direnv` (`.envrc`); never introduce `.env` files.
- Lint/format: Ruff is authoritative (no Black/isort).
- Commit format: Conventional Commits (types: feat, fix, refactor, perf, style, test, docs, build, ci, ops, chore; `!` for breaking).
- Directory Tree in any README must be regenerated & aligned (exclude `.gitignore` paths; align `#` comment column).

## External Knowledge Usage
- Library/API details: use Context7 tools (`resolve-library-id` then `get-library-docs`) before coding against unfamiliar libs.
- GitHub repo references: fetch real source (repo search) not memory.
- Web knowledge: use fetch/puppeteer; cite source briefly.

## Change Control & Scope
- One logical concern per commit (code + directly related tests/docs only).
- Large refactors: stage as incremental PRs (≤ ~400 LOC diff guideline).
- Introduce new dependency only with justification + tests + style guide update (if convention-level impact).

## Testing Expectations
- Minimum: each new function/class = 1 happy + 1 edge case test.
- No network I/O in unit tests; stub/fake externally.
- Mark slower/external style tests with `@pytest.mark.integration` (directory to be added when such tests exist).

## Documentation Responsibilities
- When structural changes occur (new directories/files), update root `README.md` tree in same PR.
- If adding cross-cutting convention, update `src/LLM.md` AND reference in PR description.

## Ambiguity Handling
If no precedent: implement minimal viable pattern, document it (`src/LLM.md` Future Sections), and clearly annotate rationale in PR body. Defer broader design unless requested.

## Quality Gate Before Requesting Commit
- Ruff: clean (no unfixable errors) & formatted.
- Tests: pass locally (`uv run pytest -q`).
- README trees (changed ones) reflect actual FS.
- No orphaned or unused imports / dependencies.
- No secrets or ignored file listings.

## Quick PR Preparation Checklist
- [ ] Context read (LLM.md hierarchy + relevant README)
- [ ] Minimal focused change
- [ ] Tests updated/added
- [ ] Ruff lint & format clean
- [ ] Docs/trees synced (if affected)
- [ ] Conventional Commit message drafted (not committed yet)

End of instructions.
