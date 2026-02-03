---
date: 2026-02-03
---

(adr-0001)=
# 0001 Use Text-Based Repository for Planning

## Context and Problem Statement

Where and how do we manage planning artifacts — results, plans, decisions, and
task tracking? We need something that works now, without committing to a
specific toolchain or hosting platform.

## Considered Options

* GitHub/GitLab issues — integrated with code hosting but ties planning to a
  specific platform
* Taiga — open-source project management with good agile support
* Text-based repository — markdown files in version control

## Decision Outcome

Chosen option: "Text-based repository" because:

- Low friction start: no tool setup, no accounts, no learning curve
- Full transparency: everything is visible in plain text, easy to review
- Portable: no dependency on a specific platform or vendor
- Tools can be added later when a concrete need arises
- The project's permanent home is not yet determined — a git repository can
  move anywhere

## Consequences

Decisions about project home and tooling can be delayed — which is exactly what
we want at this stage.
