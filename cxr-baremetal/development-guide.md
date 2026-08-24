# Introduction to Bare-Metal Development on Rokid Glasses

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

This guide describes how to build and debug standard Android applications **on Rokid Glasses** (system **YodaOS-Sprite**, based on **Android 12 (API 31)** / Android Go) using standard Android APIs, **without** a phone-side co-SDK.

Typical use cases: launchers, on-device utilities, local audio capture / photo / video, key and wear-state listeners, IMU demos, and similar glasses-only apps.

## Runtime environment

| Item | Details |
| --- | --- |
| Glasses OS | YodaOS-Sprite, **Android 12 (API 31)** / Android Go |
| App minSdk | **31** (same as `GlassesBareDevSample`) |
| Sample targetSdk | **36** (same as the sample project) |
| Recommended IDE | Android Studio |
| Debug setup | Dedicated dev cable + Rokid AI App on phone to enable glasses ADB (see [Bare-Metal Development Quick Start](./quick-start.md)) |

## System and constraints

- Follow the [Android Go developer guidance](https://developer.android.com/guide/topics/androidgo).
- Display: **480 × 640** px. UI rules are in [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md).
- Product visuals: Rokid glasses design guidelines (`https://t.rokid.com/0w0opp8x`).

## Device capabilities

| Capability | Integration | Chapter |
| --- | --- | --- |
| 480×640 single-green display | Compose / standard Views | [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md) |
| Temple function key + right touch panel | System broadcasts + `KeyEvent` | [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) |
| 8-channel raw audio | `AudioRecord` | [Raw Audio on Glasses](./audio-recording.md) |
| Photo / video | CameraX | [Photo Capture](./photo-capture.md) / [Video Recording](./video-recording.md) |
| 6-axis IMU | `SensorManager` | [IMU and Sensors](./imu-sensors.md) |

## Suggested reading order

1. Introduction to Bare-Metal Development on Rokid Glasses (this page)
2. [Bare-Metal Development Quick Start](./quick-start.md)
3. [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md)
4. [GlassesBareDevSample Project and Pages](./sample-project.md)
5. [Keys, Wear Detection, and Fold Events](./key-broadcasts.md)
6. [Raw Audio on Glasses](./audio-recording.md) → [Photo Capture](./photo-capture.md) → [Video Recording](./video-recording.md) → [Camera Preview Outlining](./camera-preview-outlining.md) → [IMU and Sensors](./imu-sensors.md)

## Document index

Each chapter is published as a separate page on the documentation platform:

| Chapter | Topics |
| --- | --- |
| [Bare-Metal Development Quick Start](./quick-start.md) | Environment, ADB, sample build |
| [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md) | Resolution, safe area, wireframe UI |
| [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) | Keys, touch panel, wear/fold (broadcasts + `KeyEvent`) |
| [Raw Audio on Glasses](./audio-recording.md) | 8-channel `AudioRecord` |
| [Photo Capture](./photo-capture.md) / [Video Recording](./video-recording.md) | CameraX |
| [Camera Preview Outlining](./camera-preview-outlining.md) | Real-time edge-detection preview shader pipeline |
| [IMU and Sensors](./imu-sensors.md) | `SensorManager` |
| [GlassesBareDevSample Project and Pages](./sample-project.md) | Sample app structure and capability screens |

## Sample project

**GlassesBareDevSample** (package `com.rokid.glassesbaredevsample`). Download and build steps are in [Bare-Metal Development Quick Start](./quick-start.md).

<!-- Earlier revision (v0.0.1, 2026-03-01) also documented reserved system interactions (long-press touchpad → AI module, double-click → back, top button → photo/video) and developer-cable / ADB-enablement steps inline on this page. That material now lives in the Quick Start and Keys/Wear/Fold chapters linked above, matching the upstream v1.0.0 restructuring. -->
