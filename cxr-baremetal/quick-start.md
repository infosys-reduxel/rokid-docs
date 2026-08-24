# Bare-Metal Development Quick Start

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=63837f8f9e3d4f3387517b224e5c9875> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0**

See the [Introduction to Bare-Metal Development on Rokid Glasses](./development-guide.md) for background and scope.

## Prerequisites

- Android Studio with Gradle Kotlin DSL.
- Glasses OS: **Android 12 (API 31)** / YodaOS-Sprite.
- App **minSdk 31**, sample **targetSdk 36**, same as `GlassesBareDevSample`.
- Rokid Glasses hardware + a **development cable** (retail packaging often includes a charging cable only; debugging requires a dedicated dev cable — obtain one via the Rokid Developer Forum (`https://forum.rokid.com/index`) or official developer support).
- **Rokid AI App** on your phone to enable glasses ADB.

## Get the sample project

- **Project name**: `GlassesBareDevSample` (package `com.rokid.glassesbaredevsample`)
- **Zip package**: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR_Bare/GlassesBareDevSample.zip`

1. Download and extract the zip.
2. Android Studio → Open → select the extracted project root.
3. Sync Gradle (standard Android repositories are sufficient).
4. Build a debug APK:

   ```bash
   cd <project-root>
   ./gradlew :app:assembleDebug
   ```

   On Windows, use `gradlew.bat`.

5. Install:

   ```bash
   adb install -r app/build/outputs/apk/debug/app-debug.apk
   ```

## Enable glasses ADB

1. Connect the glasses to the PC USB port with the development cable.
2. On the phone, open glasses **ADB** in the Rokid AI App.
3. `adb devices` should list the glasses.
4. Use **scrcpy** to mirror the 480×640 display if needed.

## Minimal verification path

1. Install `GlassesBareDevSample`.
2. Launch the app → the Hub should list: Keys / wear & fold, Raw audio, Photo, Video, IMU.
3. Open **IMU** → during verification watch the gyro sliding bars; in demo click to cycle: head ball, six-axis meters, accel level (touch-panel long press to calibrate; see [IMU and Sensors](./imu-sensors.md)).
4. Open **Keys / wear & fold** → on page 2, check Key/broadcast logs (including **aborted** hints); exercise the touch panel and temple key.
5. **Photo / Video** → no live preview; double-tap from the Hub to enter, then **single tap** on the touch panel when ready:

   - **Photo**: hero `Capturing` → `Saved`; path example `/sdcard/Pictures/bare_photo/yyyyMMdd_HHmmss.jpg`
   - **Video**: tap to start/stop; after stop, `Saved`; path example `/sdcard/Video/bare_video/yyyyMMdd_HHmmss.mp4`
   - On PC: `adb pull <path shown on screen>` to verify the file

See also [GlassesBareDevSample Project and Pages](./sample-project.md) and [Glasses UI Design Guidelines (Bare Metal)](./ui-design-guidelines.md).
