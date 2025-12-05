# Plan Status Skill

Display and calculate implementation plan progress.

## When to Use

- User runs `/lancelot/status`
- User asks "how much is done?" or "what's the progress?"
- User wants to see plan overview

## Workflow

### 1. Load Plan

```
.lancelot/active-plan → UUID
.lancelot/plans/{UUID}.json → Plan data
```

If no active plan:
```
No active plan found.
Run /lancelot/plan to create one.
```

### 2. Calculate Statistics

```javascript
// Count by status
const stats = {
  milestones: { complete: 0, inProgress: 0, pending: 0 },
  tasks: { complete: 0, inProgress: 0, pending: 0 },
  subtasks: { complete: 0, inProgress: 0, pending: 0 },
  steps: { complete: 0, pending: 0 }
}

// Traverse and count
milestones.forEach(m => {
  stats.milestones[m.status]++
  m.tasks.forEach(t => {
    stats.tasks[t.status]++
    t.subtasks.forEach(s => {
      stats.subtasks[s.status]++
      s.steps.forEach(step => {
        stats.steps[step.status]++
      })
    })
  })
})
```

### 3. Find Next Subtask

Traverse in order to find first pending:
```
milestones (ordered) → tasks (ordered) → subtasks (ordered)
→ first where status === "pending"
```

### 4. Display Progress

```
────────────────────────────────────────────
Plan: {title}
ID: {first 8 chars}

Progress: {completed}/{total} subtasks ({percentage}%)
[████████░░░░░░░░░░░░] 

Milestones:
{visual tree}

Next: {subtask.id} - {subtask.title}
────────────────────────────────────────────
```

## Visual Tree Format

```
Milestones:
├─ ✅ Setup project structure
├─ 🔄 Implement user authentication (2/4 tasks)
│  ├─ ✅ Create user model
│  ├─ 🔄 Add login endpoint (1/3 subtasks)
│  │  ├─ ✅ Create route handler
│  │  ├─ ➡️ Add validation ← NEXT
│  │  └─ ⬚ Add tests
│  ├─ ⬚ Add logout endpoint
│  └─ ⬚ Add session management
└─ ⬚ Add admin dashboard
```

## Status Icons

| Icon | Status | Meaning |
|------|--------|---------|
| ✅ | complete | Done |
| 🔄 | in_progress | Has some complete children |
| ➡️ | pending | Next to do (marked) |
| ⬚ | pending | Not started |

## Progress Bar

```
Total: 15 subtasks
Complete: 6

[████████░░░░░░░░░░░░] 40%
 ^^^^^^^^            
 6/15 filled
```

## After Display

STOP and wait for user decision:
- `/lancelot/next` - See next subtask details
- `/lancelot/prompt {id}` - Start implementing
