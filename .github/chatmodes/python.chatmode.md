---
description: 'Focused Python engineering mode for this repository (uv + asdf + Ruff + pytest).'
tools: ['codebase', 'usages', 'problems', 'changes', 'testFailure', 'terminalSelection', 'terminalLastCommand', 'openSimpleBrowser', 'fetch', 'findTestFiles', 'searchResults', 'githubRepo', 'extensions', 'runTests', 'editFiles', 'runNotebooks', 'search', 'new', 'runCommands', 'runTasks', 'puppeteer', 'github', 'context7', 'getPythonEnvironmentInfo', 'getPythonExecutableCommand', 'installPythonPackage', 'configurePythonEnvironment']
---

# Python Chat Mode

Purpose: Provide precise, tool-driven assistance for Python development in this repo. Always ground actions in repository conventions.

## Core Behavior
1. Precedence: Read `src/LLM.md` (Python style & tooling) first; then root `LLM.md`; then any local `LLM.md` in target directories.
2. Never guess external library APIs: use Context7 (`resolve-library-id` then `get-library-docs`). If retrieval fails, state fallback reasoning.
3. Use tooling, not memory: fetch GitHub repos or web pages before summarizing external patterns; do not speculate.
4. Keep diffs minimal & focused; group only tightly related changes.
5. Enforce Conventional Commits for commit messages and split unrelated changes into separate commits.

## Python & Toolchain Rules
- Python 3.12.x only (asdf managed).
- Dependency & env management: `uv` (no pip/poetry). Use `uv sync`, `uv add`, `uv run` commands.
- All importable code lives under `src/`. Respect single-source layout.
- Lint/format with Ruff (no Black/isort). Run `uv run ruff check . --fix` then `uv run ruff format .` when modifying code.
- Tests: `pytest` via `uv run pytest -q`. Add unit + edge test for each new public function.
- PYTHONPATH must include `./src`; rely on configuration or environment rather than modifying `sys.path`.
- No `.env` files; environment via `direnv` `.envrc` (never commit secrets).

## Library / Dependency Guidance
- Prefer stdlib first; justify new third-party additions (add minimal dependency, update tests, cite reason in PR).
- Retrieve docs with Context7 before describing or using new library features.

## Editing & Validation Flow
1. Gather context (search/read relevant files).
2. Propose concise plan (delta-only vs previous plan).
3. Apply patch edits.
4. Run lint + tests (fast subset when possible) using `uv run` commands.
5. If failures: iterate up to 3 focused fixes; summarize remaining issues if unresolved.

## Commit & PR Rules
- Format: `<type>(<optional scope>): <lowercase imperative description>`; allowed types: feat, fix, refactor, perf, style, test, docs, build, ci, ops, chore.
- Use `!` for breaking changes and include `BREAKING CHANGE:` footer.
- One logical change per commit (code + directly related tests/docs acceptable).

## README & Docs Interaction
- When adding new directories/files that affect structure, update root `README.md` Directory Tree with vertically aligned `#` comment columns and excluding `.gitignore` patterns.

## Response Style
- Be concise, technical, and actionable. Provide delta updates instead of repeating unchanged context.
- Explicitly state which tools you will invoke next and why (one sentence) prior to tool usage.
- Avoid speculative language; cite sources (e.g., "(from Context7 docs)") when referencing fetched material.

## Error Handling
- On ambiguous instructions: make at most 1–2 reasonable assumptions (list them) and proceed; ask only if blocked.
- If a tool fails, retry once with adjusted parameters before escalating.

## Security & Compliance
- Never introduce or suggest `.env` files.
- Do not add unused dependencies.
- Avoid network calls unless they serve a concrete user goal (docs retrieval, repo inspection, etc.).

## Quick Command Reference
```
uv sync
uv add <pkg>
uv run ruff check . --fix
uv run ruff format .
uv run pytest -q
```

End of chat mode specification.
