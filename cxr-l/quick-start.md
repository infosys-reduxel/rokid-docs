# CXR-L SDK Quick Start

> Source: <https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html?documentId=3878a4cd92e649e99a1e490ea034e603> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.4**

See [Introduction](intro.md) for SDK positioning and capability prerequisites.

## Environment prerequisites

- Real device or Bluetooth-debuggable environment, paired with target glasses.
- **Rokid AI App** (mainland, version **≥ 1.9.0**) or **Hi Rokid** (overseas) installed.
- Understand capability prerequisites in the [Introduction](intro.md).

## Get the phone-side samples

### Android (RenewCXRLSample, v1.0.4)

- **Zip package**: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/CXRLSample.zip`

### iOS (ios_cxr_l_sample, v1.0.4)

- **Zip package**: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.4/iOS/ios_cxr_l_sample.zip`

### Glasses side (CXRSWithCXRLSample)

- **Zip package**: `https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.3/cxrssample.zip`

Extract and open the `cxrswithcxrl` project in Android Studio; sync Gradle (`https://maven.rokid.com/repository/maven-public/`). Package `com.rokid.cxrswithcxrl` matches RenewCXRLSample `CONSTANT`.

## Minimal verification path (Android)

1. Open RenewCXRLSample in Android Studio; sync Gradle (`https://maven.rokid.com/repository/maven-public/`).
2. Confirm SDK dependency is `com.rokid.cxr:client-l:1.0.4` in `app/build.gradle.kts`.
3. Build and install; confirm **Rokid AI App ≥ 1.9.0** (mainland) or Hi Rokid (overseas).
4. Complete authorization on the home screen; obtain `token`.
5. Choose **CustomView** or **CustomApp**; enter `CxrSessionActivity`.
6. Wait for link ready: `onCXRLConnected(true)` **and** `onGlassBtConnected(true)` (both session types).
7. **Complete scene building**:

   - **CustomView**: after link ready, `customViewSetIcons` (if needed) and `customViewOpen` → `onCustomViewOpened`
   - **CustomApp**: APK installed (with storage permissions), `appStart` → `onOpenAppResult(true)`
8. Enter **Audio** / **Photo** from hub; **Custom Command** from CustomApp only.
9. Enter **Device Control** from hub; drag sliders to adjust brightness/volume and verify glasses-side response.

**Important:** Use photo, audio, and custom commands only **after scene building**.

## CustomApp joint-debug path (phone + glasses)

Under a **CUSTOMAPP** session, verify custom commands and key reporting:

1. Install CXRSWithCXRLSample on glasses (or via `appUploadAndInstall`).
2. Install RenewCXRLSample on the phone; complete auth and obtain `token`.
3. Choose **CustomApp** → Session Hub; wait for link ready (CXR + Bluetooth).
4. Complete scene building: install/start APK from Hub, receive `onOpenAppResult(true)`; glasses `MainViewModel` runs `subscribe("rk_custom_client", …)`.
5. Open **Custom Commands**: phone `sendCustomCmd` ↔ glasses `sendMessage`.
6. Press leg keys, touchpad, or back on glasses; phone should receive `rk_custom_key` payloads.

| Constant | Value |
| --- | --- |
| `APP_PACKAGE_NAME` | `com.rokid.cxrswithcxrl` |
| `MAIN_PAGE` | `.activities.main.MainActivity` |
| `appStart` argument | `"${APP_PACKAGE_NAME}${MAIN_PAGE}"` |

## Minimal verification path (iOS)

1. Configure Pod, `Info.plist`, and URL callbacks per the iOS SDK Integration chapter.
2. Forward `CxrClient.shared.handleOpenURL` in `AppDelegate` / `SceneDelegate`.
3. Call `client.auth.authenticate`.
4. Establish link and complete scene building on glasses.
5. Verify audio, photo, custom command per dedicated chapters.
6. Enter **Device Control** from hub; drag sliders to adjust brightness/volume and verify glasses-side response.

**Important:** Same gating as Android — capabilities require scene building, not link-only success.

## Appendix: RenewCXRLSample modules

| Module | Path |
| --- | --- |
| Home / auth | `activities/main/` |
| Session hub | `activities/session/SessionHubViewModel.kt` |
| Connection | `link/CxrLinkConnectionHub.kt`, `utils/CxrSessionGate.kt` |
| Capabilities | `activities/audio/`, `photo/`, `customCMD/` |
| Device control | `activities/device/DeviceControlViewModel.kt` |
| Global link | `app/CXRLApplication.kt` |
| APK Install Access | `utils/ApkInstallAccess.kt` |
| Glasses demo | CXRSWithCXRLSample — `activities/main/`, `receiver/KeyReceiver.kt` |

See also [Introduction](intro.md) and [Terms and Abbreviations](terms-and-abbreviations.md).
