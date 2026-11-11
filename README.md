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

Stop (the container stays, stopped, visible in Podman Desktop) and start again:

```bash
podman-compose -f deploy/local/compose.yaml --env-file .env stop
podman-compose -f deploy/local/compose.yaml --env-file .env start
```

Ctrl+C on a foreground `up` does the same as `stop`. Avoid `down`: it removes the container. Data survives either way (named volume `langflow_data`).

## Flows as code

- Any `*.json` in `flows/` is loaded at startup.
- To save a flow you built in the UI: flow menu → **Export** → save to `flows/`, then restart.

## OpenShift (Argo CD)

Dev instance (IDE) deployed as an Argo CD `Application` ([`deploy/openshift/application-langflow.yaml`](deploy/openshift/application-langflow.yaml)) that uses the official [`langflow-ide` Helm chart](https://docs.langflow.org/deployment-kubernetes-dev) (0.1.2, Langflow 1.13.1): backend with SQLite on a PVC plus frontend, with the values inline and `CreateNamespace=true` for the `langflow` namespace. It runs with the default `restricted-v2` SCC; no SCC changes and no cluster-scoped objects (the chart's ServiceAccount/ClusterRole are disabled). The chart's Ingress is turned into an edge-TLS Route by OpenShift.

1. **Apply the Application** (`$CLUSTER_DOMAIN` is substituted with `envsubst`):
```bash
export CLUSTER_DOMAIN=$(oc get dns cluster -o jsonpath='{.spec.baseDomain}')
envsubst < deploy/openshift/application-langflow.yaml | oc apply -f -
```

2. **Apply the Secret** with the credentials from your local `.env`. It is generated on the fly, so credentials never enter git or the Application:
```bash
oc create secret generic langflow-env --from-env-file=.env -n langflow \
  --dry-run=client -o yaml | oc apply -f -
```

The `.env` needs `LANGFLOW_SECRET_KEY`. The backend shows `CreateContainerConfigError` until this Secret exists, then recovers by itself. After changing `.env`, re-apply it and run `oc rollout restart statefulset/langflow-service -n langflow`.

Langflow will be available at `https://langflow-langflow.apps.$CLUSTER_DOMAIN`.

> The route is public and the IDE can run Python: set a strong `LANGFLOW_SUPERUSER_PASSWORD` in `.env` before applying the Secret (not `admin`). Flows in `flows/` are not auto-loaded here: import them from the UI.

---

## License
[MIT](LICENSE)
