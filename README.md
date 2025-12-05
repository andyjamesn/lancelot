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

Because each subtask targets exactly one file, you can execute multiple subtasks in parallel—either with multiple terminals or using the built-in `/lancelot:parallel` command.

### Using `/lancelot:parallel`

```bash
# Auto-select and run next 3 pending subtasks
/lancelot:parallel

# Run specific subtasks
/lancelot:parallel abc123 def456 ghi789

# Run next 5 pending subtasks
/lancelot:parallel --count 5
```

The command:
1. Validates subtasks can run in parallel (no conflicts)
2. Spawns multiple agents simultaneously
3. Each agent implements one subtask (one file)
4. Reports results when all complete
5. Prompts you to review each

```
> /lancelot:parallel

Finding next 3 pending subtasks...

Spawning parallel agents:
├─ Agent 1: abc123 → src/models/User.ts
├─ Agent 2: def456 → src/services/AuthService.ts  
└─ Agent 3: ghi789 → src/controllers/AuthController.ts

⏳ Executing in parallel...

✅ All 3 subtasks completed!

Review each:
  /lancelot:review abc123
  /lancelot:review def456
  /lancelot:review ghi789
```

### Manual Parallel Execution

You can also run multiple Claude Code terminals:

```
┌──────────────────────────────────────────────┐
│  Terminal 1        Terminal 2        Terminal 3  │
│       │                 │                 │      │
│       ▼                 ▼                 ▼      │
│  /lancelot:prompt   /lancelot:prompt  /lancelot:prompt │
│     abc123             def456            ghi789   │
│       │                 │                 │      │
│       ▼                 ▼                 ▼      │
│  UserService.ts    UserForm.vue     userRoutes.ts │
└──────────────────────────────────────────────┘
```

### Why This Works

- **No git conflicts** — Different files = clean merges
- **3-5x faster** — Parallelize your feature development
- **Independent review** — Review and approve each file separately
- **Failure isolation** — One subtask failing doesn't block others
- **Dependency detection** — Lancelot warns if subtasks can't run in parallel

## Features

- **Atomic Planning**: Break features into milestones → tasks → subtasks → steps
- **One File Per Subtask**: Each subtask targets exactly one file—no sprawling changes
- **Parallel Execution**: `/lancelot:parallel` runs multiple subtasks simultaneously
- **Review Gates**: Only `/lancelot:review` can mark work complete
- **Stop Points**: Claude pauses after implementation, waits for review
- **Stack Expertise**: Auto-detect your tech stack and apply best practices
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
/lancelot:init
```

This analyzes your codebase and creates `.lancelot/config.json` with detected:
- Tech stack (Laravel, Vue, React, etc.)
- Coding patterns and conventions
- Anti-patterns to avoid

### 2. Create a plan

Discuss what you want to build, then:

```
/lancelot:plan
```

Lancelot will:
1. Explore your codebase (parallel agents)
2. Ask clarifying questions (one at a time)
3. Design a simple, focused plan
4. Save to `.lancelot/plans/`

### 3. Implement subtasks

**Sequential (one at a time):**
```
/lancelot:next          # See what's next
/lancelot:prompt abc123 # Implement subtask abc123
```

**Parallel (multiple at once):**
```
/lancelot:parallel      # Run next 3 subtasks in parallel
```

After implementation, Claude will **STOP** and wait for review.

### 4. Review and complete

```
/lancelot:review abc123
```

Reviews the implementation against plan steps. Only reports issues at 80+ confidence. If approved, asks for confirmation before marking complete.

### 5. Repeat

Continue with `/lancelot:next` or `/lancelot:parallel` until the plan is complete.

## Commands

| Command | Description |
|---------|-------------|
| `/lancelot:init` | Initialize Lancelot, detect stack, create config |
| `/lancelot:plan` | Create implementation plan from conversation |
| `/lancelot:status` | Show current plan progress |
| `/lancelot:next` | Show next subtask to implement |
| `/lancelot:prompt <id>` | Implement a specific subtask |
| `/lancelot:review <id>` | Review implementation, mark complete |
| `/lancelot:parallel [ids]` | Execute multiple subtasks in parallel |

## The Workflow

### Planning Phase

```
/lancelot:plan
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

**Sequential:**
```
/lancelot:prompt {id}
      │
      ▼
  Implement (ONE file only)
      │
      ▼
  ⛔ STOP — Wait for user
      │
      ▼
/lancelot:review {id}
      │
      ▼
  ⛔ STOP — Wait for user
```

**Parallel:**
```
/lancelot:parallel
      │
      ▼
┌─────────────────────────────┐
│ Agent 1 → file1.ts          │
│ Agent 2 → file2.ts          │
│ Agent 3 → file3.ts          │
└─────────────────────────────┘
      │
      ▼
  All complete
      │
      ▼
  Review each: /lancelot:review {id}
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
