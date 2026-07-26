# CXR-L SDK API Reference

Base API decompiled from `com.rokid.cxr:client-l:1.0.1` AAR. v1.0.3 additions (new callbacks, `GlassInfo`, CUSTOMAPP session) are noted inline; v1.0.3 entries are reconstructed from a binary diff of the 1.0.2 and 1.0.3 AARs, cross-referenced against the official Rokid changelog published 2026-06-02. v1.0.4 entries are reconstructed from a binary diff of the 1.0.3 and 1.0.4 AARs (2026-06-25); no official Rokid changelog has been published for v1.0.4. v1.1.0 entries (the new `com.rokid.cxr.session` package — see [CxrSession API (v1.1.0, provisional)](#cxrsession-api-v110-provisional) below) are reconstructed from a `javap -p` inspection of the v1.1.0 `classes.jar`, diffed against v1.0.4 (2026-07-26); no official Rokid changelog has been published for v1.1.0 either. See [release-notes.md](release-notes.md) for the full changelogs.

## Overview

CXR-L is the mobile-side SDK for extending the Rokid AI app's use cases. The Rokid AI app manages the connection to Rokid Glasses; integrate the CXR-L SDK into your app to access the glasses' I/O capabilities — image, audio, display, and command channel — through the Rokid AI app via AIDL bound service.

- **Maven (base decompile)**: `com.rokid.cxr:client-l:1.0.1`
- **Maven (latest release)**: `com.rokid.cxr:client-l:1.1.0` (uploaded 2026-07-02; confirmed live via `maven-metadata.xml`, `lastUpdated` 2026-07-18 07:24:55 UTC. No official changelog published as of 2026-07-26 — see [release-notes.md](release-notes.md#v110--uploaded-2026-07-02-no-changelog-published).)
- **Repository**: `https://maven.rokid.com/repository/maven-public/`
- **minSdk (1.0.1–1.0.2)**: 28 | **minSdk (1.0.3+)**: 31 (per official docs at `developerdoc.rokid.com`)
- **targetSdk**: not declared in AAR manifest from v1.0.4 onward (was 28 in v1.0.1–1.0.3)
- **Dependencies (1.0.3–1.0.4)**: `kotlin-stdlib:1.6.0`, `gson:2.10.1`, `cxr-service-bridge:1.0-20260522.063600-105`
- **Dependencies (1.1.0)**: `kotlin-stdlib:1.6.0`, `gson:2.10.1`, `kotlinx-coroutines-android:1.6.4` (new). `cxr-service-bridge` is **no longer a declared dependency** — the bridge/protocol/`Caps` classes and their native `.so` libraries are bundled directly inside the `client-l:1.1.0` AAR instead (see release notes).
- **Companion app requirement (1.0.3+)**: Rokid AI App (domestic) ≥ 1.7.14
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
| `setCXRSessionCbk` (via `configCXRSession`) | `(session: CxrDefs.CXRSession, cb: ICXRSessionCbk)` | `Boolean` | v1.0.4+; register session lifecycle callback at the same time as configuring the session. See `configCXRSession` below. |

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

### Session Configuration (v1.0.4+)

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `configCXRSession` | `(session: CxrDefs.CXRSession)` | `Boolean` | Configure the session type before calling `connect`. The `CXRSession` wraps a `CXRSessionType` (NONE / CUSTOMVIEW / CUSTOMAPP) and an optional `customAppPackageName`. 1-arg overload exists since v1.0.3. |
| `configCXRSession` | `(session: CxrDefs.CXRSession, cb: ICXRSessionCbk)` | `Boolean` | v1.0.4+ 2-arg overload. Configures the session and registers a session lifecycle callback atomically. |
| `getCXRSessionState` | `()` | `CxrDefs.CXRSessionState` | v1.0.4+. Query the current session state (Available / Start / Pause / Unavailable). |

### Device Controls (v1.0.4+)

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `setGlassBrightness` | `(level: Int)` | `Boolean` | v1.0.4+. Set the glasses display brightness level. Value range undocumented (inferred from binary diff). |
| `setGlassVolume` | `(level: Int)` | `Boolean` | v1.0.4+. Set the glasses speaker volume level. Value range undocumented (inferred from binary diff). |

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

### ICXRSessionCbk (v1.0.4+)

> **Provisional — reconstructed from binary diff of 1.0.4 AAR.** Adding this interface to your class is a **breaking change**: any class implementing it must provide all four methods.

```kotlin
package com.rokid.cxr.link.callbacks

interface ICXRSessionCbk {
    /** Session became available — glasses and link are ready for the app's scene. */
    fun onSessionAvailable(reason: CxrDefs.CXRSessionReason)

    /** Session started — app's scene is now active on the glasses. */
    fun onSessionStart(reason: CxrDefs.CXRSessionReason)

    /** Session paused — e.g. an OS overlay took over; scene suspended. */
    fun onSessionPause(reason: CxrDefs.CXRSessionReason)

    /** Session unavailable — link disconnected or glasses entered idle. */
    fun onSessionUnavailable(reason: CxrDefs.CXRSessionReason)
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

### CxrDefs (v1.0.4+ additions)

> **Provisional — reconstructed from binary diff of 1.0.4 AAR.** Enums below are additions; `CXRSessionType` and `CXRSession` existed from v1.0.3.

```kotlin
package com.rokid.cxr.link.utils

class CxrDefs {

    /** Session type — configure via configCXRSession() before connect(). Unchanged from v1.0.3. */
    enum class CXRSessionType { NONE, CUSTOMVIEW, CUSTOMAPP }

    /** Session configuration object passed to configCXRSession(). Unchanged from v1.0.3. */
    class CXRSession(val sessionType: CXRSessionType, val customAppPackageName: String? = null)

    /** Current session lifecycle state. Added v1.0.4. */
    enum class CXRSessionState {
        SessionAvailable,    // Session is available and ready
        SessionStart,        // Session is active (scene showing on glasses)
        SessionPause,        // Session is paused (e.g. OS overlay active)
        SessionUnavailable   // Session is unavailable (disconnected or idle)
    }

    /** Reason code supplied to all ICXRSessionCbk callbacks. Added v1.0.4. */
    enum class CXRSessionReason {
        SESSION_GLASS_READY,       // Glasses signalled ready state
        SESSION_GLASS_IDLE,        // Glasses entered idle / standby
        SESSION_LINK_CONNECT,      // CXR link connected
        SESSION_LINK_DISCONNECT,   // CXR link disconnected
        SESSION_SCREEN_OFF,        // Glasses display turned off
        SESSION_AI_START,          // On-device AI session started
        SESSION_AI_STOP,           // On-device AI session stopped
        SESSION_SCENE_TAKEOVER,    // Another scene took over the display
        SESSION_OTHER              // Other / unspecified reason
    }
}
```

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

## CxrSession API (v1.1.0, provisional)

> **Provisional — reconstructed from `javap -p` inspection of the `client-l:1.1.0` `classes.jar` (2026-07-26), not from decompiled source or official documentation.** No official Rokid changelog exists for v1.1.0 as of 2026-07-26 (see [release-notes.md](release-notes.md#v110--uploaded-2026-07-02-no-changelog-published)). Method and property names below are exact (taken directly from compiled bytecode signatures — Kotlin property names survive as `getX()`/`isX()` accessors), but behavior (retry semantics, thread affinity, exception types, default timeout values, valid ranges) is **not** confirmed and must not be assumed. This is a new package (`com.rokid.cxr.session`) added **alongside** the existing `CXRLink`/`ExternalAppClient` API described above — the older API is unchanged and still present in v1.1.0.

Internally, the new API is a facade over the same transport: `javap` on the package-private `CapabilityBroker` class shows its constructor takes a `com.rokid.sprite.aiapp.externalapp.example.ExternalAppClient` instance — i.e. `CxrSession` wraps `ExternalAppClient`/`CXRLink` rather than introducing a new wire protocol.

### CxrSessionManager

```kotlin
package com.rokid.cxr.session

interface CxrSessionManager {
    companion object {
        fun getInstance(context: Context): CxrSessionManager
    }

    fun create(config: SessionConfig): CxrSession
    fun getSession(): CxrSession
    fun requestAuthorization(
        activity: Activity,
        permissions: List<GlassPermission>,
        callback: (AuthResult) -> Unit
    )
    fun parseAuthorizationResult(resultCode: Int, data: Intent): AuthResult
    fun isRokidAppInstalled(context: Context): Boolean
    fun checkRokidAppCompatibility(context: Context): RokidAppStatus
    fun isGlassesBtConnected(): Boolean
}
```

### CxrSession

```kotlin
package com.rokid.cxr.session

interface CxrSession {
    val state: SessionState
    val stateFlow: StateFlow<SessionState>
    val config: SessionConfig

    fun connect(token: String)
    fun close()

    fun startAudioStream(): SessionResult<Unit>
    fun stopAudioStream(): SessionResult<Unit>
    fun customViewUpdate(data: String): SessionResult<Unit>
    fun takePhoto(width: Int, height: Int, quality: Int): SessionResult<Unit>
    fun sendCustomCmd(cmd: String, caps: Caps, extra: ByteArray): SessionResult<Unit>
    fun setGlassBrightness(level: Int): SessionResult<Unit>
    fun setGlassVolume(level: Int): SessionResult<Unit>
    fun queryGlassesInfo(): SessionResult<GlassesInfo>

    fun addLifecycleCallback(cb: ISessionLifecycleCbk)
    fun removeLifecycleCallback(cb: ISessionLifecycleCbk)
    fun addAudioCallback(cb: IAudioCallback)
    fun removeAudioCallback(cb: IAudioCallback)
    fun addImageCallback(cb: IImageCallback)
    fun removeImageCallback(cb: IImageCallback)
    fun addCustomCmdCallback(cb: ICustomCmdSessionCallback)
    fun removeCustomCmdCallback(cb: ICustomCmdSessionCallback)
    fun addGlassesEventListener(cb: IGlassesEventListener)
    fun removeGlassesEventListener(cb: IGlassesEventListener)
}
```

`Caps` here is `com.rokid.cxr.Caps` — now bundled directly in `client-l:1.1.0` (see [release-notes.md](release-notes.md#v110--uploaded-2026-07-02-no-changelog-published)); see [cxr-s/data-structure.md](../cxr-s/data-structure.md) for the general Caps wire format used across the CXR SDK family. `stateFlow` uses `kotlinx.coroutines.flow.StateFlow`, consistent with the new `kotlinx-coroutines-android:1.6.4` POM dependency.

### Callback interfaces

```kotlin
package com.rokid.cxr.session

interface ISessionLifecycleCbk {
    fun onSessionStarted()
    fun onSessionPaused(reason: PausedReason)
    fun onSessionResumed()
    fun onSessionTerminating(reason: TerminatingReason, graceMs: Long)
    fun onSessionClosed(reason: CloseReason)
    fun onConnectResult(success: Boolean, code: SessionErrorCode)
}

interface IGlassesEventListener {
    fun onGlassesAppResumed()
    fun onGlassesAppPaused()
    fun onWearingStatusChanged(isWearing: Boolean)
    fun onDeviceInfoChanged(info: GlassesInfo)
    fun onScreenOff()
    fun onScreenOn()
    fun onLauncherResumed()
    fun onAiWake()
    fun onAiInterruptChanged(interrupted: Boolean)
}

interface IAudioCallback {
    fun onAudioReceived(data: ByteArray)
    fun onAudioError(code: Int, msg: String)
    fun onAudioStreamStateChanged(streaming: Boolean)
}

interface IImageCallback {
    fun onImageReceived(data: ByteArray)
    fun onImageError(code: SessionErrorCode, extraCode: Int, msg: String)
}

interface ICustomCmdSessionCallback {
    fun onCustomCmdResult(cmd: String, data: ByteArray)
}
```

### Value / config types

```kotlin
package com.rokid.cxr.session

data class SessionConfig(
    val sessionType: SessionType,
    val glassesPackageName: String,
    val aiInterceptMode: AiInterceptMode,
    val terminatingGracePeriodMs: Long,
    val timeouts: SessionTimeouts,
    val viewData: String,
    val glassesActivityName: String,
    val glassesApkPath: String
)

data class SessionTimeouts(
    val connectTimeoutMs: Long,
    val takePhotoTimeoutMs: Long,
    val customCmdTimeoutMs: Long
) {
    constructor() // no-arg constructor exists; default values not confirmed from bytecode
}

data class SessionResult<T>(
    val code: SessionErrorCode,
    val data: T?,
    val message: String?
) {
    val isSuccess: Boolean
}

data class GlassesInfo(
    val osVersion: String,
    val batteryPercent: Int,
    val freeMemoryMb: Long,
    val isCharging: Boolean,
    val displayWidth: Int,
    val displayHeight: Int,
    val screenOn: Boolean
)

data class AuthResult(
    val isSuccess: Boolean,
    val token: String?,
    val errorCode: SessionErrorCode,
    val message: String?
)

sealed class RokidAppStatus {
    data class Compatible(val version: String) : RokidAppStatus()
    data class NotInstalled(val minimumVersion: String, val downloadUrl: String) : RokidAppStatus()
    data class VersionTooLow(
        val installedVersion: String,
        val minimumVersion: String,
        val downloadUrl: String
    ) : RokidAppStatus()
}
```

### Enums

```kotlin
package com.rokid.cxr.session

enum class SessionType { CUSTOM_VIEW, CUSTOM_APP }

enum class SessionState { Idle, Starting, Started, Paused, Terminating }

enum class GlassPermission { CAMERA, MICROPHONE, MEDIA }

enum class CloseReason { USER_CLOSED, CONNECT_FAILED, GLASSES_EXIT, LINK_LOST, CHAIN_TORN, FATAL_ERROR }

enum class PausedReason { AI_ASSIST, BT_DISCONNECTED }

enum class TerminatingReason {
    GLASSES_APP_EXIT, GLASSES_APP_CRASH, GLASSES_RESOURCE_RECLAIMED, LINK_CHAIN_TORN, OTHER
}

enum class AiInterceptMode { ALLOW_WITH_PAUSE, BLOCK_AI }

enum class SessionErrorCode(val code: Int, val message: String) {
    OK, NOT_AUTHENTICATED, TOKEN_EXPIRED, ROKID_APP_NOT_INSTALLED, ROKID_APP_VERSION_LOW,
    LINK_NOT_READY, BT_NOT_CONNECTED, LINK_LOST, LINK_TIMEOUT, SESSION_NOT_STARTED,
    SESSION_TERMINATING, SESSION_ALREADY_EXISTS, CONNECT_FAILED, SCENE_OPEN_FAILED,
    RESOURCE_PREPARE_FAILED, SESSION_PAUSED, DATA_NOT_READY, INVALID_ARGUMENT,
    OPERATION_IN_PROGRESS, OPERATION_CANCELLED, GLASSES_SIGNAL_TIMEOUT, GLASSES_APP_NOT_FOUND,
    GLASSES_APP_INSTALL_FAILED, GLASSES_MEMORY_PRESSURE, GLASSES_CAMERA_ERROR,
    GLASSES_AUDIO_ERROR, INTERNAL_ERROR, UNKNOWN
}
```

> `SessionErrorCode` constant order above is the enum declaration order recovered from bytecode (`values()`); numeric `code` values are not derivable from `javap` output alone and are left unspecified. `SessionConfig`, `AuthResult`, and `RokidAppStatus` field/property names are the exact Kotlin-generated getter names (`getSessionType()`, `getGlassesPackageName()`, etc.) — not inferred or guessed.

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
8. `connect()` takes a token string (not a package name). From v1.0.3, the SDK queries both `com.rokid.sprite.aiapp` and `com.rokid.sprite.global.aiapp` to handle different hardware variants or regions (domestic vs. overseas "Hi Rokid" app).
9. `AuthorizationHelper.isRequiredRokidAppInstalled()` checks that `com.rokid.sprite.aiapp` versionCode >= 100000.
10. Decompiled source is available in `cxr-l/decompiled/`.
11. v1.0.3 downgraded `kotlin-stdlib` from `2.1.0` to `1.6.0` as a runtime dependency. If your app targets Kotlin 2.x, declare your own explicit `kotlin-stdlib` dependency to avoid being silently downgraded by dependency resolution.

## Notable API changes (v1.0.2 / v1.0.3 / v1.0.4 / v1.1.0)

> Source: official Rokid changelogs at `https://developerdoc.rokid.com/sdk` (fetched 2026-06-06) for v1.0.2–v1.0.4. The v1.1.0 subsection below is **not** from an official changelog — see its own source note. The class inventory above is the 1.0.1 baseline; the changes below layer on top.

### v1.0.2 breaking changes (Android)

- **`AuthorizationHelper.requestAuthorization` signature changed.** Now requires a `GlassPermission[]` array declaring the permissions the app needs (e.g. microphone, camera, media). If the user has already authorized all requested permissions, the call can return a `Pair<resultCode, Intent>` synchronously — parse the token from that directly instead of waiting for `onActivityResult`.
- **`sendCustomCmd` now accepts `Caps` directly.** The existing byte-array overload remains; the new `sendCustomCmd(caps: Caps)` overload removes the manual serialization step.

### v1.0.3 additions (Android)

- **`minSdk` raised to 31.** Apps targeting `minSdk < 31` must update their `build.gradle` before upgrading to 1.0.3.
- **Required companion app version raised.** `com.rokid.sprite.aiapp` (Rokid AI App, domestic) must be ≥ 1.7.14.
- **`com.rokid.sprite.global.aiapp` added to `<queries>`.** The SDK now binds to either package name, enabling use with the overseas "Hi Rokid" app variant without code changes.
- **`ICXRLinkCbk` gains three new methods** (see `ICXRLinkCbk (v1.0.3+)` above; implementing classes must add stubs):
  - `onGlassDeviceInfo(info: GlassInfo)` — structured device-state snapshot.
  - `onGlassWearingStatus(isWearing: Boolean)` — wearing / not-wearing transition events.
  - `onGlassAiInterrupt(interrupted: Boolean)` — AI session interrupted by a system event.
- **Session architecture (CUSTOMVIEW / CUSTOMAPP)** — see the Session Types and Capability Matrix section above.
- **Kotlin stdlib downgraded to 1.6.0** in the POM (from 2.1.0 in 1.0.2). If your app depends on Kotlin 2.x transitively, declare your own `kotlin-stdlib` dependency at the desired version.

### v1.0.3 additions (iOS — RGCxrClient, CocoaPods)

- `CxrClient.initialize(mode:options:)` now takes a mode parameter distinguishing `customApp` / `customView`.
- Auth scopes use SDK permission enums (e.g. `.microphone`) instead of string constants.
- Most capability APIs return `RGCxrClientError?` synchronously.
- `sendCustomCmd` sends without a callback; subscribe to incoming events via `notifyEventPublisher`.

> iOS documentation and sample (`ios_cxr_l_sample`) remain at v1.0.1 as of 2026-06-06; the Android and iOS doc chapters version independently.

### v1.0.4 additions (Android — provisional, binary diff)

> Source: binary diff of `client-l:1.0.3` and `client-l:1.0.4` AARs (2026-06-25). No official changelog published.

- **Session lifecycle callbacks.** New `ICXRSessionCbk` interface with four methods: `onSessionAvailable`, `onSessionStart`, `onSessionPause`, `onSessionUnavailable` — each receives a `CXRSessionReason` enum value. Register via the new 2-arg `configCXRSession(session, callback)` overload before calling `connect()`.
- **`getCXRSessionState()` added.** Returns the current `CXRSessionState` (Available / Start / Pause / Unavailable) without needing a callback.
- **`setGlassBrightness(int)` and `setGlassVolume(int)` added.** Programmatic control of glasses display brightness and speaker volume from the mobile-side SDK. Value ranges are undocumented in the binary; consult official docs when released.
- **`CXRSessionReason` enum added** with 9 reason codes covering glass state, link state, AI events, and scene takeovers.
- **`CXRSessionState` enum added** with 4 state values mirroring the `ICXRSessionCbk` callback names.
- **`targetSdkVersion` removed from AAR manifest.** The `<uses-sdk>` element in the AAR no longer declares `targetSdkVersion`. Host apps are unaffected — their own `targetSdkVersion` in `build.gradle` takes precedence.
- **No dependency changes.** POM is identical to v1.0.3: `cxr-service-bridge:1.0-20260522.063600-105`, `kotlin-stdlib:1.6.0`, `gson:2.10.1`.

### v1.1.0 additions (Android — provisional, no changelog published)

> Source: `javap -p` inspection of `client-l:1.1.0` `classes.jar`, diffed against v1.0.4 (2026-07-26). No official changelog published; confirmed as the live Maven release only via `maven-metadata.xml`. Full method inventory in [CxrSession API (v1.1.0, provisional)](#cxrsession-api-v110-provisional) above.

- **New `com.rokid.cxr.session` package** — a coroutine-based session API (`CxrSessionManager`, `CxrSession`, plus supporting callbacks/enums/data classes) added alongside the existing `CXRLink`/`ExternalAppClient` API, which is unchanged and still present.
- **`CXRServiceBridge`/`CXRSocketProtocol`/`Caps` classes and their native `.so` libraries are now bundled directly in the AAR**; the `cxr-service-bridge` Maven dependency was dropped from the POM accordingly.
- **New runtime dependency**: `org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4`, backing the new `CxrSession.stateFlow: StateFlow<SessionState>` property.
- `AndroidManifest.xml` and `R.txt` are byte-for-byte identical to v1.0.4 — no manifest-level changes.
- No pre-existing classes were removed or renamed (name-level diff only).
