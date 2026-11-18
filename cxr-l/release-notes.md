# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese; official Rokid changelog, CXR-L card with the "更新内容" panel expanded). v1.1.2 and the official v1.0.4 entry were fetched 2026-11-18; v1.0.1–v1.0.3 were fetched 2026-06-11. The detailed v1.0.4 API tables come from a binary diff of Maven AARs (2026-06-25) and are marked as such._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.2 — published 2026-09-08

> Source: official Rokid changelog and quick-start panel at `https://developerdoc.rokid.com/sdk` (CXR-L card, fetched 2026-11-18). `com.rokid.cxr:client-l:1.1.2` is the current `<release>` in `maven-metadata.xml` (`lastUpdated` 20261105064636, fetched 2026-11-18).

**Official changelog:**

1. Android `client-l` upgraded to 1.1.2; fixes the link timeout (`LINK_TIMEOUT`) on the first `CUSTOM_VIEW` session connection.
2. Architecture refactor: the mobile (companion) side no longer connects directly to the glasses — everything is relayed through the Rokid AI App (authorization → link establishment → capability calls).
3. Session creation is decoupled from connection, and a five-state state machine is introduced (`Idle` / `Starting` / `Started` / `Paused` / `Terminating`).
4. Authorization is now scope-based (`GlassPermission`); blanket (all-permission) authorization has been removed.
5. The `CUSTOM_APP` scenario adds a `glassesApkPath` setting — the on-device (glasses-side) APK is installed automatically on connect. The default AI intercept mode is changed to foreground mode.
6. Audio is unified to PCM 16 kHz / mono / 16-bit; photo capture is unified to WebP encoding.
7. The sample project is restructured as `CxrlSample` (MVVM, single Activity) and is available as an OSS download; iOS is upgraded to `RGCxrClient` 1.1.1.1.

> **This is a breaking, re-architected API.** The v1.1.x entry point is `CxrSessionManager` / `SessionConfig` / `ISessionLifecycleCbk`, not the `CXRLink` + `configCXRSession` surface documented for v1.0.x in [api-reference.md](api-reference.md) and [intro.md](intro.md). Those two documents still describe v1.0.x. Only the quick start below has been captured for v1.1.x; the full v1.1.x detail documentation has not yet been retrieved or translated, and no v1.1.x AAR has been decompiled in this repository.

### Quick start (Android, v1.1.2)

Add the Rokid Maven repository in `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        maven { url = uri("https://maven.rokid.com/repository/maven-public/") }
    }
}
```

Add the SDK dependency in `app/build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.rokid.cxr:client-l:1.1.2")
}
```

Create a session and connect (authorization must be completed first to obtain a `token`):

```kotlin
// Singleton entry point
val manager = CxrSessionManager.getInstance(context.applicationContext)

// Create a session, using the CustomView scenario as an example
// (viewData is the on-device view JSON; see the upstream docs)
val config = SessionConfig(
    sessionType = SessionType.CUSTOM_VIEW,
    aiInterceptMode = AiInterceptMode.ALLOW_WITH_PAUSE,
    timeouts = SessionTimeouts(),
    viewData = viewDataJson,
)
val session = manager.create(config)

// Connect after authorization has returned a token
session.connect(token)
```

**Wait for the session to be ready (important):** a successful `connect` does **not** mean photo capture or streaming is available. Wait for the lifecycle callback:

```kotlin
// Call capability APIs only after onSessionStarted
session.addLifecycleCallback(object : ISessionLifecycleCbk {
    override fun onSessionStarted() {
        // Session construction complete; audio / photo capabilities can be called
        // Custom commands are not available in the CUSTOM_VIEW scenario
    }
    override fun onSessionPaused(reason: PausedReason) { /* handle pause */ }
    override fun onSessionResumed() { /* resume */ }
    override fun onSessionTerminating(reason: TerminatingReason, gracePeriodMs: Long) { /* handle termination */ }
    override fun onSessionClosed(reason: CloseReason) { /* back to Idle */ }
})
```

**Common pitfalls:**

- **`minSdk` must be ≥ 28** (the SDK uses Android 9+ features internally). Note that this is what the v1.1.2 quick start states; the v1.0.3 documentation required `minSdk` 31.
- **Reuse the singleton entry point** `CxrSessionManager.getInstance()`; do not create it repeatedly per screen.
- A successful connection must still wait for the `onSessionStarted` callback — the return value of `connect()` does not indicate that the session is usable.
- Custom commands are not available in the `CUSTOM_VIEW` scenario; use the `CUSTOM_APP` scenario if you need them.

### Quick start (iOS, conceptual)

After integrating the Pod or the bundled source module, the flow is authentication → link establishment → session. Upstream notes that exact class names and signatures should be taken from the sample source.

Declare the dependency in the `Podfile` (`RGCxrClient` requires the Rokid private specs source, or use the source module bundled with the sample):

```ruby
pod 'RGCxrClient', '1.1.1.1'
```

Forward the URL callback (authentication return) from `AppDelegate` / `SceneDelegate`:

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    if CxrClient.shared.handleOpenURL(URLContexts.first?.url) {
        // URL was handled by the SDK
    }
}
```

Authenticate and create a session (CustomView scenario shown; after connecting, subscribe to `statePublisher` and wait for the session to become available):

```swift
let link = CxrClient.makeLink(appDisplayName: "YourApp")

link.authenticate(scopes: [.camera, .media, .microphone, .deviceManage]) { result in
    switch result {
    case .success(let auth):
        let token = auth.token
        // Create a CustomView session (makeCustomAppSession(packageName:) is also available)
        let session = link.makeCustomViewSession()
        // Subscribe to statePublisher; wait for available / started before calling capability APIs
    case .failure(let error):
        // Handle the error
        break
    }
}
```

**Notes:**

- Upstream states that the iOS and Android state machines are consistent, and describes the iOS one as four states (`available` / `started` / `paused` / `unavailable`), with the same capability gating (photo, audio, and custom commands require session construction to be complete). The Android changelog above describes five states; both statements are reproduced as published.
- `CxrClient.makeLink(appDisplayName:)` is a **class method**, not `CxrClient.shared.makeLink`.

## v1.1.0 / v1.1.1 — no changelog published

`maven-metadata.xml` for `com.rokid.cxr:client-l` (fetched 2026-11-18) lists releases `1.1.0` and `1.1.1` between `1.0.4` and `1.1.2`. The portal changelog jumps directly from V1.0.4 to V1.1.2, so there is no official description of these two releases, and they have not been diffed in this repository.

## v1.0.4 — published 2026-06-18

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L card, fetched 2026-11-18), which dates this release 2026-06-25.

**Official changelog:**

1. Added volume control.
2. Added brightness control.

> **The detail below is not from the official changelog.** It was reconstructed from a binary diff of `client-l:1.0.3` and `client-l:1.0.4` AARs (downloaded 2026-06-25 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`), before Rokid published the two-line changelog above. The official changelog confirms the volume and brightness controls; the session lifecycle additions are not mentioned in it. The 2026-06-18 date in the heading was recorded with that binary diff. AAR size: 70,543 bytes vs 65,494 bytes for v1.0.3 (+7.7 %).

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
