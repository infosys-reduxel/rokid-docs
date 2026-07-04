# CXR-L SDK API Reference

Base API decompiled from `com.rokid.cxr:client-l:1.0.1` AAR. v1.0.3 additions (new callbacks, `GlassInfo`, CUSTOMAPP session) are noted inline; v1.0.3 entries are reconstructed from a binary diff of the 1.0.2 and 1.0.3 AARs, cross-referenced against the official Rokid changelog published 2026-06-02. v1.0.4 device-control APIs (`setGlassBrightness`/`setGlassVolume`) are now confirmed by an official Rokid changelog published 2026-06-29 (fetched 2026-07-03); the v1.0.4 session-lifecycle additions (`ICXRSessionCbk`, `CxrDefs.CXRSessionReason`, `CxrDefs.CXRSessionState`) remain reconstructed from a binary diff of the 1.0.3 and 1.0.4 AARs (2026-06-25) and are not covered by the official changelog. `client-l:1.1.0` (uploaded to Maven 2026-07-02, fetched/diffed 2026-07-04) adds a bundled native CXR wire-protocol stack (`com.rokid.cxr.Caps`/`CXRSocketProtocol`/`CXRServiceBridge`) and an entirely new `com.rokid.cxr.session` coroutine/`StateFlow`-based session API, documented in the [CxrSession API (v1.1.0+)](#cxrsession-api-v110) section below — this is a **provisional binary-diff reconstruction**, not covered by any official changelog. See [release-notes.md](release-notes.md) for the full changelogs.

## Overview

CXR-L is the mobile-side SDK for extending the Rokid AI app's use cases. The Rokid AI app manages the connection to Rokid Glasses; integrate the CXR-L SDK into your app to access the glasses' I/O capabilities — image, audio, display, and command channel — through the Rokid AI app via AIDL bound service.

- **Maven (base decompile)**: `com.rokid.cxr:client-l:1.0.1`
- **Maven (latest documented release)**: `com.rokid.cxr:client-l:1.0.4` (2026-06-18); `1.1.0` (uploaded 2026-07-02, diffed 2026-07-04) is now provisionally documented — see [CxrSession API (v1.1.0+)](#cxrsession-api-v110) — pending an official changelog.
- **Repository**: `https://maven.rokid.com/repository/maven-public/`
- **minSdk (1.0.1–1.0.2)**: 28 | **minSdk (1.0.3+)**: 31 (per official docs at `developerdoc.rokid.com`; the AAR manifest itself still declares `minSdkVersion="28"` through 1.1.0)
- **targetSdk**: not declared in AAR manifest from v1.0.4 onward (was 28 in v1.0.1–1.0.3)
- **Dependencies (1.0.3–1.0.4)**: `kotlin-stdlib:1.6.0`, `gson:2.10.1`, `cxr-service-bridge:1.0-20260522.063600-105`
- **Dependencies (1.1.0)**: `kotlin-stdlib:1.6.0`, `gson:2.10.1`, `kotlinx-coroutines-android:1.6.4` (new). `cxr-service-bridge` is no longer a POM dependency — its classes and 5 native `.so` libraries are now bundled directly inside `client-l`'s own AAR (see [release-notes.md](release-notes.md#v110--uploaded-to-maven-2026-07-02-provisional-binary-diff-reconstruction--no-official-changelog)).
- **Companion app requirement (1.0.3+)**: Rokid AI App (domestic) ≥ 1.7.14. v1.1.0 exposes this as a public constant, `AuthorizationHelper.minRokidAppRequired = 10090000` (a versionCode).
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
| `setCXRGlassAppCbk` | `(cb: IGlassAppCbk)` | `Unit` | v1.1.0+ (provisional). Public setter for the glasses-app-management callback (`appUploadAndInstall`/`appUninstall`/`appStart`/`appStop`/`appIsInstalled` results). `IGlassAppCbk` itself already existed in v1.0.4's decompile, but no public method registered it until this release. |

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
| `setGlassBrightness` | `(level: Int)` | `Boolean` | v1.0.4+. Set the glasses display brightness level. **Value range: 0–15** (confirmed by official changelog, 2026-06-29). |
| `setGlassVolume` | `(level: Int)` | `Boolean` | v1.0.4+. Set the glasses speaker volume level. **Value range: 0–15** (confirmed by official changelog, 2026-06-29). |

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

### ICXRLinkCbk (v1.0.3+, extended v1.1.0)

> **Provisional — reconstructed from binary diff of 1.0.3 AAR; extended by a v1.1.0 binary diff (2026-07-04).** Adding this interface to your class is a **breaking change**: any class implementing it must provide all methods, including the v1.1.0 addition.

```kotlin
package com.rokid.cxr.link.callbacks

interface ICXRLinkCbk {
    /** Fired when the SDK receives a device-state update from the glasses. */
    fun onGlassDeviceInfo(info: GlassInfo)

    /** Fired when the glasses detect a wearing / not-wearing transition. */
    fun onGlassWearingStatus(isWearing: Boolean)

    /** Fired when an in-progress AI session on the glasses is interrupted. */
    fun onGlassAiInterrupt(interrupted: Boolean)

    /** v1.1.0+ (provisional). Fired when the glasses-side launcher resumes to the foreground. */
    fun onGlassLauncherResume()
}
```

> Decompiled `javap -p` output for the full v1.1.0 interface (for reference; method order per the compiled class, not necessarily declaration order): `onCXRLConnected(boolean)`, `onGlassBtConnected(boolean)`, `onGlassDeviceInfo(GlassInfo)`, `onGlassWearingStatus(boolean)`, `onGlassAiAssistStart()`, `onGlassAiAssistStop()`, `onGlassAiInterrupt(boolean)`, `onGlassLauncherResume()`. The `onCXRLConnected`/`onGlassBtConnected`/`onGlassAiAssistStart`/`onGlassAiAssistStop` methods were already present in the v1.0.4 decompile (not previously listed in this doc's Kotlin sketch above) — only `onGlassLauncherResume()` is new in v1.1.0.

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

## CxrSession API (v1.1.0+)

> **Entirely new in v1.1.0 — provisional binary-diff reconstruction, not covered by any official changelog** (source: diff of `client-l:1.0.4` vs `client-l:1.1.0` AARs, fetched/diffed 2026-07-04; see [release-notes.md](release-notes.md#v110--uploaded-to-maven-2026-07-02-provisional-binary-diff-reconstruction--no-official-changelog)). This is a parallel API surface living in package `com.rokid.cxr.session`, alongside (not replacing) the existing `CXRLink`/`ExternalAppClient` callback API documented above. Internally, the sole implementation (`CxrSessionImpl`, package-private) wraps an `ExternalAppClient` instance — the two APIs share one underlying connection. Confirmed as genuine public API (not just an internal implementation detail) because v1.1.0's bundled `proguard.txt` explicitly `-keep`s these exact types as `public`.
>
> All signatures below come from `javap -p` disassembly of the v1.1.0 `classes.jar`; method **bodies** were not decompiled (no `jadx`/`apktool` available in this pass — see the Tooling note in the v1.1.0 release-notes entry), so behavior is inferred only from names/types, not verified against implementation.

### CxrSessionManager

Entry point for the new API.

```kotlin
package com.rokid.cxr.session

interface CxrSessionManager {
    companion object {
        fun getInstance(context: Context): CxrSessionManager
    }

    fun create(config: SessionConfig): CxrSession
    fun getSession(): CxrSession
    fun requestAuthorization(activity: Activity, permissions: List<GlassPermission>, callback: (AuthResult) -> Unit)
    fun parseAuthorizationResult(resultCode: Int, data: Intent): AuthResult
    fun isRokidAppInstalled(context: Context): Boolean
    fun checkRokidAppCompatibility(context: Context): RokidAppStatus
    fun isGlassesBtConnected(): Boolean
}
```

### CxrSession

The session object returned by `CxrSessionManager.create()` / `getSession()`.

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
    fun sendCustomCmd(name: String, caps: Caps, data: ByteArray): SessionResult<Unit>
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
    fun addGlassesEventListener(listener: IGlassesEventListener)
    fun removeGlassesEventListener(listener: IGlassesEventListener)
}
```

Compared to the v1.0.4 `configCXRSession(CXRSession, ICXRSessionCbk)` + `getCXRSessionState()` pattern, `CxrSession` replaces the single callback + 4-state `CxrDefs.CXRSessionState` with: a Kotlin `StateFlow<SessionState>` for reactive observation, a 5-value `SessionState` state machine, typed `SessionResult<T>` return values instead of raw `Boolean`/`Integer`, and up to 5 independently add/remove-able callback interfaces instead of one monolithic callback.

### SessionConfig

```kotlin
package com.rokid.cxr.session

class SessionConfig(
    val sessionType: SessionType,
    val glassesPackageName: String?,
    val aiInterceptMode: AiInterceptMode,
    val terminatingGracePeriodMs: Long,
    val timeouts: SessionTimeouts,
    val viewData: String?,
    val glassesActivityName: String?,
    val glassesApkPath: String?
)
```

A Kotlin data class (`copy()`/`component1..8()`/`equals`/`hashCode`/`toString` all present in the decompile). Field names are decompiled getter names (`getSessionType`, `getGlassesPackageName`, etc.) rendered back to property form.

### SessionTimeouts

```kotlin
package com.rokid.cxr.session

class SessionTimeouts(
    val connectTimeoutMs: Long = /* default via no-arg constructor */,
    val takePhotoTimeoutMs: Long = /* default via no-arg constructor */,
    val customCmdTimeoutMs: Long = /* default via no-arg constructor */
) {
    constructor()  // no-arg convenience constructor also present in the decompile
}
```

The specific default timeout values are not visible via `javap` signatures alone (they're inside the no-arg constructor's bytecode body, which was not decompiled this pass).

### SessionResult\<T\>

```kotlin
package com.rokid.cxr.session

class SessionResult<T>(
    val code: SessionErrorCode,
    val data: T?,
    val message: String?
) {
    val isSuccess: Boolean
}
```

### GlassesInfo

```kotlin
package com.rokid.cxr.session

class GlassesInfo(
    val osVersion: String,
    val batteryPercent: Int,
    val freeMemoryMb: Long,
    val isCharging: Boolean,
    val displayWidth: Int,
    val displayHeight: Int,
    val screenOn: Boolean
)
```

Note this is a distinct type from `com.rokid.cxr.link.utils.GlassInfo` (v1.0.3+, used by the `ExternalAppClient`/`ICXRLinkCbk` API) — different package, different field set (`GlassesInfo` adds `freeMemoryMb`, `displayWidth`/`displayHeight`, `screenOn`; it drops `deviceName`/`sn`/`wearingStatus` from the older `GlassInfo`). The two are not interchangeable.

### RokidAppStatus (sealed hierarchy)

```kotlin
package com.rokid.cxr.session

sealed class RokidAppStatus {
    class Compatible(val version: String) : RokidAppStatus()
    class NotInstalled(val minimumVersion: String, val downloadUrl: String) : RokidAppStatus()
    class VersionTooLow(val installedVersion: String, val minimumVersion: String, val downloadUrl: String) : RokidAppStatus()
}
```

Returned by `CxrSessionManager.checkRokidAppCompatibility(Context)` — a structured replacement for the old boolean `isRokidAppInstalled()`/`isRequiredRokidAppInstalled()` checks, distinguishing "not installed" from "installed but too old" and carrying a suggested download URL.

### AuthResult (session variant)

```kotlin
package com.rokid.cxr.session

class AuthResult(
    val isSuccess: Boolean,
    val token: String?,
    val errorCode: SessionErrorCode,
    val message: String?
)
```

Distinct from `com.rokid.sprite.aiapp.externalapp.auth.AuthResult` (the sealed `AuthCancel`/`AuthFail`/`AuthSuccess` type used by the older `AuthorizationHelper`). `CxrSessionManagerImpl` decompiles to a private static bridge method converting the old `AuthResult` into this new one, so the new type is a facade over the same underlying authorization flow, not a new flow.

### Callback interfaces

```kotlin
package com.rokid.cxr.session

interface ISessionLifecycleCbk {
    fun onSessionStarted()
    fun onSessionPaused(reason: PausedReason)
    fun onSessionResumed()
    fun onSessionTerminating(reason: TerminatingReason, gracePeriodMs: Long)
    fun onSessionClosed(reason: CloseReason)
    fun onConnectResult(success: Boolean, errorCode: SessionErrorCode)
}

interface IAudioCallback {
    fun onAudioReceived(data: ByteArray)
    fun onAudioError(code: Int, msg: String)
    fun onAudioStreamStateChanged(streaming: Boolean)
}

interface IImageCallback {
    fun onImageReceived(data: ByteArray)
    fun onImageError(errorCode: SessionErrorCode, code: Int, msg: String)
}

interface ICustomCmdSessionCallback {
    fun onCustomCmdResult(name: String, data: ByteArray)
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
```

All five interfaces decompile with Kotlin `DefaultImpls` companion classes, meaning each has at least one method with a default body in the original Kotlin source — which methods have defaults is not visible via `javap` signatures alone.

### Session enums

```kotlin
package com.rokid.cxr.session

enum class SessionType { CUSTOM_VIEW, CUSTOM_APP }

enum class SessionState { Idle, Starting, Started, Paused, Terminating }

enum class AiInterceptMode { ALLOW_WITH_PAUSE, BLOCK_AI }

enum class CloseReason { USER_CLOSED, CONNECT_FAILED, GLASSES_EXIT, LINK_LOST, CHAIN_TORN, FATAL_ERROR }

enum class PausedReason { AI_ASSIST, BT_DISCONNECTED }

enum class TerminatingReason { GLASSES_APP_EXIT, GLASSES_APP_CRASH, GLASSES_RESOURCE_RECLAIMED, LINK_CHAIN_TORN, OTHER }

enum class GlassPermission { CAMERA, MICROPHONE, MEDIA }  // com.rokid.cxr.session.GlassPermission — a second, distinct enum from
                                                            // com.rokid.sprite.aiapp.externalapp.auth.GlassPermission used by AuthorizationHelper
```

`SessionErrorCode` is the largest of the new enums (29 constants) and carries a `code: Int` + `message: String` per constant:

```kotlin
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

The per-constant `code`/`message` values themselves are not visible via `javap -p` signatures (they're arguments passed in the static initializer's bytecode, which was not fully decompiled) — only the constant names and the fact that the enum carries these two typed fields are confirmed.

### Internal implementation (not public API — for context only)

- `CxrSessionImpl` (package-private, `final class ... implements CxrSession`) is the sole implementation. It holds a `com.rokid.sprite.aiapp.externalapp.example.ExternalAppClient` field and a `CapabilityBroker` field, confirming the new API is built on top of the same AIDL-bound-service connection as `CXRLink`, not a separate transport.
- `CapabilityBroker` (public class, but **not** listed in the v1.1.0 `proguard.txt`'s public-API keep rules — likely an implementation detail exposed with default Kotlin `public` visibility rather than an intentional API surface) mirrors `CxrSession`'s capability methods (`startAudioStream`, `takePhoto`, `sendCustomCmd`, `setGlassBrightness`, `setGlassVolume`, `queryGlassesInfo`) and delegates to the wrapped `ExternalAppClient`.
- `CxrSessionManagerImpl` is a process-wide singleton (`getInstance(Context)` backed by a `volatile static` field with double-checked-locking-style access, per an `AtomicInteger` reference-count field also present).

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

    // v1.1.0+ (provisional binary diff, 2026-07-04): public constant, not previously visible.
    const val minRokidAppRequired: Int = 10090000
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
8. `connect()` takes a token string (not a package name). From v1.0.3, the SDK queries both `com.rokid.sprite.aiapp` and `com.rokid.sprite.global.aiapp` to handle different hardware variants or regions (domestic vs. overseas "Hi Rokid" app).
9. `AuthorizationHelper.isRequiredRokidAppInstalled()` checks that `com.rokid.sprite.aiapp` versionCode >= 100000. As of v1.1.0 (2026-07-04 binary diff), `AuthorizationHelper` also exposes this threshold as a public constant, `minRokidAppRequired = 10090000` — a much larger, more plausible-looking real-world versionCode than the `100000` figure this note has carried since an earlier pass. Which of the two values `isRequiredRokidAppInstalled()` actually compares against at runtime was not independently re-verified this pass (method bodies were not decompiled); both figures are recorded here rather than silently discarding the older one.
10. Decompiled source is available in `cxr-l/decompiled/`.
11. v1.0.3 downgraded `kotlin-stdlib` from `2.1.0` to `1.6.0` as a runtime dependency. If your app targets Kotlin 2.x, declare your own explicit `kotlin-stdlib` dependency to avoid being silently downgraded by dependency resolution.
12. v1.1.0 bundles its own copy of the native CXR wire-protocol stack (`Caps`/`CXRSocketProtocol`/`CXRServiceBridge`, formerly the external `cxr-service-bridge` Maven dependency) plus a new coroutine/`StateFlow`-based `com.rokid.cxr.session` API alongside the existing `CXRLink`/`ExternalAppClient` API — see [CxrSession API (v1.1.0+)](#cxrsession-api-v110) and [release-notes.md](release-notes.md#v110--uploaded-to-maven-2026-07-02-provisional-binary-diff-reconstruction--no-official-changelog). No official changelog covers this release as of 2026-07-04.

## Notable API changes (v1.0.2 / v1.0.3 / v1.0.4 / v1.1.0)

> Source: official Rokid changelogs at `https://developerdoc.rokid.com/sdk` (fetched 2026-06-06) for v1.0.2–v1.0.4. The v1.1.0 subsection below has **no official changelog** — it is a provisional binary-diff reconstruction (fetched/diffed 2026-07-04). The class inventory above is the 1.0.1 baseline; the changes below layer on top.

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

### v1.0.4 additions (Android — official changelog: device control; provisional binary diff: session lifecycle)

> Device-control APIs: source is the official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, published 2026-06-29, fetched 2026-07-03). Session-lifecycle APIs: source is a binary diff of `client-l:1.0.3` and `client-l:1.0.4` AARs (2026-06-25) — the official changelog does not mention these, so they remain provisional.

- **`setGlassBrightness(int)` and `setGlassVolume(int)` added.** Programmatic control of glasses display brightness and speaker volume from the mobile-side SDK. **Value range confirmed as 0–15** by the official changelog.
- **New "设备控制" (Device Control) documentation chapter** published for both Android and iOS, covering brightness/volume set and query.
- **Session lifecycle callbacks (provisional).** New `ICXRSessionCbk` interface with four methods: `onSessionAvailable`, `onSessionStart`, `onSessionPause`, `onSessionUnavailable` — each receives a `CXRSessionReason` enum value. Register via the new 2-arg `configCXRSession(session, callback)` overload before calling `connect()`.
- **`getCXRSessionState()` added (provisional).** Returns the current `CXRSessionState` (Available / Start / Pause / Unavailable) without needing a callback.
- **`CXRSessionReason` enum added (provisional)** with 9 reason codes covering glass state, link state, AI events, and scene takeovers.
- **`CXRSessionState` enum added (provisional)** with 4 state values mirroring the `ICXRSessionCbk` callback names.
- **`targetSdkVersion` removed from AAR manifest.** The `<uses-sdk>` element in the AAR no longer declares `targetSdkVersion`. Host apps are unaffected — their own `targetSdkVersion` in `build.gradle` takes precedence.
- **No dependency changes.** POM is identical to v1.0.3: `cxr-service-bridge:1.0-20260522.063600-105`, `kotlin-stdlib:1.6.0`, `gson:2.10.1`.

### v1.0.4 additions (iOS — RGCxrClient, CocoaPods)

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, published 2026-06-29, fetched 2026-07-03).

- `RGCxrClient` gains `setBrightness()` / `getBrightness()` / `setVolume()` / `getVolume()`, mirroring the Android device-control APIs.
- `RGCxrDeviceInfo` gains `brightness` / `sound` fields.
- iOS documentation and sample (`ios_cxr_l_sample`) are now version-aligned to v1.0.4 (previously pinned at v1.0.1 as of the v1.0.3 changelog check).

### v1.1.0 additions (Android — entirely provisional binary diff; no official changelog; no iOS artifact observed)

> Source: binary diff of `client-l:1.0.4` and `client-l:1.1.0` AARs, both downloaded from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/` on 2026-07-04. `developerdoc.rokid.com/sdk` (CXR-L tab) still shows only the v1.0.4 changelog (dated 2026-06-29) as of this fetch — nothing here is confirmed by an official source. See the [v1.1.0 entry in release-notes.md](release-notes.md#v110--uploaded-to-maven-2026-07-02-provisional-binary-diff-reconstruction--no-official-changelog) for the full write-up including tooling notes and what could not be characterized.

- **New `com.rokid.cxr.session` package (provisional)** — a parallel, coroutine/`StateFlow`-based session API (`CxrSessionManager`, `CxrSession`, `SessionConfig`, `SessionResult<T>`, and 5 callback interfaces) alongside the existing `CXRLink`/`ExternalAppClient` API. See [CxrSession API (v1.1.0+)](#cxrsession-api-v110) above for full decompiled signatures.
- **`ICXRLinkCbk` gains `onGlassLauncherResume()` (provisional).** Breaking change for implementors — add a stub.
- **`ExternalAppClient` gains public `setCXRGlassAppCbk(IGlassAppCbk)` (provisional)** — the first public way to register the pre-existing `IGlassAppCbk` callback.
- **`AuthorizationHelper` gains public constant `minRokidAppRequired = 10090000` (provisional, confirmed real constant value via bytecode).**
- **`cxr-service-bridge` is no longer an external POM dependency — it is now bundled inside `client-l`'s own `classes.jar` and `jni/{arm64-v8a,armeabi-v7a}/` folders.** Confirmed via POM diff (dependency entry removed) and via embedded compiler debug-path strings inside `libflora-cli.so` referencing the `cxr-service-bridge` module's own C++ sources.
- **New dependency: `kotlinx-coroutines-android:1.6.4`** (runtime scope), required by the new `StateFlow`/`MutableStateFlow`-based session API.
- **New bundled `proguard.txt`** (consumer ProGuard/R8 rules), explicitly commented `CXR-L SDK v1.1.0` in Chinese — independently corroborates the version number and the intended public surface of the new `com.rokid.cxr.session` types.
- **AndroidManifest.xml is unchanged** (byte-identical to v1.0.4).
- **AAR size: 1,286,574 bytes vs 70,543 bytes for v1.0.4 (+1,724 %)** — the largest point-release jump in this SDK's history, split between ~2.4 MB (uncompressed, both ABIs) of newly-bundled native `.so` libraries and genuine new Java/Kotlin bytecode (`classes.jar` grew from 66 to 160 class files, 79,030 → 203,274 bytes).
- **Not characterized this pass:** the internal contents of `libmutils.so` and the deeper internals of `libcxr-sock-proto-jni.so`/`libflora-cli.so` beyond their JNI method bindings and embedded strings — no `jadx`/`apktool`/native disassembler was available in this environment.
