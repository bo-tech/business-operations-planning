# Repository Structure

How this planning repository is organized.


## Overview

```
planning/
├── docs/
│   ├── decisions/          # Architecture Decision Records
│   ├── decision-backlog/   # Pending decisions
│   ├── plans/              # Month-level plans
│   ├── process/            # How planning works
│   └── milestone-*/        # Milestone-specific planning
├── sprints/                # Sprint files
└── notes/                  # Working notes
```


## Sections

### `docs/decisions/`

Architecture Decision Records (ADRs). Immutable once accepted — new ADRs
supersede old ones rather than editing.

### `docs/decision-backlog/`

Decisions needed eventually, but not now. Each file captures context, options,
and a trigger for when to decide.

Workflow: capture → forget → revisit when triggered → create ADR.

### `docs/plans/`

Month-level planning documents (`YYYY-MM-plan.md`). Provides direction without
micromanagement, bridges vision and sprint execution.

### `docs/process/`

How planning and execution works. Onboarding material for collaborators.

### `docs/milestone-*/`

Planning documents for specific milestones. Each milestone folder contains
overview, goals, and relevant details.

### `sprints/`

Sprint files for execution. A sprint is a small chunk of work — could be 2
focused days or a few weeks as a side project.

Format documented in `sprints/sprint-file-spec.org`.

### `notes/`

Working notes during active work. Temporary capture, processed into permanent
docs during daily review.


## Added as Needed

These sections are added when the project needs them:

### `docs/idea/`

Vision, values, and positioning documents. Added when project identity
solidifies and needs to be communicated.

### `docs/knowledge/`

Captured knowledge about tools, patterns, and research. Added when accumulated
learnings need a permanent home outside of decision records.
