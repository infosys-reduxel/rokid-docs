# CXR-L SDK Release Notes

_Source: https://ar.rokid.com/sdk (Chinese, fetched 2026-05-28; rechecked 2026-05-29)._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.0.2 — published 2026-05-19 (undocumented)

`com.rokid.cxr:client-l:1.0.2` was uploaded to Maven on 2026-05-19 and finalized in the repository index on 2026-05-28, but as of 2026-05-29 Rokid has not yet published a changelog for it on the developer portal or on its `custom.rokid.com` doc site. The POM declares the same dependency set as 1.0.1 (`cxr-service-bridge 1.0-20260212.103714-88`, `kotlin-stdlib 2.1.0`, `gson 2.10.1`). Features below remain those of 1.0.1 until Rokid publishes the 1.0.2 changelog.

<!-- TODO: append the v1.0.2 feature list once it is published upstream. -->

## v1.0.1 — 2026-05-07

1. Initial SDK release.
2. Support for obtaining authorization from the Rokid AI app.
3. Support for creating on-device custom View scenes.
4. Support for creating on-device custom app scenes.
5. Support for accessing on-device audio.
6. Support for capturing photos through the glasses.
7. Support for custom-command exchange with on-device custom apps.
