# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4 entry sourced from binary diff of Maven AARs (2026-06-25); no official changelog has been published for this release yet. v1.1.0/v1.1.1 entries sourced from `javap` bytecode inspection of Maven AARs (2026-08-16); the official changelog at `developerdoc.rokid.com/sdk` still shows v1.0.4 as of 2026-08-16 — Maven leads the documented portal by two minor releases._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.1 — published 2026-08-15

> **Provisional — not an official Rokid changelog.** Reconstructed from `javap` bytecode inspection of `client-l:1.1.1` (downloaded 2026-08-16 from Maven), cross-referenced against the AAR's own `proguard.txt` consumer keep-rules and diffed against `client-l:1.1.0`. No official changelog published as of 2026-08-16.

**Theme: rollback of the v1.1.0 native-bridge experiment; retains the new coroutine-based `CxrSession` API.**

- AAR size: 171,369 bytes (vs. 1,286,574 bytes for v1.1.0, −86.7%; vs. 70,543 bytes for v1.0.4, +142.9%).
- **Removes the native JNI libraries bundled in v1.1.0** — `libcaps.so`, `libcxr-bridge-jni.so`, `libcxr-sock-proto-jni.so`, `libflora-cli.so`, `libmutils.so` for both `arm64-v8a` and `armeabi-v7a` — along with the copies of `com.rokid.cxr.Caps` / `CXRServiceBridge` / `CXRSocketProtocol` / `RLog` that v1.1.0 had inlined directly into `client-l`'s own `classes.jar`. These are supplied transitively via the `cxr-service-bridge` dependency again, as in the v1.0.x line.
- **Dependency changes vs. v1.0.4**: `cxr-service-bridge` `1.0-20260522.063600-105` → `1.0-20260715.121510-107`; `kotlin-stdlib` `1.6.0` → `1.9.0`; `gson:2.10.1` unchanged. **New dependency**: `org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0`, backing the new `StateFlow`-based session API.
- **AndroidManifest `package` attribute** changed from `com.rokid.cxr.client.extend` to `com.rokid.cxr.link`. `minSdkVersion` unchanged (28 for the AAR manifest declaration; official docs still specify 31+ for the documented v1.0.4 API surface).
- **Retains the entire `com.rokid.cxr.session` package** introduced in v1.1.0 (see below), unchanged at the class level between 1.1.0 and 1.1.1.
- **The pre-existing `com.rokid.cxr.link.*` package is byte-identical to v1.0.4** — `CXRLink`, `ICXRLinkCbk`, `CxrDefs`, and all other v1.0.x public types are unaffected. No breaking changes to the documented API.

See [api-reference.md](api-reference.md#cxrsession-api-v110-provisional) for the full reconstructed `CxrSession` API surface.

## v1.1.0 — published 2026-07-02

> **Provisional — not an official Rokid changelog.** Reconstructed from `javap` bytecode inspection of `client-l:1.1.0` (downloaded 2026-08-16 from Maven), diffed against `client-l:1.0.4`. No official changelog published; the SDK selection page at `developerdoc.rokid.com/sdk` (mirrored at `open.rokid.com/sdk`) still lists CXR-L at 1.0.4 as of 2026-08-16.

**Theme: new coroutine/`Flow`-based `CxrSession` API layered on top of the existing `CXRLink`/AIDL layer.**

- AAR size: 1,286,574 bytes vs. 70,543 bytes for v1.0.4 (+1,724%) — driven almost entirely by newly bundled native libraries under `jni/arm64-v8a/` and `jni/armeabi-v7a/` (`libcaps.so`, `libcxr-bridge-jni.so`, `libcxr-sock-proto-jni.so`, `libflora-cli.so`, `libmutils.so`) plus inlined copies of `com.rokid.cxr.Caps`, `CXRServiceBridge`, `CXRSocketProtocol`, and `RLog` — classes normally supplied transitively via the `cxr-service-bridge` dependency. This suggests `client-l` briefly linked the native socket-protocol / Flora IPC layer directly in this build, mirroring the architecture CXR-M uses (see the repository's architectural notes in `CLAUDE.md`); v1.1.1 reverted this change (see above). **v1.1.0 is superseded by v1.1.1 — prefer 1.1.1 or later for new integrations.**
- **New package `com.rokid.cxr.session`** introducing a higher-level session API: `CxrSessionManager` (authorization + session creation), `CxrSession` (connect/close, audio/photo/custom-command, `StateFlow<SessionState>`), `SessionConfig`, and a family of supporting enums, result types, and callback interfaces. This is purely additive — the existing `com.rokid.cxr.link.CXRLink` entry point is untouched (byte-identical to v1.0.4).

See [api-reference.md](api-reference.md#cxrsession-api-v110-provisional) for the full reconstructed API surface.

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
