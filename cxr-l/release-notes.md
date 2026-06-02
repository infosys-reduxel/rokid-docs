# CXR-L SDK Release Notes

_Source: https://ar.rokid.com/sdk (Chinese, fetched 2026-05-28; rechecked 2026-06-02)._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.0.3 — published 2026-06-02 (undocumented)

> **Provisional — not an official Rokid changelog.** `com.rokid.cxr:client-l:1.0.3` was uploaded to Maven on 2026-06-02 (AAR date: Tue Jun 02 03:00:04 UTC 2026, size: 65,494 bytes vs 57,145 bytes for 1.0.2, +14.6 %). Rokid has not yet published a changelog on the developer portal or on its `custom.rokid.com` doc site as of 2026-06-02. The entries below are reconstructed from a binary diff of the 1.0.2 and 1.0.3 AARs (downloaded from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`).

**Theme: device-state callbacks and wearing-detection.**

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

## v1.0.2 — published 2026-05-19 (undocumented)

`com.rokid.cxr:client-l:1.0.2` was uploaded to Maven on 2026-05-19 and finalized in the repository index on 2026-05-28, but as of 2026-06-02 Rokid has not yet published a changelog for it on the developer portal or on its `custom.rokid.com` doc site. The POM declares the same dependency set as 1.0.1 (`cxr-service-bridge 1.0-20260212.103714-88`, `kotlin-stdlib 2.1.0`, `gson 2.10.1`). Features below remain those of 1.0.1 until Rokid publishes the 1.0.2 changelog.

<!-- TODO: append the v1.0.2 feature list once it is published upstream. -->

## v1.0.1 — 2026-05-07

1. Initial SDK release.
2. Support for obtaining authorization from the Rokid AI app.
3. Support for creating on-device custom View scenes.
4. Support for creating on-device custom app scenes.
5. Support for accessing on-device audio.
6. Support for capturing photos through the glasses.
7. Support for custom-command exchange with on-device custom apps.
