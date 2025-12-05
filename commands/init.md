---
description: Initialize Lancelot in this project by analyzing the codebase and generating config
allowed-tools: Read, Grep, Glob, Write, Bash, Task
---

# Initialize Lancelot

Analyze this codebase and generate a `.lancelot/config.json` with detected stack and conventions.

## Step 1: Check Existing Config

First, check if `.lancelot/config.json` already exists.

If it exists, ask:
```
Lancelot is already initialized in this project.
Do you want to regenerate the config? This will overwrite the existing one.
```

Wait for user response before proceeding.

## Step 2: Analyze Codebase

Launch **stack-analyzer agent** to detect the technology stack:

```
Task: stack-analyzer agent (use code-explorer)
"Analyze this codebase to detect the technology stack.

Check these files:
- package.json (JS/TS dependencies, scripts)
- composer.json (PHP dependencies)
- requirements.txt / pyproject.toml (Python)
- Cargo.toml (Rust)
- go.mod (Go)
- Gemfile (Ruby)
- tsconfig.json (TypeScript config)
- vite.config.* / webpack.config.* (build tools)
- tailwind.config.* (CSS framework)
- .eslintrc* / eslint.config.* (linting)
- phpunit.xml / pest.php / vitest.config.* (testing)

For each technology found, note:
- Name and version
- How it's used (backend, frontend, tooling)
- Any configuration that indicates patterns

Return a structured report of the detected stack."
```

## Step 3: Analyze Conventions

Launch **conventions-checker agent**:

```
Task: conventions-checker agent
"Analyze this codebase for coding conventions and patterns.

Focus on:
- File naming patterns
- Directory structure
- Common design patterns used
- Import/export styles
- Error handling approaches
- Testing patterns

Return specific examples from the code."
```

## Step 4: Generate Config

Based on the analysis, generate a `.lancelot/config.json`:

```json
{
  "version": "1.0",
  "stack": {
    // Detected technologies
  },
  "expertise": {
    "patterns": [
      // Detected patterns from code
    ],
    "testing": {
      // Detected test frameworks
    }
  },
  "conventions": {
    // Detected conventions
  },
  "avoid": [
    // Inferred anti-patterns (opposite of detected patterns)
  ],
  "preferences": {
    "complexity": "simple"
  }
}
```

## Step 5: Create Directory Structure

```bash
mkdir -p .lancelot/plans
```

## Step 6: Save and Display

1. Save config to `.lancelot/config.json`
2. Display the generated config to the user
3. Ask for feedback:

```
────────────────────────────────────────────
Lancelot initialized!

Created:
  .lancelot/
  ├── config.json    ← Project configuration
  └── plans/         ← Plans will be saved here

Detected stack:
  - {backend}
  - {frontend}
  - {testing}

Review .lancelot/config.json and adjust as needed.
Key sections to verify:
  - expertise.patterns (are these the patterns you use?)
  - avoid (anything to add/remove?)

⏭ Next: Run /lancelot/plan to create your first plan
────────────────────────────────────────────
```

## Step 7: Offer Refinement

Ask ONE question at a time to refine:

```
Would you like to customize anything in the config?

1. Add specific patterns you follow
2. Add things to avoid
3. Adjust stack details
4. Keep as-is

Reply with a number or describe what to change.
```

If they want changes, update the config and save again.

## User Input

$ARGUMENTS
