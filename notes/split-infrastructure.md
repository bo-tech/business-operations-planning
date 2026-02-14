# Splitting b-ops: Infrastructure Layer Analysis

Analysis date: 2026-02-03

## Overview

This layer covers everything from bare metal to Flux handover. Four phases:

1. **Machine deployment**: NixOS via nixos-anywhere
2. **Cluster formation**: k0s cluster setup (tokens, join nodes)
3. **Pre-Flux infrastructure**: Cilium, OpenEBS, Rook-Ceph (deployed via Ansible/Helm)
4. **Flux bootstrap**: Gitea, push code, kick off Flux (from here Flux takes over)

**Approach**: NixOS is the most fundamental layer, so start there. Keep as monorepo
(`business-operations`), further splitting can happen later. Focus on simplest
possible split while staying in a working state.

**Extraction sequence**:
1. Hardware modules first (100% reusable, no secrets, no parameterization)
2. Parameterize k0s-node modules (SSH keys, DNS as inputs)
3. Extract parameterized modules
4. Validate by using in both `b-ops` and `demo-ops`

## Scope

This covers the bare-metal to Kubernetes bootstrap:
- NixOS machine provisioning (`infrastructure/nixos/`)
- Ansible automation (`infrastructure/ansible/`)
- k0s cluster configuration (`infrastructure/k0s/`)

## Current Architecture

### NixOS Flake Structure

```
infrastructure/nixos/
├── flake.nix              # Main flake with 16 host configurations
├── hosts/                 # Per-machine configs (IPs, hostnames, k0s role)
├── modules/               # Reusable system modules
├── machine-classes/       # k0s-node composition (disks + defaults)
└── hardware/              # Hardware-specific configs (Minisforum models, VMs)
```

**Flake Inputs:**
- `nixpkgs`: NixOS 24.11 (stable)
- `disko`: Declarative disk partitioning
- `k0s-nix`: github:johbo/k0s-nix (custom k0s NixOS module)

**Composition Pattern:**
```
hardwareCfg + machineClassCfg + moduleCfg + hostSpecificCfg
```

### Ansible Structure

```
infrastructure/ansible/
├── inventory-*.yaml       # Per-cluster host definitions
├── *.yaml                 # Playbooks (re-create, bootstrap, rebuild, etc.)
├── roles/                 # Reusable automation roles
└── artifacts/             # Generated kubeconfigs, tokens
```

**Key Playbooks:**
- `re-create-machines.yaml`: Full cluster from scratch
- `bootstrap-cluster.yaml`: Kubernetes bootstrap (Gitea, Flux)
- `rebuild-machines.yaml`: NixOS updates on existing machines
- `add-new-workers.yaml`: Scale cluster

### Bootstrap Flow

```
Bare Metal
    │
    ▼
1. Machine deployment (deploy_nixos role)
    │  - nixos-anywhere with flake: infrastructure/nixos#hostname
    │  - Disko partitions disk
    │  - Installs NixOS + k0s service
    ▼
2. Cluster formation (re-create-machines.yaml)
    │  - create_tokens role (leader generates join tokens)
    │  - join_controllers role
    │  - join_workers role
    │  - fetch_kubeconfig role
    ▼
3. Pre-Flux infrastructure (Ansible/Helm, before Flux)
    │  - deploy_cilium role (CNI networking)
    │  - deploy_openebs role (storage)
    │  - deploy_rook_ceph role (optional, Ceph storage)
    ▼
4. Flux bootstrap (bootstrap-cluster.yaml)
    │  - kubectl apply --kustomize ./bootstrap
    │  - Deploy secrets (age key, Gitea secret)
    │  - Wait for Gitea, push cluster code
    │  - Deploy cluster-settings
    │  - kubectl apply --kustomize ./flux/config
    ▼
Flux takes over
    │  - Watches internal Gitea
    │  - Deploys remaining applications
    ▼
(Optional) Migrate to external Git source
```

## Extraction Analysis

### NixOS Modules

| Module | File | Reusability | Notes |
|--------|------|-------------|-------|
| k0s-node-defaults | `machine-classes/k0s-node-defaults.nix` | 80% | SSH keys need parameterization |
| k0s-node-disks | `machine-classes/k0s-node-disks.nix` | 90% | Disko config, device path overridable |
| k0s-node-vm-disks | `machine-classes/k0s-node-vm-disks.nix` | 90% | VM variant |
| default-values | `modules/default-values.nix` | 70% | Flake registry is site-specific |
| Hardware (um690, um790-pro, ums690s) | `hardware/minisforum/*.nix` | 100% | Pure hardware config |
| Hardware (VMs) | `hardware/amd64-qemu.nix`, etc. | 100% | Pure hardware config |

**What makes modules site-specific:**
- SSH public keys hardcoded in k0s-node-defaults.nix
- Flake registry pointing to internal GitLab
- DNS servers

### Ansible Roles

| Role | Reusability | Notes |
|------|-------------|-------|
| deploy_nixos | 95% | Generic nixos-anywhere wrapper |
| create_tokens | 100% | Standard k0s workflow |
| join_controllers | 100% | Generic token-based join |
| join_workers | 100% | Generic token-based join |
| fetch_kubeconfig | 95% | Generic credential extraction |
| rebuild_nixos | 100% | Generic NixOS rebuild |
| deploy_cilium | 70% | Hardcoded paths to k8s manifests |
| deploy_openebs | 70% | Hardcoded paths to k8s manifests |
| deploy_rook_ceph | 70% | Hardcoded paths to k8s manifests |

### Must Stay Private

- **Host configs** (`hosts/*.nix`): IPs, hostnames, k0s API SANs
- **Inventory files**: MAC addresses, IPs, cluster paths
- **Secrets**: Age keys, Gitea passwords, SSH keys
- **Network topology**: Subnet, gateway, DNS servers

## Extraction Options

### Option 1: Separate NixOS Flake

Create `business-operations-nixos` or similar:

```
business-operations-nixos/
├── flake.nix
├── modules/
│   ├── k0s-node-defaults.nix   # Parameterized (SSH keys as input)
│   ├── k0s-node-disks.nix
│   └── base-system.nix         # Timezone, locale, nix settings
└── hardware/
    ├── minisforum/
    └── vm/
```

Usage in b-ops:
```nix
inputs.bo-nixos.url = "github:yourorg/business-operations-nixos";

nixosConfigurations.exp-k0s-01 = nixpkgs.lib.nixosSystem {
  modules = [
    bo-nixos.nixosModules.k0s-node
    bo-nixos.nixosModules.hardware.minisforum-um790-pro
    ./hosts/exp-k0s-01.nix  # Site-specific: IP, hostname, k0s role
  ];
};
```

**Pros:**
- Clean separation of generic vs site-specific
- Versioned via flake inputs
- Can be used by demo-ops and others

**Cons:**
- Two repos to maintain
- Need to parameterize SSH keys, DNS

### Option 2: Ansible Collection

Extract roles to Ansible Galaxy collection:

```
ansible-galaxy collection install yourorg.k0s_cluster

# In playbook:
roles:
  - yourorg.k0s_cluster.deploy_nixos
  - yourorg.k0s_cluster.create_tokens
  - yourorg.k0s_cluster.join_workers
```

**Pros:**
- Standard Ansible distribution
- Versioned via Galaxy

**Cons:**
- Galaxy publishing overhead
- deploy_cilium/openebs need path abstraction

### Option 3: Keep in business-operations Monorepo

```
business-operations/
├── kubernetes/           # Flux layer (from split-flux.md)
└── infrastructure/
    ├── nixos/
    │   ├── modules/      # Reusable modules
    │   └── hardware/     # Hardware configs
    └── ansible/
        └── roles/        # Reusable roles
```

b-ops and demo-ops reference via:
- Nix flake input pointing to business-operations
- Ansible roles_path or git submodule

**Pros:**
- Single source of truth
- Simpler than multiple repos

**Cons:**
- Larger repo
- Mixed concerns (k8s + infra)

## Outcome

Approach: monorepo (`business-operations`). All three phases completed.

**Phase 1: Extract hardware modules** (done)

Hardware modules submitted upstream to nixos-hardware instead of keeping
them in business-operations. b-ops imports from nixos-hardware directly.

**Phase 2: Extract k0s modules** (done)

Evolved differently than planned. Instead of parameterizing the original
b-ops machine-classes, business-operations got a fresh module structure:

- `nixos/profiles/k0s-node.nix` — composable profile
- `nixos/modules/kubernetes/k0s.nix` — k0s configuration module

SSH keys were never hardcoded in business-operations — user management
stays external via `base-users` flake input. Site-specific disk layouts
(`k0s-node-disks.nix`, `k0s-node-vm-disks.nix`) remain in b-ops where
they belong. Both b-ops and demo-ops consume the business-operations
modules via flake input.

**Phase 3: Extract Ansible roles** (done, ADR-0018)

Roles and playbooks extracted to `business-operations/ansible/`. Helm roles
parameterized with configurable paths (defaulting to cluster-0 as reference).
b-ops ansible directory reduced to inventories and artifacts only.

Consuming repositories use `nix develop github:bo-tech/business-operations#ansible`
which provides tools and sets `ANSIBLE_ROLES_PATH` and `BO_PLAYBOOKS`.
