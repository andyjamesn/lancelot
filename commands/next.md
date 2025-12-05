# Show Next Subtask

Display the next subtask to implement.

## Instructions

1. Read the active plan ID from `.lancelot/active-plan`
2. Load the plan from `.lancelot/plans/{id}.json`
3. Find the next pending subtask (in order)
4. Display its details

## Finding Next Subtask

Traverse in order:
```
milestones (by order) →
  tasks (by order) →
    subtasks (by order) →
      find first with status === "pending"
```

## Output Format

```
────────────────────────────────────────────
Next Subtask

ID: {first 8 chars}
Title: {title}
File: {targetFile} ({action})

Milestone: {milestone.title}
Task: {task.title}

Description:
{description}

Steps ({count}):
1. {step 1 instruction}
2. {step 2 instruction}
...

Complexity: {complexity}
────────────────────────────────────────────

⏭ Run: /lancelot/prompt {id}
```

## If All Complete

```
────────────────────────────────────────────
🎉 All subtasks complete!

Plan: {title}
Total subtasks: {count}

The implementation plan is finished.
────────────────────────────────────────────
```

## If No Active Plan

```
────────────────────────────────────────────
No active plan found.

Run /lancelot/plan to create one.
────────────────────────────────────────────
```

## After Display

STOP and wait for user to run `/lancelot/prompt`.
