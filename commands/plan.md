---
description: Create a structured implementation plan from conversation context
allowed-tools: Read, Grep, Glob, Task, Write, Bash
---

# Create Implementation Plan

Create a structured implementation plan using the Lancelot agent pipeline.

## Phase 0: Load Project Config

First, check for `.lancelot/config.json`:

```
If exists → Load and use as expertise context
If missing → Suggest running /lancelot/init first, but continue without config
```

Store the config for injection into agent prompts.

## Phase 1: Context Summary

Summarize what needs to be built based on our conversation:
- What feature/fix/change is being requested?
- What are the key requirements?
- Any constraints or preferences mentioned?

## Phase 2: Codebase Exploration (Parallel Agents)

Launch **THREE agents in parallel** using the Task tool:

```
Task 1: code-explorer agent
"Find existing patterns for {feature-type}. Look for:
- Similar implementations
- How this type of feature is structured
- Key files and entry points"

Task 2: code-explorer agent  
"Analyze project structure. Find:
- package.json, tsconfig.json, key configs
- Directory organization
- Dependencies and tooling"

Task 3: conventions-checker agent
"Extract codebase conventions:
- Naming patterns (files, functions, variables)
- Code style (error handling, async patterns)
- File organization rules
- Anti-patterns to avoid"
```

**Wait for all agents to complete**, then synthesize their findings.

## Phase 3: Clarifying Questions ⚠️ REQUIRED

**STOP here. Do not proceed to Phase 4 until this phase is complete.**

### Step 3a: Identify Gaps

Review requirements and exploration findings. List what's unclear:
- Scope uncertainties
- Architectural decisions needing input
- User preferences that matter
- Edge cases worth clarifying

### Step 3b: Ask ONE Question at a Time

**IMPORTANT: Ask only ONE question, then WAIT for the answer.**

Pick the most important gap and ask about it. Keep questions focused:
- "Should this include X, or keep it simple?"
- "I see two approaches - which fits better?"
- "Should I follow pattern X from the codebase?"

After the user answers, evaluate:
- If more clarification needed → Ask the next ONE question
- If requirements are clear → Proceed to Phase 4

**Don't be shy - asking 3-5 good questions leads to a better, simpler plan.**

### Step 3c: Confirm Before Proceeding

When requirements are complete, confirm:

```
Ready to design the plan:
- {key requirement 1}
- {key requirement 2}
- Following conventions: {key conventions}

Proceeding to architecture design...
```

## Phase 4: Architecture Design (Agent)

Launch **plan-architect agent** with expertise context:

```
Task: plan-architect agent
"Design a SIMPLE implementation plan for: {feature description}

## YOUR EXPERTISE (from .lancelot/config.json)

Stack:
{stack from config}

Patterns to follow:
{expertise.patterns from config}

Testing approach:
{expertise.testing from config}

AVOID:
{avoid list from config}

## Requirements
{summarized requirements from Phase 3}

## Codebase Conventions (from exploration)
{conventions from conventions-checker}

## Exploration Findings
{synthesized findings from code-explorers}

Remember:
- Keep it simple, don't over-engineer
- Follow the patterns listed above
- Ask questions if anything is unclear
- Minimum viable plan - remove anything unnecessary"
```

## Phase 5: Generate Plan JSON (Agent)

Launch **plan-builder agent**:

```
Task: plan-builder agent
"Generate valid Plan JSON from this design:

{architectural design from Phase 4}

Read the lancelot/schema skill first.
Return ONLY valid JSON, no markdown."
```

## Phase 6: Validate and Save

1. Validate the JSON:
   - All IDs are unique UUIDs
   - Parent references are correct
   - No empty arrays
   - All required fields present

2. Create `.lancelot/plans/` directory if needed

3. Save to `.lancelot/plans/{id}.json`

4. Write the ID to `.lancelot/active-plan`

## Phase 7: Display Summary

```
────────────────────────────────────────────
Plan Created: {title}
ID: {first 8 chars}

Milestones: {count}
├─ Tasks: {count}
│  └─ Subtasks: {count}
│     └─ Steps: {count}

Stack expertise applied:
- {key stack item}
- {key pattern followed}

⏭ Next: 
  /lancelot/status - See full plan
  /lancelot/prompt {subtask-id} - Start implementing
────────────────────────────────────────────
```

## User Input

$ARGUMENTS

## Agent Pipeline

```
Load .lancelot/config.json
      │
      ▼
Context Summary
      │
      ▼
┌─────────────────────────────────────┐
│ code-explorer ×2 + conventions-checker │ (parallel)
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────┐
│ PHASE 3: CLARIFY│◄──┐
│ Ask ONE question│    │ Loop until
│ Wait for answer │────┘ clear
└─────────────────┘
      │
      ▼
Confirm requirements
      │
      ▼
┌─────────────────┐
│ plan-architect  │ ← Receives expertise from config
└─────────────────┘
      │
      ▼
┌─────────────────┐
│ plan-builder    │
└─────────────────┘
      │
      ▼
Save to .lancelot/plans/
```

## Rules

- Each subtask targets exactly ONE file
- Steps must be atomic (one logical change)
- **Phase 3 is NOT optional** - always ask, always confirm
- **ONE question at a time** - never batch questions
- **Keep it simple** - if in doubt, do less
- **Follow config expertise** - match the project's stack and patterns
