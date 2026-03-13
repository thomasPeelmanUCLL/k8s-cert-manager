# cert-manager GitOps Wiki

> This file mirrors the GitHub Wiki. Keep both in sync.

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Secrets Setup](#secrets)
4. [Deploying](#deploying)
5. [Updating](#updating)
6. [Helm Values Reference](#helm-values-reference)
7. [Troubleshooting](#troubleshooting)
8. [Useful Commands](#useful-commands)

---

## Overview

This repository manages the lifecycle of [cert-manager](https://cert-manager.io) on a Kubernetes cluster using Helm and GitHub Actions.  
The design is **manual-first** — you decide when to deploy — but the weekly update check ensures you never miss a new release.

---

## Prerequisites

| Tool | Minimum version |
|------|-----------------|
| Kubernetes | 1.25+ |
| Helm | 3.x |
| kubectl | Matching cluster version |
| GitHub Actions | Enabled on repo |

---

## Secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret name | Description |
|-------------|-------------|
| `KUBECONFIG` | Full contents of your `~/.kube/config` (base64 or raw YAML) |

For K3s clusters, you can get the kubeconfig with:

```bash
cat /etc/rancher/k3s/k3s.yaml
```

Replace `127.0.0.1` with the public IP or hostname of your cluster API.

---

## Deploying

### Via GitHub Actions (recommended)

1. Navigate to **Actions → Deploy cert-manager → Run workflow**
2. Optionally enter a specific chart version (e.g. `v1.17.0`)
3. Leave blank to automatically resolve the latest available version
4. Click **Run workflow** and monitor the logs

### Locally

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true \
  -f helm/values.yaml
```

---

## Updating

Version management follows this priority:

1. **Workflow input** — version entered at dispatch time wins
2. **`helm/values.yaml` chartVersion** — used if set (non-empty string)
3. **Latest** — resolved live from Helm repo if both above are empty

The `update-check.yml` workflow runs weekly and opens a PR when a newer chart version is found (only when `chartVersion` is pinned).  
Merge the PR after reviewing the [cert-manager changelog](https://github.com/cert-manager/cert-manager/releases).

To switch to always-latest mode, set `chartVersion: ""` in `helm/values.yaml`.

---

## Helm Values Reference

| Key | Default | Description |
|-----|---------|-------------|
| `chartVersion` | `""` | Pin a chart version; empty = latest |
| `installCRDs` | `true` | Install CRDs as part of release |
| `replicaCount` | `1` | Controller replicas |
| `resources.requests.cpu` | `10m` | CPU request |
| `resources.requests.memory` | `64Mi` | Memory request |
| `prometheus.enabled` | `false` | Enable Prometheus scraping |
| `global.logLevel` | `2` | Controller log verbosity (1–6) |

---

## Troubleshooting

### CRDs already exist

If you see `Error: INSTALLATION FAILED: ... already exists`, CRDs from a previous install are present.

```bash
# Option A: delete and reinstall
kubectl delete -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.crds.yaml

# Option B: upgrade instead of install (the workflow uses upgrade --install, so this shouldn't happen)
```

### Webhook not ready

```bash
kubectl describe pod -n cert-manager -l app=cert-manager-webhook
kubectl logs -n cert-manager -l app=cert-manager-webhook
```

### Check cert-manager pods

```bash
kubectl get pods -n cert-manager
kubectl get events -n cert-manager --sort-by='.lastTimestamp'
```

---

## Useful Commands

```bash
# List installed releases
helm list -n cert-manager

# Show current values
helm get values cert-manager -n cert-manager

# Uninstall (keeps CRDs)
helm uninstall cert-manager -n cert-manager

# Remove CRDs manually
kubectl get crds | grep cert-manager | awk '{print $1}' | xargs kubectl delete crd

# Check certificate status
kubectl get certificate -A
kubectl describe certificate <name> -n <namespace>

# Check ClusterIssuers
kubectl get clusterissuer
```
