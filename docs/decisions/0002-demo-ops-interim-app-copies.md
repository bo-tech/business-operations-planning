---
date: 2026-02-08
---

(adr-0002)=
# 0002 Copy Cilium and OpenEBS Into demo-ops as Interim Step

## Context and Problem Statement

demo-ops needs Cilium and OpenEBS manifests to deploy a fully functional
single-node k0s cluster. These files currently live in b-ops and will
eventually be extracted into business-operations as reusable components,
but that extraction work is not yet complete.

Waiting for the full app extraction blocks demo-ops from being usable.

## Considered Options

* Wait for business-operations to expose Cilium and OpenEBS — blocks
  demo-ops progress until the extraction is fully worked out
* Copy the needed files from b-ops into demo-ops — demo-ops works now,
  files are replaced once business-operations provides them
* Reference b-ops directly — defeats the purpose of demo-ops being a
  standalone public example

## Decision Outcome

Chosen option: "Copy the needed files into demo-ops" because:

- Unblocks demo-ops immediately — the local VM deployment can be
  validated end-to-end without waiting for the extraction work
- Low risk — the files are well-understood infrastructure manifests
- Temporary by design — once business-operations exposes these
  components, demo-ops switches to consuming them from upstream
- Keeps demo-ops self-contained and publicly usable as an example
