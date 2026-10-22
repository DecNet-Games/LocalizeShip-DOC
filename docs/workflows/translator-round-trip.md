---
layout: default
title: Translator Round-Trip
parent: Production Workflows
nav_order: 2
description: "Managing external translator handoffs, formula protection, and 3-way merge conflict resolution."
permalink: /docs/workflows/translator-round-trip/
---

# Translator Round-Trip
{: .fs-9 }

### Managing External Agencies & Resolving Concurrent Edits
{: .fs-6 .text-grey-dk-000 }

---

External translation takes time. While translators work for two weeks on a 20,000-word script, your development team does not freeze work: developers fix typos in English, rewrite quest objectives, and reorder tables.

When the translator returns their spreadsheet, standard tools crash or blindly overwrite the developer's intermediate changes. LocalizeShip handles this through an auditable **Round-Trip Workflow**.

---

## Step 1: Exporting the Handoff Package

1. Open **Tools > Decnet > LocalizeShip > Translation Handoff**.
2. Select target locale (e.g., `de-DE`).
3. Set **Scope** to `Changed & Stale Lines Only` to exclude unchanged lines.
4. Check **Include Visual Context Screenshots**.
5. Click **Export Handoff Package**.

The generated ZIP archive contains:
* `LocalizeShip_Handoff_de-DE.csv` (RFC 4180 with formula protection).
* `HandoffManifest.json` (Cryptographic source revision hashes).
* `ContextScreenshots/` (Relevant UI captures).

---

## Step 2: External Translation Work

The translator opens the CSV in Excel, Google Sheets, or OmegaT. 
* Formula characters (`=`, `+`, `-`, `@`) have been safely prepended with `'` so no malicious code runs.
* Full Unicode quotes and multiline line breaks are preserved cleanly.
* Context columns provide character limits and speaker notes.

---

## Step 3: Concurrent In-Editor Changes (The Conflict)

While the translator was working, a designer edited English line `quest_01` in Unity:
* **Base (at export)**: `"Meet John at the docks."`
* **Current (in Unity)**: `"Meet Captain John at the southern docks."`
* **Incoming (from translator)**: `"Triff John an den Docks."`

---

## Step 4: Ingestion & 3-Way Merge

1. Open **Tools > Decnet > LocalizeShip > Review Imports**.
2. Select the returned CSV file.
3. The engine parses the file and immediately flags `quest_01` as a **`StaleSourceConflict`**.
4. The 3-column view clearly highlights:
   * **Base**: Original English and German.
   * **Current**: Designer's updated English and unreviewed German.
   * **Incoming**: Translator's German translation of the *old* line.
5. In the resolution dropdown:
   * If the translation is still acceptable, select **Accept Incoming**.
   * If the translation needs rework, select **Keep Current** (leaves line marked as `Stale` for the next round-trip).
6. Click **Apply Import Plan**. All changes are written to Unity tables and logged in the `ImportJournal`.
