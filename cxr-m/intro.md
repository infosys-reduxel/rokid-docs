
The Rokid CXR-M SDK is a developer toolkit provided by Rokid for building mobile applications that work with Rokid Glasses. With the CXR-M SDK, developers can establish a stable connection between the phone and glasses, enabling data communication, real-time audio/video retrieval, and scene customization. It is suited to applications that need phone-side UI interaction, remote control, or complex collaborative features with the glasses. The SDK currently provides an Android version only.

See [release-notes.md](release-notes.md) for the full version history. The current **publicly documented** release is **v1.1.0** (2026-04-01); Maven additionally publishes **v1.2.2** (2026-06-09) but Rokid has not released a changelog for the 1.1.x / 1.2.x line as of 2026-06-10.

## CXR SDK and Glasses Architecture

![CXR-M Architecture](https://ota.rokidcdn.com/toB/Document/CXR/CXR-M%20Final.png)

## Rokid Glasses Device Connection and Management

Developers can use the CXR-M SDK to connect to Rokid Glasses, retrieve device status, and manage the device.

## Rokid Glasses Custom Scene Interaction

Developers can use the CXR-M SDK to quickly integrate the scene interaction flows defined by YodaOS-Sprite and develop custom capabilities based on those flows, including:

- Custom AI assistant

## Rokid Glasses Assist Service

Developers can use the CXR-M SDK to efficiently utilise the services within Rokid Assist Service, including:

- Audio recording
- Photo capture
- Video recording
- File transfer (Feichuan / 飞传)

> **Note:** Rokid Assist Service is a series of services provided by Rokid based on YodaOS-Sprite. If any of these services are disabled they cannot be used through the CXR-M SDK.

> Source: `https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/9d9dea4799ca4dd2a1176fedb075b6f2.html` (v1.0.1, scraped 2026-06-11)
