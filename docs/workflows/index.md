---
layout: default
title: Production Workflows
nav_order: 9
has_children: true
description: "Real-world production workflows: first release, translator round-trips, CI gates, narrative iterations, and hotfixes."
permalink: /docs/workflows/
---

# Production Workflows
{: .fs-9 }

### Real-World Guides for Multilingual Game Production
{: .fs-6 .text-grey-dk-000 }

---

This section outlines proven, repeatable workflows used by game studios to manage localization from initial alpha through retail launch and live-ops patching.

![Release Cycle Showcase]({{ site.baseurl }}/assets/images/13-release-cycle.png)

---

## Workflow Directory

* [First Multilingual Release]({{ site.baseurl }}/docs/workflows/first-release/) — End-to-end milestone checklist for shipping your game's first multilingual build.
* [Translator Round-Trip]({{ site.baseurl }}/docs/workflows/translator-round-trip/) — Exporting packages, handling external freelancer updates, and resolving 3-way merge conflicts.
* [CI/CD Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/) — Automating release gate evaluations in headless GitHub Actions and Jenkins pipelines.
* [Narrative Iteration Loop]({{ site.baseurl }}/docs/workflows/narrative-iteration/) — Managing rapid dialogue rewrites without breaking downstream translations.
* [Hotfix & Day-1 Patch Merge]({{ site.baseurl }}/docs/workflows/hotfix-merge/) — Safely patching typos and critical text bugs in a live retail build.
