---
layout: default
title: Domain Services
parent: Architecture & API
nav_order: 2
description: "C# reference for RevisionTracker, ImpactEngine, MergePlanner, and TextRuleEngine."
permalink: /docs/api/domain-services/
---

# Domain Services
{: .fs-9 }

### Business Logic & Evaluation Engines
{: .fs-6 .text-grey-dk-000 }

---

LocalizeShip's business logic is encapsulated in stateless, high-performance domain services located in `Decnet.LocalizeShip.Core.Services`.

---

## 1. `RevisionTracker`

Responsible for indexing string tables, normalizing text, and detecting source mutations:

```csharp
namespace Decnet.LocalizeShip.Core.Services
{
    public sealed class RevisionTracker
    {
        public static string NormalizeText(string rawText, WhitespacePolicy policy);
        public static SourceRevisionHash ComputeHash(string normalizedText, string context, string speaker);
        public static bool HasChanged(SourceRevision previous, string newText);
    }
}
```

---

## 2. `ImpactEngine`

Calculates the blast radius of source edits across translations, audio takes, and UI captures:

```csharp
public sealed class ImpactEngine
{
    public ImpactSummary CalculateImpact(
        ReleaseWorkspace workspace, 
        StableEntryId entryId, 
        SourceRevision newRevision);
}
```

---

## 3. `MergePlanner`

Executes the 3-way compare algorithm, classifies merge rows, and generates atomic apply transactions:

```csharp
public sealed class MergePlanner
{
    public MergePlan CreatePlan(
        HandoffManifest baseManifest, 
        ReleaseWorkspace currentWorkspace, 
        ImportDataset incomingData);

    public bool ApplyPlan(
        MergePlan plan, 
        out ImportJournal journal, 
        out string error);

    public bool RollbackJournal(
        ImportJournal journal, 
        out string error);
}
```

---

## 4. `TextRuleEngine`

Parses placeholders, Unity Smart Strings, and TextMeshPro rich text formatting:

```csharp
public sealed class TextRuleEngine
{
    public static ValidationResult ValidateFormat(
        string sourceText, 
        string targetText, 
        LocaleId targetLocale);
}
```
