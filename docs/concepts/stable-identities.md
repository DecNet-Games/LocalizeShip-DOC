---
layout: default
title: Stable Identities
parent: Concepts
nav_order: 1
description: "How LocalizeShip addresses lines with StableEntryId, how that maps to provider keys, and what happens on rename or move."
permalink: /docs/concepts/stable-identities/
---

# Stable Identities
{: .fs-9 }

### Why a row number is a terrible identity
{: .fs-6 .text-grey-dk-000 }

Every localization bug starts the same way: someone treats the position of a row as its identity. Insert a line above it in a spreadsheet, or re-sort the table, and every review comment, voice take and screenshot association shifts by one. LocalizeShip avoids this by addressing lines with an explicit string.

---

## `StableEntryId`

`StableEntryId` is a small struct around a string:

```csharp
[Serializable]
public struct StableEntryId : IEquatable<StableEntryId>
{
    public string Value;
    public bool IsValid => !string.IsNullOrEmpty(Value);
    public bool Equals(StableEntryId other)
        => string.Equals(Value, other.Value, StringComparison.Ordinal);
}
```

Properties that matter:

- **Ordinal comparison.** `quest_intro` and `Quest_Intro` are different IDs. Do not rely on case-insensitive lookups.
- **Implicit string conversion.** You can pass a string where an ID is expected, which keeps provider adapters readable.
- **Serializable.** It survives Unity serialization inside workspace assets.
- **No registry.** The ID is whatever the provider's key is. LocalizeShip does not maintain a separate global dictionary in 1.0.0.

---

## How the Unity adapter assigns IDs

For Unity Localization, the stable ID is the **shared table entry key**, not the numeric entry ID:

```csharp
foreach (var entry in sharedData.Entries)
{
    string key = entry.Key;
    long keyId = entry.Id;

    var tableEntry = sourceTable.GetEntry(keyId);
    string text = tableEntry != null ? tableEntry.Value : string.Empty;

    var stableId = new StableEntryId(key);
    var rev = new SourceRevision(stableId, text, string.Empty, string.Empty, policy);
    results.Add(rev);
}
```

Implications:

- Two collections can contain the same key. In the current adapter there is no collection name in the ID, so `menus/play` and `dialogue/play` would collide. Use unique keys across the project until a collection-qualified ID is added.
- The numeric `entry.Id` is used only to fetch the value; it is not the identity.
- Source reads set speaker and context to empty strings. Unity Localization does not expose a speaker/context concept, so the hash covers version, text and an empty context. Provider-specific adapters can supply richer context.

---

## Reorder, move, rename

| Operation | Result in 1.0.0 |
|---|---|
| **Reorder rows** | Identity unchanged. Approvals, voice and findings still match. |
| **Move an entry between tables** | Identity unchanged **if the key is unchanged**, but the current adapter cannot distinguish collections with identical keys. |
| **Rename a key** | Identity changes. The old approval record and voice bindings no longer match. There is no automatic rename map in 1.0.0. |
| **Delete and recreate with the same key** | Identity is the same. History for that key remains, which may be misleading if the line is conceptually new. |
| **Duplicate the same key** | The adapter reads both; the merge planner treats duplicate incoming IDs as `DuplicateIdError`. Avoid duplicates at the source. |

{: .warning }
> **No rename map is shipped.** The product concept calls for one; the 1.0.0 code does not implement an alias table. If your writers rename keys often, establish a “key is permanent” rule, or keep a manual mapping and re-review after rename. This is tracked in [Limitations]({{ site.baseurl }}/docs/limitations/).

---

## Locale identity

`LocaleId` is a second small struct. It trims and lowercases its input:

```csharp
public LocaleId(string code)
{
    Code = code != null ? code.Trim().ToLowerInvariant() : string.Empty;
}
```

So `de-DE`, `DE-de` and `DE` are all normalized to lowercase, but `de-DE` and `de` are **different** locales. Be consistent. The Unity adapter constructs `new LocaleIdentifier(locale.Code)`, so use the identifier form your tables actually contain.

---

## Practical rules for writers and integrators

1. **Treat keys as permanent API.** Rename the display name, not the key.
2. **Namespace keys by screen or system** (`ui.settings.audio.title`) to avoid collisions.
3. **Do not reuse a key for a different meaning.** History will follow it.
4. **Keep case consistent.** Ordinal comparison is deliberate.
5. **Do not put dynamic values in keys.** Use placeholders in the text instead.

---

## Where identity is used

| Consumer | Uses the ID to |
|---|---|
| `ReleaseWorkspace.GetSourceRevision` | Find the current source revision |
| `ReleaseWorkspace.GetApproval` | Find the line × locale approval record |
| `ImpactEngine` | Collect voice, subtitle and capture artifacts for the line |
| `MergePlanner` | Match incoming entries against current and base |
| `SourceNavigator` | Select the scene object or shared table data |
| `VisualFinding` fingerprint | Keep suppressions scoped to a line |

Next: [Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/).
