# GlassesBareDevSample Project and Pages

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=629cc9baf4124789b868a4f349817117> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

## Project info

| Item | Value |
| --- | --- |
| Project name | `GlassesBareDevSample` |
| Package | `com.rokid.glassesbaredevsample` |
| minSdk | 31 (Android 12) |
| targetSdk | 36 |
| UI | Jetpack Compose, black background + green wireframe |
| Architecture | Single Activity + `NavHost` + per-capability ViewModels |
| Download | `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR_Bare/GlassesBareDevSample.zip` |

## Module layout

```text
app/src/main/java/com/rokid/glassesbaredevsample/
  app/CONSTANT.kt
  camera/CameraBind.kt                 # async CameraX bind (rememberCameraBound)
  input/BareGlassesInputDispatcher.kt  # KeyEvent + broadcasts (two-finger abort-only)
  navigation/BareSceneRoutes.kt
  ui/design/BareDesign.kt
  ui/design/BareSavedPathBlock.kt      # save-path display
  ui/theme/
  ui/imu/ImuBallScreen.kt
  ui/imu/ImuAxisMeterScreen.kt
  ui/imu/ImuAccelLevelScreen.kt
  utils/BareMediaStorage.kt            # photo/video output directories
  activities/main/MainActivity.kt      # NavHost + Hub
  activities/keys/                     # keys / wear / fold
  activities/audio/
  activities/photo/
  activities/video/
  activities/imu/
  sensor/                              # IMU demo and axis mapping
```

## Sample input model (simplified)

For clarity, the sample uses **touch-panel single-finger tap, double-tap, and long press for navigation**. Temple long press and two-finger system broadcasts are registered in `BareGlassesInputDispatcher` and **`abortBroadcast()` is called**; they do not drive UI. Product apps may use the full two-finger actions described in [Keys, Wear Detection, and Fold Events](./key-broadcasts.md).

| Semantics | Source |
| --- | --- |
| Click / Enter | Touch panel `KEYCODE_ENTER`, temple `ACTION_SPRITE_BUTTON_CLICK` |
| Double-tap / back | Touch panel `KEYCODE_BACK` |
| Long press | Touch panel `ACTION_AI_START` |

## Hub and routes

| Route | Screen | Click/Enter | Double-tap | Long press |
| --- | --- | --- | --- | --- |
| `hub` | Capability 1/5 | Next item (wrap) | Enter | — |
| `keys_wear` | Wear / key log | Next page (wrap) | Back | — |
| `audio` | Recording | Start/stop | Back | — |
| `photo` | Photo | Tap to capture | Back | — |
| `video` | Video | Tap start/stop | Back | — |
| `imu` | Verify (gyro bars) / demo 5 pages | See [IMU and Sensors](./imu-sensors.md) | Back | See [IMU and Sensors](./imu-sensors.md) |

### Photo / Video pages

- **No live preview**; when ready, use touch-panel **single tap** (not a wireframe button).
- Subtitle when ready: `Camera ready · tap to capture` or `Standby · tap to start`.
- On success, **Save path** shows the full `.jpg` / `.mp4` path; errors appear in the status area.
- Default output: `/sdcard/Pictures/bare_photo/`, `/sdcard/Video/bare_video/` (falls back to app storage — see [Photo Capture](./photo-capture.md) / [Video Recording](./video-recording.md) chapters).
- Ignores duplicate double-tap for ~400 ms after entering a sub-page (`rememberSubPageEnterDebounce`) so Hub double-tap does not immediately pop back.

## Build and install

See [Bare-Metal Development Quick Start](./quick-start.md).

See also [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md).
