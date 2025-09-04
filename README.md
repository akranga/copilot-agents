# copilot-agents

Automation and AI assistant orchestration utilities: a minimal Python 3.12+ toolkit scaffold for building and coordinating GitHub Copilot / LLM-driven workflows.

## Directory Tree
```
.                       # Repository root
├── .editorconfig       # Editor formatting baseline
├── .gitignore          # Git ignore patterns
├── .tool-versions      # asdf toolchain pins (python, etc.)
├── LICENSE             # MIT license
├── LLM.md              # Root LLM orchestration & README authoring rules
├── pyproject.toml      # Project metadata, dependencies, build config (hatchling + uv)
├── src/                # All Python source (single-source layout root)
│   ├── LLM.md          # Python style & tooling guide (authoritative technical rules)
│   └── copilot_agents/ # Primary package namespace
│       └── __init__.py # Package marker / version exposure placeholder
└── .github/            # GitHub configuration & AI workflow guardrails (workflows, instructions)
```

## Tools
- Language: Python 3.12
- Environment / dependency manager: uv (with asdf for runtime installation)
- Build backend: hatchling
- Lint & format: Ruff
- Testing: pytest
- Env management: direnv (+ `.envrc` / `.envrc.example` pattern)

## Developer Setup
```shell
# Install asdf & required plugins (if not already)
# macOS (Homebrew):
# brew install asdf

# Ensure python 3.12 toolchain
asdf install

# Sync dependencies (creates .venv)
uv sync

# Run lint + format check
uv run ruff check .
uv run ruff format .

# Run tests (none yet; scaffold command)
uv run pytest -q
```
