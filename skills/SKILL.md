---
name: lancelot
description: Structured implementation planning system. Use when creating implementation plans, tracking progress, implementing subtasks, or reviewing code against plan requirements.
---

# Lancelot Planning System

Lancelot provides structured, granular implementation planning with enforced review gates.

## When to Use

Activate this skill when:
- Creating implementation plans for features
- Breaking down complex work into atomic steps
- Tracking implementation progress
- Implementing subtasks from a plan
- Reviewing code against plan requirements

## Commands

| Command | Purpose |
|---------|---------|
| `/lancelot/plan` | Create a plan from conversation |
| `/lancelot/status` | Show plan progress |
| `/lancelot/next` | Show next subtask |
| `/lancelot/prompt <id>` | Implement a subtask |
| `/lancelot/review <id>` | Review implementation |

## Core Concepts

### Plan Hierarchy
```
Plan
├── Milestones (feature-level)
│   └── Tasks (component-level)
│       └── Subtasks (file-level) ← Unit of work
│           └── Steps (atomic changes)
```

### The Golden Rules

1. **One file per subtask** - Never combine multiple files
2. **Atomic steps** - One logical change per step
3. **Review gates** - Only `/lancelot/review` can mark complete
4. **STOP points** - Always pause after prompt/review

### The Workflow Cycle

```
/lancelot/prompt {id}
      │
      ▼
  Implement
      │
      ▼
   ⛔ STOP ──→ User runs /lancelot/review {id}
                    │
                    ▼
                 Review
                    │
                    ▼
               ⛔ STOP
```

## Related Skills

- `lancelot/schema` - Plan JSON structure
- `lancelot/workflow` - Detailed workflow rules
- `lancelot/prompt` - Implementation guidelines
- `lancelot/review` - Review process

## Agents Used

| Agent | Purpose |
|-------|---------|
| `code-explorer` | Find patterns in codebase |
| `conventions-checker` | Extract coding conventions |
| `plan-architect` | Design simple, focused plans |
| `plan-builder` | Generate valid Plan JSON |
| `code-reviewer` | Verify implementations |

## Philosophy

> Keep it simple. Follow conventions. Ask questions.

Lancelot plans should be:
- **Minimal** - Only what's needed
- **Convention-compliant** - Match the codebase
- **Atomic** - Small, verifiable steps
- **Clear** - Anyone can follow them
