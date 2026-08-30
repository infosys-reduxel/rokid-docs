# CXR-L SDK Introduction

_Source: <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html> (reached via short link <https://t.rokid.com/uwxdzi51>), fetched 2026-08-30. Doc UI reports "Version 1.0.4". This is a **new docId** — the previous revision of this page lived at docId `9adcfb07939846e5945e79dfbd923f63`; the doc was reorganized/replaced upstream, not merely edited in place. The `/pc/us/` path serves this content natively in English (not machine-translated from the `/pc/cn/` Chinese revision), so no zh→en translation pass was needed for this update._

## Positioning

The CXR-L SDK runs on the **mobile phone**. It works with **Rokid Glasses** and the **Rokid AI app** (or **Hi Rokid** overseas) to complete authentication, session establishment, Custom View, glasses-side app control, audio, photo capture, and custom commands.

Typical flow:

1. Integrate the SDK and guide the user to install/launch the required app.
2. Obtain a `token` through authorization.
3. Establish a `CustomView` or `CustomApp` session and keep the link available.
4. **Complete scene building** on the glasses.
5. Use **photo capture, audio, and custom commands** after the scene is ready.
6. Once the link is ready, use **device control (brightness/volume)**.

**Scene building** means the glasses have reached the business working state — not merely phone-side `connect` success.

- **CustomView**: `customViewOpen` succeeds and `onCustomViewOpened` is received.
- **CustomApp**: target package installed and `appStart` succeeds; app is foreground/interactive.

**Custom commands** are available **only in CustomApp** sessions.

## Core Capabilities

| Capability | Description |
| --- | --- |
| Connection and session | Create link, configure session type, register callbacks, `connect(token)` |
| Glasses Custom View | Layout JSON + icon resources; open, update, close |
| Glasses Custom App | Query/install/upload APK, start, stop, uninstall |
| Custom command | Bidirectional messages (CustomApp only) |
| Audio | PCM audio stream |
| Photo capture | Remote JPEG capture |
| Device control | Set/query glasses brightness (0–15) and volume (0–15); available once link is ready |

## Capability Prerequisites

| Capability | Prerequisites |
| --- | --- |
| Audio | Scene building complete; reuse global `CXRLink` |
| Photo capture | Same as audio |
| Custom command | CustomApp only; glasses app opened |
| Device control (brightness/volume) | Available once link is ready; **no scene building required** |

## Capability Availability Matrix

| Session / State | Audio | Photo | Custom command | Brightness | Volume |
| --- | --- | --- | --- | --- | --- |
| Not authenticated | No | No | No | No | No |
| Authenticated, not connected | No | No | No | No | No |
| Connected, scene not built | No | No | No | Yes | Yes |
| `CUSTOMVIEW` + view opened | Yes | Yes | No | Yes | Yes |
| `CUSTOMAPP` + app opened | Yes | Yes | Yes | Yes | Yes |

Note the last two prerequisite/availability rows above: unlike audio, photo capture, and custom commands, **device control (brightness/volume) does not require scene building** — it becomes available as soon as the link is connected.

## Sample Projects

### Android (v1.0.4)

- **Project**: `RenewCXRLSample` (`com.rokid.renewcxrlsample`) — renamed from the previous `CXRLSample` (`com.rokid.cxrlsample`)
- **Zip package**: [`https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/CXRLSample.zip`](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/CXRLSample.zip) (the zip file name itself still reads `CXRLSample.zip` upstream, despite the project/package rename — reproduced verbatim, not a transcription error)
- **SDK**: `com.rokid.cxr:client-l:1.0.4`
- **Required app**: Rokid AI App **≥ 1.9.0** (mainland) or Hi Rokid (overseas)
- **Architecture**: single Activity with NavHost (`CxrSessionActivity`), global `CXRLApplication.sharedLink`

### iOS (v1.0.4)

- **Project**: `ios_cxr_l_sample`
- **Zip package**: [`https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/iOS/ios_cxr_l_sample.zip`](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/iOS/ios_cxr_l_sample.zip)
- **SDK**: CocoaPods `RGCxrClient` `1.0.4`

### CXR-S SDK (glasses side)

The **CXR-S SDK** (Maven artifact `cxr-service-bridge`) lets Rokid glasses-side Android apps join the CXR protocol and work with phone-side CXR-L. The phone app handles auth, sessions, and remote control; the glasses app runs CustomApp logic and exchanges [`Caps`](../cxr-s/data-structure.md) with the phone. See [CXR-S SDK Brief](../cxr-s/brief.md) and [CXR-S SDK Import](../cxr-s/sdk-import.md) for glasses-side integration details.

| Side | SDK | Runtime | Responsibility |
| --- | --- | --- | --- |
| Phone | CXR-L (`client-l`) | Phone app | Auth, sessions, CustomView, CustomApp remote control, audio/photo/custom commands |
| Glasses | CXR-S (`cxr-service-bridge`) | Glasses Android app | CustomApp logic, `CXRServiceBridge`, `Caps` with phone |

- **CustomView session**: phone sends layout JSON via CXR-L; **phone app does not embed CXR-S**.
- **CustomApp session**: phone installs/launches the glasses APK via CXR-L; that APK **must integrate CXR-S** and match `CUSTOMAPP.packageName`.

**CXRSWithCXRLSample** (`com.rokid.cxrswithcxrl`) pairs with `RenewCXRLSample`. This sample covers **CustomApp + custom commands + key reporting** only — not CustomView rendering, audio, or photo (per upstream doc, reserved for future doc releases).

| Item | Value |
| --- | --- |
| Glasses package | `com.rokid.cxrswithcxrl` |
| Entry Activity | `.activities.main.MainActivity` |
| SDK dependency | `com.rokid.cxr:cxr-service-bridge` (version per sample / release notes) |
| Zip package | [`cxrssample.zip`](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.3/cxrssample.zip) |

Related chapters (upstream): SDK Integration (glasses-side), Glasses Custom App, Custom Commands, Keys and System Broadcasts.

**Link ready**, a precondition for CustomView/CustomApp APIs, is defined as: `onCXRLConnected(true)` **and** `onGlassBtConnected(true)`.

This corresponds to the `CXRSWithCXRLSample` project referenced in the [v1.0.3 release notes](release-notes.md).

> **See also:** [API Reference](api-reference.md) for the full `CXRLink` / `ExternalAppClient` method reference, and [Release Notes](release-notes.md) for the SDK version history.
