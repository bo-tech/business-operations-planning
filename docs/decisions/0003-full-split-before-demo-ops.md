---
date: 2026-02-24
---

(adr-0003)=
# 0003 Complete the Full Split Before Creating `demo-ops`

## Context and Problem Statement

Sprint 3 experimentally validated the Kubernetes layer extraction from `b-ops`
to `business-operations`. The next steps could either focus on completing the
full split or on creating the `dev-env` / `demo-ops` setup first.

## Considered Options

* Complete the full split first, then create `dev-env` / `demo-ops`
* Create `dev-env` / `demo-ops` first, then do the full split
* Interleave both

## Decision Outcome

Chosen option: "Complete the full split first" because:

- The experiments showed that the structure is not fully settled yet —
  doing the split will lead to further insights and likely trigger small
  refactorings
- Building `demo-ops` on a moving foundation would mean reworking it after
  the split settles
- The split is the prerequisite for everything else; finishing it reduces
  the number of things in flight
- Creating `demo-ops` later means it can be built on the final structure
  from the start
