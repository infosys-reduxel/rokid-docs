# CXR-L SDK Terms and Abbreviations

> Source: <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html?documentId=ff91652f16c54420ae81da192350782d> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.4**

## Platforms and products

| Term | Meaning |
| --- | --- |
| Rokid AI App | Required app (mainland); package `com.rokid.sprite.aiapp`; requires **≥ 1.9.0** when using **client-l:1.0.4** |
| Hi Rokid | Required app (overseas) |
| Glasses | Rokid glasses device |
| CXR-L SDK | Android: `com.rokid.cxr:client-l:1.0.4`; iOS: CocoaPods `RGCxrClient` (v1.0.4) |
| CXR-S SDK | Glasses-side Android SDK, paired with CXR-L; Maven artifact `cxr-service-bridge`; see [CXR-S docs](../cxr-s/) |
| CXRServiceBridge | Glasses CXR-S bridge entry, `com.rokid.cxr.CXRServiceBridge`; `subscribe` / `sendMessage` pair with phone `sendCustomCmd` / `ICustomCmdCbk` |
| cxr-service-bridge | Maven coordinate prefix for CXR-S, e.g. `com.rokid.cxr:cxr-service-bridge` (version per Sample / release notes) |

## Authentication and connection

| Term | Meaning |
| --- | --- |
| token | Auth token for `CXRLink.connect(token)` |
| CXRLink | Android SDK entry, `com.rokid.cxr.link.CXRLink` |
| CxrClient | iOS SDK entry, `CxrClient.shared` |
| ICXRLinkCbk | Android link callbacks |

## Scenes and sessions

| Term | Meaning |
| --- | --- |
| Scene building | Glasses in working state: Custom View opened or Custom App running |
| Link ready | Transport available: `onCXRLConnected`; CustomApp also needs Bluetooth |
| Scene ready | CustomView: `onCustomViewOpened`; CustomApp: app opened on glasses |

## Session types

| Term | Android enum | Notes |
| --- | --- | --- |
| CustomView session | `CxrDefs.CXRSessionType.CUSTOMVIEW` | JSON layout delivery |
| CustomApp session | `CxrDefs.CXRSessionType.CUSTOMAPP` | Remote APK control; requires `packageName` |

iOS uses initialization modes (customView / customApp) — see `RGCxrClient`.

## Capability abbreviations

| Abbrev | Capability |
| --- | --- |
| CustomView | Glasses custom UI (JSON + Base64 icons) |
| CustomApp | Remote glasses Android app control |
| CustomCMD | Binary messages via `Caps` + `sendCustomCmd` |
| Caps | Rokid-defined protocol for custom message communication between phone-side CXR-L and glasses-side CXR-S |
| Audio | `startAudioStream` / `stopAudioStream` |
| Photo | `takePhoto` + image callback |
| Device Control | Glasses brightness (0…15) and volume (0…15) set/query; Android: `setGlassBrightness`/`setGlassVolume`, iOS: `setBrightness`/`setVolume` |

## Caps (Android)

`com.rokid.cxr.Caps` — serializes custom command payloads. See also [cxr-s/data-structure.md](../cxr-s/data-structure.md) for the shared Caps wire format used across the CXR SDK suite.

## CustomView JSON terms

| Term | Meaning |
| --- | --- |
| View tree | Recursive JSON for `customViewOpen`: `type` + `props` + `children` |
| props.id | Unique node id for `customViewUpdate` |
| Icon name | Key linking `customViewSetIcons` and `ImageView.props.name` |

See also [Introduction](intro.md) and [Quick Start](quick-start.md).
