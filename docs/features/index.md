---
layout: default
title: Features
nav_order: 6
has_children: true
description: "The LocalizeShip feature set: release board, impact engine, handoff, merge review, capture matrix, visual QA, voice, subtitles and gates."
permalink: /docs/features/
---

# Features
{: .fs-9 }

### The release toolchain, module by module
{: .fs-6 .text-grey-dk-000 }

Each page in this section covers one module: what it does, where it lives, how it decides, and what it does not do. The status column matters — some capabilities are full editor windows, some are domain services you call from code, and a few are deliberately narrow in 1.0.0.

![Release Board]({{ site.baseurl }}/assets/images/03-release-board.png)

---

## Feature matrix

| Module | Surface | Status in 1.0.0 |
|---|---|---|
| [**Release Board**]({{ site.baseurl }}/docs/features/release-board/) | Window | Full review matrix, filters, approvals, audit, diagnostics |
| [**Guided Setup**]({{ site.baseurl }}/docs/features/guided-setup/) | Window | Creates a workspace; does not import tables |
| [**Impact Engine**]({{ site.baseurl }}/docs/features/impact-engine/) | Service | Full for translations, voice, subtitles, captures |
| [**Translation Handoff**]({{ site.baseurl }}/docs/features/translation-handoff/) | Window | JSON + CSV export with formula protection |
| [**Merge Imports**]({{ site.baseurl }}/docs/features/merge-imports/) | Window | Three-way plan, conflicts, journal, rollback |
| [**Capture Matrix**]({{ site.baseurl }}/docs/features/capture-matrix/) | Window + runtime | Editor-side fixture evaluation |
| [**Visual QA**]({{ site.baseurl }}/docs/features/visual-qa/) | Service | Overflow, safe area, touch target, glyphs |
| [**Text Rules**]({{ site.baseurl }}/docs/features/text-rules/) | Service | Placeholders, braces, rich text, glossary |
| [**Voice & Subtitles**]({{ site.baseurl }}/docs/features/voice-and-subtitles/) | Window | Coverage, audition, CPS, timing |
| [**Glossary**]({{ site.baseurl }}/docs/features/glossary/) | Model + service | Validation API; no dedicated editor window |
| [**Release Gates**]({{ site.baseurl }}/docs/features/release-gates/) | Window + service | Four checks; HTML + JSON reports |
| [**Release Comparison**]({{ site.baseurl }}/docs/features/release-comparison/) | Workflow | Compare exported reports; no diff window |
| [**Advisory Drafts**]({{ site.baseurl }}/docs/features/advisory-drafts/) | Service | Validation guardrails; no outbound HTTP client |
| [**Documentation Viewer**]({{ site.baseurl }}/docs/features/documentation-viewer/) | Window | Opens the ten bundled offline manuals |

---

## The pipeline

```
Provider tables
      │  adapter read
      ▼
RevisionTracker ──► ReleaseWorkspace ──► Release Board
      │                                   │
      │                                   ├── Translation Handoff ──► translator
      │                                   │                              │
      │                                   │                              ▼
      │                                   │                        Review Imports
      │                                   │                              │
      │                                   ▼                              ▼
      │                            Capture Matrix                 Merge plan + journal
      │                                   │
      ▼                                   ▼
ImpactEngine ◄──────────────────── Visual QA findings
      │
      ├── Voice & Subtitles
      └── Release Gates ──► HTML / JSON evidence
```

---

## Two rules that apply to every module

1. **Nothing auto-approves.** Not an import, not an advisory draft, not a batch operation you did not click.
2. **Coverage is explicit.** A tile that was not visited is `Unknown`, not `Passed`.

---

## Where to start

- New to the tool: [Release Board]({{ site.baseurl }}/docs/features/release-board/)
- Translator pipeline: [Translation Handoff]({{ site.baseurl }}/docs/features/translation-handoff/) → [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/)
- UI quality: [Capture Matrix]({{ site.baseurl }}/docs/features/capture-matrix/) → [Visual QA]({{ site.baseurl }}/docs/features/visual-qa/)
- Release sign-off: [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
