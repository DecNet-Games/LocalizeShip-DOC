---
layout: default
title: TextRuleEngine
parent: Architecture & API
nav_order: 9
description: "TextRuleEngine placeholder, markup, glossary and pseudolocalization API."
permalink: /docs/api/text-rule-engine/
---

# `TextRuleEngine`
{: .fs-9 }

`ValidatePlaceholders` compares source and translated brace tokens, reports unexpected tokens, checks brace balance and validates paired rich-text tags. TMP tags `br`, `page` and `sprite` are treated as self-closing.

```csharp
List<TextRuleViolation> issues = engine.ValidatePlaceholders(
    revision, translatedText, new LocaleId("de"));
```

`CheckGlossary` searches configured forbidden variants using each term's case policy. It does not require the approved term and does not calculate an accuracy score.

`GeneratePseudoText` accents characters, pads to an expansion factor and can reverse the result with bidi override marks. That option is a layout stress helper, not shaping or language validation.

> [!TIP]
> Run the text rules before approval and before your release gate. `ReleaseGateEvaluator` does not invoke this service.
