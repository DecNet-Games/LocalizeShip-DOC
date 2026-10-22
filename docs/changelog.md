---
layout: default
title: Changelog
nav_order: 18
description: "Release history and version notes for LocalizeShip."
permalink: /docs/changelog/
---

# Changelog & Release Notes
{: .fs-9 }

### Complete Release History for LocalizeShip
{: .fs-6 .text-grey-dk-000 }

---

## [1.0.0] — 2026-10-22

### Initial Commercial Release

#### Core Architecture
* **Stable Line Identities (`Identifiers.cs`)**: Type-safe `StableEntryId`, `SourceRevisionHash`, `TranslationRevisionHash`, and `LocaleId` structures.
* **Revision Graph & Normalization (`SourceRevision.cs`)**: Deterministic text normalization (CRLF/whitespace), placeholder extraction, and SHA-256 revision hashing.
* **The 6 Review States (`ReviewState.cs`)**: Explicit state machine tracking `Missing`, `Draft`, `NeedsReview`, `Approved`, `Stale`, and `Rejected`.
* **Impact Engine (`ImpactEngine.cs`)**: Multi-asset cascade tracking across changed source, stale translations, voice re-records, subtitle reviews, and visual rechecks.

#### Editors & Windows
* **Release Board Window (`ReleaseBoardWindow.cs`)**: High-performance 3-pane release workspace with real-time status filtering and batch approvals.
* **Guided Setup Wizard (`GuidedSetupWindow.cs`)**: 3-step setup with capability detection and dry-run preview.
* **Translation Handoff (`TranslationHandoffWindow.cs`, `HandoffCsvSerializer.cs`)**: RFC 4180 CSV and JSON handoff export with bidirectional formula injection protection (`=`, `+`, `-`, `@`, `\t`, `\r`).
* **Review Imports (`ReviewImportsWindow.cs`, `MergePlanner.cs`)**: True 3-way merge review with conflict detection and atomic rollback via `ImportJournal`.
* **Capture Matrix (`CaptureMatrixWindow.cs`, `CaptureRunner.cs`)**: Automated grid execution across screen fixtures, viewports, and target locales.
* **Voice & Subtitles Studio (`VoiceSubtitlesWindow.cs`)**: In-Editor audio auditioning via `AudioUtil` and subtitle reading speed (CPS) threshold validation.
* **Release Gate & Reports (`ReleaseGateWindow.cs`, `ReleaseGateEvaluator.cs`)**: Deterministic gate evaluation with standalone HTML and JSON report exports.

#### Provider Integrations
* **Unity Localization Adapter (`UnityLocalizationAdapter.cs`)**: Full 2-way adapter for `com.unity.localization` 1.5.13+.
* **I2 Localization Adapter (`I2LocalizationAdapter.cs`)**: Compile-isolated module (`LOCALIZESHIP_I2_SUPPORT`) verified for clean compilation and provider-absent safety.
* **Pixel Crushers Dialogue System Adapter (`DialogueSystemAdapter.cs`)**: Compile-isolated module (`LOCALIZESHIP_PIXELCRUSHERS_SUPPORT`) verified for clean compilation and provider-absent safety.

#### Samples & Automated Testing
* **RPG Sample Studio (`RPGSampleSetup.cs`, `RPGSampleUI.cs`)**: One-click multilingual RPG workshop with pre-seeded layout overflows, audio cues, and dialogue trees.
* **12 EditMode Acceptance Test Suites (`LocalizeShipAcceptanceTests.cs`)**: Verified and 100% passing (12/12) on Unity 6000.5.4f1 LTS (`LS_T01` to `LS_T12`).
