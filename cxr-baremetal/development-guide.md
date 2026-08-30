# Rokid Glasses Bare-Metal Development Guide

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html> — "Introduction" and "Quick Start" pages (English, fetched 2026-08-30)
>
> **Doc version: 1.0.0** — this workspace superseded the earlier Chinese workspace (`57e35cd3ae294d16b1b8fc8dcbb1b7c7`, doc v0.0.1, 2026-03-01) that this page was previously translated from.

This guide describes how to build plain Android apps that run directly on Rokid Glasses (YodaOS-Sprite) **without using any CXR SDK**. These are side-loaded standalone apps that handle their own input, audio, and camera. For apps that integrate with the on-device Rokid AI app or the mobile companion, see the [CXR-L](../cxr-l/), [CXR-S](../cxr-s/), and [CXR-M](../cxr-m/) SDK docs instead.

## Overview

Bare-metal development on Rokid Glasses is standard Android application development targeting **YodaOS-Sprite**, which runs **Android 12 (API level 31) / Android Go**, using stock Android APIs and no phone-side companion SDK.

Typical use cases: launchers, on-device utilities, local audio capture, photo/video capture, button and wear-state listeners, IMU demos, and other glasses-only apps.

### Runtime environment

| Item | Details |
|---|---|
| Glasses OS | YodaOS-Sprite, **Android 12 (API 31)** / Android Go |
| App `minSdk` | **31** (matches the `GlassesBareDevSample` reference project) |
| Sample `targetSdk` | **36** (matches the sample project) |
| Recommended IDE | Android Studio |
| Debug setup | Dedicated developer cable + Rokid AI mobile app to enable glasses ADB (see [Device Connection](#device-connection) below) |

### System and constraints

- Follow the [Android Go developer guidance](https://developer.android.com/guide/topics/androidgo) — the YodaOS-Sprite system on Rokid Glasses is built on Android Go, so you must follow Android Go's development constraints.
- The Rokid Glasses display is designed for a **480 × 640 pixel** viewport. Follow the design spec when laying out your UI.
- YodaOS-Sprite defines several interactions with the Rokid Glasses mobile companion app that bare-metal apps **cannot** override:
  - Long-press the touchpad on the side of the right temple to enter the on-device Rokid AI module.
  - Double-click the button on the right temple to trigger the back action.
  - Click the top button on the right temple to take a photo.
  - Long-press the top button on the right temple to record video.
  - Certain wake words activate dedicated functions (specific functions will be documented in future revisions).

For everything else — clicks, long presses, wear/fold detection, two-finger gestures on the touchpad, and the Settings key — see [Keys, Wear Detection, and Fold Events](./key-broadcasts.md).

### Device capabilities

| Capability | Integration | See also |
|---|---|---|
| 480×640 single-color (green) display | Compose / standard Views | Glasses UI Design Guidelines (Bare Metal) — upstream topic, not yet added to this repo |
| Temple function key + right touch panel | System broadcasts + `KeyEvent` | [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) |
| 8-channel raw audio | `AudioRecord` | [Audio Recording](./audio-recording.md) |
| Photo / video capture | CameraX | Photo Capture / Video Recording — upstream topics, not yet added to this repo |
| 6-axis IMU | `SensorManager` | IMU and Sensors — upstream topic, not yet added to this repo |

<!-- [TODO] Upstream also documents "Glasses UI Design Guidelines (Bare Metal)", "GlassesBareDevSample Project and Pages", "Photo Capture", "Video Recording", "Camera Preview Outlining", and "IMU and Sensors" as separate pages. These are new topics with no corresponding file in this repo yet; see https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html for the full nav tree. Out of scope for this pass — flagged for a follow-up translation pass. -->

## Development Environment

Prerequisites, per the upstream Quick Start guide:

- Android Studio, with the Gradle Kotlin DSL.
- Glasses OS: **Android 12 (API 31)** / YodaOS-Sprite.
- App **`minSdk` 31**, sample **`targetSdk` 36** — same as the `GlassesBareDevSample` reference project.
- A Rokid Glasses device plus a **developer cable** (retail packaging typically ships a charging-only cable; debugging requires the dedicated developer cable — request one via the Rokid Developer Forum or official developer support).
- The **Rokid AI mobile app**, to enable glasses ADB.

### Sample project

- **Project name:** `GlassesBareDevSample` (package `com.rokid.glassesbaredevsample`)
- **Download:** [`GlassesBareDevSample.zip`](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR_Bare/GlassesBareDevSample.zip)

1. Download and extract the zip.
2. In Android Studio: **Open** → select the extracted project root.
3. Sync Gradle (standard Android repositories are sufficient).
4. Build a debug APK:

   ```bash
   cd <project-root>
   ./gradlew :app:assembleDebug
   ```

   On Windows, use `gradlew.bat`.
5. Install it:

   ```bash
   adb install -r app/build/outputs/apk/debug/app-debug.apk
   ```

## Device Connection

The charging contacts on the side of the left temple of Rokid Glasses are dual-purpose: they carry both power and data. To use them for data, however, you need the dedicated developer cable.

### Use the dedicated developer cable

ADB and other data flows over USB require the dedicated **developer cable** — the standard charging cable that ships in-box will not work.

> **Tip:** The cable that ships in-box is charge-only. To request a developer cable, contact the Rokid **Developer Assistant** or use the [Rokid Developer Forum](https://forum.rokid.com/index).

### Enable ADB

ADB on Rokid Glasses must be enabled through the **Rokid AI mobile app**, not via the on-device Settings UI:

1. Connect the glasses to the PC's USB port with the developer cable.
2. On the phone, open glasses **ADB** in the Rokid AI app.
3. `adb devices` should list the glasses.

Once ADB is on, you can mirror the 480×640 glasses display during development using a screen-mirroring tool such as **scrcpy**.

### Minimal verification path

A quick smoke test using the `GlassesBareDevSample` reference project:

1. Install `GlassesBareDevSample`.
2. Launch the app — the Hub screen should list: Keys / wear & fold, Raw audio, Photo, Video, IMU.
3. Open **Keys / wear & fold** — on the second page, check the key/broadcast logs (including "aborted" hints) while exercising the touchpad and temple key. See [Keys, Wear Detection, and Fold Events](./key-broadcasts.md).
4. Open **Raw audio** — click the temple button to toggle recording; the status area shows the saved PCM file path. See [Audio Recording](./audio-recording.md).

<!-- [TODO] The sample's Photo/Video/IMU screens are not documented in this repo yet — see the "new upstream topics" note above. -->

## Related docs

- [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) — the `KeyType` action strings, wear/fold broadcasts, and ordered-broadcast + `KeyEvent` patterns for intercepting button / touchpad events.
- [Audio Recording](./audio-recording.md) — 8-channel microphone capture (`ChannelMask = 0x6000FC`, 16 kHz, 16-bit PCM).

<!-- TODO: Source mentions "specific functions [for wake words] will be documented in future revisions" but does not list any. Refresh when upstream publishes the list. -->
