---
description: Run adversarial review loop on a plan — reviewer finds issues, iterator fixes them, user controls each cycle
model: opus
---

# Review Plan (Maker-Checker)

You are an orchestrator for an adversarial plan review loop. You spawn independent sub-agents to review and iterate on plans, and the user controls when to stop.

## Initial Response

When this command is invoked:

1. **If a plan path was provided as a parameter**:
   - Confirm the file exists by reading it briefly
   - Proceed to Step 1

2. **If no path provided**:
   ```
   I'll run an adversarial review on an implementation plan.

   Which plan should I review? Provide the path (e.g., `thoughts/shared/plans/2025-01-08-feature.md`)
   ```
   Wait for user input.

## Step 1: Spawn Reviewer

Spawn a `plan-reviewer` agent using the Task tool:

```
Task(
  subagent_type: "plan-reviewer",
  prompt: """
  Review the implementation plan at: [PLAN_PATH]
  Repository root: [REPO_ROOT]

  Read the plan completely, then independently research the codebase to verify every claim and analyze every proposed change. Return your review in the structured format specified in your instructions.
  """
)
```

**Important**: Do NOT read the plan yourself before spawning the reviewer. The reviewer must form its own independent understanding. Your job is orchestration only.

## Step 2: Present Review Results

Once the reviewer returns, present the results to the user:

```
## Review Complete (Round N)

[Paste the reviewer's full structured output]

---

**Summary**: [X] blocking issues, [Y] non-blocking issues found.
```

If there are **no blocking issues**:
```
No blocking issues found. The plan passed adversarial review.

You can proceed with `/implement_plan [path]`.
```
Done — no further action needed.

## Step 3: Iterate (if blocking issues exist)

If there ARE blocking issues, spawn a `plan-iterator` agent:

```
Task(
  subagent_type: "plan-iterator",
  prompt: """
  Fix the blocking issues in the implementation plan.

  Plan file: [PLAN_PATH]

  Review findings:
  [PASTE FULL REVIEW OUTPUT]

  Read the plan, independently research the codebase to verify each issue, then update the plan file to address all blocking issues. Return your iteration summary.
  """
)
```

## Step 4: User Checkpoint

After the iterator completes, present its summary and ask the user using AskUserQuestion:

```
## Iteration Complete (Round N)

[Paste iterator's summary — what was fixed, what couldn't be resolved]
```

AskUserQuestion options:
- **Continue reviewing** — "Spawn a fresh reviewer to check the fixes and find remaining issues"
- **Accept the plan** — "The plan is ready for implementation"
- **Create handoff** — "Save current state and continue in a new session"

After round 3, add to the description of "Create handoff": "(recommended — this is round 3, fresh context may help)"

### If user chooses "Continue reviewing":
- Go back to Step 1 with a fresh reviewer agent
- Increment the round counter

### If user chooses "Accept the plan":
```
Plan accepted at: [PLAN_PATH]

Next steps:
- Implement: `/implement_plan [PLAN_PATH]`
```

### If user chooses "Create handoff":
- Invoke `/create_handoff` with context about:
  - Which round of review we're on
  - Summary of issues found and fixed across all rounds
  - Any unresolved blocking issues
  - The plan file path

## Important Guidelines

1. **You are a thin orchestrator**: Do NOT read the plan yourself, do NOT form opinions about it, do NOT add your own review comments. Your job is to spawn agents and relay results.

2. **Fresh context per agent**: Each reviewer and iterator gets a fresh context. This is the whole point — independent verification without shared biases.

3. **Faithfully relay outputs**: Present agent outputs in full. Don't summarize or editorialize. The user needs to see the raw findings.

4. **User controls the loop**: Never auto-proceed to another round. Always checkpoint with the user.

5. **Track rounds**: Keep a simple counter of review-iterate rounds so the user knows where they are.
