# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). v1.0.4 entry sourced from binary diff of Maven AARs (2026-06-25); no official changelog has been published for that release. v1.1.0 / v1.1.1 / v1.1.2 entries sourced from a binary diff of Maven AARs (downloaded 2026-09-04 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`); `developerdoc.rokid.com/sdk` and `open.rokid.com/sdk` are React SPAs that render no server-side changelog content and could not be checked for an official write-up of these releases as of 2026-09-04._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.2 — published 2026-08-28

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.1` and `client-l:1.1.2` AARs (downloaded 2026-09-04). AAR size: 171,307 bytes vs 171,369 bytes for v1.1.1 (−0.04 %).

Byte-identical class inventory to v1.1.1 (same 142 `.class` files, same package layout) and an identical POM (same dependency versions). No API-surface change detected. Likely an internal bugfix / resource-only patch release.

## v1.1.1 — published 2026-08-14

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.0` and `client-l:1.1.1` AARs (downloaded 2026-09-04). AAR size: 171,369 bytes vs 1,286,574 bytes for v1.1.0 (−86.7 %; see packaging change below).

**Theme: packaging cleanup after the v1.1.0 session-API drop.** No changes to the classic `com.rokid.cxr.link` (`CXRLink`) API surface or to the new `com.rokid.cxr.session` API introduced in v1.1.0.

**Packaging changes:**

- **Native libraries un-bundled.** v1.1.0 shipped `jni/{arm64-v8a,armeabi-v7a}/lib{caps,cxr-bridge-jni,cxr-sock-proto-jni,flora-cli,mutils}.so` directly inside the AAR. v1.1.1 removes the `jni/` directory entirely — natives are once again pulled in transitively via the `cxr-service-bridge` dependency, which is re-added to the POM (see below). This explains most of the AAR size drop.
- **`cxr-service-bridge` dependency restored.** Re-added to the POM at a newer snapshot build (`1.0-20260715.121510-107`, vs `1.0-20260522.063600-105` used through v1.0.4). It had been dropped from the POM in v1.1.0, whose AAR instead embedded the bridge classes (`com.rokid.cxr.CXRServiceBridge`, `CXRSocketProtocol`, `Caps`, `RLog`, `BuildConfig`) directly — those embedded copies are removed in v1.1.1.
- **ProGuard rules added.** New `proguard.txt` at the AAR root (first appeared in v1.1.0, retained here).
- **AIDL stub inner classes obfuscated.** The `$Stub$a` proxy inner classes for the eight `com.rokid.sprite.aiapp.externalapp.I*Callback` / `IMediaStreamService` AIDL interfaces are renamed to single-letter classes `a.class`–`h.class` (ProGuard name-shrinking artifact, not an API change).
- **Manifest package attribute changed** from `com.rokid.cxr.client.extend` to `com.rokid.cxr.link` (cosmetic; matches the `BuildConfig` class's actual package).

**Dependency changes vs v1.1.0:**

| Dependency | v1.1.0 | v1.1.1 |
|------------|--------|--------|
| `cxr-service-bridge` | *(not declared; embedded in AAR)* | `1.0-20260715.121510-107` |
| `kotlin-stdlib` | `1.6.0` | `1.9.0` |
| `gson` | `2.10.1` | `2.10.1` |
| `kotlinx-coroutines-android` | `1.6.4` | `1.9.0` |

## v1.1.0 — published 2026-07-02

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.4` and `client-l:1.1.0` AARs (downloaded 2026-09-04). AAR size: 1,286,574 bytes vs 70,543 bytes for v1.0.4 (+1724%; driven mostly by bundled native libraries, see below). Neither `developerdoc.rokid.com/sdk` nor `open.rokid.com/sdk` rendered a changelog for this release as of 2026-09-04 (both are client-rendered SPAs; no server-side content to scrape from this environment).

**Theme: new coroutine/StateFlow-based session API, additive alongside the existing `CXRLink` API.**

This release adds an entirely new `com.rokid.cxr.session` package (~94 new classes) implementing a higher-level, Kotlin-coroutine-friendly session facade. It is **additive**: every class in the existing `com.rokid.cxr.link` package (`CXRLink`, `ICXRLinkCbk`, `CxrDefs`, `GlassInfo`, etc., documented in [api-reference.md](api-reference.md)) is still present, byte-identical in the class list, in v1.1.0. Binary evidence (`CapabilityBroker` holds a `com.rokid.sprite.aiapp.externalapp.example.ExternalAppClient` field) confirms the new session API is a facade built on top of the same `ExternalAppClient` AIDL transport that `CXRLink` uses — it does not replace the transport layer described in the [architectural mental model](../CLAUDE.md).

See [api-reference.md § Session API (v1.1.0+)](api-reference.md#session-api-v110) for the reconstructed public surface (`CxrSessionManager`, `CxrSession`, callback interfaces, and supporting data/enum types).

**Packaging changes:**

- **Native libraries bundled directly in the AAR** for the first time: `jni/arm64-v8a/` and `jni/armeabi-v7a/`, each containing `libcaps.so`, `libcxr-bridge-jni.so`, `libcxr-sock-proto-jni.so`, `libflora-cli.so`, `libmutils.so`. (Reverted in v1.1.1 — see above.)
- **`proguard.txt` added** at the AAR root (first appearance).
- **`cxr-service-bridge` dropped from the POM.** Its classes (`CXRServiceBridge`, `CXRSocketProtocol`, `Caps`, `RLog`) are instead embedded directly in `classes.jar` under `com.rokid.cxr.*` in this release. (Reverted in v1.1.1.)
- **`kotlinx-coroutines-android:1.6.4` added** as a new runtime dependency — required by the `StateFlow`-based `CxrSession.getStateFlow()` API.

**Dependency changes vs v1.0.4:**

| Dependency | v1.0.4 | v1.1.0 |
|------------|--------|--------|
| `cxr-service-bridge` | `1.0-20260522.063600-105` | *(not declared; embedded in AAR)* |
| `kotlin-stdlib` | `1.6.0` | `1.6.0` |
| `gson` | `2.10.1` | `2.10.1` |
| `kotlinx-coroutines-android` | *(none)* | `1.6.4` |

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
