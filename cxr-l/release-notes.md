# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4, v1.1.0, v1.1.1, and v1.1.2 entries are sourced from binary diffs of Maven AARs; no official changelog has been published for any release past v1.0.3 as of 2026-09-07 (`developerdoc.rokid.com/sdk` still displays "1.0.4" as the latest CXR-L version)._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.2 — uploaded 2026-08-28 (build-only republish, provisional)

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.1` and `client-l:1.1.2` AARs (downloaded 2026-09-07 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). AAR size: 171,307 bytes vs 171,369 bytes for v1.1.1 (-0.04%).

The `.class` file inventory, `AndroidManifest.xml`, and JNI libraries are byte-identical to v1.1.1. This appears to be a build/version-metadata-only republish (`BuildConfig.VERSION_NAME` and related constants) with no observable API or dependency changes. Maven's `<release>`/`<latest>` tags point at this version.

## v1.1.1 — uploaded 2026-08-14 (provisional — corrects an anomalous v1.1.0)

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.4`, `client-l:1.1.0`, and `client-l:1.1.1` AARs (downloaded 2026-09-07). Rokid has not published a portal changelog for any of the 1.1.x releases as of 2026-09-07.

**Theme: a new Kotlin coroutine/StateFlow-based Session API, layered on top of the existing `CXRLink`/`ExternalAppClient` foundation.**

`client-l` gained a new top-level package, `com.rokid.cxr.session`, alongside the pre-existing `com.rokid.cxr.link` (`CXRLink`) and `com.rokid.sprite.aiapp.externalapp.example` (`ExternalAppClient`) classes — both of which are unchanged and still present. The new API does not replace `CXRLink`; it appears to be a higher-level, Kotlin-idiomatic wrapper (`CapabilityBroker` internally holds a reference to an `ExternalAppClient` instance).

**New entry point:**

- `com.rokid.cxr.session.CxrSessionManager` (interface, singleton via companion) — obtain with `CxrSessionManager.Companion.getInstance(context: Context): CxrSessionManager`.

  | Method | Signature | Description |
  |--------|-----------|-------------|
  | `create` | `(config: SessionConfig): CxrSession` | Create a new session for the given configuration. |
  | `getSession` | `(): CxrSession` | Retrieve the current/active session. |
  | `requestAuthorization` | `(activity: Activity, permissions: List<GlassPermission>, callback: (AuthResult) -> Unit)` | Kotlin-lambda-based authorization request (vs. the `Activity.onActivityResult` pattern used elsewhere in the SDK). |
  | `parseAuthorizationResult` | `(resultCode: Int, data: Intent): AuthResult` | Parse the result of an authorization flow. |
  | `isRokidAppInstalled` | `(context: Context): Boolean` | Check whether a companion app package is installed. |
  | `checkRokidAppCompatibility` | `(context: Context): RokidAppStatus` | Structured compatibility check (see `RokidAppStatus` below). |
  | `isGlassesBtConnected` | `(): Boolean` | Query current glasses Bluetooth connection state. |

**New session object:**

- `com.rokid.cxr.session.CxrSession` (interface) — the coroutine/`StateFlow`-based counterpart to `CXRLink`.

  | Member | Signature | Description |
  |--------|-----------|-------------|
  | `state` / `stateFlow` | `SessionState` / `StateFlow<SessionState>` | Current lifecycle state, observable reactively. |
  | `config` | `SessionConfig` | The configuration this session was created with. |
  | `connect` | `(token: String)` | Connect using an auth token. |
  | `close` | `()` | Close the session. |
  | `startAudioStream` / `stopAudioStream` | `(): SessionResult<Unit>` | Audio stream control, now returning a structured `SessionResult` instead of a bare `Boolean`. |
  | `customViewUpdate` | `(data: String): SessionResult<Unit>` | Update the custom view. |
  | `takePhoto` | `(width: Int, height: Int, quality: Int): SessionResult<Unit>` | Capture a photo. |
  | `sendCustomCmd` | `(cmd: String, caps: Caps?, data: ByteArray?): SessionResult<Unit>` | Send a custom command — accepts `Caps` and/or raw bytes in a single call. |
  | `setGlassBrightness` / `setGlassVolume` | `(level: Int): SessionResult<Unit>` | Device controls (mirrors `CXRLink.setGlassBrightness`/`setGlassVolume`, added in v1.0.4). |
  | `queryGlassesInfo` | `(): SessionResult<GlassesInfo>` | Query a structured `GlassesInfo` snapshot. |
  | `add`/`remove*Callback`/`*Listener` | — | Register/unregister `ISessionLifecycleCbk`, `IAudioCallback`, `IImageCallback`, `ICustomCmdSessionCallback`, `IGlassesEventListener`. |

**New callback interfaces** (`com.rokid.cxr.session`):

- `ISessionLifecycleCbk` — `onSessionStarted()`, `onSessionPaused(PausedReason)`, `onSessionResumed()`, `onSessionTerminating(TerminatingReason, graceMs: Long)`, `onSessionClosed(CloseReason)`, `onConnectResult(success: Boolean, SessionErrorCode)`.
- `IAudioCallback` — `onAudioReceived(ByteArray)`, `onAudioError(code: Int, msg: String)`, `onAudioStreamStateChanged(streaming: Boolean)`.
- `IImageCallback` — `onImageReceived(ByteArray)`, `onImageError(SessionErrorCode, code: Int, msg: String)`.
- `ICustomCmdSessionCallback` — `onCustomCmdResult(cmd: String, data: ByteArray)`.
- `IGlassesEventListener` — `onGlassesAppResumed()`, `onGlassesAppPaused()`, `onWearingStatusChanged(Boolean)`, `onDeviceInfoChanged(GlassesInfo)`, `onScreenOff()`, `onScreenOn()`, `onLauncherResumed()`, `onAiWake()`, `onAiInterruptChanged(Boolean)`.

**New data classes / enums** (`com.rokid.cxr.session`):

| Type | Kind | Members |
|------|------|---------|
| `SessionConfig` | data class | `sessionType: SessionType`, `glassesPackageName: String`, `aiInterceptMode: AiInterceptMode`, `terminatingGracePeriodMs: Long`, `timeouts: SessionTimeouts`, `viewData`/`viewIconData`/`glassesActivityName`/`glassesApkPath: String` |
| `SessionTimeouts` | data class | `connectTimeoutMs`, `takePhotoTimeoutMs`, `customCmdTimeoutMs: Long`; has a no-arg constructor with defaults |
| `SessionType` | enum | `CUSTOM_VIEW`, `CUSTOM_APP` — same two session kinds as `CxrDefs.CXRSessionType` in the `CXRLink` API |
| `SessionState` | enum | `Idle`, `Starting`, `Started`, `Paused`, `Terminating` |
| `SessionErrorCode` | enum (29 values, each with an `Int` code + `String` message) | `OK`, `NOT_AUTHENTICATED`, `TOKEN_EXPIRED`, `ROKID_APP_NOT_INSTALLED`, `ROKID_APP_VERSION_LOW`, `LINK_NOT_READY`, `BT_NOT_CONNECTED`, `LINK_LOST`, `LINK_TIMEOUT`, `SESSION_NOT_STARTED`, `SESSION_TERMINATING`, `SESSION_ALREADY_EXISTS`, `CONNECT_FAILED`, `SCENE_OPEN_FAILED`, `RESOURCE_PREPARE_FAILED`, `SESSION_PAUSED`, `DATA_NOT_READY`, `INVALID_ARGUMENT`, `OPERATION_IN_PROGRESS`, `OPERATION_CANCELLED`, `GLASSES_SIGNAL_TIMEOUT`, `GLASSES_APP_NOT_FOUND`, `GLASSES_APP_INSTALL_FAILED`, `GLASSES_MEMORY_PRESSURE`, `GLASSES_CAMERA_ERROR`, `GLASSES_AUDIO_ERROR`, `INTERNAL_ERROR`, `UNKNOWN` |
| `SessionResult<T>` | data class | `code: SessionErrorCode`, `data: T?`, `message: String?`, `isSuccess: Boolean` — the generic result wrapper returned by most `CxrSession` methods |
| `AiInterceptMode` | enum | `ALLOW_WITH_PAUSE`, `BLOCK_AI` — controls how the session behaves when the on-device AI assistant wants to take over |
| `GlassPermission` | enum | `CAMERA`, `MICROPHONE`, `MEDIA`, `DEVICE_MANAGE` — superset of the permission constants already used by `AuthorizationHelper.requestAuthorization` in the `CXRLink` API |
| `GlassesInfo` | data class | `osVersion: String`, `batteryPercent: Int`, `freeMemoryMb: Long`, `isCharging: Boolean`, `displayWidth`/`displayHeight: Int`, `screenOn: Boolean` — a different (larger) shape than the `CXRLink`-era `GlassInfo` (note singular naming difference) |
| `AuthResult` | data class | `isSuccess: Boolean`, `token: String?`, `errorCode: SessionErrorCode?`, `message: String?` |
| `RokidAppStatus` | sealed class | `Compatible(version: String)`, `NotInstalled(minimumVersion: String, downloadUrl: String)`, `VersionTooLow(installedVersion: String, minimumVersion: String, downloadUrl: String)` |
| `CloseReason` | enum | `USER_CLOSED`, `CONNECT_FAILED`, `GLASSES_EXIT`, `LINK_LOST`, `CHAIN_TORN`, `FATAL_ERROR` |
| `PausedReason` | enum | `AI_ASSIST`, `BT_DISCONNECTED` |
| `TerminatingReason` | enum | `GLASSES_APP_EXIT`, `GLASSES_APP_CRASH`, `GLASSES_RESOURCE_RECLAIMED`, `LINK_CHAIN_TORN`, `OTHER` |

`CapabilityBroker` (internal, package-visible surface only) wraps a `SessionConfig` + `ExternalAppClient` + a state-supplier lambda and exposes the same capability methods (`startAudioStream`, `takePhoto`, `sendCustomCmd`, etc.) that `CxrSessionImpl` delegates to — i.e. the new Session API is implemented on top of the same `ExternalAppClient` AIDL binding as `CXRLink`, not a separate transport.

**Dependency changes vs v1.0.4:**

| Dependency | v1.0.4 | v1.1.1/v1.1.2 |
|------------|--------|----------------|
| `kotlin-stdlib` | `1.6.0` | `1.9.0` |
| `kotlinx-coroutines-android` | *(absent)* | `1.9.0` *(new)* |
| `gson` | `2.10.1` | `2.10.1` (unchanged) |
| `cxr-service-bridge` | `1.0-20260522.063600-105` | `1.0-20260715.121510-107` (build-timestamp bump only; version stays `1.0`) |

**AAR resource-namespace change:** the manifest `package` attribute (used for `R` class generation, not the Java code package) changed from `com.rokid.cxr.client.extend` to `com.rokid.cxr.link`. This has no effect on the Java/Kotlin class packages, which were already `com.rokid.cxr.link.*` for the `CXRLink` API.

> **v1.1.0 anomaly (uploaded 2026-07-02, superseded 2026-08-14).** The intermediate v1.1.0 release introduced the new `com.rokid.cxr.session` package described above, but its AAR (1,286,574 bytes — 18× the size of v1.0.4) additionally bundled `com.rokid.cxr.CXRServiceBridge`, `com.rokid.cxr.CXRSocketProtocol`, `com.rokid.cxr.Caps`, and `com.rokid.cxr.RLog` — classes that belong to the CXR-M / CXR-S native bridge layer (see `cxr-s/data-structure.md` and `CLAUDE.md`'s architectural model), not previously part of `client-l` — along with five native `.so` libraries per ABI (`libcaps.so`, `libcxr-bridge-jni.so`, `libcxr-sock-proto-jni.so`, `libflora-cli.so`, `libmutils.so`). `libcaps.so`, `libflora-cli.so`, and `libcxr-sock-proto-jni.so` were byte-identical in size to the same-named libraries published for `client-m:1.2.2`, suggesting shared native sources across the CXR-M and CXR-L artifacts at that point. All of these extra classes and native libraries were removed again in v1.1.1 (AAR size dropped back to 171,369 bytes), leaving only the new `com.rokid.cxr.session` Kotlin API as the net addition. Whether v1.1.0 was ever intended for production use, or was a packaging mistake caught and reverted, is not knowable from the binaries alone — treat v1.1.0 as superseded and do not target it.

<!-- TODO: replace this section with Rokid's official changelog for the Session API once developerdoc.rokid.com publishes one. Semantics for AiInterceptMode, SessionTimeouts defaults, and the exact retry/timeout behavior of CxrSessionManagerImpl are not derivable from bytecode alone. -->

## v1.0.4 — published 2026-06-18

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.3` and `client-l:1.0.4` AARs (downloaded 2026-06-25 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). Rokid has not published a portal changelog for this release as of 2026-06-25. AAR size: 70,543 bytes vs 65,494 bytes for v1.0.3 (+7.7 %).

**Theme: structured session lifecycle callbacks and direct device controls.**

**New interfaces:**

- `com.rokid.cxr.link.callbacks.ICXRSessionCbk` — session lifecycle callback interface.

  | Method | Parameter | Description |
  |--------|-----------|-------------|
  | `onSessionAvailable` | `CXRSessionReason` | Session became available (glasses and link ready for use). |
  | `onSessionStart` | `CXRSessionReason` | Session started (app's scene is now active). |
  | `onSessionPause` | `CXRSessionReason` | Session paused (e.g. OS overlay took over; scene suspended). |
  | `onSessionUnavailable` | `CXRSessionReason` | Session became unavailable (link disconnected or glasses idle). |

  > **Breaking change for implementors of `ICXRSessionCbk`.** Any class implementing this interface must provide all four methods.

**New enums:**

- `com.rokid.cxr.link.utils.CxrDefs$CXRSessionReason` — reason code passed to all `ICXRSessionCbk` callbacks.

  | Constant | Description |
  |----------|-------------|
  | `SESSION_GLASS_READY` | Glasses signalled ready state. |
  | `SESSION_GLASS_IDLE` | Glasses entered idle / standby. |
  | `SESSION_LINK_CONNECT` | CXR link connected. |
  | `SESSION_LINK_DISCONNECT` | CXR link disconnected. |
  | `SESSION_SCREEN_OFF` | Glasses display turned off. |
  | `SESSION_AI_START` | On-device AI session started. |
  | `SESSION_AI_STOP` | On-device AI session stopped. |
  | `SESSION_SCENE_TAKEOVER` | Another scene took over the display. |
  | `SESSION_OTHER` | Other / unspecified reason. |

- `com.rokid.cxr.link.utils.CxrDefs$CXRSessionState` — current session state, queryable via `getCXRSessionState()`.

  | Constant | Meaning |
  |----------|---------|
  | `SessionAvailable` | Session is available and ready. |
  | `SessionStart` | Session is active. |
  | `SessionPause` | Session is paused. |
  | `SessionUnavailable` | Session is unavailable. |

**New public methods on `ExternalAppClient` / `CXRLink`:**

- `boolean configCXRSession(CxrDefs.CXRSession, ICXRSessionCbk)` — 2-argument overload of the existing `configCXRSession(CXRSession)`. Registers a session lifecycle callback at the same time as configuring the session type. The 1-argument overload remains available.
- `CxrDefs.CXRSessionState getCXRSessionState()` — query the current session state.
- `boolean setGlassBrightness(int)` — set the glasses display brightness level programmatically.
- `boolean setGlassVolume(int)` — set the glasses speaker volume level programmatically.

**AndroidManifest change:**

`targetSdkVersion` attribute removed from the `<uses-sdk>` element in the AAR manifest (was `"28"`). The `minSdkVersion` remains `"28"`. This is an AAR-level declaration only; host apps are unaffected.

**Dependency changes vs v1.0.3:** None — `cxr-service-bridge:1.0-20260522.063600-105`, `kotlin-stdlib:1.6.0`, and `gson:2.10.1` are unchanged.

## v1.0.3 — published 2026-06-02

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, fetched 2026-06-11).

`com.rokid.cxr:client-l:1.0.3` was uploaded to Maven on 2026-06-02 (AAR size: 65,494 bytes vs 57,145 bytes for 1.0.2, +14.6 %).

**Official changelog:**

1. Android `client-l` upgraded to 1.0.3.
2. Required companion app: when integrating `client-l:1.0.3`, Rokid AI App (China mainland) must be ≥ 1.7.14.
3. Documentation v1.0.3 rewritten from a developer-integration perspective, with unified "session construction" (会话构建) terminology throughout.
4. Android on-device Custom View chapter supplemented with a CustomView JSON Schema (LinearLayout, TextView, ImageView, RelativeLayout).
5. On-device CXR-S integration documentation merged into the CXR-L doc: SDK import, custom app integration, custom commands, key and broadcast chapters.
6. New reference sample apps published with OSS download archive: mobile-side `RenewCXRLSample` (`com.rokid.renewcxrlsample`) and glasses-side `CXRSWithCXRLSample` (`com.rokid.cxrswithcxrl`).
7. iOS documentation and sample remain at v1.0.1 — the version of iOS-specific chapters follows each platform chapter's own timeline.

**Additional technical findings (binary diff of v1.0.2 → v1.0.3 AAR):**

**New class:**

- `com.rokid.cxr.link.utils.GlassInfo` — data class representing a snapshot of connected-glasses state.

  | Field | Type | Description |
  |-------|------|-------------|
  | `deviceName` | `String` | Advertised Bluetooth device name |
  | `batteryLevel` | `int` | Battery level (0–100) |
  | `sound` | `int` | Current speaker volume level |
  | `brightness` | `int` | Display brightness level |
  | `systemVersion` | `String` | Glasses firmware / OS version string |
  | `ischarging` | `boolean` | Whether the glasses are on charge |
  | `sn` | `String` | Device serial number |
  | `wearingStatus` | `String` | Wearing-state descriptor (raw; see `onGlassWearingStatus`) |

**New callbacks on `ICXRLinkCbk`:**

- `void onGlassDeviceInfo(GlassInfo info)` — fired when the SDK receives a device-state update from the glasses. Provides a structured snapshot instead of discrete per-field queries.
- `void onGlassWearingStatus(boolean isWearing)` — fired when the glasses detect a wearing / not-wearing transition (via proximity / IMU sensor).
- `void onGlassAiInterrupt(boolean interrupted)` — fired when an in-progress AI session on the glasses is interrupted (e.g. by a system event or OS overlay).

  > **Breaking change for implementors of `ICXRLinkCbk`.** Any class implementing this interface must now implement the three new methods. Add empty stubs if the behaviour is not needed.

**AndroidManifest change:**

The AAR's `<queries>` block now also declares `com.rokid.sprite.global.aiapp` (in addition to the existing `com.rokid.sprite.aiapp`). This suggests Rokid has introduced or renamed the on-device AI app package for a new hardware variant or region — the SDK will now resolve to either package name when binding the AIDL service.

**Dependency changes vs v1.0.2:**

| Dependency | v1.0.2 | v1.0.3 |
|------------|--------|--------|
| `cxr-service-bridge` | `1.0-20260212.103714-88` | `1.0-20260522.063600-105` |
| `kotlin-stdlib` | `2.1.0` | `1.6.0` |
| `gson` | `2.10.1` | `2.10.1` |

> **Note on kotlin-stdlib downgrade.** The Kotlin stdlib runtime dependency was downgraded from 2.1.0 to 1.6.0. Apps that relied on the transitive Kotlin 2.x stdlib should declare their own `kotlin-stdlib` dependency at the desired version to avoid being silently downgraded by dependency resolution.

## v1.0.2 — published 2026-05-20

> Source: official changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, fetched 2026-06-06).

`com.rokid.cxr:client-l:1.0.2` was uploaded to Maven on 2026-05-19. Rokid published the official changelog on 2026-05-20.

**Android changes:**

1. Android `client-l` upgraded to 1.0.2.
2. **Auth API change:** `requestAuthorization` now requires a `GlassPermission` array (e.g. microphone, camera, media). If the user has already authorized, the call can return a `Pair` synchronously — parse the token directly from that.
3. **`sendCustomCmd` enhancement:** now accepts a `Caps` object directly (in addition to the existing form).
4. `CXRLSample` updated to reflect the above API changes.

**iOS changes (RGCxrClient 1.0.2):**

5. iOS `RGCxrClient` upgraded to 1.0.2 via CocoaPods; requires the Rokid specs source to be configured in your `Podfile`.
6. App startup: `CxrClient.initialize(mode:options:)` now explicitly distinguishes `customApp` / `customView` session modes.
7. Auth scopes changed from string constants to SDK permission enums (e.g. `.microphone`).
8. Most capability APIs now return `RGCxrClientError?` synchronously. `sendCustomCmd` sends without a completion callback; subscribe to events via `notifyEventPublisher`.
9. `ios_cxr_l_sample` updated to reflect the above API changes.

## v1.0.1 — 2026-05-07

1. Initial SDK release.
2. Support for obtaining authorization from the Rokid AI app.
3. Support for creating on-device custom View scenes.
4. Support for creating on-device custom app scenes.
5. Support for accessing on-device audio.
6. Support for capturing photos through the glasses.
7. Support for custom-command exchange with on-device custom apps.
