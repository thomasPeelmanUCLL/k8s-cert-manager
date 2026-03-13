# k8s-cert-manager

GitOps-style deployment of [cert-manager](https://cert-manager.io) on Kubernetes via Helm.

## Features

- Installs cert-manager with CRDs via Helm
- GitHub Actions workflow that checks for new Helm chart releases weekly
- Manual `workflow_dispatch` trigger to deploy at any time
- Stores Helm values in `values.yaml` for easy customisation

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       ├── deploy.yml         # Manual deploy + auto-update check
│       └── update-check.yml   # Weekly latest-version check
├── helm/
│   └── values.yaml            # Helm values for cert-manager
├── docs/
│   └── wiki.md                # Full usage & ops guide (mirrors GitHub Wiki)
└── README.md
```

## Quick Start

### Prerequisites

- `kubectl` configured against your cluster
- `helm` v3 installed
- GitHub Actions secrets set (see [Wiki](docs/wiki.md#secrets))

### Manual Deploy via GitHub Actions

1. Go to **Actions → Deploy cert-manager**
2. Click **Run workflow**
3. Optionally pin a specific chart version (leave blank for latest)

### Manual Local Deploy

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true \
  -f helm/values.yaml
```

## Updating

The `update-check.yml` workflow runs every Monday at 08:00 UTC.  
It compares the latest Helm chart version against the value stored in `helm/values.yaml`.  
If a new version is available it opens a PR bumping the version — you review and merge manually.

## License

MIT
