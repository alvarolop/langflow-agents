# Langflow Agents

Minimal, reproducible setup of the **official Langflow image** to build AI agents, with flows versioned as code. Uses Google Gemini for testing. Local (Podman/Docker) and OpenShift (Helm) flavours.

## Local

```bash
cp .env.example .env        # fill GEMINI_API_KEY
podman-compose -f deploy/local/compose.yaml --env-file .env up
# or: docker compose -f deploy/local/compose.yaml --env-file .env up
```

- UI: http://localhost:7860
- User: `admin@example.org` / Password: `admin` (override in `.env`)
- `GEMINI_API_KEY` is available in flows as the global variable `GOOGLE_API_KEY`.

Stop / recreate:

```bash
podman-compose -f deploy/local/compose.yaml --env-file .env down
```

## Flows as code

- Any `*.json` in `flows/` is loaded at startup.
- To save a flow you built in the UI: flow menu → **Export** → save to `flows/`, then restart.

## OpenShift

Uses the `langflow-ide` chart (backend + frontend, SQLite on a PVC). Installing it needs cluster-admin: the chart creates a ClusterRole so its ServiceAccount can use the `nonroot-v2` SCC.

```bash
helm repo add langflow https://langflow-ai.github.io/langflow-helm-charts && helm repo update
oc new-project langflow-agents
oc create secret generic langflow-agents-env --from-env-file=.env   # .env needs LANGFLOW_SECRET_KEY
helm install langflow-agents langflow/langflow-ide -f deploy/openshift/values.yaml
oc get svc                          # find the frontend service (port 8080)
oc expose svc/<frontend-service>    # create the Route
```

> Untested on a cluster yet. The values render correctly with `helm template`. Flows in `flows/` are not auto-loaded here: import them from the UI.

## License
MIT
