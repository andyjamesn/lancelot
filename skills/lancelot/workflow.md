# Lancelot Workflow Rules

**⛔ CRITICAL: Read this before ANY Lancelot operation.**

## The Golden Rule

```
STOP after every operation. User controls the flow.
```

## The Implementation Cycle

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   /lancelot/prompt {id}                                     │
│           │                                                 │
│           ▼                                                 │
│   ┌───────────────┐                                        │
│   │  IMPLEMENT    │  Write code for the subtask            │
│   └───────────────┘                                        │
│           │                                                 │
│           ▼                                                 │
│   ┌───────────────┐                                        │
│   │  ⛔ STOP      │  Wait for user                         │
│   └───────────────┘                                        │
│           │                                                 │
│           │  User runs /lancelot/review {id}               │
│           ▼                                                 │
│   ┌───────────────┐                                        │
│   │  REVIEW       │  Analyze implementation                │
│   └───────────────┘                                        │
│           │                                                 │
│           ├── NEEDS WORK ──→ User fixes ──→ /review again  │
│           │                                                 │
│           ▼  APPROVED                                       │
│   ┌───────────────┐                                        │
│   │  ASK USER     │  "Mark complete? (yes)"                │
│   └───────────────┘                                        │
│           │                                                 │
│           │  User confirms                                  │
│           ▼                                                 │
│   ┌───────────────┐                                        │
│   │  UPDATE JSON  │  Mark subtask complete                 │
│   └───────────────┘                                        │
│           │                                                 │
│           ▼                                                 │
│   ┌───────────────┐                                        │
│   │  ⛔ STOP      │  Wait for user                         │
│   └───────────────┘                                        │
│           │                                                 │
│           │  User runs /lancelot/next or /prompt {next}    │
│           ▼                                                 │
│        (repeat)                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## STOP Points (Mandatory)

| After | You Must |
|-------|----------|
| `/prompt` implementation | STOP. Tell user to run `/review` |
| `/review` report | STOP. Wait for user response |
| Marking complete | STOP. Tell user to run `/next` |
| `/status` display | STOP. Wait for user decision |
| `/next` display | STOP. Wait for user to run `/prompt` |

## Permission Matrix

| Action | /prompt | /review | /status | /next | /plan |
|--------|---------|---------|---------|-------|-------|
| Read plan JSON | ✅ | ✅ | ✅ | ✅ | ✅ |
| Write target file | ✅ | ❌ | ❌ | ❌ | ❌ |
| Mark step complete | ❌ | ✅* | ❌ | ❌ | ❌ |
| Mark subtask complete | ❌ | ✅* | ❌ | ❌ | ❌ |
| Modify plan JSON | ❌ | ✅* | ❌ | ❌ | ✅ |
| Create new plan | ❌ | ❌ | ❌ | ❌ | ✅ |
| Auto-continue | ❌ | ❌ | ❌ | ❌ | ❌ |

*Only with explicit user confirmation

## Forbidden Sequences

These sequences must NEVER happen:

```
❌ /prompt → implement → mark complete
   (Only /review can mark complete)

❌ /prompt → implement → /review (auto)
   (User must invoke /review)

❌ /review → approve → /prompt {next} (auto)
   (User must request next subtask)

❌ /prompt → implement → implement next
   (One subtask at a time, with review)
```

## Required Sequences

These are the ONLY valid sequences:

```
✅ /prompt → implement → STOP → (user) → /review

✅ /review → APPROVED → ask user → (user says yes) → update JSON → STOP

✅ /review → NEEDS WORK → STOP → (user fixes) → /review

✅ /next → show subtask → STOP → (user) → /prompt
```

## Anti-Patterns

**WRONG - Auto-continuing:**
```
"Done implementing! Now let me review it..."
"Marked complete! Starting the next subtask..."
"All steps look good, marking as complete and moving on..."
```

**WRONG - Skipping confirmation:**
```
"All steps satisfied, marked complete."
(Must ask user first!)
```

**WRONG - Modifying plan in /prompt:**
```
"Updated the plan to mark step 1 as complete..."
(Only /review modifies the plan)
```

**RIGHT - Proper /prompt ending:**
```
✓ Implementation complete: Add user validation
File: src/validators/user.ts
Steps completed: 3

⏭ Next: Run /lancelot/review a1b2c3d4
```
[STOP]

**RIGHT - Proper /review ending:**
```
✅ All steps satisfied

Mark this subtask as complete?
Reply 'yes' to confirm.
```
[STOP - wait for "yes"]

## Why These Rules Exist

1. **Quality Gates** - Review catches issues before marking done
2. **User Control** - User decides pace and can pause anytime
3. **Audit Trail** - Clear separation of implement vs verify
4. **Error Recovery** - Easy to fix before committing completion
5. **Predictability** - Same flow every time, no surprises

## Quick Reference

```
/plan    → Creates plan → STOP
/status  → Shows progress → STOP
/next    → Shows next subtask → STOP
/prompt  → Implements → STOP (wait for /review)
/review  → Reviews → Ask → STOP (wait for /next)
```
