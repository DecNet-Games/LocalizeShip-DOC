---
layout: default
title: FAQ
nav_order: 16
description: "Direct answers about licensing, supported Unity versions, offline operation, CI, providers and what the gate actually checks."
permalink: /docs/faq/
---

# Frequently Asked Questions
{: .fs-9 }

### Answers that match the shipped code
{: .fs-6 .text-grey-dk-000 }

---

### Does LocalizeShip replace Unity Localization, I2 or Dialogue System?

No. It sits beside them. Strings stay in your provider; LocalizeShip stores revisions, approvals, QA findings and release evidence in a workspace asset.

### Is it a subscription or a cloud service?

No. It is a one-time paid Unity Asset Store package. There is no account, server, telemetry or mandatory network access.

### Does anything leave my machine?

No. The package has no HTTP client. Diagnostics are written locally. Handoff files leave only when you send them.

### Does it need an external drafting service?

No. The advisory module validates a JSON payload you supply. It does not call a model.

### Which Unity version is supported?

Verified on **Unity 6000.5.4f1** with Unity Localization **1.5.13** and Addressables **1.25.0**. No claim is made for 2022.3, 2023 or other 6.x patches. Run the 12-test suite on your version before relying on it.

### How many tests pass?

**12 of 12** EditMode acceptance tests pass on the verified Unity version.

### How does it decide a translation is stale?

It hashes the normalized source text, speaker, context and policy version with SHA-256 (first 8 bytes, 16 hex chars). An approval records that hash. When the source hash changes, `Approved` becomes `Stale`.

### Does editing a comment or metadata stale translations?

Only fields included in the hash: normalized text, speaker and context description. The Unity adapter imports empty speaker/context, so for provider-imported lines those do not affect the hash until you populate them.

### Can I import my existing strings?

Not through a window. There is no source-import UI in 1.0.0. Use `UnityLocalizationAdapter.ReadCollection` or write a reader for your format, then add revisions to the workspace. See [First Release]({{ site.baseurl }}/docs/workflows/first-release/).

### Does approving a translation write it back to Unity tables?

Not automatically. The Unity adapter has `WriteTranslation`, but no window calls it. Apply imports to the workspace, then write back with your own script if that is your pipeline.

### Do the I2 and Dialogue System adapters work?

They compile safely and report absence. They do not read or write data, even with their compile symbols defined. They are scaffolds, not integrations, and have not been tested against licensed installs.

### Is there a CLI for CI?

No. There is no `ReleaseGateCLI`. You can run the EditMode tests in batch mode and write a custom `-executeMethod` entry point that calls `ReleaseGateEvaluator`. See [CI Release Gates]({{ site.baseurl }}/docs/workflows/ci-release-gates/).

### Does the gate check placeholders or glossary terms?

No. It checks approval state, visual errors, stale voice and subtitle timing/CPS. `BlockOnGlossaryViolations` exists but is not read. Run `TextRuleEngine` yourself.

### Does the gate check missing audio or duration mismatch?

No. It only checks stale voice bindings. Missing and duration-mismatched takes are displayed in the Voice window but not gated.

### Does the gate require every locale to be captured?

It requires no `Unknown` or `Registered` tiles when `BlockOnUnvisitedCaptures` is true (the default). A `Failed` tile is not treated as unvisited, so track failures separately.

### Are capture screenshots real?

No. The PNG is a solid background colour. Findings come from live RectTransform and font measurements. There is no player capture.

### Can I trust the safe-area findings?

Treat them as heuristic. The engine compares screen-space safe area with local rect coordinates without conversion.

### Does it support RTL or Arabic shaping?

It can display RTL text and report missing glyphs. It does not shape, mirror layouts or certify linguistic correctness. The pseudolocalization helper reverses characters, which is not a shaping test.

### Can it auto-translate and auto-approve?

No. Advisory drafts are validated suggestions only. Nothing transitions to `Approved` without a human action.

### Can I roll back an import after restarting Unity?

No. The rollback journal is session-only. Use version control for durable rollback.

### Is the workspace safe to commit?

It contains every source string, translation and approval event. Commit it only if your repository access matches the confidentiality of the game text.

### What does the diagnostics export contain?

Unity version, OS, graphics API, workspace counts and provider status. No strings, IDs, paths or secrets. See [Support & Diagnostics]({{ site.baseurl }}/docs/support-diagnostics/).

### Where do I buy it and where do I report issues?

Purchase LocalizeShip only through its Unity Asset Store product page. The documentation repository is not a product download.

Issues: <https://github.com/DecNet-Games/LocalizeShip-DOC/issues>

---

## Related pages

- [Limitations]({{ site.baseurl }}/docs/limitations/)
- [Testing & Verification]({{ site.baseurl }}/docs/testing-and-verification/)
- [Getting Started]({{ site.baseurl }}/docs/getting-started/)
- [Troubleshooting]({{ site.baseurl }}/docs/troubleshooting/)
