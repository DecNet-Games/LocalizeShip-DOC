---
layout: default
title: Revision Graph
parent: Concepts
nav_order: 2
description: "How source revisions are normalized, hashed with SHA-256, and used to bind approvals, voice takes and merge bases."
permalink: /docs/concepts/revision-graph/
---

# The Source Revision Graph
{: .fs-9 }

### The hash is the review contract
{: .fs-6 .text-grey-dk-000 }

A translation review is meaningless unless you know *what* was reviewed. “German is approved” is a sentence with a missing clause: approved for which English sentence, with which speaker and context, under which normalization rules?

LocalizeShip answers that by turning the source into a revision hash and binding everything to it.

![Revision identity]({{ site.baseurl }}/assets/images/08-revision-identity.png)

---

## What a source revision contains

`SourceRevision` stores:

```csharp
public StableEntryId EntryId;
public SourceRevisionHash RevisionHash;
public string RawText;
public string NormalizedText;
public string Speaker;
public string ContextDescription;
public string SceneOrFixture;
public int NormalizationVersion;
public long TimestampTicks;
public List<string> Placeholders;
```

The fields that feed the hash are **normalized text**, **speaker** and **context description**, plus the normalization policy version.

---

## How the hash is computed

`NormalizationPolicy.ComputeHash` builds a payload and runs SHA-256:

```csharp
string payload = $"{Version}|{normalizedText}|{context ?? string.Empty}";

using (var sha = SHA256.Create())
{
    byte[] bytes = sha.ComputeHash(Encoding.UTF8.GetBytes(payload));
    var sb = new StringBuilder(16);
    for (int i = 0; i < 8; i++)
    {
        sb.Append(bytes[i].ToString("x2"));
    }
    return sb.ToString();
}
```

And the constructor passes the combined context:

```csharp
string combinedContext = $"{Speaker}:{ContextDescription}";
string hash = policy.ComputeHash(NormalizedText, combinedContext);
```

So the effective payload is:

```
<policyVersion>|<normalizedText>|<speaker>:<contextDescription>
```

The digest is truncated to the first **8 bytes** and rendered as **16 lowercase hex characters**. It is a compact collaboration key, not a cryptographic signature.

{: .note }
> The prompt-level description says “SHA-256 of the normalized text”. The implementation is more precise: SHA-256 over a versioned payload that includes normalized text, speaker and context. That is intentional — changing the speaker or context is a real review change even when the words are identical.

---

## What changes the hash

| Change | Hash changes? |
|---|---|
| Source text changes | Yes |
| Speaker changes | Yes |
| Context description changes | Yes |
| Normalization policy version changes | Yes |
| Whitespace at the ends (with default policy) | No |
| CRLF vs LF (with default policy) | No |
| Reordering entries | No |
| Moving an entry without changing its key | No |
| Translation text changes | No — that is the translation hash |
| Voice clip changes | No |
| Screenshot changes | No |

---

## One current revision, plus event history

The name “graph” is aspirational. In 1.0.0 the workspace stores **one current `SourceRevision` per entry**:

```csharp
public SourceRevision GetSourceRevision(StableEntryId entryId)
{
    for (int i = 0; i < SourceRevisions.Count; i++)
    {
        if (SourceRevisions[i].EntryId == entryId) return SourceRevisions[i];
    }
    return null;
}
```

When the hash changes, `RevisionTracker.ProcessSourceEntry` removes the old revision and adds the new one. Previous raw text is not retained in the revision list.

What *is* retained is the approval event history. Each `ApprovalEvent` stores:

- the entry and locale;
- the bound source and translation revision hashes at the time of the event;
- the old and new review state;
- reviewer label, role, rationale and timestamp.

That gives you an auditable chain of *decisions* (`Approved → Stale`, with both hashes) even though the full source text history is not stored. If you need complete text archaeology, keep the project under version control as well.

---

## What the revision binds

### Approval
`ApprovalRecord.ApplyReview` stores `ApprovedSourceHash` and `ApprovedTranslationHash` when the new state is `Approved`:

```csharp
if (newState == ReviewState.Approved)
{
    ApprovedSourceHash = currentSourceHash;
    ApprovedTranslationHash = currentTransHash;
}
```

`CheckSourceChange` compares the approved source hash with the new one and, if it differs and the record is currently approved, moves it to `Stale` and appends a system event.

### Voice takes
`VoiceEntry` stores both `BoundSourceHash` and `BoundTranslationHash`. `RecheckStatus` evaluates, in order:

1. no audio path → `Missing`;
2. source hash differs → `StaleSourceRevision`;
3. translation hash differs → `StaleTranslationRevision`;
4. duration difference greater than 2.0 s → `DurationMismatch`;
5. otherwise → `BoundCurrent`.

### Subtitle cues
`SubtitleCue` stores `BoundSourceHash`. The impact engine flags a cue when its bound hash differs from the current source hash.

### Merge base
`HandoffEntry.BaseSourceHash` is the source hash at export time. The merge planner compares it with the current source hash to detect `StaleSourceConflict`.

---

## Translation revisions

Translations get their own hash through `RevisionTracker.ComputeTranslationHash`:

```csharp
public TranslationRevisionHash ComputeTranslationHash(string translationText)
{
    string norm = _policy.Normalize(translationText);
    string hash = _policy.ComputeHash(norm, "translation");
    return new TranslationRevisionHash(hash);
}
```

The context string is the literal `"translation"`. The same normalization policy applies.

{: .warning }
> In the Release Board's manual approval path, `ApprovedTranslationHash` is passed as `default`, not as a computed translation hash. This means voice entries that depend on the translation hash can report `StaleTranslationRevision` after a manual approval. Compute and pass the translation hash in your integration if you rely on that binding.

---

## Worked example

```
Source:   "Collect {count} herbs."
Speaker:  "Herbalist"
Context:  "Quest objective"
Policy:   v1
Payload:  "1|Collect {count} herbs.|Herbalist:Quest objective"
Hash:     "9f2c1a7b3d4e5f60"   (illustrative)
```

A writer changes it to `"Collect {count} rare herbs."`:

```
Payload:  "1|Collect {count} rare herbs.|Herbalist:Quest objective"
Hash:     "41ab90ce77d2015b"   (illustrative)
```

The approval bound to `9f2c…` is no longer valid. It moves to **Stale**, the history gets an event, the impact engine reports the translation, and any voice or subtitle bound to the old hash is flagged.

---

## Integration advice

- **Always process source lines through `RevisionTracker`** rather than writing `SourceRevision` objects directly, if you want invalidation to run.
- **Pass speaker and context** from your provider when you have them. Empty context is valid but produces a weaker review contract.
- **Do not change `NormalizationPolicy.Version` casually.** It is part of the hash and will invalidate every existing approval when new revisions are processed.
- **Store the workspace asset in version control.** The hashes and approval records are the evidence.

Next: [Review States]({{ site.baseurl }}/docs/concepts/review-states/).
