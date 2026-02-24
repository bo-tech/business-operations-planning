# Flux App Deployment Architecture

Date: 2026-02-24

## Overview

Flux manages deployments through a three-tier hierarchy of Kustomizations,
driven by path-based convention inside `kubernetes/<cluster>/`.

## Tier 1: Source

`flux/config/cluster.yaml` defines:
- **GitRepository** `bv-ops-kubernetes` — points to the in-cluster Gitea mirror
  of b-ops, filtered to only `/kubernetes`
- **Kustomization** `cluster` — reconciles `kubernetes/<cluster>/flux/`,
  depends on Flux itself being ready

## Tier 2: Cluster-Level Orchestration

Four Flux Kustomizations in `flux/`:

| Kustomization      | Path                                 | Wait | Depends On                  |
|---------------------|--------------------------------------|------|-----------------------------|
| `cluster-crds`      | `./kubernetes/cluster-0/crds`        | yes  | (none)                      |
| `cluster-base-apps` | `./kubernetes/cluster-0/base-apps`   | yes  | cluster-crds                |
| `cluster-apps`      | `./kubernetes/cluster-0/apps`        | no   | cluster-base-apps, cluster-crds |
| `cluster-secrets`   | `./kubernetes/${cluster_name}/secrets`| no   | (none)                      |

All four inject SOPS decryption + variable substitution from `cluster-settings`
ConfigMap and `secret-cluster-settings` Secret. A label selector
(`substitution.flux.home.arpa/disabled notin (true)`) allows individual child
Kustomizations to opt out.

## Tier 3: Per-App Kustomizations

Each directory under `apps/`, `base-apps/`, `crds/` is a namespace grouping.
Each namespace has a `kustomization.yaml` listing:
- `./ns` or `./namespace.yaml` (namespace resource)
- `./app-name/ks.yaml` (per-app Flux Kustomization)

The `ks.yaml` files define individual Flux Kustomizations that point to
`app/helmrelease.yaml` or similar, with their own `dependsOn` chains.

## Acceptance Cluster Overlay

`cluster-0-acceptance/flux/kustomization.yaml` reuses cluster-0's definitions:

```yaml
resources:
  - ../../cluster-0/flux/cluster-apps.yaml
  - ../../cluster-0/flux/cluster-base-apps.yaml
  - ../../cluster-0/flux/cluster-crds.yaml
  - ../../cluster-0/flux/cluster-secrets.yaml
  - ../../cluster-0/flux/repositories
  - ./config
  - ./vars
```

The `cluster.yaml` in `config/` overrides the Kustomization path to
`./kubernetes/cluster-0-acceptance/flux`. Cluster settings provide
acceptance-specific IPs, domain (`test-v2.lab.bo-tech.de`), and backup/restore
config.

Since `cluster-apps` and `cluster-base-apps` point to `./kubernetes/cluster-0/...`
(hardcoded in the YAML), the acceptance cluster currently deploys the **exact
same apps** as production — it's a full mirror.

## Current App Inventory

### base-apps (infrastructure)
- kube-system: cilium, snapshot-controller
- network: coredns, ingress-nginx
- openebs
- rook-ceph (acceptance has values overlay for `/dev/vdb`)
- secrets: vault, vault-volumes, external-secrets
- backup: volsync
- flux-bootstrap: flux-webhook
- gitlab: gitlab, gitlab-volumes

### crds
- kube-prometheus-stack CRDs
- external-secrets CRDs
- volsync CRDs

### apps (applications)
- cert-manager
- database: cloudnative-pg
- default: hajimari, mayan-edms, taiga6, tryton, unifi
- flux-system: weave-gitops
- gitlab: renovate, squid-ci-cache
- monitoring: grafana, kube-prometheus-stack
- network: echo-server
- private: mayan-edms
- security: authelia, lldap
- test: (currently empty/disabled)

## Cluster Settings Variables

Variable substitution via `cluster-settings` ConfigMap:
- `cluster_name`, `cluster_domain`
- `cluster_ingress_ip`, `cluster_service_host`, `coredns_ip`, `gitlab_ingress_ip`
- `cilium_pool_cidr`, `cilium_static_pool_cidr`
- `cluster_bootstrap_mode` (none/bootstrap/restore)
- `use_gitlab_system` (allows pointing to different GitLab instance)
- Various backup/restore path settings
