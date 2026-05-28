# YodaOS-Sprite

> Source: https://ar.rokid.com/sprite (rendered SPA, Chinese, fetched 2026-05-29)

## About YodaOS-Sprite

YodaOS-Sprite is a smart-glasses operating system purpose-built for all-day wear. Through end-to-end optimization across chip, drivers, system, applications, and product scenarios, it strikes a balance between battery life, performance, and functionality. The system is built around a philosophy of unobtrusive design — letting the technology fade into the background so the glasses feel like a seamless extension of daily life.

<!-- TODO: Source had a "[查看配置]" (View specs) button here, but no destination URL was captured. Link a hardware-spec page when available. -->

## Developer Toolkit

YodaOS-Sprite offers developers a range of toolkits. Depending on the use case, pick the SDK that fits — building on-device applications, or coordinated experiences that span the glasses and a companion mobile app.

### Mobile Application Development

#### CXR-L SDK (Android / iOS)

The CXR-L SDK lets developers extend the use cases of the Rokid AI app. The Rokid AI app manages the connection to Rokid Glasses; integrate the CXR-L SDK into your app to reach the glasses' I/O capabilities — image, audio, display, command channel, and more — via the Rokid AI app.

See: [CXR-L API reference](../../cxr-l/api-reference.md)

#### CXR-M SDK

The CXR-M SDK is a mobile-side developer toolkit for building Android applications that work in concert with Rokid Glasses. It supports stable connection management, data communication, real-time audio/video, and scene customization, and pairs with the on-device CXR-S SDK. The SDK is not published on the public developer site; to request access to the CXR-M SDK, its documentation, and technical support, contact business partnerships at `Glasses.BD@rokid.com`.

See: [CXR-M SDK introduction](../../cxr-m/intro.md)

### On-device Application Development

#### CXR-S SDK

The CXR-S SDK is the on-device (glasses-side) developer toolkit that runs on YodaOS-Sprite. It is focused on helping developers build standalone applications that run directly on Rokid Glasses. In addition to exposing the on-device data channel, it can establish bidirectional communication with the mobile-side CXR-M SDK, with support for custom protocols and command transport. With CXR-S, developers can reach deeper into the device's hardware, unlock its full potential, and deliver low-latency intelligent experiences.

See: [CXR-S SDK overview](../../cxr-s/brief.md)

## FAQ

**Q: What are the main capabilities of the CXR-M SDK?**

**A:** A mobile application built with the CXR-M SDK can communicate with Rokid Glasses, access audio and video from the device, and customize the implementation of existing on-device scenes. When paired with the CXR-S SDK, it can also exchange custom commands.

**Q: Which devices does the CXR-M SDK currently support?**

**A:** At present, the CXR-M SDK is provided only for Android mobile devices.

**Q: What are the main capabilities of the CXR-S SDK?**

**A:** The CXR-S SDK exposes the YodaOS-Sprite data channel and forwards custom commands through that channel to the CXR-M SDK on the mobile side.

**Q: When building a mobile-only application with just the CXR-M SDK, do I need to enable developer mode on Rokid Glasses?**

**A:** No — it is not required.

For additional questions, see the Rokid Developer Forum. <!-- TODO: source did not provide a URL for the forum. Add when known. -->

## Notes

- **Source**: https://ar.rokid.com/sprite — the page is a single-page application that loads developer content from `https://developerdoc.rokid.com/sprite?lang=zh` in an iframe. Captured on 2026-05-29 via `firecrawl-interact` (rendered DOM, Chinese locale).
- **Scope**: This document covers the **YodaOS-Sprite** tab only — the OS that runs on Rokid Glasses and Rokid AI Glasses. A separate "YodaOS-Master" tab exists on the upstream page but is out of scope for this repository.
- **Known TODO links**:
  - "View specs" button under *About YodaOS-Sprite* — destination not captured.
  - Rokid Developer Forum — URL not provided by source.
