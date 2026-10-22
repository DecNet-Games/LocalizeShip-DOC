---
layout: default
title: Data & Privacy
nav_order: 17
description: "Local-first operation, what is stored where, and the privacy properties of the workspace, diagnostics and advisory configuration."
permalink: /docs/data-and-privacy/
---

# Data & Privacy
{: .fs-9 }

### Local by construction, not by policy promise
{: .fs-6 .text-grey-dk-000 }

LocalizeShip has no servers, no telemetry and no network client. Your strings stay in your project. This page states what is stored and where, so you can review it against your own NDA and data-handling rules.

---

## Network behaviour

| Component | Network access |
|---|---|
| Core services | None |
| Editor windows | None |
| Capture runner | None |
| Advisory validation | None — parses a string you supply |
| Diagnostics export | None — writes a local file |

`AdvisoryConfig` contains an endpoint URL and model name as **defaults**. No code in the package sends a request to them. If you write a client, its network behaviour is yours.

---

## What is stored in the workspace asset

`ReleaseWorkspace` is a `ScriptableObject` saved under `Assets/`. It contains:

- workspace ID, release version, source and target locales;
- normalization and subtitle policies;
- every source string (`RawText` and `NormalizedText`);
- every approval: translation text, state, hashes and full event history;
- voice entries with audio asset paths;
- subtitle cues and their text;
- glossary terms;
- capture results, including descriptions and screenshot paths;
- finding suppressions.

Treat the asset as confidential production data. It contains unreleased narrative and every translation.

### Version control

The asset is plain Unity YAML. If you commit it:

- your whole localization state is in the repository history;
- reverting the asset reverts approvals and history;
- merge conflicts are painful because the lists are serialized in order.

Many teams commit it deliberately for auditability. Decide with your legal and production leads.

---

## What is stored elsewhere

| Data | Location |
|---|---|
| Handoff packages | The output folder you choose |
| Gate reports | Project root (`LocalizeShip_Report_v*.html/.json`) |
| Capture PNGs | `Assets/Decnet/LocalizeShip/Generated/Captures` |
| Diagnostics | Project root, timestamped |
| Import journals | Memory only, lost on editor restart |
| Advisory API key | `[NonSerialized]` field — not persisted anywhere by the package |

The generated capture folder lives under `Assets/`, so Unity imports the PNGs. Add it to `.gitignore` if you do not want placeholder images in version control.

---

## Secrets

There is no secret storage in 1.0.0.

- `AdvisoryConfig.SessionApiKey` is `[NonSerialized]`; Unity will not write it to the asset.
- The package does not read environment variables or `EditorPrefs`.
- If your integration reads a key, store it in your OS keychain, CI secret store or an environment variable, and never in the workspace.

{: .warning }
> Do not add a serialized API key field to a ScriptableObject in your project. Unity assets are easy to commit and hard to scrub from history.

---

## Diagnostics

The export contains environment strings and counts only. It does not include source text, translations, IDs, paths or reviewer names. See [Support & Diagnostics]({{ site.baseurl }}/docs/support-diagnostics/) for the exact format.

---

## Handoff packages

A handoff file necessarily contains source text and translations. It is meant to leave your machine.

- Send it over a channel approved for confidential material.
- Do not put it in a public issue or repository.
- Prefer JSON if you need full metadata; both formats contain the strings.
- Formula protection prevents spreadsheet execution, not disclosure.

---

## Reports

Gate reports contain entry IDs, locales and violation descriptions. They do **not** contain full translations, but they do reveal the existence and scale of localization problems. Treat them as internal unless reviewed.

---

## Data deletion

There is no delete API for revisions or approvals. To remove data:

1. edit the workspace asset in the Inspector or with a script;
2. remove the relevant lists;
3. save and commit the change.

Deleting an approval removes its event history with it. If you need an audit record of the deletion, export a report or keep a copy of the asset before editing.

---

## Compliance checklist

- [ ] Decide whether the workspace asset is committed.
- [ ] Add generated captures and reports to `.gitignore` if they are not evidence.
- [ ] Confirm handoff channels are approved for confidential text.
- [ ] Confirm no API keys are serialized in your integration.
- [ ] Review the diagnostics file before sending it.
- [ ] Document who can read the workspace asset.

---

## Related pages

- [Security & Privacy]({{ site.baseurl }}/docs/security-and-privacy/)
- [Support & Diagnostics]({{ site.baseurl }}/docs/support-diagnostics/)
- [Audit Trail]({{ site.baseurl }}/docs/concepts/audit-trail/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
