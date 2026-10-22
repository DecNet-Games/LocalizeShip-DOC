---
layout: default
title: Production Workflows
nav_order: 9
has_children: true
description: "Practical release workflows built on the shipped windows and services, with honest notes on the manual steps."
permalink: /docs/workflows/
---

# Production Workflows
{: .fs-9 }

### Processes that survive contact with a real release
{: .fs-6 .text-grey-dk-000 }

These workflows use the windows and services that exist in 1.0.0. Where a step is manual, it says so. None of them assumes a feature that is not shipped.

![Release gates]({{ site.baseurl }}/assets/images/14-release-gates.png)

---

## The workflows

| Workflow | Use it when |
|---|---|
| [First Multilingual Release]({{ site.baseurl }}/docs/workflows/first-release/) | You are taking a monolingual project into its first set of locales |
| [Translator Round-Trip]({{ site.baseurl }}/docs/workflows/translator-round-trip/) | You send strings out and receive translations back |
| [Narrative Iteration]({{ site.baseurl }}/docs/workflows/narrative-iteration/) | Writers change dialogue while translations are in flight |
| [Hotfix & Patch]({{ site.baseurl }}/docs/workflows/hotfix-merge/) | You need to ship a small correction to a live build |
| [CI Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/) | You want automated evidence, with a custom headless entry point |

---

## The shared spine

All five workflows follow the same spine:

```
source strings
    │  ingest (adapter or your own script)
    ▼
workspace revisions ──► review states ──► approvals
    │                                      │
    ├── handoff export ──► translator ──► import plan ──► apply
    │                                      │
    ├── capture matrix ──► visual findings  │
    ├── voice / subtitles ─────────────────┤
    │                                      ▼
    └──────────────────────────────► release gate report
```

If a step is missing from your project, the spine still works — it just has a manual gap. The most common gap is source ingestion: 1.0.0 has no import window.

---

## Rules that keep the process honest

1. Never carry an approval across a source change without re-review.
2. Never treat an unvisited capture tile as a pass.
3. Prefer JSON for three-way imports; CSV is lossy.
4. Archive the gate report with the build, not in a chat message.
5. Record who applied an import; the journal records the reviewer label.
6. Decide explicitly what the gate does not check (text rules, glossary, duration mismatch) and cover it elsewhere.

---

## Related pages

- [Release Board]({{ site.baseurl }}/docs/features/release-board/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
