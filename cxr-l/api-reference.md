# CXR-L SDK API Reference

Decompiled from `com.rokid.cxr:client-l:1.0.1` AAR <!-- TODO: verify AAR size for 1.0.1 (was 28KB for 0.0.1) -->. **Maven now publishes 1.0.2 (uploaded 2026-05-19) but no public changelog has been released for it yet** — the dev portal still shows 1.0.1 as the current version. See [release-notes.md](release-notes.md) for changelog.

## Overview

CXR-L is the glasses-side SDK for building standalone apps that replace the default Rokid apps entirely. It communicates with the Rokid AI app service (`com.rokid.sprite.aiapp`) via AIDL bound service.

- **Maven (current decompile)**: `com.rokid.cxr:client-l:1.0.1`
- **Maven (latest published, undocumented)**: `com.rokid.cxr:client-l:1.0.2` <!-- TODO: re-decompile against 1.0.2 once Rokid publishes its changelog -->.
- **Repository**: `https://maven.rokid.com/repository/maven-public/`
- **minSdk**: 28, **targetSdk**: 28
- **Dependencies**: `kotlin-stdlib:2.1.0`, `gson:2.10.1`
- **Network**: Allows cleartext HTTP traffic (via `network_security_config.xml`)

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

| Method | Signature | Returns |
|--------|-----------|---------|
| `setCXRLinkCallBack` | `(cb: ICXRLConnectCbk)` | `Unit` |
| `setCXRImageCbk` | `(cb: IImageStreamCbk)` | `Unit` |
| `setCXRAudioCbk` | `(cb: IAudioStreamCbk)` | `Unit` |
| `setCXRCustomViewCbk` | `(cb: ICustomViewCbk)` | `Unit` |

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

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `openCustomView` | `(data: String)` | `Boolean` | Open a custom view with JSON data. |
| `updateCustomView` | `(data: String)` | `Boolean` | Update the currently open custom view. |
| `closeCustomView` | `()` | `Boolean` | Close the custom view. |
| `isCustomViewOpened` | `()` | `Boolean` | Check if a custom view is currently active. |
| `getCurrentCustomViewData` | `()` | `String` | Get the JSON data of the current custom view. |
| `setIcons` | `(iconsJson: String)` | `Boolean` | Set display icons (JSON array of `IconInfo`). |
| `getCurrentIcons` | `()` | `String` | Get current icons JSON. |

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

## Notes

1. CXR-L does NOT access hardware directly. It communicates with the Rokid AI app's service via AIDL bound service.
2. Image capture is async: call `takePhoto()`, receive bytes in `IImageStreamCbk.onImageReceived()`.
3. Custom views use JSON strings for data, rendered by the Rokid system.
4. No continuous camera stream API exists -- only `takePhoto` for snapshots and the image callback for receiving frames.
5. Authorization flow requires the Rokid companion app (`com.rokid.sprite.aiapp`) to be installed on the glasses. Min version code: 100000.
6. The third parameter in `takePhoto` is quality (default 80), not format. Width defaults to 1920, height to 1080.
7. Audio streaming supports codec type selection via `startAudioStream(codecType)`. Codec values are undocumented.
8. `connect()` takes a token string (not a package name). The target package is hardcoded to `com.rokid.sprite.aiapp`.
9. `AuthorizationHelper.isRequiredRokidAppInstalled()` checks that `com.rokid.sprite.aiapp` versionCode >= 100000.
10. Decompiled source is available in `rokid-docs/cxr-l/decompiled/`.
