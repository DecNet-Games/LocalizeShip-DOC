---
layout: default
title: Normalization
parent: Concepts
nav_order: 6
description: "The versioned normalization policy that decides when a source edit becomes a new revision."
permalink: /docs/concepts/normalization/
---

# Normalization
{: .fs-9 }

### Not every whitespace change is a translation change
{: .fs-6 .text-grey-dk-000 }

If a formatter re-saves a table and changes CRLF to LF, every source line technically differs. If a writer adds a trailing space, the string is not semantically new. LocalizeShip normalizes text before hashing so cosmetic churn does not invalidate a release.

---

## The policy

```csharp
[Serializable]
public class NormalizationPolicy
{
    public int Version = 1;
    public bool TrimWhitespace = true;
    public bool NormalizeLineEndings = true;   // CRLF -> LF
    public bool CollapseDuplicateWhitespace = false;
    public bool IgnoreCase = false;
}
```

Defaults:

| Rule | Default | Effect |
|---|---|---|
| Trim whitespace | On | Removes leading and trailing whitespace |
| Normalize line endings | On | `\r\n` and `\r` become `\n` |
| Collapse duplicate whitespace | Off | When on, collapses runs of horizontal whitespace to one space, preserving newlines |
| Ignore case | Off | When on, lowercases the text with the invariant culture |
| Version | 1 | Included in the hash payload |

---

## Normalize order

```csharp
public string Normalize(string rawText)
{
    if (rawText == null) return string.Empty;
    string text = rawText;
    if (NormalizeLineEndings) text = text.Replace("\r\n", "\n").Replace("\r", "\n");
    if (TrimWhitespace) text = text.Trim();
    if (CollapseDuplicateWhitespace) text = Regex.Replace(text, @"[^\S\r\n]+", " ");
    if (IgnoreCase) text = text.ToLowerInvariant();
    return text;
}
```

Order matters: line endings are normalized first, then trimming, then horizontal whitespace collapse, then case.

---

## Why the version is in the hash

The payload is:

```
{Version}|{normalizedText}|{context}
```

Changing a rule without changing the version would make two different normalization regimes produce identical hashes for some inputs. Including the version makes the policy part of the identity.

{: .warning }
> If you change the policy **and** bump `Version`, every newly processed source line gets a different hash and every previously approved record becomes **Stale** on the next revision pass. That is correct behaviour, but schedule it — do not flip normalization rules the week before launch.

---

## Examples

Given the default policy:

| Raw | Normalized | Same hash as previous? |
|---|---|---|
| `"Hello world"` | `"Hello world"` | — |
| `"Hello world  "` | `"Hello world"` | Yes |
| `"Hello world\r\n"` | `"Hello world"` | Yes |
| `"Hello\r\nworld"` | `"Hello\nworld"` | Different text, stable across line-ending churn |
| `"Hello world"` vs `"Hello  world"` | `"Hello world"` vs `"Hello  world"` | Different (collapse is off) |
| `"HELLO WORLD"` | `"HELLO WORLD"` | Different (ignore case is off) |

With `CollapseDuplicateWhitespace = true`, the fourth comparison would match. With `IgnoreCase = true`, the fifth would match.

---

## What normalization does not do

- It does not strip rich-text tags. `<b>Hello</b>` and `<b>Hello</b>` hash as text; a tag change is a real change.
- It does not normalize Unicode forms (NFC/NFD). If your pipeline can produce canonically equivalent but differently encoded text, normalize before it reaches LocalizeShip.
- It does not remove zero-width characters or soft hyphens.
- It does not translate or validate placeholders. That is the text rule engine.
- It does not apply to translation hashes differently — `ComputeTranslationHash` uses the same policy but the literal context `"translation"`.

---

## Placeholder extraction

`SourceRevision` also extracts placeholders from the **raw** text:

```csharp
var matches = Regex.Matches(RawText, @"\{([^{}]+)\}");
```

Each match's inner tag is trimmed and de-duplicated. The extracted list is used for:

- handoff constraints (`RequiredPlaceholders`),
- placeholder validation in the text rule engine,
- translation handoff context.

Because the regex excludes nested braces, complex ICU plural expressions are not parsed as a single AST. See [Text Rules]({{ site.baseurl }}/docs/features/text-rules/) for the practical consequences.

---

## Practical policy guidance

1. **Keep the default policy** unless you have a concrete reason to change it.
2. **Do not enable ignore-case** for languages where case changes meaning, and do not enable it globally just to suppress churn.
3. **Do not enable whitespace collapse** if your game renders intentional double spaces.
4. **Treat a policy change as a release event.** Record it in the changelog and expect invalidations.
5. **Store the policy in the workspace asset.** It is serialized with the release evidence.

---

## Related pages

- [Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/)
- [Review States]({{ site.baseurl }}/docs/concepts/review-states/)
- [Text Rules]({{ site.baseurl }}/docs/features/text-rules/)
