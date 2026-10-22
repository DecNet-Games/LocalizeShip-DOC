---
layout: default
title: RPG Sample Studio
nav_order: 8
has_children: true
description: "A complete, self-contained multilingual RPG project bundled with LocalizeShip for evaluation and learning."
permalink: /docs/sample/
---

# RPG Sample Studio
{: .fs-9 }

### A Complete, Working Multilingual Testbed
{: .fs-6 .text-grey-dk-000 }

---

LocalizeShip includes a comprehensive, production-grade sample project called the **RPG Sample Studio**. It provides a fully working multilingual RPG environment with dialogue branching, quest logs, inventory tooltips, voiced audio greetings, and UI fixtures.

![Release Cycle Showcase]({{ site.baseurl }}/assets/images/13-release-cycle.png)

---

## Why a Bundled Sample?

Most localization assets require you to configure complex string tables, author dialogue, record audio, and build UI before you can test a single feature.

The RPG Sample Studio generates a complete, isolated multilingual world with a single click:
* **Languages Included**: English (`en`), German (`de`), Japanese (`ja`), and Arabic (`ar`).
* **Content Surfaces**: Dialogue box, Quest log, Inventory panel, Audio voice audition, and Subtitle display.
* **Pre-Seeded Scenarios**: Includes a seeded text overflow in German, an outdated voice take in English, and complex Smart String plurals.
* **Completely Isolated**: Operates inside its own owned directory (`Assets/Decnet/LocalizeShip/Samples/RPGSample/`) without touching or polluting your project's production assets.

---

## Sample Guides

* [RPG Sample Studio Setup]({{ site.baseurl }}/docs/sample/rpg-sample-studio/) — How to generate, configure, and inspect the sample.
* [Scenario Walkthrough]({{ site.baseurl }}/docs/sample/scenario-walkthrough/) — Step-by-step reproduction of the full release QA loop.
