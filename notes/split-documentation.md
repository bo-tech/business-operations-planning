# Splitting b-ops: Documentation Layer Analysis

Analysis date: 2026-02-03

## Overview

This layer covers Sphinx documentation, ADRs, and operational guides.

- **ADRs**: Done - extracted to `business-operations` with cross-reference labels and ID prefixes
- **sphinx-builder**: Done - published to GitHub/bo-tech
- **Other docs**: Assess and categorize (`business-operations` / `demo-ops` / `b-ops`), then extract public docs

## Current Structure

```
docs/
├── flake.nix              # Nix build for HTML/PDF output
├── conf.py                # Sphinx configuration
├── Makefile               # Build commands
├── index.rst              # Main entry point
├── decisions/             # 16 ADRs (Architecture Decision Records)
├── apps/                  # Application-specific documentation
├── backup-restore/        # Backup/restore procedures
├── core-components/       # Infrastructure component docs
├── dev/                   # Development guides
├── exp/                   # Experimental cluster docs
├── kubernetes/            # Kubernetes-specific docs
└── services/              # Service documentation
```

## Documentation Sections

From `index.rst`:
- about, todo
- kubernetes/index
- core-components/index
- services/index
- apps/index
- exp/index
- backup-restore/index
- dev/index
- decision-log
- known-issues
- credits, glossary

## Build System

**sphinx-builder will be published** to GitHub/bo-tech to unblock documentation
extraction. It's a generic Sphinx + LaTeX environment with no secrets.

The docs have their own `flake.nix` that:
- Depends on `sphinx-builder` (currently internal GitLab, will be public)
- Outputs HTML and PDF packages
- Provides dev shell for local editing

```nix
inputs.sphinx-builder.url =
  "git+https://gitlab.<lab-domain>/bv/images/sphinx-builder.git";
```

## ADRs (Architecture Decision Records)

16 decisions documented in `docs/decisions/`:

| ADR | Title | Status |
|-----|-------|--------|
| 0000 | Use Markdown Any Decision Records | Active |
| 0001 | Use Gitea for bootstrap | Active |
| 0002 | Use Authelia as IDP | Active |
| 0003 | Use CloudNative-PG | Active |
| 0004 | Multiple databases in one cluster | Active |
| 0005 | Max body size per ingress | Active |
| 0006 | Add HTTP caching proxy | Active |
| 0007 | Use nixos-anywhere to provision the OS | Active |
| 0008 | Use Ansible for automation | Active |
| 0009 | Use Nix flakes | Active |
| 0010 | Add secret cluster settings | Active |
| 0011 | VolSync for backup | Superseded |
| 0012 | Cluster bootstrap mode | Active |
| 0013 | Restore approach | Active |
| 0014 | Stay with k8up for backup | Superseded |
| 0015 | Backup database volumes | Active |
| 0016 | Backup refactoring VolSync | Active |

## Extraction Analysis

### Fully Reusable (No Secrets)

- **ADRs**: All decision records are public-safe, document architectural choices
- **Template**: `adr-template.md` for new decisions
- **Patterns**: dev guides, backup procedures (sanitized)

### Needs Sanitization

- **App docs**: May reference internal URLs, IPs
- **Known issues**: References specific cluster behavior
- **Kubernetes docs**: May have internal references

### Must Stay Private

- **Nothing** - documentation should be publishable after sanitization

## Extraction Options

### Option 1: Copy ADRs to business-operations

ADRs document decisions applicable to the product. Titles will be modified to
include the ADR ID (e.g., "Use Gitea for bootstrap" → "ADR-0001: Use Gitea for
bootstrap").

```
business-operations/
└── docs/
    └── decisions/
        ├── 0001-use-gitea-for-bootstrap.md
        ├── 0002-use-authelia-as-idp.md
        └── ...
```

### Option 2: Extract Documentation Framework

Create reusable Sphinx setup:
```
business-operations/
└── docs/
    ├── flake.nix          # Generic sphinx-builder (public)
    ├── conf.py            # Base Sphinx config
    └── templates/         # ADR template, doc structure
```

b-ops extends with site-specific content.

### Option 3: Publish sphinx-builder

**Decision: Will publish to GitHub/bo-tech.**
- Generic Sphinx + LaTeX environment, no secrets
- Used in default repository template for personal and business repos
- Unblocks documentation extraction

## Dependencies

| Dependency | Location | Blocker? |
|------------|----------|----------|
| sphinx-builder | GitHub/bo-tech (to be published) | No |
| Sphinx/RST tooling | Nix packages | No |

## Recommended First Steps

1. Done - Publish sphinx-builder to GitHub/bo-tech
2. Done - Copy all ADRs to `business-operations` (modify titles to include ADR ID)
3. Assess remaining docs: categorize as `business-operations` / `demo-ops` / `b-ops`
4. Extract public documentation to `business-operations`
