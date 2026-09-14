# CXR-L SDK Introduction

_Source: <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/cn/9adcfb07939846e5945e79dfbd923f63.html> (Chinese, fetched 2026-06-20). Content version: SDK v1.0.6 (per `window.relatedVersion` in the page HTML shell). Translation produced from Firecrawl-cached markdown; static.rokidcdn.com JS assets are not reachable from this environment. The **Device control** capability (brightness/volume) was added to this page upstream and pulled in on 2026-09-13 — same content version (v1.0.6), page otherwise unchanged._

## Positioning

The CXR-L SDK runs on the **mobile (companion)** side and works in conjunction with **Rokid Glasses** and the **Rokid AI app** to handle authorization, session establishment, and capabilities such as custom View scenes, on-device (glasses-side) app control, audio, photo capture, and custom commands.

Typical integration flow:

1. The mobile app integrates the SDK and guides the user to install or launch the Rokid AI app.
2. Obtain a `token` through the accompanying authorization flow.
3. Establish a `CustomView` or `CustomApp` session and keep the link available (`connect` succeeds and service-side prerequisites such as Bluetooth are met).
4. **Complete scene construction**: the on-device (glasses-side) end has actually presented the UI or app process that the target capability depends on (see "Scene Construction" below).
5. After the scene is ready, use capabilities such as **photo capture, audio, and custom commands**. The APIs for connection, custom View, and app control themselves are still called in the order described in their respective chapters.
6. Once the **link is ready** (see below), the **device control (brightness/volume)** capability becomes available — it does not require scene construction.

**Scene construction (important):** This means the on-device (glasses-side) end is in the operational state agreed upon by the business, not merely that the mobile-side `connect` call has returned successfully.

- **Custom View scenes**: Under a `CUSTOMVIEW` session, `customViewOpen` (or an equivalent operation) has completed and the "view opened" callback has been received — the glasses are displaying the custom interface.
- **Custom app scenes**: Under a `CUSTOMAPP` session, the target package is installed and `openApp` has succeeded — the on-device app is in the foreground and interactive (sample apps use `appOpened` to indicate the scene is open).

**Photo capture, audio, and custom commands** must all be called **after scene construction is complete** for the corresponding scene; merely establishing a session or having a live link is not sufficient to guarantee these capabilities are operational. **Custom commands** in particular must be used under a custom app scene.

**Link ready:** before calling `CustomView`/`CustomApp`-related APIs, both `onCXRLConnected(true)` and `onGlassBtConnected(true)` must have fired. Device control (brightness/volume) only requires the link to be ready — it does not wait on scene construction.

## Core Capabilities

| Capability | Description |
| --- | --- |
| Connection and session | Create a link instance, connect to the `Rokid AI / Hi Rokid` app, configure the session type, and register status callbacks. |
| On-device custom View | Push layout JSON and icon resources; supports open, update, and close operations. |
| On-device custom app | Query installation state, upload and install an APK, launch, stop, and uninstall the target package. |
| Custom commands | Bidirectional custom messages (used together with a `CUSTOMAPP` session). |
| Audio | Start/stop an audio stream and receive PCM data. |
| Photo capture | Trigger capture at a specified resolution and quality; receive a JPEG byte stream. |
| Device control | Set/query the glasses' display brightness (0–15) and speaker volume (0–15); usable once the link is ready. |

## Capability Prerequisites

| Capability | Prerequisites |
| --- | --- |
| Audio | **After scene construction is complete**: for the `CustomView` path, `customViewOpen` must have succeeded; for the `CustomApp` path, `openApp` must have succeeded (on-device scene is open). The same global `CXRLink` instance must be reused. |
| Photo capture | Same as audio. |
| Custom commands | **CustomApp scene only**; must be called **after the on-device app has been opened**. |
| Device control (brightness/volume) | Usable once the link is ready; scene construction is **not** required. |

## Capability Availability Matrix (Summary)

| Session / State | Audio | Photo Capture | Custom Commands | Brightness | Volume |
| --- | --- | --- | --- | --- | --- |
| Not authorized / no token | No | No | No | No | No |
| Authorized but `connect` not called | No | No | No | No | No |
| `connect` succeeded but scene construction not complete (custom View not opened / `openApp` not called) | No | No | No | Yes | Yes |
| `CUSTOMVIEW` session and custom View is open | Yes | Yes | No | Yes | Yes |
| `CUSTOMAPP` session and on-device app is open | Yes | Yes | Yes (same `CXRLink` instance must be reused) | Yes | Yes |

## Sample Projects

- **Android**: `CXRLSample` (`com.rokid.cxrlsample`), depends on `com.rokid.cxr:client-l` (pin to the version in the project's `app/build.gradle.kts`).
- **iOS**: `ios_cxr_l_sample` (clone the repository in full before cross-referencing class names and paths in the documentation).

The on-device companion sample app package name is defined in the Android sample's `CONSTANT.APP_PACKAGE_NAME` constant as `com.rokid.cxrswithcxrl`; it is used to demonstrate `CUSTOMAPP` installation and launch. This corresponds to the `CXRSWithCXRLSample` project referenced in the [v1.0.3 release notes](release-notes.md).

> **See also:** [API Reference](api-reference.md) for the full `CXRLink` / `ExternalAppClient` method reference, and [Release Notes](release-notes.md) for the SDK version history.
