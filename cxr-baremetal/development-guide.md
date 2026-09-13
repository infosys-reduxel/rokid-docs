# Rokid Glasses Bare-Metal Development Guide

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/cn/a38e99d9ab364a13abd506f6cfed2856.html> (Chinese, "Rokid Glasses 裸机开发简介", fetched 2026-08-03)
>
> **Doc version: v1.0.0 (2026-06-05)**

<!-- REFRESH-PENDING (2026-08-03): This file's "Scope"/"Runtime Environment"/"System and Constraints"/"Device Capability Overview"/"Suggested Reading Order"/"Doc Index"/"Sample Project" sections are translated from the 简介 (intro) sub-page of the upstream workspace ff28c865a9634876be98cbc293588460, confirmed retrievable this cycle via `firecrawl scrape --format rawHtml --wait-for 12000`. The remaining 7 sub-pages of that same workspace (快速开始, Sample 工程与页面说明, 眼镜 UI 设计规范, 按键与佩戴和折叠, 原始音频, 拍照, 录像, IMU与传感器) are reachable only through a client-side antd Tree-nav click-through that has not rendered real content in any Firecrawl attempt across many prior cycles (dispatchEvent/`.click()`/executeJavascript techniques all returned the identical 简介 body). The pre-existing "Development Environment" / "Device Connection" sections below (sourced 2026-05-29 from the older 57e35cd3ae294d16b1b8fc8dcbb1b7c7 workspace, which this doc superseded) are retained as-is since they still track upstream's ADB/developer-cable guidance and were not contradicted by anything in the new 简介 content; they should be folded into a dedicated 快速开始 (Quick Start) translation once that sub-page becomes retrievable. 按键与佩戴和折叠 (→ key-broadcasts.md) and 原始音频 (→ audio-recording.md) have their own separate REFRESH-PENDING notes. Sample 工程与页面说明, 眼镜 UI 设计规范, 拍照, 录像, and IMU与传感器 have no local doc yet — standing P0 candidates, not translated here since their content has never been retrieved. -->

This guide describes how to build plain Android apps that run directly on Rokid Glasses (YodaOS-Sprite) **without using any CXR SDK**. These are side-loaded standalone apps that handle their own input, audio, and camera. For apps that integrate with the on-device Rokid AI app or the mobile companion, see the [CXR-L](../cxr-l/), [CXR-S](../cxr-s/), and [CXR-M](../cxr-m/) SDK docs instead.

## Scope

This guide explains how to write and debug standard Android apps directly on Rokid Glasses (system **YodaOS-Sprite**, based on **Android 12, API 31** / Android Go), using standard Android APIs, **without depending on** a mobile-side companion SDK.

Typical use cases: launchers, system utilities, on-device audio recording / photo capture / video recording, button and wear-state monitoring, IMU demos, and similar.

## Runtime Environment

| Item | Description |
|---|---|
| Glasses system | YodaOS-Sprite, **Android 12 (API 31)** / Android Go |
| App minSdk | **31** (matches the `GlassesBareDevSample` sample) |
| Sample targetSdk | **36** (matches the sample project) |
| Recommended IDE | Android Studio |
| Debugging | Dedicated developer cable + enable glasses ADB via the Rokid AI mobile app (see Quick Start) |

## System and Constraints

- Must follow the [Android Go development constraints](https://developer.android.com/guide/topics/androidgo?hl=zh-cn).
- Screen: **480 × 640 px**; see the Glasses UI Design Spec chapter for UI guidelines (not yet translated locally — see REFRESH-PENDING note above).
- Product visual design reference: Rokid Glasses Design Spec (upstream short link `t.rokid.com/0w0opp8x`).

## Device Capability Overview

| Capability | Integration | Chapter |
|---|---|---|
| 480×640 single-color display | Compose / standard `View` | Glasses UI Design Spec |
| Temple function button + right touchpad | System broadcasts + `KeyEvent` | [Button Broadcasts](./key-broadcasts.md) |
| 8-channel raw audio | `AudioRecord` | [Audio Recording](./audio-recording.md) |
| Photo / video capture | CameraX | Photo / Video (not yet translated locally) |
| 6-axis IMU | `SensorManager` | IMU and Sensors (not yet translated locally) |

## Suggested Reading Order

1. Bare-Metal Development Guide (this doc)
2. Quick Start
3. Glasses UI Design Spec
4. Sample Project and Screen Reference
5. [Button Broadcasts](./key-broadcasts.md)
6. [Audio Recording](./audio-recording.md) → Photo Capture → Video Recording → IMU and Sensors

## Sample Project

The upstream doc ships with a sample project, **`GlassesBareDevSample`** (package `com.rokid.glassesbaredevsample`). The archive and build steps live in the Quick Start chapter (not yet translated locally).

## Overview

Bare-metal development on Rokid Glasses is essentially the same as standard Android app development.

A few things to keep in mind:

- The YodaOS-Sprite system on Rokid Glasses is built on Android Go, so you must follow Android Go's development constraints.
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

## Related docs

- [Button Broadcasts](./key-broadcasts.md) — the `KeyType` enum and ordered-broadcast pattern for intercepting button / touchpad events.
- [Audio Recording](./audio-recording.md) — 8-channel microphone capture (`ChannelMask = 0x6000FC`, 16 kHz, 16-bit PCM).

<!-- TODO: Source mentions "specific functions [for wake words] will be documented in future revisions" but does not list any. Refresh when upstream publishes the list. -->
