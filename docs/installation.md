---
layout: default
title: Installation
nav_order: 3
description: "Install LocalizeShip from the Unity Asset Store, add the Unity Localization dependency, and enable optional provider adapters."
permalink: /docs/installation/
---

# Installation
{: .fs-9 }

### Asset Store import, dependencies, and first compile
{: .fs-6 .text-grey-dk-000 }

LocalizeShip is a **commercial Unity Asset Store package**. It is not distributed for free on GitHub, and there is no public UPM registry endpoint for it. The GitHub repository you may have found is documentation and issue tracking only.

![LocalizeShip runs on top of an existing provider]({{ site.baseurl }}/assets/images/15-existing-provider.png)

---

## Requirements

| Requirement | Version | Why it matters |
|---|---|---|
| **Unity Editor** | **6000.5.4f1** — the verified baseline | All development and the 12 acceptance suites were run here |
| **Unity Localization** | **1.5.13** (`com.unity.localization`) | The official provider adapter uses its editor API |
| **Addressables** | **1.25.0** (`com.unity.addressables`) | Installed automatically as a Localization dependency |
| **TextMeshPro / uGUI** | Included with Unity 6 | Layout measurement and glyph checks |
| **I2 Localization** | Optional, your licence | Compile-isolated adapter only |
| **Pixel Crushers Dialogue System** | Optional, your licence | Compile-isolated adapter only |

{: .warning }
> Only Unity **6000.5.4f1** is claimed as verified. Other Unity 6 patch versions will usually work, but LocalizeShip does not claim a test matrix it has not run. Do not promise “2022.3 LTS supported” on the basis of this documentation.

---

## Step 1 — Install Unity Localization first

Install the dependency before LocalizeShip, so the adapter compiles on first import.

1. Open **Window → Package Manager**.
2. Switch the package source to **Unity Registry**.
3. Search for **Localization**.
4. Select **Localization** and install version **1.5.13** (or verify that version is already pinned).
5. Let Unity resolve **Addressables 1.25.0**.
6. Wait for the project to finish compiling and for the Localization Tables window to become available.

You can confirm the version in `Packages/manifest.json`:

```json
{
  "dependencies": {
    "com.unity.localization": "1.5.13",
    "com.unity.addressables": "1.25.0"
  }
}
```

---

## Step 2 — Get LocalizeShip from the Asset Store

1. Sign in to the Unity account that purchased LocalizeShip.
2. Open the **Unity Asset Store** listing and add the package to **My Assets** if you have not already.
3. In Unity, open **Window → Package Manager**.
4. Change the source dropdown to **My Assets**.
5. Search for **LocalizeShip**.
6. Click **Download**, then **Import**.
7. In the import window, keep all files selected and click **Import**.

{: .note }
> Depending on the Asset Store delivery, the package may arrive as a `.unitypackage` through **Assets → Import Package → Custom Package**. Both paths produce the same project layout. There is no `.unitypackage` download on GitHub.

### What gets added

```
Assets/Decnet/LocalizeShip/
├── Adapters/        ILocalizationProvider + Unity, I2, Dialogue System adapters
├── Core/
│   ├── Models/      workspace, revisions, approvals, findings, merge plans
│   └── Services/    revision tracker, impact engine, merge planner, gates, rules
├── Editor/
│   ├── Menus/       Tools > Decnet > LocalizeShip
│   ├── Services/    provider discovery, source navigation, diagnostics
│   └── Windows/     the studio windows
├── Runtime/Capture/ CaptureFixture + CaptureRunner
├── Samples/RPGSample/
├── Tests/Editor/    12 EditMode acceptance suites
└── Documentation/   offline markdown manual
```

Assembly definitions isolate the layers:

| Assembly | Purpose |
|---|---|
| `Decnet.LocalizeShip.Core` | Models and domain services; no editor UI |
| `Decnet.LocalizeShip.Adapters` | Provider interface and adapter implementations |
| `Decnet.LocalizeShip.Runtime` | `CaptureFixture` and `CaptureRunner` |
| `Decnet.LocalizeShip.Editor` | Editor windows, menus and editor-only services |
| `Decnet.LocalizeShip.Samples` | RPG sample UI and scene setup |
| `Decnet.LocalizeShip.Tests.Editor` | EditMode acceptance tests |

If your own asmdef needs to attach capture fixtures, add a reference to `Decnet.LocalizeShip.Runtime`. Do not reference the Editor assembly from runtime code.

---

## Step 3 — Verify the install

1. Wait for compilation to finish with no errors.
2. Confirm **Tools → Decnet → LocalizeShip** exists.
3. Open **Tools → Decnet → LocalizeShip → Guided Setup**.
4. The provider list should show:
   - **Unity Localization (Official)** — installed, version string `1.5.13`.
   - **I2 Localization (Optional Adapter)** — not installed unless the define and package are present.
   - **Pixel Crushers Dialogue System (Optional Adapter)** — not installed unless the define and package are present.
5. Close the wizard without creating anything if you only wanted to verify compilation.

{: .tip }
> The Unity adapter reports `IsInstalled = true` and a fixed version string `1.5.13`. That is a build-time baseline, not a live package query. Use the Package Manager to confirm the actual resolved version.

---

## Optional adapter setup

Both optional adapters are **compile-isolated**. Without their define, the adapter still compiles and reports `Not Installed`; write operations throw `NotSupportedException` instead of failing silently.

### I2 Localization

1. Install I2 Localization in the project and let it compile.
2. Open **Project Settings → Player → Other Settings → Scripting Define Symbols**.
3. Add `LOCALIZESHIP_I2_SUPPORT`.
4. Apply and wait for recompilation.

### Pixel Crushers Dialogue System

1. Install Dialogue System for Unity and let it compile.
2. Add `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` to the same define list.
3. Apply and wait for recompilation.

{: .important }
> The I2 and Dialogue System adapters are verified for **clean compilation and provider-absent behaviour**. They have **not** been verified against licensed installations, and their collection read/write method bodies are placeholders behind the define. Treat them as integration scaffolding, not as a certified one-click integration. See [I2 Localization]({{ site.baseurl }}/docs/providers/i2-localization/) and [Dialogue System]({{ site.baseurl }}/docs/providers/dialogue-system/).

---

## Version control setup

Commit the workspace asset and any generated handoff files you want to keep as release evidence. Do not commit `Library/`, `Temp/` or `Logs/`.

A minimal `.gitignore` addition:

```gitignore
# Unity generated
/[Ll]ibrary/
/[Tt]emp/
/[Oo]bj/
/[Bb]uild/
/[Ll]ogs/
/[Uu]ser[Ss]ettings/

# LocalizeShip generated captures and handoffs (optional)
Assets/Decnet/LocalizeShip/Generated/Captures/
Assets/Decnet/LocalizeShip/HandoffExport/
```

If you want audit evidence in the repository, keep `LocalizeShip_Report_v*.json` and `LocalizeShip_Report_v*.html`. They are plain files written next to the project folder when you export from the Release Gate window.

---

## Uninstalling

1. Close all LocalizeShip windows.
2. Delete `Assets/Decnet/LocalizeShip`.
3. Remove the optional define symbols if you added them.
4. Keep your generated workspace and report files if they are part of release evidence; they are plain assets/files and do not depend on the package being present to be read.

---

## Installation checklist

- [ ] Unity 6000.5.4f1 project opens cleanly.
- [ ] Unity Localization 1.5.13 installed and compiling.
- [ ] LocalizeShip imported from the Asset Store.
- [ ] `Tools → Decnet → LocalizeShip` menu appears.
- [ ] Guided Setup shows Unity Localization as installed.
- [ ] Optional defines added only if the corresponding package is installed.
- [ ] Workspace asset path chosen and added to version control.

Next: [Quick Start]({{ site.baseurl }}/docs/quick-start/).
