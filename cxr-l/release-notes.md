# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese; official Rokid changelog, CXR-L card, fetched 2026-11-13) and the "Version History" (版本历史) chapter of the CXR-L v1.1.2 documentation at <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/cn/fa92e9303bc848e6b4a20c062f2c792b.html?documentId=2b189da5908f4e48b2d1b749a38888a5> (Chinese, fetched 2026-11-13). The v1.0.4 technical detail is additionally sourced from a binary diff of Maven AARs (2026-06-25)._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## Current versions (as of 2026-11-13)

The CXR-L SDK ships as two independent release lines, Android and iOS. Their version numbers evolve separately and are **not aligned** with each other.

| Platform | Dependency coordinates | Current version | Upstream documentation baseline |
|----------|------------------------|-----------------|---------------------------------|
| Android | `com.rokid.cxr:client-l` | `1.1.2` (recommended) | `1.1.0` |
| iOS | `RGCxrClient` (CocoaPods) | `1.1.1.1` | `1.1.1.1` |

> **Version mismatch note (upstream).** Android and iOS version numbers do not correspond to each other — Android `1.1.2` and iOS `1.1.1.1` are each the latest release on their own line, and the relative size of the numbers says nothing about which is newer. Follow each platform's own version history for feature evolution.

```kotlin
implementation("com.rokid.cxr:client-l:1.1.2")
```

iOS: `pod 'RGCxrClient', '1.1.1.1'` (the Rokid private specs source is also required).

Maven cross-check (`maven-metadata.xml` for `com.rokid.cxr:client-l`, read 2026-11-13): `<release>` is `1.1.2`; the published versions after `1.0.4` are `1.1.0`, `1.1.1`, and `1.1.2`; `lastUpdated` is `20261105064636`.

> **The local [Introduction](intro.md) and [API Reference](api-reference.md) still describe the v1.0.x API** (`CXRLink` / `ExternalAppClient`). v1.1.0 restructured the SDK around `CxrSessionManager` and a five-state session model, and removed the app-management APIs — see the migration notes below. Those two pages have not yet been refreshed against the v1.1.2 documentation.

## v1.1.2 — portal changelog dated 2026-09-08

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L card, "V1.1.2", updated 2026.09.08) and the v1.1.2 documentation's Version History chapter, both fetched 2026-11-13.

**Official changelog (portal).** The portal publishes a single entry for the v1.1.x line under the V1.1.2 heading:

1. Android `client-l` upgraded to 1.1.2; fixes a connection timeout (`LINK_TIMEOUT`) on the first `CUSTOM_VIEW` session.
2. Architecture refactor: the mobile side no longer connects to the glasses directly; everything is relayed through the Rokid AI app (authorization → link establishment → capability calls).
3. Session creation and connection are decoupled, and a five-state state machine is introduced (Idle / Starting / Started / Paused / Terminating).
4. Authorization becomes scope-based (`GlassPermission`); blanket authorization is removed.
5. The `CUSTOM_APP` scene gains a `glassesApkPath` setting, and the on-device (glasses-side) APK is installed automatically on connect; the default AI intercept mode changes to foreground mode.
6. Audio is unified to PCM 16 kHz / mono / 16-bit; photo capture is unified to WebP encoding.
7. The sample project is restructured as `CxrlSample` (MVVM, single Activity) and offered as an OSS download; iOS is upgraded to `RGCxrClient` 1.1.1.1.

**Version History chapter — 1.1.2:**

- Fixes a `LINK_TIMEOUT` (connection timeout) that could occur the first time a session is created with the `CUSTOM_VIEW` scene.
- Projects on `1.1.0` / `1.1.1` are advised to upgrade to `1.1.2`. If you cannot upgrade yet, a temporary workaround is to complete one connection with the `CUSTOM_APP` scene first and then switch to `CUSTOM_VIEW`.

## v1.1.1

> Source: Version History chapter of the v1.1.2 documentation (fetched 2026-11-13). Upstream gives no release date for this version.

Changes relative to 1.1.0:

- The `CUSTOM_APP` scene configuration gains a `glassesApkPath` field (path of the APK on the mobile side).
- `CUSTOM_VIEW` icon delivery now goes through the `viewData` data chain; the old icon API is removed.
- The default AI intercept mode changes from "background mode" to "foreground mode".

## v1.1.0

> Source: Version History chapter of the v1.1.2 documentation (fetched 2026-11-13). The chapter gives the release time as "2026.6.25" — the same date the portal changelog assigns to V1.0.4. The discrepancy is upstream's and is reproduced here as published.

Changes relative to v1.0.4.

**Changes to the mental model:**

1. **Direct glasses connection removed.** The "app connects directly to the glasses" path is no longer used; everything is relayed through the Rokid AI app (authorization → link establishment → capability calls).
2. **Simpler session creation.** Session creation is decoupled from connection (create first, connect afterwards).
3. **Tighter authorization scope.** Authorization is scope-based (`GlassPermission`); blanket authorization is removed.
4. **Sample project restructured.** Capability pages talk to the SDK session directly rather than through an intermediate layer; session instances are created and released with the page lifecycle.
5. **Explicit capability gating.** The differences in capability support between `CUSTOM_VIEW` and `CUSTOM_APP` are called out in each chapter.

**Per-platform details** (table reproduced as published upstream):

| Dimension | Android | iOS |
|-----------|---------|-----|
| State machine | Five states: Idle / Starting / Started / Paused / Terminating | (single cell upstream; applies to both columns) |
| Authorization | `authenticate(scopes:completion:)` scope-based authorization | Same as Android |
| `CUSTOM_VIEW` | `viewData` JSON data chain + icon delivery | Same as Android |
| `CUSTOM_APP` | On-device (glasses-side) APK installed automatically (`glassesApkPath`) | Same as Android |
| Audio | PCM 16 kHz / mono / 16-bit, single stream | Same as Android |
| Photo capture | Returned WebP-encoded | Same as Android |
| Buttons | Reported through the on-device CXR-S side (`rk_custom_key`) | Same as Android |

### Migration notes (v1.0.x → v1.1.0)

- The mobile side no longer connects to the glasses directly: install and authorize the Rokid AI app first, then let it relay through the SDK.
- The `authenticated(token:)` case of `authStatePublisher` carries both the token and `expiresAt`; adapt your code to handle expiry.
- The app-management APIs (`openApp` / `closeApp` / `startApp` / `stopApp` / `uninstallApp` and similar) are no longer provided. Use "`glassesApkPath` preset + automatic install on connect" instead.
- Photo output is unified from JPEG to WebP; the reading and display side must decode WebP.

### Known implementation quirks (confirmed upstream)

- Audio capture is unified to PCM 16 kHz / mono / 16-bit (verified on a real device); configuring a sample-rate parameter does not change the actual format.
- Photos are WebP-encoded (verified on a real device); the file-name suffix may not match the encoding, so trust the decode result.
- Rokid AI app v1.9.0 may wrongly display the connection state as connected; rely on the SDK state callbacks for the actual behavior.
- For `LINK_LOST` recovery, the uniformly recommended and most robust path is to re-authorize.

### Earlier versions as summarized upstream (v1.0.x excerpt)

The v1.1.2 Version History chapter summarizes the v1.0.x line as follows. Note that it attributes the sample projects differently from the portal changelog entries reproduced further down this page (which list both samples under V1.0.3); both are reproduced as published.

| Version | Change |
|---------|--------|
| v1.0.4 | On-device (glasses-side) sample project `CXRSWithCXRLSample` first published with the documentation |
| v1.0.3 | `RenewCXRLSample` sample project published |
| v1.0.1 | CXR-L SDK first release (Android / iOS) |

## v1.0.4 — published 2026-06-18

> **Official changelog (portal, "V1.0.4", updated 2026.06.25; fetched 2026-11-13):** 1. Adds volume control. 2. Adds brightness control. This confirms the `setGlassVolume` / `setGlassBrightness` additions found in the binary diff below; the remaining detail in this entry is still reconstructed rather than official.

> **Provisional detail — not from an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.0.3` and `client-l:1.0.4` AARs (downloaded 2026-06-25 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). AAR size: 70,543 bytes vs 65,494 bytes for v1.0.3 (+7.7 %).

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
