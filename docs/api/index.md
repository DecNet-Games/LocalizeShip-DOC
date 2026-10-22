---
layout: default
title: Architecture & API
nav_order: 10
has_children: true
description: "Assembly layout and the public C# surface of LocalizeShip 1.0.0."
permalink: /docs/api/
---

# Architecture & API
{: .fs-9 }

### The domain is plain C#; the editor is a client
{: .fs-6 .text-grey-dk-000 }

LocalizeShip separates data and rules from windows. You can evaluate a release, validate text or plan a merge without opening a single editor window.

---

## Assembly layout

```
Assets/Decnet/LocalizeShip/
├── Core/      Decnet.LocalizeShip.Core       (all platforms)
├── Runtime/   Decnet.LocalizeShip.Runtime    (all platforms)
├── Adapters/  Decnet.LocalizeShip.Adapters   (Editor only)
├── Editor/    Decnet.LocalizeShip.Editor     (Editor only)
├── Tests/     Decnet.LocalizeShip.Tests.Editor (Editor only, UNITY_INCLUDE_TESTS)
├── Samples/   RPGSample
└── Documentation/  bundled offline manual
```

| Assembly | References | Contains |
|---|---|---|
| Core | none | Identifiers, models, `RevisionTracker`, `ImpactEngine`, `MergePlanner`, `ApplyCoordinator`, `TextRuleEngine`, `VisualRuleEngine`, `ReleaseGateEvaluator`, `HandoffCsvSerializer`, `AdvisoryProvider` |
| Runtime | Core, TMP, uGUI | `CaptureFixture`, `CaptureRunner`, `CaptureTileResult` glue |
| Adapters | Core, Unity Localization, Addressables | `ILocalizationProvider` and three adapters |
| Editor | Core, Adapters, Runtime, Unity Localization, Addressables, TMP, uGUI | Windows, menus, provider discovery, diagnostics, source navigation |
| Tests | Core, Adapters, Runtime, Editor, Test Runner, Unity Localization | 12 EditMode acceptance tests |

The adapters assembly hard-references Unity Localization, so the package requires it. There is no separate “no localization package” build.

---

## What is public

Everything under `Decnet.LocalizeShip.Core` is a normal public C# API. The editor windows are public classes with static `Open` methods, but their internals are not a stable API.

| Page | Covers |
|---|---|
| [Identifiers & Models]({{ site.baseurl }}/docs/api/identifiers-and-models/) | ID structs, workspace, revisions, approvals, findings, reports |
| [Domain Services]({{ site.baseurl }}/docs/api/domain-services/) | Tracking, impact, merge, apply, text rules, visual rules |
| [Capture Runner]({{ site.baseurl }}/docs/api/capture-runner-api/) | `CaptureFixture` and `CaptureRunner.ExecuteFixture` |
| [Release Gate API]({{ site.baseurl }}/docs/api/release-gate-api/) | `ReleaseGateEvaluator`, policy, report generation |
| [SourceRevision]({{ site.baseurl }}/docs/api/source-revision/) | Normalization, hashing, context and placeholder extraction |
| [ImpactEngine]({{ site.baseurl }}/docs/api/impact-engine/) | Cascade computation and emitted impact categories |
| [HandoffManifest]({{ site.baseurl }}/docs/api/handoff-manifest/) | Portable export schema and stable entry payloads |
| [MergePlanner]({{ site.baseurl }}/docs/api/merge-planner/) | Three-way classifications and formula protection |
| [TextRuleEngine]({{ site.baseurl }}/docs/api/text-rule-engine/) | Placeholder, rich-text and glossary validation |
| [VisualRuleEngine]({{ site.baseurl }}/docs/api/visual-rule-engine/) | Geometry, glyph and suppression evaluation |

---

## Design rules you can rely on

- IDs are value structs with string backing and ordinal comparison.
- Hashing is SHA-256 over policy version, normalized text and context, truncated to 8 bytes (16 hex chars).
- Review state transitions go through `ApprovalRecord.ApplyReview`.
- A changed source hash invalidates `Approved` records.
- Imported text always lands in `NeedsReview`.
- Reports are serializable with `UnityEngine.JsonUtility`.

---

## What is not public API

- Window layout and private fields.
- The exact wording of violation messages (treat as human-readable, not parseable).
- Serialized YAML field order in the workspace asset.
- Internal reflection calls (audio preview).
- The bundled sample's data, which is illustrative.

---

## Compatibility

- Unity 6000.5.4f1 (verified).
- Unity Localization 1.5.13 (verified).
- Addressables 1.25.0 (referenced; no direct feature use beyond assembly references).
- C# as shipped by Unity 6.

No claim is made for older Unity versions.

---

## Related pages

- [Testing & Verification]({{ site.baseurl }}/docs/testing-and-verification/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
- [Menu Reference]({{ site.baseurl }}/docs/menu-reference/)
