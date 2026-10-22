---
layout: default
title: Support & Diagnostics
nav_order: 13
description: "The built-in diagnostics export: exact contents, where it writes, and what it deliberately omits."
permalink: /docs/support-diagnostics/
---

# Support & Diagnostics
{: .fs-9 }

### Counts, versions and environment — nothing else
{: .fs-6 .text-grey-dk-000 }

The diagnostics export is designed to be safe to attach to a support ticket. It contains environment metadata and workspace counters. It does not contain your strings.

---

## Export

1. Open the Release Board.
2. Select a workspace.
3. In the left filter panel, click **Export Support Diagnostics**.
4. A dialog shows the saved path.

`DiagnosticsExporter.ExportDiagnostics(workspace, projectRoot)` writes:

```
LocalizeShip_Diagnostics_<yyyyMMdd_HHmmss>.txt
```

The timestamp is UTC. The Studio passes `Application.dataPath + "/../"`, so the file lands in the project root.

---

## Exact contents

```text
=== LocalizeShip Support Diagnostics Bundle ===
Timestamp (UTC): 2026-10-22 09:30:00
Unity Editor: 6000.5.4f1
Operating System: <SystemInfo.operatingSystem>
Graphics API: <SystemInfo.graphicsDeviceType>

=== Release Workspace Metadata ===
Workspace ID: <id>
Release Version: <version>
Source Locale: en
Target Locales: de, ja, ar
Source Lines: <count>
Approvals Recorded: <count>
Voice Entries: <count>
Subtitle Cues: <count>
Capture Results: <count>
Suppressions: <count>

=== Installed Provider Packages ===
- Unity Localization (Official): Installed=True, Version=1.5.13, Collections=<n>
- I2 Localization (Optional Adapter): Installed=False, Version=Not Installed, Collections=0
- Pixel Crushers Dialogue System (Optional Adapter): Installed=False, Version=Not Installed, Collections=0
```

If no workspace is selected, that section says `No active workspace selected.` and the provider section is still written.

---

## What it does not contain

- source text, translations, speaker names or context notes;
- entry IDs or keys;
- file paths for audio or screenshots;
- API keys or endpoint URLs;
- stack traces or log files;
- approval history or reviewer names.

There is no redaction step because no narrative data is collected in the first place.

{: .note }
> Provider `Version` values are adapter properties. Unity Localization reports the hard-coded `"1.5.13"`, and optional adapters report `"Detected"` when their compile symbol is set. The diagnostics file is not a package-manager inventory.

---

## Privacy properties

| Property | Status |
|---|---|
| Network upload | None — the file is written locally |
| Telemetry | None in the package |
| Contains project strings | No |
| Contains secrets | No (no secret storage exists) |
| Human-readable | Yes, UTF-8 plain text |

Review the file before sending it. Counts can still reveal project scale, which may be sensitive under some contracts.

---

## Filing an issue

Include:

1. the diagnostics file;
2. Unity version and Localization package version;
3. exact reproduction steps;
4. the window and button used;
5. relevant console errors (copy the message, not your whole project log);
6. whether the sample reproduces the issue.

Open issues at <https://github.com/DecNet-Games/LocalizeShip-DOC/issues>.

Do not attach your workspace asset unless support asks for it. It contains every string and approval in the project.

---

## Related pages

- [Troubleshooting]({{ site.baseurl }}/docs/troubleshooting/)
- [Data & Privacy]({{ site.baseurl }}/docs/data-and-privacy/)
- [Security & Privacy]({{ site.baseurl }}/docs/security-and-privacy/)
- [FAQ]({{ site.baseurl }}/docs/faq/)
