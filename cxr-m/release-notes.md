# CXR-M SDK Release Notes

> Source: https://ar.rokid.com/sdk (CXR-M section, Chinese, fetched 2026-05-29)

The CXR-M SDK is a developer toolkit for mobile apps that pair with Rokid Glasses. It supports stable connectivity, data communication, real-time audio and video, and custom scenes, and is meant to be used together with the on-device CXR-S SDK. **The CXR-M SDK is not publicly distributed on the Rokid developer site.** To request the SDK, documentation, or technical support, contact business partnerships at `Glasses.BD@rokid.com`.

Latest release: **v1.1.0** (2026-04-01).

## Changelog

### v1.1.0 — 2026-04-01

1. Improved the voice-stream listener API.
2. Photo capture now returns the raw image straight from the on-glasses Camera.

### v1.0.9 — 2026-03-01

1. Real-time audio streaming.
2. Improved voice control.
3. Added an alternate media-file transfer scheme — the host app can now bring up the Wi-Fi P2P connection itself.
4. Added documentation for the CXR-S-side data API.

### v1.0.4 — 2025-12-25

1. Added SN-based access control.
2. Added a standalone photo-capture API that returns the captured image data.
3. Fixed an OOM issue when syncing large files over Wi-Fi P2P.
4. Fixed the issue where AI scenes exited after 6 seconds.
5. Added an API to subscribe to glasses screen-on / screen-off state updates.
6. Added an API to put the glasses display to sleep on demand.
7. Added an API to switch the microphone pickup field.
8. Added an API to set the on-device TTS voice.
9. Added an API to set the on-device TTS speech rate.

### v1.0.1 — 2025-08-25

1. Connect to the glasses.
2. Read and set glasses screen brightness.
3. Custom AI assistant scenes.
4. Custom translation scenes.
5. Custom teleprompter scenes.
6. Capture audio from the on-device microphones.
7. Configure and capture photos.
8. Configure video recording parameters.
9. Launch on-device scenes (AI assistant, translation, etc.) from the mobile app.
