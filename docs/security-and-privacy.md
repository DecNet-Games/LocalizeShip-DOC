---
layout: default
title: Security & Privacy
nav_order: 18
description: "Threat-focused guidance: untrusted imports, formula injection, file exposure, CI secrets and third-party adapters."
permalink: /docs/security-and-privacy/
---

# Security & Privacy
{: .fs-9 }

### Treat imports as untrusted input
{: .fs-6 .text-grey-dk-000 }

LocalizeShip is a local editor tool, not a server. Its security surface is the files you feed it, the project data it stores, and the scripts you add around it.

---

## Threat model

| Asset | Risk | Mitigation |
|---|---|---|
| Workspace asset | Contains unreleased narrative and translations | Restrict repository access; know whether it is committed |
| Handoff files | Leave the machine with full source text | Use an approved transfer channel |
| Import files | Untrusted text from external translators | Review the plan; never auto-apply |
| CSV exports | Formula injection in spreadsheets | Built-in apostrophe prefixing |
| Gate reports | Reveal defect scale and IDs | Treat as internal |
| CI entry points | Shell execution with project credentials | Pin actions, scope secrets |
| Advisory integration | Outbound text and API key | You own the client, key storage and data-sharing decision |

---

## Formula injection

Translated text is attacker-controlled input when it comes from an external vendor. A cell beginning with `=` can execute in a spreadsheet.

`MergePlanner.ProtectFormula` prefixes `=`, `+`, `-`, `@`, tab and CR with `'`. CSV export applies it; CSV import removes it. This is a transport defence, not a sanitizer for your runtime.

What it does not cover:

- text beginning with `'` followed by a formula character (it will unprotect on import);
- other spreadsheet tools with different triggers;
- values that become formulas after your own concatenation.

If you generate your own exports, reuse `ProtectFormula` or an equivalent.

---

## Import review

- `DuplicateIdError` and `UnknownIdError` rows are skipped on apply. Check them before applying; a skipped row can look like a successful import.
- `BlankIncomingValue` rows can downgrade an approved cell to `NeedsReview`.
- Unresolved conflicts block apply, but resolved conflicts still need human judgment.
- There is no signature or checksum on a handoff file. Verify the sender and the file hash out of band.

---

## File exposure

The workspace contains more than translations: it contains context notes, speaker names and reviewer history. Some studios treat that as spoiler material.

Recommended:

- add `Assets/Decnet/LocalizeShip/Generated/Captures/` to `.gitignore` unless the placeholders are deliberate evidence;
- keep `LocalizeShip_Report_v*.json/html` out of public repositories;
- store handoff packages outside the `Assets/` tree so Unity does not import them;
- review the diagnostics file before sharing (counts only, but still internal).

---

## CI secrets

If you add an advisory client:

- read the key from a CI secret or OS keychain;
- never pass it on the command line where it appears in process lists;
- never write it into a ScriptableObject or scene;
- do not log request bodies containing unreleased text;
- set timeouts and retry caps.

The package provides no key store and no HTTP client, so these decisions are entirely yours.

---

## Third-party adapters

The I2 and Dialogue System adapters do not reference or bundle third-party code. If you implement them:

- add the third-party assembly reference in your own assembly;
- respect the vendor's licence;
- do not copy proprietary source into the package;
- test against the version you own.

No claim of licensed-install verification is made by DecNet Games.

---

## Editor scripts

Unity editor scripts run with your user's permissions. A malicious import cannot execute code by itself, but a custom importer that deserializes arbitrary types or shells out can. Keep importers narrow:

- parse known fields;
- do not use type-name-based deserialization;
- do not evaluate expressions from the file;
- validate IDs against the workspace before writing.

---

## Reporting a security issue

Do not open a public issue for a vulnerability. Contact DecNet Games through the support channel listed on the Asset Store page, with reproduction steps and impact.

---

## Related pages

- [Data & Privacy]({{ site.baseurl }}/docs/data-and-privacy/)
- [Merge Imports]({{ site.baseurl }}/docs/features/merge-imports/)
- [Advisory Drafts]({{ site.baseurl }}/docs/features/advisory-drafts/)
- [Support & Diagnostics]({{ site.baseurl }}/docs/support-diagnostics/)
