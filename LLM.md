# LLM Instructions (root)

Purpose: Provide a compact, machine-readable prompt and local rules for any LLM (GitHub Copilot or similar) working on this repository. Update when repository structure or tooling changes.

---

PROMPT: You are a coding assistant (GitHub Copilot) that will make edits, run checks, and create PRs for this repository. Follow these rules strictly.

1. Read order and precedence
- Always read `/src/LLM.md` first for language/tooling conventions.
- Then read `/.github/copilot-instructions.md` for AI workflow guardrails.
- If a directory contains `LLM.md`, read it before modifying files in that directory or any files under it.
When merging conflicting guidance, directory-local `LLM.md` overrides root `LLM.md`, and `/src/LLM.md` (Python style) wins over root for technical style rules.

2. When to follow subdirectory `LLM.md`
- Default: for any file under `<subdirectory>/`, follow instructions in `<subdirectory>/LLM.md`.
- Exception: if a task explicitly requests ignoring subdirectory rules, annotate the PR and explain why.
- If a subdirectory `LLM.md` conflicts with `/src/LLM.md` (Python style guide), follow `/src/LLM.md` and record the conflict in the PR description.

3. Tools you may use (local dev environment)
- Shell (via run_in_terminal): to run tests, linters, or setup commands.
- File edit tools (apply_patch): to create/modify files. Make minimal, focused changes.
- Search tools (file_search, grep_search): to find usages and patterns.
- Test runner (runTests): to execute unit tests relevant to changed files.
- Version sources: consult the root `.tool-versions` file (asdf) for authoritative language & tool version pins (e.g., python, node, etc.). Do not propose version bumps unless explicitly asked; if a bump is required, note rationale in PR description and update affected style guides.

4. Directory conventions (how the repo is organized)
- `src/` — all Python source code MUST live under this directory. Do not place importable packages/modules at repo root.
- `tests/unit/` and `tests/integration/` — test suites.
- `.github/` — CI and AI instruction files.
- `src/LLM.md` — authoritative project style and tooling guide (Python + automation conventions).
- `LLM.md` — per-directory LLM guidance (optional in subdirectories).
- PYTHONPATH: always include `./src` (e.g., `PYTHONPATH=./src uv run pytest -q`). Prefer configuring via environment (see `.envrc.example` if present) rather than relative imports hacks.
- Environment management: Use `direnv` with a committed template (`.envrc.example`) plus a developer-specific `.envrc` (may contain secrets). DO NOT introduce `.env` or `.env.*` files; they are disallowed.

5. Commit & PR conventions for LLM
- Keep diffs small and focused.
- Update `/src/LLM.md` when introducing new Python conventions.
- Add tests for behavioral changes.
- Document any deviations from global rules in the PR body.

6. Maintenance rule
- If you edit `src/LLM.md`, also update `/.github/copilot-instructions.md` to reference the change when it affects AI workflows.

7. Python instruction escalation ("remember" / important)
- If a user provides a new instruction that includes the word "remember" and it concerns Python code style, tooling, structure, testing, dependencies, or execution, you MUST also add (or merge) that rule into `src/LLM.md` (updating its revision history) and then follow it thereafter.
- If the user explicitly marks an instruction as "important" or says it must "always" be followed (and it's Python-related), treat it as a MUST-FOLLOW rule: do not ignore or downgrade it unless the user later revokes it. Record inclusion in the PR description.
- Avoid duplicating semantically equivalent guidance: search existing sections first; if overlap exists, refine or extend instead of repeating.
- Non-Python "remember" instructions belong in the most relevant `LLM.md` (root or subdirectory) only.
- When propagating, keep wording concise, actionable, and consistent with existing style; note any conflicts and prefer existing convention unless the user explicitly overrides.

8. Generic (non-Python) "remember" instructions
- When a user asks the assistant to "remember" an instruction that is NOT Python-specific, add it to this root `LLM.md` under an appropriate existing section or create (or append to) an `Additional remembered instructions` section at the bottom of the file with a datestamped bullet.
- If the user labels the instruction as "important" or "always", mark it as MUST-FOLLOW and do not remove or downgrade it unless the user explicitly rescinds it later (note rescission in a future commit).
- De-duplicate: search current file first; if related guidance exists, integrate by refining/expanding instead of adding a near-duplicate bullet.
- Keep each remembered item: (date YYYY-MM-DD) concise imperative phrasing.
- Subdirectory-scoped generic instructions should be placed in that subdirectory's `LLM.md` instead (and optionally cross-referenced here if they alter global behavior).

---

If you find any `LLM.md` files in subdirectories, add a reference to them in this file under a `Subdirectory LLM references` section and briefly summarize their scope and any special rules.

### README Authoring (MUST-FOLLOW)
Purpose: Standardize creation & maintenance of `README.md` files. Applies globally unless overridden explicitly.

General rules:
- A `README.md` in the repository root is the root README; any `README.md` inside a subdirectory documents that directory and all of its descendant subdirectories (scoped).
- Documentation must be concise, technical, and human-readable. Avoid filler.
- Do NOT invent new chapters/sections beyond the mandatory list unless explicitly instructed by a user or an existing directory `LLM.md` rule.
- Mandatory chapter order (exact, no extras unless authorized):
	1. H1 Title (single `#` heading) — short, specific.
	2. Concise summary paragraph (what the reader sees / purpose of this subtree).
	3. Directory Tree section (heading `## Directory Tree`) containing a current listing of this directory and all nested non-empty subdirectories/files (exclude: empty directories and anything ignored by `.gitignore`). Each entry MUST end with ` # <one-line summary>` explaining its role.
- The directory tree MUST stay up-to-date whenever files are added, removed, or renamed. Update before completing a PR that changes structure.
- Generate the baseline tree with the `tree` command (example root usage shown below) then manually append one-line summaries:
	- Example (root): `tree -F -I '.git|.venv' .` (adjust `-I` to exclude patterns from `.gitignore`; never list ignored files).
	- Remove any empty directory lines from the output.
	- Append ` # <summary>` to each listed path (files and directories).
- Always read the existing Directory Tree before making structural changes to ensure consistency and to update summaries if semantics change.

Root `README.md` additional mandatory chapters (after Directory Tree):
	4. Tools (heading `## Tools`) — brief bullet list of primary languages, package managers, linters, test frameworks.
	5. Developer Setup (heading `## Developer Setup`) — fenced `shell` code block with minimal reproducible setup commands (e.g., version manager install, dependency sync, test run). Keep only authoritative, working commands; no commentary inside the block other than `#` inline clarifications.

Subdirectory `README.md` differences:
- Omit the root-only "Tools" and "Developer Setup" sections unless the subtree introduces additional, localized tools; if so, explicitly state they are local to that subtree.
- Title should reflect the directory path or component name.

Prohibited unless explicitly requested:
- Adding marketing fluff, badges, or unrelated sections (e.g., Roadmap, FAQ) without an instruction.
- Auto-generated trees that include ignored or empty directories.

If instructed to remove (delete) a previously mandated chapter:
- Create (or update) an `LLM.md` in that directory documenting the deletion rule so automation does not reintroduce it. State: `(YYYY-MM-DD) <Chapter Name> removed by request; do not recreate unless explicitly reinstated.`

Validation checklist before committing a README change:
- [ ] Mandatory chapters present in correct order.
- [ ] Tree excludes ignored + empty dirs, includes summaries.
- [ ] Root-only sections included (root) or excluded (subdirectory) appropriately.
- [ ] No unauthorized chapters added.
- [ ] Commands in Developer Setup verified locally.

### Additional remembered instructions
- (2025-09-04) MUST-FOLLOW: Maintain cross-file deduplication across `LLM.md` and all `**/LLM*.md` files. Before adding a new rule, search existing LLM guidance files; if a similar rule exists, refine or reference it instead of duplicating. Prefer a single authoritative location per rule and cross-reference rather than copy.
- (2025-09-04) MUST-FOLLOW: Do not create or rely on `.env` / `.env.*` files. Manage all runtime environment variables via `direnv` (`.envrc`) with a tracked `.envrc.example` providing placeholder values.
- (2025-09-04) MUST-FOLLOW: For Python library details, always employ Context7 tools (resolve-library-id then get-library-docs) instead of guessing or relying solely on memory; document fallback if docs cannot be retrieved.
- (2025-09-04) MUST-FOLLOW: When referenced to a GitHub repository for more information about a tool/library, use GitHub repo search tools (not guesses) to inspect code/docs. For arbitrary web pages, use the fetch or puppeteer tools to retrieve/inspect content instead of relying on memory.
