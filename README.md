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

---

## Command Reference

### `/lancelot:init`

**Initialize Lancelot in your project.**

Run this once per project to analyze your codebase and create a configuration file.

```
/lancelot:init
```

**What it does:**
1. Scans your project for `package.json`, `composer.json`, config files
2. Detects your tech stack (Laravel, Vue, React, etc.)
3. Identifies coding patterns and conventions
4. Creates `.lancelot/config.json` with detected settings
5. Asks if you want to customize anything

**Output:**
```
.lancelot/
├── config.json    ← Project configuration
└── plans/         ← Plans will be saved here
```

**When to use:**
- First time using Lancelot in a project
- After major stack changes
- To regenerate config with `--force`

---

### `/lancelot:plan`

**Create an implementation plan from your conversation.**

Discuss what you want to build with Claude, then run this command to formalize it into a structured plan.

```
/lancelot:plan
```

**What it does:**
1. Summarizes requirements from your conversation
2. Launches parallel agents to explore your codebase
3. Asks clarifying questions (one at a time)
4. Designs a simple, focused implementation plan
5. Saves to `.lancelot/plans/{uuid}.json`

**The planning flow:**
```
Your conversation
      │
      ▼
┌─────────────────────┐
│ code-explorer ×2    │ (parallel)
│ conventions-checker │
└─────────────────────┘
      │
      ▼
Clarifying questions (one at a time)
      │
      ▼
┌─────────────────┐
│ plan-architect  │ → Designs the plan
└─────────────────┘
      │
      ▼
┌─────────────────┐
│ plan-builder    │ → Generates JSON
└─────────────────┘
      │
      ▼
Saved to .lancelot/plans/
```

**Tips:**
- Be specific about what you want before running
- The more context in the conversation, the better the plan
- Lancelot will ask questions if requirements are unclear

---

### `/lancelot:status`

**Show the current plan's progress.**

```
/lancelot:status
```

**Output:**
```
────────────────────────────────────────────
Plan: Add user authentication
ID: a1b2c3d4

Progress: 4/12 subtasks (33%)
[████████░░░░░░░░░░░░] 

Milestones:
├─ ✅ Setup authentication config
├─ 🔄 Implement login flow (2/4 tasks)
│  ├─ ✅ Create User model
│  ├─ 🔄 Add login endpoint (1/3 subtasks)
│  │  ├─ ✅ Create route handler
│  │  ├─ ➡️ Add validation ← NEXT
│  │  └─ ⬚ Add tests
│  └─ ⬚ Add logout endpoint
└─ ⬚ Add session management

Next: a1b2c3d4 — Add validation
────────────────────────────────────────────
```

**When to use:**
- To see overall progress
- To find the next subtask ID
- To understand what's left

---

### `/lancelot:next`

**Show details of the next subtask to implement.**

```
/lancelot:next
```

**Output:**
```
────────────────────────────────────────────
Next Subtask

ID: a1b2c3d4
Title: Add request validation
File: src/requests/LoginRequest.ts (create)

Milestone: Implement login flow
Task: Add login endpoint

Description:
Create a validation request class for login credentials

Steps (3):
1. Create LoginRequest class with email and password rules
2. Add validation error messages
3. Export from requests/index.ts

Complexity: low
────────────────────────────────────────────

⏭ Run: /lancelot:prompt a1b2c3d4
```

**When to use:**
- Before starting implementation
- To see what's coming up
- To decide if you want to run parallel instead

---

### `/lancelot:prompt <id>`

**Implement a specific subtask.**

```
/lancelot:prompt a1b2c3d4
```

You can use the full UUID or just the first 8 characters.

**What it does:**
1. Loads the subtask from the plan
2. Reads the target file (if modifying)
3. Finds reference code and patterns
4. Implements all steps for that ONE file
5. **STOPS and waits for review**

**The cycle:**
```
/lancelot:prompt {id}
      │
      ▼
Implement (ONE file only)
      │
      ▼
⛔ STOP
      │
      ▼
"Run /lancelot:review {id} to verify"
```

**Important:**
- Claude will STOP after implementation
- It will NOT auto-continue to the next subtask
- It will NOT mark the subtask complete
- You must run `/lancelot:review` next

---

### `/lancelot:review <id>`

**Review an implementation and mark it complete.**

```
/lancelot:review a1b2c3d4
```

**What it does:**
1. Loads the subtask and its steps
2. Reads the implemented file
3. Launches `code-reviewer` agent (opus model)
4. Checks each step against the code
5. Only reports issues at 80+ confidence
6. Asks for confirmation before marking complete

**Output (approved):**
```
────────────────────────────────────────────
Review: Add request validation

File: src/requests/LoginRequest.ts
ID: a1b2c3d4

Step Analysis:
✅ Step 1: Create LoginRequest class — satisfied
✅ Step 2: Add validation messages — satisfied
✅ Step 3: Export from index.ts — satisfied

Result: APPROVED

All steps satisfied. Mark this subtask as complete?
Reply 'yes' to confirm.
────────────────────────────────────────────
```

**Output (needs work):**
```
────────────────────────────────────────────
Review: Add request validation

Step Analysis:
✅ Step 1: Create LoginRequest class — satisfied
⚠️ Step 2: Add validation messages — Issue (Confidence: 85)
   Missing: Custom message for email format
✅ Step 3: Export from index.ts — satisfied

Result: NEEDS WORK

Fix the issue, then run:
/lancelot:review a1b2c3d4
────────────────────────────────────────────
```

**Important:**
- Only `/lancelot:review` can mark subtasks complete
- Requires explicit 'yes' to confirm
- Claude STOPS after review—won't auto-continue

---

### `/lancelot:parallel [ids]`

**Execute multiple subtasks simultaneously.**

Because each subtask targets one file, they can run in parallel without conflicts.

**Usage:**
```bash
# Auto-select next 3 pending subtasks
/lancelot:parallel

# Run specific subtasks
/lancelot:parallel abc123 def456 ghi789

# Run next 5 pending subtasks
/lancelot:parallel --count 5
```

**What it does:**
1. Validates subtasks can run in parallel (no conflicts)
2. Spawns multiple agents simultaneously
3. Each agent implements one subtask (one file)
4. Reports results when all complete
5. Prompts you to review each

**Output:**
```
────────────────────────────────────────────
⚡ Parallel Execution Complete

Subtasks executed: 3

Results:
✅ abc123 — Create User model
   File: src/models/User.ts
   
✅ def456 — Create Auth service
   File: src/services/AuthService.ts

✅ ghi789 — Add login route
   File: src/routes/auth.ts

────────────────────────────────────────────

Review each:
  /lancelot:review abc123
  /lancelot:review def456
  /lancelot:review ghi789
────────────────────────────────────────────
```

**Dependency detection:**

If subtasks can't run in parallel, Lancelot will warn you:
```
⚠️ Cannot run these subtasks in parallel:

abc123 creates: src/services/UserService.ts
def456 imports from: src/services/UserService.ts

Suggestion: Run abc123 first, then def456
```

**Tips:**
- Maximum recommended: 5 parallel subtasks
- Review each subtask individually after completion
- Great for independent files (models, components, routes)

---

## Typical Workflow

### 1. Initialize (once per project)
```
/lancelot:init
```

### 2. Plan your feature
```
> I want to add user authentication with login, logout, and session management

/lancelot:plan
```

### 3. Check the plan
```
/lancelot:status
```

### 4. Implement

**Option A: Sequential**
```
/lancelot:next
/lancelot:prompt abc123
/lancelot:review abc123
# repeat...
```

**Option B: Parallel**
```
/lancelot:parallel
/lancelot:review abc123
/lancelot:review def456
/lancelot:review ghi789
# repeat...
```

### 5. Continue until done
```
/lancelot:status   # Check progress
/lancelot:next     # What's next?
```

---

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

---

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `code-explorer` | haiku | Fast codebase exploration |
| `conventions-checker` | haiku | Extract coding conventions |
| `plan-architect` | sonnet | Design simple, focused plans |
| `plan-builder` | sonnet | Generate valid Plan JSON |
| `code-reviewer` | opus | High-quality implementation review |

---

## Philosophy

> "Keep it simple. Follow conventions. Ask questions. One file at a time."

Lancelot plans should be:
- **Minimal** — Only what's needed, no over-engineering
- **Atomic** — One file per subtask, one change per step
- **Verifiable** — Every step can be checked
- **Convention-compliant** — Match the codebase patterns
- **Parallelizable** — Independent subtasks can run concurrently

---

## File Structure

```
.lancelot/
├── config.json      # Project expertise config
├── plans/           # Generated plans
│   └── {uuid}.json
└── active-plan      # Current plan ID
```

---

## License

MIT
