---
layout: default
title: Custom Adapters
parent: Localization Providers
nav_order: 4
description: "Authoring custom adapters for in-house narrative databases, XML, JSON, or custom table structures."
permalink: /docs/providers/custom-adapters/
---

# Authoring Custom Adapters
{: .fs-9 }

### Connecting Proprietary Narrative & String Engines
{: .fs-6 .text-grey-dk-000 }

---

If your studio uses an internal narrative engine, a custom SQLite database, or proprietary JSON localization files, you can connect it directly to LocalizeShip by implementing `ILocalizationProviderAdapter`.

---

## The `ILocalizationProviderAdapter` Interface

```csharp
namespace Decnet.LocalizeShip.Core.Adapters
{
    public interface ILocalizationProviderAdapter
    {
        string ProviderName { get; }
        bool IsAvailable { get; }
        
        System.Collections.Generic.IEnumerable<CollectionDescriptor> DiscoverCollections();
        System.Collections.Generic.IEnumerable<SourceLineData> ReadEntries(CollectionDescriptor collection);
        bool ApplyPlan(MergePlan plan, out string error);
    }
}
```

---

## Example: Building a Simple JSON Adapter

```csharp
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using Decnet.LocalizeShip.Core.Adapters;
using Decnet.LocalizeShip.Core.Models;

public class CustomJsonAdapter : ILocalizationProviderAdapter
{
    public string ProviderName => "StudioCustomJson";
    public bool IsAvailable => true;

    public IEnumerable<CollectionDescriptor> DiscoverCollections()
    {
        yield return new CollectionDescriptor("JSON_Dialogue", "Assets/Data/dialogue.json");
    }

    public IEnumerable<SourceLineData> ReadEntries(CollectionDescriptor collection)
    {
        string rawJson = File.ReadAllText(collection.Path);
        var entries = JsonUtility.FromJson<JsonStringList>(rawJson);

        foreach (var item in entries.items)
        {
            yield return new SourceLineData(
                new StableEntryId(item.guid),
                item.key,
                item.englishText,
                item.speakerName,
                item.contextNotes
            );
        }
    }

    public bool ApplyPlan(MergePlan plan, out string error)
    {
        // Write approved incoming translations back to JSON file
        error = null;
        return true;
    }
}
```

Once implemented and registered, your custom data source will appear directly inside the **Guided Setup** and **Release Board** windows alongside native Unity tables.
