---
layout: default
title: Capture Runner API
parent: Architecture & API
nav_order: 3
description: "CaptureFixture, line bindings and CaptureRunner.ExecuteFixture, including what the screenshot file actually contains."
permalink: /docs/api/capture-runner-api/
---

# Capture Runner API
{: .fs-9 }

### Evaluate a UI in the editor, with the right expectations
{: .fs-6 .text-grey-dk-000 }

`CaptureRunner` reads live RectTransforms and TMP text, applies `VisualRuleEngine`, and writes a result tile. It is runtime code (no editor dependency), although the only shipped caller is the editor window.

---

## CaptureFixture

```csharp
[DisallowMultipleComponent]
[AddComponentMenu("Decnet/LocalizeShip/Capture Fixture")]
public class CaptureFixture : MonoBehaviour
{
    public string FixtureId;
    public string DisplayName;
    public RectTransform RootContainer;
    public RectTransform SafeAreaReference;
    public List<LineBinding> BoundLines;

    public void PrepareFixtureForLocale(string localeCode);
    public void TeardownFixture();
    public Rect GetContainerRect();
    public Rect GetSafeAreaRect();
}
```

```csharp
[Serializable]
public class LineBinding
{
    public string EntryId;
    public GameObject TargetGameObject;
    public RectTransform ContainerOverride;
    public bool IsInteractive;
}
```

### Locale switching

`PrepareFixtureForLocale` scans `GetComponentsInChildren<MonoBehaviour>()` and invokes a public `SetLocale(string)` method by reflection on every component that has one. Exceptions are logged as warnings and swallowed.

This is how the sample switches text. It is not a localization system; it is a hook.

### Rects

- `GetContainerRect` returns `RootContainer.rect`, or `new Rect(0, 0, Screen.width, Screen.height)` if null.
- `GetSafeAreaRect` returns `SafeAreaReference.rect`, or `Screen.safeArea` if null.

Mixing local rects and screen rects is the caller's responsibility. See the caveat below.

---

## ExecuteFixture

```csharp
public CaptureTileResult ExecuteFixture(
    CaptureFixture fixture,
    LocaleId locale,
    Vector2Int resolution,
    string outputDirectory,
    bool isPlayerMode = false,
    List<FindingSuppression> suppressions = null)
```

Steps:

1. Throws `ArgumentNullException` if `fixture` is null.
2. Creates a `CaptureTileResult` with `Coverage = Visited`, `IsPlayerCapture = isPlayerMode`, and a UTC timestamp.
3. Calls `PrepareFixtureForLocale(locale.Code)` and `Canvas.ForceUpdateCanvases()`.
4. For each binding with a non-null target:
   - resolves the container rect (override or fixture container);
   - reads TMP text and checks `font.HasCharacter(c, true)` for every non-whitespace char;
   - falls back to legacy `Text` and `font.HasCharacter(c)`;
   - measures TMP preferred size and expands the element rect if it exceeds the container;
   - calls `VisualRuleEngine.EvaluateElement`.
5. Optionally writes a PNG.
6. Calls `TeardownFixture()`.
7. Returns the tile.

It does **not** switch scenes, wait for layout across frames, or capture the Game view. `ForceUpdateCanvases` runs synchronously; complex layouts that settle over multiple frames may not be final.

---

## The screenshot is a placeholder

```csharp
var tex = new Texture2D(resolution.x, resolution.y, TextureFormat.RGB24, false);
// fill every pixel with a solid background colour
File.WriteAllBytes(path, tex.EncodeToPNG());
```

The PNG is a solid colour, darker for `ar` than for other locales. It is not a rendering of the UI and must not be used as visual evidence of layout.

The **findings** are real measurements of the live scene objects. The **image** is not.

---

## Output path

```
<outputDirectory>/<FixtureId>_<locale>_<width>x<height>.png
```

`CaptureMatrixWindow` uses:

```
Assets/Decnet/LocalizeShip/Generated/Captures
```

The directory is created if missing. File write failures are logged as warnings and do not fail the tile.

---

## Suppressions

`FindingSuppression` matches on the exact fingerprint:

```csharp
$"{Type}_{EntryId}_{Locale}_{TargetGameObjectName}_{Description}"
```

If the description text changes, the suppression no longer applies. Suppressed findings remain in the list with `IsSuppressed = true` and a reason; `HasErrors` ignores them.

The workspace stores suppressions, and the Capture Matrix passes `workspace.Suppressions`. There is no suppression editor UI.

---

## Example

```csharp
var fixture = Object.FindFirstObjectByType<CaptureFixture>();
var runner = new CaptureRunner();

var tile = runner.ExecuteFixture(
    fixture,
    new LocaleId("de"),
    new Vector2Int(1920, 1080),
    "Assets/Decnet/LocalizeShip/Generated/Captures",
    isPlayerMode: false,
    suppressions: workspace.Suppressions);

workspace.CaptureResults.RemoveAll(t =>
    t.FixtureId == tile.FixtureId && t.Locale == tile.Locale);
workspace.CaptureResults.Add(tile);
```

`FindFirstObjectByType` is Unity 2023+/6 API. The window uses `FindObjectsByType<CaptureFixture>()` without a sort mode, which returns active loaded objects.

---

## Coverage status

| Status | Meaning |
|---|---|
| `Unknown` | Default on a new tile; not visited |
| `Registered` | Treated as unvisited by the gate |
| `Visited` | `ExecuteFixture` completed |
| `Failed` | Not produced by this runner; treated as neither visited nor unvisited by the gate |

{: .warning }
> A `Failed` tile is not counted as unvisited and may have no findings, so the gate can effectively pass it. Check failed captures separately.

---

## Geometry caveat

`Screen.safeArea` is in screen pixels. `RectTransform.rect` is in local canvas units. Passing both into `EvaluateElement` compares different coordinate spaces. Depending on your CanvasScaler and anchors, safe-area findings may be offset or spurious. Validate on your own layout and do not treat a warning as proof of a device-safe layout.

There is no player-capture bridge in 1.0.0. `isPlayerMode` only labels the result.

---

## Related pages

- [Capture Matrix]({{ site.baseurl }}/docs/features/capture-matrix/)
- [Visual QA]({{ site.baseurl }}/docs/features/visual-qa/)
- [Coverage Honesty]({{ site.baseurl }}/docs/concepts/coverage-honesty/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
