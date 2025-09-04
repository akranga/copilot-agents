# Python Project Conventions (uv + asdf)

## 7. Code Style Specifics

**Python Code Commenting:** Avoid inline comments unless the code is absolutely complex and cannot be made clear through better naming, docstrings, or refactoring. Prefer self-documenting code with clear variable/function names and comprehensive docstrings.ce for Python project conventions. Keep this file current; update alongside any tooling or structural change. Copilot / AI agents must reference this, not reinvent patterns.

## 1. Runtime & Toolchain
- Python: 3.12.x only (managed via `asdf`).
- Install: add `python 3.12.x` to `.tool-versions` then run `asdf install`.
- Package & environment manager: `uv` (no pip / poetry / pip-tools).
- Virtual env: auto-managed in `.venv/` (created by `uv`). Never commit `.venv`.

## 2. Project Initialization
```
uv init --package .           # create pyproject.toml if missing
uv add <pkg>                  # add runtime dependency
uv add --dev pytest ruff      # add dev deps
uv sync                       # install from lock

- Executable entrypoints: all executable Python files (scripts under `src/` intended to be run directly) should use `click` for their CLI interface. Prefer a small `cli.py` exposing a `cli()` group and use `if __name__ == "__main__": cli()` to allow `python -m` execution. Keep entrypoints minimal and import the application logic from other modules to avoid side effects at import time.
```

## 3. Core Commands
```
uv sync                                   # install deps from lock
uv run pytest -q                          # run all tests
uv run pytest tests/unit -q               # run unit tests only
uv run pytest -m 'integration' -q         # (optional) only integration tests
uv run ruff check . --fix                 # lint & auto-fix
uv run ruff format .                      # format code
```

## 4. Directory & Module Layout
- All importable Python code MUST reside under `src/` (single source layout). Do not create top-level packages at repository root.
- Primary application package lives at `src/copilot_agents/` (named after the uv project). Use this as the root namespace.
- Tests: `tests/unit/` (fast, hermetic) and `tests/integration/` (may touch I/O, external services mocked or containerized).
- PYTHONPATH must include `./src` (CI, local dev, and test commands). Prefer environment variable (`PYTHONPATH=./src`) or tooling config rather than sys.path mutation.
- Avoid side effects at import time (no network / disk writes). Use `if __name__ == "__main__":` for CLI entrypoints.
- Central configuration: `src/copilot_agents/config.py` loads environment variables (use `python-dotenv` only if truly needed). Provide `.env.example`.

## 5. Dependency Management
- Prefer stdlib first. Justify any new third-party dependency in PR description.
- Never hand-edit `uv.lock`.
- Remove unused deps promptly (`uv remove <pkg>`).
- Pin optional tooling with extras in `pyproject.toml` (e.g., `[project.optional-dependencies]` groups) if/when needed.

## 6. Testing Conventions
- Each new public function/class: at least one happy-path + one edge/invalid test.
- Test file placement: tests should be in the same directory as the original file but prefixed with `test_` beneath the `./src` directory (e.g., `src/copilot_agents/cli.py` → `src/copilot_agents/test_cli.py`).
- Use `pytest` fixtures for shared setup; keep them small & explicit.
- Mark slower / external tests with `@pytest.mark.integration`.
- No network calls in unit tests (use stubs/fakes).
- Temporary filesystem usage: prefer `tmp_path` fixture.

- Operational rule: run the test suite (unit tests) only when preparing commits or when explicitly requested by a reviewer/maintainer. Avoid running the full test suite on every edit to keep feedback fast; use targeted test runs for local development (e.g., specific test files or markers). When you do run tests as part of commit validation, prefer `uv run pytest -q` or target specific files with `uv run pytest src/copilot_agents/test_<module>.py -q`.

## 7. Linting & Formatting
- Single tool: Ruff (both lint + format). No Black/isort separately.
- Run both check + format before committing (see commands above).
- Keep import grouping default; avoid manual sorting.
- Code style: avoid inline comments unless code is absolutely complex; prefer clear variable names, function names, and docstrings for explanation.
- Character usage: never use emojis, smileys, or other special unicode symbols in code, comments, docstrings, or program outputs. Use plain ASCII text only.

- Operational rule: run Ruff only when preparing commits or when explicitly requested by the reviewer/maintainer. Do not run Ruff automatically on every edit or as a background task; this keeps edit-feedback fast and avoids spurious changes. When you do run Ruff for a commit, prefer `uv run ruff check . --fix` followed by `uv run ruff format .` as part of your commit validation step.

## 8. Logging & Errors
- Implement `copilot_agents/logging.py` with a `get_logger(name: str)` returning a structured JSON logger (e.g., stdlib `logging` + custom JSON formatter; avoid extra deps initially).
- Central exception types in `copilot_agents/errors.py` (e.g., `AppError`, domain-specific subclasses). Catch narrowly; avoid blanket `except Exception`.

### Exception Handling Guidelines
- **Never catch and rethrow exceptions unnecessarily**: Do not create try-catch blocks that only catch an exception and immediately rethrow it with a different message. Let the original exceptions (like `httpx.RequestError`, `json.JSONDecodeError`, `KeyError`) propagate naturally as they are already descriptive.
- **Avoid long try-catch blocks**: Keep try-catch blocks minimal and focused. Only wrap the specific operations that need error handling, not entire functions or large code blocks.
- **Catch exceptions only when you can handle them meaningfully**: Only use try-catch when you can take corrective action, provide fallback behavior, or need to transform the exception for the caller's context.
- **Use specific exception types in docstrings**: Document the specific exceptions that methods can raise (e.g., `httpx.RequestError`, `json.JSONDecodeError`) rather than generic `Exception` types.

## 9. Configuration
- Use `direnv` (`.envrc`) to load environment variables locally. Do NOT add `.env` or `.env.*` files; they are disallowed.
- Access env vars only inside config layer; elsewhere import from `app.config`.

## 10. PR & Branch Workflow
- Branch prefix: `feat/`, `fix/`, `chore/`, `refactor/`.
- Commit messages MUST follow Conventional Commits: `<type>(<optional scope>): <lowercase imperative description>`.
	- Allowed baseline types: `feat`, `fix`, `refactor`, `perf`, `style`, `test`, `docs`, `build`, `ops`, `chore`.
	- Use `!` before `:` to denote breaking change (e.g., `feat(api)!: remove legacy endpoint`). Provide a `BREAKING CHANGE:` footer explanation when used.
	- Description: imperative, present tense, no leading capital, no trailing period.
	- Prefer a scope only when it adds clarity (package, module, or domain); keep it short.
- Split unrelated logical changes into separate commits (enforced expectation for review hygiene). A commit may modify code + related docs/tests if tightly coupled.
- Keep diffs < ~400 LOC (excluding generated files) when possible.
- Include updated tests + doc snippets (update this STYLEGUIDE if conventions shift).

## 11. AI / Automation Expectations
- Before proposing new patterns, search existing modules; reuse shapes & naming.
- If adding a cross-cutting convention (e.g., new logging field), update this file in same PR.
- When needing authoritative details about an external Python library (APIs, configuration, version-specific changes), ALWAYS use the Context7 documentation tools: first call `resolve-library-id` (unless ID explicitly provided), then `get-library-docs` targeting only needed topics. Prefer citing only the minimal relevant snippet; avoid speculative descriptions without fetched docs. If docs retrieval fails, note fallback reasoning explicitly.
- When a GitHub repository is referenced for details, use GitHub search/repo tools to inspect the actual source (functions, README, examples) instead of relying on memory. For non-GitHub web sources, use fetch or puppeteer tooling to retrieve current content before summarizing.

- When a task specifically asks you to use Gradio or Click, follow the Context7 workflow first: call `resolve-library-id` for `gradio` or `click`, then `get-library-docs` for topics like `usage`, `examples`, `api`, or `quickstart`. If Context7 returns no useful documentation, fall back to the official docs (Gradio: `https://www.gradio.app/docs`, Click: `https://click.palletsprojects.com/`). Record the doc lookup (library id + topics fetched) in PR notes and cite any snippets used.

## 12. LiteLLM client & HTTP fallback (new guidance)

- Prefer the official `litellm` Python client when integrating code with LiteLLM services or servers. It provides a higher-level, stable API surface and will usually handle authentication, retries, and model-routing semantics.
- When adding or coding against the `litellm` client, use the Context7 library-doc workflow to fetch authoritative docs before coding: call `resolve-library-id` for the package name, then `get-library-docs` with the specific topic (e.g., `client`, `usage`, `examples`). Record the doc lookup in PR notes.
- If the `litellm` client is not available or the environment requires a thin fallback, use `httpx` (not `requests`) for HTTP interactions. `httpx` is preferred for its async support, modern API, and better timeouts.
- All example or runnable Python scripts in `src/` should include a shebang (`#!/usr/bin/env python3.12`) so they can be executed directly, and CI/dev tasks should `chmod +x` the file when installing local tools.
- When providing a fallback HTTP implementation, keep it small and clearly documented: the REST contract may vary between LiteLLM versions, so prefer calling the client API where possible and only use HTTP fallback for quick admin/debug flows.

Assumptions made in examples in this repo:
- The Python package name is assumed to be `litellm` for the official client. If your environment uses a different package name, adapt imports accordingly.
- The example HTTP fallback posts JSON to the configured `LITELLM_URL` (default `http://localhost:4000`); adapt endpoint paths to your deployment if needed.

---
Revision history:
- v0.1 initial Python-focused style.
- v0.2 (2025-09-04): Adopt single-source layout under `src/` and require PYTHONPATH inclusion.
- v0.3 (2025-09-04): Mandate `direnv` for env management; forbid `.env` / `.env.*` files.
- v0.4 (2025-09-04): Require use of Context7 tools (resolve-library-id + get-library-docs) for Python library information lookups.
- v0.5 (2025-09-04): Add rule to use GitHub repo tools for repository references and fetch/puppeteer for external web pages.
- v0.6 (2025-09-04): Remove future placeholder section per new rule forbidding speculative "Future Enhancements" content in LLM.md files.
- v0.7 (2025-09-04): Add code style rule to avoid inline comments unless absolutely necessary; prefer clear naming and docstrings.
- v0.8 (2025-09-04): Update testing convention to place test files in same directory as source files with `test_` prefix under `./src`.
- v0.9 (2025-09-04): Add rule to avoid emojis, smileys, or special unicode symbols in code, comments, and outputs.
- v0.10 (2025-09-04): Add exception handling guidelines: avoid catch-and-rethrow patterns, keep try-catch blocks minimal and focused.
````
