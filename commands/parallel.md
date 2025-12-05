---
description: Execute multiple subtasks in parallel using agents
allowed-tools: Read, Write, Task, Bash
---

# Parallel Subtask Execution

Execute multiple subtasks simultaneously using parallel agents.

## How It Works

This command spawns multiple agents, each implementing a different subtask. Because each subtask targets exactly ONE file, there are no conflicts.

## Arguments

$ARGUMENTS

Usage:
- `/lancelot:parallel` — Auto-select next 3 pending subtasks
- `/lancelot:parallel abc123 def456 ghi789` — Specific subtask IDs
- `/lancelot:parallel --count 5` — Run next 5 pending subtasks

## Workflow

### Step 1: Load Plan

1. Read UUID from `.lancelot/active-plan`
2. Load plan from `.lancelot/plans/{uuid}.json`

### Step 2: Select Subtasks

If specific IDs provided:
- Use those subtasks

If no IDs (or --count N):
- Find next N pending subtasks (default: 3)
- Ensure they don't have dependencies on each other

### Step 3: Validate

Before spawning agents, verify:
- All subtasks exist and are pending
- No two subtasks target the same file
- Subtasks don't have blocking dependencies

If validation fails, show error and suggest which subtasks CAN run in parallel.

### Step 4: Spawn Parallel Agents

Launch agents using the Task tool IN PARALLEL:

```
For each subtask, spawn:

Task: implementation-agent
"Implement this subtask:

Subtask ID: {id}
Title: {title}
File: {targetFile} ({action})

Steps:
{steps with instructions and codeHints}

Follow the lancelot/prompt skill guidelines.
Implement all steps for this ONE file.

Return a summary of what was implemented."
```

**IMPORTANT: Launch all agents in a single message to run them in parallel.**

### Step 5: Collect Results

Wait for all agents to complete. For each:
- Note success/failure
- Capture summary of changes

### Step 6: Report

```
────────────────────────────────────────────
⚡ Parallel Execution Complete

Subtasks executed: {count}

Results:
✅ {subtask1.id} — {subtask1.title}
   File: {targetFile}
   
✅ {subtask2.id} — {subtask2.title}
   File: {targetFile}

❌ {subtask3.id} — {subtask3.title}
   Error: {error message}

────────────────────────────────────────────

Next steps:
1. Review each completed subtask:
   /lancelot:review {subtask1.id}
   /lancelot:review {subtask2.id}

2. Fix and retry failed subtasks:
   /lancelot:prompt {subtask3.id}
────────────────────────────────────────────
```

### Step 7: STOP

**Do NOT auto-review.** Wait for user to review each subtask individually.

## Dependency Detection

Subtasks should NOT run in parallel if:
- They modify the same file (never happens with Lancelot's one-file rule)
- Subtask B imports from a file Subtask A creates
- They're in a sequence that must be ordered

When dependencies detected:
```
⚠️ Cannot run these subtasks in parallel:

{subtaskA.id} creates: src/services/UserService.ts
{subtaskB.id} imports from: src/services/UserService.ts

Suggestion: Run {subtaskA.id} first, then {subtaskB.id}
```

## Example

```
> /lancelot:parallel

Finding next 3 pending subtasks...

Spawning parallel agents:
├─ Agent 1: abc123 → src/models/User.ts
├─ Agent 2: def456 → src/services/AuthService.ts  
└─ Agent 3: ghi789 → src/controllers/AuthController.ts

⏳ Executing in parallel...

✅ All 3 subtasks completed!

Review each:
  /lancelot:review abc123
  /lancelot:review def456
  /lancelot:review ghi789
```

## Notes

- Maximum recommended parallel subtasks: 5 (to manage context)
- Each agent uses the `lancelot/prompt` skill guidelines
- Failed subtasks can be retried individually with `/lancelot:prompt`
- Always review before marking complete
