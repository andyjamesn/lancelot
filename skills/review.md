# Code Review Skill

Review implementations against plan requirements and manage completion status.

## ⛔ CRITICAL: ONLY REVIEW CAN MARK COMPLETE

This skill is the **sole authority** for marking subtasks complete.

**Rules:**
1. Only mark complete if ALL steps are satisfied
2. Always ask user for explicit confirmation
3. STOP after review - never auto-continue
4. User must explicitly say "yes" to mark complete

## When to Use

- User runs `/lancelot/review {subtask-id}`
- User wants to verify implementation
- User asks "is this done correctly?"

## Workflow

### 1. Load Context

```
.lancelot/active-plan → UUID
.lancelot/plans/{UUID}.json → Plan data
Find subtask → Read targetFile
```

### 2. Analyze Steps

For each step, determine status:

| Status | Criteria |
|--------|----------|
| ✅ satisfied | Instruction followed, codeHint applied (if any), verification passes |
| ⚠️ partial | Some aspects done, others missing |
| ❌ missing | Not implemented at all |

**Evidence-based analysis:**
- Quote actual code that satisfies/violates the step
- Reference specific line numbers
- Note any deviations from `codeHint`

### 3. Generate Report

```markdown
# Review: {subtask.title}

**File:** `{targetFile}`
**ID:** {id first 8 chars}

## Step Analysis

### Step 1: {instruction}
**Status:** ✅ satisfied
**Evidence:** Found at line 42: `const validate = ...`

### Step 2: {instruction}
**Status:** ⚠️ partial
**Evidence:** Error handling exists but missing edge case
**Issue:** No handling for empty input

### Step 3: {instruction}
**Status:** ❌ missing
**Issue:** Function not implemented

## Summary

| Status | Count |
|--------|-------|
| ✅ Satisfied | 1 |
| ⚠️ Partial | 1 |
| ❌ Missing | 1 |
| **Total** | 3 |

## Result: NEEDS WORK
```

### 4. Handle Result

**APPROVED (all satisfied):**

```
────────────────────────────────────────────
✅ All steps satisfied

Mark this subtask as complete?
Reply 'yes' to confirm.
────────────────────────────────────────────
```

**Wait for explicit "yes" from user before updating plan JSON.**

After user confirms:
```json
// Update in .lancelot/plans/{id}.json
{
  "subtask.status": "complete",
  "steps[].status": "complete",
  "plan.updatedAt": "{ISO timestamp}"
}
```

Then:
```
────────────────────────────────────────────
✓ Subtask marked complete: {title}

Progress: {completed}/{total} subtasks

⏭ Next: Run /lancelot/next to see what's next
────────────────────────────────────────────
```

**NEEDS WORK (any partial/missing):**

```
────────────────────────────────────────────
⚠️ Needs work: {title}

Issues:
1. Step 2: {specific issue}
2. Step 3: {specific issue}

Fix these issues, then run:
/lancelot/review {id}
────────────────────────────────────────────
```

### 5. STOP

After review completes:
- **STOP** - do not continue
- Do not auto-start next subtask
- Do not run `/lancelot/prompt`
- Wait for user to decide next action

## Confirmation Flow

```
┌─────────────────────────────────────┐
│ Review complete                     │
│         ↓                           │
│ All satisfied? ─No──→ Show issues   │
│         │Yes                  ↓     │
│         ↓                   STOP    │
│ Ask: "Mark complete?"               │
│         ↓                           │
│ User says "yes"? ─No──→ STOP        │
│         │Yes                        │
│         ↓                           │
│ Update plan JSON                    │
│         ↓                           │
│ Show confirmation                   │
│         ↓                           │
│       STOP                          │
└─────────────────────────────────────┘
```

## JSON Update Details

When marking complete:

```javascript
// Find and update the subtask
subtask.status = "complete"
subtask.steps.forEach(step => step.status = "complete")

// Update parent statuses if needed
if (task.status === "pending") task.status = "in_progress"
if (milestone.status === "pending") milestone.status = "in_progress"

// Update timestamp
plan.updatedAt = new Date().toISOString()

// Save to file
```

## Forbidden Actions

| Action | Allowed? |
|--------|----------|
| Read target file | ✅ Yes |
| Analyze steps | ✅ Yes |
| Update plan JSON (after confirm) | ✅ Yes |
| Mark complete without asking | ❌ No |
| Auto-continue to next subtask | ❌ No |
| Run /lancelot/prompt | ❌ No |
