# linkpulse-argo-gitops

GitOps deployment repository for **Linkpulse** on Kubernetes (k3s) managed via Argo CD.

## Directory Structure

```text
linkpulse-argo-gitops/
├── base/
│   ├── api/                 # API deployment, service, HPA, PDB
│   ├── worker/              # Worker deployment (Recreate)
│   ├── migrate/             # PreSync Alembic database migration Job
│   ├── postgres/            # StatefulSet with 2Gi local-path PVC
│   ├── redis/               # Deployment with readiness probe
│   ├── configmap.yaml       # Non-secret environment variables
│   ├── ingress.yaml         # Traefik ingress configuration
│   ├── networkpolicy.yaml   # Default-deny namespace network policy
│   ├── secret-example.yaml  # Reference schema for secrets
│   └── kustomization.yaml
├── overlays/
│   └── oci/
│       ├── kustomization.yaml  # OCI cluster overlay with image tag & patches
│       └── config.env
└── argocd-application.yaml     # Argo CD Application manifest
```

## Setup & Deployment

### 1. Create Secret Out-of-Band

Create the `linkpulse-secrets` secret before syncing:

```bash
kubectl create namespace linkpulse --dry-run=client -o yaml | kubectl apply -f -

kubectl create secret generic linkpulse-secrets \
  --namespace linkpulse \
  --from-literal=DATABASE_URL="postgresql+psycopg://postgres:postgres@postgres:5432/linkpulse" \
  --from-literal=API_KEY="your-secure-api-key" \
  --from-literal=POSTGRES_PASSWORD="postgres"
```

### 2. Deploy with Argo CD

Apply the Argo CD Application:

```bash
kubectl apply -f argocd-application.yaml
```
