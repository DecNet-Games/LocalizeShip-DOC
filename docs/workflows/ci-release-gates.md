---
layout: default
title: CI/CD Release Gates
parent: Production Workflows
nav_order: 3
description: "Automating headless release gate evaluation in GitHub Actions, GitLab CI, and Jenkins."
permalink: /docs/workflows/ci-release-gates/
---

# CI/CD Release Gates
{: .fs-9 }

### Headless Build Verification & Automated Pull Request Guard Rails
{: .fs-6 .text-grey-dk-000 }

---

Preventing localization regressions from entering your main branch requires automated verification. You can run LocalizeShip's release gate engine headlessly in your continuous integration pipeline.

---

## Command-Line Interface (CLI)

Execute Unity in batchmode and invoke LocalizeShip's headless gate runner:

```bash
Unity.exe -batchmode -quit -nographics \
  -projectPath "$PROJECT_DIR" \
  -executeMethod Decnet.LocalizeShip.Editor.ReleaseGateCLI.EvaluateGates \
  -workspacePath "Assets/Decnet/LocalizeShip/Workspaces/MainReleaseWorkspace.asset" \
  -reportOutputPath "Build/Reports/GateReport.html" \
  -jsonOutputPath "Build/Reports/GateReport.json" \
  -logFile "Build/Logs/localizeship_ci.log"
```

### Exit Codes
* **`0`**: Gate Passed. All required locales are approved; zero critical visual overflows; 100% fixture coverage.
* **`1`**: Gate Blocked. One or more release gate policies were violated.
* **`2`**: Invalid parameters or missing workspace file.

---

## Example: GitHub Actions Workflow

Here is a production-ready `.github/workflows/localization-gate.yml` workflow:

```yaml
name: Localization Release Gate

on:
  pull_request:
    branches: [ main, release/* ]
  push:
    branches: [ main ]

jobs:
  evaluate-gates:
    runs-on: windows-2022
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: Cache Unity Library
        uses: actions/cache@v4
        with:
          path: Library
          key: Library-${{ hashFiles('Assets/**', 'Packages/**') }}
          restore-keys: Library-

      - name: Run LocalizeShip Release Gates
        run: |
          & "C:\Program Files\Unity\Hub\Editor\6000.5.4f1\Editor\Unity.exe" `
            -batchmode -quit -nographics `
            -projectPath "." `
            -executeMethod Decnet.LocalizeShip.Editor.ReleaseGateCLI.EvaluateGates `
            -workspacePath "Assets/Decnet/LocalizeShip/Workspaces/MainReleaseWorkspace.asset" `
            -reportOutputPath "Build/Reports/GateReport.html" `
            -jsonOutputPath "Build/Reports/GateReport.json" `
            -logFile "unity_ci.log"

      - name: Archive QA Reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: Localization-Gate-Report
          path: Build/Reports/
```
