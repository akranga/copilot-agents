# Deployment LLM Instructions (deploy/)

Purpose: Guidance for running the LiteLLM stack defined in this directory. These rules scope ONLY to deployment-related workflows (containers, local orchestration). They supplement root `LLM.md` (do not duplicate general conventions).

## Stack Overview
- Orchestrator: `docker-compose` (`docker-compose.yaml`).
- Services:
  - `postgres` (required persistence for LiteLLM; no Redis by design here).
  - `wait-ollama` (one-shot readiness gate ensuring Ollama API is reachable before starting LiteLLM).
  - `litellm` (application gateway / router using `litellm-config.yaml`).
- External dependency: A running Ollama daemon (local host or remote) providing models referenced in `litellm-config.yaml` (e.g., `OLLAMA_BASE_URL`).

## Local Run Procedure
1. Ensure Docker engine is available. Preferred on macOS: `colima` (lightweight) else Docker Desktop.
2. (macOS only) Start Colima if installed and not running (skip on other OS). Do NOT attempt to install Colima automatically:
   ```shell
   if [ "$(uname)" = "Darwin" ] && command -v colima >/dev/null 2>&1; then
     colima status >/dev/null 2>&1 || colima start
   fi
   ```
3. Export / load environment via `direnv` (place overrides like `LITELLM_MASTER_KEY`, `OLLAMA_BASE_URL` in your local `.envrc`). Do NOT create `.env` files.
4. Launch stack:
   ```shell
   docker compose -f deploy/docker-compose.yaml up -d
   ```
5. Verify health:
   ```shell
   docker compose -f deploy/docker-compose.yaml ps
   curl -s http://localhost:4000/health || true
   ```

## Environment Variables
Key variables (recommended to set in `.envrc`):
- `LITELLM_MASTER_KEY` – master key for LiteLLM (defaults to `dev-master-key` if unset; override in real usage).
- `OLLAMA_BASE_URL` – URL to Ollama (default `http://host.docker.internal:11434`). Adjust if running Ollama in another container / host.

`DATABASE_URL` is auto-derived inside the compose file; adjust only if modifying Postgres service names or credentials.

## Customization Tips
- Change exposed LiteLLM port: modify `litellm` service `ports` mapping (host side before colon).
- Persist different Postgres path: rename `postgres_data` volume (ensure uniqueness) or bind mount a host directory.
- Add Redis later (if budget / caching required): create a new service & supply its env vars to LiteLLM—keep this documented here when done.

## Operational Notes
- The `wait-ollama` service exits successfully once `/api/tags` responds; if it fails repeatedly (non-zero exit), LiteLLM won't start (due to dependency condition). Investigate Ollama accessibility.
- To force a clean restart including volume retention:
  ```shell
  docker compose -f deploy/docker-compose.yaml down && \
  docker compose -f deploy/docker-compose.yaml up -d --pull always
  ```
- To remove persisted database data entirely:
  ```shell
  docker compose -f deploy/docker-compose.yaml down -v
  ```

## Colima Notes (macOS)
- Default resources may be minimal; increase for model-heavy usage:
  ```shell
  colima stop
  colima start --cpu 4 --memory 8 --disk 50
  ```
- If using GPU acceleration (Apple Silicon), consult Colima docs; not required for basic LiteLLM routing.

---
Revision history:
- v0.1 (2025-09-04): Initial deployment instructions added.
 - v0.2 (2025-09-04): Removed "Future Enhancements" section per global rule prohibiting speculative sections.
