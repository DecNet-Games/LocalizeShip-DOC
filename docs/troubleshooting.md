---
layout: default
title: Troubleshooting
nav_order: 15
description: "Diagnosing common setup, import, capture and gate problems with the shipped UI and services."
permalink: /docs/troubleshooting/
---

# Troubleshooting
{: .fs-9 }

### Symptom, cause, action
{: .fs-6 .text-grey-dk-000 }

---

## Setup and compilation

### `The type or namespace name 'Localization' does not exist`

The adapters assembly references `Unity.Localization` unconditionally.

**Fix:** install `com.unity.localization` (verified with 1.5.13). Do not try to remove the reference; the package is a dependency.

### `The type or namespace name 'TMPro' does not exist`

TextMeshPro is required by the runtime and editor assemblies. In Unity 6 it is normally present via `com.unity.ugui`. Confirm the package is installed and let Unity recompile.

### Guided Setup opens but shows only “Not Installed” providers

Expected for I2 and Dialogue System without their compile symbols. Unity Localization should show **Active (1.5.13)**. If it does not, the adapters assembly failed to compile; fix the console errors first.

### Guided Setup created a workspace with zero lines

Expected. Guided Setup does not import strings. See [First Release]({{ site.baseurl }}/docs/workflows/first-release/) for ingestion options.

---

## Review Board

### Matrix is empty

- Confirm a workspace asset is assigned.
- Confirm `SourceRevisions` is not empty (inspect the asset).
- Clear the search and state filters.

### A cell shows `—` or no state

There is no `ApprovalRecord` for that entry and locale. Filter **Missing**, open the cell, enter text and approve, or import a translation.

### Approving does not stick after restart

The Studio calls `EditorUtility.SetDirty` but persistence depends on Unity saving assets. Use **Ctrl+S** / **File → Save Project** before closing, and confirm the asset is not read-only or in a package cache.

### Everything turned `Stale` after an edit

A source or context change invalidates approvals by design. If it happened without an edit, check whether your ingestion script changed speaker/context or used a different normalization policy.

### “Batch Approve Visible” approved more than intended

It acts on the current filtered/visible set. Clear filters and review the visible count before confirming.

---

## Handoff and import

### A leading `'` appears in exported text

Formula protection. It is removed on CSV import. JSON export stores the protected form, so a JSON consumer may see the apostrophe; strip it only if you know the original started with `=`, `+`, `-`, `@`, tab or CR.

### Import plan shows every row as `ConcurrentLocaleEdit`

You imported CSV into cells that already had translations. CSV reconstruction has an empty base translation, so existing text looks concurrently edited. Use JSON for true three-way comparison.

### `UnknownIdError` for rows that look correct

The incoming ID does not exist in the workspace. Check for whitespace, renamed keys, or a handoff from a different workspace.

### Apply is disabled

There is an unresolved conflict. Set **Take Incoming**, **Keep Current**, or type a **Manual Override** for each conflict.

### Apply succeeded but some rows did not change

`DuplicateIdError` and `UnknownIdError` rows are skipped. `Unchanged` rows are skipped by design. Check the classification column.

### An approved cell became `NeedsReview` after import

Expected: imported text always enters review. If it happened on a blank row, note that `BlankIncomingValue` is applied conservatively and downgrades the state.

### Rollback button disappeared

The journal is held in a static field for the current editor session. Restarting the editor loses it. Use version control.

### Rollback says a row was modified after import

That is the restore-conflict guard. Resolve manually or use `Rollback(..., force: true)` from code, accepting that you overwrite later edits.

---

## Captures and visual QA

### “No active CaptureFixture components found”

The selected scene has none, or they are on inactive objects. Open `RPGSampleScene` or add a `CaptureFixture` to your canvas root.

### Capture tile says PASS but the layout looks broken

The PNG is a placeholder, not a screenshot. Findings come from RectTransform measurements and glyph checks. Verify your container and element rects; a wrong `RootContainer` can make overflow undetectable.

### Safe-area warnings look wrong

`Screen.safeArea` is screen space; `RectTransform.rect` is local space. The comparison does not convert between them. Treat safe-area findings as heuristic.

### Missing glyphs reported for Arabic or Japanese

The TMP font asset lacks those characters. Assign a font with the required coverage or add fallbacks. This is a real finding.

### Running the matrix twice duplicates tiles

Tiles are keyed by fixture ID + locale and replaced. If duplicates appear, check for duplicate fixture IDs in the scene.

---

## Voice and subtitles

### Audition does nothing

`PlayPreviewClip` is an internal Unity API called by reflection. It may be unavailable or renamed in your Unity version. The window logs a warning. Duration and stale logic still work.

### Voice is `StaleTranslationRevision` after a manual approval

Manual approvals in the Studio pass a default translation hash. A voice entry bound to a previous translation hash will look stale. Either supply a real translation hash in your integration or treat the status as advisory until you do.

### Voice is `DurationMismatch` but the gate passes

The gate only checks `IsStale`. Duration mismatch is displayed, not gated.

### Subtitle is red but the gate is green

The gate checks invalid timing and CPS. If you expected a min/max duration or line-length failure, those policy fields are not enforced.

### CPS seems wrong for Japanese or Arabic

CPS uses `string.Length` (UTF-16 code units), not displayed characters or graphemes. Use it as a rough guardrail.

---

## Release gates

### `IncompleteCoverage` with no errors

Either a tile is `Unknown`/`Registered`, or translations are missing. Open the report's details lines and count unvisited tiles and missing cells.

### Gate fails but the Studio looks approved

Check the locale list. A locale added after review has no approvals, so every line is missing for it.

### `Failed` but violation lists look empty

Read the check details. A stale translation or subtitle CPS violation may be counted without a long list, or a check failed due to a disabled policy switch.

### Gate ignores a glossary or placeholder problem

Expected. Those are not gate checks in 1.0.0. Run `TextRuleEngine` and your own glossary check.

### JSON `OverallStatus` is a number

`JsonUtility` serializes enums as integers: `0` Passed, `1` Failed, `2` IncompleteCoverage.

---

## CI

### Unity exits 0 even though the gate failed

`-quit` does not propagate your verdict. Call `EditorApplication.Exit(code)` from your `-executeMethod`.

### `-executeMethod` cannot find the class

Use `Namespace.Class.Method`, ensure the class is in an Editor assembly, and check the log for compile errors before the method runs.

### Tests pass locally but not on the agent

Confirm the Unity version, installed packages, licence activation and project path. Batch mode does not have a GUI to recover from dialogs.

---

## Diagnostics

**Release Board → Export Support Diagnostics** writes a text file to the project root containing environment info and counts only. It does not include strings. Attach it to a support issue.

If the export fails, check that the project root is writable.

---

## Related pages

- [Support & Diagnostics]({{ site.baseurl }}/docs/support-diagnostics/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
- [FAQ]({{ site.baseurl }}/docs/faq/)
- [Installation]({{ site.baseurl }}/docs/installation/)
