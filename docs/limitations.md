---
layout: default
title: Limitations & Honesty Fence
nav_order: 11
description: "Explicit technical limitations, supported scopes, and honesty boundaries for LocalizeShip."
permalink: /docs/limitations/
---

# Limitations & Honesty Fence
{: .fs-9 }

### Transparent Boundaries of What LocalizeShip Does and Does Not Do
{: .fs-6 .text-grey-dk-000 }

---

To maintain professional engineering standards, LocalizeShip explicitly documents its technical boundaries and non-goals. We believe in complete transparency rather than marketing exaggeration.

---

## What LocalizeShip Does NOT Do

### 1. No Automated Linguistic RTL Correctness Certification
While LocalizeShip provides bidirectional (Bidi) text expansion and stress fixtures to test layout clipping for right-to-left languages (such as Arabic and Hebrew), it **does not claim to certify linguistic or grammatical correctness**. 
* The engine verifies that font glyphs exist and text fits within bounding boxes.
* Cultural appropriateness, correct word shaping, and narrative nuance require human native-speaker review.

### 2. No Player-Capture Guarantees Without Registered Fixtures
LocalizeShip does not possess magic insight into runtime gameplay. It cannot automatically know how a dynamic inventory menu will look during deep gameplay unless you attach a `CaptureFixture` component to that UI prefab or scene.
* Editor previews are explicitly labelled as **`Editor Approximation`**.
* True player layout verification requires executing registered fixtures via the development player capture bridge.

### 3. No Bundling of Third-Party Commercial Code
LocalizeShip includes compile-isolated adapters for **I2 Localization** and **Pixel Crushers Dialogue System**. 
* We do **not** bundle or redistribute proprietary third-party code.
* The adapters are verified in our acceptance tests for **clean compilation and safe provider-absent fallback**. We do not claim CI verification against licensed proprietary installations.

### 4. No Unsupervised "One-Click AI Translation"
LocalizeShip explicitly rejects fully automated, unsupervised machine translation into retail games. The optional Advisory Drafts engine strictly outputs strings as `Draft`. It can never transition a line directly to `Approved`. Human review is mandatory.

---

## Technical Performance Limits

* **String Table Size**: Tested up to 25,000 active string entries per workspace without UI degradation. For games exceeding 50,000 lines, we recommend partitioning your project into multiple workspaces (e.g., `Core_UI.asset`, `Dialogue_Act1.asset`).
* **Supported Render Pipelines**: Universal Render Pipeline (URP), High Definition Render Pipeline (HDRP), and Built-in Render Pipeline.
* **Supported UI Systems**: uGUI (Unity UI), TextMeshPro, and runtime UI Toolkit.
