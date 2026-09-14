# YodaOS-Sprite

> Source: https://developerdoc.rokid.com/sprite (rendered SPA, Chinese, fetched 2026-06-08); hardware-family wording updated from ar.rokid.com and developer.rokid.com (2026-06-26).

## About YodaOS-Sprite

YodaOS-Sprite is a smart-glasses operating system purpose-built for all-day wear. Through end-to-end optimization across chip, drivers, system, applications, and product scenarios, it strikes a balance between battery life, performance, and functionality. The system is built around a philosophy of unobtrusive design — letting the technology fade into the background so the glasses feel like a seamless extension of daily life.

YodaOS-Sprite runs on **Rokid Glasses** and **Bolon AI Glasses** (OEM variants sharing the same SoC and software stack). For the full hardware model list see [hardware/product-variants.md](hardware/product-variants.md).

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

> This FAQ was re-scraped from the upstream `developerdoc.rokid.com/sprite` page on 2026-07-17. The upstream FAQ now shows only 4 Q&As, all focused on CXR-M/CXR-S topics — replacing the 7 CXR-L/bare-metal Q&As that had been captured here on 2026-06-08. This appears to be a content rollback or restructuring on Rokid's side; the cause is unknown. The discrepancy was flagged and monitored across five refresh cycles (2026-07-03 through 2026-07-17) before being actioned here, including two independent scrapes on 2026-07-17 that were verified byte-identical. The removed CXR-L/bare-metal FAQ content is preserved in this repository's git history (prior versions of this file) in case it becomes relevant again.

**Q: What are the main capabilities of the CXR-M SDK?**

**A:** A mobile application built with the CXR-M SDK can communicate with Rokid Glasses to receive audio and video from the glasses, and can customize the implementation of existing scenes on Rokid Glasses. When paired with the Rokid CXR-S SDK, it can also exchange custom commands.

**Q: Which devices does the CXR-M SDK currently support?**

**A:** Currently, the CXR-M SDK is only available for Android mobile devices.

**Q: What are the main capabilities of the CXR-S SDK?**

**A:** The CXR-S SDK provides access to the data channel on YodaOS-Sprite, letting custom commands be passed to the CXR-M SDK through that data channel.

**Q: When building a mobile application using only the CXR-M SDK, do I need to enable developer permissions on Rokid Glasses?**

**A:** No.

For additional questions, see the [Rokid Developer Forum](https://forum.rokid.com/).

## Notes

- **Source**: https://developerdoc.rokid.com/sprite — the canonical developer portal for YodaOS-Sprite (loaded via iframe from `https://ar.rokid.com/sprite?lang=zh`). Initially captured on 2026-05-29; device specifications table added 2026-06-03; optical display specs (FOV, brightness, resolution, optics design) added 2026-06-04; FAQ section refreshed 2026-06-08. FAQ Q4 updated 2026-06-27 to pin `client-l:1.0.4` (Maven latest as of 2026-06-25; portal FAQ still shows 1.0.3). FAQ re-synced 2026-07-17 — upstream reverted to 4 CXR-M/CXR-S Q&As, replacing the CXR-L/bare-metal content added 2026-06-08 (see the note at the top of the FAQ section above for details).
- **Scope**: This document covers the **YodaOS-Sprite** tab only — the OS that runs on Rokid Glasses and Rokid AI Glasses. A separate "YodaOS-Master" tab exists on the upstream page (covering Station 2 / Station Pro / AR Lite / AR Studio) but is out of scope for this repository.
- **Known TODO links**:
  - "View specs" button under *About YodaOS-Sprite* — destination not captured; links to a hardware-spec modal on the portal SPA.
