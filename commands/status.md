# Show Plan Status

Display the current implementation plan progress.

## Instructions

1. Read the active plan ID from `.lancelot/active-plan`
2. Load the plan from `.lancelot/plans/{id}.json`
3. Calculate progress statistics
4. Display a visual progress report

## Output Format

```
────────────────────────────────────────────
Plan: {title}
ID: {first 8 chars}
Status: {status}

Progress: {completed}/{total} subtasks ({percentage}%)
[████████░░░░░░░░░░░░] 

Milestones:
├─ ✅ {completed milestone}
├─ 🔄 {in-progress milestone} ({x}/{y} tasks)
│  ├─ ✅ {completed task}
│  ├─ 🔄 {current task} ({a}/{b} subtasks)
│  │  ├─ ✅ {completed subtask}
│  │  ├─ ➡️ {current subtask} ← NEXT
│  │  └─ ⬚ {pending subtask}
│  └─ ⬚ {pending task}
└─ ⬚ {pending milestone}

Next subtask: {id} - {title}
────────────────────────────────────────────
```

## Status Icons

- ✅ Complete
- 🔄 In Progress
- ➡️ Next (current)
- ⬚ Pending

## If No Active Plan

```
────────────────────────────────────────────
No active plan found.

Run /lancelot/plan to create one from your conversation.
────────────────────────────────────────────
```

## After Display

STOP and wait for user to decide next action.
