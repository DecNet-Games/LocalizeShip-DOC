---
layout: default
title: Advisory Drafts
parent: Features
nav_order: 13
description: "The advisory validation contract: parse an external suggestion, bind it to an entry, and reject anything that breaks placeholders."
permalink: /docs/features/advisory-drafts/
---

# Advisory Drafts
{: .fs-9 }

### Suggestions are input, not output
{: .fs-6 .text-grey-dk-000 }

LocalizeShip defines a small, strict contract for machine suggestions. It does not call a model, does not ship an HTTP client, and never approves anything. It takes a JSON payload, checks that it belongs to the right entry, validates its placeholders, and returns a proposal you can inspect.

![Release board]({{ site.baseurl }}/assets/images/03-release-board.png)

---

## What ships

| Component | Purpose |
|---|---|
| `AdvisoryConfig` | Endpoint, model, timeout and retry defaults |
| `AdvisoryProvider.ValidateSuggestion` | Parse and validate a raw JSON suggestion |
| `DraftProposal` | Result object with validity and error |

What does **not** ship:

- an HTTP request implementation;
- an API key field that is serialized;
- a chat window or “Generate” button;
- automatic approval or application of suggestions;
- prompt templates, token counting or cost tracking.

The endpoint and model are configuration **defaults**, not evidence of an integration. Nothing contacts them.

---

## Configuration

```csharp
[Serializable]
public class AdvisoryConfig
{
    public string EndpointUrl = "https://api.openai.com/v1/chat/completions";
    public string ModelName = "gpt-4o-mini";
    public int TimeoutSeconds = 15;
    public int MaxRetries = 2;
    [NonSerialized] public string SessionApiKey;
}
```

`SessionApiKey` is marked `[NonSerialized]`, so it is not written into the workspace asset or any serialized object. It lives only for the session.

{: .warning }
> Configuration is not a client. You must implement the network call yourself, and you are responsible for key handling, privacy, cost and provider terms.

The defaults name an OpenAI-compatible chat completions endpoint. That is a convenience for integrations that already use that shape. It is not an endorsement, and the product works without any network access.

---

## The response contract

The validator expects:

```json
{
  "id": "dialogue.greeting.001",
  "suggestion": "Hallo, Reisender!",
  "rationale": "Matches the informal register used in the German UI."
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | Yes, effectively | Must equal `source.EntryId.Value` exactly |
| `suggestion` | Yes | Must be non-empty |
| `rationale` | No | Defaults to `"Advisory AI Draft Proposal"` |

Parsing uses `UnityEngine.JsonUtility.FromJson<AdvisoryResponseSchema>`. Unknown fields are ignored.

---

## Validation pipeline

```csharp
var provider = new AdvisoryProvider(new AdvisoryConfig());
DraftProposal proposal = provider.ValidateSuggestion(
    sourceRevision,
    rawJsonFromYourClient,
    new LocaleId("de"));
```

Steps:

1. **Empty payload** → invalid, `"Empty or null response payload received."`
2. **Parse** → `JsonUtility` is lenient; missing `suggestion` → invalid, `"Malformed suggestion payload; missing 'suggestion' field."`
3. **ID match** → missing or different ID → invalid, `"Response ID mismatch: expected '…', received '…'."`
4. **Placeholder validation** → `TextRuleEngine.ValidatePlaceholders`; any `Error` or `Fatal` → invalid, `"Placeholder validation failed: …"`.
5. **Success** → `IsValid = true`, text and rationale populated.

Warnings (for example an unexpected placeholder) do **not** invalidate the proposal.

### DraftProposal

```csharp
public class DraftProposal
{
    public StableEntryId EntryId;
    public LocaleId TargetLocale;
    public string SuggestedText;
    public string Rationale;
    public List<string> TerminologyUsed;
    public bool IsValid;
    public string ValidationError;
}
```

`TerminologyUsed` is declared but not populated by `ValidateSuggestion`. If your integration wants to record which glossary terms were used, fill it yourself.

---

## Why the ID check matters

A suggestion is only meaningful for one entry and one source revision. Without an ID check, a retry, a shuffled batch response or a prompt injection could place German text on a French line.

The validator checks the ID. It does **not** check:

- that the `SourceRevision` passed in is the current one;
- that the target locale matches the entry;
- that the suggestion respects glossary terms;
- that the tone, length or register is appropriate;
- that the response is free of prompt-injection content.

Those are your responsibilities.

---

## Reference integration shape

```csharp
public DraftProposal RequestDraft(SourceRevision rev, LocaleId locale)
{
    var config = new AdvisoryConfig
    {
        SessionApiKey = Environment.GetEnvironmentVariable("LOCALIZESHIP_ADVISORY_KEY")
    };

    string prompt =
        $"Translate entry '{rev.EntryId}' into {locale}.\n" +
        $"Source: {rev.SourceText}\n" +
        $"Return JSON: {{\"id\":\"{rev.EntryId}\",\"suggestion\":\"...\",\"rationale\":\"...\"}}";

    string raw = MyHttpClient.PostJson(config, prompt); // your code
    return new AdvisoryProvider(config).ValidateSuggestion(rev, raw, locale);
}
```

Rules for the client you write:

- read the key from the environment or a secret store, never from the asset;
- send only the source text and context you are allowed to share;
- set a timeout and a retry cap;
- log the request ID, not the key or the full prompt;
- treat the response as untrusted text;
- never auto-approve.

---

## Human review is the point

A `DraftProposal` with `IsValid = true` is mechanically sound. It is not reviewed. It has not been checked for meaning, register, cultural fit or legal constraints.

Recommended flow:

1. Request a draft.
2. Validate it.
3. Show it beside the source and the current translation.
4. Let a human edit and approve.
5. Record the approval event; `ApprovalEvent` does not currently record “source = advisory”, so note it in your process or in the rejection reason if discarded.

---

## Failure messages

| Message | Cause |
|---|---|
| `Empty or null response payload received.` | Whitespace or null input |
| `Malformed suggestion payload; missing 'suggestion' field.` | Not JSON, or missing/empty `suggestion` |
| `Response ID mismatch: expected '…', received '…'.` | Wrong ID or missing ID |
| `Placeholder validation failed: …` | Missing placeholder, unexpected brace, tag issue |
| `Failed to parse advisory suggestion: …` | Exception during parsing |

The validator never throws for bad input; it returns an invalid proposal.

---

## What to tell a reviewer

- “Advisory” means optional assistance.
- Nothing is sent anywhere by the package.
- Nothing is approved automatically.
- The only automatic check is ID binding plus placeholder validity.
- A human remains the release gate for language quality.

---

## Related pages

- [Text Rules]({{ site.baseurl }}/docs/features/text-rules/)
- [Review States]({{ site.baseurl }}/docs/concepts/review-states/)
- [Data & Privacy]({{ site.baseurl }}/docs/data-and-privacy/)
- [Limitations]({{ site.baseurl }}/docs/limitations/)
