---
layout: default
title: RPG Sample Setup
parent: RPG Sample Studio
nav_order: 1
description: "How to generate and inspect the bundled RPG Sample Studio in Unity."
permalink: /docs/sample/rpg-sample-studio/
---

# RPG Sample Studio Setup
{: .fs-9 }

### One-Click Multilingual Project Generation
{: .fs-6 .text-grey-dk-000 }

---

The **RPG Sample Studio** is generated on demand to keep package import sizes minimal and avoid cluttering your project before you ask for it.

---

## Generating the Sample

1. In the top Unity menu, go to **Tools > Decnet > LocalizeShip > Open Sample Release**.
2. Click **Generate / Open RPG Sample**.
3. LocalizeShip executes `RPGSampleSetup.cs` to generate the testbed:

```
Assets/Decnet/LocalizeShip/Samples/RPGSample/
├── Audio/
│   ├── Voice_EN_Greeting.wav
│   ├── Voice_DE_Greeting.wav
│   └── Voice_EN_Greeting_Outdated.wav
├── Data/
│   ├── RPGSampleWorkspace.asset
│   ├── Tables/ (Dialogue, Quests, Items)
│   └── Glossary/ (RPGGlossary.asset)
├── Scenes/
│   └── RPGSampleScene.unity
└── Scripts/
    └── RPGSampleUI.cs
```

---

## Exploring the Sample Scene

Double-click `RPGSampleScene.unity` to open the interactive scene:

![Sample RPG Scene]({{ site.baseurl }}/assets/images/13-release-cycle.png)

### Scene Components
* **`RPGSampleUI`**: A runtime script demonstrating dynamic string substitution, runtime locale switching, audio playback, and quest progression.
* **Dialogue Panel**: Displays branching dialogue between the Player and the Village Elder.
* **Quest Log**: Demonstrates plural smart strings (`{count, plural: =1 {1 Herb} other {{count} Herbs}}`).
* **Inventory Grid**: Demonstrates compact UI layouts with tight bounding boxes for testing text expansion.
* **Audio Voice Rig**: Features a simple audio player that auditions greeting voice takes synchronized with subtitle cues.

---

## Pre-Seeded Defects for QA Practice

The sample is deliberately seeded with realistic localization defects to help you learn the tool:

1. **German Quest Text Overflow**: The German description for the herbalist quest is 38% longer than English, overflowing its parent `RectTransform`.
2. **Outdated English Voice Take**: The English greeting audio (`Voice_EN_Greeting_Outdated.wav`) was recorded against an older version of the script, triggering an instant `VoiceStale` warning.
3. **Missing Japanese Character**: A specialized quest kanji character is missing from the fallback font atlas, triggering a `MissingFontGlyph` critical finding.

In the next guide, we will walk through resolving these issues!
