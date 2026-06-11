# Design Specifications

> Source: <https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/2786298057084a82b170bf725aef6b5d.html?documentId=d0db23be629d4b09be197bd0dd931298>
> (Chinese, attempted 2026-06-11)
>
> **Retrieval note:** The upstream page is a fully JavaScript-rendered single-page application (SPA)
> hosted on Alibaba OSS. Static fetch returns only the SPA shell ("Rokid AR Platform"); no article
> content could be extracted. The CDN domain (`static.rokidcdn.com`) and all known Rokid API
> domains are inaccessible from the build environment. The confirmed content below is derived from
> cross-references within this repository (see [Bare-Metal Development Guide](../cxr-baremetal/development-guide.md))
> and the project glossary. Sections that could not be retrieved are marked with TODO.
>
> **SDK version this document was derived from:** CXR-S SDK `com.rokid.cxr:cxr-service-bridge:1.0`
> (cross-referenced with bare-metal guide v0.0.1, 2026-03-01)

This chapter describes the UI layout constraints and design guidelines that apply when building
on-device (glasses-side) applications with the CXR-S SDK on YodaOS-Sprite.

---

## Screen Size

The Rokid Glasses display is designed for a **480 × 640 pixel** viewport. All UI layouts must be
sized and positioned to fit within this viewport.

| Property | Value |
|----------|-------|
| **Display viewport** | 480 × 640 px |
| **OS base** | Android 12 (Go edition, low-RAM) |
| **Display** | JBD JBD4020 Micro-LED (right eye) |

> Cross-reference: the viewport dimensions are also noted in the [Bare-Metal Development Guide](../cxr-baremetal/development-guide.md).

<!-- TODO: Retrieve density/dpi, safe-area insets, recommended font sizes, and color-contrast guidelines
     from the upstream source once the SPA is accessible.
     Upstream: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/2786298057084a82b170bf725aef6b5d.html?documentId=d0db23be629d4b09be197bd0dd931298 -->

---

## Development Constraints

Because YodaOS-Sprite is built on Android Go (a low-RAM-optimized variant of Android 12), UI
design must respect both Android Go constraints and the platform-reserved interactions listed below.

### Android Go constraints

- Target the Android Go memory budget: avoid holding large bitmaps or view hierarchies in memory.
- Use hardware-accelerated drawing (enabled by default on API 32) and avoid software canvas paths
  for real-time overlays.

### System-reserved interactions

The following interactions are reserved by YodaOS-Sprite and **cannot be overridden** by
third-party on-device apps:

| Gesture / button | Action |
|------------------|--------|
| Long-press the right-temple touchpad | Launches the on-device Rokid AI module |
| Double-click the right-temple side button | Triggers the back action (`KEYCODE_BACK`) |
| Click the top button on the right temple | Captures a photo |
| Long-press the top button on the right temple | Starts video recording |
| Specific wake words | Activate dedicated platform functions (details TBD by Rokid) |

All other button and touchpad events are delivered to apps as system broadcasts. See
[Button Broadcasts (Bare-Metal)](../cxr-baremetal/key-broadcasts.md) for the full `KeyType`
enumeration and ordered-broadcast interception pattern.

<!-- TODO: The upstream "Design Specifications" page likely contains additional UI design rules
     (typography guidelines, component sizes, touch-target sizing, color palette, safe-area
     margins, and animation guidelines). These could not be retrieved because the page is
     JS-rendered. Refresh this section once the SPA content is accessible.
     Upstream: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/2786298057084a82b170bf725aef6b5d.html?documentId=d0db23be629d4b09be197bd0dd931298 -->

---

## Related Documents

| Document | Description |
|----------|-------------|
| [brief.md](brief.md) | CXR-S SDK overview |
| [development-environment.md](development-environment.md) | Dev environment setup |
| [sdk-import.md](sdk-import.md) | SDK import guide |
| [manage-device-connection.md](manage-device-connection.md) | Connection management |
| [message-subscription.md](message-subscription.md) | Receiving messages from mobile |
| [message-sending.md](message-sending.md) | Sending messages to mobile |
| [data-structure.md](data-structure.md) | Caps serialization format |
| [../cxr-baremetal/development-guide.md](../cxr-baremetal/development-guide.md) | Bare-metal Android dev guide (480 × 640 px viewport, reserved interactions) |
| [../cxr-baremetal/key-broadcasts.md](../cxr-baremetal/key-broadcasts.md) | Button and touchpad broadcast Intents |
