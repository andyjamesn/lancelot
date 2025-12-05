# Lancelot

> Structured implementation planning for Claude Code

Lancelot brings systematic, granular implementation planning to your Claude Code workflow. Break down features into atomic steps, enforce review gates, and leverage stack-specific expertise.

## Features

- **Atomic Planning**: Break features into milestones → tasks → subtasks → steps
- **Review Gates**: Only `/lancelot/review` can mark work complete
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

### Local Development

For local development or testing:
```
git clone https://github.com/andyjamesn/lancelot.git ~/lancelot
/plugin marketplace add ~/lancelot
/plugin install lancelot@local
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

```
/lancelot/plan
```

Discuss what you want to build, then run this command. Lancelot will:
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

Reviews the implementation against plan steps. Only approves issues at 80+ confidence. If approved, asks for confirmation before marking complete.

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

```
/lancelot/plan
      │
      ▼
┌─────────────────┐
│ code-explorer ×2│ (parallel)
│ conventions-    │
│ checker         │
└─────────────────┘
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

Then for each subtask:

```
/lancelot/prompt {id}
      │
      ▼
  Implement
      │
      ▼
  ⛔ STOP
      │
      ▼
/lancelot/review {id}
      │
      ▼
  ⛔ STOP
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

> "Keep it simple. Follow conventions. Ask questions."

Lancelot plans should be:
- **Minimal** - Only what's needed
- **Convention-compliant** - Match the codebase
- **Atomic** - Small, verifiable steps
- **Clear** - Anyone can follow them

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
