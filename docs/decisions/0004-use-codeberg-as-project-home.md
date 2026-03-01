---
date: 2026-03-01
---

(adr-0004)=
# 0004 Use Codeberg as Project Home

## Context and Problem Statement

The public repositories (`business-operations`, `planning`) need a hosting
platform. Initially GitHub was used under the `bo-tech` organization, but
the project's values align more closely with a community-driven,
non-commercial platform.

## Considered Options

* GitHub under the `bo-tech` organization
* Codeberg under a dedicated organization

## Decision Outcome

Chosen option: "Codeberg under a dedicated organization" with the
organization name `business-operations`.

Repository mapping:

| Repository           | URL                                                        |
|----------------------|------------------------------------------------------------|
| business-operations  | https://codeberg.org/business-operations/business-operations |
| planning             | https://codeberg.org/business-operations/planning          |

### Rationale

- Codeberg is a non-profit, community-driven platform that aligns with the
  project's goal of empowering small organizations
- The organization name `business-operations` matches the project name
  directly, making it easy to discover
- GitHub mirrors can be maintained if needed for visibility

### Consequences

- Nix flake references change from `github:bo-tech/...` to
  `git+https://codeberg.org/business-operations/...` since Nix does not
  yet have a native Codeberg fetcher (NixOS/nix#14064)
- The git submodule URL in `b-ops` points to Codeberg
- Documentation and ADRs across repositories updated to reflect the new URLs
- Mirroring into other platforms (GitHub, GitLab) can be set up if needed
  for visibility or contributor convenience
