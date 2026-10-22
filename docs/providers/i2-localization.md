---
layout: default
title: I2 Localization
parent: Localization Providers
nav_order: 2
description: "The compile-isolated I2 Localization adapter: what the flag changes, what the methods currently do, and how to finish the integration."
permalink: /docs/providers/i2-localization/
---

# I2 Localization Adapter
{: .fs-9 }

### Compile isolation, not a finished integration
{: .fs-6 .text-grey-dk-000 }

The I2 Localization adapter exists so projects that use InterIllusion's asset can compile LocalizeShip without the I2 package installed. That is the verified claim. The adapter is a scaffold, not a working two-way bridge.

---

## What is verified

| Claim | Status |
|---|---|
| Compiles without I2 installed | **Verified** by the acceptance suite |
| Reports `IsInstalled = false` without the flag | **Verified** |
| `WriteTranslation` throws a clear `NotSupportedException` without the flag | **Verified** |
| Compiles with `LOCALIZESHIP_I2_SUPPORT` defined | **Verified** (no unresolved I2 symbols, because no I2 types are referenced) |
| Reads or writes real I2 terms | **Not implemented** |
| Tested against a licensed I2 installation | **Not claimed** |

The adapter does not reference any type from the I2 namespace. Defining the symbol does not unlock hidden code paths.

---

## Behaviour with the flag off

```csharp
public bool IsInstalled => false;
public string VersionString => "Not Installed";

public List<string> GetAvailableCollectionNames() => new List<string>();
public List<SourceRevision> ReadCollection(...) => new List<SourceRevision>();
public string GetTranslation(...) => null;

public void WriteTranslation(...)
{
    throw new NotSupportedException(
        "I2 Localization is not installed in this project. " +
        "Please install I2 Localization to use this adapter.");
}
```

Guided Setup shows the card as **Not Installed** with the message that the optional package is absent and the adapter compiles cleanly.

## Behaviour with `LOCALIZESHIP_I2_SUPPORT` defined

```csharp
public bool IsInstalled => true;
public string VersionString => "Detected";
```

The rest is unchanged in 1.0.0:

- `GetAvailableCollectionNames` returns an empty list;
- `ReadCollection` returns an empty list;
- `GetTranslation` returns `null`;
- `WriteTranslation` is an empty method.

{: .warning }
> With the flag on, the card claims “Active (Detected)” but no data flows. Do not enable the symbol expecting a working import or export.

---

## Why ship a scaffold at all?

- It proves the provider boundary is real and can be isolated at compile time.
- It gives integrators a named starting point with the correct interface shape.
- It keeps the core package free of third-party dependencies and licence obligations.
- It avoids shipping or redistributing proprietary code.

A scaffold that is honest about being a scaffold is more useful than a broken “integration” with marketing claims attached.

---

## Finishing the integration

To make it real, you would:

1. Add the I2 assembly to a new adapter assembly (or to your own editor assembly), with an explicit reference.
2. Add a version define or asmdef constraint so the reference only resolves when I2 is present.
3. Implement collection discovery against I2's term sources.
4. Map I2 term keys to `StableEntryId` — use the term key, not a transient index.
5. Populate `SourceRevision` with speaker/context where I2 provides them.
6. Implement `GetTranslation` and `WriteTranslation`, marking changed assets dirty.
7. Remove the empty `#if` branches and add tests against a licensed installation in your own project.

### Skeleton

```csharp
#if LOCALIZESHIP_I2_SUPPORT
using I2.Loc;

public List<string> GetAvailableCollectionNames()
{
    var list = new List<string>();
    foreach (var source in LocalizationManager.Sources)
        list.Add(source.mSourceName);   // confirm the API for your I2 version
    return list;
}
#endif
```

{: .note }
> I2's API surface varies across versions. Confirm type and member names against the version you own. The line above is illustrative, not compiled or verified by Decnet Games.

---

## Recommended project setup

- Keep LocalizeShip's bundled adapters untouched.
- Put your I2 adapter in your own editor assembly.
- Register it in your own discovery step rather than editing the package (package updates would overwrite your changes).
- If you need `GuidedSetupWindow` to list it, either patch discovery in your project or accept that the bundled window lists only the three built-in provider IDs.

---

## What not to claim

- That LocalizeShip “supports I2 out of the box.”
- That the adapter was tested with a licensed I2 install.
- That enabling the symbol enables translation sync.
- That any I2 terms are read or written by the shipped code.

The accurate sentence is: “A compile-isolated I2 adapter scaffold is included; it compiles and reports provider-absent status safely, and does not yet implement I2 data access.”

---

## Related pages

- [Localization Providers]({{ site.baseurl }}/docs/providers/)
- [Custom Adapters]({{ site.baseurl }}/docs/providers/custom-adapters/)
- [Testing & Verification]({{ site.baseurl }}/docs/testing-and-verification/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
