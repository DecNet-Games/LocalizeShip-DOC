---
layout: default
title: Identifiers & Models
parent: Architecture & API
nav_order: 1
description: "StableEntryId, hash structs, LocaleId, ReleaseWorkspace, revisions, approvals, findings and reports."
permalink: /docs/api/identifiers-and-models/
---

# Identifiers & Models
{: .fs-9 }

### Value types at the boundary, serializable classes inside
{: .fs-6 .text-grey-dk-000 }

---

## Identifiers

All four are `[Serializable]` structs with a public string field, value equality and implicit string conversions.

```csharp
public struct StableEntryId        { public string Value; }
public struct SourceRevisionHash   { public string Hash; }
public struct TranslationRevisionHash { public string Hash; }
public struct LocaleId             { public string Code; }
```

| Type | Constructor behaviour | Equality |
|---|---|---|
| `StableEntryId` | null → `""` | `StringComparison.Ordinal` |
| `SourceRevisionHash` | null → `""` | Ordinal |
| `TranslationRevisionHash` | null → `""` | Ordinal |
| `LocaleId` | trims and lowercases; null → `""` | `OrdinalIgnoreCase` |

Each has `IsValid` (non-empty), `==`, `!=`, `GetHashCode`, `ToString`, and implicit conversions both ways.

```csharp
StableEntryId id = "ui.play";
LocaleId de = "DE";              // stored as "de"
bool same = de == new LocaleId("de");
```

Because conversion is implicit, a string can slip into an API that expects an ID. The type system will not catch a locale passed where an entry ID belongs if both are strings at the call site.

---

## Hash construction

`NormalizationPolicy.ComputeHash(normalizedText, context)`:

```csharp
string payload = $"{Version}|{normalizedText}|{context ?? ""}";
byte[] bytes = SHA256(payload);
// first 8 bytes -> 16 lowercase hex chars
```

`SourceRevision` passes `context = $"{Speaker}:{ContextDescription}"`. So the same text with a different speaker name has a different revision hash.

Translation hashes use `ComputeTranslationHash`, which passes the literal context `"translation"`.

---

## ReleaseWorkspace

```csharp
[CreateAssetMenu(menuName = "Decnet/LocalizeShip/Release Workspace")]
public class ReleaseWorkspace : ScriptableObject
{
    public string WorkspaceId;
    public string ReleaseVersion;
    public string SourceLocale;
    public List<string> TargetLocales;

    public NormalizationPolicy Normalization;
    public SubtitlePolicy Subtitles;

    public List<SourceRevision> SourceRevisions;
    public List<ApprovalRecord> Approvals;
    public List<VoiceEntry> VoiceEntries;
    public List<SubtitleCue> SubtitleCues;
    public List<GlossaryTerm> Glossary;
    public List<CaptureTileResult> CaptureResults;
    public List<FindingSuppression> Suppressions;

    public string ActiveProviderType;
    public List<string> BoundCollectionGuids;
}
```

Helpers:

```csharp
SourceRevision GetSourceRevision(StableEntryId entryId);
ApprovalRecord GetApproval(StableEntryId entryId, LocaleId locale);
ApprovalRecord GetOrCreateApproval(StableEntryId entryId, LocaleId locale, string text);
```

All three are linear scans. A 5,000-line workspace × 3 locales means 15,000 approval lookups per full matrix render; the Release Board calls these in loops. Performance is adequate for the tested sizes, not for very large tables.

A new `ApprovalRecord` is `Missing` if the text is empty, otherwise `NeedsReview`.

---

## SourceRevision

```csharp
public class SourceRevision
{
    public StableEntryId EntryId;
    public SourceRevisionHash RevisionHash;
    public string RawText;
    public string NormalizedText;
    public string Speaker;
    public string ContextDescription;
    public string SceneOrFixture;
    public int NormalizationVersion;
    public long TimestampTicks;
    public List<string> Placeholders;
}
```

The constructor normalizes, computes the hash and calls `ExtractPlaceholders`. It sets `TimestampTicks` to `DateTime.UtcNow.Ticks`, so two otherwise identical constructions produce different timestamps but the same hash.

`ExtractPlaceholders` uses `\{([^{}]+)\}` and stores the trimmed inner text, de-duplicated. Nested braces are not handled as single expressions.

`SceneOrFixture` is set by `RevisionTracker.ProcessSourceEntry(..., fixture)` and is used as: a fixture name for impact matching, and a “relative screenshot path” in handoff exports. Neither is a screenshot path.

---

## ApprovalRecord & ApprovalEvent

```csharp
public class ApprovalRecord
{
    public StableEntryId EntryId;
    public LocaleId Locale;
    public ReviewState CurrentState;
    public SourceRevisionHash ApprovedSourceHash;
    public TranslationRevisionHash ApprovedTranslationHash;
    public string TranslationText;
    public List<ApprovalEvent> History;
}
```

`ApplyReview(newState, sourceHash, transHash, reviewer, role, rationale)` appends an event and sets state. On `Approved` it stores both approved hashes.

`CheckSourceChange(newHash)`:

- only acts when state is `Approved` and the approved source hash differs;
- sets state to `Stale`;
- appends a system event with reviewer `"System (Auto-Invalidation)"` and role `"Engine"`.

`ApprovalEvent` records old/new state, bound source and translation hashes, reviewer label/role, rationale and UTC ticks. `ReviewerLabel` defaults to `"Local Reviewer"` and role to `"Localization Lead"` when empty.

```csharp
public enum ReviewState
{
    Missing, NeedsReview, Approved, Rejected, Stale
}
```

---

## Voice and subtitle models

```csharp
public enum AudioMatchStatus
{
    Missing, BoundCurrent, StaleSourceRevision,
    StaleTranslationRevision, DurationMismatch
}

public class VoiceEntry
{
    public string VoiceId;
    public StableEntryId EntryId;
    public LocaleId Locale;
    public string ActorName;
    public int TakeNumber;
    public SourceRevisionHash BoundSourceHash;
    public TranslationRevisionHash BoundTranslationHash;
    public string AudioAssetPath;
    public float DurationSeconds;
    public float ExpectedDurationSeconds;
    public AudioMatchStatus Status;
}
```

`RecheckStatus(sourceHash, translationHash)` applies the ordered rules and sets `Status`.

```csharp
public class SubtitlePolicy
{
    public float MaxCharactersPerSecond = 18f;
    public float MinCueDurationSeconds = 1f;
    public float MaxCueDurationSeconds = 7f;
    public int MaxCharactersPerLine = 42;
}

public class SubtitleCue
{
    public string CueId;
    public StableEntryId EntryId;
    public LocaleId Locale;
    public float StartTimeSeconds;
    public float EndTimeSeconds;
    public string Text;
    public SourceRevisionHash BoundSourceHash;
}
```

`CharactersPerSecond` uses `Text.Length` (UTF-16 code units). `HasInvalidTiming` is `start < 0 || end <= start`.

---

## Visual models

```csharp
public enum DefectSeverity { Info, Warning, Error, Fatal }
public enum DefectType
{
    TextOverflow, Clipping, SafeAreaViolation,
    TouchTargetTooSmall, MissingGlyph, LayoutCollision
}
public enum FixtureCoverageStatus { Unknown, Registered, Visited, Failed }
```

`VisualFinding` carries the entry, locale, object name, element/container rects, description, missing glyph details, screenshot path and suppression flags. `GetFingerprint()` is:

```csharp
$"{Type}_{EntryId}_{Locale}_{TargetGameObjectName}_{Description}"
```

Changing the description changes the fingerprint and invalidates a suppression.

`CaptureTileResult` carries fixture ID, locale, resolution, `IsPlayerCapture`, screenshot path, coverage, findings and timestamp. `HasErrors` is true for any unsuppressed `Error` or `Fatal`.

---

## Gate models

```csharp
public enum GateStatus { Passed, Failed, IncompleteCoverage }

public class GateCheckItem
{
    public string CheckName;
    public bool Passed;
    public string Details;
    public List<string> Violations;
}

public class ReleaseGateReport
{
    public string ReportId;
    public string ReleaseVersion;
    public GateStatus OverallStatus;
    public long TimestampTicks;
    public List<GateCheckItem> Checks;
    public int TotalChecks;   // computed
    public int PassedChecks;  // computed
}
```

`GatePolicy` has eight booleans, all default `true`. Seven are read; `BlockOnGlossaryViolations` is not.

---

## Serialization notes

- `JsonUtility` serializes public fields, not properties. Computed properties such as `TotalChecks` do not appear in JSON.
- Enums serialize as integers (`GateStatus.Passed` → `0`).
- `JsonUtility` does not serialize `null` strings as `null` on read; missing strings become `""` in many cases. Validate your inputs.
- The workspace asset is Unity YAML. Hand-editing it is possible but bypasses hash invalidation.

---

## Related pages

- [Domain Services]({{ site.baseurl }}/docs/api/domain-services/)
- [Stable Identities]({{ site.baseurl }}/docs/concepts/stable-identities/)
- [Review States]({{ site.baseurl }}/docs/concepts/review-states/)
- [Normalization]({{ site.baseurl }}/docs/concepts/normalization/)
