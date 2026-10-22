---
layout: default
title: Documentation Viewer
parent: Features
nav_order: 14
description: "Open the ten manuals that ship inside the LocalizeShip package without leaving Unity."
permalink: /docs/features/documentation-viewer/
---

# Documentation Viewer
{: .fs-9 }

### Keep the operating manual beside the release workspace
{: .fs-6 .text-grey-dk-000 }

The Documentation Viewer is the seventh LocalizeShip editor window. Open it from `Tools > Decnet > LocalizeShip > Documentation` when you need the package manual without switching away from Unity.

## What problem it solves

Release work crosses setup, provider integration, review, handoff, capture, gates and troubleshooting. The viewer makes the ten offline Markdown manuals discoverable from one window, so the instructions remain available without a network connection.

## Use it with the RPG sample

1. Generate or open the RPG sample workspace.
2. Open `Tools > Decnet > LocalizeShip > Documentation`.
3. Select the manual relevant to the task: start with Quick Start for setup, Concepts for revision behaviour, and Workflows for a complete release path.
4. Keep the viewer open beside the Release Board while you work through the sample.
5. Use the dedicated feature pages on this site when you need source-level detail beyond the bundled manual.

## What ships

The package contains ten offline manuals under `Assets/Decnet/LocalizeShip/Documentation`. The viewer presents those package resources; it is not a web browser and does not fetch remote documentation.

> [!TIP]
> Use the in-Editor viewer for the immediate task, then archive the generated JSON or HTML release report with your release evidence. Reading documentation does not change workspace state.

## Honest limits

- The viewer does not update manuals over the network.
- It does not edit the Markdown files.
- It does not validate, approve, capture or gate a release.
- This page has no product capture because the supplied media inventory contains no dedicated Documentation Viewer image.

## Related pages

- [Getting Started]({{ site.baseurl }}/docs/getting-started/)
- [Menu Reference]({{ site.baseurl }}/docs/menu-reference/)
- [Third-Party Notices]({{ site.baseurl }}/docs/third-party-notices/)
