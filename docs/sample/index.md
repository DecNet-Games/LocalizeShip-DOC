---
layout: default
title: RPG Sample Studio
nav_order: 8
has_children: true
description: "The bundled RPG sample: a pre-generated workspace, scene, UI and synthetic audio for evaluating the release workflow."
permalink: /docs/sample/
---

# RPG Sample Studio
{: .fs-9 }

### A small, pre-generated project to learn the workflow
{: .fs-6 .text-grey-dk-000 }

The package includes a ready-made sample under `Assets/Decnet/LocalizeShip/Samples/RPGSample/`. It is not a game. It is a compact fixture with one screen, five string entries, two synthetic audio clips and two subtitle cues, sized so you can walk the whole release loop in a few minutes.

![Voice coverage]({{ site.baseurl }}/assets/images/10-voice-coverage.png)

---

## What is actually in the sample

| Asset | Content |
|---|---|
| `RPGSampleScene.unity` | One canvas with a dialogue line, quest title/description, inventory tooltip and subtitle label |
| `RPGSampleWorkspace.asset` | 5 source entries, approvals for de/ja/ar, 2 voice entries, 2 subtitle cues, 1 glossary term |
| `RPGSampleUI.cs` | Switches the four texts between en/de/ja/ar; plays the EN or DE clip |
| `Audio/Voice_EN_Greeting.wav` | 440 Hz sine tone, 2.4 s — **not speech** |
| `Audio/Voice_DE_Greeting.wav` | 330 Hz sine tone, 2.8 s — **not speech** |
| `Audio/Voice_EN_Greeting_Outdated.wav` | 520 Hz sine tone, 1.8 s — **not speech** |

The audio exists to exercise audition, duration comparison and stale binding. It is not voice acting and should not be used to judge performance quality.

---

## Seeded states

The workspace deliberately contains a mixture of states:

- `quest_title_01`, `quest_desc_01`, `dialogue_npc_01`, `item_relic_01` — approved in German and Japanese; Arabic is `NeedsReview`.
- `ui_player_gold` — German translation is missing `{count}` and is `NeedsReview`.
- `dialogue_npc_01` — voice takes bound for English and German; two subtitle cues.
- `Sunshard Amulet` — one German glossary term with two forbidden variants.

That gives you at least one approved cell, one review cell, one text-rule defect and one audio/subtitle pair to inspect.

---

## Opening it

**Tools → Decnet → LocalizeShip → Open Sample Release**

This loads `RPGSampleScene.unity` if it exists and opens the Release Board with `RPGSampleWorkspace`. If the window opens with no workspace, drag the asset into the Workspace field manually.

![Release board]({{ site.baseurl }}/assets/images/03-release-board.png)

---

## Honest limits of the sample

- No dialogue branching, quest logic or inventory system. The labels are static.
- No real localization table is used at runtime; `RPGSampleUI` holds an in-memory dictionary.
- Arabic text is displayed with the same TMP font as the other locales. If the font lacks Arabic or Japanese glyphs, the capture pass reports missing glyphs — that is the point of the check, not a bug to hide.
- No RTL layout mirroring is implemented.
- The generator method `RPGSampleSetup.GenerateSampleAssets()` is not bound to a menu item. The assets are already present; if you delete them, you must call the method from your own script or reinstall the package.
- `RPGSampleSetup` only creates assets when they are missing and returns early if the scene exists.

---

## Sample guides

- [RPG Sample Studio Setup]({{ site.baseurl }}/docs/sample/rpg-sample-studio/) — navigate the assets and understand each seeded state.
- [Scenario Walkthrough]({{ site.baseurl }}/docs/sample/scenario-walkthrough/) — a step-by-step pass from review to gate report.

---

## Related pages

- [Quick Start]({{ site.baseurl }}/docs/quick-start/)
- [Release Board]({{ site.baseurl }}/docs/features/release-board/)
- [Voice & Subtitles]({{ site.baseurl }}/docs/features/voice-and-subtitles/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
