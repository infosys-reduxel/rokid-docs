# YodaOS-Sprite

> Source: https://developerdoc.rokid.com/sprite (rendered SPA, Chinese, fetched 2026-06-08)

## About YodaOS-Sprite

YodaOS-Sprite is a smart-glasses operating system purpose-built for all-day wear. Through end-to-end optimization across chip, drivers, system, applications, and product scenarios, it strikes a balance between battery life, performance, and functionality. The system is built around a philosophy of unobtrusive design — letting the technology fade into the background so the glasses feel like a seamless extension of daily life.

## Device Specifications — Rokid Glasses

> Source: https://ar.rokid.com/sprite ("View specifications" panel), fetched 2026-06-03.

| Property | Value |
|----------|-------|
| **Device** | Rokid Glasses |
| **Dimensions** | 143 × 44 × 160.5 mm |
| **Operating temperature** | 0 °C – 35 °C |
| **Water resistance** | IPX4 |
| **SoC** | Qualcomm AR1 |
| **Wi-Fi** | Wi-Fi 6 |
| **Bluetooth** | BT 5.3 |
| **RAM** | 2 GB |
| **ROM** | 32 GB |
| **Battery** | 210 mAh |
| **Microphone** | 4-mic directional array |
| **Speaker** | 2 high-quality ultra-linear speakers |
| **Camera** | SONY IMX681 — 12 MP, 3024 × 4032 px, aperture F/2.25, FOV D: 109 ° |
| **Camera indicator light** | Supported |
| **P-sensor** | Supported |
| **IMU** | 6-axis IMU |
| **Audio PA** | Supported |
| **Interaction** | Function key × 1, touchpad × 1 |
| **Charging contacts** | 5 V / 1 A |
| **Wear detection** | Supported |
| **Optics design** | Micro LED + glass diffractive waveguide |
| **Light engine** | Binocular single green |
| **FOV (display)** | 30 ° |
| **Convergence distance** | Infinity |
| **Brightness** | 1500 nits |
| **Resolution (display)** | 480 × 640 px |

## Developer Toolkit

YodaOS-Sprite offers developers a range of toolkits. Depending on the use case, pick the SDK that fits — building on-device applications, or coordinated experiences that span the glasses and a companion mobile app.

### Mobile Application Development

#### CXR-L SDK (Android / iOS)

The CXR-L SDK is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app manages the connection to Rokid Glasses; integrate the CXR-L SDK into your own app to access the glasses' I/O capabilities — image, audio, display, and command channel — via the Rokid AI app.

See: [CXR-L API reference](../../cxr-l/api-reference.md)

#### CXR-M SDK

The CXR-M SDK is a mobile-side developer toolkit for building Android applications that work in concert with Rokid Glasses. It supports stable connection management, data communication, real-time audio/video, and scene customization, and pairs with the on-device CXR-S SDK. The SDK is not published on the public developer site; to request access to the CXR-M SDK, its documentation, and technical support, contact business partnerships at `Glasses.BD@rokid.com`.

See: [CXR-M SDK introduction](../../cxr-m/intro.md)

### On-device Application Development

#### Bare-Metal Development

Rokid Glasses supports bare-metal (system-direct) development. Developers can build custom applications that run directly on the glasses and manage hardware resources including keys, IMU, and Camera. Depending on the scenario, this approach enables fully standalone glasses applications.

See: [Bare-metal development guide](../../cxr-baremetal/development-guide.md)

#### CXR-S SDK

The CXR-S SDK is the on-device (glasses-side) developer toolkit that runs on YodaOS-Sprite. It is focused on helping developers build standalone applications that run directly on Rokid Glasses. In addition to exposing the on-device data channel, it can establish bidirectional communication with the mobile-side CXR-M SDK, with support for custom protocols and command transport. With CXR-S, developers can reach deeper into the device's hardware, unlock its full potential, and deliver low-latency intelligent experiences.

See: [CXR-S SDK overview](../../cxr-s/brief.md)

## FAQ

> This FAQ was updated from the upstream `developerdoc.rokid.com/sprite` page (fetched 2026-06-08). The upstream page now focuses on CXR-L 1.0.3 and bare-metal development; the earlier CXR-M/CXR-S Q&As have been replaced by the content below.

**Q: What are the main capabilities of the CXR-L SDK?**

**A:** CXR-L runs on the mobile side. After authentication via the Rokid AI App or Hi Rokid (international), it coordinates with Rokid Glasses and supports: connection and session management; custom Views on the glasses (JSON layout and icons); custom Apps on the glasses (install/launch/control APK); audio streaming; remote photo capture; and, within a CustomApp session, custom commands (bidirectional Caps communication). The public documentation and samples are primarily CXR-L-focused; the glasses-side CustomApp requires the CXR-S SDK (`cxr-service-bridge`) as its counterpart.

**Q: What is the difference between a CustomView session and a CustomApp session?**

**A:** A CustomView session has the mobile side push a layout JSON that the glasses render as a custom UI; the mobile app does not need to integrate CXR-S. A CustomApp session has the mobile side remotely install and launch a glasses-side Android application; that APK must integrate CXR-S, and its package name must match the CUSTOMAPP session configuration. Both session types support audio and photo capture once the session is fully constructed; custom commands are only available in CustomApp sessions, and only after the glasses-side target application is open.

**Q: What is "link ready" and "session construction"? Why does photo capture still fail after `connect()` succeeds?**

**A:** "Link ready" means `onCXRLConnected(true)` and `onGlassBtConnected(true)` have both fired — only then can CustomView or CustomApp APIs be called. "Session construction" means the glasses are in the agreed business working state: for CustomView, `customViewOpen` must have succeeded and the `onCustomViewOpened` callback received; for CustomApp, `appStart` must have succeeded and `onOpenAppResult(true)` received. A successful `connect()` alone, or having a token alone, does not mean session construction is complete. Audio, photo capture, and custom commands all require session construction to have been completed first; calling them before that point will fail or be gated.

**Q: What are the environment prerequisites for CXR-L SDK development?**

**A:** A real device with a working Bluetooth environment is required, along with the Rokid AI App or Hi Rokid (international) installed on the phone, with `requestAuthorization` completed to obtain a token. For Android, integrate `com.rokid.cxr:client-l:1.0.3` (`minSdk 31+`). For iOS, use CocoaPods `RGCxrClient` — the current iOS chapter in the documentation corresponds to v1.0.1; alignment with Android v1.0.3 capability follows the timeline of each platform chapter. Refer to RenewCXRLSample (mobile) and CXRSWithCXRLSample (glasses CustomApp joint-debugging) samples and the documentation OSS archive.

**Q: How are custom commands (CustomCmd) used? What are the restrictions?**

**A:** Custom commands enable bidirectional binary messaging between the mobile app and the glasses-side custom application. On Android, send via `Caps` and `sendCustomCmd`; on the glasses side, CXR-S uses `CXRServiceBridge` `subscribe`/`sendMessage` as the counterpart. Custom commands are only available in a CUSTOMAPP session, and only after the glasses-side application has started (session construction complete); CustomView sessions do not support them. Key and touchpad events can be reported from the glasses-side application and received on the mobile side via custom command callbacks (see the key and system broadcast chapter).

**Q: What scenarios is bare-metal glasses development suited for? How does it compare to CXR-L?**

**A:** Bare-metal development means writing standard Android applications on YodaOS-Sprite (Android Go) without a mobile-companion SDK — appropriate for Launcher-type apps, on-device audio recording, photo/video capture, key and wear/fold event listening, and IMU access — all scenarios that run entirely on the glasses without a phone. If you need a mobile app to remotely control the glasses UI or an app, or to receive glasses audio/video and command channels, use CXR-L (with CXR-S for CustomApp). The two tracks can coexist on different product lines; do not mix the connection models within the same business flow.

**Q: What are the main capabilities of bare-metal glasses development?**

**A:** Bare-metal applications use standard Android mechanisms: system broadcasts for wear/fold detection (`ACTION_TAKE_STATUS_CHANGED`, `ACTION_LEG_STATUS_CHANGED`) and function-key/touchpad events; 8-channel `AudioRecord` raw audio; CameraX for photo and video capture (no preview, triggered by a single touchpad tap, files saved to `/sdcard/Pictures/bare_photo/*.jpg` and `/sdcard/Video/bare_video/*.mp4`); `SensorManager` six-axis IMU. Screen resolution is 480 × 640 px; Android Go constraints and glasses UI design guidelines apply. Documentation and samples describe application-layer APIs only and do not cover system-internal implementation details.

**Q: How do I install and debug a bare-metal application on the glasses?**

**A:** Build the APK in Android Studio (see `GlassesBareDevSample`, `minSdk 31`), connect the glasses to the computer via the dedicated developer cable, enable ADB in the Rokid AI App on the phone, then run `adb install`. Use `scrcpy` to mirror the 480 × 640 display. The cable included in the retail box is typically a charge-only cable — contact the developer support team to obtain a developer cable. The bare-metal path does not replace the CXR-L authentication and Rokid AI App coordination flow.

For additional questions, see the [Rokid Developer Forum](https://forum.rokid.com/).

## Notes

- **Source**: https://developerdoc.rokid.com/sprite — the canonical developer portal for YodaOS-Sprite (loaded via iframe from `https://ar.rokid.com/sprite?lang=zh`). Initially captured on 2026-05-29; device specifications table added 2026-06-03; optical display specs (FOV, brightness, resolution, optics design) added 2026-06-04; FAQ section refreshed 2026-06-08 to reflect CXR-L 1.0.3 and bare-metal guidance.
- **Scope**: This document covers the **YodaOS-Sprite** tab only — the OS that runs on Rokid Glasses and Rokid AI Glasses. A separate "YodaOS-Master" tab exists on the upstream page (covering Station 2 / Station Pro / AR Lite / AR Studio) but is out of scope for this repository.
- **Known TODO links**:
  - "View specs" button under *About YodaOS-Sprite* — destination not captured; links to a hardware-spec modal on the portal SPA.
