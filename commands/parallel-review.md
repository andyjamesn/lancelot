---
description: Review multiple completed subtasks in parallel using agents
allowed-tools: Read, Write, Task, Bash, Glob, Grep
---

# Parallel Review Execution

Review multiple completed subtasks simultaneously using parallel agents.

## Pre-Review Question

**IMPORTANT: Before spawning review agents, ALWAYS ask:**

```
Is there a spec doc that this implementation must align with?

Enter the file path or URL to the spec doc, or press Enter to skip:
> 
```

- If user provides a path/URL: Read the spec doc and include its content in each agent's context
- If user presses Enter/skips: Proceed without spec alignment checking

## Arguments

$ARGUMENTS

Usage:
- `/lancelot:parallel-review` — Auto-select next 3 completed (unreviewed) subtasks
- `/lancelot:parallel-review abc123 def456` — Review specific subtask IDs
- `/lancelot:parallel-review --count 5` — Review next 5 completed subtasks

## Workflow

### Step 1: Ask for Spec Doc

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

If provided:
- Read the spec doc content
- This will be passed to each review agent

### Step 2: Load Plan

1. Read UUID from `.lancelot/active-plan`
2. Load plan from `.lancelot/plans/{uuid}.json`

### Step 3: Select Subtasks

If specific IDs provided:
- Use those subtasks (verify they're completed)

If no IDs (or --count N):
- Find next N completed but unreviewed subtasks (default: 3)

### Step 4: Validate

Before spawning agents, verify:
- All subtasks exist and are completed (not pending)
- Subtasks have implementation to review

If validation fails, show error.

### Step 5: Spawn Parallel Review Agents

Launch agents using the Task tool IN PARALLEL:

```
For each subtask, spawn:

Task: code-reviewer
"Review this completed subtask:

Subtask ID: {id}
Title: {title}
File: {targetFile}

Steps that were implemented:
{steps with instructions and codeHints}

{IF spec_doc PROVIDED}
## Spec Document Context

This implementation must align with the following spec:

---
{spec_doc_content}
---

When reviewing, verify:
- Implementation matches spec requirements
- No deviations from spec without justification
- All spec-defined behaviors are present
{END IF}

Follow the code-reviewer agent guidelines:
1. Verify all imports exist
2. Detect hallucinations (fake APIs, methods, types)
3. Check for over-engineering
4. Verify step compliance
5. Categorize issues as Critical/Major/Minor
{IF spec_doc}6. Check spec alignment{END IF}

Return your full review with severity-categorized issues."
```

**IMPORTANT: Launch all agents in a single message to run them in parallel.**

### Step 6: Collect Results

Wait for all agents to complete. For each:
- Capture review findings
- Note Critical/Major/Minor counts
- Note APPROVED or NEEDS WORK status

### Step 7: Report Summary

```
────────────────────────────────────────────
🔍 Parallel Review Complete

Subtasks reviewed: {count}
{IF spec_doc}Spec alignment: {spec_doc_path}{END IF}

Results:
────────────────────────────────────────────

✅ APPROVED: {subtask1.id} — {subtask1.title}
   File: {targetFile}
   Issues: 0 Critical, 0 Major, 1 Minor

⚠️ NEEDS WORK: {subtask2.id} — {subtask2.title}
   File: {targetFile}
   Issues: 1 Critical, 2 Major, 0 Minor
   Critical: Hallucinated import 'validateEmail'

⚠️ NEEDS WORK: {subtask3.id} — {subtask3.title}
   File: {targetFile}
   Issues: 0 Critical, 1 Major, 3 Minor
   Major: Over-engineered validation class

────────────────────────────────────────────

Summary:
- ✅ Approved: 1
- ⚠️ Needs Work: 2
- Total Critical Issues: 1
- Total Major Issues: 3
- Total Minor Issues: 4

────────────────────────────────────────────

Next steps:

For approved subtasks:
  Mark as reviewed in plan

For subtasks needing work:
  1. Fix issues in {subtask2.targetFile}
     → /lancelot:review {subtask2.id}
  
  2. Fix issues in {subtask3.targetFile}
     → /lancelot:review {subtask3.id}

────────────────────────────────────────────
```

### Step 8: STOP

**Do NOT auto-fix issues.** Wait for user to address each subtask's issues.

## Spec Alignment Checks

When a spec doc is provided, each review agent will additionally check:

| Check | What It Catches |
|-------|-----------------|
| Missing requirements | Spec says X, but X isn't implemented |
| Wrong behavior | Implementation does Y, spec says Z |
| Extra features | Implementation has features not in spec |
| API mismatches | Endpoints/methods don't match spec |
| Data model issues | Schema doesn't match spec definition |

These are categorized as:
- 🔴 **Critical**: Missing required spec feature
- 🟠 **Major**: Behavior differs from spec
- 🟡 **Minor**: Minor deviation from spec style/naming

## Example

```
> /lancelot:parallel-review

────────────────────────────────────────────
📋 Spec Document Alignment Check

Is there a spec doc that this implementation must align with?
Enter the file path or URL, or press Enter to skip:
> ./docs/auth-feature-prd.md

Reading spec doc...

Finding next 3 completed subtasks...

Spawning parallel review agents:
├─ Reviewer 1: abc123 → src/models/User.ts
├─ Reviewer 2: def456 → src/services/AuthService.ts  
└─ Reviewer 3: ghi789 → src/controllers/AuthController.ts

⏳ Reviewing in parallel...

────────────────────────────────────────────
🔍 Parallel Review Complete

Results:
✅ abc123 — APPROVED (0 Critical, 0 Major, 0 Minor)
⚠️ def456 — NEEDS WORK (1 Critical, 0 Major, 2 Minor)
✅ ghi789 — APPROVED (0 Critical, 0 Major, 1 Minor)

Fix issues then re-review:
  /lancelot:review def456
────────────────────────────────────────────
```

## Notes

- Maximum recommended parallel reviews: 5
- Spec doc can be local file or URL
- Each agent uses opus model for thorough review
- Critical issues block approval regardless of spec
