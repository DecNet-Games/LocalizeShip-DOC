---
layout: default
title: Testing & Verification
nav_order: 11
description: "What the 12 EditMode acceptance tests actually assert on Unity 6000.5.4f1, and what remains outside the suite."
permalink: /docs/testing-and-verification/
---

# Testing & Verification
{: .fs-9 }

### Twelve tests, twelve passes, precise claims
{: .fs-6 .text-grey-dk-000 }

The package ships an EditMode acceptance suite. It is the evidence behind this documentation. This page describes what the tests do — and, just as importantly, what they do not cover.

---

## Verified baseline

| Component | Version |
|---|---|
| Unity Editor | **6000.5.4f1** |
| Unity Localization | **1.5.13** |
| Addressables | **1.25.0** |
| Test framework | Unity Test Framework (NUnit) |

No claim is made for Unity 2022.3, 2023, or other 6.x patch versions. Re-run the suite before assuming compatibility.

---

## Result

**12 EditMode test cases, 12 passed, 0 failed, 0 skipped.**

The suite is `Assets/Decnet/LocalizeShip/Tests/Editor/LocalizeShipAcceptanceTests.cs` in the source project. The bundled package may omit test sources; if so, the result is a build-time baseline, not something you can re-run from the package alone.

---

## What each test asserts

### LS-T01 — Stable line identities

Creates two revisions by key, verifies their hashes, reverses the `SourceRevisions` list, and verifies lookup by ID still returns the same hashes.

It does **not** test renaming a key or provider renumbering. The claim is “list order does not affect identity,” not “renaming is safe.”

### LS-T02 — Approval invalidation

Approves a German cell, changes the source text, and asserts:

- the hash changed;
- the approval state is `Stale`;
- history has at least two events;
- `ImpactEngine` reports at least one stale translation.

It does not assert that a non-approved cell changes state.

### LS-T03 — Stale-source conflict

Exports a base manifest, edits the source, submits a translator change, and asserts:

- one conflict;
- classification `StaleSourceConflict`;
- unresolved conflicts exist;
- `ApplyPlan` throws `InvalidOperationException`.

### LS-T04 — Import safety

- Protects and unprotects `=`, `+`, `-`, `@`, tab and CR prefixes.
- Round-trips Unicode, commas, quotes, multiline text and a formula-like source through CSV.
- Classifies whitespace-only incoming as `BlankIncomingValue`.
- Classifies a duplicate incoming ID as `DuplicateIdError`.

It does not test JSON round-trip or constraint preservation.

### LS-T05 — Provider absence

- `UnityLocalizationAdapter.IsInstalled` is true.
- I2 and Dialogue System adapters report false (flags off).
- Their `WriteTranslation` throws `NotSupportedException`.

It does not test the adapters with the optional packages installed.

### LS-T06 — Visual matrix

Builds a 250×50 element inside a 180×50 container with a safe-area rect offset by 10 px, marks it interactive with a missing glyph, and asserts exactly three findings: overflow, safe-area violation, missing glyph.

The touch-target check is not triggered because the element is 250 wide. The test does not prove touch-target detection.

### LS-T07 — Format semantics

- Valid reordered placeholders pass.
- Missing `{gems}` produces one `MissingPlaceholder`.
- Mismatched `<b>...</i>` produces at least one violation.
- Self-closing `<sprite=0>` and `<br>` pass.
- Unclosed `<b>` produces one `UnclosedRichTextTag`.

Despite the test description, the case uses simple `{count}` and `{gems}` placeholders, not nested ICU plurals.

### LS-T08 — Coverage honesty

- An `Unknown` capture tile yields `IncompleteCoverage`.
- A clean workspace with a visited tile containing one error finding yields `Failed`.

### LS-T09 — Audio and subtitles

- Binds a voice entry, edits the source, calls `RecheckStatus`, and asserts `StaleSourceRevision`.
- Creates a cue with 40 CPS and asserts it exceeds 18.

It does not test cue overlap, invalid timing, or the gate's subtitle path directly.

### LS-T10 — Advisory rejection

Rejects malformed JSON, mismatched ID, missing placeholder and omitted ID; accepts a valid payload. It does not perform network calls, rate-limit handling or prompt-injection detection despite older descriptions.

### LS-T11 — Journal rollback

Applies a plan, verifies text/state, rolls back, verifies restoration, applies again, performs a later edit, and confirms:

- rollback throws without `force`;
- one restore conflict is recorded;
- forced rollback restores the previous text.

The journal is an in-memory object; the test does not simulate an assembly reload or editor restart.

### LS-T12 — End-to-end report

Builds an in-memory workspace with one line approved in three locales and three visited tiles, evaluates the gate, and asserts:

- verdict `Passed`;
- HTML contains `Verdict:` and `badge-pass`;
- JSON contains `OverallStatus`.

It does not load the RPG sample scene, run captures, or exercise the handoff window.

---

## What the suite does not prove

- Provider adapters work with licensed I2 or Dialogue System installs.
- Player-build rendering.
- Arabic shaping, RTL layout or linguistic quality.
- Screenshot fidelity (the PNG is a solid colour).
- Performance at production table sizes.
- Persistence of rollback journals across sessions.
- JSON import three-way behaviour.
- Glossary UI or gate enforcement.
- Headless CLI behaviour.

Do not cite 12/12 as evidence for any of those.

---

## Running the suite

### Editor

1. **Window → General → Test Runner**.
2. Select **EditMode**.
3. Filter `LocalizeShipAcceptanceTests`.
4. **Run All**.

Expect 12 green tests.

### Batch mode

```bat
Unity.exe -batchmode -nographics -projectPath . ^
  -runTests -testPlatform EditMode ^
  -testResults testresults.xml ^
  -logFile test.log -quit
```

Parse `testresults.xml` for `passed="12" failed="0"`.

---

## Writing your own acceptance tests

Test the contracts that matter to your integration:

- your adapter produces deterministic hashes;
- a source change stales exactly the approvals you expect;
- your gate policy produces the verdict you intend;
- your CI entry point exits non-zero on failure.

The package's suite is a starting point, not a substitute for your project's tests.

---

## Related pages

- [CI Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
- [Installation]({{ site.baseurl }}/docs/installation/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
