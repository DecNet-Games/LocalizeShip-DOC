---
layout: default
title: Identifiers & Data Models
parent: Architecture & API
nav_order: 1
description: "C# reference for strongly-typed identifiers and core ScriptableObject data structures."
permalink: /docs/api/identifiers-and-models/
---

# Identifiers & Data Models
{: .fs-9 }

### Type-Safe Value Objects and Domain Entities
{: .fs-6 .text-grey-dk-000 }

---

All core domain operations in LocalizeShip rely on strongly-typed value objects (`Identifiers.cs`) to eliminate string typing bugs and ensure compiler safety.

---

## 1. Strongly-Typed Value Objects

### `StableEntryId`
An immutable, unique identifier for a localized line that persists across table key renames, file moves, and re-orderings:

```csharp
namespace Decnet.LocalizeShip.Core.Models
{
    [System.Serializable]
    public readonly struct StableEntryId : System.IEquatable<StableEntryId>
    {
        public string Value { get; }
        public StableEntryId(string value);
        public bool Equals(StableEntryId other);
    }
}
```

### `SourceRevisionHash`
A 64-character hex string representing the SHA-256 hash of the normalized source text, context metadata, and placeholders:

```csharp
[System.Serializable]
public readonly struct SourceRevisionHash : System.IEquatable<SourceRevisionHash>
{
    public string Value { get; }
    public SourceRevisionHash(string value);
    public bool Equals(SourceRevisionHash other);
}
```

### `LocaleId`
A standardized locale code conforming to IETF BCP 47 (e.g., `en-US`, `de-DE`, `ja-JP`, `zh-Hans`):

```csharp
[System.Serializable]
public readonly struct LocaleId : System.IEquatable<LocaleId>
{
    public string Code { get; }
    public LocaleId(string code);
    public bool Equals(LocaleId other);
}
```

---

## 2. Core Domain Entities

### `SourceRevision`
Represents a specific snapshot in a source line's mutation history:

```csharp
public sealed class SourceRevision
{
    public StableEntryId EntryId { get; }
    public SourceRevisionHash RevisionHash { get; }
    public SourceRevisionHash ParentHash { get; }
    public string NormalizedText { get; }
    public string ContextNote { get; }
    public string SpeakerId { get; }
    public System.DateTime TimestampUtc { get; }
    public string[] Placeholders { get; }
}
```

### `ApprovalEvent`
An immutable audit record representing a verified human sign-off:

```csharp
public sealed class ApprovalEvent
{
    public StableEntryId EntryId { get; }
    public LocaleId Locale { get; }
    public SourceRevisionHash SourceHash { get; }
    public TranslationRevisionHash TranslationHash { get; }
    public string ReviewerId { get; }
    public System.DateTime TimestampUtc { get; }
    public string Rationale { get; }
}
```

### `ReleaseWorkspace`
The primary `ScriptableObject` storing release configurations, registered collections, policies, and the approval ledger:

```csharp
[CreateAssetMenu(fileName = "ReleaseWorkspace", menuName = "Decnet/LocalizeShip/Release Workspace")]
public sealed class ReleaseWorkspace : ScriptableObject
{
    public string ReleaseId;
    public LocaleId SourceLocale;
    public List<LocaleId> TargetLocales;
    public List<CollectionDescriptor> BoundCollections;
    public ReleasePolicy Policy;
}
```
