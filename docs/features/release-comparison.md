---
layout: default
title: Release Comparison
parent: Features
nav_order: 12
description: "Diffing release baselines to track string mutations, new locales, and remaining unverified coverage."
permalink: /docs/features/release-comparison/
---

# Release Comparison
{: .fs-9 }

### Structural Milestones & Release-to-Release Diffing
{: .fs-6 .text-grey-dk-000 }

---

When preparing a major patch (e.g., updating from `v1.0.0` to `v1.1.0`), teams often ask: **"What actually changed in our localization between these two builds?"**

Standard git diffs on binary `.asset` files or sprawling CSVs are unreadable. LocalizeShip's **Release Comparison Tool** performs a semantic, release-level comparison.

![Release Comparison Overview]({{ site.baseurl }}/assets/images/13-release-cycle.png)

---

## What the Comparison Reveals

Selecting two releases to compare produces an actionable diff breakdown:

```
┌─────────────────────────────────────────────────────────────┐
│ RELEASE DIFF:  v1.0.0  ──►  v1.1.0                          │
├─────────────────────────────────────────────────────────────┤
│ * Added Strings:          48 lines                          │
│ * Modified Source Lines:  14 lines                          │
│ * Deleted Strings:         3 lines                          │
│ * Newly Added Locales:    1 locale (ja-JP)                  │
│ * Audio Re-recordings:    12 takes updated                  │
│ * Remaining Unapproved:   6 lines in de-DE                  │
│ * Visual Coverage Delta:  +4 new fixtures registered        │
└─────────────────────────────────────────────────────────────┘
```

### Deep ID-Aware Tracking
Rather than reporting "Table A grew by 45 rows", the comparison tool identifies:
* Exactly which `StableEntryId`s were added, modified, or removed.
* Lines where source text remained unchanged, but translators updated phrasing.
* Lines that had approvals revoked due to upstream script modifications.

---

## Patching & DLC Workflows

For DLC expansions or seasonal content updates:
1. Snapshot your base game release as `v1.0-Shipped`.
2. Author your DLC content in the same or new string tables.
3. Compare `v1.0-Shipped` against `v1.1-DLC`.
4. The tool generates an isolated handoff package containing **only the net-new and modified lines for the DLC**, saving your team thousands of dollars in translation overhead.
