---
layout: default
title: Data & Privacy Policy
nav_order: 14
description: "Data retention, offline operation, and security architecture of LocalizeShip."
permalink: /docs/data-and-privacy/
---

# Data & Privacy Policy
{: .fs-9 }

### Local-First Architecture & Zero Data Leakage
{: .fs-6 .text-grey-dk-000 }

---

LocalizeShip is built with strict privacy and security standards suitable for commercial enterprise studios, NDAs, and confidential narrative IPs.

---

## 1. 100% Local-First Operation

* **No Cloud Servers**: LocalizeShip does not operate any remote servers, hosted databases, or telemetry aggregators.
* **No Tracking or Telemetry**: LocalizeShip contains zero analytics libraries, crash telemetry pings, or usage tracking.
* **On-Disk Serialization**: All release workspaces, revision graphs, approval events, and journals serialize cleanly into standard Unity `ScriptableObject` assets within your project's `Assets/` directory.

---

## 2. API Key Security (Advisory Module)

If you choose to use the optional Bring-Your-Own-Endpoint Advisory module:
* **Storage Location**: Endpoint URLs and API keys are stored exclusively in local Editor user preferences (`EditorPrefs`), which reside outside the project repository.
* **No Git Leaks**: API keys are **never written** to `ScriptableObject` assets, scene files, CSV exports, or release reports.
* **No Unsolicited Requests**: LocalizeShip will never send text to an external endpoint without explicit user interaction and configuration.

---

## 3. Redacted Support Diagnostics

When generating a diagnostic export for customer support:
* All game narrative text, character dialogue, and proprietary strings are **automatically redacted**.
* The diagnostic package includes only environment metadata (Unity version, OS, package versions, and error stack traces).
