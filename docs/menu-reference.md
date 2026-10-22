---
layout: default
title: Menu Reference
nav_order: 12
description: "Every editor menu item and window in LocalizeShip 1.0.0, with exact titles and where each tool actually lives."
permalink: /docs/menu-reference/
---

# Menu Reference
{: .fs-9 }

### Every entry, without invented shortcuts
{: .fs-6 .text-grey-dk-000 }

All tools are under **Tools → Decnet → LocalizeShip**. There are no keyboard shortcuts registered by the package.

---

## Menu items

| Menu path | Priority | Opens | Window title |
|---|---|---|---|
| **Release Board** | 1 | `ReleaseBoardWindow.Open()` | `LocalizeShip Studio` |
| **Guided Setup** | 2 | `GuidedSetupWindow.Open()` | `LocalizeShip Guided Setup` |
| **Open Sample Release** | 3 | Loads sample scene + workspace | `LocalizeShip Studio` |
| **Translation Handoff** | 20 | `TranslationHandoffWindow.Open()` | `Translation Handoff` |
| **Capture Matrix** | 21 | `CaptureMatrixWindow.Open()` | `Capture Matrix` |
| **Voice & Subtitles** | 22 | `VoiceSubtitlesWindow.Open()` | `Voice & Subtitles` |
| **Review Imports** | 23 | `ReviewImportsWindow.Open()` | `Review Imports` |
| **Documentation** | 40 | `DocumentationWindow.Open()` | `LocalizeShip Documentation` |

Minimum window sizes:

| Window | Min size |
|---|---|
| Studio | 800 × 600 |
| Guided Setup | 550 × 420 |
| Translation Handoff | 500 × 400 |
| Capture Matrix | 750 × 500 |
| Voice & Subtitles | 700 × 500 |
| Review Imports | 750 × 520 |
| Release Gates & Reports | 650 × 480 |
| Documentation | 700 × 500 |

---

## Not in the menu

| Tool | Where it lives |
|---|---|
| **Release Gates & Reports** | Toolbar button **Release Gates** inside the Studio |
| **Export Support Diagnostics** | Left filter panel of the Studio |
| **Audit History** | Studio toolbar |
| **Navigate to Source in Scene/Table** | Line detail panel in the Studio |
| **Batch Approve Visible** | Matrix header in the Studio |
| **Capture Runner** | No menu; **Run Matrix Captures** inside Capture Matrix |
| **Sample generator** | No menu; `RPGSampleSetup.GenerateSampleAssets()` is code-only |
| **Text rules / glossary checks** | No window; call the services |
| **Advisory drafts** | No window; call `AdvisoryProvider` |
| **Headless gate** | No CLI; write an `-executeMethod` entry point |

This list matters: several capabilities are libraries, not buttons.

---

## Studio toolbar

From left to right:

- Workspace object field and **Create New Workspace**;
- **Guided Setup**;
- **Handoff**;
- **Review Imports**;
- **Capture Matrix**;
- **Voice & Subtitles**;
- **Release Gates**;
- **Audit History**.

The empty state offers **Run Guided Setup (Recommended)** and **Open Ready-Made RPG Sample Release**.

The matrix header has **Batch Approve Visible...** and the filter panel has **Export Support Diagnostics**.

---

## Opening a window from code

```csharp
using Decnet.LocalizeShip.Editor.Windows;

ReleaseBoardWindow.Open(workspace);
ReleaseGateWindow.Open(workspace);
CaptureMatrixWindow.Open(workspace);
VoiceSubtitlesWindow.Open(workspace);
TranslationHandoffWindow.Open(workspace);
ReviewImportsWindow.Open(workspace);
GuidedSetupWindow.Open();
DocumentationWindow.Open();
```

Most windows accept an optional workspace; without one they show an object field and wait for you to select an asset.

---

## Sample menu behaviour

**Open Sample Release**:

1. searches `AssetDatabase.FindAssets("RPGSampleWorkspace t:ReleaseWorkspace")`;
2. loads the first match;
3. opens `Assets/Decnet/LocalizeShip/Samples/RPGSample/RPGSampleScene.unity` if the file exists;
4. opens the Studio with that workspace.

If the sample is missing, the Studio opens empty. There is no regeneration prompt.

---

## Related pages

- [Getting Started]({{ site.baseurl }}/docs/getting-started/)
- [Release Board]({{ site.baseurl }}/docs/features/release-board/)
- [Sample Setup]({{ site.baseurl }}/docs/sample/rpg-sample-studio/)
