---
layout: default
title: Coverage Honesty
parent: Concepts
nav_order: 4
description: "Registered, visited, unknown and failed coverage; Editor approximation versus player capture; what LocalizeShip does not certify."
permalink: /docs/concepts/coverage-honesty/
---

# Coverage Honesty
{: .fs-9 }

### The most dangerous test result is the one that never ran
{: .fs-6 .text-grey-dk-000 }

A release dashboard that shows 100% green because nothing was checked is worse than no dashboard. LocalizeShip treats *not visited* as a first-class state and refuses to call it a pass.

![Capture matrix]({{ site.baseurl }}/assets/images/09-capture-matrix.png)

---

## The four coverage states

```csharp
public enum FixtureCoverageStatus
{
    Unknown,
    Registered,
    Visited,
    Failed
}
```

| State | Meaning | Gate treatment |
|---|---|---|
| `Unknown` | A capture fixture exists in the workspace but has no result tile | Counted as unvisited |
| `Registered` | A tile is known but has not been visited | Counted as unvisited |
| `Visited` | `CaptureRunner.ExecuteFixture` produced a tile | Inspected for findings |
| `Failed` | The tile was marked failed | **Not counted as unvisited** in the current gate |

{: .warning }
> `Failed` is not treated as a blocker by the current gate's unvisited count. If a fixture failed to capture, add a finding or handle the state in your own gate extension. Do not assume `Failed` automatically fails a release.

A fresh `CaptureTileResult` starts as `Unknown`. `CaptureRunner` sets `Coverage = Visited` at the start of a run.

---

## Editor approximation versus player capture

The model carries a label:

```csharp
public bool IsPlayerCapture; // true = Player bridge, false = Editor approximation
```

The Capture Matrix window calls the runner with `isPlayerMode: false`, and the UI prints **Editor Approximation**. There is no player bridge in 1.0.0, and the shipped code does not produce a `true` value for that field.

What that means in practice:

- Layout is measured against the current Editor canvas state.
- `PrepareFixtureForLocale` switches the fixture's locale through any child `SetLocale(string)` method.
- `Canvas.ForceUpdateCanvases()` is called before measurement.
- The result is an Editor-side layout evaluation, not proof of what a device renders.

{: .important }
> LocalizeShip does **not** claim player-capture verification. The product deliberately does not certify device rendering, platform font fallback, or runtime aspect-ratio behaviour in 1.0.0. Treat the matrix as a fast Editor-side early-warning system, not a substitute for a device smoke test.

---

## What the screenshot file is — and is not

`CaptureRunner` writes a PNG named:

```
{FixtureId}_{locale}_{width}x{height}.png
```

The current implementation fills a texture with a background colour (slightly different for Arabic) and writes it. It does **not** read the game camera, does not call `ScreenCapture`, and does not composite the UI.

So the screenshot is a **file artifact that records that a capture ran at a resolution**. It is not visual evidence of the rendered screen. Findings come from rectangle measurement and font glyph queries, not from image analysis.

This is exactly the kind of thing a documentation site should say out loud. If you need rendered frames, extend the runner to capture the camera/render texture and keep the same coverage labels.

---

## Resolution and viewport honesty

The runner accepts a `Vector2Int resolution` and stores it on the tile. The Capture Matrix window always passes `1920 × 1080` for a tile run. It does **not** resize the game view, the canvas, or the screen before measuring.

Therefore:

- “Locale × viewport” in 1.0.0 means locale × recorded resolution label.
- There is no device-preset enumerator and no automatic resize loop.
- If you need true multi-viewport coverage, drive the runner from your own code after resizing the target canvas, and store the resulting tile per resolution.

The product concept describes a fuller viewport matrix. The current implementation is narrower, and the documentation does not pretend otherwise.

---

## Unknown is not empty

An empty findings list on a `Visited` tile means “the rules found nothing on the elements that were measured.” It does not mean the screen is correct. Concretely, the rules cannot see:

- text that is visually clipped by a mask without a rect overflow;
- overlap between two elements (`LayoutCollision` is defined but no rule emits it);
- clipping that does not exceed the container rect;
- untranslated content that renders fine;
- linguistic quality, grammar, tone or RTL correctness.

A tile with zero findings is evidence that four geometric checks passed on the bound elements. That is all.

---

## The honesty contract

| Claim | Status |
|---|---|
| “We measured overflow against the container rect” | Supported |
| “We detected missing glyphs in the font asset” | Supported |
| “We checked safe-area bounds” | Supported |
| “We checked minimum touch-target size” | Supported |
| “We evaluated every registered fixture” | Only if `Coverage == Visited` for each tile |
| “We captured the actual rendered player screen” | Not claimed |
| “The layout is correct on device” | Not claimed |
| “The Arabic/RTL text is linguistically correct” | Not claimed |
| “Every dynamic string was visited” | Not claimed; dynamic paths need explicit fixtures |

---

## How to use coverage honestly in a release review

1. **Register fixtures for every screen that contains localized text.** A missing fixture is worse than an unknown tile because it does not even appear as a gap.
2. **Run every tile for every target locale.** Do not sample.
3. **Read the gate’s “unvisited” count before reading the findings count.** Zero findings with three unvisited tiles is not a pass.
4. **Label the evidence.** If you attach an Editor run to a release ticket, say “Editor approximation, 1920×1080, [date]”.
5. **Keep the device test separate.** A green matrix does not replace a build smoke test.

---

## Related pages

- [Capture Matrix]({{ site.baseurl }}/docs/features/capture-matrix/)
- [Visual QA]({{ site.baseurl }}/docs/features/visual-qa/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
