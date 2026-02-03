---
date: 2026-01-06
---

(adr-0000)=
# 0000 Use Any Decision Records

## Context and Problem Statement

How do we document and track decisions made during the project lifecycle? We
need a lightweight way to capture the context, options considered, and rationale
behind significant choices — not just architectural ones, but process, tooling,
and scope decisions too.

## Considered Options

* No formal documentation — rely on commit messages and code comments
* Wiki pages — centralized documentation but disconnected from code
* Any Decision Records (ADRs) — lightweight markdown files in the repository

## Decision Outcome

Chosen option: "Any Decision Records" because:

- ADRs live alongside the planning docs in version control
- They provide a clear template for capturing context and rationale
- Historical decisions remain accessible even as team members change
- The numbered format creates a natural chronological log
- Markdown format is easy to write and renders well in GitLab/GitHub
- "Any" rather than "Architecture" reflects the broader scope: we record
  decisions about process, tooling, and scope — not just technical architecture
