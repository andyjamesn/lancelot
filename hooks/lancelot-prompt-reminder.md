---
name: lancelot-prompt-reminder
description: Reminds to run review after implementing a subtask
event: stop
action: warn
---

# Lancelot Prompt Reminder

This hook triggers when Claude attempts to stop/complete.

## Warning Message

```
⏸️ BEFORE YOU FINISH

If you just implemented a Lancelot subtask:
→ Did you tell the user to run /lancelot/review?
→ Did you STOP and wait (not auto-continue)?

If you're mid-workflow, remind the user of the next step.
```

## Purpose

Catches cases where Claude might try to end the conversation or move on without properly completing the Lancelot workflow cycle.
