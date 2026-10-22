---
layout: default
title: Scenario Walkthrough
parent: RPG Sample Studio
nav_order: 2
description: "A truthful end-to-end walkthrough of the sample: review, edit, capture, export and gate."
permalink: /docs/sample/scenario-walkthrough/
---

# Scenario Walkthrough
{: .fs-9 }

### Fifteen minutes, one complete release loop
{: .fs-6 .text-grey-dk-000 }

This walkthrough uses only shipped UI. Where a step requires code, it says so instead of pretending a button exists.

![Release board]({{ site.baseurl }}/assets/images/03-release-board.png)

---

## Before you start

- Unity 6000.5.4f1 with the package imported.
- `RPGSampleScene` open.
- `RPGSampleWorkspace` loaded in the Release Board.

Open with **Tools → Decnet → LocalizeShip → Open Sample Release**.

---

## Step 1 — Read the matrix

In the Release Board:

1. Confirm the locale headers are `DE`, `JA`, `AR`.
2. Use the state filter to show **Needs Review**. Expect the Arabic cells and `ui_player_gold`/DE.
3. Use the search field to filter `quest`. Expect two rows.
4. Click `dialogue_npc_01` and read the right-hand detail panel: source text, speaker `Eldrin`, context `Main greeting dialogue`.

What you are learning: the board is a review surface, not a table editor for source strings.

---

## Step 2 — Approve one cell

1. Select `dialogue_npc_01` and the `AR` cell.
2. Edit the translation text in the detail panel if you want.
3. Click **Approve**.

Expected:

- state becomes `Approved`;
- an `ApprovalEvent` is appended with your reviewer label and a timestamp;
- the summary counts update.

The source hash recorded is the current revision hash. If the source later changes, the approval becomes `Stale`.

---

## Step 3 — See the text-rule defect

`ui_player_gold` is seeded with `Du hast gold übrig.` — missing `{count}`.

The board will not flag it. To see the defect, run the guardrail from an editor script:

```csharp
var rev = workspace.GetSourceRevision(new StableEntryId("ui_player_gold"));
var app = workspace.GetApproval(rev.EntryId, new LocaleId("de"));
var violations = new TextRuleEngine()
    .ValidatePlaceholders(rev, app.TranslationText, new LocaleId("de"));

foreach (var v in violations)
    Debug.Log($"{v.Severity} {v.RuleName}: {v.Message}");
```

Expected: a `MissingPlaceholder` error for `count`.

Fix the text in the cell, approve it, and re-run to see zero errors. This is the correct division of responsibility in 1.0.0: the rule engine exists, the board does not call it.

---

## Step 4 — Check voice coverage

Open **Voice & Subtitles**.

1. Both entries should be `BoundCurrent` (green).
2. Click **Audition** on the EN entry. You should hear a 440 Hz tone, not speech.
3. Note the durations: 2.4 s and 2.8 s.

To demonstrate staleness without editing the scene:

```csharp
var tracker = new RevisionTracker();
tracker.ProcessSourceEntry(
    workspace,
    new StableEntryId("dialogue_npc_01"),
    "Greetings, traveler. Welcome to our sanctuary. Beware of the coming dark.",
    "Eldrin",
    "Main greeting dialogue");
EditorUtility.SetDirty(workspace);
```

Reopen Voice & Subtitles. Both takes should be `StaleSourceRevision`. The source edit also marks the de/ja/ar approvals stale in the Release Board.

This is the core promise of the product: one edit, visible downstream impact.

{: .warning }
> You have now modified the sample workspace. Restore it from source control or reinstall the sample before repeating the walkthrough.

---

## Step 5 — Review the subtitles

Open the **Subtitles** tab.

- EN: 0–2.4 s.
- DE: 0–2.8 s.

CPS is `text.Length / duration`. If a cue exceeds 18 CPS it turns red. Invalid timing is highlighted and will fail the gate.

There is no min/max duration enforcement in the gate, despite the policy fields. Do not claim otherwise in your review notes.

---

## Step 6 — Run the visual matrix

1. With `RPGSampleScene` open, open **Capture Matrix**.
2. Click **Run Matrix Captures**.
3. Inspect a tile.

Expected:

- tiles for `de`, `ja`, `ar` become `PASS` or `DEFECT DETECTED`;
- findings list overflow, safe-area or missing-glyph issues;
- the saved PNG is a flat colour placeholder.

If Arabic or Japanese glyphs are missing from the TMP font, that is a genuine finding from `font.HasCharacter`, not a rendering artifact of the placeholder image.

---

## Step 7 — Export a handoff

Open **Translation Handoff**.

1. Choose `de`.
2. Export JSON and CSV to a folder you control.
3. Open the CSV and confirm the header and rows.

The export contains source text, current translation, base hashes and constraints. Formulas are escaped. It does not contain glossary columns or audio.

---

## Step 8 — Round-trip an import

1. Edit one translation in the CSV (change a word, keep the entry ID).
2. Open **Review Imports**, select the CSV, and load it.
3. The planner classifies the row. If the source hash matches the base, it is a safe incoming change.
4. Apply the plan.
5. Check the board: the cell should be updated and in review, not silently approved.

For a conflict demonstration, change the source first (Step 4), then import a CSV based on the old source. The planner should classify it as a conflict and block apply until you resolve it.

---

## Step 9 — Generate the gate report

1. Return to the Release Board and open **Release Gates**.
2. Generate the report.
3. Read the verdict and the four details lines.

Expected on a fresh sample with unvisited captures and Arabic review pending: not `Passed`. After approving everything, running the matrix and fixing `ui_player_gold`, it can pass.

Export both HTML and JSON. Open the HTML in a browser and confirm it is self-contained and readable.

---

## Step 10 — Record what you learned

Write down, for your own project:

- which surface you will use for source ingestion (the sample does not include one);
- whether you will call text rules before approval;
- how you will handle voice duration mismatch, which the gate ignores;
- who owns Arabic review and font coverage;
- where reports will be archived.

Those five decisions matter more than the sample's green cells.

---

## Related pages

- [Sample Setup]({{ site.baseurl }}/docs/sample/rpg-sample-studio/)
- [Release Board]({{ site.baseurl }}/docs/features/release-board/)
- [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
