# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4 entry sourced from binary diff of Maven AARs (2026-06-25); no official changelog has been published for this release yet. v1.1.0 entry sourced from binary diff of Maven AARs (2026-08-09); as of that date `developerdoc.rokid.com/sdk` still lists 1.0.4 as the CXR-L "最新版本" (latest version) — no official changelog has been published for v1.1.0 either._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.0 — uploaded to Maven 2026-07-02

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.4` and `client-l:1.1.0` AARs (downloaded 2026-08-09 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). `maven-metadata.xml` `<lastUpdated>` reads `20260718072455` (2026-07-18); individual artifact file timestamps on the Nexus browse listing read 2026-07-02 for the `.aar`/`.pom`. Rokid has not published a portal changelog for this release as of 2026-08-09 — `developerdoc.rokid.com/sdk` still shows 1.0.4 as CXR-L's "最新版本". AAR size: 1,286,574 bytes vs 70,543 bytes for v1.0.4 (+18.2×) — the AAR now bundles native libraries that were previously shipped only inside the separate `cxr-service-bridge` dependency (see below).

**Theme: new Kotlin coroutine-based `CxrSession` API, and the CXR-S service bridge (`Caps`, `CXRServiceBridge`, `CXRSocketProtocol`) is now bundled directly into `client-l` instead of pulled in as a separate dependency.**

This is purely additive — no classes were removed and the existing `CXRLink` / `ExternalAppClient` API (documented in [api-reference.md](api-reference.md)) is unchanged and still present. Total class count in `classes.jar` grew from 66 to 160.

**AAR structure changes:**

- **Native libraries added** (`jni/arm64-v8a/`, `jni/armeabi-v7a/`): `libcaps.so`, `libcxr-bridge-jni.so`, `libcxr-sock-proto-jni.so`, `libflora-cli.so`, `libmutils.so`. These back the new `com.rokid.cxr.Caps` / `CXRServiceBridge` / `CXRSocketProtocol` classes (see below) — library names line up with the JNI-backed methods on those classes (`Caps.serialize()`/`parse()`, `CXRServiceBridge.sendMessage()`, `CXRSocketProtocol.nativeRequest()`/`nativeSend()`).
- **`proguard.txt` added** — a consumer ProGuard/R8 rules file bundled in the AAR (new in 1.1.0; absent from 1.0.4). Its header comment literally reads `# CXR-L SDK v1.1.0 — 消费者混淆规则` ("consumer obfuscation rules"), confirming the version number and the fact that the `com.rokid.cxr.session` package below is Rokid's intended new public surface (`-keep class com.rokid.cxr.session.* { public *; }` for `CxrSession`, `CxrSessionManager`, `SessionConfig`, `SessionResult`, `GlassesInfo`, `AuthResult`, and all `*Reason`/`*Code` enums; `-keep interface` for all five new callback interfaces).

**Dependency changes vs v1.0.4 (from POM diff):**

| Dependency | v1.0.4 | v1.1.0 |
|------------|--------|--------|
| `kotlin-stdlib` | `1.6.0` | `1.6.0` (unchanged) |
| `gson` | `2.10.1` | `2.10.1` (unchanged) |
| `cxr-service-bridge` | `1.0-20260522.063600-105` | **removed** — bundled directly instead (see AAR structure changes above) |
| `kotlinx-coroutines-android` | — | **`1.6.4` (new)** — backs `CxrSession.getStateFlow(): StateFlow<SessionState>` |

`AndroidManifest.xml` is byte-identical to v1.0.4 (`minSdkVersion="28"`, same `<queries>` package names, no `targetSdkVersion`).

**New package `com.rokid.cxr.session` — Kotlin coroutine/StateFlow-based session API.**

This sits *alongside* the existing `CXRLink`/`ExternalAppClient` API as a higher-level, more idiomatic-Kotlin entry point — `com.rokid.cxr.session.CapabilityBroker` (an internal implementation class in this package) wraps an `ExternalAppClient` instance under the hood, so this is a new façade over the same underlying transport rather than a parallel transport. Signatures below are extracted directly from the compiled bytecode (`javap -p`); method/parameter *names* are ground truth (preserved by the `proguard.txt` keep rules above), semantic descriptions are inferred from those names and are marked as such.

- `CxrSessionManager` — entry point / factory, replaces manually constructing `CXRLink`:

  ```kotlin
  interface CxrSessionManager {
      companion object Companion
      fun create(config: SessionConfig): CxrSession
      val session: CxrSession
      fun requestAuthorization(activity: Activity, permissions: List<GlassPermission>, callback: (AuthResult) -> Unit)
      fun parseAuthorizationResult(resultCode: Int, data: Intent): AuthResult
      fun isRokidAppInstalled(context: Context): Boolean
      fun checkRokidAppCompatibility(context: Context): RokidAppStatus
      fun isGlassesBtConnected(): Boolean
  }
  ```

- `CxrSession` — the session object itself (returned by `CxrSessionManager.create()`):

  ```kotlin
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
      fun sendCustomCmd(cmd: String, caps: Caps, data: ByteArray): SessionResult<Unit>
      fun setGlassBrightness(level: Int): SessionResult<Unit>
      fun setGlassVolume(level: Int): SessionResult<Unit>
      fun queryGlassesInfo(): SessionResult<GlassesInfo>
      fun addLifecycleCallback(cb: ISessionLifecycleCbk); fun removeLifecycleCallback(cb: ISessionLifecycleCbk)
      fun addAudioCallback(cb: IAudioCallback); fun removeAudioCallback(cb: IAudioCallback)
      fun addImageCallback(cb: IImageCallback); fun removeImageCallback(cb: IImageCallback)
      fun addCustomCmdCallback(cb: ICustomCmdSessionCallback); fun removeCustomCmdCallback(cb: ICustomCmdSessionCallback)
      fun addGlassesEventListener(cb: IGlassesEventListener); fun removeGlassesEventListener(cb: IGlassesEventListener)
  }
  ```

  Unlike the old `CXRLink` API (one callback slot per type, set via `setCXR*Cbk`), every callback category here is a multi-listener `add`/`remove` pair. `SessionResult<T>` wraps every mutating call (`{ code: SessionErrorCode, data: T?, message: String? }`, with `isSuccess` derived from `code`), replacing the old plain-`Boolean` returns — errors are now reported inline instead of only via a separate error callback.

- **New callback interfaces** (`com.rokid.cxr.session`), full signatures extracted from bytecode:

  | Interface | Methods |
  |---|---|
  | `ISessionLifecycleCbk` | `onSessionStarted()`, `onSessionPaused(PausedReason)`, `onSessionResumed()`, `onSessionTerminating(TerminatingReason, graceMs: Long)`, `onSessionClosed(CloseReason)`, `onConnectResult(success: Boolean, SessionErrorCode)` |
  | `IAudioCallback` | `onAudioReceived(ByteArray)`, `onAudioError(code: Int, msg: String)`, `onAudioStreamStateChanged(streaming: Boolean)` |
  | `IImageCallback` | `onImageReceived(ByteArray)`, `onImageError(SessionErrorCode, code: Int, msg: String)` |
  | `ICustomCmdSessionCallback` | `onCustomCmdResult(cmd: String, data: ByteArray)` |
  | `IGlassesEventListener` | `onGlassesAppResumed()`, `onGlassesAppPaused()`, `onWearingStatusChanged(Boolean)`, `onDeviceInfoChanged(GlassesInfo)`, `onScreenOff()`, `onScreenOn()`, `onLauncherResumed()`, `onAiWake()`, `onAiInterruptChanged(Boolean)` |

  `IGlassesEventListener` is new territory — screen on/off, launcher-resume, and AI-wake events were not observable through the old `CXRLink` callback set at all.

- **New data classes / enums** (`com.rokid.cxr.session`), field names from the compiled getters:

  - `SessionConfig(sessionType: SessionType, glassesPackageName: String, aiInterceptMode: AiInterceptMode, terminatingGracePeriodMs: Long, timeouts: SessionTimeouts, viewData: String?, glassesActivityName: String?, glassesApkPath: String?)` — passed to `CxrSessionManager.create()`.
  - `SessionType` enum: `CUSTOM_VIEW`, `CUSTOM_APP` (mirrors the old `CxrDefs.CXRSessionType` `CUSTOMVIEW`/`CUSTOMAPP`).
  - `SessionState` enum: `Idle`, `Starting`, `Started`, `Paused`, `Terminating` (five states, vs. the old `CXRSessionState`'s four — adds `Starting` as a distinct state before `Started`, and `Terminating` for the new graceful-close flow).
  - `SessionTimeouts(connectTimeoutMs: Long, takePhotoTimeoutMs: Long, customCmdTimeoutMs: Long)` — has a no-arg constructor (defaults not extracted from bytecode).
  - `AiInterceptMode` enum: `ALLOW_WITH_PAUSE`, `BLOCK_AI` — controls whether an on-device AI session pauses the app's session or is blocked outright while the session is active. New concept, no v1.0.x equivalent.
  - `GlassesInfo(osVersion: String, batteryPercent: Int, freeMemoryMb: Long, isCharging: Boolean, displayWidth: Int, displayHeight: Int, screenOn: Boolean)` — returned by `queryGlassesInfo()`; distinct from the older `com.rokid.cxr.link.utils.GlassInfo` (v1.0.3+, still present, delivered via the old `ICXRLinkCbk.onGlassDeviceInfo`) — the two info classes are **not** the same shape (this one adds free memory / display size / screen-on state; the old one has battery/volume/brightness/serial number/firmware version instead).
  - `GlassPermission` enum: `CAMERA`, `MICROPHONE`, `MEDIA` — a **new, separate** enum from the existing `com.rokid.sprite.aiapp.externalapp.auth.GlassPermission` used by the old `AuthorizationHelper.requestAuthorization`; both classes now coexist in the AAR. Do not assume they are interchangeable across the two auth entry points.
  - `AuthResult(isSuccess: Boolean, token: String?, errorCode: SessionErrorCode, message: String?)` — returned by the new `CxrSessionManager.requestAuthorization` callback, replacing the old `AuthorizationResult(token: String)`.
  - `RokidAppStatus` sealed class with three cases: `Compatible(version: String)`, `NotInstalled(minimumVersion: String, downloadUrl: String)`, `VersionTooLow(installedVersion: String, minimumVersion: String, downloadUrl: String)` — returned by `checkRokidAppCompatibility()`; a structured replacement for manually combining `isRokidAppInstalled()` / `isRequiredRokidAppInstalled()` version checks.
  - `CloseReason` enum: `USER_CLOSED`, `CONNECT_FAILED`, `GLASSES_EXIT`, `LINK_LOST`, `CHAIN_TORN`, `FATAL_ERROR`.
  - `PausedReason` enum: `AI_ASSIST`, `BT_DISCONNECTED`.
  - `TerminatingReason` enum: `GLASSES_APP_EXIT`, `GLASSES_APP_CRASH`, `GLASSES_RESOURCE_RECLAIMED`, `LINK_CHAIN_TORN`, `OTHER`.
  - `SessionErrorCode` enum (28 values, each carrying an `Int` `code` and `String` `message`): `OK`, `NOT_AUTHENTICATED`, `TOKEN_EXPIRED`, `ROKID_APP_NOT_INSTALLED`, `ROKID_APP_VERSION_LOW`, `LINK_NOT_READY`, `BT_NOT_CONNECTED`, `LINK_LOST`, `LINK_TIMEOUT`, `SESSION_NOT_STARTED`, `SESSION_TERMINATING`, `SESSION_ALREADY_EXISTS`, `CONNECT_FAILED`, `SCENE_OPEN_FAILED`, `RESOURCE_PREPARE_FAILED`, `SESSION_PAUSED`, `DATA_NOT_READY`, `INVALID_ARGUMENT`, `OPERATION_IN_PROGRESS`, `OPERATION_CANCELLED`, `GLASSES_SIGNAL_TIMEOUT`, `GLASSES_APP_NOT_FOUND`, `GLASSES_APP_INSTALL_FAILED`, `GLASSES_MEMORY_PRESSURE`, `GLASSES_CAMERA_ERROR`, `GLASSES_AUDIO_ERROR`, `INTERNAL_ERROR`, `UNKNOWN`.
  - `SessionResult<T>(code: SessionErrorCode, data: T?, message: String?)` with a computed `isSuccess` — generic result wrapper used throughout `CxrSession`.

**Newly-public low-level classes in `com.rokid.cxr`** (previously shipped only inside the separate `cxr-service-bridge` artifact, which `cxr-s/` docs describe as the on-device/glasses-side counterpart — see [cxr-s/data-structure.md](../cxr-s/data-structure.md) for the `Caps` wire format):

- `Caps` — binary serialization container; `write(Boolean/Int32/UInt32/Int64/UInt64/Float/Double/String/ByteArray/Caps)`, `size()`, `at(index)`, `serialize()`/`parse()` (native), `fromBytes()`. Matches the `Caps` format already documented in `cxr-s/data-structure.md`.
- `CXRServiceBridge` — glasses-side message bridge: `subscribe(topic, MsgCallback|MsgReplyCallback)`, `sendMessage(topic, Caps[, data[, offset, len]])` (native), `startAudioStream`/`stopAudioStream`/`openAudioRecord`/`closeAudioRecord`, `startBTPairing`, `sendARTCFrame`, `disconnectCXRDevice`, `appLaunch`. Error constants `EINVAL`/`EDUP`/`EFAULT`/`EBUSY`.
- `CXRSocketProtocol` — lower-level Bluetooth-socket framing/transport layer (`request`/`send`/`startAudioStream`/`getClientList`/`removeClient`, all backed by `native*` methods over a `BluetoothSocket`).
- `RLog` — logging shim (signature not extracted; low value).

> **Why this matters for the architecture model in this repo's `CLAUDE.md`:** previously `CXRSocketProtocol`/`Caps`/`CXRServiceBridge` were described as living in `cxr-service-bridge` (the CXR-S / on-device side). As of client-l 1.1.0, `client-l` also bundles them directly (with native `.so` backing) rather than depending on the `cxr-service-bridge` artifact. It is not yet clear from this binary diff alone whether this is purely an implementation-sharing detail (e.g. both artifacts built from the same internal module) or signals a deeper convergence of the CXR-M/CXR-L/CXR-S transport stack — flagging for a human to confirm against official docs once Rokid publishes the v1.1.0 changelog.

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
