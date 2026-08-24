# CXR-L SDK Introduction

> Source: <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.4** (supersedes the v1.0.6-labelled Chinese revision fetched 2026-06-20 from the same workspace; the doc portal's own version banner now reads v1.0.4, matching the latest entry in [Release Notes](release-notes.md). See [Release Notes](release-notes.md) for a note on newer `client-l` releases published on Maven without an accompanying changelog.)

## Positioning

The CXR-L SDK runs on the **mobile phone**. It works with **Rokid Glasses** and the **Rokid AI App** (or **Hi Rokid**) to complete authentication, session establishment, Custom View, glasses-side app control, audio, photo capture, and custom commands.

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

## Core capabilities

| Capability | Description |
| --- | --- |
| Connection and session | Create link, configure session type, register callbacks, `connect(token)` |
| Glasses Custom View | Layout JSON + icon resources; open, update, close |
| Glasses Custom App | Query/install/upload APK, start, stop, uninstall |
| Custom command | Bidirectional messages (CustomApp only) |
| Audio | PCM audio stream |
| Photo capture | Remote JPEG capture |
| Device control | Set/query glasses brightness (0…15) and volume (0…15); available once link is ready |

## Capability prerequisites

| Capability | Prerequisites |
| --- | --- |
| Audio | Scene building complete; reuse global `CXRLink` |
| Photo capture | Same as audio |
| Custom command | CustomApp only; glasses app opened |
| Device control (brightness/volume) | Available once link is ready; no scene building required |

## Capability availability matrix

| Session / state | Audio | Photo | Custom command | Brightness | Volume |
| --- | --- | --- | --- | --- | --- |
| Not authenticated | No | No | No | No | No |
| Authenticated, not connected | No | No | No | No | No |
| Connected, scene not built | No | No | No | Yes | Yes |
| CUSTOMVIEW + view opened | Yes | Yes | No | Yes | Yes |
| CUSTOMAPP + app opened | Yes | Yes | Yes | Yes | Yes |

## Sample projects

### Android (v1.0.4)

- **Project**: RenewCXRLSample (`com.rokid.renewcxrlsample`)
- **Zip package**: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/CXRLSample.zip`
- **SDK**: `com.rokid.cxr:client-l:1.0.4`
- **Required app**: Rokid AI App **≥ 1.9.0** (mainland) or Hi Rokid (overseas)
- **Architecture**: single Activity with `NavHost` (`CxrSessionActivity`), global `CXRLApplication.sharedLink`

> **Naming note:** the Android sample project was renamed from `CXRLSample` (`com.rokid.cxrlsample`) to **RenewCXRLSample** (`com.rokid.renewcxrlsample`) as of this v1.0.4 documentation refresh.

### iOS (v1.0.4)

- **Project**: `ios_cxr_l_sample`
- **Zip package**: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/iOS/ios_cxr_l_sample.zip`
- **SDK**: CocoaPods `RGCxrClient` `1.0.4`

### CXR-S SDK (glasses)

The **CXR-S SDK** (Maven artifact `cxr-service-bridge`) lets Rokid glasses-side Android apps join the CXR protocol and work with phone-side CXR-L. The phone app handles auth, sessions, and remote control; the glasses app runs CustomApp logic and exchanges `Caps` with the phone.

| Side | SDK | Runtime | Responsibility |
| --- | --- | --- | --- |
| Phone | CXR-L (`client-l`) | Phone app | Auth, sessions, CustomView, CustomApp remote control, audio/photo/custom commands |
| Glasses | CXR-S (`cxr-service-bridge`) | Glasses Android app | CustomApp logic, `CXRServiceBridge`, `Caps` with phone |

- **CustomView session**: phone sends layout JSON via CXR-L; **phone app does not embed CXR-S**.
- **CustomApp session**: phone installs/launches the glasses APK via CXR-L; that APK **must integrate CXR-S** and match `CUSTOMAPP.packageName`.

**CXRSWithCXRLSample** (`com.rokid.cxrswithcxrl`) pairs with RenewCXRLSample. This sample covers **CustomApp + custom commands + key reporting** only — not CustomView rendering, audio, or photo (future doc releases).

| Item | Value |
| --- | --- |
| Glasses package | `com.rokid.cxrswithcxrl` |
| Entry Activity | `.activities.main.MainActivity` |
| SDK dependency | `com.rokid.cxr:cxr-service-bridge` (version per Sample / release notes) |
| Zip package | `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.3/cxrssample.zip` |

Related chapters: [Quick Start](quick-start.md), [Terms and Abbreviations](terms-and-abbreviations.md).

**Link ready** before CustomView/CustomApp APIs: `onCXRLConnected(true)` and `onGlassBtConnected(true)`.

> **See also:** [Quick Start](quick-start.md) for build/install/verification steps, [Terms and Abbreviations](terms-and-abbreviations.md) for the SDK glossary, [API Reference](api-reference.md) for the full `CXRLink` / `ExternalAppClient` method reference, and [Release Notes](release-notes.md) for the SDK version history.
