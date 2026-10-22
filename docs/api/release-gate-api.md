---
layout: default
title: Release Gate API
parent: Architecture & API
nav_order: 4
description: "C# reference for evaluating release readiness gates and generating HTML/JSON reports."
permalink: /docs/api/release-gate-api/
---

# Release Gate API
{: .fs-9 }

### Programmatic Gate Evaluation & Automated Audit Reporting
{: .fs-6 .text-grey-dk-000 }

---

Teams integrating LocalizeShip into custom build pipelines can use the `ReleaseGateEvaluator` API to evaluate policies and export audit reports programmatically.

---

## 1. `ReleaseGateEvaluator`

Located in `Decnet.LocalizeShip.Core.Services`:

```csharp
namespace Decnet.LocalizeShip.Core.Services
{
    public static class ReleaseGateEvaluator
    {
        public static ReleaseGateReport Evaluate(
            ReleaseWorkspace workspace, 
            MatrixRunReport visualReport, 
            GatePolicy policy);

        public static void ExportHtmlReport(
            ReleaseGateReport report, 
            string outputPath);

        public static void ExportJsonReport(
            ReleaseGateReport report, 
            string outputPath);
    }
}
```

---

## 2. Example: Custom Pre-Build Step

You can enforce release gates before creating a Unity player build:

```csharp
#if UNITY_EDITOR
using UnityEditor;
using UnityEditor.Build;
using UnityEditor.Build.Reporting;
using Decnet.LocalizeShip.Core.Services;
using Decnet.LocalizeShip.Core.Models;

public class PreBuildGateCheck : IPreprocessBuildWithReport
{
    public int callbackOrder => 0;

    public void OnPreprocessBuild(BuildReport report)
    {
        var workspace = AssetDatabase.LoadAssetAtPath<ReleaseWorkspace>(
            "Assets/Decnet/LocalizeShip/Workspaces/MainWorkspace.asset");

        var gateResult = ReleaseGateEvaluator.Evaluate(workspace, null, workspace.Policy);

        if (gateResult.OverallStatus != GateStatus.Passed)
        {
            throw new BuildFailedException(
                $"Build aborted: Localization Release Gate BLOCKED. " +
                $"Reasons: {string.Join(", ", gateResult.BlockingReasons)}");
        }
    }
}
#endif
```
