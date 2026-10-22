---
layout: default
title: Scenario Walkthrough
parent: RPG Sample Studio
nav_order: 2
description: "Step-by-step walkthrough reproducing the complete localization release QA cycle in the RPG sample."
permalink: /docs/sample/scenario-walkthrough/
---

# Scenario Walkthrough
{: .fs-9 }

### Fixing Real Defects in the RPG Sample Studio
{: .fs-6 .text-grey-dk-000 }

---

This guide walks you through resolving the pre-seeded defects in the RPG Sample Studio to achieve a clean **PASSED** release gate verdict.

---

## Exercise 1: Resolving the German Overflow

1. Open **Tools > Decnet > LocalizeShip > Capture Matrix**.
2. Click **Run All Fixtures**.
3. In the matrix grid, locate the row for `QuestLog_Fixture` and column `de-DE`. Notice the red **`CRITICAL: Text Overflow`** finding:
   `"Text bounds exceed container width by 38.2px"`.
4. Click the tile, then click **Ping Object** to select `QuestDescriptionText` in the Hierarchy.
5. In the TextMeshPro component in the Inspector:
   * Option A: Enable **Auto Size** (Min: 14, Max: 18).
   * Option B: In the parent `RectTransform`, increase width from `240` to `290`.
6. Return to the Capture Matrix and click **Re-run Tile**.
7. The tile now turns green: **PASSED**!

---

## Exercise 2: Resolving the Stale English Voice Take

1. Open **Tools > Decnet > LocalizeShip > Voice & Subtitles**.
2. Locate entry `npc_elder_greeting`.
3. Notice the status displays <span style="color:#e67e22; font-weight:bold;">VoiceStale</span>:
   * The audio file `Voice_EN_Greeting_Outdated.wav` duration is 2.8s, but the current source line is 4.6s long.
4. In the Take Selector dropdown, switch the active take from `Take 1 (Outdated)` to `Take 2 (Current)`.
5. Click **Audition Audio** to listen to the new take.
6. The voice status updates to **Matched**, and the subtitle reading speed evaluates to a comfortable **14.2 CPS**.

---

## Exercise 3: Achieving Gate Sign-Off

1. Open **Tools > Decnet > LocalizeShip > Release Board**.
2. Filter by `NeedsReview`.
3. Select the updated German and Japanese strings and click **Batch Approve Filtered Lines**.
4. Open **Tools > Decnet > LocalizeShip > Release Gate**.
5. Click **Evaluate Release Gates**.
6. All gates turn green:
   * **Approval Completeness**: 100% (Zero missing or stale strings).
   * **Visual QA**: 0 Critical Defects.
   * **Coverage Honesty**: 3/3 Fixtures Visited.
   * **Audio/Subtitle Parity**: 100% Matched.
7. Verdict: **`PASSED`**!
8. Click **Export HTML Report** to view your publication-ready release report.
