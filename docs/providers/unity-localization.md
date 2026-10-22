---
layout: default
title: Unity Localization
parent: Localization Providers
nav_order: 1
description: "Native 2-way adapter for official Unity Localization package (com.unity.localization 1.5.13+)."
permalink: /docs/providers/unity-localization/
---

# Unity Localization Adapter
{: .fs-9 }

### Deep 2-Way Integration with Unity's Official Localization Stack
{: .fs-6 .text-grey-dk-000 }

---

LocalizeShip provides native, out-of-the-box support for the official **Unity Localization** package (`com.unity.localization`, version 1.5.13 or newer).

There are no additional setup steps or compile symbols required. LocalizeShip binds directly to your `StringTableCollection` and `AssetTableCollection` assets.

---

## How the Adapter Operates

The `UnityLocalizationAdapter` communicates bidirectionally with Unity's table database:

```
┌──────────────────────────────────────┐
│ Unity Localization Assets            │
│ * SharedTableData.asset              │
│ * StringTable (en.asset, de.asset)   │
│ * AssetTable (AudioClips)            │
└──────────────────┬───────────────────┘
                   │
                   ▼ (2-Way Adapter)
┌──────────────────────────────────────┐
│ LocalizeShip Engine                  │
│ * Reads TableEntry.Id as StableEntryId
│ * Reads Metadata (Speaker, Context)  │
│ * Injects Merge Updates & Marks Dirty│
│ * Preserves Addressables Keys        │
└──────────────────────────────────────┘
```

### 1. Stable Key & ID Resolution
Unity Localization assigns each string entry an internal 64-bit integer ID (`TableEntry.Id`) and a shared string key (`SharedTableData`). LocalizeShip maps this internal ID to its `StableEntryId`, ensuring that if you rename a key in Unity's table editor, LocalizeShip preserves all translation approvals and historical audio links.

### 2. Context Metadata Binding
When authoring tables in Unity Localization, you can attach `CommentMetadata` or custom entry metadata. LocalizeShip reads this metadata automatically, using it to populate translator context notes and speaker identity fields.

### 3. Safe 2-Way Writing & Dirty Flagging
When applying merge plans from [Review Imports]({{ site.baseurl }}/docs/features/merge-imports/):
* Updates strings directly in the relevant `StringTable`.
* Calls `EditorUtility.SetDirty(table)` and `AssetDatabase.SaveAssets()` to guarantee disk persistence.
* Leverages Unity's `Undo.RecordObject` to allow immediate in-Editor undos.

---

## Verifying Unity Localization Setup

1. Open **Tools > Decnet > LocalizeShip > Guided Setup**.
2. Step 1 will display **Unity Localization: Detected** with the installed package version.
3. Select your collections to generate the workspace.
