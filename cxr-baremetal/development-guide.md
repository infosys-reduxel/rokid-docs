# Rokid Glasses Bare-Metal Development Guide

> Source: <https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/13083daf77dd40bf84cf5c59711e987a.html> (Chinese, fetched 2026-05-29); positioning/environment details refreshed against <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html> (English, fetched 2026-09-21)
>
> **Doc version: v0.0.1 (2026-03-01), partially refreshed against upstream v1.0.0 (2026-06-05)**
>
> **Upstream restructure note (2026-09-21):** Rokid republished bare-metal development docs at a new workspace as v1.0.0, split across several chapters: *Introduction*, *Quick Start*, *Glasses UI Design Guidelines (Bare Metal)*, *GlassesBareDevSample Project and Pages*, *Keys, Wear Detection, and Fold Events*, *Raw Audio on Glasses*, *Photo Capture / Video Recording*, and *IMU and Sensors*. Only the *Introduction* chapter's content (positioning, runtime environment, system constraints) could be retrieved this cycle — the site is a single-page app and Firecrawl's site map surfaces only the index URL, not the individual chapter routes for Quick Start / UI Guidelines / Photo Capture / Video Recording / IMU. This section has been refreshed with what was retrieved; the sibling docs ([Audio Recording](./audio-recording.md), [Button Broadcasts](./key-broadcasts.md)) have **not** been refreshed against v1.0.0 and may be missing new detail — a follow-up crawl targeting the individual chapter routes is needed. TODO: crawl the remaining v1.0.0 chapters listed above.

This guide describes how to build plain Android apps that run directly on Rokid Glasses (YodaOS-Sprite) **without using any CXR SDK**. These are side-loaded standalone apps that handle their own input, audio, and camera. For apps that integrate with the on-device Rokid AI app or the mobile companion, see the [CXR-L](../cxr-l/), [CXR-S](../cxr-s/), and [CXR-M](../cxr-m/) SDK docs instead.

## Overview

Bare-metal development on Rokid Glasses is essentially the same as standard Android app development, using standard Android APIs directly on-device — no phone-side co-SDK required.

Typical use cases: launchers, on-device utilities, local audio capture / photo / video, key and wear-state listeners, IMU demos, and similar glasses-only apps.

A few things to keep in mind:

- The YodaOS-Sprite system on Rokid Glasses is built on Android Go, so you must follow Android Go's development constraints. Per the v1.0.0 docs, app `minSdk` is **31** and the reference sample's `targetSdk` is **36**.
- The Rokid Glasses display is designed for a **480 × 640 pixel** viewport. Follow the design spec when laying out your UI.
- YodaOS-Sprite defines several interactions with the Rokid Glasses mobile companion app that bare-metal apps **cannot** override:
  - Long-press the touchpad on the side of the right temple to enter the on-device Rokid AI module.
  - Double-click the button on the right temple to trigger the back action.
  - Click the top button on the right temple to take a photo.
  - Long-press the top button on the right temple to record video.
  - Certain wake words activate dedicated functions (specific functions will be documented in future revisions).

For everything else — clicks, long presses, two-finger gestures on the touchpad, and the Settings key — see [Button Broadcasts](./key-broadcasts.md).

## Development Environment

Essentially the same as standard Android app development:

- A computer capable of Android development, with a standard USB port.
- An Android IDE such as Android Studio.
- A Rokid Glasses device.

## Device Connection

The charging contacts on the side of the left temple of Rokid Glasses are dual-purpose: they carry both power and data. To use them for data, however, you need the dedicated developer cable.

### Use the dedicated developer cable

ADB and other data flows over USB require the dedicated **developer cable** — the standard charging cable that ships in-box will not work.

> **Tip:** The cable that ships in-box is charge-only. To request a developer cable, contact the Rokid **Developer Assistant**.

### Enable ADB

ADB on Rokid Glasses must be enabled through the **Rokid AI mobile app**, not via the on-device Settings UI.

Once ADB is on, you can mirror the glasses display during development using a screen-mirroring tool such as **SCRCPY**.

## Sample project

Rokid publishes a reference sample, **GlassesBareDevSample** (package `com.rokid.glassesbaredevsample`), demonstrating the device capabilities below. Download/build steps live in the upstream *Quick Start* chapter (not yet mirrored here — see the restructure note above).

| Capability | Integration | Upstream chapter |
| --- | --- | --- |
| 480×640 single-green display | Compose / standard Views | Glasses UI Design Guidelines (Bare Metal) — not yet mirrored |
| Temple function key + right touch panel | System broadcasts + `KeyEvent` | [Button Broadcasts](./key-broadcasts.md) |
| 8-channel raw audio | `AudioRecord` | [Audio Recording](./audio-recording.md) |
| Photo / video | CameraX | Photo Capture / Video Recording — not yet mirrored |
| 6-axis IMU | `SensorManager` | IMU and Sensors — not yet mirrored |

## Related docs

- [Button Broadcasts](./key-broadcasts.md) — the `KeyType` enum and ordered-broadcast pattern for intercepting button / touchpad events.
- [Audio Recording](./audio-recording.md) — 8-channel microphone capture (`ChannelMask = 0x6000FC`, 16 kHz, 16-bit PCM).

<!-- TODO: Source mentions "specific functions [for wake words] will be documented in future revisions" but does not list any. Refresh when upstream publishes the list. -->
