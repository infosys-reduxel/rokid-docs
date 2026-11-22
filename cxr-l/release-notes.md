# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese; official Rokid changelog, "更新内容" panel of the CXR-L card) and the "版本历史" (Version History) chapter of the CXR-L v1.1.2 doc set at <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/cn/fa92e9303bc848e6b4a20c062f2c792b.html?documentId=2b189da5908f4e48b2d1b749a38888a5> (Chinese). Both fetched 2026-11-22. v1.0.1–v1.0.3 entries were fetched 2026-06-11. The v1.0.4 entry combines the official changelog (published after the binary-diff analysis below was written) with the earlier binary diff of Maven AARs (2026-06-25)._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## Current versions

> Source: "版本历史" (Version History) chapter of the CXR-L v1.1.2 doc set, fetched 2026-11-22.

The CXR-L SDK ships as two release lines, Android and iOS. Their version numbers evolve independently and are **not aligned** with each other.

| Platform | Dependency coordinate | Current version | Baseline of the upstream doc set |
| --- | --- | --- | --- |
| Android | `com.rokid.cxr:client-l` | `1.1.2` (recommended) | `1.1.0` |
| iOS | `RGCxrClient` (CocoaPods) | `1.1.1.1` | `1.1.1.1` |

> **Version mismatch note (upstream):** Android and iOS version numbers do not correspond to each other. Android `1.1.2` and iOS `1.1.1.1` are each the latest release on their own line; the magnitude of the number says nothing about which is newer. Follow each platform's own version history for feature evolution.

```kotlin
implementation("com.rokid.cxr:client-l:1.1.2")
```

iOS: `pod 'RGCxrClient', '1.1.1.1'` (the Rokid private specs source is also required; upstream covers this in its "iOS: SDK import" chapter, which is not yet translated in this repo).

Maven cross-check (`https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/maven-metadata.xml`, fetched 2026-11-22): `<release>` is `1.1.2`, `<latest>` is `1.2.X-SNAPSHOT`, `lastUpdated` is `20261105064636` (the scrape returned the metadata with its XML tags stripped, so the latest/release assignment is read from the standard element order). The 1.x release versions listed are 1.0.0, 1.0.1, 1.0.2, 1.0.3, 1.0.4, 1.1.0, 1.1.1, and 1.1.2. No changelog exists for the 1.2.X snapshot line.

> **The v1.1.x line is a re-architecture, not an incremental update.** The entry point, session model, and callback interfaces all changed (upstream now documents `CxrSessionManager`, `SessionConfig`, and `ISessionLifecycleCbk`). [api-reference.md](api-reference.md) and [intro.md](intro.md) in this repo still describe the v1.0.x API (`CXRLink`) and have not yet been refreshed against the v1.1.2 doc set.

## v1.1.2 — updated 2026-09-08

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L card, fetched 2026-11-22) and the upstream "Version History" chapter.

**Official changelog (portal):**

1. Android `client-l` upgraded to 1.1.2; fixes a connection timeout (`LINK_TIMEOUT`) on the first `CUSTOM_VIEW` session.
2. Architecture rework: the mobile (companion) side no longer connects to the glasses directly. Everything is relayed through the Rokid AI App (authorization → link establishment → capability calls).
3. Session creation is decoupled from connection, and a five-state state machine is introduced (Idle / Starting / Started / Paused / Terminating).
4. Authorization is now scope-based (`GlassPermission`); blanket authorization is removed.
5. The `CUSTOM_APP` scene gains a `glassesApkPath` setting, and the on-device (glasses-side) APK is installed automatically on connect. The default AI intercept mode changes to foreground mode.
6. Audio is unified to PCM 16 kHz / mono / 16-bit; photo capture is unified to WebP encoding.
7. The sample project is restructured as `CxrlSample` (MVVM, single Activity) and is available as an OSS download; iOS is upgraded to `RGCxrClient` 1.1.1.1.

The portal lists the whole v1.1.x rework under the single V1.1.2 heading. The upstream "Version History" chapter splits it per Android release as follows.

**1.1.2**

- Fixes a `LINK_TIMEOUT` (connection timeout) that could occur the first time a session is created with the `CUSTOM_VIEW` scene.
- Projects on `1.1.0` / `1.1.1` are advised to upgrade to `1.1.2`. As a temporary workaround without upgrading, complete one connection with the `CUSTOM_APP` scene first and then switch to `CUSTOM_VIEW`.

**1.1.1 (relative to 1.1.0)**

- The `CUSTOM_APP` scene configuration gains a `glassesApkPath` field (path of the APK on the phone).
- `CUSTOM_VIEW` icon delivery moves onto the `viewData` data chain; the old icon API is removed.
- The default AI intercept mode changes from "background mode" to "foreground mode".

**1.1.0 (relative to v1.0.4)**

Upstream gives the release date of 1.1.0 as 2026-06-25. The portal changelog gives the same date for V1.0.4; this repo records both as published and does not reconcile them.

Changes to the mental model:

1. **Direct glasses connection removed.** The "app connects directly to the glasses" path is no longer used; everything is relayed through the Rokid AI App (authorization → link establishment → capability calls).
2. **Simpler session creation.** Session creation is decoupled from connection (create first, connect afterwards).
3. **Narrower authorization.** Authorization is scope-based (`GlassPermission`); blanket authorization is removed.
4. **Sample project restructured.** Capability pages talk to the SDK session directly instead of going through a relay layer; session instances are created and released with the page lifecycle.
5. **Explicit capability gating.** The differences in capability support between `CUSTOM_VIEW` and `CUSTOM_APP` are stated in each chapter.

Per-platform details (upstream table; cells upstream left blank are shown as "—"):

| Dimension | Android | iOS |
| --- | --- | --- |
| State machine | Five states: Idle / Starting / Started / Paused / Terminating | — |
| Authorization | `authenticate(scopes:completion:)` scope-based authorization | Same |
| `CUSTOM_VIEW` | `viewData` JSON data chain + icon delivery | Same |
| `CUSTOM_APP` | Automatic install of the on-device (glasses-side) APK (`glassesApkPath`) | Same |
| Audio | PCM 16 kHz / mono / 16-bit, single stream | Same |
| Photo capture | Returned WebP-encoded | Same |
| Buttons | Reported through the on-device (glasses-side) CXR-S (`rk_custom_key`) | Same |

### Migrating from v1.0.x to v1.1.0

- The mobile (companion) side no longer connects to the glasses directly: install and authorize the Rokid AI App first, and the app then relays through the SDK.
- `authenticated(token:)` on `authStatePublisher` carries both the token and `expiresAt`; handle expiry accordingly.
- App-management APIs (`openApp` / `closeApp` / `startApp` / `stopApp` / `uninstallApp` and similar) are no longer provided. Use "`glassesApkPath` preset + automatic install on connect" instead.
- Photo output changes from JPEG to WebP throughout; decode as WebP when reading and displaying.

### Known implementation flaws (confirmed by upstream)

- Audio capture is always PCM 16 kHz / mono / 16-bit (verified on a real device); setting the sample-rate parameter does not change the actual format.
- Photos are WebP-encoded (verified on a real device); the file-name extension may not match the encoding, so trust the decoded result.
- With Rokid AI App v1.9.0, the connection state may be falsely shown as connected; treat the SDK state callbacks as authoritative.
- For `LINK_LOST` recovery, the "re-authorize" path is the single recommended and most reliable approach.

### Requirements stated for v1.1.2

From the upstream v1.1.2 introduction page and the portal quick-start panel (both fetched 2026-11-22):

- Android: `minSdk 28`, `targetSdk 34`, Kotlin 1.9+, AGP 8.2+, Java 17. This reverses the `minSdk 31` requirement that the v1.0.3 documentation stated.
- Rokid AI App: do not hard-code a version string; use the return value of `checkRokidAppCompatibility`. Upstream states the current SDK check threshold is 1.11.0 and that the 1.9.0 string shown inside the SDK is a known implementation flaw.
- Samples: `CxrlSample` (Android, depends on `com.rokid.cxr:client-l:1.1.2`), `CXRClientDemo` (iOS), `CXRSWithCXRLSample` (on-device, integrates the CXR-S SDK).

## v1.0.4 — published 2026-06-18

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L card, fetched 2026-11-22), which lists V1.0.4 with an update date of 2026-06-25. The 2026-06-18 date in the heading is the Maven upload date recorded when this section was first written.

**Official changelog:**

1. Volume control added.
2. Brightness control added.

The upstream "Version History" chapter additionally notes for v1.0.4 that the on-device (glasses-side) sample project `CXRSWithCXRLSample` was first published with the documentation.

**Binary-diff detail (provisional).** The remainder of this section predates the official changelog. It was reconstructed from a binary diff of the `client-l:1.0.3` and `client-l:1.0.4` AARs (downloaded 2026-06-25 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`) and is not official Rokid documentation. The official changelog confirms the volume and brightness controls; it does not mention the session-lifecycle additions described below. AAR size: 70,543 bytes vs 65,494 bytes for v1.0.3 (+7.7 %).

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
