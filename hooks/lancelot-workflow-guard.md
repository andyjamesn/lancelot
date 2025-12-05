---
name: lancelot-workflow-guard
description: Prevents unauthorized modifications to plan JSON files outside of /lancelot/review
event: file
pattern: "\.lancelot/plans/.*\.json$"
action: warn
---

# Lancelot Workflow Guard

This hook monitors writes to plan JSON files.

## When Triggered

Any write to `.lancelot/plans/*.json` files.

## Warning Message

```
⚠️ LANCELOT WORKFLOW WARNING

You are modifying a plan JSON file.

Only /lancelot/review should modify plan status.
If you're marking something complete outside of review, STOP.

The workflow is:
  /lancelot/prompt → implement → STOP
  /lancelot/review → approve → mark complete → STOP

Continue only if this is:
- /lancelot/plan creating a new plan
- /lancelot/review marking completion after approval
```

## Purpose

Reminds Claude (and users) that plan modifications should follow the Lancelot workflow. This is a warning (not a block) to allow legitimate modifications while discouraging workflow violations.
