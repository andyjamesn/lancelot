---
description: Review a subtask implementation against plan requirements
allowed-tools: Read, Grep, Glob, Task, Write
---

# Review Implementation

Review a subtask implementation using the code-reviewer agent.

## ⛔ CRITICAL: CONFIRMATION REQUIRED

```
┌────────────────────────────────────────────┐
│  This is the ONLY place that can mark      │
│  subtasks as complete.                     │
│                                            │
│  ✓ Agent reviews the implementation        │
│  ✓ You show the review report              │
│  ✓ ASK user before marking complete        │
│  ✓ STOP after - don't auto-continue        │
└────────────────────────────────────────────┘
```

## Subtask ID

$ARGUMENTS

## Workflow

### Step 0: Ask for Spec Doc

**ALWAYS ask this question before reviewing:**

```
────────────────────────────────────────────
📋 Spec Document Alignment Check

Is there a spec doc that this implementation must align with?

This could be:
- A PRD or requirements document
- A design doc or RFC  
- An API specification
- Any markdown file with context

Enter the file path or URL, or press Enter to skip:
> 
────────────────────────────────────────────
```

- If user provides a path/URL: Read the spec doc content
- If user presses Enter/skips: Proceed without spec alignment

### Step 1: Load Context

1. Read UUID from `.lancelot/active-plan`
2. Load plan from `.lancelot/plans/{uuid}.json`
3. Find subtask by ID (partial match OK - first 8 chars)
4. Read the target file contents

### Step 2: Launch Code Reviewer Agent

```
Task: code-reviewer agent
"Review this implementation:

Subtask: {subtask.title}
File: {targetFile}
Description: {description}

Steps to verify:
{list all steps with instructions and codeHints}

Current file contents:
{file contents}

{IF spec_doc PROVIDED}
## Spec Document Context

This implementation must align with the following spec:

---
{spec_doc_content}
---

When reviewing, also verify:
- Implementation matches spec requirements
- No deviations from spec without justification
- All spec-defined behaviors are present
{END IF}

Analyze each step and provide evidence-based assessment."
```

### Step 3: Display Review Report

Show the agent's review report to the user.

### Step 4: Handle Result

**If APPROVED (all steps satisfied):**

Ask the user explicitly:
```
────────────────────────────────────────────
✅ All steps satisfied

Mark this subtask as complete?
Reply 'yes' to confirm.
────────────────────────────────────────────
```

**Wait for user response.** Only if they say "yes":

1. Update subtask status to "complete" in plan JSON
2. Update each step status to "complete"  
3. Update parent task/milestone status to "in_progress" if needed
4. Update plan's `updatedAt` timestamp
5. Save the plan file

Then show:
```
────────────────────────────────────────────
✓ Subtask marked complete: {title}

Progress: {completed}/{total} subtasks

⏭ Next: Run /lancelot/next to see what's next
────────────────────────────────────────────
```

**If NEEDS WORK (any partial/missing):**

Show the issues:
```
────────────────────────────────────────────
⚠️ Needs work: {title}

Issues to fix:
{issues from agent review}

After fixing, run: /lancelot/review {id}
────────────────────────────────────────────
```

### Step 5: STOP

After review (whether approved or needs work):
- **STOP** and wait for user
- Do NOT auto-start the next subtask
- Do NOT run `/lancelot/prompt` for next item

## Agent Pipeline

```
Load Plan & Subtask
        │
        ▼
Read Target File
        │
        ▼
┌───────────────────┐
│   code-reviewer   │──→ Detailed analysis
└───────────────────┘
        │
        ▼
Display Report
        │
        ▼
    APPROVED? ──No──→ Show issues → STOP
        │Yes
        ▼
Ask user "Mark complete?"
        │
        ▼
  User says "yes"? ──No──→ STOP
        │Yes
        ▼
Update plan JSON
        │
        ▼
      STOP
```

## Reference

See `lancelot/workflow` skill for the full implementation cycle.
