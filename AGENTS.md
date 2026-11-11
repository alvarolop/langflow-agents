# AGENTS.md

Langflow (official image) setup for building AI agents. Flows are versioned as JSON.

## Layout
- `flows/` — exported Langflow flow JSON (loaded at startup via `LANGFLOW_LOAD_FLOWS_PATH`)
- `deploy/local/compose.yaml` — Podman/Docker Compose
- `deploy/openshift/application-langflow.yaml` — Argo CD Application (`langflow-ide` chart, values inline)
- `.env.example` — env template; real `.env` is git-ignored

## Commands
- Start: `podman-compose -f deploy/local/compose.yaml --env-file .env up`
- UI: http://localhost:7860 (login from `.env`)

## Rules
- Never commit secrets or API keys, neither in `.env`, flow JSON nor values files. Flows must reference global variables (e.g. `GOOGLE_API_KEY`), never literal keys.
- Pin the Langflow image version; don't use `latest`.
- After editing a flow in the UI, export it to `flows/<name>.json` and review the diff for leaked keys.
- Use Context7 / official docs (docs.langflow.org) for Langflow config; env var names change between versions.
