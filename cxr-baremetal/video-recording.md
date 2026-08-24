# Video Recording (CameraX) (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=d43738accccd44a1866a59b3495ad39f> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

## Overview

On YodaOS-Sprite, use CameraX **`VideoCapture`** (`Recorder` + `VideoRecordEvent`) to record **MP4** — the standard Android camera stack.

**No** live preview. `GlassesBareDevSample` records **video-only** (no `withAudioEnabled()` on `VideoCapture`; use a separate `AudioRecord` for an audio track — see [Raw Audio on Glasses](./audio-recording.md)). Touch-panel **single tap** starts/stops recording. **Show the save path only after a successful `VideoRecordEvent.Finalize`** — the file may still be flushing when `stop()` returns.

## Prerequisites

- `CAMERA` permission (the sample video page requests camera only; use `RECORD_AUDIO` if you enable an in-track audio source).
- Manifest: `RECORD_VIDEO_OUTPUT` and storage permissions (same as the sample).
- Asynchronous `ProcessCameraProvider` binding (`rememberCameraBound`).

## Flow

1. Obtain `ProcessCameraProvider` asynchronously; bind only `VideoCapture` (no `Preview`).
2. Configure `Recorder` + `QualitySelector` (sample: `HD` / `SD` with `FallbackStrategy`).
3. `VideoCapture.Builder(recorder).build()`.
4. `prepareRecording(context, FileOutputOptions)` → `start`; **tap again** to call `Recording.stop()`.
5. In `VideoRecordEvent.Finalize`, check `hasError()`; on success, publish the full `.mp4` path; on failure, show `error` / `cause`.
6. On page destroy, `stop()` and `unbindAll`.

## Save location

| Priority | Path |
| --- | --- |
| 1 | `/sdcard/Video/bare_video/yyyyMMdd_HHmmss.mp4` |
| 2 | `…/Android/data/com.rokid.glassesbaredevsample/files/Movies/bare_video/` |

## UI feedback (sample)

| Phase | Hero / status |
| --- | --- |
| Binding | `Preparing…` |
| Ready | `Standby` · tap to start |
| Recording | `Recording` |
| After stop | `Stopping…` → `Saved` + path, or `Save failed(…)` |

## System video key

Temple **long press** may trigger system recording. In-app recording is independent CameraX control (touch-panel tap to start/stop).

## Sample

`GlassesBareDevSample` → double-tap to enter **Video** from the Hub → tap to start/stop; double-tap to return.

Related code: `camera/CameraBind.kt`, `activities/video/`, `utils/BareMediaStorage.kt`.

See also [Photo Capture (CameraX)](./photo-capture.md) and [GlassesBareDevSample Project and Pages](./sample-project.md).
