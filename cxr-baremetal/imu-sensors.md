# IMU and Sensors (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=28007d95eb7a46828f9ac7f19421867a> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

## Overview

Read on-device IMU data with Android **`SensorManager`** and draw wireframe indicators inside the safe area described in [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md).

## 6-axis and axes

The sample uses **gyroscope + accelerometer** (6-axis). The **head ball** uses gyro-only quaternion integration (no magnetometer, no accel correction); the accelerometer feeds the level bubble and raw meter pages.

The device frame matches a **portrait** Android phone with the screen facing the wearer:

- **+X** right
- **+Y** up
- **+Z** out of the screen toward the wearer

The UI shows live `acc` / `gyro` via **sliding meters, a level bubble, and numeric readouts** for debugging.

## Axis verification and keys

Before the demo, complete three steps (turn head / nod / tilt); the gyroscope infers axes and writes `GlassesAxisRemapper`. While verifying, **three gyro sliding bars** are shown: the expected axis is outlined, and the **dominant axis** highlights during motion.

| Phase | Click/Enter | Double-tap | Long press |
| --- | --- | --- | --- |
| Verification intro | Auto-start | Back to Hub | — |
| Verifying | — (head motion) | Cancel | Retry step |
| Verified | Enter demo | Back to Hub | — |
| Demo | Next page | Back to Hub | Calibrate; re-verify on last page |

## Demo pages (5)

| Page | Content |
| --- | --- |
| 1 | **Head pose ball** + mini gyro strip; yaw/pitch dot |
| 2 | **Six-axis meters**: 3 acc + 3 gyro bipolar bars with values |
| 3 | **Accel level**: circle + tilt dot; `acc Z` at bottom |
| 4 | Axis mapping (verification result) |
| 5 | Standard coordinate reference |

Click cycles pages; long press on the last page re-runs verification.

## Calibration and wireframe UI

- On demo entry, the current pose becomes "forward" (touch-panel **long press** recalibrates). Ball pose follows gyro integration only — motion stops when the head stops, without gravity correction pulling back to center.
- Yaw/pitch relative to the reference pose map to a dot in the safe area: **head turn → horizontal**, **nod → vertical**.
- Black background, green strokes and crosshair; meters use stroked tracks and dot markers; **no** on-screen tap targets.

## Sample

`GlassesBareDevSample` → **IMU**: `ImuBallScreen`, `ImuAxisMeterScreen`, `ImuAccelLevelScreen` (sensor logic in the sample `sensor` module).

See also [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md) and [GlassesBareDevSample Project and Pages](./sample-project.md).
