# Plan Schema Reference

This document defines the JSON schema for implementation plans used by the planner skills.

## Plan Structure

```json
{
  "schemaVersion": "1.0",
  "id": "<UUID v4>",
  "title": "<short title for the feature>",
  "summary": "<2-3 sentence description>",
  "status": "active",
  "milestones": [...],
  "risks": [...],
  "questions": [],
  "questionsResolved": true,
  "assumptions": ["<assumptions made>"],
  "createdAt": "<ISO timestamp>",
  "updatedAt": "<ISO timestamp>"
}
```

## Milestone Structure

```json
{
  "id": "<UUID>",
  "planId": "<same as plan id>",
  "title": "<feature-level milestone>",
  "description": "<what this milestone achieves>",
  "order": 1,
  "status": "pending",
  "tasks": [...]
}
```

## Task Structure

```json
{
  "id": "<UUID>",
  "milestoneId": "<parent milestone id>",
  "title": "<component-level task>",
  "description": "<what this task achieves>",
  "order": 1,
  "complexity": "low|medium|high",
  "dependencies": [],
  "status": "pending",
  "subtasks": [...]
}
```

## Subtask Structure

```json
{
  "id": "<UUID>",
  "taskId": "<parent task id>",
  "title": "<file-level subtask>",
  "description": "<what changes in this file>",
  "order": 1,
  "targetFile": "<exact file path>",
  "action": "create|modify|delete",
  "status": "pending",
  "steps": [...]
}
```

## Step Structure

```json
{
  "id": "<UUID>",
  "subtaskId": "<parent subtask id>",
  "order": 1,
  "instruction": "<atomic, clear instruction>",
  "codeHint": "<optional code pattern>",
  "referenceSymbols": ["<existing symbols to reference>"],
  "verification": "<how to verify this step>",
  "status": "pending"
}
```

## Risk Structure

```json
{
  "description": "<potential risk>",
  "severity": "low|medium|high",
  "mitigation": "<how to mitigate>"
}
```

## Status Values

- `pending` - Not started
- `in_progress` - Currently being worked on
- `complete` - Finished successfully
- `blocked` - Waiting on something
- `skipped` - Intentionally skipped
- `failed` - Failed to complete

## File Locations

Plans are stored in `.lancelot/plans/` directory:
- Each plan is saved as `{plan-id}.json`
- The active plan ID is stored in `.lancelot/active-plan`

## UUID Format

Generate UUIDs in v4 format: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`

## Critical Rules

1. **Subtasks target ONE file** - Each subtask's `targetFile` is a single file path
2. **Steps are atomic** - One logical change per step, no branching logic
3. **Questions array is empty** - Questions are handled before plan generation
4. **All IDs are UUIDs** - Use proper v4 UUID format
5. **Status starts as pending** - All items begin with `"status": "pending"`
