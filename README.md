# Lancelot

> Structured implementation planning for Claude Code

## Why Lancelot?

When Claude builds features, it often jumps straight into coding—making decisions on the fly, touching multiple files at once, and sometimes going in circles. **Lancelot changes that.**

Lancelot enforces a disciplined workflow:

1. **Plan first** — Break down work into atomic, verifiable steps before writing code
2. **One file at a time** — Each subtask targets exactly ONE file, making changes traceable and reviewable
3. **Review gates** — Code can't be marked "complete" without explicit review and approval
4. **Stop points** — Claude pauses after each implementation, waiting for your review before continuing

### The Problem

Without structure, AI-assisted coding often leads to:
- Scope creep ("while I'm here, let me also refactor...")
- Multi-file changes that are hard to review
- Skipped steps and forgotten edge cases
- No clear checkpoint to verify work

### The Solution

Lancelot breaks features into a hierarchy:

```
Plan
├── Milestone (feature-level)
│   └── Task (component-level)
│       └── Subtask (ONE file) ← The unit of work
│           └── Steps (atomic changes)
```

**The key insight: one file per subtask.** This means:
- Every change is isolated and reviewable
- You can verify each piece before moving on
- Nothing gets marked complete without your approval
- Claude can't "run ahead" and make sweeping changes

## Parallel Execution

Because each subtask targets exactly one file, you can run multiple Claude Code instances in parallel—each working on a different subtask without conflicts.

```
/lancelot/plan
      │
      ▼
  Generate subtasks (each = one file)
      │
      ▼
┌──────────────────────────────────────────────┐
│  Terminal 1        Terminal 2        Terminal 3  │
│  Claude Agent 1    Claude Agent 2    Claude Agent 3  │
│       │                 │                 │      │
│       ▼                 ▼                 ▼      │
│  /lancelot/prompt   /lancelot/prompt  /lancelot/prompt │
│     abc123             def456            ghi789   │
│       │                 │                 │      │
│       ▼                 ▼                 ▼      │
│  UserService.ts    UserForm.vue     userRoutes.ts │
└──────────────────────────────────────────────┘
      │
      ▼
  Review each as they complete
```

### Why This Works

- **No git conflicts** — Different files = clean merges
- **3-5x faster** — Parallelize your feature development
- **Independent review** — Review and approve each file separately
- **Failure isolation** — One subtask failing doesn't block others
- **Natural batching** — Group related subtasks by dependency

### How To Use

1. Run `/lancelot/plan` to create your plan
2. Run `/lancelot/status` to see all subtasks
3. Open multiple terminals with Claude Code
4. In each terminal, run `/lancelot/prompt <different-subtask-id>`
5. Review each with `/lancelot/review` as they complete

The one-file-per-subtask constraint isn't a limitation—it's what makes parallel execution possible.

## Features

- **Atomic Planning**: Break features into milestones → tasks → subtasks → steps
- **One File Per Subtask**: Each subtask targets exactly one file—no sprawling changes
- **Parallel Execution**: Run multiple Claude instances on different subtasks
- **Review Gates**: Only `/lancelot/review` can mark work complete
- **Stop Points**: Claude pauses after implementation, waits for review
- **Stack Expertise**: Auto-detect your tech stack and apply best practices
- **Parallel Agents**: Fast codebase exploration with specialized agents
- **Confidence Scoring**: Reviews only flag issues at 80+ confidence
- **Convention Following**: Automatically matches your codebase patterns

## Installation

**1. Add the marketplace:**
```
/plugin marketplace add andyjamesn/lancelot
```

**2. Install the plugin:**
```
/plugin install lancelot@andyjamesn
```

**3. Restart Claude Code**

### Updating

To update to the latest version:
```bash
rm -rf ~/.claude/plugins/marketplaces/andyjamesn-lancelot
```
Then reinstall using the commands above.

### Local Development

For local development or testing:
```bash
git clone https://github.com/andyjamesn/lancelot.git ~/lancelot
/plugin marketplace add ~/lancelot
/plugin install lancelot@lancelot-marketplace
```

## Quick Start

### 1. Initialize in your project

```
/lancelot/init
```

This analyzes your codebase and creates `.lancelot/config.json` with detected:
- Tech stack (Laravel, Vue, React, etc.)
- Coding patterns and conventions
- Anti-patterns to avoid

### 2. Create a plan

Discuss what you want to build, then:

```
/lancelot/plan
```

Lancelot will:
1. Explore your codebase (parallel agents)
2. Ask clarifying questions (one at a time)
3. Design a simple, focused plan
4. Save to `.lancelot/plans/`

### 3. Implement subtasks

```
/lancelot/next          # See what's next
/lancelot/prompt abc123 # Implement subtask abc123
```

After implementation, Claude will **STOP** and wait for review.

### 4. Review and complete

```
/lancelot/review abc123
```

Reviews the implementation against plan steps. Only reports issues at 80+ confidence. If approved, asks for confirmation before marking complete.

### 5. Repeat

Continue with `/lancelot/next` until the plan is complete.

## Commands

| Command | Description |
|---------|-------------|
| `/lancelot/init` | Initialize Lancelot, detect stack, create config |
| `/lancelot/plan` | Create implementation plan from conversation |
| `/lancelot/status` | Show current plan progress |
| `/lancelot/next` | Show next subtask to implement |
| `/lancelot/prompt <id>` | Implement a specific subtask |
| `/lancelot/review <id>` | Review implementation, mark complete |

## The Workflow

### Planning Phase

```
/lancelot/plan
      │
      ▼
┌─────────────────────┐
│ code-explorer ×2    │ (parallel)
│ conventions-checker │
└─────────────────────┘
      │
      ▼
  Clarifying Questions (one at a time)
      │
      ▼
┌─────────────────┐
│ plan-architect  │ → Simple, focused design
└─────────────────┘
      │
      ▼
┌─────────────────┐
│ plan-builder    │ → Valid JSON
└─────────────────┘
      │
      ▼
  Save to .lancelot/plans/
```

### Implementation Cycle

For each subtask (one file at a time):

```
/lancelot/prompt {id}
      │
      ▼
  Implement (ONE file only)
      │
      ▼
  ⛔ STOP — Wait for user
      │
      ▼
/lancelot/review {id}
      │
      ▼
  Review against steps
      │
      ▼
  Ask: "Mark complete?"
      │
      ▼
  ⛔ STOP — Wait for user
      │
      ▼
  (repeat with next subtask)
```

## Project Config

`.lancelot/config.json` stores your project's expertise:

```json
{
  "version": "1.0",
  "stack": {
    "backend": "Laravel 11, PHP 8.3",
    "frontend": "Vue 3.4, Inertia 1.0"
  },
  "expertise": {
    "patterns": [
      "Laravel Actions pattern",
      "Composition API with <script setup>",
      "Pinia for state management"
    ],
    "testing": {
      "backend": "Pest PHP",
      "frontend": "Vitest"
    }
  },
  "avoid": [
    "Repository pattern",
    "Options API",
    "Raw SQL queries"
  ]
}
```

This config is injected into agents, making them experts in YOUR stack.

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `code-explorer` | haiku | Fast codebase exploration |
| `conventions-checker` | haiku | Extract coding conventions |
| `plan-architect` | sonnet | Design simple, focused plans |
| `plan-builder` | sonnet | Generate valid Plan JSON |
| `code-reviewer` | opus | High-quality implementation review |

## Philosophy

> "Keep it simple. Follow conventions. Ask questions. One file at a time."

Lancelot plans should be:
- **Minimal** — Only what's needed, no over-engineering
- **Atomic** — One file per subtask, one change per step
- **Verifiable** — Every step can be checked
- **Convention-compliant** — Match the codebase patterns
- **Parallelizable** — Independent subtasks can run concurrently

## File Structure

```
.lancelot/
├── config.json      # Project expertise config
├── plans/           # Generated plans
│   └── {uuid}.json
└── active-plan      # Current plan ID
```

## Hooks

Lancelot includes workflow enforcement hooks:

- **lancelot-workflow-guard**: Warns on plan JSON modifications outside review
- **lancelot-prompt-reminder**: Reminds about review after implementation

## License

MIT
