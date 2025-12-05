# Implementation Prompt Skill

Generate and execute implementation prompts for plan subtasks.

## ⛔ CRITICAL: STOP RULES

**You MUST stop after implementing a subtask. Never:**
- Mark subtasks/steps as complete
- Modify the plan JSON file
- Auto-continue to the next subtask
- Run /lancelot/review automatically

**The cycle is:** `/prompt` → implement → **STOP** → user runs `/review`

## When to Use

- User runs `/lancelot/prompt {subtask-id}`
- User wants to implement a specific subtask

## Workflow

### 1. Load Plan

```
.lancelot/active-plan → UUID
.lancelot/plans/{UUID}.json → Plan data
Find subtask by ID prefix match
```

### 2. Gather Context

**For modify action:**
- Read current file contents
- Find related files showing patterns

**For create action:**
- Find similar files as templates
- Check for existing patterns

**For all:**
- Look up `referenceSymbols` from steps
- Identify conventions to follow

### 3. Generate Implementation Context

Before implementing, establish:

```markdown
## Target
File: {targetFile}
Action: {create|modify|delete}

## Steps
1. {instruction} [codeHint: {hint}]
2. {instruction}
...

## Patterns to Follow
{relevant code from codebase}

## Conventions
- {naming conventions}
- {structural patterns}
```

### 4. Implement

Execute each step:
- One logical change at a time
- Follow the codeHint patterns
- Match existing code style
- Handle edge cases appropriately

### 5. STOP AND REPORT

After implementation, output exactly:

```
────────────────────────────────────────────
✓ Implementation complete: {subtask.title}

File: {targetFile}
Steps completed: {count}

⏭ Next: Run /lancelot/review {subtask-id}
────────────────────────────────────────────
```

Then **STOP**. Wait for user input.

## Finding Subtasks

Subtask IDs can be:
- Full UUID: `a1b2c3d4-5678-90ab-cdef-1234567890ab`
- Partial (8 chars): `a1b2c3d4`

Search: `milestones[].tasks[].subtasks[].id`

## Forbidden Actions

| Action | Allowed? |
|--------|----------|
| Write/edit target file | ✅ Yes |
| Read reference files | ✅ Yes |
| Mark step complete | ❌ No |
| Mark subtask complete | ❌ No |
| Modify plan JSON | ❌ No |
| Start next subtask | ❌ No |
| Run /review | ❌ No |

## Anti-Pattern Examples

**WRONG:**
```
"Done! I've marked the subtask complete and will now start the next one..."
```

**WRONG:**
```
"Implementation finished. Let me now review it..."
[runs review]
```

**RIGHT:**
```
✓ Implementation complete: Add user validation

File: src/validators/user.ts
Steps completed: 3

⏭ Next: Run /lancelot/review a1b2c3d4
```
[STOP - wait for user]
