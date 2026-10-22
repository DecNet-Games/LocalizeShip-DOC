---
layout: default
title: SourceRevision
parent: Architecture & API
nav_order: 5
description: "SourceRevision and NormalizationPolicy API reference."
permalink: /docs/api/source-revision/
---

# `SourceRevision`
{: .fs-9 }

`SourceRevision` is the revision-bound source record used by approvals, voice, subtitles and impact analysis.

```csharp
var revision = new SourceRevision(
    new StableEntryId("quest.accept"),
    "Accept {count} quests",
    "Quartermaster",
    "Quest confirmation button",
    workspace.Normalization);
```

The constructor stores raw and normalized text, speaker, context, normalization version and UTC ticks. It hashes `Version|NormalizedText|Speaker:ContextDescription` with SHA-256 and stores the first eight bytes as 16 lowercase hexadecimal characters. `ExtractPlaceholders()` collects unique brace-delimited tokens.

> [!WARNING]
> `SceneOrFixture` is not set by the constructor. Set it directly or use `RevisionTracker.ProcessSourceEntry(..., fixture)` when visual impact should follow a fixture.

Hash equality is revision equality; it is not a linguistic judgment. For invalidation behaviour, see [Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/).
