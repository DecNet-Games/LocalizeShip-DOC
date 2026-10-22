---
layout: default
title: VisualRuleEngine
parent: Architecture & API
nav_order: 10
description: "VisualRuleEngine geometry and glyph checks."
permalink: /docs/api/visual-rule-engine/
---

# `VisualRuleEngine`
{: .fs-9 }

`EvaluateElement` returns findings for text overflow, safe-area violations, undersized interactive targets and missing glyphs. The touch target constants are `44f` by `44f`. Overflow allows one unit of tolerance before reporting an error.

```csharp
List<VisualFinding> findings = engine.EvaluateElement(
    entryId, locale, objectName,
    elementRect, containerRect, safeArea,
    isInteractive, renderedText, missingGlyphs, suppressions);
```

Suppression uses an exact finding fingerprint. The engine marks a matched finding suppressed and copies the reason; it does not delete the finding.

{: .warning }
> Supply rectangles in compatible coordinate spaces. The engine compares them as received and performs no canvas-to-screen conversion.

This is deterministic geometry and glyph analysis. It does not inspect screenshot pixels or verify a player build.
