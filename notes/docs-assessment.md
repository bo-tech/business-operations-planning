# B-Ops Documentation Assessment

## Approach: Docs Move With Features

Documentation extracts together with its corresponding code/feature. This ensures:
- Docs and code stay in sync
- Learning from code extraction feeds directly into doc updates
- Natural review checkpoint when moving features
- No rework if code structure changes during extraction

**Workflow:** When extracting a feature, use the mapping below to identify which docs
belong to it. Sanitize and move docs as part of the same commit/PR as the code.

---

## Feature-to-Docs Mapping

Primary lookup table. When moving a feature, these docs go with it.

### Infrastructure Layer

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **NixOS modules** | `infrastructure/nixos/` | `docs/kubernetes/nixos.md`, `infrastructure/nixos/README.md` | [sensitive] IPs, internal GitLab URLs |
| **k0s installation** | `infrastructure/k0s/` | `docs/kubernetes/installation.md`, `infrastructure/k0s/README.md` | [sensitive] internal paths |
| **Ansible automation** | `infrastructure/ansible/` | `docs/kubernetes/ansible.md`, `infrastructure/ansible/README.md` | Mostly generic |
| **Hardware configs** | `infrastructure/nixos/hardware/` | (none yet) | Targeting nixos-hardware upstream |

### Kubernetes Bootstrap

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **Bootstrap process** | `kubernetes/cluster-0/bootstrap/` | `docs/kubernetes/bootstrap.md`, `docs/kubernetes/bootstrap-overview.rst`, `kubernetes/cluster-0/README.md` | [sensitive] SOPS commands |
| **Flux configuration** | `kubernetes/cluster-0/flux/` | `docs/dev/flux.md`, `docs/kubernetes/gitops.md` | gitops.md is stub |
| **Cluster settings** | `kubernetes/cluster-0/flux/vars/` | (covered by bootstrap docs) | |

### Core Components

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **Authelia (SSO)** | `kubernetes/*/apps/security/authelia/` | `docs/core-components/authelia.md`, `docs/core-components/lldap.md` | LLDAP is Authelia backend |
| **Cert-manager** | `kubernetes/*/apps/cert-manager/` | `docs/core-components/cert-manager.md` | [sensitive] domain, AWS policy |
| **Cilium (CNI)** | `kubernetes/*/base-apps/kube-system/cilium/` | `docs/core-components/cilium.md` | |
| **Gitea (bootstrap)** | `kubernetes/*/bootstrap/gitea/` | `docs/core-components/gitea.md` | |
| **Rook-Ceph** | `kubernetes/*/apps/rook-ceph/` | `docs/core-components/rook.md` | |
| **OpenEBS** | `kubernetes/*/apps/openebs/` | `docs/core-components/openebs.md` | [stub] |
| **Vault** | `kubernetes/*/base-apps/secrets/vault/` | `docs/core-components/vault.md` | |
| **CoreDNS** | (internal config) | `docs/core-components/coredns.rst` | b-ops-only |

### Services

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **CloudNativePG** | `kubernetes/*/apps/*/db/` | `docs/services/postgresql.md` | [literalinclude] [sensitive] - high value |

### Backup/Restore

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **VolSync templates** | `kubernetes/*/apps/*/volumes/` | `docs/backup-restore/approach.rst`, `docs/backup-restore/volumes.rst`, `docs/backup-restore/s3.md` | [literalinclude] - high value |
| **CNPG backup** | (in db clusters) | (covered by postgresql.md) | |

### Developer Tooling

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **Dev environment** | `kubernetes/dev-env/` | `docs/dev/dev-env.md` | |
| **SOPS/Age secrets** | `.sops.yaml`, age keys | `docs/dev/sops.md` | |
| **Kustomize patterns** | (used throughout) | `docs/dev/kustomize.md` | |
| **Kubectl tips** | (general) | `docs/dev/kubernetes.md` | |
| **Tooling (ORAS, docs)** | `docs/flake.nix` | `docs/dev/tooling.md` | |

### Applications (demo-ops candidates)

| Feature | Code Location | Docs to Move | Notes |
|---------|---------------|--------------|-------|
| **Renovate Bot** | `kubernetes/*/apps/renovate/` | `docs/apps/renovate-bot.md` | Generic, good for business-operations |
| **GitLab** | `kubernetes/*/base-apps/gitlab/` | `docs/apps/gitlab.md` | [sensitive] demo-ops |
| **Mayan EDMS** | `kubernetes/*/apps/default/mayan-edms/` | `docs/apps/mayan-edms.rst` | [sensitive] demo-ops |
| **Tryton** | `kubernetes/*/apps/default/tryton/` | `docs/apps/tryton.rst` | b-ops-only, very site-specific |

### Cross-cutting Docs (move with first relevant feature)

| Doc | Move With | Notes |
|-----|-----------|-------|
| `docs/index.rst` | First extraction | Update toctree as sections move |
| `docs/known-issues.md` | Cilium or ingress-nginx | Generic issues |
| `docs/glossary.rst` | First extraction | Kubernetes terms |
| `docs/credits.rst` | First extraction | Attribution |
| `docs/decision-log.rst` | Already done (ADRs) | |

### Stays in b-ops

| Doc | Reason |
|-----|--------|
| `docs/exp/*` | Experimental cluster internal docs |
| `docs/apps/tryton.rst` | Very site-specific |
| `docs/core-components/coredns.rst` | Internal DNS config |
| `docs/wip-decisions.md` | Internal planning |

### Delete from b-ops

| Doc | Reason |
|-----|--------|
| `docs/backup-restore/utils.md` | Empty since VolSync switch |
| `docs/todo.md` | Outdated stub |
| `docs/wip-hardware.md` | Empty |
| `docs/about.md` | Empty stub |
| `docs/kubernetes/gitops.md` | Empty stub |

---

## Sanitization Patterns

Apply when moving docs:

| Pattern | Replace With |
|---------|--------------|
| Internal domain `*.<domain>` | `*.example.com` |
| Private IPs `192.168.x.x` | `192.0.2.x` (TEST-NET-1) |
| Internal GitLab URLs | `https://gitlab.example.com/...` |
| LDAP DN | `dc=example,dc=com` |
| Site-specific namespace refs | Generic names (e.g., `mayan-prod`) |

---

## Reference: Detailed Section Analysis

Categories used below:
- `business-operations` - Generic patterns, reusable across deployments
- `demo-ops` - Example configurations, tutorials
- `b-ops-only` - Site-specific, internal reference only

Flags:
- `[sensitive]` - Contains domains, IPs, paths needing sanitization
- `[include]` - File uses `{include}` directive, content lives elsewhere
- `[literalinclude]` - References actual cluster config files
- `[stub]` - Minimal/empty content
- `[wip]` - Work in progress

---

## Section 1: kubernetes/

Infrastructure setup documentation for NixOS, k0s, Ansible, and cluster bootstrap.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 27 lines | business-operations | | Good overview, links to NixOS/Ansible |
| `nixos.md` | 3 lines | business-operations | [include] | Pulls from `infrastructure/nixos/README.md` |
| `installation.md` | 4 lines | business-operations | [include] | Pulls from `infrastructure/k0s/README.md` |
| `ansible.md` | 3 lines | business-operations | [include] | Pulls from `infrastructure/ansible/README.md` |
| `bootstrap.md` | 4 lines | demo-ops | [include] | Pulls from `kubernetes/cluster-0/README.md` |
| `bootstrap-overview.rst` | 128 lines | business-operations | | Excellent stage-by-stage bootstrap explanation |
| `crds.rst` | ? | business-operations | | CRD documentation |
| `core-components.md` | 25 lines | business-operations | | Component overview with links |
| `gitops.md` | 2 lines | business-operations | [stub] | Empty, just heading |

**Included source files (need separate review):**

| Source File | Category | Flags | Notes |
|-------------|----------|-------|-------|
| `infrastructure/nixos/README.md` | business-operations | [sensitive] | Contains internal GitLab URL, private IPs |
| `infrastructure/k0s/README.md` | business-operations | [sensitive] | Contains internal paths |
| `infrastructure/ansible/README.md` | business-operations | | Vision, usage, limitations - mostly generic |
| `kubernetes/cluster-0/README.md` | demo-ops | [sensitive] | Bootstrap steps with SOPS commands |

**Section summary:**
- Core content is valuable for `business-operations`
- Need to resolve `{include}` files and sanitize
- `gitops.md` needs content or removal

---

## Section 2: core-components/

Documentation for infrastructure components.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 6 lines | business-operations | | Just toctree |
| `authelia.md` | 317 bytes | business-operations | | Brief, links to ADR |
| `cert-manager.md` | 1478 bytes | business-operations | [sensitive] | Contains internal domain, AWS policy example |
| `cilium.md` | 679 bytes | business-operations | | L2 announcements, troubleshooting |
| `coredns.rst` | 351 bytes | b-ops-only | [sensitive] | Internal DNS, likely site-specific |
| `gitea.md` | 757 bytes | business-operations | | Bootstrap Git server, known issues |
| `lldap.md` | 548 bytes | business-operations | | LDAP backend for Authelia |
| `openebs.md` | 144 bytes | business-operations | [stub] | Very brief |
| `rook.md` | 665 bytes | business-operations | | Ceph setup, HA considerations |
| `vault.md` | 1695 bytes | business-operations | | Bootstrap, config, unsealing |

**Section summary:**
- Most content is generic and valuable
- Several files are brief stubs - decide if expansion needed
- `cert-manager.md` and `coredns.rst` need sanitization

---

## Section 3: services/

Shared services documentation.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 253 bytes | business-operations | | Just toctree |
| `postgresql.md` | 4708 bytes | business-operations | [literalinclude] [sensitive] | Excellent CloudNativePG docs, PITR, backup/restore |

**Section summary:**
- `postgresql.md` is high-value, well-documented
- Contains `literalinclude` refs to actual cluster configs
- Site-specific namespace refs need sanitization

---

## Section 4: apps/

Application-specific documentation.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 14 lines | business-operations | | Just toctree |
| `gitlab.md` | 17 lines | demo-ops | [sensitive] | Brief backup/restore command |
| `mayan-edms.rst` | 57 lines | demo-ops | [sensitive] | Config, deployment, internal GitLab refs |
| `renovate-bot.md` | 51 lines | business-operations | | Generic setup, resource sizing, token config |
| `taiga.rst` | ? | demo-ops | | Project management app |
| `tryton.rst` | 166 lines | b-ops-only | [sensitive] | Very site-specific, internal repos |
| `wekan.md` | ? | demo-ops | | Kanban app |

**Section summary:**
- `renovate-bot.md` is generic and valuable
- Most app docs are site-specific examples
- `tryton.rst` is heavily tied to internal setup - likely stays in b-ops

---

## Section 5: backup-restore/

Backup and restore documentation.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 18 lines | business-operations | | Just toctree |
| `approach.rst` | 62 lines | business-operations | | Excellent: credentials, tool selection, VolSync rationale |
| `volumes.rst` | 126 lines | business-operations | [literalinclude] | VolSync config, restore process, Restic CLI |
| `s3.md` | 69 lines | business-operations | | S3 policies for Restic and CNPG |
| `utils.md` | 4 lines | - | [stub] | Empty since VolSync switch - DELETE |

**Section summary:**
- High-value section, mostly generic
- `literalinclude` refs need resolution
- `utils.md` should be removed

---

## Section 6: dev/

Developer workflow documentation.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.md` | 14 lines | business-operations | | Just toctree |
| `dev-env.md` | 85 lines | business-operations | | Local dev, minikube, Flux deployment |
| `add-application.md` | 24 lines | demo-ops | [literalinclude] [sensitive] | Authelia config path, Hajimari example |
| `flux.md` | 59 lines | business-operations | | Flux commands, dry-run, Helm manual runs |
| `sops.md` | 27 lines | business-operations | | Age key generation, pointers |
| `kubernetes.md` | 21 lines | business-operations | | Finalizer patching, kubectl debug |
| `kustomize.md` | 54 lines | business-operations | | Resource omission patterns |
| `tooling.md` | 27 lines | business-operations | | ORAS, docs build with Nix flake |

**Section summary:**
- Mostly generic developer workflows
- Good candidates for `business-operations`
- `add-application.md` needs path sanitization

---

## Section 7: exp/ (Experimental Cluster)

Documentation for applications deployed to the experimental cluster (cluster-exp).
These are internal deployment plans and configurations, not generic examples.

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 11 lines | b-ops-only | | Toctree for cluster-exp apps |
| `synapse.md` | 460 lines | b-ops-only | [sensitive] | Full deployment plan: Matrix/Element for AI agent infra |
| `ollama.md` | 136 lines | b-ops-only | [sensitive] | LLM runtime for assist project, internal URLs |

**Section summary:**
- Internal infrastructure docs for `cluster-exp`
- Detailed deployment plans with architecture diagrams, step-by-step manifests
- Tied to specific projects (assist AI agent infrastructure)
- Stay in `b-ops` as internal reference
- Could inspire sanitized examples for `demo-ops` later, but not direct extraction

---

## Section 8: decisions/

Architecture Decision Records (already extracted to business-operations).

| File | Category | Notes |
|------|----------|-------|
| `README.md` | business-operations | Already extracted |
| `adr-template.md` | business-operations | Already extracted |
| `0000-0016` (17 ADRs) | business-operations | Already extracted |

**Section summary:**
- Complete, already in `business-operations`

---

## Section 9: Root-level files

| File | Size | Category | Flags | Notes |
|------|------|----------|-------|-------|
| `index.rst` | 32 lines | business-operations | | Main toctree |
| `about.md` | 31 bytes | business-operations | [stub] | Empty - needs content or removal |
| `todo.md` | 29 bytes | - | [stub] | Likely outdated - DELETE |
| `known-issues.md` | 38 lines | business-operations | | Ingress-nginx, Cilium, snapshot issues |
| `wip-decisions.md` | 126 lines | b-ops-only | [wip] | Future architecture (ChatGPT output, emojis) |
| `wip-hardware.md` | 0 bytes | - | [stub] | Empty - DELETE |
| `decision-log.rst` | 114 bytes | business-operations | | Links to decisions/ |
| `credits.rst` | 594 bytes | business-operations | | Attribution |
| `glossary.rst` | 893 bytes | business-operations | | Kubernetes terms |

**Section summary:**
- `known-issues.md` is valuable, generic
- Delete empty stubs: `todo.md`, `wip-hardware.md`
- `wip-decisions.md` is internal planning, stays in b-ops
- `about.md` needs content or removal

---

## Summary by Category

### business-operations (public template)

High-priority extractions:
1. `kubernetes/bootstrap-overview.rst` - Stage-by-stage bootstrap
2. `backup-restore/approach.rst` - Tool selection, credentials strategy
3. `backup-restore/volumes.rst` - VolSync patterns (sanitize literalinclude)
4. `services/postgresql.md` - CloudNativePG patterns (sanitize)
5. `dev/flux.md` - Flux workflows
6. `dev/kustomize.md` - Kustomize patterns
7. `known-issues.md` - Common problems

Lower-priority but valuable:
- `dev/dev-env.md`, `dev/sops.md`, `dev/kubernetes.md`, `dev/tooling.md`
- `core-components/*.md` (most files)
- `glossary.rst`, `credits.rst`

### demo-ops (example configurations)

- `kubernetes/bootstrap.md` (via include from cluster-0)
- `apps/gitlab.md`, `apps/mayan-edms.rst`, `apps/wekan.md`
- `dev/add-application.md`

### b-ops-only (internal reference)

- `apps/tryton.rst` - Very site-specific
- `wip-decisions.md` - Internal planning
- `core-components/coredns.rst` - Internal DNS config
- `exp/*` - Experimental cluster deployment docs (synapse, ollama for AI agent infra)

### Delete

- `backup-restore/utils.md` - Empty
- `todo.md` - Outdated stub
- `wip-hardware.md` - Empty
- `about.md` - Stub (or add content)
- `kubernetes/gitops.md` - Empty stub

---

## Workflow Checklist

When extracting a feature:

1. [ ] Look up feature in mapping table above
2. [ ] Identify all docs that move with this feature
3. [ ] Check for `[sensitive]` flag - apply sanitization patterns
4. [ ] Check for `[include]`/`[literalinclude]` - resolve or convert to inline examples
5. [ ] Move docs in same commit/PR as code
6. [ ] Update `docs/index.rst` toctree if needed
7. [ ] Delete empty stubs if encountered

## Pending Cleanup (do when convenient)

- [ ] Delete `docs/backup-restore/utils.md`
- [ ] Delete `docs/todo.md`
- [ ] Delete `docs/wip-hardware.md`
- [ ] Delete `docs/about.md` (or add content)
- [ ] Delete `docs/kubernetes/gitops.md` (or add content)
