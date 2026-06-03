# CXR-L SDK API Reference

Base API decompiled from `com.rokid.cxr:client-l:1.0.1` AAR. v1.0.3 additions (new callbacks, `GlassInfo`, CUSTOMAPP session) are noted inline — they are reconstructed from a binary diff of the 1.0.2 and 1.0.3 AARs (see [release-notes.md](release-notes.md) for full details). The developer portal at `custom.rokid.com` still shows version 1.0.1 as of 2026-06-03.

## Overview

CXR-L is the mobile-side SDK for extending the Rokid AI app's use cases. The Rokid AI app manages the connection to Rokid Glasses; integrate the CXR-L SDK into your app to access the glasses' I/O capabilities — image, audio, display, and command channel — through the Rokid AI app via AIDL bound service.

- **Maven (base decompile)**: `com.rokid.cxr:client-l:1.0.1`
- **Maven (latest published)**: `com.rokid.cxr:client-l:1.0.3` (2026-06-02; provisional changelog — see [release-notes.md](release-notes.md))
- **Repository**: `https://maven.rokid.com/repository/maven-public/`
- **minSdk**: 28, **targetSdk**: 28
- **Dependencies (1.0.3)**: `kotlin-stdlib:1.6.0`, `gson:2.10.1`, `cxr-service-bridge:1.0-20260522.063600-105`
- **Network**: Allows cleartext HTTP traffic (via `network_security_config.xml`)
- **Target packages**: `com.rokid.sprite.aiapp` (primary) and `com.rokid.sprite.global.aiapp` (added in v1.0.3 for new hardware variant / region)

## Class Hierarchy

```
ExternalAppClient (com.rokid.sprite.aiapp.externalapp.example)
  └── CXRLink (com.rokid.cxr.link)
```

`CXRLink` is the entry point. It extends `ExternalAppClient` which contains all methods. `ExternalAppClient` binds to `IMediaStreamService` via Android AIDL.

## CXRLink

```kotlin
package com.rokid.cxr.link

class CXRLink(context: Context) : ExternalAppClient(context)
```

Constructor takes an Android `Context`. All public methods are inherited from `ExternalAppClient`.

## Public Methods

### Connection

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `connect` | `(token: String)` | `Boolean` | Binds to the Rokid AI app service with auth token. Internally creates intent for `com.rokid.sprite.aiapp.externalapp.MEDIA_STREAM_SERVICE` on package `com.rokid.sprite.aiapp` and passes token as `auth_token` extra. |
| `disconnect` | `()` | `Unit` | Unbinds from the service. |

### Callback Registration

| Method | Signature | Returns | Notes |
|--------|-----------|---------|-------|
| `setCXRLinkCallBack` | `(cb: ICXRLConnectCbk)` | `Unit` | v1.0.1+ |
| `setCXRImageCbk` | `(cb: IImageStreamCbk)` | `Unit` | v1.0.1+ |
| `setCXRAudioCbk` | `(cb: IAudioStreamCbk)` | `Unit` | v1.0.1+ |
| `setCXRCustomViewCbk` | `(cb: ICustomViewCbk)` | `Unit` | v1.0.1+ |
| `setCXRLinkCbk` | `(cb: ICXRLinkCbk)` | `Unit` | v1.0.3+; replaces `setCXRLinkCallBack` for device-state callbacks |

### Camera / Image

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `takePhoto` | `(width: Int = 1920, height: Int = 1080, quality: Int = 80)` | `Boolean` | Captures a photo. Defaults: 1920x1080, quality 80. Results delivered via `IImageStreamCbk.onImageReceived()`. |

### Audio

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `startAudioStream` | `(codecType: Int)` | `Boolean` | Start audio capture with specified codec. |
| `stopAudioStream` | `()` | `Boolean` | Stop audio capture. |

### Custom View (Display Rendering)

Used in the `CUSTOMVIEW` session type. Scene is ready when `openCustomView` succeeds and the glasses have acknowledged the view open event.

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `openCustomView` | `(data: String)` | `Boolean` | Open a custom view with JSON data. |
| `updateCustomView` | `(data: String)` | `Boolean` | Update the currently open custom view. |
| `closeCustomView` | `()` | `Boolean` | Close the custom view. |
| `isCustomViewOpened` | `()` | `Boolean` | Check if a custom view is currently active. |
| `getCurrentCustomViewData` | `()` | `String` | Get the JSON data of the current custom view. |
| `setIcons` | `(iconsJson: String)` | `Boolean` | Set display icons (JSON array of `IconInfo`). |
| `getCurrentIcons` | `()` | `String` | Get current icons JSON. |

### Custom App Management (CUSTOMAPP session)

Used in the `CUSTOMAPP` session type. The target glasses-side app must be installed and in the foreground before using audio, photo, or custom command capabilities.

> **Added in v1.0.3 (provisional — reconstructed from binary diff).** Exact method signatures may differ from official documentation when it is published.

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `openApp` | `(packageName: String)` | `Boolean` | Launch the specified app on the glasses (brings it to foreground). |
| `stopApp` | `(packageName: String)` | `Boolean` | Stop the running app on the glasses. |
| `isInstalled` | `(packageName: String)` | `Boolean` | Check whether a package is installed on the glasses. |
| `installApp` | `(apkPath: String, packageName: String)` | `Boolean` | Upload and install an APK on the glasses. |
| `uninstallApp` | `(packageName: String)` | `Boolean` | Uninstall a package from the glasses. |

### Service Info

| Method | Signature | Returns |
|--------|-----------|---------|
| `getServiceVersion` | `()` | `String` |
| `getServiceVersionCode` | `()` | `Integer` |

## Callback Interfaces

### ICXRLConnectCbk

```kotlin
package com.rokid.cxr.link.callbacks

interface ICXRLConnectCbk {
    fun onCXRLConnected(connected: Boolean)
}
```

### IImageStreamCbk

```kotlin
package com.rokid.cxr.link.callbacks

interface IImageStreamCbk {
    fun onImageReceived(data: ByteArray)
    fun onImageError(code: Int, msg: String)
}
```

### IAudioStreamCbk

```kotlin
package com.rokid.cxr.link.callbacks

interface IAudioStreamCbk {
    fun onAudioReceived(data: ByteArray, sampleRate: Int, channels: Int)
    fun onAudioError(code: Int, msg: String)
    fun onAudioStreamStateChanged(streaming: Boolean)
}
```

### ICustomViewCbk

```kotlin
package com.rokid.cxr.link.callbacks

interface ICustomViewCbk {
    fun onCustomViewOpened()
    fun onCustomViewUpdated()
    fun onCustomViewClosed()
    fun onCustomViewIconsSent()
    fun onCustomViewError(code: Int, msg: String)
}
```

### ICXRLinkCbk (v1.0.3+)

> **Provisional — reconstructed from binary diff of 1.0.3 AAR.** Adding this interface to your class is a **breaking change**: any class implementing it must provide all three methods.

```kotlin
package com.rokid.cxr.link.callbacks

interface ICXRLinkCbk {
    /** Fired when the SDK receives a device-state update from the glasses. */
    fun onGlassDeviceInfo(info: GlassInfo)

    /** Fired when the glasses detect a wearing / not-wearing transition. */
    fun onGlassWearingStatus(isWearing: Boolean)

    /** Fired when an in-progress AI session on the glasses is interrupted. */
    fun onGlassAiInterrupt(interrupted: Boolean)
}
```

## AIDL Service Interface (IMediaStreamService)

The underlying bound service used internally by `ExternalAppClient`. These are the raw AIDL methods:

```kotlin
package com.rokid.sprite.aiapp.externalapp

interface IMediaStreamService {
    // Image
    fun registerImageCallback(cb: IImageStreamCallback): Boolean
    fun unregisterImageCallback(cb: IImageStreamCallback): Boolean
    fun takePhoto(width: Int, height: Int, format: Int): Boolean

    // Audio
    fun registerAudioCallback(cb: IAudioStreamCallback): Boolean
    fun unregisterAudioCallback(cb: IAudioStreamCallback): Boolean
    fun startAudioStream(codecType: Int): Boolean
    fun stopAudioStream(): Boolean
    fun isAudioStreaming(): Boolean

    // Service info
    fun getServiceVersion(): String
    fun getServiceVersionCode(): Int

    // Custom view
    fun registerCustomViewCallback(cb: ICustomViewCallback): Boolean
    fun unregisterCustomViewCallback(cb: ICustomViewCallback): Boolean
    fun setIcons(iconsJson: String): Boolean
    fun openCustomView(data: String): Boolean
    fun updateCustomView(data: String): Boolean
    fun closeCustomView(): Boolean
    fun isCustomViewOpened(): Boolean
    fun getCurrentIcons(): String
    fun getCurrentCustomViewData(): String
}
```

## Utility Classes

### GlassInfo (v1.0.3+)

> **Provisional — reconstructed from binary diff of 1.0.3 AAR.**

```kotlin
package com.rokid.cxr.link.utils

data class GlassInfo(
    val deviceName: String,     // Advertised Bluetooth device name
    val batteryLevel: Int,      // Battery level (0–100)
    val sound: Int,             // Current speaker volume level
    val brightness: Int,        // Display brightness level
    val systemVersion: String,  // Glasses firmware / OS version string
    val ischarging: Boolean,    // Whether the glasses are on charge
    val sn: String,             // Device serial number
    val wearingStatus: String   // Wearing-state descriptor (raw; see ICXRLinkCbk.onGlassWearingStatus)
)
```

### IconInfo

```kotlin
package com.rokid.cxr.link.utils

class IconInfo(name: String, data: String) {
    fun getName(): String
    fun setName(name: String)
    fun getData(): String      // base64-encoded icon data
    fun setData(data: String)
}
```

### AuthorizationHelper

```kotlin
package com.rokid.sprite.aiapp.externalapp.auth

object AuthorizationHelper {
    fun requestAuthorization(activity: Activity, requestCode: Int)
    fun parseAuthorizationResult(resultCode: Int, data: Intent): AuthorizationResult
    fun isRokidAppInstalled(activity: Activity): Boolean
    fun isRequiredRokidAppInstalled(activity: Activity): Boolean
}
```

### AuthorizationResult

```kotlin
package com.rokid.sprite.aiapp.externalapp.auth

data class AuthorizationResult(val token: String)
```

## Usage Pattern

```kotlin
// 1. Create CXRLink instance
val cxrLink = CXRLink(context)

// 2. Set connection callback
cxrLink.setCXRLinkCallBack(object : ICXRLConnectCbk {
    override fun onCXRLConnected(connected: Boolean) {
        if (connected) {
            // Service is ready
        }
    }
})

// 3. Set image callback
cxrLink.setCXRImageCbk(object : IImageStreamCbk {
    override fun onImageReceived(data: ByteArray) {
        // Process captured frame
    }
    override fun onImageError(code: Int, msg: String) {
        // Handle error
    }
})

// 4. Connect with auth token (package is hardcoded internally)
cxrLink.connect(authToken)

// 5. Capture a photo (results arrive in onImageReceived)
cxrLink.takePhoto(1920, 1080, 0)

// 6. Disconnect when done
cxrLink.disconnect()
```

## Session Types and Capability Matrix

The SDK operates in one of two session modes set before calling `connect`. Capabilities differ by session:

| Session / State | Audio | Photo | Custom Command |
|-----------------|-------|-------|----------------|
| Unauthenticated / no token | No | No | No |
| Authenticated but not connected | No | No | No |
| Connected but scene not ready (View not opened / app not launched) | No | No | No |
| `CUSTOMVIEW` + custom view opened | Yes | Yes | No |
| `CUSTOMAPP` + glasses-side app opened | Yes | Yes | Yes (requires same `CXRLink` instance) |

> Source: `custom.rokid.com` CXR-L SDK intro page (Chinese), fetched 2026-06-03.

## Notes

1. CXR-L does NOT access hardware directly. It communicates with the Rokid AI app's service via AIDL bound service.
2. Image capture is async: call `takePhoto()`, receive bytes in `IImageStreamCbk.onImageReceived()`.
3. Custom views use JSON strings for data, rendered by the Rokid system.
4. No continuous camera stream API exists — only `takePhoto` for snapshots and the image callback for receiving frames.
5. Authorization flow requires the Rokid companion app (`com.rokid.sprite.aiapp` or `com.rokid.sprite.global.aiapp`) to be installed on the glasses. Min version code: 100000.
6. The third parameter in `takePhoto` is quality (default 80), not format. Width defaults to 1920, height to 1080.
7. Audio streaming supports codec type selection via `startAudioStream(codecType)`. Codec values are undocumented.
8. `connect()` takes a token string (not a package name). From v1.0.3, the SDK queries both `com.rokid.sprite.aiapp` and `com.rokid.sprite.global.aiapp` to handle different hardware variants or regions.
9. `AuthorizationHelper.isRequiredRokidAppInstalled()` checks that `com.rokid.sprite.aiapp` versionCode >= 100000.
10. Decompiled source is available in `cxr-l/decompiled/`.
11. v1.0.3 downgraded `kotlin-stdlib` from `2.1.0` to `1.6.0` as a runtime dependency. If your app targets Kotlin 2.x, declare your own explicit `kotlin-stdlib` dependency to avoid being silently downgraded by dependency resolution.
