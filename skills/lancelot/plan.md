# Implementation Planning Skill

Create detailed, actionable implementation plans from conversation context.

## Overview

Transform requirements from conversation into a structured implementation plan with atomic, verifiable steps.

## When to Use

- User runs `/lancelot/plan`
- Requirements have been discussed and need formalization
- User wants to break down a feature into executable steps

## Planning Phases

### Phase 1: Context Summary

Summarize from conversation:
- What feature/change is being requested?
- Key requirements and constraints
- User preferences mentioned

### Phase 2: Codebase Exploration

**REQUIRED** before generating a plan.

Use `code-explorer` agents (in parallel) to find:
- Project structure (package.json, tsconfig.json, configs)
- Existing patterns and similar implementations
- Naming conventions and code style

### Phase 3: Clarifying Questions

If requirements are unclear, ask 1-2 focused questions.
Skip if requirements are clear.

### Phase 4: Architecture Design

Use `plan-architect` agent to design:
- Milestones (feature-level)
- Tasks (component-level)
- Subtasks (file-level) - the unit of work
- Step outlines (atomic changes)

### Phase 5: Generate Plan JSON

Use `plan-builder` agent to generate valid JSON following `lancelot/schema`.

### Phase 6: Save Plan

```
1. mkdir -p .lancelot/plans/
2. Save to .lancelot/plans/{id}.json
3. Write ID to .lancelot/active-plan
```

### Phase 7: Display Summary

```
────────────────────────────────────────────
Plan Created: {title}
ID: {first 8 chars}

Milestones: {count}
├─ Tasks: {count}
│  └─ Subtasks: {count}
│     └─ Steps: {count}

⏭ Next: /lancelot/status or /lancelot/prompt {id}
────────────────────────────────────────────
```

## Plan Quality Checklist

Before saving, verify:

- [ ] All subtasks have unique UUIDs
- [ ] Each subtask has exactly one `targetFile`
- [ ] All `parentId` references are correct
- [ ] Steps are truly atomic (one thing each)
- [ ] No conditional logic in steps
- [ ] Complexity ratings are realistic
- [ ] Risks are meaningful, not generic

## Subtask Guidelines

### Action Types

| Action | When | File Must |
|--------|------|-----------|
| `create` | New file | Not exist |
| `modify` | Change existing | Exist |
| `delete` | Remove file | Exist |

### For Renames

Don't use "rename" action. Instead:
1. Subtask 1: `create` new file with content
2. Subtask 2: `modify` files that import it
3. Subtask 3: `delete` old file

## Step Guidelines

### Atomicity Criteria

A step is atomic if it's:
- Single file only (the subtask's targetFile)
- One logical change
- No conditional branching
- Verifiable completion
- Self-contained

### Good vs Bad Steps

**Good:**
```
"Add validateEmail function that checks format using regex"
```

**Bad:**
```
"Add validation - if it's email use regex, if phone use different pattern"
(Branching logic - split into separate steps)
```

**Bad:**
```
"Update user.ts and also add tests in user.test.ts"
(Multiple files - split into separate subtasks)
```

### Complexity Ratings

| Rating | Description |
|--------|-------------|
| `trivial` | Single line, obvious change |
| `low` | Simple addition, clear pattern |
| `medium` | Requires context, some decisions |
| `high` | Complex logic, multiple considerations |
| `complex` | Architectural impact |

## Do NOT

- Generate plans without exploring codebase first
- Create subtasks spanning multiple files
- Include conditional logic in steps
- Over-engineer (keep plans focused)
- Add generic risks (be specific)
- Skip the save step
