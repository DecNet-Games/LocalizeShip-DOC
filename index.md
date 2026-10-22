---
layout: default
title: Home
nav_order: 1
description: "LocalizeShip — the localization release and visual QA studio for Unity. Revision-bound approvals, real impact analysis, translator round-trips and release gates."
permalink: /
---

# LocalizeShip
{: .fs-9 }

### Localization Release & Visual QA Studio for Unity
{: .fs-6 .text-grey-dk-000 }

**Edit a source line once — and see every translation, voice take, subtitle and UI capture it touches, before you ship.**
{: .fs-5 }

[Get Started]({{ site.baseurl }}/docs/getting-started/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Installation]({{ site.baseurl }}/docs/installation/){: .btn .fs-5 .mb-4 .mb-md-0 }

![LocalizeShip cover artwork]({{ site.baseurl }}/assets/images/02-cover.png)

---

## The release is multilingual. The workflow is still a spreadsheet.

You already have a localization system. It stores strings and serves them at runtime. What it does **not** do is tell you, on a Tuesday afternoon three weeks before launch, that:

- the English quest objective changed after German was signed off, and the approval still says **Approved**;
- the Japanese voice take is bound to a sentence that no longer exists;
- the Arabic subtitle runs at 26 characters per second because the line was re-translated but never re-timed;
- the German settings label overflows its button by 41 pixels;
- the file you just imported from a freelancer silently overwrote a fix made in the project last week.

LocalizeShip is the missing release workspace between "the string table has values" and "this build is ready to ship". It sits on top of the localization system your team already uses, records what was reviewed against which source revision, and turns a multilingual release into something you can audit.

![LocalizeShip identity card]({{ site.baseurl }}/assets/images/01-card.png)

---

## Built for the questions release managers actually ask

| Question | What LocalizeShip gives you |
|---|---|
| **"Is German really approved?"** | A per-line, per-locale review state bound to a specific source revision hash — not a value in a table. |
| **"What does this edit break?"** | An impact report covering translations, voice takes, subtitle cues and capture fixtures tied to that line. |
| **"What do we send the translator?"** | A revision manifest in JSON and CSV, with base hashes, speaker/context, placeholder constraints and formula-injection escaping. |
| **"What changed while they were working?"** | A three-way merge plan with explicit `StaleSourceConflict` and `ConcurrentLocaleEdit` rows, plus a journal that supports rollback. |
| **"Does it actually fit on screen?"** | Measured layout rules for overflow, safe area, touch-target size and missing glyphs, attached to real UI fixtures. |
| **"Can we prove any of this?"** | Gate checks, HTML/JSON reports, approval audit events and a diagnostics bundle you can attach to a bug report. |

---

## The five-minute mental model

```
Source line (normalized text + speaker + context)
        │  SHA-256 short hash
        ▼
Source revision ──────► Approval record per locale
        │                 (Missing / Draft / NeedsReview / Approved / Stale / Rejected)
        │
        ├──► Voice take        bound to source + translation revision
        ├──► Subtitle cue      bound to source revision, checked for CPS
        └──► Capture fixture   measured for overflow, safe area, glyphs
                              │
                              ▼
                    Release gate checks + reports
```

Everything in LocalizeShip hangs off one idea: **review is not a property of a string; it is a property of a string at a revision.** When the revision changes, the review no longer applies. That single rule removes most of the "we thought it was approved" failures in a multilingual release.

Read the full model in [Core Concepts]({{ site.baseurl }}/docs/concepts/).

---

## What is in the box

### Release Board
A three-pane studio: workspace statistics and filters, the key × locale matrix, and a detail inspector with source hash, context, impact report, locale approvals and audit history.

![Release Board]({{ site.baseurl }}/assets/images/03-release-board.png)

[Explore the Release Board]({{ site.baseurl }}/docs/features/release-board/){: .btn .btn-outline }

### Revision-bound approvals and audit trail
Approvals are explicit events with reviewer label, role, rationale, timestamp and the exact source and translation hashes that were reviewed. A source change automatically moves only approved records to **Stale** and writes a system event explaining why.

[How approvals bind to revisions]({{ site.baseurl }}/docs/concepts/revision-graph/){: .btn .btn-outline }

### Impact engine
For any line, compute the affected translation cells, stale or missing voice takes, subtitle cues bound to an older revision, and capture fixtures that show the line. Each item says *why* it is impacted.

[Inside the impact engine]({{ site.baseurl }}/docs/features/impact-engine/){: .btn .btn-outline }

### Translator handoff and reviewed imports
Export a JSON manifest and an RFC 4180-style CSV with base source hashes, speaker/context and required placeholders. When the file comes back, review a three-way plan before anything is written. Imported text enters **Needs Review**, never **Approved**.

[Handoff]({{ site.baseurl }}/docs/features/translation-handoff/){: .btn .btn-outline } · [Merge imports]({{ site.baseurl }}/docs/features/merge-imports/){: .btn .btn-outline }

### Capture Matrix and visual QA rules
Register UI fixtures once, then evaluate every bound text element against overflow, safe-area, touch-target and glyph rules. Findings carry severities and can be suppressed by fingerprint with a recorded reason.

[Capture Matrix]({{ site.baseurl }}/docs/features/capture-matrix/){: .btn .btn-outline } · [Visual QA rules]({{ site.baseurl }}/docs/features/visual-qa/){: .btn .btn-outline }

### Voice and subtitle coverage
Voice takes store the source and translation revision they were recorded against. Subtitle cues are checked for invalid timing and reading speed. Both can be auditioned from the Editor.

[Voice and subtitles]({{ site.baseurl }}/docs/features/voice-and-subtitles/){: .btn .btn-outline }

### Release gates
Four deterministic checks — approval matrix, visual matrix, voice coverage and subtitle timing — produce **Passed**, **Failed** or **IncompleteCoverage**, with dark-themed HTML and machine-readable JSON reports.

[Release gates]({{ site.baseurl }}/docs/features/release-gates/){: .btn .btn-outline }

---

## Works with the provider you already have

![LocalizeShip on top of an existing provider]({{ site.baseurl }}/assets/images/15-existing-provider.png)

| Provider | How it connects | Verification status |
|---|---|---|
| **Unity Localization** `1.5.13` | Implemented editor adapter over `StringTableCollection`; key-based identity | Primary path; the adapter is implemented, but no shipped window calls it for import or write-back |
| **I2 Localization** | Compile-isolated module behind `LOCALIZESHIP_I2_SUPPORT` | Verified for clean compilation and provider-absent behaviour |
| **Pixel Crushers Dialogue System** | Compile-isolated module behind `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` | Verified for clean compilation and provider-absent behaviour |

{: .note }
> LocalizeShip does **not** replace runtime localization, and it does not redistribute third-party commercial source. The I2 and Dialogue System modules are not claimed as verified against licensed installs. See [Providers]({{ site.baseurl }}/docs/providers/) for the exact contract.

---

## Verified baseline

| Item | Value |
|---|---|
| Unity Editor | **6000.5.4f1** — developed and test-verified |
| Unity Localization | **1.5.13** (free Unity package) |
| Addressables | **1.25.0** (installed with Localization) |
| Acceptance tests | **12 EditMode suites, 12/12 passing** — independently re-run |
| Runtime network calls | None in the shipped core |
| Distribution | Commercial package on the **Unity Asset Store** only |

{: .important }
> LocalizeShip is a paid Unity Asset Store package. The GitHub repository is documentation and issue tracking — it is not a free download of the tool.

[Read the verification notes]({{ site.baseurl }}/docs/testing-and-verification/){: .btn .btn-outline }

---

## Start here

1. **[Getting Started]({{ site.baseurl }}/docs/getting-started/)** — the mental model, the menu map, and the first workspace.
2. **[Installation]({{ site.baseurl }}/docs/installation/)** — Asset Store import, Unity Localization dependency, optional adapters.
3. **[Quick Start]({{ site.baseurl }}/docs/quick-start/)** — open the RPG sample and run one complete release loop.
4. **[First Release Workflow]({{ site.baseurl }}/docs/workflows/first-release/)** — the same loop applied to your own project.
5. **[Limitations]({{ site.baseurl }}/docs/limitations/)** — read this before promising anything to your producer.

---

{: .warning }
> **Media note:** this site uses the 15 final marketing images from the LocalizeShip delivery. No demo video is published because no demo video file exists in the current delivery. Nothing on this site links to a video that is not real.
