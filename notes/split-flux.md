# Splitting b-ops: Flux Layer Analysis

Analysis date: 2026-02-03

## Overview

This layer covers Kubernetes manifests, Kustomize templates, and HelmReleases.

- **Kluctl migration**: Deferred. Need something working first before larger restructuring.
- **Approach**: Experiment to find the best multi-repo pattern.
- **Candidates**: Flux multi-source, git submodule (interim), possibly others to discover.

## Goal

Split b-ops into three repositories:
- **business-operations**: Public product (reusable infrastructure patterns)
- **demo-ops**: Public example deployment
- **b-ops**: Private production use

## Current Architecture

The b-ops repo follows a monolithic GitOps pattern:

1. Single `GitRepository` points to `b-ops.git`
2. All Kustomizations reference paths relative to repo root: `./kubernetes/cluster-0/...`
3. Templates referenced via deep relative paths: `../../../../../../templates/volume/components/...`
4. Variable substitution via `cluster-settings` ConfigMap provides instance-specific values

### Coupling Points

| Coupling Point | Example | Challenge |
|----------------|---------|-----------|
| Hardcoded cluster paths | `path: ./kubernetes/cluster-0/apps/...` | Every Kustomization assumes single-repo |
| Relative template refs | `components: - ../../../../../../templates/...` | 6-level deep relative paths |
| Single GitRepository | `bv-ops-kubernetes` | All sources from one repo |
| Inline variable refs | `${cluster_domain}`, `${use_gitlab_system}` | Fine, works across repos |

## Proposed Repository Structure

```
business-operations/          # Public - the "product"
├── kubernetes/
│   ├── base/                 # Reusable base infrastructure
│   │   ├── flux-system/      # Flux bootstrap patches
│   │   ├── network/          # Ingress-nginx, CoreDNS
│   │   ├── storage/          # Rook-Ceph, VolSync patterns
│   │   ├── secrets/          # Vault, External-Secrets
│   │   └── monitoring/       # Prometheus stack
│   ├── templates/            # Reusable Kustomize components
│   │   ├── volume/           # VolSync backup/restore
│   │   ├── cnpg/             # CloudNative-PG patterns
│   │   └── app/              # Common app patterns
│   └── apps/                 # Reference app deployments
│       └── examples/         # Sanitized examples
├── infrastructure/
│   └── nixos/                # NixOS reference config (sanitized)
└── docs/
    └── decisions/            # ADRs (already public-safe)

demo-ops/                     # Public - example deployment
├── kubernetes/
│   ├── cluster-demo/         # Example cluster configuration
│   │   ├── flux/
│   │   │   ├── config/       # Points to BOTH repos
│   │   │   └── vars/         # Example cluster-settings
│   │   └── apps/             # Demo apps using business-operations templates
│   └── README.md             # "How to use business-operations"
└── infrastructure/           # Example infra config

b-ops/                        # Private - production use
├── kubernetes/
│   ├── cluster-0/            # Production cluster (private)
│   ├── cluster-0-acceptance/ # Acceptance cluster
│   └── ...                   # All existing structure
└── infrastructure/           # Private NixOS/Ansible config
```

## Migration Options

### Option A: Flux Multi-Source (Recommended)

business-operations becomes a second `GitRepository` in Flux. Kustomizations can reference bases from either repo. Templates migrate one-by-one.

Configuration change in b-ops:
```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: business-operations
  namespace: flux-system
spec:
  url: https://github.com/yourorg/business-operations.git
  ref:
    branch: main
```

Kustomization using remote base:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
components:
  # OLD: relative path within same repo
  # - ../../../../../../templates/volume/components/backup-volsync
  # NEW: reference from business-operations repo via Flux
  - https://github.com/yourorg/business-operations//kubernetes/templates/volume/components/backup-volsync?ref=v0.1.0
```

**Pros:**
- Incremental: migrate one template at a time
- No paradigm change (stays pure Flux/Kustomize)
- Version pinning via `?ref=` tag
- Works today with existing tools

**Cons:**
- HTTPS URLs in Kustomizations (not as clean as relative paths)
- Need to tag releases in business-operations
- Two repos to keep in sync during transition

### Option B: Git Submodule

business-operations added as submodule at `kubernetes/shared/`. Relative paths change to `../../../shared/templates/...`.

**Pros:**
- Relative paths still work
- Single `git pull --recurse-submodules` updates everything
- Clear version pinning (submodule commit)

**Cons:**
- Git submodules are notoriously painful
- Flux doesn't handle submodules well (needs workarounds)
- CI/CD complexity increases

### Option C: Kluctl Migration

Restructure around Kluctl deployments. Kluctl has native multi-repo inclusion and better environment/variant management.

**Pros:**
- Aligns with long-term preference for Kluctl
- Better multi-environment support
- Cleaner separation of concerns

**Cons:**
- Significant migration effort
- Learning curve for new paradigm
- Deferred benefit (no incremental wins now)

### Option D: Helm Library Charts

Convert templates to Helm library charts, publish to business-operations OCI registry, consume as chart dependencies.

**Pros:**
- Well-established pattern
- Clear versioning via chart versions
- Works with existing HelmRelease model

**Cons:**
- Templates use Kustomize components (not Helm)
- Would need to rewrite backup/restore patterns
- Significant effort for limited benefit

## Comparison

| Option | Effort | Incrementality | Long-term fit |
|--------|--------|----------------|---------------|
| **A: Flux Multi-Source** | Low | High | Good |
| B: Git Submodule | Medium | Medium | Poor |
| C: Kluctl Migration | High | Low | Best |
| D: Helm Library | High | Low | Moderate |

## Recommended First Step: Experiment

Before committing to an approach, run an experiment to validate options.

### Experiment Goals

1. **Explore**: What other options/approaches exist beyond the four listed?
2. **Evaluate**: Which approaches are most promising for our use case?
3. **Try out**: Test the most promising options with a simple template extraction

### Experiment Candidates

- **Flux multi-source** (Option A): Add `business-operations` as second GitRepository
- **Git submodule interim** (Option B): Submodule with relative paths, assess Flux compatibility
- **Other approaches**: Discover during exploration phase

### Test Case: VolSync Templates

Use VolSync templates as the test case:
- Self-contained Kustomize components
- Used by ~10+ apps (high reuse value)
- No secrets involved (safe to publish)
- Clear interface (PVC -> backup/restore configuration)

### Decision Point

After the experiment:
- Document findings in notes
- Select approach for production use
- Proceed with full template extraction

Kluctl migration (Option C) remains the long-term preference but is deferred until
something is working and public first.
