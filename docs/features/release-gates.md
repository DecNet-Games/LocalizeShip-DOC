---
layout: default
title: Release Gates
parent: Features
nav_order: 11
description: "The four deterministic gate checks, GatePolicy booleans, Passed/Failed/IncompleteCoverage logic, and HTML/JSON report export."
permalink: /docs/features/release-gates/
---

# Release Gates
{: .fs-9 }

### A report that is allowed to say no
{: .fs-6 .text-grey-dk-000 }

**Release Board → Release Gates** (there is no separate menu item)

A release gate is only useful if it can fail. LocalizeShip evaluates four checks against the workspace and returns one of three verdicts, then writes evidence you can attach to a build.

![Release gates]({{ site.baseurl }}/assets/images/14-release-gates.png)

---

## The four checks

### 1. Translations Approval Matrix

Walks every source revision × target locale:

| Condition | Counter | Violation text |
|---|---|---|
| No approval or `Missing` | `missingCount` | “Entry '…' missing translation for [DE].” |
| `Stale` | `staleCount` | “Entry '…' in [DE] is Stale (source revision updated).” |
| Any other non-approved state | `unapprovedCount` | “Entry '…' in [DE] is in state NeedsReview.” |

Pass condition:

```csharp
(policy.RequireAllLocalesApproved ? unapprovedCount == 0 : true) &&
(policy.BlockOnMissingTranslations ? missingCount == 0 : true) &&
(policy.BlockOnStaleTranslations ? staleCount == 0 : true)
```

Details line: total cells, unapproved, stale, missing.

### 2. Visual QA Matrix & Geometry

Walks every capture tile:

- `Coverage == Unknown` or `Registered` → unvisited;
- any unsuppressed finding with severity `Error` or `Fatal` → error finding.

Pass condition:

```csharp
(policy.BlockOnUnvisitedCaptures ? unvisitedTiles == 0 : true) &&
(policy.BlockOnVisualErrors ? errorFindings == 0 : true)
```

{: .important }
> A tile with `Coverage == Failed` is **not** counted as unvisited and may have no findings, so it can effectively pass this check. Track failed captures separately.

### 3. Voice Coverage & Revisions

Counts voice entries where `IsStale` is true (stale source or stale translation revision). Pass condition is `BlockOnStaleVoiceClips ? staleVoices == 0 : true`.

It does not count `Missing` or `DurationMismatch`.

### 4. Subtitle Timing & Reading Speed

For each cue:

- `HasInvalidTiming` (`start < 0` or `end <= start`) → violation;
- otherwise `CharactersPerSecond > workspace.Subtitles.MaxCharactersPerSecond` → violation.

Pass condition is `BlockOnSubtitleTimingErrors ? timingViolations == 0 : true`.

It does not enforce min/max duration or characters per line.

---

## Overall verdict

```csharp
bool allPassed = approvalCheck.Passed && visualCheck.Passed &&
                 voiceCheck.Passed && subCheck.Passed;

int defectFailures = errorFindings + staleCount + staleVoices + timingViolations;
int incompleteFailures = unvisitedTiles + missingCount;

if (allPassed) report.OverallStatus = GateStatus.Passed;
else if (defectFailures > 0) report.OverallStatus = GateStatus.Failed;
else if (incompleteFailures > 0 || unapprovedCount > 0)
    report.OverallStatus = GateStatus.IncompleteCoverage;
else report.OverallStatus = GateStatus.Failed;
```

```csharp
public enum GateStatus { Passed, Failed, IncompleteCoverage }
```

| Verdict | Practical meaning |
|---|---|
| **Passed** | All four checks passed under the active policy |
| **Failed** | At least one defect exists (visual error, stale translation, stale voice, subtitle timing/CPS) |
| **IncompleteCoverage** | No defects, but coverage is incomplete (unvisited tiles or missing translations), or there are unapproved cells |

A gate that says **IncompleteCoverage** is not a soft pass. It means “we have not looked at everything.”

---

## GatePolicy

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
    public bool BlockOnGlossaryViolations = true;
}
```

All default to `true`.

{: .warning }
> The Release Gate window calls `Evaluate(workspace)` without passing a policy, so every run uses the defaults. The booleans are configurable through the API, not through the window. `BlockOnGlossaryViolations` is not read by any check in 1.0.0.

Example of a custom policy from code:

```csharp
var policy = new GatePolicy
{
    RequireAllLocalesApproved = true,
    BlockOnMissingTranslations = true,
    BlockOnStaleTranslations = true,
    BlockOnUnvisitedCaptures = true,
    BlockOnVisualErrors = true,
    BlockOnStaleVoiceClips = false,
    BlockOnSubtitleTimingErrors = true
};

var report = new ReleaseGateEvaluator().Evaluate(workspace, policy);
```

---

## Reports

### HTML

`GenerateHtmlReport` produces a self-contained dark-themed page with:

- release version and UTC generation timestamp;
- source locale;
- overall verdict and passed/total check count;
- one card per check with PASS/FAIL, details and an escaped violation list.

Violation text is escaped with `SecurityElement.Escape`. The HTML contains no external assets.

### JSON

`GenerateJsonReport` calls `UnityEngine.JsonUtility.ToJson(report, true)`. It serializes the `ReleaseGateReport`:

```json
{
  "ReportId": "...",
  "ReleaseVersion": "1.0.0",
  "OverallStatus": 0,
  "TimestampTicks": 638965000000000000,
  "Checks": [
    {
      "CheckName": "Translations Approval Matrix",
      "Passed": false,
      "Details": "...",
      "Violations": ["..."]
    }
  ]
}
```

`OverallStatus` is serialized as an integer (`Passed = 0`, `Failed = 1`, `IncompleteCoverage = 2`). Parse it with that in mind.

### File locations

Both export buttons write next to the project folder:

```
LocalizeShip_Report_v<ReleaseVersion>.html
LocalizeShip_Report_v<ReleaseVersion>.json
```

They reveal the file in the OS browser after writing.

---

## What the gate does not check

- placeholder or rich-text validity;
- glossary violations;
- merge conflicts or unreviewed imports;
- voice duration mismatch or missing audio;
- subtitle min/max duration or characters per line;
- visual warnings (only errors and fatals);
- player-build rendering;
- linguistic or RTL correctness;
- whether a provider table was actually written after approval.

If you need these as blockers, extend `ReleaseGateEvaluator` or run your own checks alongside it. The JSON report is easy to post-process.

---

## CI reality

There is no shipped headless gate CLI. The evaluator is a plain C# class in the editor assembly, and the report writer lives in the editor window. You can:

- run the **12 EditMode acceptance tests** in batch mode with the included `run_tests.ps1` pattern;
- call `ReleaseGateEvaluator` from a custom editor script executed with `-executeMethod`;
- archive the JSON/HTML produced by that script.

Do not describe the included gate window as a CI tool. See [CI Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/) for an honest recipe.

---

## Reading a report in a release meeting

1. Read the verdict.
2. Read `PassedChecks / TotalChecks`.
3. Read the **Details** line of each failed check before the violation list; it tells you the scale.
4. Separate defects (Failed) from coverage gaps (IncompleteCoverage).
5. Assign owners by category: translations, visual, voice, subtitles.
6. Re-run after fixes and archive both reports if the verdict changed.

---

## Related pages

- [Release Board]({{ site.baseurl }}/docs/features/release-board/)
- [Coverage Honesty]({{ site.baseurl }}/docs/concepts/coverage-honesty/)
- [CI Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/)
- [Release Comparison]({{ site.baseurl }}/docs/features/release-comparison/)
