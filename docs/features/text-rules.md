---
layout: default
title: Text Rules
parent: Features
nav_order: 8
description: "Placeholder, brace and rich-text validation plus glossary checks and pseudolocalization helpers."
permalink: /docs/features/text-rules/
---

# Text Rules
{: .fs-9 }

### The string that renders is not always the string that ships
{: .fs-6 .text-grey-dk-000 }

A translation can be fluent and still break the build: a missing `{count}`, an unclosed `<b>`, a stray brace. The text rule engine validates the mechanics that translators cannot see in a spreadsheet.

---

## Placeholder validation

```csharp
var violations = new TextRuleEngine()
    .ValidatePlaceholders(sourceRevision, translationText, locale);
```

The engine extracts placeholders from the source revision's `Placeholders` list and from the translation with:

```csharp
var matches = Regex.Matches(translationText, @"\{([^{}]+)\}");
```

For each matched tag it trims the content. If the tag contains a comma, it keeps only the part before the first comma as the argument name — a pragmatic way to recognize `{count, plural, ...}` style expressions.

### Results

| Rule | Severity | When |
|---|---|---|
| `MissingPlaceholder` | `Error` | A source argument is absent from the translation |
| `UnexpectedPlaceholder` | `Warning` | The translation contains an argument not present in the source (only checked when the source has at least one placeholder) |
| `MalformedBraces` | `Fatal` | The count of `{` and `}` in the translation is unbalanced |
| `MismatchedRichTextTag` | `Error` | A closing tag does not match the open stack |
| `UnclosedRichTextTag` | `Error` | A tag remains open at the end |

Each violation carries the entry ID, locale, rule name and message.

---

## Example

Source:

```
You have <b>{count}</b> gold and {gems} gems.
```

| Translation | Result |
|---|---|
| `Du besitzt {gems} Edelsteine und <b>{count}</b> Gold.` | Pass — reordering is allowed |
| `Du besitzt {count} Gold.` | `MissingPlaceholder` for `gems` |
| `Du besitzt {gems} Edelsteine und <b>{count} Gold.` | `UnclosedRichTextTag` |
| `Du hast <b>{count} Gold</i>.` | `MismatchedRichTextTag` |
| `Du hast {count Gold.` | `MalformedBraces` |

---

## Rich-text handling

The supported tag set is generic, not TMP-specific. The parser:

- treats tags ending with `/` as self-closing;
- treats `br`, `page` and `sprite` as self-closing even without a slash;
- lowercases opening and closing tag names;
- takes the first token before `=` or a space as the tag name.

This covers common TMP forms such as `<b>`, `<color=#ff0000>`, `<sprite=0>` and `<br>`. It does not validate attribute syntax, nesting semantics, or whether the TMP component supports the tag.

---

## The plural caveat

The regex `\{([^{}]+)\}` cannot match across nested braces. For a full ICU expression such as:

```
{count, plural, one {# item} other {# items}}
```

the matcher sees several inner fragments, not one expression. `ValidatePlaceholders` reduces a fragment with a comma to its first token, which usually recovers `count`, but the extracted source `Placeholders` list can also contain fragments like `# item` and `other`.

Consequences:

- Simple `{name}`, `{0}` and shallow plural arguments work.
- Deeply nested plural/select expressions can produce noisy `MissingPlaceholder` results.
- The engine is a guardrail, not a full ICU MessageFormat parser.

{: .warning }
> If your project uses complex ICU messages, validate them with your own parser before relying on the green result from this engine.

---

## Glossary checking

`CheckGlossary` is a separate method:

```csharp
List<GlossaryViolation> CheckGlossary(
    string text,
    StableEntryId entryId,
    LocaleId locale,
    List<GlossaryTerm> terms);
```

It checks **forbidden variants only**, not whether the approved term is present:

```csharp
foreach (var forbidden in term.ForbiddenVariants)
{
    var comp = term.CaseSensitive
        ? StringComparison.Ordinal
        : StringComparison.OrdinalIgnoreCase;

    int index = text.IndexOf(forbidden, comp);
    if (index >= 0) { /* violation with surrounding snippet */ }
}
```

Violations include the expected term, the found variant and a snippet around it.

It is not called by the Release Board or the gate in 1.0.0. See [Glossary]({{ site.baseurl }}/docs/features/glossary/).

---

## Pseudolocalization helper

`TextRuleEngine.GeneratePseudoText` produces a stress string:

```csharp
string pseudo = TextRuleEngine.GeneratePseudoText(
    "Hello traveler",
    expansionFactor: 1.35f,
    rtlSimulation: false);
```

Behaviour:

- wraps the result in `[!! ... !!]`;
- replaces vowels with accented characters (`a→å`, `e→é`, `i→î`, `o→ø`, `u→ü`, `c→ç`, `n→ñ`);
- pads with lowercase letters to simulate expansion;
- when `rtlSimulation` is true, reverses the characters and wraps them in U+202E / U+202C bidi override marks.

It is a public helper, not wired to an editor window in 1.0.0. Use it in your own fixture setup to stress layouts before real translations arrive.

{: .note }
> Reversing characters is not a substitute for real RTL shaping or linguistic review. It is a crude layout stress test.

---

## Where text rules run

| Surface | Runs placeholder rules? |
|---|---|
| Release Board | No |
| Translation Handoff | No (placeholders are exported as constraints) |
| Review Imports | No |
| Capture Matrix | No |
| Release Gates | No (only approval, visual, voice, subtitle checks) |
| Advisory validation | Yes, as a guardrail |
| Your integration code | Yes, when you call it |

This is a deliberate scope boundary in 1.0.0: the engine is available, but only the advisory validator calls it automatically. A practical pipeline should call `ValidatePlaceholders` before approving a cell and before applying an import.

---

## Recommended integration

```csharp
var textRules = new TextRuleEngine();

foreach (var rev in workspace.SourceRevisions)
{
    foreach (var locale in workspace.TargetLocales)
    {
        var approval = workspace.GetApproval(rev.EntryId, new LocaleId(locale));
        if (approval == null || string.IsNullOrEmpty(approval.TranslationText)) continue;

        var violations = textRules.ValidatePlaceholders(
            rev, approval.TranslationText, new LocaleId(locale));

        foreach (var v in violations)
        {
            Debug.LogWarning($"{v.EntryId} [{v.Locale}] {v.RuleName}: {v.Message}");
        }
    }
}
```

Run it before generating a gate report and treat `Error`/`Fatal` as blockers in your own policy.

---

## Related pages

- [Glossary]({{ site.baseurl }}/docs/features/glossary/)
- [Advisory Drafts]({{ site.baseurl }}/docs/features/advisory-drafts/)
- [Normalization]({{ site.baseurl }}/docs/concepts/normalization/)
- [Release Gates]({{ site.baseurl }}/docs/features/release-gates/)
