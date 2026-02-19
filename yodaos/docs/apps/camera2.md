# Camera2 (com.android.camera2)

System camera application for YodaOS/Rokid AR glasses. Based on AOSP Camera2 (Google Camera) codebase, compiled against Android 12 (SDK 32) with target SDK 1000 (platform development build).

**APK**: `Camera2.apk`
**Version**: 2.0.002 (code 20002100)
**Min SDK**: 32 (Android 12)
**App label**: "Camera"
**Application class**: `com.android.camera.app.CameraApp`

---

## Permissions

| Permission | Purpose |
|---|---|
| `CAMERA` | Camera hardware access |
| `RECORD_AUDIO` | Video recording audio capture |
| `ACCESS_FINE_LOCATION` | Geotagging photos/videos |
| `ACCESS_COARSE_LOCATION` | Geotagging photos/videos |
| `INTERNET` | Network access (usage statistics) |
| `ACCESS_NETWORK_STATE` | Network connectivity checks |
| `ACCESS_WIFI_STATE` | WiFi state queries |
| `CHANGE_WIFI_STATE` | WiFi state changes |
| `NFC` | NFC beam for photo sharing |
| `WAKE_LOCK` | Keep device awake during capture |
| `VIBRATE` | Haptic feedback |
| `RECEIVE_BOOT_COMPLETED` | Enable/disable camera launcher on boot |
| `WRITE_SETTINGS` | System settings access |
| `SET_WALLPAPER` | Set wallpaper from captured photos |
| `BIND_WALLPAPER` | Wallpaper service binding |
| `USE_CREDENTIALS` | Account credentials |
| `READ_SYNC_SETTINGS` | Sync settings |
| `WRITE_SYNC_SETTINGS` | Sync settings |
| `SUBSCRIBED_FEEDS_READ` | Feed subscriptions |
| `SUBSCRIBED_FEEDS_WRITE` | Feed subscriptions |

**Required library**: `org.apache.http.legacy`

**Screen support**: Normal and large screens. Small screens explicitly excluded (`android:smallScreens="false"`).

---

## Activities and Components

### CameraActivity (`com.android.camera.CameraActivity`)

Main camera activity. Exported, single-task launch mode, clears task on launch.

**Intent filters**:
- `android.media.action.STILL_IMAGE_CAMERA` -- Opens camera in photo mode (default camera action)
- `android.intent.action.MAIN` with `DEFAULT` category -- Direct launch

**Configuration**: Handles `keyboardHidden|orientation|screenSize` changes without restart.

**Key behavior**:
- Extends `QuickActivity` and implements `AppController`
- Manages module system (Photo, Video, Panorama, PhotoSphere, Refocus, HDR+)
- Handles filmstrip (gallery review of captured media)
- Supports NFC beam for sharing photos
- Controls camera open/close lifecycle through `CameraController`
- Uses `Glide` library for image loading and caching

### CameraLauncher (`com.android.camera.CameraLauncher`)

Activity alias targeting `CameraActivity`. This is the home screen launcher entry point.

**Intent filters**:
- `android.intent.action.MAIN` with `LAUNCHER` + `DEFAULT` categories

### CaptureActivity (`com.android.camera.CaptureActivity`)

Handles third-party image capture requests. Empty subclass of `CameraActivity` (all logic inherited).

**Intent filters**:
- `android.media.action.IMAGE_CAPTURE` -- Standard Android image capture intent

### VideoCamera (`com.android.camera.VideoCamera`)

Activity alias targeting `CaptureActivity`. Handles video recording intents.

**Intent filters**:
- `android.media.action.VIDEO_CAMERA` -- Opens video camera mode
- `android.media.action.VIDEO_CAPTURE` -- Third-party video capture intent

### SecureCameraActivity (`com.android.camera.SecureCameraActivity`)

Camera accessible from lock screen. Empty subclass of `CameraActivity`. Excluded from recents, clears task on launch.

**Intent filters**:
- `android.media.action.STILL_IMAGE_CAMERA_SECURE` -- Secure photo capture from lock screen
- `android.media.action.IMAGE_CAPTURE_SECURE` -- Secure image capture intent

**Keyguard metadata**: Uses `@layout/keyguard_widget` for lock screen widget display.

### DirectCaptureActivity (`com.android.camera.DirectCaptureActivity`)

Lightweight direct capture activity using Camera2 API directly (bypasses the OneCamera abstraction). Exported, no intent filters (launched programmatically).

**Key details**:
- Uses Android `camera2` API directly (`CameraManager`, `CameraDevice`, `CameraCaptureSession`)
- Default preview size: 640x480
- Default capture size: 4032x3024
- Auto-captures after 10 preview frames
- Saves JPEG to `Environment.DIRECTORY_PICTURES` with filename format `JPEG_yyyyMMdd_HHmmss.jpg`
- Supports "zero shutter" mode (uses `TEMPLATE_ZERO_SHUTTER_LAG` when enabled)
- Calculates JPEG rotation from display rotation and sensor orientation

### PermissionsActivity (`com.android.camera.PermissionsActivity`)

Runtime permission request handler. Not exported. Requests CAMERA, RECORD_AUDIO, and optionally ACCESS_COARSE_LOCATION. Redirects to CameraActivity on success, shows error dialog on failure.

### CameraSettingsActivity (`com.android.camera.settings.CameraSettingsActivity`)

Settings screen using `PreferenceFragment`. Not exported. Handles camera-specific settings through the standard Android preference framework.

### SetActivitiesCameraReceiver (`com.android.camera.SetActivitiesCameraReceiver`)

Broadcast receiver that runs on `BOOT_COMPLETED`. Checks if camera hardware is present (`camera`, `camera.front`, or `camera.external` features) and enables/disables the `CameraLauncher` component accordingly.

---

## Camera Modes and Modules

The app uses a modular architecture where each camera mode is a separate `ModuleController` implementation, managed by `ModuleManagerImpl`.

### Photo Mode

Default module. Two implementations depending on `OneCameraFeatureConfig`:
- **CaptureModule**: Uses the Camera2 API through the `OneCamera` abstraction layer. Preferred when available.
- **PhotoModule**: Legacy fallback using `CameraAgent`/`CameraProxy` portability layer.

**CaptureModule capabilities**:
- Tap-to-focus (auto-focus and auto-exposure at touch point)
- Digital zoom (pinch or slider)
- HDR and HDR+ (Google Camera HDR pipeline, backed by `GcamHelper`)
- Burst capture (long-press shutter)
- Countdown timer (1s, 3s, 10s, 15s)
- Flash modes: auto, on, off
- Front/back camera switching
- Focus modes: continuous-picture (default), auto, macro, infinity
- Exposure compensation (manual adjustment)
- Grid lines overlay
- Volume keys mapped as shutter button (KEYCODE_VOLUME_UP/DOWN)
- DPAD center / camera button also triggers shutter

**PhotoModule capabilities**:
- Face detection
- Scene modes: auto, action, HDR, and others
- JPEG quality: normal, fine, superfine
- Picture sizes: large, medium, small (per camera)
- Remote shutter support (via `RemoteCameraModule` interface)

### Video Mode (`VideoModule`)

Video recording module using `MediaRecorder`.
- Front and back camera support
- Video quality settings (per camera, back/front)
- Video effects (backdropper, goofy face effects)
- Video flash mode
- Focus tracking during recording
- Audio recording via `AudioManager`
- Output formats: MP4 (`video/mp4`), 3GPP (`video/3gpp`)
- Time-lapse mode

### Panorama Mode

Requires LightCycle capture support (`PhotoSphereHelper`).
- Wide angle panorama
- Photo Sphere (360-degree)
- Orientation options: horizontal, vertical, wide, fisheye

### Lens Blur / Refocus Mode

Requires refocus capture support (`RefocusHelper`). Produces RGBZ depth data for post-capture refocusing.

### Tiny Planet

Post-capture effect that creates a tiny planet projection from Photo Sphere images. Uses native code (`TinyPlanetNative`).

### Capture Intent Mode (`CaptureIntentModule`)

Handles third-party capture intents (`IMAGE_CAPTURE`). State machine-based architecture with states: Background, Foreground, OpeningCamera, StartingPreview, ReadyForCapture, ReviewingPicture.

---

## Intent Filters Summary (How to Launch)

| Intent Action | Component | Result |
|---|---|---|
| `android.media.action.STILL_IMAGE_CAMERA` | CameraActivity | Opens photo mode |
| `android.intent.action.MAIN` (LAUNCHER) | CameraLauncher | Opens from home screen |
| `android.media.action.IMAGE_CAPTURE` | CaptureActivity | Third-party photo capture, returns result |
| `android.media.action.VIDEO_CAMERA` | VideoCamera | Opens video mode |
| `android.media.action.VIDEO_CAPTURE` | VideoCamera | Third-party video capture, returns result |
| `android.media.action.STILL_IMAGE_CAMERA_SECURE` | SecureCameraActivity | Lock screen photo capture |
| `android.media.action.IMAGE_CAPTURE_SECURE` | SecureCameraActivity | Lock screen photo capture intent |
| `android.intent.action.BOOT_COMPLETED` | SetActivitiesCameraReceiver | Enables/disables launcher based on camera hardware |

**Launching from another app (ADB or programmatically)**:

```bash
# Open camera
am start -a android.media.action.STILL_IMAGE_CAMERA

# Open video
am start -a android.media.action.VIDEO_CAMERA

# Capture photo (returns result)
am start -a android.media.action.IMAGE_CAPTURE

# Direct capture activity (no intent filter, must use component)
am start -n com.android.camera2/com.android.camera.DirectCaptureActivity
```

---

## Camera Architecture

### OneCamera Abstraction Layer

The app wraps Android Camera2 API behind a `OneCamera` interface:

```
OneCamera (interface)
  - startPreview(Surface, CaptureReadyCallback)
  - takePicture(PhotoCaptureParameters, CaptureSession)
  - setZoom(float)
  - triggerFocusAndMeterAtPoint(float, float)
  - close()

OneCameraCharacteristics (interface)
  - isFlashSupported()
  - isHdrSceneSupported()
  - isAutoFocusSupported()
  - isAutoExposureSupported()
  - isExposureCompensationSupported()
  - getSupportedFaceDetectModes() -> FULL, SIMPLE, NONE
  - getSupportedHardwareLevel() -> FULL, LIMITED, LEGACY
  - getAvailableMaxDigitalZoom()
  - getSupportedPictureSizes(format)
  - getSupportedPreviewSizes()

OneCameraManager
  - findFirstCameraFacing(Facing.FRONT / Facing.BACK)
  - hasCameraFacing(Facing)
  - getOneCameraCharacteristics(CameraId)

OneCameraOpener
  - open(CameraId, settings, handler, mainThread, rotationCalc, burst, sound, callback, errorHandler)
```

### AutoFocus States

The camera tracks focus through these states:
- `INACTIVE` -- Not focusing
- `PASSIVE_SCAN` -- Continuous AF scanning
- `PASSIVE_FOCUSED` -- Continuous AF locked
- `PASSIVE_UNFOCUSED` -- Continuous AF failed
- `ACTIVE_SCAN` -- User-triggered AF scanning
- `ACTIVE_FOCUSED` -- User-triggered AF locked
- `ACTIVE_UNFOCUSED` -- User-triggered AF failed

### Session Management

Photos and videos are managed through `CaptureSessionManager`:
- Session lifecycle: queued -> progress -> done/failed/canceled
- Sessions tracked by URI
- Placeholder bitmaps shown during processing
- Progress updates shown in filmstrip UI

---

## Storage

Photos saved to app-specific external pictures directory: `context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)`.

- JPEG images saved through `ContentResolver` to `MediaStore.Images.Media`
- Videos saved through `MediaRecorder` to `MediaStore.Video.Media`
- Filename format for photos: `IMG_yyyyMMdd_HHmmss.jpg`
- Filename format for panoramas: `PANO_yyyyMMdd_HHmmss`
- DirectCaptureActivity saves to: `Environment.getExternalStoragePublicDirectory(DIRECTORY_PICTURES)` as `JPEG_yyyyMMdd_HHmmss.jpg`

---

## Settings (Preference Keys)

| Key | Type | Default | Description |
|---|---|---|---|
| `pref_camera_id_key` | int | 0 (back) | Active camera: 0=back, 1=front |
| `pref_camera_flashmode_key` | string | "auto" | Flash: auto, on, off |
| `pref_camera_focusmode_key` | string | "continuous-picture" | Focus: auto, macro, infinity |
| `pref_camera_hdr_key` | boolean | false | HDR scene mode |
| `pref_camera_hdr_plus_key` | boolean | false | HDR+ (Google pipeline) |
| `pref_camera_countdown_duration_key` | int | 0 | Timer: 0 (off), 1, 3, 10, 15 seconds |
| `pref_camera_grid_lines` | boolean | false | Grid lines overlay |
| `pref_camera_exposure_key` | int | 0 | Exposure compensation value |
| `pref_camera_jpegquality_key` | string | "normal" | JPEG quality: normal, fine, superfine |
| `pref_camera_picturesize_back_key` | string | "large" | Back camera picture size |
| `pref_camera_picturesize_front_key` | string | "large" | Front camera picture size |
| `pref_video_quality_back_key` | string | "large" | Back camera video quality |
| `pref_video_quality_front_key` | string | "large" | Front camera video quality |
| `pref_camera_recordlocation_key` | boolean | false | Save GPS location in EXIF |
| `pref_camera_scenemode_key` | string | "auto" | Scene mode |
| `pref_camera_video_flashmode_key` | string | varies | Video flash mode |
| `pref_video_effect_key` | string | varies | Video effect (none, goofy faces, backdrops) |
| `pref_camera_pano_orientation` | string | "pano_horizontal" | Panorama orientation |
| `camera.startup_module` | int | 0 | Module to open on launch |

---

## Audio Resources

| File | Purpose |
|---|---|
| `shutter.ogg` | Photo capture shutter sound |
| `material_camera_focus.ogg` | Focus confirmation sound |
| `focus_complete.ogg` | Focus complete sound |
| `video_record.ogg` | Video recording start/stop |
| `timer_increment.ogg` | Countdown timer tick (2s, 3s remaining) |
| `timer_final_second.ogg` | Countdown timer final second |
| `staged_shot_complete.ogg` | Staged shot completion |

---

## Key Input Handling

The camera responds to hardware key events (relevant for Rokid glasses button navigation):

| Key Code | Action |
|---|---|
| `KEYCODE_DPAD_CENTER` (23) | Take photo (shutter) |
| `KEYCODE_CAMERA` (27) | Take photo (shutter) |
| `KEYCODE_VOLUME_UP` (24) | Take photo (on key up) |
| `KEYCODE_VOLUME_DOWN` (25) | Take photo (on key up) |

Long press on shutter button starts burst capture mode.

---

## Rokid/CXR Integration

This is a standard AOSP Camera2 build with **no Rokid CXR SDK integration**. There are no references to CXR-L, CXR-M, CXR-S, Caps serialization, Bluetooth communication, or any Rokid-specific APIs in the decompiled code.

The app operates as a standalone camera using the Android Camera2 API. It does not:
- Stream camera frames to external devices
- Integrate with Rokid AR overlay system
- Use any custom Rokid camera service
- Communicate with companion phone apps

The `DirectCaptureActivity` is the simplest entry point for programmatic camera access, using raw Camera2 API with minimal overhead (no filmstrip, no module system, no settings UI). It auto-captures after 10 preview frames, making it suitable for automated/headless capture scenarios.

---

## Package Structure

```
com.android.camera
  +-- app/                  # Application, services, managers
  |   +-- CameraApp         # Application class (init, profiling, usage stats)
  |   +-- CameraAppUI       # Main UI controller
  |   +-- CameraController  # Camera open/close lifecycle
  |   +-- LocationManager   # GPS for geotagging
  |   +-- MemoryManager     # Memory pressure handling
  |   +-- MotionManager     # Device motion tracking
  |   +-- OrientationManager # Device orientation
  +-- async/                # Threading utilities (MainThread, HandlerExecutor)
  +-- burst/                # Burst capture (BurstFacade, EvictionHandler)
  +-- captureintent/        # IMAGE_CAPTURE intent handling (state machine)
  +-- data/                 # Filmstrip data adapters (photo, video items)
  +-- debug/                # Debug logging, property helpers
  +-- device/               # Camera device tracking (CameraId, ActiveCameraDeviceTracker)
  +-- exif/                 # EXIF metadata (ExifInterface, Rational)
  +-- filmstrip/            # Gallery filmstrip UI
  +-- hardware/             # Hardware abstraction (HardwareSpec, HeadingSensor)
  +-- module/               # Module system (ModuleController, ModulesInfo)
  +-- one/                  # OneCamera abstraction layer
  |   +-- v2/               # Camera2 API implementation
  |   +-- config/           # Feature config (HDR+ support levels)
  +-- processing/           # Image processing backend
  +-- remote/               # Remote shutter interface
  +-- session/              # Capture session management
  +-- settings/             # Settings manager, keys, preferences
  +-- stats/                # Usage statistics, capture stats
  +-- tinyplanet/           # Tiny planet effect (native)
  +-- ui/                   # UI components (focus ring, mode list, preview)
  +-- util/                 # Utilities (CameraUtil, ApiHelper, GcamHelper)
  +-- widget/               # Custom widgets (FilmstripView, Preloader)
com.android.ex.camera2.portability  # Camera API portability layer
com.adobe.xmp                       # XMP metadata (embedded library)
com.bumptech.glide                  # Glide image loading (embedded library)
```
