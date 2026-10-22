---
layout: default
title: HandoffManifest
parent: Architecture & API
nav_order: 7
description: "HandoffManifest, HandoffEntry and HandoffConstraint schema reference."
permalink: /docs/api/handoff-manifest/
---

# `HandoffManifest`
{: .fs-9 }

`HandoffManifest` is the revision-aware exchange envelope used for translator handoff and merge planning.

```csharp
public string SchemaVersion = "1.0.0";
public string ReleaseId;
public string WorkspaceId;
public string SourceLocale;
public List<string> TargetLocales;
public long ExportTimestampTicks;
public List<HandoffEntry> Entries;
```

Each `HandoffEntry` carries the stable ID, base source and translation hashes, source and current translation text, target locale, speaker, context, relative screenshot path and constraints. Constraints hold character and line limits, context notes and required placeholders.

The default constructor sets `ExportTimestampTicks` with UTC ticks. It does not populate release, workspace or locale fields. Populate those before serialization.

JSON preserves the complete model. CSV is RFC 4180-oriented but intentionally loses structured constraints and base translation text; prefer JSON when round-trip fidelity matters.
