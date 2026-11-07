# Flux Cluster Template

> [!WARNING]
> **Retired (2026-11-07).** This repo was part of an experiment with a Flux CD and Knative platform. It was never deployed to a real cluster and is no longer maintained. It will be archived.
>
> afloren.dev apps now deploy from a cloud-neutral app spec (`app.yaml`) through [`afloren-dev/terraform`](https://github.com/afloren-dev/terraform). Kubernetes may come back later as one backend for that spec; see [`docs/kubernetes-backend.md`](https://github.com/afloren-dev/terraform/blob/main/docs/kubernetes-backend.md) there for what we learned here. Don't build on this repo.

Scaffold for creating new Flux CD cluster instances using the [flux-platform](https://github.com/afloren-dev/flux-platform) infrastructure template.

## Quick Start

1. Click "Use this template" to create a new repository
2. Follow the setup guide in [SETUP.md](SETUP.md)
3. Customize your cluster configuration
4. Bootstrap Flux and deploy!

## What's Included

This template provides:

- **Flux Bootstrap**: Pre-configured Flux CD setup for GitOps
- **Platform Integration**: Points to [flux-platform](https://github.com/afloren-dev/flux-platform) for reusable infrastructure modules
- **Modular Architecture**: Enable/disable features by including/excluding module files
- **Variable Substitution**: ConfigMap-based configuration for environment-specific values
- **CI/CD**: GitHub Actions workflow for validation and E2E testing

## Modules Available

All modules are optional except `core` and `mesh`:

- ✅ **core** (required) - Metrics server
- ✅ **mesh** (required) - Istio service mesh
- ⚙️ **certs** (optional) - cert-manager for TLS
- ⚙️ **auth** (optional) - Dex + OAuth2 Proxy
- ⚙️ **monitoring** (optional) - Prometheus + Grafana + Loki
- ⚙️ **tracing** (optional) - Jaeger distributed tracing
- ⚙️ **autoscaling** (optional) - Knative Serving + Eventing
- ⚙️ **dashboard** (optional) - Kubernetes Dashboard

## Configuration

### Required Variables

Edit `clusters/production/platform-vars.yaml`:

- `DOMAIN` - Your domain name
- `ACME_EMAIL` - Email for Let's Encrypt
- `INGRESS_TYPE` - `LoadBalancer` or `NodePort`
- `GRAFANA_ADMIN_PASSWORD` - Grafana password

### Required Secrets

Create `platform-secrets` Secret with:

- `OAUTH_CLIENT_ID` - Google OAuth client ID
- `OAUTH_CLIENT_SECRET` - Google OAuth secret
- `OAUTH_COOKIE_SECRET` - Random 16+ byte string

See [SETUP.md](SETUP.md) for detailed instructions.

## Repository Structure

```
clusters/production/
  flux-system/          # Flux bootstrap files
  platform-source.yaml  # GitRepository pointing to flux-platform
  platform-vars.yaml    # Your configuration values
  modules/              # Module enablement (remove files to disable)
  apps/                 # Your application deployments
  kustomization.yaml    # Root Kustomization
```

## Documentation

- **Setup Guide**: [SETUP.md](SETUP.md)
- **Platform Docs**: [flux-platform](https://github.com/afloren-dev/flux-platform)
- **Flux Docs**: [fluxcd.io](https://fluxcd.io/)

## Support

For issues or questions:
- Platform template: [flux-platform issues](https://github.com/afloren-dev/flux-platform/issues)
- Cluster template: [flux-cluster-template issues](https://github.com/afloren-dev/flux-cluster-template/issues)

## License

MIT
