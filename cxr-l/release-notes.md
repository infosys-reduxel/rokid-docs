# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4 entry sourced from binary diff of Maven AARs (2026-06-25); no official changelog has been published for this release yet. v1.1.0–v1.1.2 entries sourced from binary diff of Maven AARs plus `javap` decompilation (2026-09-28) — `developerdoc.rokid.com/sdk` confirms CXR-L is at "最新版本 1.1.2" (updated 2026.09.08) but its collapsed changelog panel did not yield readable text via Firecrawl on this pass, so no official changelog transcript exists for these three releases yet._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.2 — uploaded 2026-08-28

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.1` and `client-l:1.1.2` AARs (downloaded 2026-09-28 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). AAR size: 171,307 bytes vs 171,369 bytes for v1.1.1 (−0.04%). Class list is byte-identical to v1.1.1 (142 classes in both, no additions/removals). No observable public-API changes — treat as a maintenance/patch release.

## v1.1.1 — uploaded 2026-08-14

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.0` and `client-l:1.1.1` AARs (downloaded 2026-09-28). AAR size: 171,369 bytes vs 1,286,574 bytes for v1.1.0 (−86.7%).

**Theme: externalize the bundled service-bridge classes again; dependency bumps.**

- The `com.rokid.cxr.CXRServiceBridge`, `com.rokid.cxr.CXRSocketProtocol`, and `com.rokid.cxr.Caps` classes that v1.1.0 had bundled directly into the AAR (see below) are gone from the class list — v1.1.1 goes back to consuming `cxr-service-bridge` as a regular Maven dependency (pinned to `1.0-20260715.121510-107` per the AAR's `pom.xml`, a pre-release snapshot of `cxr-service-bridge` — note this predates the `cxr-service-bridge:1.1`–`1.4` public releases documented in [cxr-s/release-notes.md](../cxr-s/release-notes.md)).
- `com.rokid.cxr.BuildConfig` was renamed to `com.rokid.cxr.link.BuildConfig`.
- The AIDL stub proxy inner classes for `IAiEventCallback`, `IAudioStreamCallback`, `ICustomCmdCallback`, `ICustomViewCallback`, `IDeviceStatusCallback`, `IGlassAppCallback`, `IImageStreamCallback`, and `IMediaStreamService` were re-obfuscated (their `$Stub$a` inner class was renamed to single letters `a`–`h`). No functional change implied — this is a build-tool artifact of R8/ProGuard renaming, not a public-API change.

**Dependency changes vs v1.1.0:**

| Dependency | v1.1.0 | v1.1.1 |
|------------|--------|--------|
| `kotlin-stdlib` | `1.6.0` | `1.9.0` |
| `kotlinx-coroutines-android` | `1.6.4` | `1.9.0` |
| `gson` | `2.10.1` | `2.10.1` (unchanged) |
| `cxr-service-bridge` | bundled inline (no separate dependency) | `1.0-20260715.121510-107` |

## v1.1.0 — uploaded 2026-07-02

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.4` and `client-l:1.1.0` AARs, plus `javap` decompilation of the new `com.rokid.cxr.session.*` classes (downloaded/decompiled 2026-09-28). AAR size: 1,286,574 bytes vs 70,543 bytes for v1.0.4 (+1,724%) — largely explained by `cxr-service-bridge`'s classes (`CXRServiceBridge`, `CXRSocketProtocol`, `Caps`, `RLog`) being bundled directly into this AAR rather than pulled in as a separate Maven dependency (v1.1.1 reverts this — see above).

**Theme: a new session-based programming model (`com.rokid.cxr.session`) alongside the existing `CXRLink`/`ExternalAppClient` API.** This does not replace `CXRLink` in this release (both class hierarchies coexist in the AAR); treat it as an SDK-side preview of a session API until Rokid's official docs describe its relationship to `CXRLink`.

**New interfaces:**

- `com.rokid.cxr.session.CxrSession` — the session handle. Key methods (via `javap`, signatures verbatim):

  | Method | Description |
  |--------|-------------|
  | `getState(): SessionState` / `getStateFlow(): StateFlow<SessionState>` | Current state, observable via Kotlin `StateFlow`. |
  | `getConfig(): SessionConfig` | The config the session was created with. |
  | `connect(String)` | Connect using the given identifier (e.g. glasses address/token). |
  | `close()` | Tear down the session. |
  | `startAudioStream()` / `stopAudioStream(): SessionResult<Unit>` | Start/stop the audio stream. |
  | `customViewUpdate(String): SessionResult<Unit>` | Push updated custom-view JSON. |
  | `takePhoto(Int, Int, Int): SessionResult<Unit>` | Capture a photo. The three `Int` parameters' meaning is not recoverable from bytecode alone — Rokid has not published an accompanying guide. |
  | `sendCustomCmd(String, Caps, ByteArray): SessionResult<Unit>` | Send a structured + binary custom command. |
  | `setGlassBrightness(Int)` / `setGlassVolume(Int): SessionResult<Unit>` | Device controls, same shape as the `ICXRSessionCbk`-era methods added in v1.0.4. |
  | `queryGlassesInfo(): SessionResult<GlassesInfo>` | Query a `GlassesInfo` snapshot. |
  | `add`/`removeLifecycleCallback(ISessionLifecycleCbk)`, `add`/`removeAudioCallback(IAudioCallback)`, `add`/`removeImageCallback(IImageCallback)`, `add`/`removeCustomCmdCallback(ICustomCmdSessionCallback)`, `add`/`removeGlassesEventListener(IGlassesEventListener)` | Multi-listener registration — a departure from the single-callback pattern used by `ICXRLinkCbk`/`ICXRSessionCbk`. |

- `com.rokid.cxr.session.CxrSessionManager` — the session factory/singleton surface.

  | Method | Description |
  |--------|-------------|
  | `create(SessionConfig): CxrSession` | Create a new session. |
  | `getSession(): CxrSession` | Get the current session. |
  | `requestAuthorization(Activity, List<GlassPermission>, (AuthResult) -> Unit)` | Request one or more `GlassPermission`s, async via callback. |
  | `parseAuthorizationResult(Int, Intent): AuthResult` | Parse an `onActivityResult`-style callback into an `AuthResult`. |
  | `isRokidAppInstalled(Context): Boolean` | Presence check for the Rokid AI app. |
  | `checkRokidAppCompatibility(Context): RokidAppStatus` | Version compatibility check, returns a `RokidAppStatus` sealed result (`Compatible`, `NotInstalled`, `VersionTooLow`). |
  | `isGlassesBtConnected(): Boolean` | Bluetooth connection check. |

- Callback interfaces: `ISessionLifecycleCbk` (`onSessionStarted`, `onSessionPaused(PausedReason)`, `onSessionResumed`, `onSessionTerminating(TerminatingReason, Long)`, `onSessionClosed(CloseReason)`, `onConnectResult(Boolean, SessionErrorCode)`), `IAudioCallback` (`onAudioReceived(ByteArray)`, `onAudioError(Int, String)`, `onAudioStreamStateChanged(Boolean)`), `IImageCallback` (`onImageReceived(ByteArray)`, `onImageError(SessionErrorCode, Int, String)`), `ICustomCmdSessionCallback` (`onCustomCmdResult(String, ByteArray)`), `IGlassesEventListener` (`onGlassesAppResumed/Paused`, `onWearingStatusChanged(Boolean)`, `onDeviceInfoChanged(GlassesInfo)`, `onScreenOff/On`, `onLauncherResumed`, `onAiWake`, `onAiInterruptChanged(Boolean)`).

**New data/enum types:**

| Type | Values / fields |
|------|------------------|
| `SessionType` | `CUSTOM_VIEW`, `CUSTOM_APP` |
| `SessionState` | `Idle`, `Starting`, `Started`, `Paused`, `Terminating` |
| `SessionErrorCode` | 28 values incl. `OK`, `NOT_AUTHENTICATED`, `TOKEN_EXPIRED`, `ROKID_APP_NOT_INSTALLED`, `ROKID_APP_VERSION_LOW`, `LINK_NOT_READY`, `BT_NOT_CONNECTED`, `LINK_LOST`, `LINK_TIMEOUT`, `SESSION_NOT_STARTED`, `SESSION_TERMINATING`, `SESSION_ALREADY_EXISTS`, `CONNECT_FAILED`, `SCENE_OPEN_FAILED`, `RESOURCE_PREPARE_FAILED`, `SESSION_PAUSED`, `DATA_NOT_READY`, `INVALID_ARGUMENT`, `OPERATION_IN_PROGRESS`, `OPERATION_CANCELLED`, `GLASSES_SIGNAL_TIMEOUT`, `GLASSES_APP_NOT_FOUND`, `GLASSES_APP_INSTALL_FAILED`, `GLASSES_MEMORY_PRESSURE`, `GLASSES_CAMERA_ERROR`, `GLASSES_AUDIO_ERROR`, `INTERNAL_ERROR`, `UNKNOWN` — each with an `Int` `code` and `String` `message` |
| `GlassPermission` | `CAMERA`, `MICROPHONE`, `MEDIA`, `DEVICE_MANAGE` |
| `CloseReason` | `USER_CLOSED`, `CONNECT_FAILED`, `GLASSES_EXIT`, `LINK_LOST`, `CHAIN_TORN`, `FATAL_ERROR` |
| `TerminatingReason` | `GLASSES_APP_EXIT`, `GLASSES_APP_CRASH`, `GLASSES_RESOURCE_RECLAIMED`, `LINK_CHAIN_TORN`, `OTHER` |
| `PausedReason` | `AI_ASSIST`, `BT_DISCONNECTED` |
| `AiInterceptMode` | `ALLOW_WITH_PAUSE`, `BLOCK_AI` |
| `GlassesInfo` (data class) | `osVersion: String`, `batteryPercent: Int`, `freeMemoryMb: Long`, `isCharging: Boolean`, `displayWidth: Int`, `displayHeight: Int`, `screenOn: Boolean` |
| `AuthResult` (data class) | `isSuccess: Boolean`, `token: String?`, `errorCode: SessionErrorCode`, `message: String?` |
| `SessionConfig` (data class) | `sessionType: SessionType`, `glassesPackageName: String`, `aiInterceptMode: AiInterceptMode`, `terminatingGracePeriodMs: Long`, `timeouts: SessionTimeouts`, `viewData: String?`, `viewIconData: String?`, `glassesActivityName: String?`, `glassesApkPath: String?` |
| `SessionTimeouts` (data class) | `connectTimeoutMs: Long`, `takePhotoTimeoutMs: Long`, `customCmdTimeoutMs: Long` (has a no-arg constructor, i.e. ships defaults) |
| `SessionResult<T>` (data class) | `code: SessionErrorCode`, `data: T?`, `message: String?`, plus a computed `isSuccess: Boolean` |

Rokid has not published how `CxrSession`/`CxrSessionManager` relate to `CXRLink`/`ExternalAppClient` going forward (parallel API vs. eventual replacement); this note will be revisited once an official changelog or SDK guide addresses it.

**Dependency changes vs v1.0.4:** `kotlin-stdlib` unchanged at `1.6.0`; adds `kotlinx-coroutines-android:1.6.4` (new, backs the `StateFlow` surface on `CxrSession`); `cxr-service-bridge` is bundled inline rather than referenced as a separate dependency in this release only (see v1.1.1 above for the revert).

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
