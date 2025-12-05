# Implement Subtask

Implement a specific subtask from the active plan.

## ⛔ CRITICAL: WORKFLOW RULES

```
┌────────────────────────────────────────────┐
│  YOU ARE HERE: /lancelot/prompt            │
│                                            │
│  implement → STOP → wait for /review       │
│                                            │
│  ❌ DO NOT mark anything complete          │
│  ❌ DO NOT modify plan JSON                │
│  ❌ DO NOT proceed to next subtask         │
│  ❌ DO NOT run /lancelot/review yourself   │
└────────────────────────────────────────────┘
```

## Subtask ID

$ARGUMENTS

## Workflow

### Step 1: Load the Plan

1. Read UUID from `.lancelot/active-plan`
2. Load plan from `.lancelot/plans/{uuid}.json`
3. Find subtask by ID (partial match OK - first 8 chars)

### Step 2: Gather Context

- If **modify**: Read the current file
- If **create**: Find similar files as reference
- Look up any `referenceSymbols` in the steps
- Find patterns to follow

### Step 3: Implement

Execute each step in the subtask:
- Follow the `instruction` exactly
- Apply any `codeHint` patterns
- Write clean, consistent code
- Match existing conventions

### Step 4: STOP

After implementation is complete, say EXACTLY:

```
────────────────────────────────────────────
✓ Implementation complete: {subtask.title}

File: {targetFile}
Steps completed: {count}

⏭ Next: Run /lancelot/review {subtask-id}
────────────────────────────────────────────
```

Then **STOP**. Do not continue. Wait for the user.

## What You Must NOT Do

1. **Never mark subtasks complete** - Only `/lancelot/review` can do this
2. **Never modify the plan JSON** - That's review's job
3. **Never auto-continue** - User decides when to proceed
4. **Never run review yourself** - User must invoke it
5. **Never start the next subtask** - Wait for explicit request

## Reference

See `lancelot/workflow` skill for the full implementation cycle.
