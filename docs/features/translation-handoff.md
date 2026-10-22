---
layout: default
title: Translation Handoff
parent: Features
nav_order: 4
description: "Export revision-bound JSON manifests and RFC 4180-style CSV packages with speaker context, base hashes, placeholder constraints and formula escaping."
permalink: /docs/features/translation-handoff/
---

# Translation Handoff
{: .fs-9 }

### Send the translator the revision, not just the words
{: .fs-6 .text-grey-dk-000 }

**Tools → Decnet → LocalizeShip → Translation Handoff**

A plain export of key/value pairs loses the two things a translator and a merge engine both need: the source revision the translation was based on, and the context that changes meaning. LocalizeShip exports a manifest with both, in a format a spreadsheet can open and a parser can trust.

![Translation handoff]({{ site.baseurl }}/assets/images/11-translation-handoff.png)

---

## Export options

| Option | Default | Behaviour |
|---|---|---|
| **Target Locale** | First workspace target | Selects one locale per export |
| **Only Changed / Unapproved Lines** | On | Skips cells whose state is `Approved`; includes everything else |
| **Include Speaker & Context Notes** | On | Fills `Speaker` and `ContextDescription` from the source revision |
| **Include Relative Screenshot Paths** | On | Fills `RelativeScreenshotPath` with the source revision's `SceneOrFixture` |
| **Output Folder** | `Assets/Decnet/LocalizeShip/HandoffExport` | Created if missing |

{: .note }
> “Only Changed / Unapproved” is literal: it skips only **Approved**. A `NeedsReview`, `Stale`, `Rejected`, `Draft` or `Missing` cell is included. There is no “changed since date” comparison in 1.0.0.

### What is not copied

The checkbox says “screenshot paths”, but the exporter writes `rev.SceneOrFixture`, not a file path, and does not copy any image. Treat it as a fixture/screen identifier for the translator's reference.

The constraint object also has `MaxLengthCharacters`, `MaxLines` and `ContextNotes`. The export window currently fills only `RequiredPlaceholders`; the other fields exist for your own integrations.

---

## Files written

For release `1.0.0` and locale `de`:

```
Handoff_1.0.0_DE.json
Handoff_1.0.0_DE.csv
```

Both are written with UTF-8 encoding. The folder is revealed in the OS file browser when the export finishes.

---

## The JSON manifest

`HandoffManifest` is serialized with Unity's `JsonUtility` (pretty printed). Structs serialize as objects with their single field, which matters if a tool round-trips the file.

Representative shape:

```json
{
  "SchemaVersion": "1.0.0",
  "ReleaseId": "1.0.0",
  "WorkspaceId": "rpg_sample_v1",
  "SourceLocale": "en",
  "TargetLocales": ["de"],
  "ExportTimestampTicks": 638965000000000000,
  "Entries": [
    {
      "EntryId": { "Value": "quest_title_01" },
      "BaseSourceHash": { "Hash": "9f2c1a7b3d4e5f60" },
      "SourceText": "The Whispering Glade",
      "TargetLocale": { "Code": "de" },
      "BaseTranslationHash": { "Hash": "" },
      "CurrentTranslationText": "Die flüsternde Lichtung",
      "Speaker": "Eldrin",
      "ContextDescription": "Quest journal main title",
      "RelativeScreenshotPath": "RPGSample_DialogueAndQuest",
      "Constraints": {
        "MaxLengthCharacters": 0,
        "MaxLines": 0,
        "ContextNotes": "",
        "RequiredPlaceholders": []
      }
    }
  ]
}
```

Fields that matter for the round-trip:

- `BaseSourceHash` — the revision the translator is working against. The merge planner uses it to detect stale-source conflicts.
- `BaseTranslationHash` — the approved translation hash at export time (empty when not approved).
- `CurrentTranslationText` — the text as it existed at export.
- `RequiredPlaceholders` — extracted from the source line.

{: .warning }
> The JSON is a Unity `JsonUtility` shape, not a generic hand-written schema. If an external tool rebuilds the file, it must preserve the nested `{ "Value": ... }`, `{ "Hash": ... }` and `{ "Code": ... }` objects, or use the CSV format instead.

---

## The CSV format

Header, always in this order:

```csv
EntryId,SourceText,Translation,Speaker,Context,BaseSourceHash
```

Example rows:

```csv
"EntryId","SourceText","Translation","Speaker","Context","BaseSourceHash"
"quest_title_01","The Whispering Glade","Die flüsternde Lichtung","Eldrin","Quest journal main title","9f2c1a7b3d4e5f60"
"dialogue_npc_01","Greetings, traveler.","Seid gegrüßt, Reisender.","Eldrin","Main greeting dialogue","41ab90ce77d2015b"
```

Parsing behaviour:

- All exported fields are quoted.
- Embedded double quotes are doubled (`""`).
- Embedded newlines are preserved inside quotes.
- The parser accepts `\r\n`, `\n` and lone `\r` row separators.
- Header names are matched case-insensitively; unknown columns are ignored; missing columns default to empty.
- Rows with a blank entry ID are skipped.

The parser is an RFC 4180-style state machine written for this format. It is not a general-purpose CSV library and does not support configurable delimiters or encodings other than the text you pass in.

---

## Formula-injection protection

Spreadsheet applications treat cells beginning with `=`, `+`, `-`, `@`, a tab or a carriage return as formulas. A translation containing `=SUM(...)` or `+cmd|...` can become an attack when a reviewer opens the file.

`MergePlanner.ProtectFormula` prefixes such text with a single quote:

```csharp
if (text.StartsWith("=") || text.StartsWith("+") || text.StartsWith("-") ||
    text.StartsWith("@") || text.StartsWith("\t") || text.StartsWith("\r"))
{
    return "'" + text;
}
```

`UnprotectFormula` removes the quote only when the following character is one of those triggers. The CSV serializer applies protection to source **and** translation text; the CSV parser unprotects on read.

{: .important }
> Because `-` is a trigger, a legitimate negative number such as `-5` is exported as `'-5`. This is safe and reversible, but it will look odd in a raw spreadsheet cell until it is unquoted on import. Do not “clean up” exported CSV by removing leading apostrophes.

---

## Round-trip contract

| Export | Import | Result |
|---|---|---|
| JSON | **Review Imports** | Parsed as `HandoffManifest`; entries become the incoming and base manifest |
| CSV | **Review Imports** | Parsed per row; base translation text is reconstructed as empty because CSV has no base translation column |
| CSV edited in Excel/Sheets | **Review Imports** | Must be saved as CSV; quoted multiline values must survive the editor |

The CSV round-trip reconstructs a base manifest containing the entry ID, source text and base source hash. It does not know the original base translation text, so a concurrent-edit classification that depends on the base translation body is weaker for CSV than for JSON. Prefer JSON when conflict fidelity matters.

---

## A safe handoff checklist

- [ ] Choose one target locale per package.
- [ ] Confirm the release version is correct; it becomes the file name.
- [ ] Export JSON when the translator can handle it; CSV when they cannot.
- [ ] Tell the translator not to rename or translate the entry ID column.
- [ ] Tell them not to delete the base hash column.
- [ ] Tell them a leading apostrophe in a formula-looking cell is intentional.
- [ ] Keep the export file next to the returned file; the merge review needs both for full fidelity.
- [ ] Archive the package with the release evidence.

---

## Related pages

- [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/)
- [Translator Round-Trip Workflow]({{ site.baseurl }}/docs/workflows/translator-round-trip/)
- [Text Rules]({{ site.baseurl }}/docs/features/text-rules/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
