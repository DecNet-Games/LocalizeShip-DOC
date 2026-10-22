---
layout: default
title: First Multilingual Release
parent: Production Workflows
nav_order: 1
description: "A milestone checklist for taking a project from monolingual to its first localized release with LocalizeShip."
permalink: /docs/workflows/first-release/
---

# First Multilingual Release
{: .fs-9 }

### From one language to an auditable set
{: .fs-6 .text-grey-dk-000 }

This is the long path: set up the workspace, get source strings in, send them out, bring translations back, check the UI, and produce evidence. Budget real time for the ingestion step — it is the one place 1.0.0 does not give you a window.

![Release cycle]({{ site.baseurl }}/assets/images/01-card.png)

---

## Phase 0 — Decide scope

Write these down before opening Unity:

- source locale (`en` is typical);
- target locales, in priority order;
- which surfaces are in scope: UI, dialogue, items, quests, subtitles, voice;
- who reviews each locale;
- what “done” means for this release: all approved? all captured? all voice current?
- where reports and handoffs are archived.

A release with three locales and no named reviewers is not a plan.

---

## Phase 1 — Install and verify

1. Install the package per [Installation]({{ site.baseurl }}/docs/installation/).
2. Confirm Unity Localization 1.5.13+ is present.
3. Open **Tools → Decnet → LocalizeShip → Release Board** once.
4. Run the EditMode acceptance suite. Expect **12/12 passing**.

If the suite does not pass, stop. Do not build a release process on a red baseline.

---

## Phase 2 — Create the workspace

**Tools → Decnet → LocalizeShip → Guided Setup**

1. Choose the provider ID (`UnityLocalization` for the official package).
2. Enter source and target locales.
3. Choose a name and directory.
4. Create the workspace.

What you get: a `ReleaseWorkspace` asset with locales and an empty revision list. What you do not get: imported strings.

{: .warning }
> The “Include Sample Test Fixtures” toggle is present in the wizard but is not read by `ExecuteSetup`. It does not add fixtures.

---

## Phase 3 — Ingest source strings

This is the manual gap. Options:

### A. Use the Unity Localization adapter from a script

```csharp
var adapter = new UnityLocalizationAdapter();
var revisions = adapter.ReadCollection("UI", "en", workspace.Normalization);
foreach (var rev in revisions) workspace.SourceRevisions.Add(rev);
```

See [Unity Localization]({{ site.baseurl }}/docs/providers/unity-localization/) for a full example that uses `RevisionTracker` to avoid duplicates.

### B. Export from your own pipeline

If your strings live in CSV/JSON, write a small importer that constructs `SourceRevision` objects with stable keys. See [Custom Adapters]({{ site.baseurl }}/docs/providers/custom-adapters/).

### C. Start from the sample

Copy the sample workspace and replace its entries. This is fine for a prototype, not for production.

After ingestion, open the Release Board and confirm:

- the row count matches your expectation;
- keys are stable and readable;
- speaker and context are populated where you need them;
- placeholders were extracted (inspect `Placeholders` on a revision).

---

## Phase 4 — Normalize and freeze

The workspace uses a `NormalizationPolicy` (default version 1: trim, CRLF→LF, preserve duplicate spaces, case-sensitive). Decide whether that matches your pipeline. If you change the policy after approving translations, hashes change and approvals go stale.

Freeze source text for the handoff. If writers are still editing, you will produce stale conflicts.

---

## Phase 5 — Handoff

Open **Translation Handoff**:

1. Select a locale.
2. Export JSON (recommended) and CSV (convenience).
3. Send the package with instructions.

What the package contains:

- entry ID, source text, current translation;
- base source hash;
- speaker and context (optional);
- required placeholders (JSON only);
- a relative path field that is populated from `SceneOrFixture`, not a real screenshot (JSON only).

What it does not contain: screenshots, glossary, audio, max length, or a translation memory.

---

## Phase 6 — Receive and review

When translations come back:

1. Open **Review Imports**.
2. Select the JSON if you need true three-way comparison; CSV if you accept its limits.
3. Read each classification: safe change, unchanged, stale source conflict, concurrent edit, duplicate, unknown, blank.
4. Resolve conflicts explicitly.
5. Apply the plan.

Imported text enters `NeedsReview`. Nothing is auto-approved.

Rollback is available only for the last applied journal in the current window session. It is not persisted across editor restarts.

---

## Phase 7 — Visual QA

1. Open the scene with your `CaptureFixture`.
2. Open **Capture Matrix**.
3. Run the matrix for every target locale.
4. Inspect findings: overflow, safe area, touch target, missing glyphs.
5. Fix layouts or fonts, then re-run.

The saved PNG is a solid-colour placeholder. The findings come from live RectTransforms and TMP font checks, not from the image. Do not attach the PNG as visual evidence of rendering.

If a screen has no fixture, it has no coverage. Add fixtures or accept that the gate cannot see it.

---

## Phase 8 — Voice and subtitles

1. Bind voice takes to entry + locale + source hash.
2. Bind subtitle cues with start/end times.
3. Review stale takes after any source or translation change.
4. Check CPS and invalid timing.

Remember: duration mismatch is displayed but not gated; missing audio is displayed but not gated.

---

## Phase 9 — Gate

Generate the report from the Release Board.

Read it in this order:

1. verdict;
2. passed/total checks;
3. details per failed check;
4. violations.

Separate **Failed** (defects) from **IncompleteCoverage** (unvisited or missing). Then assign owners.

Before you call it green, verify the things the gate does not check:

- placeholders and rich text (run the text rules);
- glossary variants (if terminology is a blocker);
- voice duration and missing audio;
- subtitle min/max duration and line length;
- player-build rendering.

---

## Phase 10 — Archive and ship

- Export HTML and JSON reports.
- Store them with the build number.
- Record the verdict change versus the previous release.
- Keep the handoff packages and the applied journal metadata.
- Note the manual steps in your release runbook so the next release is repeatable.

---

## Definition of done

A first release is done when:

- every in-scope line has an approval that matches the current source hash;
- every in-scope screen has a visited capture tile;
- voice and subtitle issues are either fixed or explicitly accepted;
- the gate verdict is understood, including its blind spots;
- evidence is archived and reproducible.

“The gate is green” is necessary, not sufficient.

---

## Related pages

- [Translator Round-Trip]({{ site.baseurl }}/docs/workflows/translator-round-trip/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
- [Coverage Honesty]({{ site.baseurl }}/docs/concepts/coverage-honesty/)
- [Troubleshooting]({{ site.baseurl }}/docs/troubleshooting/)
