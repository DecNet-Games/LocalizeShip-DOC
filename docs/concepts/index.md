---
layout: default
title: Concepts
nav_order: 5
has_children: true
description: "The four ideas behind LocalizeShip: stable identities, source revisions, explicit review states and coverage honesty."
permalink: /docs/concepts/
---

# Core Concepts
{: .fs-9 }

### The rules the rest of the product is built on
{: .fs-6 .text-grey-dk-000 }

LocalizeShip is small on purpose. It does not invent a new localization format, a new runtime, or a new translation memory. It adds four pieces of discipline around the data you already have.

![Revision identity]({{ site.baseurl }}/assets/images/08-revision-identity.png)

---

## 1. Stable line identity

A line is addressed by a `StableEntryId` — a string key such as `dialogue_npc_01`. Reordering entries, moving them between tables, or sorting a collection must not change which line is which.

[Read Stable Identities]({{ site.baseurl }}/docs/concepts/stable-identities/)

## 2. Source revision

A line's source is not just text. It is normalized text plus declared speaker and context, hashed together. The hash is the handle used by approvals, voice takes and merge base records.

[Read the Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/)

## 3. Explicit review state

Text present in a table is not a review. LocalizeShip keeps six explicit states per line × locale and binds approval to the exact source hash that was reviewed. A source change invalidates approval instead of silently inheriting it.

[Read Review States]({{ site.baseurl }}/docs/concepts/review-states/)

## 4. Coverage honesty

“Not checked” is a real answer. A fixture that was never visited is `Unknown`, not `Passed`. A screenshot produced in the Editor is labelled an Editor approximation, not player capture. A gate that has not run is not a green gate.

[Read Coverage Honesty]({{ site.baseurl }}/docs/concepts/coverage-honesty/)

---

## Supporting mechanics

### Normalization
Whitespace and line-ending policy is versioned, so trivial formatting churn does not create false invalidations — and a policy change is visible in the hash input.

[Read Normalization]({{ site.baseurl }}/docs/concepts/normalization/)

### Audit trail
Every state change is an `ApprovalEvent` with reviewer, role, rationale, timestamps and the old/new hashes. The history is stored on the approval record.

[Read the Audit Trail]({{ site.baseurl }}/docs/concepts/audit-trail/)

---

## How the concepts fit together

```
StableEntryId  ──────────────┐
                             ▼
Raw source text ──► NormalizationPolicy ──► normalized text + speaker + context
                                                   │
                                                   ▼
                                            SourceRevisionHash
                                                   │
              ┌────────────────────────────────────┼──────────────────────────────┐
              ▼                                    ▼                              ▼
     ApprovalRecord                      VoiceEntry binding                Merge base hash
     (state + history)                   (source + translation)            (stale conflicts)
              │
              ▼
     ReleaseGateEvaluator
```

{: .tip }
> If you are integrating LocalizeShip into an existing pipeline, read [Revision Graph]({{ site.baseurl }}/docs/concepts/revision-graph/) and [Review States]({{ site.baseurl }}/docs/concepts/review-states/) before writing any glue code. Most integration mistakes come from treating a review state as a property of a string rather than a property of a revision.
