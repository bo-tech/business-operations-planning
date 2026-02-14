# Splitting b-ops: Overview

Analysis date: 2026-02-03

## Goal

Split b-ops into three repositories:

- **business-operations**: Public product with reusable infrastructure patterns
- **demo-ops**: Public example deployment showing how to use business-operations
- **b-ops**: Private production deployment

## Layers

| Layer | Document | Action Needed |
|-------|----------|---------------|
| Flux/GitOps | [split-flux.md](split-flux.md) | Yes |
| Documentation | [split-documentation.md](split-documentation.md) | Yes |
| Tooling | [split-tooling.md](split-tooling.md) | Yes |
| Infrastructure | [split-infrastructure.md](split-infrastructure.md) | Done |
| Configuration | [split-configuration.md](split-configuration.md) | Done |

## Layer Summary

### Flux/GitOps

Kubernetes manifests, Kustomize templates, HelmReleases.

- **Extract**: VolSync templates, base infrastructure patterns
- **Approach**: Experiment first - explore options, try most promising (Flux multi-source, git submodule)
- **Deferred**: Kluctl migration (long-term preference, but need something working first)
- **First step**: Explore and experiment with multi-repo approaches

### Infrastructure

NixOS modules, Ansible roles, k0s bootstrap.

- **Approach**: Keep as monorepo (`business-operations`). NixOS is most fundamental, start there.
- **Sequence**: Hardware modules first → parameterize k0s modules → extract → validate with `demo-ops`
- **Ansible**: Deferred to later phase
- **First step**: Extract hardware configs, create flake.nix

### Documentation

Sphinx docs, ADRs, operational guides.

- **Extract**: ADRs (modify titles to include ID), public docs after assessment
- **Prerequisite**: Publish `sphinx-builder` to GitHub/bo-tech
- **First step**: Assess docs and categorize (`business-operations` / `demo-ops` / `b-ops`)

### Tooling

Taskfile, utilities, Renovate config.

- **Root tasks**: Stay in `b-ops` (site-specific WOL/shutdown; document pattern later)
- **Cluster tasks**: Experiment needed for multi-repo pattern (`push`, `info`, `forward-gitea`)
- **Utilities**: Stay in `b-ops` for now
- **Renovate**: Stays in `b-ops`; `business-operations` needs new config for GitHub (backlog)
- **Cleanup**: Remove unused tasks from `b-ops` (`bootstrap`, `trigger-backup`, `k3s-*`)

### Configuration

SOPS rules, AGENTS.md, project metadata.

- **Extract**: Nothing
- **Reason**: business-operations won't contain secrets; contributor docs added after split
