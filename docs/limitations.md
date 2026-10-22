---
layout: default
title: Limitations
nav_order: 14
description: "The honest boundary of LocalizeShip 1.0.0: what is a window, what is a library, what is a scaffold, and what is not shipped."
permalink: /docs/limitations/
---

# Limitations
{: .fs-9 }

### Read this before you promise anything
{: .fs-6 .text-grey-dk-000 }

LocalizeShip is a release-review layer with real domain logic. It is not a complete localization platform. This page lists the gaps that matter in production, without hedging.

---

## Workflow gaps

### No source-import window

There is no menu item or window that imports strings from Unity Localization, I2, Dialogue System or a file. Guided Setup creates an empty workspace. You populate `SourceRevisions` with a script, using the adapter or your own reader.

This is the largest practical gap in 1.0.0.

### No provider write-back from the UI

Applying an import writes to the workspace. It does not write to Unity Localization tables. The Unity adapter has a `WriteTranslation` method, but no window calls it. You must integrate that yourself.

### No delete or rename workflow

There is no supported way to delete a revision, rename a key while preserving history, or merge two identities. Removing entries requires editing the asset or writing a script, and history is lost with the record.

### No persistent import journal

The Review Imports window keeps the last journal in a static field. Closing the editor loses it. Rollback across sessions is not available. Use version control for durable rollback.

### No release-diff window

Comparing releases means diffing exported JSON reports or the workspace asset in Git. There is no side-by-side comparison UI.

---

## Provider gaps

| Provider | Reality |
|---|---|
| Unity Localization | Implemented for String Tables; uses the shared key, not the numeric ID; no asset tables, metadata, comments or Addressables awareness |
| I2 Localization | Compile-isolated scaffold; no data access even with the symbol defined |
| Dialogue System | Compile-isolated scaffold; no data access even with the symbol defined |

The I2 and Dialogue System adapters are verified only for clean compilation and safe provider-absent behaviour. They are not licensed-install integrations.

---

## QA gaps

### No player capture

`CaptureRunner` runs in the editor against live scene objects. There is no player bridge, no device capture, no multi-frame settling. `isPlayerMode` only labels the tile.

### Screenshots are placeholders

The PNG written by `CaptureRunner` is a solid background colour. Findings come from RectTransform and font measurements, not from the image. Never present the PNG as rendered evidence.

### Fixture-only coverage

A screen with no `CaptureFixture` has no coverage. The matrix finds active fixtures in the open scene; it does not open other scenes or prefabs.

### Gate blind spots

The gate checks approval state, visual errors, stale voice and subtitle timing/CPS. It does not check:

- placeholders or rich-text validity;
- glossary violations;
- merge conflicts;
- missing or duration-mismatched voice;
- subtitle min/max duration or line length;
- stale subtitle source hashes;
- warnings;
- player rendering.

### Geometry assumptions

Safe-area checks compare rects that may be in different coordinate spaces (`Screen.safeArea` vs local RectTransform units). Results can be offset or spurious.

### Failed tiles

A capture tile marked `Failed` is not counted as unvisited and may have no findings, so it can effectively pass the visual check. Track failures separately.

---

## Text and language

- Placeholder extraction is regex-based and does not parse nested ICU expressions.
- Glossary checking only finds forbidden variants; it does not require approved terms.
- The glossary has no UI and no gate enforcement; `BlockOnGlossaryViolations` is inert.
- Pseudolocalization is a crude accent/padding/reversal helper, not a shaping or RTL test.
- No linguistic, grammatical, cultural or RTL certification is claimed.
- CPS uses UTF-16 length, not grapheme clusters.

---

## Advisory drafting and automation

- No HTTP client, prompt templates or model calls ship.
- `AdvisoryProvider` validates a payload you already obtained.
- No auto-approval path exists.
- No CLI ships. CI requires a custom `-executeMethod` entry point and an activated Unity licence.
- No background or scheduled jobs.

---

## Data and scale

- All lookups are linear scans over lists.
- No indexing, paging or virtualized matrix.
- No performance claim beyond the sample and test workspaces.
- The workspace asset grows with approval history; very large histories make the asset heavy and diffs noisy.
- No built-in pruning or archiving of history.

---

## Platform support

- Verified on Unity **6000.5.4f1** only.
- Requires Unity Localization **1.5.13** (the adapters assembly hard-references it).
- Addressables **1.25.0** is referenced; no direct feature depends on it.
- No claim for Unity 2022.3, 2023, consoles or mobile player capture.

---

## Non-claims

LocalizeShip does **not**:

- certify a release as linguistically correct;
- replace a translator, reviewer or LQA pass;
- guarantee that an approved string renders correctly in a player;
- redistribute I2 or Pixel Crushers code;
- send your data anywhere;
- decide policy for you when a check is non-blocking.

---

## How to work within the limits

1. Write a source-ingestion script and keep it in your project.
2. Call text rules before approving; do not rely on the gate for tags or placeholders.
3. Re-run captures after layout-affecting edits; tiles do not invalidate by hash.
4. Prefer JSON handoffs.
5. Archive reports and workspace commits together.
6. Decide explicitly which non-gated issues are release blockers.

---

## Related pages

- [Coverage Honesty]({{ site.baseurl }}/docs/concepts/coverage-honesty/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
- [Testing & Verification]({{ site.baseurl }}/docs/testing-and-verification/)
- [FAQ]({{ site.baseurl }}/docs/faq/)
