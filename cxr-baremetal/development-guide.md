# Rokid Glasses Bare-Metal Development Guide

> Source: <https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/13083daf77dd40bf84cf5c59711e987a.html> (Chinese, fetched 2026-05-29)
>
> **Doc version: v0.0.1 (2026-03-01)**
>
> **Known stale (unresolved as of 2026-08-09).** `developerdoc.rokid.com/sdk` now lists "眼镜端裸机开发" (bare-metal development) as doc version **1.0.0**, "更新于 2026.06.05" (updated 2026-06-05) — newer than the v0.0.1 this page was translated from. The detail page itself is unreachable: its `custom.rokid.com` CMS host (workspace `57e35cd3ae294d16b1b8fc8dcbb1b7c7`) has returned OSS `NoSuchKey` since at least 2026-06-28, and no replacement URL for the bare-metal detail content has been found on `developerdoc.rokid.com` or `open.rokid.com` (their maps return only the SDK-picker and Sprite landing pages, not per-SDK detail routes). Content could not be verified or updated beyond noting this version drift — do not assume this page still matches upstream 1.0.0.

This guide describes how to build plain Android apps that run directly on Rokid Glasses (YodaOS-Sprite) **without using any CXR SDK**. These are side-loaded standalone apps that handle their own input, audio, and camera. For apps that integrate with the on-device Rokid AI app or the mobile companion, see the [CXR-L](../cxr-l/), [CXR-S](../cxr-s/), and [CXR-M](../cxr-m/) SDK docs instead.

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
