---
layout: default
title: Architecture & API
nav_order: 10
has_children: true
description: "C# architecture, public domain models, services, and headless runner APIs for LocalizeShip."
permalink: /docs/api/
---

# Architecture & API Reference
{: .fs-9 }

### Developer Reference for C# Domain Models, Services & Automation
{: .fs-6 .text-grey-dk-000 }

---

LocalizeShip is built with clean, modular C# architecture adhering to domain-driven design principles. The codebase is strictly split between runtime components and editor tooling.

---

## Assembly Definition Structure

```
Assets/Decnet/LocalizeShip/
├── Runtime/
│   └── Decnet.LocalizeShip.Runtime.asmdef
│       └── Lightweight capture fixtures, locale listeners, and test hooks.
│           (Safe to include in player builds; zero Editor dependencies).
└── Editor/
    └── Decnet.LocalizeShip.Editor.asmdef
        └── Window UI, merge planners, serializers, gate evaluators, and CLI.
            (Editor-only; stripped automatically from runtime builds).
```

---

## API Documentation Directory

* [Identifiers & Data Models]({{ site.baseurl }}/docs/api/identifiers-and-models/) — Strongly-typed IDs (`StableEntryId`, `SourceRevisionHash`, `LocaleId`) and serialized models.
* [Domain Services]({{ site.baseurl }}/docs/api/domain-services/) — `RevisionTracker`, `ImpactEngine`, `MergePlanner`, and `TextRuleEngine`.
* [Capture Runner API]({{ site.baseurl }}/docs/api/capture-runner-api/) — Programmatic visual fixture execution and canvas measurement.
* [Release Gate API]({{ site.baseurl }}/docs/api/release-gate-api/) — Headless gate evaluation and report generation interfaces.
