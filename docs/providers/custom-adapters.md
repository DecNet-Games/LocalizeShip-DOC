---
layout: default
title: Custom Adapters
parent: Localization Providers
nav_order: 4
description: "Implement ILocalizationProvider for your own string pipeline and register it with LocalizeShip."
permalink: /docs/providers/custom-adapters/
---

# Custom Adapters
{: .fs-9 }

### Bring your own string database
{: .fs-6 .text-grey-dk-000 }

If your strings live in JSON, XML, a database or an in-house editor, you can implement `ILocalizationProvider` and feed the same workspace, handoff and gate pipeline.

![Provider architecture]({{ site.baseurl }}/assets/images/15-existing-provider.png)

*The capture shows the optional I2 Localization and Dialogue System adapters absent; it is not licensed-install verification.*

---

## The interface, again

```csharp
public interface ILocalizationProvider
{
    string ProviderId { get; }
    string DisplayName { get; }
    bool IsInstalled { get; }
    string VersionString { get; }

    List<string> GetAvailableCollectionNames();
    List<SourceRevision> ReadCollection(
        string collectionName, string sourceLocale, NormalizationPolicy policy);
    string GetTranslation(
        string collectionName, StableEntryId entryId, LocaleId locale);
    void WriteTranslation(
        string collectionName, StableEntryId entryId, LocaleId locale, string newText);
}
```

The interface lives in `Decnet.LocalizeShip.Adapters`. Implement it in an **editor** assembly, because the pipeline writes assets and marks them dirty.

---

## A JSON adapter, end to end

Assume files like:

```json
{
  "locale": "en",
  "entries": [
    {
      "key": "ui.play_button",
      "text": "Play",
      "speaker": "System",
      "context": "Main menu"
    }
  ]
}
```

### The adapter

```csharp
using System.Collections.Generic;
using System.IO;
using Decnet.LocalizeShip.Adapters;
using Decnet.LocalizeShip.Core.Models;
using UnityEngine;

public class JsonProvider : ILocalizationProvider
{
    private readonly string _root;

    public JsonProvider(string root) { _root = root; }

    public string ProviderId => "JsonFiles";
    public string DisplayName => "JSON String Files";
    public bool IsInstalled => Directory.Exists(_root);
    public string VersionString => "1";

    public List<string> GetAvailableCollectionNames()
    {
        var list = new List<string>();
        if (!Directory.Exists(_root)) return list;
        foreach (var f in Directory.GetFiles(_root, "*.json"))
            list.Add(Path.GetFileNameWithoutExtension(f));
        return list;
    }

    private string PathFor(string collection, string locale) =>
        Path.Combine(_root, $"{collection}.{locale}.json");

    public List<SourceRevision> ReadCollection(
        string collection, string sourceLocale, NormalizationPolicy policy)
    {
        var results = new List<SourceRevision>();
        string path = PathFor(collection, sourceLocale);
        if (!File.Exists(path)) return results;

        var file = JsonUtility.FromJson<JsonTable>(File.ReadAllText(path));
        foreach (var e in file.entries)
        {
            results.Add(new SourceRevision(
                new StableEntryId(e.key), e.text, e.speaker, e.context, policy));
        }
        return results;
    }

    public string GetTranslation(
        string collection, StableEntryId entryId, LocaleId locale)
    {
        string path = PathFor(collection, locale.Code);
        if (!File.Exists(path)) return null;
        var file = JsonUtility.FromJson<JsonTable>(File.ReadAllText(path));
        foreach (var e in file.entries)
            if (e.key == entryId.Value) return e.text;
        return null;
    }

    public void WriteTranslation(
        string collection, StableEntryId entryId, LocaleId locale, string newText)
    {
        string path = PathFor(collection, locale.Code);
        JsonTable file = File.Exists(path)
            ? JsonUtility.FromJson<JsonTable>(File.ReadAllText(path))
            : new JsonTable { locale = locale.Code, entries = new List<JsonEntry>() };

        var found = file.entries.Find(e => e.key == entryId.Value);
        if (found != null) found.text = newText;
        else file.entries.Add(new JsonEntry { key = entryId.Value, text = newText });

        File.WriteAllText(path, JsonUtility.ToJson(file, true));
    }

    [System.Serializable] private class JsonTable
    {
        public string locale;
        public List<JsonEntry> entries = new List<JsonEntry>();
    }

    [System.Serializable] private class JsonEntry
    {
        public string key;
        public string text;
        public string speaker;
        public string context;
    }
}
```

---

## Register it

Discovery is a code list, not a plugin scan:

```csharp
public static class MyProviderRegistration
{
    public static List<ProviderInfo> DiscoverAll()
    {
        var list = ProviderDiscoveryService.DiscoverProviders();
        var json = new JsonProvider("Assets/Localization");
        list.Add(new ProviderInfo
        {
            ProviderId = json.ProviderId,
            DisplayName = json.DisplayName,
            IsInstalled = json.IsInstalled,
            Version = json.VersionString,
            Collections = json.GetAvailableCollectionNames()
        });
        return list;
    }
}
```

To change Guided Setup, patch your own copy of its provider loop (or your project's fork). The shipped window calls `ProviderDiscoveryService.DiscoverProviders()` directly.

---

## Ingest revisions safely

Do not blindly add duplicates. Use the tracker so changed source lines mark translations stale:

```csharp
var tracker = new RevisionTracker();
var provider = new JsonProvider("Assets/Localization");

foreach (var rev in provider.ReadCollection("UI", "en", workspace.Normalization))
{
    var current = workspace.GetSourceRevision(rev.EntryId);
    if (current == null)
        workspace.SourceRevisions.Add(rev);
    else if (current.RevisionHash != rev.RevisionHash)
        tracker.ProcessSourceEntry(
            workspace, rev.EntryId, rev.RawText, rev.Speaker, rev.ContextDescription);
}
```

`ProcessSourceEntry` updates the existing revision, invalidates approvals, and rechecks bound voice entries. Review the [Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/) before wiring this into a build step.

---

## Design rules

1. **Stable keys.** Never use array indices or auto-increment IDs that can change.
2. **Deterministic reading.** The same file should produce the same hashes. Avoid timestamps in context fields.
3. **No hidden normalization.** Let the workspace's `NormalizationPolicy` do it; do not trim twice with different rules.
4. **Fail visibly on write.** If a collection or locale is missing, throw rather than silently doing nothing.
5. **No approval logic in the adapter.** The adapter moves text; review state belongs to the workspace.
6. **No network in the adapter.** Keep ingestion local and auditable.

---

## Testing your adapter

- Read the same collection twice and compare hashes — they must match.
- Rename a key and confirm the workspace sees a new entry, not a silent overwrite.
- Change a source line and confirm existing approvals become `Stale`.
- Write a translation and confirm the file is valid JSON/XML and the entry is found on re-read.
- Run the read path on a missing collection and confirm it fails or returns empty deliberately.

See [Testing & Verification]({{ site.baseurl }}/docs/testing-and-verification/) for the package's own acceptance coverage.

---

## Related pages

- [Localization Providers]({{ site.baseurl }}/docs/providers/)
- [Unity Localization]({{ site.baseurl }}/docs/providers/unity-localization/)
- [Stable Identities]({{ site.baseurl }}/docs/concepts/stable-identities/)
- [Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/)
