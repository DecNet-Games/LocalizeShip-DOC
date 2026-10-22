---
layout: default
title: I2 Localization
parent: Localization Providers
nav_order: 2
description: "Compile-isolated adapter for InterIllusion's I2 Localization system."
permalink: /docs/providers/i2-localization/
---

# I2 Localization Adapter
{: .fs-9 }

### Compile-Isolated Adapter for Legacy & Existing I2 Projects
{: .fs-6 .text-grey-dk-000 }

---

Many mature Unity projects use **I2 Localization** by InterIllusion. LocalizeShip includes a specialized, compile-isolated adapter (`I2LocalizationAdapter.cs`) that allows teams to use LocalizeShip's visual QA, revision-bound approvals, and release gates without migrating away from I2.

---

## Enabling the I2 Adapter

To prevent compilation errors in projects that do not own I2 Localization, the adapter is compile-isolated behind a preprocessor define:

1. Import your licensed copy of **I2 Localization** into your project.
2. In the Unity Editor, open **Edit > Project Settings > Player**.
3. Under **Other Settings > Script Compilation**, locate **Scripting Define Symbols**.
4. Add the symbol:
   ```text
   LOCALIZESHIP_I2_SUPPORT
   ```
5. Click **Apply**.

Upon recompilation, LocalizeShip's `ProviderDiscoveryService` will detect your active I2 `LanguageSourceAsset` and expose I2 terms in the Release Board.

---

## Feature Mapping

| I2 Concept | LocalizeShip Mapping | Notes |
|---|---|---|
| **Term** (e.g., `Quests/Objective1`) | `StableEntryId` & Key | Hierarchical term slashes preserved. |
| **Language Index** | `LocaleId` | Mapped to BCP 47 language codes. |
| **Term Description** | `ContextNote` | Exported in handoff packages. |
| **Attached Audio** | `VoiceEntry` | Audited against source text revisions. |

---

## Verification & Absence Safety

> [!WARNING] **Honesty Fence**
> LocalizeShip does not redistribute I2 Localization. In our automated test suite (`LS_T05_ProviderAbsent_CompilesAndDetectsStatusSafely`), the I2 adapter is verified for **clean compilation and safe provider-absent fallback** when `LOCALIZESHIP_I2_SUPPORT` is not set. We do not claim CI verification against proprietary licensed installations.
