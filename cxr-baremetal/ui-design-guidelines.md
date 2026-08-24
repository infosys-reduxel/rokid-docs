# Glasses UI Design Guidelines (Bare Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=161dddee340444108682adb8693fdb20> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

## Overview

Rokid Glasses use a **monochrome green** display. In Compose/Android, **black is treated as transparent (non-emissive)**. Use **stroke outlines** for UI elements and avoid large solid fills.

## Resolution and safe area

| Item | Value |
| --- | --- |
| Physical resolution | 480 × 640 px |
| Top margin (avoid drawing) | 80 px |
| Bottom margin (avoid drawing) | 80 px |
| **Recommended content area** | 480 × **480** (y: 80–560) |

```
┌──────────────── 480 ────────────────┐  y=0
│░░░░░░░░░░ top band 80px ░░░░░░░░░░░░│
├─────────────────────────────────────┤  y=80
│                                     │
│      recommended UI area 480×480    │
│                                     │
├─────────────────────────────────────┤  y=560
│░░░░░░░░░░ bottom band 80px ░░░░░░░░│
└─────────────────────────────────────┘  y=640
```

## Compose conventions

- Background: `Color(0xFF000000)` (non-lit pixels).
- Foreground: single green, e.g. `Color(0xFF00FF00)`.
- Shapes: prefer `Canvas` + `drawRect` / `drawCircle` / `drawLine` with `style = Stroke(...)`.
- Text: small size, high contrast; avoid solid Material `Card` blocks (`GlassesBareDevSample` uses wireframe `BareDevScaffold`).
- Full screen: you may hide system bars and keep the screen on (demo scenarios).

## On-glasses interaction

The glasses display has **no touch focus**. The sample does not use list scrolling, swipe gestures, or tappable buttons as the only entry path.

| Input | Semantics |
| --- | --- |
| **Click / Enter** | Touch panel single tap · `KEYCODE_ENTER` or temple `ACTION_SPRITE_BUTTON_CLICK` → primary action |
| **Double-tap (single-finger)** | Touch panel single-finger double-tap · `KEYCODE_BACK` → enter Hub / back from sub-page |
| **Long press** | Touch panel single-finger long press · `ACTION_AI_START` → secondary action (e.g. IMU calibrate) |
| **Swipe forward / back (two-finger)** | Two-finger swipe broadcasts → next / previous screen (optional for product apps) |
| Physical **Back** | `KEYCODE_BACK`; same as single-finger double-tap |

Conventions:

- One **screen** of content per view (inside the 480×480 safe area). Use **pagination** (click to advance), not vertical scrolling.
- Fixed **key hint bar** at the bottom (`BareKeyHintBar`) aligned with the current page.
- `BareGlassesInputDispatcher` dispatches input (see the [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) chapter). After consuming a broadcast, call `abortBroadcast()`.

## Sample components

`GlassesBareDevSample` uses **single-finger tap, double-tap, and long press on the touch panel** for Hub and sub-page navigation; temple long press and two-finger gestures are intercept-only in the sample. See [GlassesBareDevSample Project and Pages](./sample-project.md).

| Component | Role |
| --- | --- |
| `BareScreenLayout` | Title + wireframe content panel + key hints inside safe area |
| `BareKeyLegendBar` | Click / double-tap / long-press labels for the current action |
| `BareContentPanel` | Main wireframe panel (stroke only, no solid Card) |
| `BareSavedPathBlock` | Full save path after photo / audio / video success |
| `BarePageDots` | Page indicator (Hub and sub-pages) |
| `SafeAreaFrame` | Safe-area reference stroke (y=80–560) |
| `ImuBallScreen` | Head pose ball + crosshair + mini gyro strip; calibrate with touch-panel long press |
| `ImuAxisMeterScreen` | Six-axis bipolar sliding meters (acc / gyro) |
| `ImuAccelLevelScreen` | Accelerometer level bubble (circle + tilt dot) |

See also [Introduction to Bare-Metal Development on Rokid Glasses](./development-guide.md) and [GlassesBareDevSample Project and Pages](./sample-project.md).
