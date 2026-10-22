---
layout: default
title: Domain Services
parent: Architecture & API
nav_order: 2
description: "RevisionTracker, ImpactEngine, MergePlanner, ApplyCoordinator, TextRuleEngine, VisualRuleEngine and AdvisoryProvider."
permalink: /docs/api/domain-services/
---

# Domain Services
{: .fs-9 }

### The rules, without the windows
{: .fs-6 .text-grey-dk-000 }

These classes live in `Decnet.LocalizeShip.Core.Services`. They have no editor dependencies and can be called from tests, scripts or your own tooling.

---

## RevisionTracker

```csharp
public RevisionTracker(NormalizationPolicy policy = null)

SourceRevision ProcessSourceEntry(
    ReleaseWorkspace workspace,
    StableEntryId entryId,
    string rawText,
    string speaker,
    string context,
    string fixture = null)

TranslationRevisionHash ComputeTranslationHash(string translationText)
```

Behaviour:

1. Builds a new `SourceRevision` with the tracker's policy and optional `SceneOrFixture`.
2. If no current revision: adds it and calls `CheckSourceChange` on all approvals for the entry.
3. If the hash differs: removes the old revision, adds the new one, stales approved records, and rechecks voice entries using the approval's `ApprovedTranslationHash` (or empty).
4. If the hash is identical: returns the new object but does not replace the stored revision or touch states.

Note: it compares hashes, so a timestamp-only change does nothing.

{: .warning }
> `RevisionTracker` owns its own `NormalizationPolicy` (default or constructor-supplied). Pass `workspace.Normalization` if you want it to match the workspace. Constructing `new RevisionTracker()` while the workspace uses a custom policy can produce hashes the rest of the workspace does not expect.

---

## ImpactEngine

```csharp
public ImpactReport ComputeImpact(ReleaseWorkspace workspace, StableEntryId entryId)
```

`ImpactReport` contains `ChangedEntryId`, `TriggerSourceHash` and a list of `ImpactedArtifact`:

```csharp
public class ImpactedArtifact
{
    public ImpactCategory Category;
    public StableEntryId EntryId;
    public LocaleId Locale;
    public string Title;
    public string Reason;
    public string Snippet;
    public string TargetAssetPath;
    public string TargetId;
}
```

Categories emitted:

| Category | Condition |
|---|---|
| `ChangedSource` | A source revision exists for the entry |
| `StaleTranslation` | Approval is `Stale`, `Missing`, or absent |
| `VoiceReRecord` | Voice entry is stale or missing |
| `SubtitleReview` | Cue's `BoundSourceHash` differs from the current source hash |
| `VisualRecheck` | A capture finding references the entry, or the tile fixture ID equals `SceneOrFixture` |

It does not check review states other than stale/missing, does not verify voice durations, and does not know about provider assets. Capture matching by fixture ID is string equality, case-insensitive.

---

## MergePlanner

```csharp
public MergePlan CreatePlan(
    ReleaseWorkspace workspace,
    HandoffManifest baseManifest,
    List<HandoffEntry> incomingEntries,
    LocaleId targetLocale)

public static string ProtectFormula(string text)
public static string UnprotectFormula(string text)
```

Classification order (first match wins):

1. duplicate incoming ID → `DuplicateIdError`;
2. no source revision and no approval → `UnknownIdError`;
3. blank incoming text → `BlankIncomingValue`;
4. base source hash ≠ current source hash → `StaleSourceConflict`;
5. current translation differs from base translation **and** from incoming → `ConcurrentLocaleEdit`;
6. current equals incoming → `Unchanged`;
7. otherwise → `SafeIncomingChange`, resolved to incoming, resolution `TakeIncoming`.

Rows for other locales are skipped. The base lookup only includes entries whose `TargetLocale == targetLocale`.

`ProtectFormula` prefixes `=`, `+`, `-`, `@`, tab or CR with `'`. `UnprotectFormula` removes it only when the next character is one of those.

---

## ApplyCoordinator

```csharp
ImportJournal ApplyPlan(ReleaseWorkspace workspace, MergePlan plan, string reviewerName)
void Rollback(ReleaseWorkspace workspace, ImportJournal journal, bool force = false)
```

`ApplyPlan`:

- throws if `plan.HasUnresolvedConflicts`;
- skips `IsBlockingError` and `Unchanged` rows;
- computes the value as `ResolvedValue`, else incoming for `TakeIncoming`, else current;
- records a backup for each applied row;
- `KeepCurrent` records a backup then leaves text and state untouched;
- applies the text, then `ApplyReview(NeedsReview, currentSourceHash, default, reviewerName, "Translator Import", ...)`.

`Rollback`:

- refuses if any row changed after apply, unless `force`;
- restores text, state and `ApprovedSourceHash`;
- does not restore `ApprovedTranslationHash`;
- does not remove approval events;
- sets `IsRolledBack`.

Neither method calls `EditorUtility.SetDirty` or `AssetDatabase.SaveAssets`; the window does that after apply.

---

## TextRuleEngine

```csharp
List<TextRuleViolation> ValidatePlaceholders(
    SourceRevision source, string translationText, LocaleId locale)

List<GlossaryViolation> CheckGlossary(
    string text, StableEntryId entryId, LocaleId locale, List<GlossaryTerm> terms)

static string GeneratePseudoText(
    string originalText, float expansionFactor = 1.35f, bool rtlSimulation = false)
```

`ValidatePlaceholders` checks missing/unexpected placeholders, brace balance and rich-text tag pairing. Source and translation are compared by tag string; plural arguments are reduced to the token before the first comma.

`CheckGlossary` searches forbidden variants only.

`GeneratePseudoText` accents vowels, pads for expansion, wraps in `[!! ... !!]` and optionally reverses with bidi marks.

---

## VisualRuleEngine

```csharp
List<VisualFinding> EvaluateElement(
    StableEntryId entryId,
    LocaleId locale,
    string objectName,
    Rect elementRect,
    Rect containerRect,
    Rect screenSafeArea,
    bool isInteractive = false,
    string renderedText = null,
    List<char> missingGlyphs = null,
    List<FindingSuppression> suppressions = null)
```

Emits:

| Finding | Severity | Rule |
|---|---|---|
| `TextOverflow` | Error | element width/height exceeds container by more than 1 px |
| `SafeAreaViolation` | Warning | element rect outside the safe-area rect |
| `TouchTargetTooSmall` | Warning | interactive element under 44×44 |
| `MissingGlyph` | Error | any missing glyph supplied |

Suppression is applied by exact fingerprint match.

{: .note }
> The safe-area comparison uses the rects as supplied. `CaptureRunner` passes `Screen.safeArea` (screen space) and local RectTransform rects, which are not the same coordinate space. Confirm results on your canvas setup rather than assuming geometric precision.

---

## AdvisoryProvider

```csharp
DraftProposal ValidateSuggestion(
    SourceRevision source, string rawSuggestionJson, LocaleId targetLocale)
```

Parses `{ id, suggestion, rationale }`, requires the ID to equal the source entry ID, and runs placeholder validation. Any `Error`/`Fatal` makes the proposal invalid. It performs no network I/O and never approves text.

---

## HandoffCsvSerializer

```csharp
static string SerializeCsv(HandoffManifest manifest)
static HandoffManifest DeserializeCsv(string csvText, LocaleId targetLocale)
static List<List<string>> ParseCsvRows(string csvText)
```

The parser handles quoted fields, doubled quotes, CRLF and LF. Serialization protects formulas; deserialization unprotects. The CSV loses constraints and base translation text by design.

---

## Related pages

- [Identifiers & Models]({{ site.baseurl }}/docs/api/identifiers-and-models/)
- [Capture Runner API]({{ site.baseurl }}/docs/api/capture-runner-api/)
- [Release Gate API]({{ site.baseurl }}/docs/api/release-gate-api/)
- [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/)
