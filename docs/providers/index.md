---
layout: default
title: Localization Providers
nav_order: 7
has_children: true
description: "The provider adapter contract, what each bundled adapter actually implements in 1.0.0, and how to add your own."
permalink: /docs/providers/
---

# Localization Providers
{: .fs-9 }

### One contract, three adapters, clear boundaries
{: .fs-6 .text-grey-dk-000 }

LocalizeShip reads and writes strings through a small adapter interface. The bundled Unity Localization adapter is implemented. The I2 Localization and Dialogue System adapters are compile-isolated scaffolds, not finished integrations.

![Existing provider architecture]({{ site.baseurl }}/assets/images/15-existing-provider.png)

---

## The contract

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

Four properties describe the provider. Five methods move data in and out.

---

## Status matrix

| Provider | `ProviderId` | Implemented? | What discovery shows |
|---|---|---|---|
| Unity Localization | `UnityLocalization` | Yes — collections, read, get, write | Always “Active (1.5.13)” |
| I2 Localization | `I2Localization` | No — method bodies are scaffolds | “Not Installed” unless `LOCALIZESHIP_I2_SUPPORT` is defined |
| Pixel Crushers Dialogue System | `PixelCrushersDialogueSystem` | No — method bodies are scaffolds | “Not Installed” unless `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` is defined |
| Your own adapter | any unique string | You write it | Only if you add it to discovery |

{: .warning }
> Defining `LOCALIZESHIP_I2_SUPPORT` or `LOCALIZESHIP_PIXELCRUSHERS_SUPPORT` changes `IsInstalled` to true, but the collection list is still empty, `GetTranslation` still returns `null`, and `WriteTranslation` becomes a no-op. Do not present these adapters as working integrations.

---

## Where adapters are used

`ProviderDiscoveryService.DiscoverProviders()` instantiates all three adapters and calls `GetAvailableCollectionNames()` for each. Guided Setup displays the results and lets you choose a provider ID, which is stored in `ReleaseWorkspace.ActiveProviderType`.

That is the only place the package calls an adapter. In 1.0.0:

- source revisions are not imported from a provider by any window;
- approvals are not written back to a provider by any window;
- Review Imports applies to the workspace, not to Unity tables.

The Unity adapter is usable **from your own editor scripts**. See the [Unity Localization]({{ site.baseurl }}/docs/providers/unity-localization/) page for its exact behaviour.

---

## Choosing a provider ID

`GuidedSetupWindow` writes the selected `ProviderId` into the workspace:

```csharp
workspace.ActiveProviderType = _selectedProvider;
```

Nothing validates that the ID corresponds to an installed adapter at runtime. It is a label used for bookkeeping and display. If you implement a custom ingestion path, read `ActiveProviderType` to decide which adapter to call, and fail loudly if it is unknown.

---

## Adding a provider

1. Implement `ILocalizationProvider` in an editor assembly that references `Decnet.LocalizeShip.Core`.
2. Add your instance to `ProviderDiscoveryService.DiscoverProviders()`.
3. If you want it selectable, make `GetAvailableCollectionNames()` return real names.
4. If you want two-way sync, call `ReadCollection` / `WriteTranslation` from your own import/apply step.
5. Keep provider-specific types out of the Core assembly so the domain stays portable.

There is no plugin folder, reflection scan or service locator. Registration is a code change.

---

## The identity rule

Whatever the provider, the adapter must return a `SourceRevision` with a `StableEntryId` built from a key that survives UI edits. If the underlying system renumbers entries, the adapter is responsible for mapping to a stable string.

For Unity Localization, the adapter uses the shared table **key**, not the numeric ID. Renaming the key creates a new identity and resets approvals for that line.

---

## Provider guides

- [Unity Localization]({{ site.baseurl }}/docs/providers/unity-localization/) — implemented adapter, with its real limitations.
- [I2 Localization]({{ site.baseurl }}/docs/providers/i2-localization/) — compile flag and scaffold status.
- [Dialogue System]({{ site.baseurl }}/docs/providers/dialogue-system/) — compile flag and scaffold status.
- [Custom Adapters]({{ site.baseurl }}/docs/providers/custom-adapters/) — implement and register your own.

---

## Related pages

- [Stable Identities]({{ site.baseurl }}/docs/concepts/stable-identities/)
- [Installation]({{ site.baseurl }}/docs/installation/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
