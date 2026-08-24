# Photo Capture (CameraX) (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=8005821179264084bccc479e6a024074> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

## Overview

On YodaOS-Sprite, use AndroidX CameraX **`ImageCapture`** for still photos — the standard `android.hardware.camera2` stack.

The monochrome wireframe display is **not suited** for live preview. The sample binds **only** `ImageCapture` (no `Preview`). A touch-panel **single tap** triggers capture. After success, the status area and **Save path** block show the full `.jpg` path — verify with `adb pull` or a file manager on your PC.

## Prerequisites

- `CAMERA` permission (requested at runtime).
- Bind `ProcessCameraProvider` to the page `LifecycleOwner` **asynchronously** (the sample uses `rememberCameraBound`; do **not** call `ListenableFuture.get()` synchronously on the Compose main thread when entering a sub-page — it can ANR).

## Flow

1. Request `CAMERA` → obtain `ProcessCameraProvider` asynchronously.
2. After `unbindAll`, bind only `ImageCapture` (`CameraSelector.DEFAULT_BACK_CAMERA`).
3. Recommended settings (same as the sample): `CAPTURE_MODE_MAXIMIZE_QUALITY`, `setJpegQuality(100)`.
4. Touch-panel tap / temple click broadcast → `ImageCapture.takePicture`.
5. Update UI in `OnImageSavedCallback`; surface `ImageCaptureException` in `onError`.

## Save location

The sample uses `BareMediaStorage.photoDir`, preferring public storage for easy `adb pull`, with fallback to app-specific storage:

| Priority | Path |
| --- | --- |
| 1 | `/sdcard/Pictures/bare_photo/yyyyMMdd_HHmmss.jpg` |
| 2 | `…/Android/data/com.rokid.glassesbaredevsample/files/Pictures/bare_photo/` |

The manifest declares `READ/WRITE_EXTERNAL_STORAGE` and `MANAGE_EXTERNAL_STORAGE`. If public storage is not writable, the sample falls back automatically and still shows the path on screen.

## UI feedback (sample)

| Area | Content |
| --- | --- |
| Hero title | `Preparing…` → `Ready` → `Capturing` → `Saved` / `Capture failed` |
| Subtitle | Synced with ViewModel status (e.g. `Camera ready · tap to capture`) |
| Save path | `BareSavedPathBlock` shows the full absolute path |

## System camera key

A temple **click** may trigger the system camera. In-app capture is a separate **CameraX** flow (touch-panel tap); both can coexist.

## Sample

`GlassesBareDevSample` → double-tap to enter **Photo** from the Hub → when ready, **single tap** to capture; double-tap to return.

Related code: `camera/CameraBind.kt`, `activities/photo/`, `utils/BareMediaStorage.kt`.

See also [Video Recording (CameraX)](./video-recording.md) and [GlassesBareDevSample Project and Pages](./sample-project.md).
