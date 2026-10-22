---
layout: default
title: Narrative Iteration Loop
parent: Production Workflows
nav_order: 4
description: "Managing rapid dialogue rewrites and script iterations during active game production."
permalink: /docs/workflows/narrative-iteration/
---

# Narrative Iteration Loop
{: .fs-9 }

### Rapid Script Rewrites Without Breaking Downstream Translations
{: .fs-6 .text-grey-dk-000 }

---

During active game development, writers frequently polish dialogue: tweaking comedic timing, improving quest clarity, or adjusting character voice. If every single comma tweak invalidated all translations in a vacuum, the localization team would be overwhelmed.

LocalizeShip provides a balanced **Narrative Iteration Loop**.

---

## 1. Classifying Source Changes

When a writer saves a modified source line, LocalizeShip's diff engine categorizes the change:

* **Cosmetic Whitespace & Punctuation**: Line ends with `.` instead of `...`, or trailing space removed. The normalization policy handles line endings and trims whitespace automatically, preventing unnecessary invalidations.
* **Minor Phrasing Fix**: Non-semantic words replaced (e.g., `"Yes, sure"` $\rightarrow$ `"Yes, certainly"`).
* **Major Semantic Rewrite**: Sentence structure changed, quest objective altered, or dynamic placeholders added.

---

## 2. Reviewing the Source Diff

In the Release Board, the right-hand Inspector displays a character-by-character diff:

```
[Source Revision Diff]
- "Travel to the old mill and find the missing key."
+ "Travel to the abandoned mill and find the rusted key."
```

If the language lead reviews the diff and determines that the German translation (`"Gehe zur alten Mühle und finde den Schlüssel"`) is still accurate and acceptable, they can click:

**[ Re-Approve for Current Revision ]**

This instantly creates an `ApprovalEvent` binding the existing German translation to the new English revision hash, clearing the `Stale` flag with an explicit audit note: *"Approved without re-translation: minor adjective polish in source"*.

---

## 3. Isolating Major Rewrites

For major rewrites, leave the line marked as `Stale`. When it's time to send out a batch to translators, use **Translation Handoff** with scope set to **Changed & Stale Lines Only**. Translators only see the lines that genuinely require rework.
