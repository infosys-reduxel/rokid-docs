# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4 entry sourced from binary diff of Maven AARs (2026-06-25); no official changelog has been published for this release yet. v1.1.0/v1.1.1/v1.1.2 entries sourced from binary diff of Maven AARs (fetched 2026-08-31) — as of this fetch the developer portal changelog (`developerdoc.rokid.com/sdk`) still lists **1.0.4** as the CXR-L "最新版本" (latest version); no official changelog exists yet for the 1.1.x line._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.2 — uploaded 2026-08-28 (provisional, binary diff)

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.1` and `client-l:1.1.2` AARs (downloaded 2026-08-31 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). The developer portal has not published a changelog entry for this release.

No class-level or `AndroidManifest.xml` differences from v1.1.1 — the `classes.jar` is byte-for-byte identical. The only change is a new `coreLibraryDesugaringEnabled=false` line in the AAR's `META-INF/com/android/build/gradle/aar-metadata.properties`, a build-tooling flag with no effect on the public API. Treat this as a housekeeping release.

## v1.1.1 — uploaded 2026-08-14 (provisional, binary diff)

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.4` and `client-l:1.1.1` AARs (downloaded 2026-08-31). No official changelog has been published for the 1.1.x line as of this fetch. See the v1.1.0 note below — 1.1.1 supersedes the anomalous 1.1.0 build.

**Theme: a new coroutine-backed `com.rokid.cxr.session` API, layered alongside the existing `CXRLink`/`ExternalAppClient` callback API (which is retained unchanged).**

**AAR package identifier changed:** the `<manifest package="...">` attribute in the AAR moved from `com.rokid.cxr.client.extend` to `com.rokid.cxr.link`. This only affects the AAR's own generated `R` class namespace (there are no meaningful resources besides `res/xml/network_security_config.xml`); it is not expected to break consumers who reference the SDK only through its Kotlin/Java classes.

**New package `com.rokid.cxr.session`** (all classes below are new; extracted directly via `javap` against the 1.1.1 AAR, not guessed):

- `CxrSessionManager` — entry point (singleton via `Companion`) for creating and authorizing sessions.

  | Method | Signature | Description |
  |--------|-----------|--------------|
  | `create` | `(config: SessionConfig): CxrSession` | Creates a new `CxrSession` from a `SessionConfig`. |
  | `getSession` | `(): CxrSession` | Returns the current session instance. |
  | `requestAuthorization` | `(activity: Activity, permissions: List<GlassPermission>, callback: (AuthResult) -> Unit)` | Requests one or more `GlassPermission`s from the Rokid AI app; result delivered async via the lambda. |
  | `parseAuthorizationResult` | `(resultCode: Int, data: Intent): AuthResult` | Parses the `onActivityResult` payload from the authorization flow. |
  | `isRokidAppInstalled` | `(context: Context): Boolean` | Checks whether a Rokid AI app package is present. |
  | `checkRokidAppCompatibility` | `(context: Context): RokidAppStatus` | Returns a `RokidAppStatus` sealed result (see below). |
  | `isGlassesBtConnected` | `(): Boolean` | Checks the current Bluetooth connection state to the glasses. |

- `CxrSession` — the session object itself, replacing/augmenting ad hoc `CXRLink` calls with a single stateful handle.

  | Method | Signature | Description |
  |--------|-----------|--------------|
  | `getState` / `getStateFlow` | `(): SessionState` / `StateFlow<SessionState>` | Current state, or a Kotlin coroutines `StateFlow` for observing state changes reactively. |
  | `getConfig` | `(): SessionConfig` | The `SessionConfig` this session was created with. |
  | `connect` | `(token: String)` | Connects using an auth token obtained via `CxrSessionManager.requestAuthorization`. |
  | `close` | `()` | Closes the session. |
  | `startAudioStream` / `stopAudioStream` | `(): SessionResult<Unit>` | Same capability as `CXRLink`, wrapped in a `SessionResult`. |
  | `customViewUpdate` | `(data: String): SessionResult<Unit>` | Update the custom view JSON. |
  | `takePhoto` | `(width: Int, height: Int, quality: Int): SessionResult<Unit>` | Same three-argument shape as `CXRLink.takePhoto`. |
  | `sendCustomCmd` | `(cmd: String, caps: Caps, data: ByteArray): SessionResult<Unit>` | Custom-command send, now taking a `Caps` object and raw bytes together. |
  | `setGlassBrightness` / `setGlassVolume` | `(level: Int): SessionResult<Unit>` | Same capability as `CXRLink`, wrapped in a `SessionResult`. |
  | `queryGlassesInfo` | `(): SessionResult<GlassesInfo>` | New: a snapshot query returning `GlassesInfo` (OS version, battery %, free memory, charging, display size, screen-on state). |
  | `add`/`removeLifecycleCallback`, `add`/`removeAudioCallback`, `add`/`removeImageCallback`, `add`/`removeCustomCmdCallback`, `add`/`removeGlassesEventListener` | — | Multi-subscriber callback registration (unlike `CXRLink`'s single-callback `setCXR*Cbk` setters, these support add/remove of multiple listeners). |

- Callback interfaces: `ISessionLifecycleCbk` (`onSessionStarted`, `onSessionPaused(PausedReason)`, `onSessionResumed`, `onSessionTerminating(TerminatingReason, graceMs: Long)`, `onSessionClosed(CloseReason)`, `onConnectResult(Boolean, SessionErrorCode)`), `IGlassesEventListener` (app resume/pause, wearing status, device info, screen on/off, launcher resume, AI wake/interrupt), `IAudioCallback`, `IImageCallback`, `ICustomCmdSessionCallback`.

- Enums: `SessionState` (`Idle`, `Starting`, `Started`, `Paused`, `Terminating`), `SessionType` (`CUSTOM_VIEW`, `CUSTOM_APP`), `GlassPermission` (`CAMERA`, `MICROPHONE`, `MEDIA`, `DEVICE_MANAGE`), `AiInterceptMode` (`ALLOW_WITH_PAUSE`, `BLOCK_AI`), `CloseReason` (`USER_CLOSED`, `CONNECT_FAILED`, `GLASSES_EXIT`, `LINK_LOST`, `CHAIN_TORN`, `FATAL_ERROR`), `PausedReason` (`AI_ASSIST`, `BT_DISCONNECTED`), `TerminatingReason` (`GLASSES_APP_EXIT`, `GLASSES_APP_CRASH`, `GLASSES_RESOURCE_RECLAIMED`, `LINK_CHAIN_TORN`, `OTHER`), and a 28-value `SessionErrorCode` (each carrying an `Int` code and a `String` message — e.g. `OK`, `NOT_AUTHENTICATED`, `TOKEN_EXPIRED`, `ROKID_APP_NOT_INSTALLED`, `LINK_LOST`, `GLASSES_CAMERA_ERROR`, `INTERNAL_ERROR`, `UNKNOWN`, …).

- Data classes: `SessionConfig` (session type, glasses package name, `AiInterceptMode`, terminating grace period ms, `SessionTimeouts`, view data/icon JSON, glasses activity name/APK path), `SessionTimeouts` (connect/take-photo/custom-cmd timeout ms), `GlassesInfo` (OS version, battery %, free memory MB, charging, display width/height, screen-on), `AuthResult` (success flag, token, error code, message), `SessionResult<T>` (error code, typed data, message — a generic `Result`-style wrapper).

- Sealed class `RokidAppStatus` with three variants: `Compatible(version: String)`, `NotInstalled(minimumVersion: String, downloadUrl: String)`, `VersionTooLow(installedVersion: String, minimumVersion: String, downloadUrl: String)`.

**The existing callback-based API is untouched.** `ICXRLinkCbk`, `ICXRSessionCbk`, `ICustomViewCbk`, `IAudioStreamCbk`, `IImageStreamCbk`, `ICustomCmdCbk`, `IGlassAppCbk` (all under `com.rokid.cxr.link.callbacks`) are all still present, byte-identical to v1.0.4. The new `com.rokid.cxr.session` package appears to be an additive, more idiomatic-Kotlin (coroutine/`StateFlow`-based) alternative entry point rather than a replacement.

**Note on the withdrawn v1.1.0 build:** see below — 1.1.1 removes internal classes and native libraries that 1.1.0 had accidentally bundled.

## v1.1.0 — uploaded 2026-07-02 (provisional, binary diff — likely a packaging defect)

> **Provisional — not an official Rokid changelog.** This version was live on Maven for roughly six weeks (2026-07-02 to 2026-08-14) before being superseded by v1.1.1. Treat its contents as unreliable for integration purposes.

The v1.1.0 AAR (1,286,574 bytes vs 70,543 for v1.0.4 — an 18× size increase) bundles content that does not belong in a `client-l` artifact:

- All of the same new `com.rokid.cxr.session` API described under v1.1.1 above (same class list).
- **Unexpectedly duplicated internal classes** normally scoped to CXR-M/CXR-S: `com.rokid.cxr.CXRServiceBridge`, `com.rokid.cxr.CXRSocketProtocol`, `com.rokid.cxr.Caps` (and nested `Caps.Value`/`Caps.Binary`/`Caps.IncorrectTypeException`), and `com.rokid.cxr.RLog` — none of these are present in v1.0.4 or v1.1.1.
- **Bundled native libraries** under `jni/arm64-v8a/` and `jni/armeabi-v7a/`: `libcaps.so`, `libcxr-bridge-jni.so`, `libcxr-sock-proto-jni.so`, `libflora-cli.so`, `libmutils.so`. These correspond to the Caps binary-serialization layer and the Flora message bus used elsewhere in the CXR SDK family (see [`cxr-s/data-structure.md`](../cxr-s/data-structure.md)) — not previously part of `client-l`.
- No `proguard.txt` was shipped with the JNI-Stub compatibility classes obfuscated in v1.1.1 (`com.rokid.sprite.aiapp.externalapp.a`–`h`); v1.1.0 ships the un-obfuscated originals (`IAiEventCallback$Stub$a`, `IAudioStreamCallback$Stub$a`, etc.), consistent with 1.1.0 being an un-shrunk/debug-style build.

This strongly resembles an internal or debug build artifact that was mistakenly published to the public Maven repository, rather than an intentional public release. v1.1.1 removes all of the above extraneous content, obfuscates the AIDL compatibility shim classes, and ships a `proguard.txt` consumer-rules file — consistent with a corrected, intentionally-public release. **Do not depend on `client-l:1.1.0`** — pin to `1.1.1` or later.

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
