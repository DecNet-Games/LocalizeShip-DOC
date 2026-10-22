---
layout: default
title: Dialogue System (Pixel Crushers)
parent: Localization Providers
nav_order: 3
description: "Compile-isolated adapter for Pixel Crushers Dialogue System for Unity."
permalink: /docs/providers/dialogue-system/
---

# Dialogue System Adapter (Pixel Crushers)
{: .fs-9 }

### Auditable Release Gates for Branching Narrative & Quest Trees
{: .fs-6 .text-grey-dk-000 }

---

**Dialogue System for Unity** by Pixel Crushers is one of the most widely used narrative engines in game development. LocalizeShip provides a dedicated compile-isolated adapter (`DialogueSystemAdapter.cs`) that tracks conversation nodes, quest entries, and actor assignments.

---

## Enabling the Dialogue System Adapter

To avoid compiler errors in projects where Dialogue System is not present, the adapter is protected by a scripting define symbol:

1. Import your licensed copy of **Dialogue System for Unity**.
2. Open **Edit > Project Settings > Player**.
3. Under **Other Settings > Script Compilation**, add the scripting define:
   ```text
   LOCALIZESHIP_PIXELCRUSHERS_SUPPORT
   ```
4. Click **Apply**.

---

## What the Adapter Tracks

When active, LocalizeShip maps dialogue databases into its release tracking graph:

* **Dialogue Entries**: Each conversation node (dialogue entry) is tracked by its unique conversation ID and entry ID.
* **Actor Bindings**: The speaking actor is extracted as `SpeakerId`, providing vital context to voice engineers and translators.
* **Quest Descriptions & Entries**: Quest states (Active, Success, Failure) are mapped into individual reviewable lines.
* **Custom Fields**: Localized fields on dialogue entries (e.g., `Dialogue Text {de}`, `Menu Text {ja}`) are mapped directly to target locales.

---

## Verification & Absence Safety

> [!WARNING] **Honesty Fence**
> LocalizeShip does not bundle or redistribute Pixel Crushers Dialogue System. Acceptance test `LS_T05_ProviderAbsent_CompilesAndDetectsStatusSafely` verifies that the adapter compiles cleanly and falls back safely when the symbol is absent. We do not claim CI verification against licensed proprietary installations.
