---
layout: default
title: ImpactEngine
parent: Architecture & API
nav_order: 6
description: "ImpactEngine API reference and cascade rules."
permalink: /docs/api/impact-engine/
---

# `ImpactEngine`
{: .fs-9 }

```csharp
ImpactReport report = new ImpactEngine().ComputeImpact(workspace, entryId);
```

The service reads one stable entry and emits `ChangedSource`, `StaleTranslation`, `VoiceReRecord`, `SubtitleReview` and `VisualRecheck` items. Translation items are emitted for missing, stale or absent approvals. Voice items require a stale or missing take. Subtitle items compare the cue's bound source hash. Visual items match a finding entry ID or a fixture ID equal to `SceneOrFixture` without case sensitivity.

`ComputeImpact` throws `ArgumentNullException` for a null workspace. It returns a report even when the entry has no current source revision.

> [!TIP]
> Recompute after `RevisionTracker` processes a source edit. The engine reports current workspace state; it does not mutate or cache it.

The engine does not evaluate voice duration, provider assets or review states other than missing and stale.
