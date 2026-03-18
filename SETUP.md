# Flux Cluster Setup Guide

This repository is a template for creating new Flux CD cluster instances. Follow these steps to set up your cluster.

## Prerequisites

- Kubernetes cluster (1.28+)
- `kubectl` configured to access your cluster
- `flux` CLI installed ([installation guide](https://fluxcd.io/flux/installation/))
- GitHub account and repository created from this template

## Step 1: Create Your Instance Repository

1. Click "Use this template" on GitHub to create a new repository
2. Name it `flux-cluster-<name>` (e.g., `flux-cluster-prod`)
3. Clone your new repository locally

## Step 2: Edit Configuration

### Edit `platform-vars.yaml`

Open `clusters/production/platform-vars.yaml` and replace all `CHANGEME` values:

```yaml
data:
  DOMAIN: "your-domain.com"              # Your actual domain
  ACME_EMAIL: "admin@your-domain.com"    # Your email for Let's Encrypt
  GCP_PROJECT: "your-gcp-project"        # Your GCP project (or leave empty)
  INGRESS_TYPE: "LoadBalancer"           # Or "NodePort" for local dev
  GRAFANA_ADMIN_PASSWORD: "secure-password"  # Change this!
```

### Create `platform-secrets` Secret

Create a Secret with your OAuth credentials:

```bash
kubectl create namespace flux-system

kubectl create secret generic platform-secrets \
  --namespace=flux-system \
  --from-literal=OAUTH_CLIENT_ID='your-google-client-id' \
  --from-literal=OAUTH_CLIENT_SECRET='your-google-client-secret' \
  --from-literal=OAUTH_COOKIE_SECRET='16-byte-random-string'
```

**Using Sealed Secrets (Recommended):**

```bash
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Create sealed secret
kubectl create secret generic platform-secrets \
  --namespace=flux-system \
  --from-literal=OAUTH_CLIENT_ID='your-google-client-id' \
  --from-literal=OAUTH_CLIENT_SECRET='your-google-client-secret' \
  --from-literal=OAUTH_COOKIE_SECRET='16-byte-random-string' \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > clusters/production/platform-secrets-sealed.yaml

# Add to kustomization
echo "  - platform-secrets-sealed.yaml" >> clusters/production/kustomization.yaml

# Commit and push
git add clusters/production/platform-secrets-sealed.yaml clusters/production/kustomization.yaml
git commit -m "Add sealed secrets"
git push
```

### Update `gotk-sync.yaml`

Edit `clusters/production/flux-system/gotk-sync.yaml` and replace the repository URL:

```yaml
spec:
  url: ssh://git@github.com/YOUR-ORG/YOUR-REPO  # Replace with your repo
```

## Step 3: Remove Unwanted Modules

If you don't need certain modules, remove their files from `clusters/production/modules/` and remove their references from `clusters/production/kustomization.yaml`.

For example, to disable tracing and dashboard:

```bash
rm clusters/production/modules/tracing.yaml
rm clusters/production/modules/dashboard.yaml
```

Edit `clusters/production/kustomization.yaml` and remove:
```yaml
  - modules/tracing.yaml
  - modules/dashboard.yaml
```

## Step 4: Bootstrap Flux

Run Flux bootstrap to install Flux and sync your cluster:

```bash
flux bootstrap github \
  --owner=YOUR-GITHUB-ORG \
  --repository=YOUR-REPO-NAME \
  --branch=main \
  --path=clusters/production \
  --personal
```

This will:
- Install Flux controllers in your cluster
- Create a deploy key in your GitHub repository
- Commit `gotk-components.yaml` if it doesn't exist
- Start reconciling your cluster configuration

## Step 5: Verify Installation

Check that all Kustomizations are ready:

```bash
flux get all -A
kubectl get kustomizations -n flux-system
```

Wait for all modules to reconcile (this may take 10-15 minutes):

```bash
watch kubectl get kustomizations -n flux-system
```

## Step 6: Access Services

Once everything is running, access your services:

- **Grafana**: https://grafana.your-domain.com
  - Username: `admin`
  - Password: (value from `platform-vars.yaml`)

- **Jaeger** (if enabled): https://jaeger.your-domain.com

- **Dashboard** (if enabled): https://dashboard.your-domain.com

- **Prometheus** (if enabled): https://prometheus.your-domain.com

## Adding Applications

To deploy applications, add GitRepository + Kustomization files under `clusters/production/apps/`:

```yaml
# clusters/production/apps/my-app-source.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/my-org/my-app
  ref:
    branch: main
---
# clusters/production/apps/my-app.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./overlays/production
  interval: 10m
  prune: true
  dependsOn:
    - name: platform-autoscaling
  postBuild:
    substituteFrom:
      - kind: ConfigMap
        name: platform-vars
```

## Upgrading the Platform

To upgrade to a new version of the flux-platform template:

1. Check available versions: https://github.com/afloren-dev/flux-platform/tags
2. Edit `clusters/production/platform-source.yaml` and update the tag:
   ```yaml
   ref:
     tag: v1.1.0  # New version
   ```
3. Commit and push
4. Flux will automatically reconcile the new version

## Troubleshooting

### Check Flux status
```bash
flux get all -A
```

### Check specific Kustomization
```bash
flux get kustomization platform-core -n flux-system
kubectl describe kustomization platform-core -n flux-system
```

### Reconcile manually
```bash
flux reconcile kustomization platform-core -n flux-system
```

### View logs
```bash
flux logs --all-namespaces --follow
```

## Support

- Flux documentation: https://fluxcd.io/docs/
- Platform template: https://github.com/afloren-dev/flux-platform
- Issues: https://github.com/afloren-dev/flux-cluster-template/issues
