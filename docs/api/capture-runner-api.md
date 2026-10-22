---
layout: default
title: Capture Runner API
parent: Architecture & API
nav_order: 3
description: "C# reference for programmatic execution of visual fixtures and screenshot captures."
permalink: /docs/api/capture-runner-api/
---

# Capture Runner API
{: .fs-9 }

### Programmatic Visual Testing & Headless Capture Execution
{: .fs-6 .text-grey-dk-000 }

---

Developers can trigger visual QA runs programmatically from custom editor scripts, unit tests, or CI runners using the `CaptureRunner` API.

---

## 1. `CaptureRunner`

Located in `Decnet.LocalizeShip.Runtime.Capture`:

```csharp
namespace Decnet.LocalizeShip.Runtime.Capture
{
    public static class CaptureRunner
    {
        public static MatrixRunReport ExecuteMatrix(
            ReleaseWorkspace workspace, 
            IEnumerable<CaptureFixture> fixtures, 
            IEnumerable<LocaleId> locales, 
            CaptureSettings settings);

        public static TileRunResult ExecuteTile(
            CaptureFixture fixture, 
            LocaleId locale, 
            CaptureSettings settings);
    }
}
```

---

## 2. Implementing a Custom `CaptureFixture`

```csharp
using UnityEngine;
using Decnet.LocalizeShip.Runtime.Capture;

public class CustomInventoryFixture : CaptureFixture
{
    [SerializeField] private Canvas rootCanvas;

    public override void SetupFixture()
    {
        gameObject.SetActive(true);
    }

    public override void PrepareFixtureForLocale(LocaleId locale)
    {
        // Switch locale dynamically and force canvas mesh generation
        UnityEngine.Localization.Settings.LocalizationSettings.SelectedLocale = 
            UnityEngine.Localization.Settings.LocalizationSettings.AvailableLocales.GetLocale(locale.Code);
            
        Canvas.ForceUpdateCanvases();
    }

    public override void TeardownFixture()
    {
        gameObject.SetActive(false);
    }
}
```
