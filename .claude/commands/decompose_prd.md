---
description: Break down a PRD into small independent tasks for create_plan_generic
model: opus
---

# Decompose PRD into Implementation Tasks

You are tasked with breaking down a Product Requirements Document into small, independent implementation tasks. Each task should be scoped so that `create_plan_generic` can implement it without exceeding ~500 lines of changed code across files.

## Initial Response

When this command is invoked:

1. **If a file path was provided as a parameter**:
   - Read the PRD file FULLY
   - Begin the decomposition process immediately

2. **If no parameter provided**, respond with:
```
I'll help break down a PRD into implementable tasks.

Please provide the path to the PRD file (e.g., `thoughts/shared/research/2026-03-02-product-requirements-document.md`)
```

Then wait for input.

## Process

### Step 1: Parallel Codebase Analysis

After reading the PRD, spawn **all of the following sub-agents concurrently** in a single message:

**Before you start**: These are the same sub-agent types used by `/research_codebase_generic`. If the user has already run `/research_codebase_generic` and a recent research document exists in `thoughts/shared/research/`, read that document instead of re-running the analysis. Only spawn fresh sub-agents if no recent research exists.

1. **codebase-analyzer** — "Analyze the current codebase to determine what already exists. For each feature/capability described below, report whether it's fully implemented, partially implemented, or missing. Include file:line references for existing code. Features to check: [paste condensed feature list from PRD]"

2. **codebase-pattern-finder** — "Find the established patterns in this codebase for: adding new routes/endpoints, adding new engine modules, modifying templates, adding new database tables/columns, adding tests. Return concrete code examples of each pattern with file:line references."

3. **codebase-locator** — "Find all files that would need modification or serve as integration points for new features. Map out: route definitions, template files, engine modules, database schema, static assets, configuration files, and test files. Return a complete file inventory with descriptions."

**CRITICAL**: Do NOT duplicate this research yourself. Wait for all agents to return (or finish reading existing research) before proceeding.

### Step 2: Gap Analysis

Using the sub-agent results, build a gap matrix:

```
| PRD Feature | Status | What Exists | What's Missing |
|-------------|--------|-------------|----------------|
| F1: ...     | Full/Partial/Missing | file:line refs | specific gaps |
```

- **Full**: Skip entirely — no task needed
- **Partial**: Task covers only the missing parts
- **Missing**: Task covers full implementation

Present this matrix to the user and ask:
```
Here's what I found after analyzing the codebase against the PRD:

[gap matrix]

Before I decompose this into tasks:
1. Are there features you want to skip or deprioritize?
2. Are there dependencies or ordering constraints I should know about?
3. Any features that should be grouped together?
```

### Step 3: Task Decomposition

For each gap, decompose into tasks following these rules:

**Sizing rules:**
- Target: ~200-400 lines of changed code per task (hard cap: 500)
- A single new route + template + engine function ≈ one task
- Database schema changes should be in their own task when possible
- Frontend-only changes (CSS/JS/templates) can be their own task
- If a feature needs both backend + frontend and total exceeds 300 lines, split into two tasks

**Independence rules:**
- Each task must be implementable and testable on its own
- Tasks should not break existing functionality when implemented alone
- If task B depends on task A, mark the dependency explicitly
- Prefer tasks that add new code over tasks that modify existing code heavily

**Decomposition heuristics:**
- **New endpoint**: route handler + template + engine logic = 1 task (if ≤500 LOC)
- **New engine module**: module + integration into existing routes = 1 task
- **Schema change**: migration + model updates + backfill script = 1 task
- **UI overhaul**: split by page/component, not by layer
- **Refactoring**: one cohesive refactor per task
- **Config/infra**: group related config changes together

### Step 4: Write Task Manifest

Write the output to `thoughts/shared/plans/YYYY-MM-DD-prd-tasks-<description>.md` using this format:

````markdown
# PRD Task Decomposition: [PRD Title]

**Source PRD**: [path to PRD file]
**Date**: YYYY-MM-DD
**Total tasks**: N
**Estimated total LOC**: ~X

## Task Overview

| # | Task | Est. LOC | Dependencies | Priority |
|---|------|----------|--------------|----------|
| 1 | Short name | ~250 | None | P0 |
| 2 | Short name | ~300 | Task 1 | P0 |
| 3 | Short name | ~200 | None | P1 |

## Dependency Graph

```
Task 1 (foundation)
├── Task 2 (depends on 1)
└── Task 4 (depends on 1)
Task 3 (independent)
Task 5 (independent)
```

## Implementation Order

Suggested execution sequence respecting dependencies:
1. **Batch 1** (parallel): Task 1, Task 3, Task 5
2. **Batch 2** (after batch 1): Task 2, Task 4
3. **Batch 3** (after batch 2): Task 6

---

## Task 1: [Descriptive Name]

**Estimated LOC**: ~250
**Dependencies**: None
**Priority**: P0
**PRD Features**: F1, F3 (partial)

### Objective
[1-2 sentences: what this task accomplishes in isolation]

### What exists
- [file:line — what's already there]

### Changes required
1. **[file path]** — [what to add/modify, ~N lines]
2. **[file path]** — [what to add/modify, ~N lines]

### Acceptance criteria
- [ ] [Specific, testable criterion]
- [ ] [Another criterion]

### Notes for planner
[Context that `create_plan_generic` needs — patterns to follow, edge cases, constraints]

---

## Task 2: [Descriptive Name]
[Same structure...]

````

### Step 5: Review with User

Present the decomposition:
```
I've written N tasks to [path].

Summary:
- X tasks with no dependencies (can start immediately)
- Y tasks in the critical path
- Estimated ~Z total LOC across all tasks

The suggested execution order is:
[batch summary]

To implement any task, run:
`/create_plan_generic thoughts/shared/plans/YYYY-MM-DD-prd-tasks-description.md — Task N`

Would you like to adjust any task scope, reorder priorities, or merge/split any tasks?
```

## Important Guidelines

1. **Context window efficiency**: All heavy codebase research happens in sub-agents. The main context only holds the PRD, gap matrix, and task definitions — never raw codebase exploration.

2. **Sub-agent parallelism**: Always spawn the three initial research agents in a single message. Never serialize what can be parallelized.

3. **500 LOC is a hard cap**: If you can't get a task under 500 LOC, split it further. Smaller is better — a 200 LOC task is preferable to a 450 LOC task.

4. **Independence over perfection**: A slightly awkward task boundary that preserves independence is better than a clean boundary that creates tight coupling.

5. **Existing code is sacred**: Tasks should add new capabilities, not rewrite existing working code. If refactoring is needed, make it a separate task.

6. **Each task must be self-verifiable**: Every task needs acceptance criteria that can be checked without implementing other tasks first.

7. **Don't over-decompose**: If a feature naturally fits in 150 lines, don't split it into two 75-line tasks. The overhead of task switching matters.

8. **Provide planner context**: The "Notes for planner" section is critical — it tells `create_plan_generic` about patterns, conventions, and gotchas so it doesn't have to rediscover them.
