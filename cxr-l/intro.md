# CXR-L SDK Introduction

_Source: <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html> (official English edition, reached via the CXR-L "View" link on `ar.rokid.com`; fetched 2026-11-12). Documentation version: **1.0.4** (per the doc site's version selector). Supersedes the earlier translation of the Chinese edition (`.../pc/cn/9adcfb07939846e5945e79dfbd923f63.html`, fetched 2026-06-20), which no longer renders content._

> **Version note (2026-11-12):** The SDK portal (`open.rokid.com/sdk`) lists CXR-L **1.1.2** as the latest release (updated 2026.09.08), and Maven publishes `client-l` 1.1.0, 1.1.1 and 1.1.2. The documentation site itself is still versioned **1.0.4**. This page reflects the 1.0.4 documentation; see [Release Notes](release-notes.md) for version history.

## Positioning

The CXR-L SDK runs on the **mobile phone**. It works with **Rokid Glasses** and the **Rokid AI App** (or **Hi Rokid**, the overseas edition) to complete authentication, session establishment, Custom View, glasses-side app control, audio, photo capture, and custom commands.

Typical flow:

1. Integrate the SDK and guide the user to install/launch the required app.
2. Obtain a `token` through authorization.
3. Establish a `CustomView` or `CustomApp` session and keep the link available.
4. **Complete scene building** on the glasses.
5. Use **photo capture, audio, and custom commands** after the scene is ready.
6. Once the link is ready, use **device control (brightness/volume)**.

**Scene building** means the glasses have reached the business working state — not merely phone-side `connect` success.

- **CustomView**: `customViewOpen` succeeds and `onCustomViewOpened` is received.
- **CustomApp**: the target package is installed and `appStart` succeeds; the app is in the foreground and interactive.

**Custom commands** are available **only in CustomApp** sessions.

## Core Capabilities

| Capability | Description |
| --- | --- |
| Connection and session | Create the link, configure the session type, register callbacks, `connect(token)` |
| Glasses Custom View | Layout JSON + icon resources; open, update, close |
| Glasses Custom App | Query/install/upload APK, start, stop, uninstall |
| Custom command | Bidirectional messages (CustomApp only) |
| Audio | PCM audio stream |
| Photo capture | Remote JPEG capture |
| Device control | Set/query glasses brightness (0…15) and volume (0…15); available once the link is ready |

## Capability Prerequisites

| Capability | Prerequisites |
| --- | --- |
| Audio | Scene building complete; reuse the global `CXRLink` |
| Photo capture | Same as audio |
| Custom command | CustomApp only; glasses app opened |
| Device control (brightness/volume) | Available once the link is ready; no scene building required |

## Capability Availability Matrix

| Session / state | Audio | Photo | Custom command | Brightness | Volume |
| --- | --- | --- | --- | --- | --- |
| Not authenticated | No | No | No | No | No |
| Authenticated, not connected | No | No | No | No | No |
| Connected, scene not built | No | No | No | Yes | Yes |
| `CUSTOMVIEW` + view opened | Yes | Yes | No | Yes | Yes |
| `CUSTOMAPP` + app opened | Yes | Yes | Yes | Yes | Yes |

## Sample Projects

### Android (v1.0.4)

| Item | Value |
| --- | --- |
| Project | RenewCXRLSample (`com.rokid.renewcxrlsample`) |
| Zip package | <https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/CXRLSample.zip> |
| SDK | `com.rokid.cxr:client-l:1.0.4` |
| Required app | Rokid AI App **≥ 1.9.0** (mainland China) or Hi Rokid (overseas) |
| Architecture | Single Activity with NavHost (`CxrSessionActivity`), global `CXRLApplication.sharedLink` |

### iOS (v1.0.4)

| Item | Value |
| --- | --- |
| Project | `ios_cxr_l_sample` |
| Zip package | <https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/iOS/ios_cxr_l_sample.zip> |
| SDK | CocoaPods `RGCxrClient` `1.0.4` |

### CXR-S SDK (glasses)

The **CXR-S SDK** (Maven artifact `cxr-service-bridge`) lets Rokid glasses-side Android apps join the CXR protocol and work with phone-side CXR-L. The phone app handles authentication, sessions, and remote control; the glasses app runs the CustomApp logic and exchanges [`Caps`](../cxr-s/data-structure.md) with the phone.

| Side | SDK | Runtime | Responsibility |
| --- | --- | --- | --- |
| Phone | CXR-L (`client-l`) | Phone app | Auth, sessions, CustomView, CustomApp remote control, audio/photo/custom commands |
| Glasses | CXR-S (`cxr-service-bridge`) | Glasses Android app | CustomApp logic, `CXRServiceBridge`, `Caps` with phone |

- **CustomView session**: the phone sends layout JSON via CXR-L; the **phone app does not embed CXR-S**.
- **CustomApp session**: the phone installs/launches the glasses APK via CXR-L; that APK **must integrate CXR-S** and match `CUSTOMAPP.packageName`.

**CXRSWithCXRLSample** (`com.rokid.cxrswithcxrl`) pairs with RenewCXRLSample. This sample covers **CustomApp + custom commands + key reporting** only — not CustomView rendering, audio, or photo (planned for future doc releases).

| Item | Value |
| --- | --- |
| Glasses package | `com.rokid.cxrswithcxrl` |
| Entry Activity | `.activities.main.MainActivity` |
| SDK dependency | `com.rokid.cxr:cxr-service-bridge` (version per sample / release notes) |
| Zip package | <https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.3/cxrssample.zip> |

Related upstream chapters: SDK Integration (glasses side), Glasses Custom App, Custom Commands, Keys and System Broadcasts.

**Link ready** — before calling any CustomView/CustomApp API, both `onCXRLConnected(true)` and `onGlassBtConnected(true)` must have fired.

> **See also:** [API Reference](api-reference.md) for the full `CXRLink` / `ExternalAppClient` method reference (note: it lists the Android launch method as `openApp`; the upstream intro uses `appStart` for the same step), and [Release Notes](release-notes.md) for the SDK version history.
