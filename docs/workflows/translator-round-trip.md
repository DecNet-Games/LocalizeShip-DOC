---
layout: default
title: Translator Round-Trip
parent: Production Workflows
nav_order: 2
description: "Export, translate externally, import safely, resolve three-way conflicts and understand the CSV/JSON trade-offs."
permalink: /docs/workflows/translator-round-trip/
---

# Translator Round-Trip
{: .fs-9 }

### Send hashes, not hopes
{: .fs-6 .text-grey-dk-000 }

A round-trip is only safe if the returning file can be compared against the exact base it was translated from. The handoff manifest carries that base. The merge planner uses it to distinguish a clean update from a concurrent edit.

![Translation handoff flow]({{ site.baseurl }}/assets/images/11-translation-handoff.png)

---

## 1. Freeze the source

Every handoff records the current `RevisionHash` of each line. If a writer edits the source after export, every returning translation for that line becomes a conflict. That is correct behaviour, but it is expensive.

Before export:

- confirm writers are done with the batch;
- run your normalization policy consistently;
- check that keys are final.

---

## 2. Export

**Tools → Decnet → LocalizeShip → Translation Handoff**

Options:

| Option | Effect |
|---|---|
| Target Locale | One locale per package |
| Only Changed / Unapproved Lines | Skips cells already `Approved` |
| Include Speaker & Context Notes | Fills those fields; otherwise they are `null` |
| Include Relative Screenshot Paths | Copies `SceneOrFixture` into `RelativeScreenshotPath` — **not a screenshot path** |
| Output Folder | Asset-relative or absolute; the folder is created |

Files:

```
Handoff_<ReleaseVersion>_<LOCALE>.json
Handoff_<ReleaseVersion>_<LOCALE>.csv
```

### JSON

`JsonUtility.ToJson(manifest, true)`. Contains the full manifest:

- `SchemaVersion`, `ReleaseId`, `WorkspaceId`, `SourceLocale`, `ExportTimestampTicks`;
- `TargetLocales` (one);
- for each entry: ID, base source hash, source text, target locale, base translation hash, current translation, speaker, context, relative path, and constraints with `RequiredPlaceholders`.

### CSV

Header:

```csv
EntryId,SourceText,Translation,Speaker,Context,BaseSourceHash
```

The CSV does **not** contain release metadata, base translation hash, required placeholders, constraints, or screenshot paths. It is a convenience format for spreadsheets and CAT-adjacent tooling, not a full manifest.

---

## 3. Formula protection

Before writing, the export prefixes text starting with `=`, `+`, `-`, `@`, tab or CR with an apostrophe. This prevents spreadsheet formula injection.

- CSV export protects, CSV import unprotects.
- JSON export also stores the protected strings (the manifest is built once and shared).
- A JSON consumer that displays `SourceText` may see a leading apostrophe on such lines.

Do not “clean” the apostrophe manually unless you know the original text started with one of those characters.

---

## 4. Translator instructions

Tell the translator:

- edit only the `Translation` / `CurrentTranslationText` field;
- never change `EntryId` or `BaseSourceHash`;
- keep placeholders exactly (`{count}`, `{name}`);
- do not add or remove columns in CSV;
- return the whole file, not a filtered extract;
- do not paste formatted text with smart quotes if your pipeline expects plain text.

A translator changing the base hash destroys three-way detection. Treat that as a broken return.

---

## 5. Import

**Tools → Decnet → LocalizeShip → Review Imports**

1. Select the workspace and target locale.
2. Browse to the JSON or CSV.
3. Review the plan summary and grid.

### JSON path

The manifest supplies the base translation and source hash from export time, giving a true three-way comparison:

| Base | Current | Incoming | Classification |
|---|---|---|---|
| same source | same translation | different | `SafeIncomingChange` |
| old source | any | any | `StaleSourceConflict` |
| same source | edited in project | edited differently | `ConcurrentLocaleEdit` |
| same | same | same | `Unchanged` |
| — | — | empty | `BlankIncomingValue` |
| duplicate ID in incoming | — | — | `DuplicateIdError` |
| ID not in workspace | — | — | `UnknownIdError` |

### CSV path

CSV import reconstructs a base manifest with:

- the source hash from the file;
- `CurrentTranslationText = ""` for every base row.

Because the base translation is empty, any existing non-empty translation that differs from the incoming one is classified as `ConcurrentLocaleEdit`. CSV round-trips therefore over-report conflicts when a cell already has text. Prefer JSON for real three-way merges.

---

## 6. Resolve conflicts

Only `StaleSourceConflict` and `ConcurrentLocaleEdit` are conflicts. For each, choose:

- **Take Incoming** — accept the translator's text;
- **Keep Current** — keep the project text, no state change;
- **Manual Override** — type the merged text.

Apply is blocked while any conflict has no resolution. The button is disabled, and `ApplyCoordinator` also throws if called directly.

### What apply does

- skips `DuplicateIdError` and `UnknownIdError` rows (they are not surfaced as a thrown error);
- skips `Unchanged`;
- for every other row, writes the resolved/current/incoming text;
- moves the cell to `NeedsReview`;
- records an approval event with reviewer `"Local QA Reviewer"` and rationale `"Applied from import plan <id>"`.

{: .warning }
> A `BlankIncomingValue` row is not blocking and has no resolution control. On apply it takes the current value and re-marks the cell `NeedsReview`, which can downgrade an approved cell. Review blank rows before applying.

Imported text is never auto-approved. That is the core safety property of the import path.

---

## 7. Rollback

After apply, the window offers **Rollback Last Journal**.

It restores, for each row:

- previous translation text;
- previous review state;
- previous approved source hash.

It does **not** restore `ApprovedTranslationHash`, and it does not remove the approval events written by the apply. The audit trail will still show the import. Rollback is a data correction, not an audit erasure.

Rollback refuses if a row changed after apply (current text differs from applied text). The API supports `force: true`, which the window does not expose.

The journal lives in the window's static field for the session. It is not serialized into the workspace and is lost on editor restart.

---

## 8. After import

1. Open the Release Board and filter `NeedsReview`.
2. Re-run text rules on the imported cells if placeholders matter.
3. Approve cell by cell.
4. Check voice entries for the affected lines; translation-hash binding may mark them stale.
5. Re-run the gate.

---

## Conflict playbook

| Situation | Action |
|---|---|
| Writer polished a line after export | `StaleSourceConflict` — compare meaning, ask for a retranslation if needed |
| Another reviewer edited the same cell | `ConcurrentLocaleEdit` — merge manually, do not blindly take either side |
| Translator returned a subset | Missing rows stay as they were; the plan does not delete |
| Translator changed IDs | `UnknownIdError` — reject the file and ask for a corrected export |
| Duplicate rows | `DuplicateIdError` — deduplicate at the source |
| Empty translation | Review manually; do not apply a batch containing blanks |

---

## Related pages

- [Translation Handoff]({{ site.baseurl }}/docs/features/translation-handoff/)
- [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/)
- [Review States]({{ site.baseurl }}/docs/concepts/review-states/)
- [Audit Trail]({{ site.baseurl }}/docs/concepts/audit-trail/)
