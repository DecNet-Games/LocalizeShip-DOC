---
layout: default
title: Localization Providers
nav_order: 7
has_children: true
description: "Integration adapters for Unity Localization, I2 Localization, Pixel Crushers Dialogue System, and custom engines."
permalink: /docs/providers/
---

# Localization Providers
{: .fs-9 }

### Compile-Isolated Adapters for Your Existing Tech Stack
{: .fs-6 .text-grey-dk-000 }

---

LocalizeShip is built on an adapter-based architecture. Rather than forcing you to migrate your strings into a proprietary data format, LocalizeShip binds directly to the localization and narrative systems your team already uses.

![Existing Provider Architecture]({{ site.baseurl }}/assets/images/15-existing-provider.png)

---

## Provider Support Matrix

| Provider | Package Identifier | Integration Status | Compilation Flag |
|---|---|---|---|
| **Unity Localization** | `com.unity.localization` (1.5.13+) | **Core Dependency** (Included & Verified) | Default (`ENABLE_UNITY_LOCALIZATION`) |
| **I2 Localization** | InterIllusion Asset Store package | Optional Compile-Isolated Module | `LOCALIZESHIP_I2_SUPPORT` |
| **Dialogue System for Unity** | Pixel Crushers Asset Store package | Optional Compile-Isolated Module | `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` |
| **Custom In-House Engine** | Any proprietary database / XML / JSON | Supported via C# Adapter Interface | Implement `ILocalizationProviderAdapter` |

---

## The Honesty Fence: Optional Adapters

> [!WARNING] **Important Verification Statement**
> LocalizeShip does not bundle or redistribute proprietary commercial source code from InterIllusion or Pixel Crushers. 
> 
> The adapters for I2 Localization and Pixel Crushers Dialogue System are compile-isolated modules verified in automated acceptance testing for **clean compilation and safe provider-absent fallback** (`LS_T05_ProviderAbsent_CompilesAndDetectsStatusSafely`). We do not claim CI verification against proprietary licensed installations of those third-party assets.

---

## Provider Guides

* [Unity Localization Adapter]({{ site.baseurl }}/docs/providers/unity-localization/) — Native 2-way binding with String Tables and Asset Collections.
* [I2 Localization Adapter]({{ site.baseurl }}/docs/providers/i2-localization/) — Compile symbols and setup for I2 string sources.
* [Pixel Crushers Dialogue System]({{ site.baseurl }}/docs/providers/dialogue-system/) — Tracking dialogue nodes, quest entries, and actor assignments.
* [Authoring Custom Adapters]({{ site.baseurl }}/docs/providers/custom-adapters/) — How to connect custom XML, JSON, or SQL string pipelines.
