# Python Project Conventions (uv + asdf)

Authoritative source for Python project conventions. Keep this file current; update alongside any tooling or structural change. Copilot / AI agents must reference this, not reinvent patterns.

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
- Use `pytest` fixtures for shared setup; keep them small & explicit.
- Mark slower / external tests with `@pytest.mark.integration`.
- No network calls in unit tests (use stubs/fakes).
- Temporary filesystem usage: prefer `tmp_path` fixture.

## 7. Linting & Formatting
- Single tool: Ruff (both lint + format). No Black/isort separately.
- Run both check + format before committing (see commands above).
- Keep import grouping default; avoid manual sorting.

## 8. Logging & Errors
- Implement `copilot_agents/logging.py` with a `get_logger(name: str)` returning a structured JSON logger (e.g., stdlib `logging` + custom JSON formatter; avoid extra deps initially).
- Central exception types in `copilot_agents/errors.py` (e.g., `AppError`, domain-specific subclasses). Catch narrowly; avoid blanket `except Exception`.

## 9. Configuration
- Use `direnv` (`.envrc`) to load environment variables locally. Provide a tracked `.envrc.example` (placeholders, no secrets). Do NOT add `.env` or `.env.*` files; they are disallowed. If migrating from `.env`, move keys into `.envrc.example` with exports.
- Access env vars only inside config layer; elsewhere import from `app.config`.

## 10. PR & Branch Workflow
- Branch prefix: `feat/`, `fix/`, `chore/`, `refactor/`.
- Keep diffs < ~400 LOC (excluding generated files) when possible.
- Include updated tests + doc snippets (update this STYLEGUIDE if conventions shift).

## 11. AI / Automation Expectations
- Before proposing new patterns, search existing modules; reuse shapes & naming.
- If adding a cross-cutting convention (e.g., new logging field), update this file in same PR.
- When needing authoritative details about an external Python library (APIs, configuration, version-specific changes), ALWAYS use the Context7 documentation tools: first call `resolve-library-id` (unless ID explicitly provided), then `get-library-docs` targeting only needed topics. Prefer citing only the minimal relevant snippet; avoid speculative descriptions without fetched docs. If docs retrieval fails, note fallback reasoning explicitly.
- When a GitHub repository is referenced for details, use GitHub search/repo tools to inspect the actual source (functions, README, examples) instead of relying on memory. For non-GitHub web sources, use fetch or puppeteer tooling to retrieve current content before summarizing.

## 12. Future Sections (Populate When Implemented)
- `app/logging.py` example snippet.
- `app/config.py` pattern with validation.
- External service adapters and interface boundaries.

---
Revision history:
- v0.1 initial Python-focused style.
- v0.2 (2025-09-04): Adopt single-source layout under `src/` and require PYTHONPATH inclusion.
- v0.3 (2025-09-04): Mandate `direnv` for env management; forbid `.env` / `.env.*` files.
- v0.4 (2025-09-04): Require use of Context7 tools (resolve-library-id + get-library-docs) for Python library information lookups.
- v0.5 (2025-09-04): Add rule to use GitHub repo tools for repository references and fetch/puppeteer for external web pages.
