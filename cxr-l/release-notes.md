# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4 entry sourced from binary diff of Maven AARs (2026-06-25); no official changelog has been published for this release yet._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## Documentation gap (2026-09-06)

`com.rokid.cxr:client-l` has published three releases beyond v1.0.4 on Maven — **v1.1.0** (2026-07-02), **v1.1.1** (2026-08-14), and **v1.1.2** (2026-08-28) — none of which are reflected in the official portal. As of this check, `https://developerdoc.rokid.com/sdk` still lists CXR-L's latest version as **1.0.4**, "更新于 2026.06.25". The per-SDK detail-doc CMS at `https://custom.rokid.com/prod/rokid_web/` (workspace `84feb39f8ef141b0ad0326f902ab881f`) remains unreachable (`NoSuchKey` from the OSS bucket, unchanged since the 2026-06-28 check), so no official changelog text exists to translate for any of these three releases.

The v1.1.0 / v1.1.1 / v1.1.2 sections below are **reverse-inferred from binary diffs** of the AARs, following the same method already used for v1.0.4 and for the [CXR-M release notes](../cxr-m/release-notes.md). They describe the *observable* public-API delta only — class/method/field names and signatures pulled directly from `javap -p` output against `classes.jar` inside each AAR — not documented intent or behavior. Treat them as a best-effort developer aid until Rokid publishes the authoritative changelog.

Method to reproduce (run any time):

```sh
# Download
for V in 1.0.4 1.1.0 1.1.1 1.1.2; do
  curl -O "https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/$V/client-l-$V.aar"
done
# Unpack AAR -> classes.jar -> .class files, then per class:
#   javap -p -classpath <extracted-dir> <fully.qualified.ClassName>
# Diff class-name lists and javap output across versions.
```

<!-- TODO: replace the inferred sections below with Rokid's official changelog when it is published. -->

## Changelog

### v1.1.2 — uploaded 2026-08-28 (inferred from binary diff)

> **Provisional — not an official Rokid changelog.** AAR size: 171,307 bytes (−0.03% vs v1.1.1, effectively unchanged).

No class was added, removed, or changed at the public-API level versus v1.1.1 (`javap -p` output is byte-for-byte identical for every class checked, including `CxrSession`, `CxrSessionManager`, `SessionConfig`, and `CXRLink`). `AndroidManifest.xml`, `proguard.txt`, and `R.txt` are also unchanged. This release appears to be a rebuild of v1.1.1 with no observable API or resource impact — likely an internal/build-only republish.

### v1.1.1 — uploaded 2026-08-14 (inferred from binary diff)

> **Provisional — not an official Rokid changelog.** AAR size: 171,369 bytes (−86.7% vs v1.1.0 — the bundled native libraries from v1.1.0 are gone; see below).

**Reverted from v1.1.0:** the `jni/` directory (native `.so` libraries) and the following classes are removed again: `com.rokid.cxr.Caps` (+ `Binary`/`Value`/`IncorrectTypeException` inner types), `com.rokid.cxr.CXRServiceBridge` (+ inner callback/param types), `com.rokid.cxr.CXRSocketProtocol` (+ inner types), `com.rokid.cxr.RLog`. The `com.rokid.cxr.session.*` package introduced in v1.1.0 (see below) is **retained** — it does not depend on the reverted native layer at the bytecode level (its `Caps` reference in `CxrSession.sendCustomCmd` resolves to the pre-existing `com.rokid.cxr.Caps` type shipped separately via the `cxr-service-bridge` dependency, not the AAR-bundled copy that briefly existed in v1.1.0).

**Dependency changes vs v1.1.0** (from the published POM):

| Dependency | v1.1.0 | v1.1.1 |
|---|---|---|
| `cxr-service-bridge` | *(dependency removed in 1.1.0)* | `1.0-20260715.121510-107` (re-added) |
| `kotlin-stdlib` | `1.6.0` | `1.9.0` |
| `kotlinx-coroutines-android` | `1.6.4` | `1.9.0` |
| `gson` | `2.10.1` | `2.10.1` (unchanged) |

**AndroidManifest / AIDL:** the `IAiEventCallback`, `IAudioStreamCallback`, `ICustomCmdCallback`, `ICustomViewCallback`, `IDeviceStatusCallback`, `IGlassAppCallback`, and `IImageStreamCallback` AIDL-generated `$Stub$a` proxy inner classes were recompiled (byte-identical public signatures; internal anonymous-class naming shuffled — consistent with a Kotlin/AIDL toolchain bump, not an API change).

Net effect: v1.1.1 looks like a course-correction after v1.1.0 accidentally (or experimentally) bundled a native socket/Caps layer that duplicated functionality already provided by the separate `cxr-service-bridge` artifact; v1.1.1 drops that duplication and keeps only the new Kotlin session API.

### v1.1.0 — uploaded 2026-07-02 (inferred from binary diff)

> **Provisional — not an official Rokid changelog.** Reconstructed from the public-API diff between `client-l:1.0.4` and `client-l:1.1.0` AARs (downloaded 2026-09-06 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). AAR size: 1,286,574 bytes (vs 70,543 bytes for v1.0.4 — the jump is almost entirely the newly bundled `jni/` native libraries, see below). Class count in `classes.jar` grew from 66 to 142.

**Theme: new coroutine-based `com.rokid.cxr.session` API, added alongside the existing `CXRLink` / `ExternalAppClient` API (no breaking changes to the latter — `javap` output for `CXRLink` is byte-identical to v1.0.4).**

This is the largest addition to CXR-L observed since v1.0.1. Rokid has not published any documentation, sample code, or changelog for it as of this check; the class/method inventory below is derived entirely from `javap -p` against the compiled bytecode, cross-referenced against the existing `CxrDefs` / `ICXRLinkCbk` vocabulary in the pre-existing API. **Treat method names and comments as observed signatures, not confirmed semantics** — none of this has been exercised at runtime.

**New top-level interface — `com.rokid.cxr.session.CxrSession`** (session handle, Kotlin coroutine/`StateFlow`-based):

| Member | Signature | Notes |
|---|---|---|
| `getState()` / `getStateFlow()` | `SessionState` / `StateFlow<SessionState>` | Current state and an observable stream of it |
| `getConfig()` | `SessionConfig` | The config the session was created with |
| `connect(String)` | `void` | Takes a token (parallels `CXRLink`'s auth flow) |
| `close()` | `void` | |
| `startAudioStream()` / `stopAudioStream()` | `SessionResult<Unit>` | |
| `customViewUpdate(String)` | `SessionResult<Unit>` | Pushes custom-View JSON, same shape as `CXRLink`'s existing custom-View channel |
| `takePhoto(int, int, int)` | `SessionResult<Unit>` | Likely `(width, height, quality)` by analogy with the existing photo-capture API — unconfirmed |
| `sendCustomCmd(String, Caps, byte[])` | `SessionResult<Unit>` | |
| `setGlassBrightness(int)` / `setGlassVolume(int)` | `SessionResult<Unit>` | Session-scoped equivalents of the v1.0.4 `CXRLink` methods of the same name |
| `queryGlassesInfo()` | `SessionResult<GlassesInfo>` | |
| `add/removeLifecycleCallback` | `(ISessionLifecycleCbk)` | |
| `add/removeAudioCallback` | `(IAudioCallback)` | |
| `add/removeImageCallback` | `(IImageCallback)` | |
| `add/removeCustomCmdCallback` | `(ICustomCmdSessionCallback)` | |
| `add/removeGlassesEventListener` | `(IGlassesEventListener)` | |

**New top-level interface — `com.rokid.cxr.session.CxrSessionManager`** (session factory / authorization entry point):

| Member | Signature | Notes |
|---|---|---|
| `create(SessionConfig)` | `CxrSession` | |
| `getSession()` | `CxrSession` | Presumably the current/singleton session |
| `requestAuthorization(Activity, List<GlassPermission>, (AuthResult) -> Unit)` | `void` | Kotlin-lambda callback form of the existing authorization flow |
| `parseAuthorizationResult(int, Intent)` | `AuthResult` | For parsing an `onActivityResult` callback |
| `isRokidAppInstalled(Context)` | `boolean` | |
| `checkRokidAppCompatibility(Context)` | `RokidAppStatus` | Sealed result type, see below |
| `isGlassesBtConnected()` | `boolean` | |

**New data/config classes** (Kotlin data classes, fields reconstructed from `javap` getters):

- `SessionConfig(sessionType: SessionType, glassesPackageName: String, aiInterceptMode: AiInterceptMode, terminatingGracePeriodMs: Long, timeouts: SessionTimeouts, viewData: String, viewIconData: String, glassesActivityName: String, glassesApkPath: String)`
- `SessionTimeouts(connectTimeoutMs: Long, takePhotoTimeoutMs: Long, customCmdTimeoutMs: Long)` — has a no-arg constructor, so likely ships with defaults.
- `GlassesInfo(osVersion: String, batteryPercent: Int, freeMemoryMb: Long, isCharging: Boolean, displayWidth: Int, displayHeight: Int, screenOn: Boolean)`
- `SessionResult<T>(code: SessionErrorCode, data: T?, message: String?)` with an `isSuccess` getter — generic result wrapper used throughout the new API.
- `AuthResult(isSuccess: Boolean, token: String?, errorCode: SessionErrorCode, message: String?)`
- `RokidAppStatus` — sealed class with three subtypes: `Compatible(version: String)`, `NotInstalled(minimumVersion: String, downloadUrl: String)`, `VersionTooLow(installedVersion: String, minimumVersion: String, downloadUrl: String)`.

**New enums:**

| Enum | Constants |
|---|---|
| `SessionType` | `CUSTOM_VIEW`, `CUSTOM_APP` — mirrors the existing `CxrDefs.CXRSessionType` vocabulary |
| `AiInterceptMode` | `ALLOW_WITH_PAUSE`, `BLOCK_AI` |
| `SessionState` | `Idle`, `Starting`, `Started`, `Paused`, `Terminating` |
| `GlassPermission` | `CAMERA`, `MICROPHONE`, `MEDIA`, `DEVICE_MANAGE` |
| `CloseReason` | `USER_CLOSED`, `CONNECT_FAILED`, `GLASSES_EXIT`, `LINK_LOST`, `CHAIN_TORN`, `FATAL_ERROR` |
| `PausedReason` | `AI_ASSIST`, `BT_DISCONNECTED` |
| `TerminatingReason` | `GLASSES_APP_EXIT`, `GLASSES_APP_CRASH`, `GLASSES_RESOURCE_RECLAIMED`, `LINK_CHAIN_TORN`, `OTHER` |
| `SessionErrorCode` | 28 constants including `OK`, `NOT_AUTHENTICATED`, `TOKEN_EXPIRED`, `ROKID_APP_NOT_INSTALLED`, `ROKID_APP_VERSION_LOW`, `LINK_NOT_READY`, `BT_NOT_CONNECTED`, `LINK_LOST`, `LINK_TIMEOUT`, `SESSION_NOT_STARTED`, `SESSION_TERMINATING`, `SESSION_ALREADY_EXISTS`, `CONNECT_FAILED`, `SCENE_OPEN_FAILED`, `RESOURCE_PREPARE_FAILED`, `SESSION_PAUSED`, `DATA_NOT_READY`, `INVALID_ARGUMENT`, `OPERATION_IN_PROGRESS`, `OPERATION_CANCELLED`, `GLASSES_SIGNAL_TIMEOUT`, `GLASSES_APP_NOT_FOUND`, `GLASSES_APP_INSTALL_FAILED`, `GLASSES_MEMORY_PRESSURE`, `GLASSES_CAMERA_ERROR`, `GLASSES_AUDIO_ERROR`, `INTERNAL_ERROR`, `UNKNOWN` — each carries an `int code` and `String message` |

**New callback interfaces:**

- `ISessionLifecycleCbk` — `onSessionStarted()`, `onSessionPaused(PausedReason)`, `onSessionResumed()`, `onSessionTerminating(TerminatingReason, Long)`, `onSessionClosed(CloseReason)`, `onConnectResult(Boolean, SessionErrorCode)`
- `IGlassesEventListener` — `onGlassesAppResumed()`, `onGlassesAppPaused()`, `onWearingStatusChanged(Boolean)`, `onDeviceInfoChanged(GlassesInfo)`, `onScreenOff()`, `onScreenOn()`, `onLauncherResumed()`, `onAiWake()`, `onAiInterruptChanged(Boolean)`
- `IAudioCallback` — `onAudioReceived(ByteArray)`, `onAudioError(Int, String)`, `onAudioStreamStateChanged(Boolean)`
- `IImageCallback` — `onImageReceived(ByteArray)`, `onImageError(SessionErrorCode, Int, String)`
- `ICustomCmdSessionCallback` — `onCustomCmdResult(String, ByteArray)`

**Native libraries newly bundled in the AAR's `jni/` folder (armeabi-v7a + arm64-v8a):** `libflora-cli.so`, `libmutils.so`, `libcaps.so`, `libcxr-sock-proto-jni.so`, `libcxr-bridge-jni.so`. These names match the native layer already documented for CXR-M (see [CXR-M release notes](../cxr-m/release-notes.md)) — i.e. v1.1.0 experimentally vendored CXR-M's Flora/Caps/socket-protocol native stack directly into client-l. As noted above, this bundling was reverted in v1.1.1.

**New AAR-level dependency:** `kotlinx-coroutines-android:1.6.4` (runtime scope) — first coroutines dependency in client-l's history. The `cxr-service-bridge` compile dependency was dropped in this version (restored in v1.1.1, see above).

**`com.rokid.sprite.aiapp.externalapp.*` (AIDL layer):** no public-signature changes; internal implementation classes were recompiled alongside the new session package, and a parallel internal copy of `AuthResult`/`GlassPermission` (`com.rokid.sprite.aiapp.externalapp.auth.*`) and a new `AuthorizationHelper` class appeared — consistent with the `session.CxrSessionManager` authorization flow being layered on top of the existing AIDL-based authorization plumbing rather than replacing it.

### v1.0.4 — published 2026-06-18

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

### v1.0.3 — published 2026-06-02

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

### v1.0.2 — published 2026-05-20

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

### v1.0.1 — 2026-05-07

1. Initial SDK release.
2. Support for obtaining authorization from the Rokid AI app.
3. Support for creating on-device custom View scenes.
4. Support for creating on-device custom app scenes.
5. Support for accessing on-device audio.
6. Support for capturing photos through the glasses.
7. Support for custom-command exchange with on-device custom apps.
