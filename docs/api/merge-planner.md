---
layout: default
title: MergePlanner
parent: Architecture & API
nav_order: 8
description: "MergePlanner and ApplyCoordinator API reference."
permalink: /docs/api/merge-planner/
---

# `MergePlanner`
{: .fs-9 }

```csharp
MergePlan plan = new MergePlanner().CreatePlan(
    workspace, baseManifest, incomingEntries, new LocaleId("de"));
```

The planner compares base, current and incoming values. Rows are classified in order as duplicate ID, unknown ID, blank incoming, stale source, concurrent locale edit, unchanged or safe incoming change. Entries for other locales are skipped.

`ProtectFormula` prefixes values beginning with `=`, `+`, `-`, `@`, tab or carriage return with an apostrophe. `UnprotectFormula` removes only that recognized protection prefix.

`ApplyCoordinator.ApplyPlan` refuses unresolved conflicts, journals backups, writes accepted text and moves imported changes to `NeedsReview`. `Rollback` detects later edits unless `force` is true.

{: .warning }
> The journal is an in-memory model unless your caller persists it. Applying a plan does not itself call `SetDirty` or save assets.
