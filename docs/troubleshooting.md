---
layout: default
title: Troubleshooting
nav_order: 15
description: "Common troubleshooting steps, compilation diagnosis, and support diagnostic exports."
permalink: /docs/troubleshooting/
---

# Troubleshooting & Diagnostics
{: .fs-9 }

### Diagnosing Common Setup Issues & Gate Blockers
{: .fs-6 .text-grey-dk-000 }

---

### Issue 1: "The Release Gate reports INCOMPLETE COVERAGE, but there are zero errors!"
* **Cause**: You have registered `CaptureFixture` components in your project that were not executed during the current session.
* **Solution**: In the **Capture Matrix** window, click **Run All Fixtures**. To pass a strict coverage gate, 100% of declared fixtures must be visited. If a fixture is obsolete, remove it from the workspace fixture list.

---

### Issue 2: "Leading apostrophe `'` appears on exported strings starting with `=` or `@`"
* **Cause**: This is LocalizeShip's deliberate **Spreadsheet Formula Injection Protection**. It protects translators from Excel/Google Sheets executing formulas or CSV injection exploits.
* **Solution**: This is expected behavior. When you re-import the translated CSV via **Review Imports**, LocalizeShip's RFC 4180 parser cleanly and automatically strips the transport apostrophe, restoring 100% bit-accurate text into your Unity tables.

---

### Issue 3: "Voice take is flagged VoiceStale even though the WAV file was not deleted"
* **Cause**: The source English text or target translation was modified after the audio clip was recorded.
* **Solution**: In **Voice & Subtitles**, inspect the bound source hash. If the script change was minor and the existing audio is still valid, click **Re-Approve Audio for Current Revision**. If the script change altered dialogue, record a pickup take and assign the new audio file.

---

### Issue 4: "Compilation error: The type or namespace name 'Localization' does not exist"
* **Cause**: The official Unity Localization package (`com.unity.localization` 1.5.13+) is not installed.
* **Solution**: Open **Window > Package Manager**, switch to **Unity Registry**, search for **Localization**, and click **Install**.

---

### Issue 5: "Exporting Support Diagnostics"
If you encounter an unexpected issue or edge case:
1. Go to **Tools > Decnet > LocalizeShip > Documentation**.
2. Click **Export Support Diagnostics**.
3. LocalizeShip generates a redacted diagnostic report containing package versions, engine environment, and workspace statistics (with private dialogue text automatically redacted) to share with customer support.
