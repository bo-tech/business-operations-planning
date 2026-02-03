# Splitting b-ops: Tooling Layer Analysis

Analysis date: 2026-02-03

## Overview

This layer covers Taskfile automation, utilities, and Renovate configuration.

- **Root tasks**: Stay in `b-ops` (site-specific WOL/shutdown for bare metal machines; document pattern later)
- **Cluster tasks**: `push`, `info`, `forward-gitea` patterns need experiment to find best multi-repo split; `bootstrap` replaced by Ansible; `k3s-*` not needed
- **Utilities**: Stay in `b-ops` for now
- **Renovate**: Stays in `b-ops`; `business-operations` will need new config for GitHub (backlog)

## Current Structure

```
b-ops/
├── Taskfile.yaml              # Root task definitions
├── .tasks/
│   ├── cluster-taskfile.yaml  # Cluster-specific tasks (included per-cluster)
│   └── push-into-gitea.sh     # Helper script for git push
├── utils/
│   └── secret-to-external-secret.jq  # Convert Secret to ExternalSecret
└── renovate.json5             # Dependency update configuration
```

## Taskfile

### Root Tasks (`Taskfile.yaml`)

**No extraction needed.** Site-specific tasks stay in `b-ops`. The WOL pattern for
bare metal machines may be documented later.

```yaml
tasks:
  wakeup-exp-machines:   # Wake cluster via WoL
  shutdown-exp-machines: # Shutdown cluster
```

Simple wrappers around Ansible playbooks.

### Cluster Tasks (`.tasks/cluster-taskfile.yaml`)

Included from each cluster directory. Provides:

| Task | Description | Status |
|------|-------------|--------|
| `push` | Push current HEAD to cluster's Gitea | Document pattern |
| `info` | Show kubectl cluster-info and nodes | Document pattern |
| `forward-gitea` | Port-forward to Gitea service | Document pattern |
| `bootstrap` | Full Flux bootstrap sequence | Replaced by Ansible `bootstrap-cluster.yaml` |
| `trigger-backup` | Trigger backup from schedules | Not needed (k8up no longer used) |
| `k3s-renew` | Experimental k3s on Docker | Not needed |
| `k3s-stop` | Stop experimental k3s | Not needed |

### Bootstrap Task Detail

The `bootstrap` task encapsulates the full bootstrap flow:
```yaml
- kubectl apply --server-side --kustomize ./bootstrap
- sops --decrypt ./bootstrap/age-key.sops.yaml | kubectl apply -f -
- sops --decrypt ./bootstrap/gitea/secret-bootstrap.sops.yaml | kubectl apply -f -
- kubectl wait for Gitea deployment
- kubectl wait for repo creation job
- kubectl apply cluster-settings
- sops --decrypt secret-cluster-settings | kubectl apply -f -
- push-into-gitea
- kubectl apply --kustomize ./flux/config
```

## Utilities

### `secret-to-external-secret.jq`

Converts Kubernetes Secret to ExternalSecret (Vault-backed):

```bash
kubectl get secret/your-secret -o yaml | yq -f secret-to-external-secret.jq -y
```

- Transforms Secret to ExternalSecret CRD
- References ClusterSecretStore "vault"
- Maps data keys to Vault paths
- Hardcoded path prefix: `apps/gitlab/`

## Renovate Configuration

```json5
{
  "extends": ["local>bv/code/renovate/default"],
  // Custom managers for kubernetes/*.yaml files
  flux: { ... },
  helm-values: { ... },
  kubernetes: { ... },
  customManagers: [
    // OCI dependency regex matcher
  ]
}
```

**Dependency**: Extends internal `bv/code/renovate/default` config.

## Extraction Analysis

### Fully Reusable

| Component | Notes |
|-----------|-------|
| cluster-taskfile.yaml | Generic pattern (needs path adjustments) |
| secret-to-external-secret.jq | Useful utility (hardcoded Vault path) |
| Bootstrap task pattern | Documented workflow |

### Needs Modification

| Component | Issue | Fix |
|-----------|-------|-----|
| Taskfile.yaml | References `inventory-exp.yaml` | Parameterize inventory |
| secret-to-external-secret.jq | Hardcoded `apps/gitlab/` path | Make configurable |
| renovate.json5 | Extends internal config | Create public base config |

### Must Stay Private

| Component | Reason |
|-----------|--------|
| push-into-gitea.sh | Uses cluster-specific secrets |
| Gitea password extraction | SOPS-encrypted secrets |

## Extraction Options

### Option 1: Extract Task Patterns

Document task patterns in business-operations:
```
business-operations/
└── tooling/
    ├── taskfile-patterns.md    # How to structure Taskfiles
    └── cluster-taskfile.yaml   # Template for cluster tasks
```

Users copy and adapt for their clusters.

### Option 2: Parameterized Taskfile

Create reusable Taskfile with variables:
```yaml
vars:
  CLUSTER_NAME: ""
  INVENTORY_FILE: ""

tasks:
  wakeup:
    cmds:
      - ansible-playbook -i {{.INVENTORY_FILE}} wakeup-machines.yaml
```

### Option 3: Extract Utilities

Publish utilities as standalone tools:
```
business-operations/
└── utils/
    ├── secret-to-external-secret.jq
    └── backup-from-schedule.jq
```

With documentation on customization.

## Renovate Strategy

Options for public Renovate config:

1. **Create public base config** in business-operations
2. **Document patterns** for Flux/Helm/OCI detection
3. **Remove internal dependency** by inlining rules

## Recommended First Steps

1. **Document the bootstrap task** as a procedure (already in AGENTS.md)
2. **Extract jq utilities** with parameterization
3. **Create public Renovate base** config

Low risk, high documentation value.
