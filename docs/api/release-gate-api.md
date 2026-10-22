---
layout: default
title: Release Gate API
parent: Architecture & API
nav_order: 4
description: "ReleaseGateEvaluator: policies, the four checks, verdict logic and HTML/JSON report generation."
permalink: /docs/api/release-gate-api/
---

# Release Gate API
{: .fs-9 }

### Deterministic evaluation in one call
{: .fs-6 .text-grey-dk-000 }

```csharp
var report = new ReleaseGateEvaluator().Evaluate(workspace, policy);
```

No editor window, no asset writes, no network. The evaluator reads the workspace and returns a serializable report.

---

## Signature

```csharp
public ReleaseGateReport Evaluate(
    ReleaseWorkspace workspace,
    GatePolicy policy = null)
```

A null policy becomes `new GatePolicy()` — all booleans `true`.

```csharp
public string GenerateHtmlReport(
    ReleaseWorkspace workspace, ReleaseGateReport report)

public string GenerateJsonReport(
    ReleaseWorkspace workspace, ReleaseGateReport report)
```

Both return strings. Writing files is the caller's job.

---

## Policy

```csharp
public class GatePolicy
{
    public bool RequireAllLocalesApproved = true;
    public bool BlockOnStaleTranslations = true;
    public bool BlockOnMissingTranslations = true;
    public bool BlockOnUnvisitedCaptures = true;
    public bool BlockOnVisualErrors = true;
    public bool BlockOnStaleVoiceClips = true;
    public bool BlockOnSubtitleTimingErrors = true;
    public bool BlockOnGlossaryViolations = true; // not read
}
```

To make a check non-blocking, set its boolean to `false`. Note that `unapprovedCount` is only consulted when `RequireAllLocalesApproved` is true; missing and stale have their own switches.

---

## The four checks

### Translations Approval Matrix

Iterates every `SourceRevisions × TargetLocales`:

- missing approval or `Missing` → missing;
- `Stale` → stale;
- any other non-`Approved` → unapproved.

Pass:

```csharp
(RequireAllLocalesApproved ? unapproved == 0 : true) &&
(BlockOnMissingTranslations ? missing == 0 : true) &&
(BlockOnStaleTranslations ? stale == 0 : true)
```

### Visual QA Matrix & Geometry

For each `CaptureTileResult`:

- coverage `Unknown` or `Registered` → unvisited;
- unsuppressed `Error` or `Fatal` finding → error.

Pass:

```csharp
(BlockOnUnvisitedCaptures ? unvisited == 0 : true) &&
(BlockOnVisualErrors ? errors == 0 : true)
```

### Voice Coverage & Revisions

Counts voice entries with `IsStale`. Pass:

```csharp
BlockOnStaleVoiceClips ? stale == 0 : true
```

### Subtitle Timing & Reading Speed

For each cue:

- invalid timing → violation;
- else CPS > `workspace.Subtitles.MaxCharactersPerSecond` → violation.

Pass:

```csharp
BlockOnSubtitleTimingErrors ? violations == 0 : true
```

---

## Verdict

```csharp
bool allPassed = approval.Passed && visual.Passed && voice.Passed && subtitle.Passed;
int defectFailures = visualErrors + staleTranslations + staleVoices + timingViolations;
int incompleteFailures = unvisitedTiles + missingTranslations;

if (allPassed) Passed;
else if (defectFailures > 0) Failed;
else if (incompleteFailures > 0 || unapprovedCount > 0) IncompleteCoverage;
else Failed;
```

The final `else` is unreachable in practice: if all checks passed the first branch wins; if not and there are no defects or incomplete counts, a check failed for another reason (for example unapproved cells with `RequireAllLocalesApproved = false` but a disabled blocking switch) and the report says `Failed`.

---

## Report shape

```csharp
public class ReleaseGateReport
{
    public string ReportId;        // GUID N
    public string ReleaseVersion;
    public GateStatus OverallStatus;
    public long TimestampTicks;    // UTC
    public List<GateCheckItem> Checks;
}
```

Exactly four checks are added, in this order:

1. `Translations Approval Matrix`
2. `Visual QA Matrix & Geometry`
3. `Voice Coverage & Revisions`
4. `Subtitle Timing & Reading Speed`

Each has `Passed`, a human-readable `Details` line, and a `Violations` list.

---

## HTML

`GenerateHtmlReport`:

- dark inline CSS;
- release version, source locale, UTC timestamp, verdict, passed/total;
- one card per check;
- violation text escaped with `SecurityElement.Escape`;
- no external assets or scripts.

It is safe to open offline. It is not a template system; branding is fixed.

## JSON

`GenerateJsonReport` is `JsonUtility.ToJson(report, true)`.

Remember:

- enums serialize as integers;
- computed properties (`TotalChecks`, `PassedChecks`) are absent;
- `TimestampTicks` is a long, not an ISO date.

Example consumer:

```csharp
var report = JsonUtility.FromJson<ReleaseGateReport>(json);
DateTime utc = new DateTime(report.TimestampTicks, DateTimeKind.Utc);
```

---

## Writing files

The Release Gate window writes next to the project:

```
LocalizeShip_Report_v<ReleaseVersion>.html
LocalizeShip_Report_v<ReleaseVersion>.json
```

Your own script can choose any path:

```csharp
var evaluator = new ReleaseGateEvaluator();
var report = evaluator.Evaluate(workspace);
File.WriteAllText("Build/gate.json", evaluator.GenerateJsonReport(workspace, report));
File.WriteAllText("Build/gate.html", evaluator.GenerateHtmlReport(workspace, report));
```

Both generators are public instance methods on the evaluator — useful for CI.

---

## Limits

- No text-rule, glossary, merge-state or duration-mismatch checks.
- No per-locale policy.
- No baseline comparison; see [Release Comparison]({{ site.baseurl }}/docs/features/release-comparison/).
- No player-build validation.
- `BlockOnGlossaryViolations` is inert.

---

## Related pages

- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
- [CI Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/)
- [Coverage Honesty]({{ site.baseurl }}/docs/concepts/coverage-honesty/)
- [Identifiers & Models]({{ site.baseurl }}/docs/api/identifiers-and-models/)
