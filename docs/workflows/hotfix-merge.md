---
layout: default
title: Hotfix & Day-1 Patches
parent: Production Workflows
nav_order: 5
description: "Safely patching typos and critical text defects in live retail builds without regressions."
permalink: /docs/workflows/hotfix-merge/
---

# Hotfix & Day-1 Patches
{: .fs-9 }

### Fast, Surgical Text Corrections for Live Game Builds
{: .fs-6 .text-grey-dk-000 }

---

Nothing is more stressful than shipping a Day-1 hotfix for a live game on Steam or consoles. You need to fix a game-breaking quest typo in 3 languages, but touching string tables manually risks corrupting IDs or introducing unintended layout bugs.

LocalizeShip provides a **Surgical Hotfix Workflow**.

---

## The 4-Step Hotfix Process

```
┌─────────────────────────────────────────────────────────────┐
│ 1. ISOLATE DEFECTIVE STRINGS                                │
│ Use the Release Board search to find the offending keys.   │
│ Tag them with a "Hotfix-v1.0.1" release label.              │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. MICRO-HANDOFF EXPORT                                     │
│ In Translation Handoff, filter by release label.            │
│ Export an RFC 4180 CSV containing only those 3 lines.       │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. TARGETED CAPTURE MATRIX RUN                              │
│ In Capture Matrix, run only the fixtures associated with     │
│ the hotfixed lines to verify no unexpected overflows occur. │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. APPLY WITH JOURNAL BACKUP & CI VERIFICATION              │
│ Review and apply incoming corrections. The ImportJournal     │
│ preserves pre-patch state for instant rollback if needed.   │
└─────────────────────────────────────────────────────────────┘
```

---

## Pro Tip for Live-Ops Teams

> [!TIP] **Zero Risk Rollback**
> If a hotfix introduces an unexpected issue in one language after deployment, open **Review Imports** and click **Revert Import Journal**. LocalizeShip will immediately restore the exact pre-hotfix table state across all modified assets.
