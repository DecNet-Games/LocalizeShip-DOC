---
layout: default
title: Dialogue System
parent: Localization Providers
nav_order: 3
description: "The compile-isolated Pixel Crushers Dialogue System adapter: verified compilation and provider-absent behaviour, and what is not implemented."
permalink: /docs/providers/dialogue-system/
---

# Dialogue System Adapter
{: .fs-9 }

### A named boundary for narrative data
{: .fs-6 .text-grey-dk-000 }

`DialogueSystemAdapter` targets Pixel Crushers' Dialogue System for Unity. Like the I2 adapter, it is compile-isolated and verified for safe absence — not for reading or writing dialogue databases.

---

## What is verified

| Claim | Status |
|---|---|
| Compiles without the Dialogue System installed | **Verified** |
| Reports `IsInstalled = false` without the flag | **Verified** |
| `WriteTranslation` throws a clear `NotSupportedException` without the flag | **Verified** |
| Compiles with `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` defined | **Verified** (no Pixel Crushers types referenced) |
| Reads dialogue entries, speakers or conversations | **Not implemented** |
| Tested against a licensed Dialogue System installation | **Not claimed** |

No Pixel Crushers namespace is imported. The symbol only flips the status properties.

---

## Behaviour

### Flag off

```csharp
public bool IsInstalled => false;
public string VersionString => "Not Installed";

GetAvailableCollectionNames()  // empty list
ReadCollection(...)            // empty list
GetTranslation(...)            // null
WriteTranslation(...)          // throws NotSupportedException
```

The exception message directs the user to install Dialogue System for Unity.

### Flag on

```csharp
public bool IsInstalled => true;
public string VersionString => "Detected";
```

`GetAvailableCollectionNames` and `ReadCollection` still return empty lists, `GetTranslation` still returns `null`, and `WriteTranslation` does nothing.

{: .warning }
> Enabling `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` does not connect to any dialogue database. It only changes the reported status.

---

## Why dialogue data is different

Dialogue is not a flat string table. A useful adapter has to decide:

- whether a collection is a `DialogueDatabase`, a conversation, or an actor's localized fields;
- how to form a stable ID from conversation ID + dialogue entry ID (numeric IDs can be renumbered by the editor);
- how to capture speaker, scene and conversation context;
- whether to write back to dialogue text, quest text, or both;
- how to handle markup, sequences and embedded codes.

None of that is implemented in 1.0.0. The adapter is a placeholder for that design work.

---

## Implementing it in your project

1. Create an editor assembly that references both `Decnet.LocalizeShip.Core` and the Dialogue System assembly.
2. Add an asmdef version define so the reference is conditional.
3. Implement `ReadCollection` to walk conversations and emit one `SourceRevision` per dialogue entry.
4. Build a stable key, for example:

```csharp
string stableKey = $"ds.{database.name}.{conversation.id}.{entry.id}.{entry.Title}";
```

Including the title makes renumbering less destructive, but renaming the title still breaks identity. Choose deliberately and document it for your team.

5. Fill `Speaker` and `ContextDescription` from the conversation and actor fields.
6. Implement `WriteTranslation` to update the localized field and mark the database dirty.
7. Register your adapter in your own discovery step.

{: .note }
> Dialogue System's API differs by version and by whether you use the built-in localization or an external table. Verify member names against the version you own.

---

## What not to claim

- That LocalizeShip reads quests, conversations or actor names from Dialogue System.
- That enabling the symbol enables narrative sync.
- That the adapter was validated with a licensed install.
- That speaking-character context is imported automatically.

The accurate sentence is: “A compile-isolated Dialogue System adapter scaffold is included; it compiles and reports provider-absent status safely, and does not yet implement database access.”

---

## Practical alternative

If you need dialogue coverage today, export your dialogue lines to CSV from your narrative tool, import them into the workspace with your own script, and use the [Translation Handoff]({{ site.baseurl }}/docs/features/translation-handoff/) and [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/) flow. That path is implemented and does not depend on an unfinished adapter.

---

## Related pages

- [Localization Providers]({{ site.baseurl }}/docs/providers/)
- [I2 Localization]({{ site.baseurl }}/docs/providers/i2-localization/)
- [Custom Adapters]({{ site.baseurl }}/docs/providers/custom-adapters/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
