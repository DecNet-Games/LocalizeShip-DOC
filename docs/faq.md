---
layout: default
title: Frequently Asked Questions
nav_order: 16
description: "Frequently asked questions regarding licensing, workflows, engine compatibility, and architecture."
permalink: /docs/faq/
---

# Frequently Asked Questions (FAQ)
{: .fs-9 }

### Common Inquiries Regarding Architecture, Licensing & Workflows
{: .fs-6 .text-grey-dk-000 }

---

### Q: Does LocalizeShip replace my existing localization or dialogue system?
**No.** LocalizeShip is an **auditable release studio and visual QA layer** that sits on top of your existing localization tool. Your string data continues to reside in standard Unity Localization tables (`StringTableCollection`), I2 terms, or Dialogue System databases.

---

### Q: Does LocalizeShip require a monthly subscription or cloud account?
**No.** LocalizeShip is a standard commercial Unity Asset purchased once from the [Unity Asset Store](https://assetstore.unity.com/packages/slug/421016). It runs 100% locally on your machine with zero cloud lock-in, zero telemetry, and zero mandatory external servers.

---

### Q: Does LocalizeShip require an internet connection or an AI service?
**No.** LocalizeShip is fully functional completely offline. All revision tracking, impact analysis, 3-way merges, visual QA geometry measurements, and release gates execute entirely inside your local Unity Editor. The Advisory Drafts module is strictly optional.

---

### Q: How does LocalizeShip know if a translation is stale?
LocalizeShip normalizes the source text and calculates a cryptographic **SHA-256 hash** over the normalized text, context, and placeholders. When you approve a translation, the approval event permanently records that specific hash. If the source text is edited in Unity, the new hash will not match the approval record, automatically transitioning the translation to **`Stale`**.

---

### Q: Can I run LocalizeShip release gates headlessly in CI/CD pipelines?
**Yes.** LocalizeShip includes a command-line interface (`ReleaseGateCLI.EvaluateGates`) that can be executed via Unity's batchmode in GitHub Actions, GitLab CI, or Jenkins. It outputs standalone HTML reports, machine-readable JSON manifests, and standard process exit codes (0 for pass, 1 for fail).

---

### Q: What versions of Unity are supported?
LocalizeShip was developed and rigorously verified against **Unity 6000.5.4f1 LTS**. It is fully compatible with **Unity 2022.3 LTS**, **Unity 2023**, and all **Unity 6** releases.

---

### Q: How does LocalizeShip protect against spreadsheet formula injection?
When exporting CSV files for external translators, `HandoffCsvSerializer` automatically prepends a transport apostrophe (`'`) to any string that begins with formula trigger characters (`=`, `+`, `-`, `@`, `\t`, `\r`). When the translated CSV is re-imported, the parser cleanly strips the transport character, guaranteeing data integrity.
