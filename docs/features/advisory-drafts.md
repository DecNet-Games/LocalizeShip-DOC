---
layout: default
title: Advisory Drafts
parent: Features
nav_order: 13
description: "Optional Bring-Your-Own-Endpoint model assistance with strict schema validation that never auto-approves."
permalink: /docs/features/advisory-drafts/
---

# Advisory Drafts
{: .fs-9 }

### Optional Machine Assistance with Human-in-the-Loop Safeguards
{: .fs-6 .text-grey-dk-000 }

---

Many localization tools tout automated "One-Click AI Translation" buttons. In actual game development, blind machine translation without review is disastrous: it introduces hallucinated placeholders, breaks lore terminology, and leads to player ridicule.

LocalizeShip supports an optional **Advisory Drafts Engine** (`AdvisoryProvider.cs`), but enforces strict architectural safeguards.

---

## Uncompromising Human-in-the-Loop Guarantees

```
                               ┌─────────────────────────────┐
                               │ User Configured Endpoint    │
                               │ (OpenAI-Compatible BYO API) │
                               └──────────────┬──────────────┘
                                              │ Returns Payload
                                              ▼
                               ┌─────────────────────────────┐
                               │ Strict Schema Validation    │
                               │ * Check JSON structure      │
                               │ * Verify parsed.id == source│
                               │ * Verify exact placeholders │
                               └──────────────┬──────────────┘
                                              │ Payload Valid
                                              ▼
                               ┌─────────────────────────────┐
                               │ Ingested as [ DRAFT ]       │
                               │ (Never sets [Approved])     │
                               └──────────────┬──────────────┘
                                              │
                                              ▼
                               ┌─────────────────────────────┐
                               │ Mandatory Human QA Review   │
                               │ Language lead inspects &    │
                               │ explicitly signs off.       │
                               └─────────────────────────────┘
```

### 1. Never Auto-Approves
Suggestions returned by the advisory provider are ingested with the review status **`Draft`**. They can never transition directly to `Approved`. A human reviewer must inspect the line in the Release Board and explicitly sign off.

### 2. Strict Schema & ID Verification
Machine learning models frequently hallucinate or drop formatting keys. `AdvisoryProvider` enforces strict validation:
* **Entry ID Matching**: The returned JSON must contain an `id` that matches the request's `StableEntryId.Value`. If IDs mismatch or are omitted, the payload is completely discarded.
* **Placeholder Parity**: The suggestion must retain all declared placeholders (e.g., `{player_name}`). If a placeholder was translated or omitted, the proposal is rejected.

### 3. Bring-Your-Own-Endpoint (Zero Vendor Lock-In)
LocalizeShip does not run a proprietary paid cloud server or force a subscription. You supply your own OpenAI-compatible endpoint (OpenAI, Azure, local Ollama, or custom proxy):
* API keys are stored exclusively in local Editor user preferences (`EditorPrefs`).
* API keys are **never serialized** into `ScriptableObject` assets, git repositories, or exported reports.

### 4. Resilient Error Handling
If an endpoint times out, cancels, or returns an HTTP 429 (Rate Limit):
* No table mutations occur.
* The system logs a descriptive diagnostic message and cleanly resumes offline operation.

---

## Acceptance Test Verification: `LS_T10`

Security and schema verification are tested under acceptance test **`LS_T10_AdvisoryFailure_RejectsMalformedPayloads`**:
* Discarding malformed JSON payloads.
* Rejecting mismatched entry IDs.
* Preventing table mutation on network errors.
