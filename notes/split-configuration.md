# Splitting b-ops: Configuration Layer Analysis

Analysis date: 2026-02-03

## Overview

This layer covers repository-level configuration files that define how the
project operates: encryption rules, AI agent guidance, and project metadata.

- **SOPS**: No extraction needed. business-operations won't contain secrets.
  Configuration stays in deployment repositories (b-ops, demo-ops).
- **AGENTS.md**: Skipped for now. Contributor information will be added after
  the split is complete and may incorporate parts of this analysis.

## Current Structure

```
b-ops/
├── .sops.yaml      # SOPS encryption rules
├── AGENTS.md       # AI agent guidance
├── README.md       # Project overview
└── TODO.md         # Task tracking
```

## SOPS Configuration

**No extraction needed for business-operations**: The product repository will not
contain encrypted secrets. SOPS configuration stays in deployment repositories
(b-ops, demo-ops) where actual secrets live. This analysis is kept for reference
when creating demo-ops.

`.sops.yaml` defines encryption rules per cluster:

```yaml
creation_rules:
  # Per-cluster rules with specific age keys
  - path_regex: kubernetes/cluster-0/.*\.sops\.ya?ml
    age: >-
      age1...(flux key),
      age1...(personal key)

  - path_regex: kubernetes/cluster-0-acceptance/.*\.sops\.ya?ml
    age: ...

  - path_regex: kubernetes/cluster-exp/.*\.sops\.ya?ml
    age: ...

  - path_regex: kubernetes/recovery/.*\.sops\.ya?ml
    age: ...

  - path_regex: kubernetes/dev-env/.*\.sops\.ya?ml
    age: ...

  # Fallback rule
  - path_regex: .*\.sops\.ya?ml
    age: >-
      age1...(personal key only)
```

### Key Structure

Each cluster has two age keys:
1. **Flux key**: Used by Flux controller for decryption in-cluster
2. **Personal key**: For local editing/decryption

Plus a PGP key as backup for all clusters.

## AGENTS.md

**Skipped for now**: Contributor information will be added after the split is
complete. This analysis is kept for reference.

Comprehensive AI guidance covering:

- **Formatting**: 80 char line wrap
- **Repository context**: Purpose, key directories
- **Environments**: cluster-0, cluster-0-acceptance, cluster-exp, dev-env, recovery
- **Secrets and safety**: SOPS rules, Vault preference
- **Working style**: Read docs first, validate locally, ADRs for decisions
- **Reliability notes**: Known issues, backup decisions
- **Useful commands**: Task targets, kubectl, Flux build

This is a valuable reference that documents operational knowledge.

## Extraction Analysis

### Publishable (No Secrets)

| File | Notes |
|------|-------|
| AGENTS.md | Sanitize internal URLs, otherwise valuable |
| README.md | Project overview, sanitize references |

### Template-able

| File | Notes |
|------|-------|
| .sops.yaml | Structure is reusable, keys are not |

### Must Stay Private

| Content | Reason |
|---------|--------|
| Age public keys | Identifies encryption recipients |
| PGP key ID | Personal identifier |
| Cluster-specific paths | Reveals internal structure |

## SOPS Template for business-operations

Create a template `.sops.yaml`:

```yaml
creation_rules:
  # Production cluster
  - path_regex: kubernetes/cluster-prod/.*\.sops\.ya?ml
    encrypted_regex: "^(data|stringData)$"
    age: >-
      age1...(flux-cluster-prod),
      age1...(operator-key)

  # Development cluster
  - path_regex: kubernetes/cluster-dev/.*\.sops\.ya?ml
    encrypted_regex: "^(data|stringData)$"
    age: >-
      age1...(flux-cluster-dev),
      age1...(operator-key)

  # Fallback
  - path_regex: .*\.sops\.ya?ml
    age: >-
      age1...(operator-key)
```

With documentation on:
- How to generate age keys
- How to configure Flux SOPS integration
- Key rotation procedures

## AGENTS.md for business-operations

Create a sanitized version:

```markdown
# Guidance for AI Agents

## Repository context
- Business Operations platform; GitOps/IaC for clusters and shared services.
- Source of truth to rebuild environments from scratch.

## Key directories
- `kubernetes/` - Cluster manifests
- `infrastructure/ansible` - Bootstrap automation
- `infrastructure/nixos` - OS provisioning flake

## Environments
- Clusters under `kubernetes/`: production, acceptance, experimental, dev
- Each cluster has Flux bootstrap/config and app manifests

## Secrets and safety
- Secrets use SOPS with age encryption
- Prefer Vault-backed secrets via ExternalSecret

## Working style
- Read docs first, validate locally
- Record decisions as ADRs
```

## Extraction Options

### Option 1: Document Patterns

Create guides in business-operations:
```
business-operations/
└── docs/
    ├── sops-setup.md       # How to configure SOPS
    ├── agents-template.md  # Template for AGENTS.md
    └── secrets-guide.md    # Secret management patterns
```

### Option 2: Provide Templates

```
business-operations/
└── templates/
    ├── .sops.yaml.template
    ├── AGENTS.md.template
    └── README.md.template
```

Users copy and fill in their values.

### Option 3: Generator Script

Create a script that generates configuration:
```bash
./init-cluster.sh --name prod --age-key age1xxx...
# Generates .sops.yaml entry, cluster directory structure
```

## Recommended First Steps

No action needed for business-operations. Documentation regarding secrets
handling will be created when working on the demo-ops repository.
