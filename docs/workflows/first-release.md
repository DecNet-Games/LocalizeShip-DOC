---
layout: default
title: First Multilingual Release
parent: Production Workflows
nav_order: 1
description: "End-to-end production guide for preparing and shipping your game's first localized build."
permalink: /docs/workflows/first-release/
---

# First Multilingual Release
{: .fs-9 }

### Step-by-Step Milestone Checklist from Setup to Store Submission
{: .fs-6 .text-grey-dk-000 }

---

Shipping your first multilingual release (such as a Steam Next Fest demo or retail launch) requires coordination across narrative, engineering, and QA. Follow this checklist to ensure zero regressions:

---

## Phase 1: Environment & Workspace Setup

1. **Verify Packages**: Ensure `com.unity.localization` (1.5.13+) and TextMeshPro are installed.
2. **Launch Guided Setup**: Run `Tools > Decnet > LocalizeShip > Guided Setup`.
3. **Bind String Collections**: Select your active collections (`Dialogue`, `Quests`, `UI`).
4. **Declare Locales**: Set your primary source language (e.g., `en-US`) and target release locales (e.g., `de-DE`, `fr-FR`, `ja-JP`, `zh-CN`).
5. **Initialize Workspace**: Create `MainReleaseWorkspace.asset` in your project.

---

## Phase 2: Terminology & Stable Identifiers

1. **Lock the Glossary**: Define critical brand names, character names, and lore terminology in the Glossary tab to prevent inconsistent naming.
2. **Index Stable Entry IDs**: Run an initial scan to ensure all keys possess immutable `StableEntryId`s.
3. **Assign Metadata**: Add speaker tags and context notes to quest and dialogue lines.

---

## Phase 3: Translation Handoff & Merge

1. **Export Translation Package**: Open **Translation Handoff**, select all target languages, and export to RFC 4180 CSV with formula escaping enabled.
2. **Translate**: External translators work in Excel, Google Sheets, or a CAT tool.
3. **Ingest via 3-Way Merge**: Open **Review Imports**. Ingest returned files, review any conflicts, and apply changes with an atomic `ImportJournal`.

---

## Phase 4: Visual QA & Audio Audition

1. **Register Screen Fixtures**: Attach `CaptureFixture` components to key UI screens (Main Menu, Inventory, Settings, Dialogue, HUD).
2. **Run Capture Matrix**: Execute the full matrix across all target languages.
3. **Fix Geometric Defects**: Adjust `RectTransform` constraints or enable font Auto Size on any text overflow findings.
4. **Verify Voice & Subtitles**: Audition dialogue lines in **Voice & Subtitles** to ensure reading speed is under 17.0 CPS and all bound voice clips match current text revisions.

---

## Phase 5: Release Gate & Publishing Sign-Off

1. **Run Release Gates**: Open **Release Gate** and click **Evaluate Release Gates**.
2. **Inspect Verdict**: Ensure the verdict is **`PASSED`** with zero critical defects, zero unvisited fixtures, and zero stale translations.
3. **Export HTML Report**: Generate `ReleaseReport_v1.0.html` and attach it to your release archive or internal QA sign-off ticket.
