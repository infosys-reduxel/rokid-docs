# Rokid Apps Overview

Summary of all Rokid-specific apps found across partitions on the YodaOS device.

Chinese payment/commerce apps (Alipay, AntPay, JdPay) are excluded from this page. See [chinese-apps.md](chinese-apps.md) for those.

## Rokid Apps

| App Name               | Package                              | Partition          | Description                                                                                              | Detailed Doc |
|------------------------|--------------------------------------|--------------------|----------------------------------------------------------------------------------------------------------|--------------|
| CXRService             | com.rokid.cxrservice                 | system/app         | Core CXR glasses-side service bridging Bluetooth connectivity; persistent, direct-boot aware, system UID  | -            |
| RokidSysConfig         | com.rokid.sysconfig                  | system/priv-app    | System configuration service; persistent, direct-boot aware, exposes ConfigService                       | -            |
| RokidOtaUpgrade        | com.rokid.glass.ota                  | product/app        | OTA firmware update client with download, verification, and reboot management                            | -            |
| RokidScreenRecord      | com.rokid.os.master.screenstream     | product/app        | Screen recording/streaming service triggered via broadcast intents (SCREENRECORD_ON/OFF)                 | -            |
| RokidSpriteAssistServer| com.rokid.os.sprite.assistserver     | product/app        | Central system service hub: Bluetooth, WiFi, payment, TTS, media, camera, web server, instruction engine | -            |
| RokidSpriteLauncher    | com.rokid.os.sprite.launcher         | product/app        | Main home launcher with pages for camera, gallery, audio, chat, translate, navigation, music, settings   | -            |

## App Details

### CXRService

- **Package**: `com.rokid.cxrservice`
- **Location**: `system/app/CXRService`
- **UID**: `android.uid.system` (shared)
- **Persistent**: Yes
- **Direct-boot aware**: Yes
- **Key components**:
  - `CXRService` - Bound service with intent action `com.rokid.cxrservice.CXRService`
- **Permissions**: BLUETOOTH_CONNECT
- **Notes**: This is the glasses-side CXR service that handles the communication bridge between the glasses and companion devices. Uses cleartext traffic. Compiled against SDK 32 (Android 12).

### RokidSysConfig

- **Package**: `com.rokid.sysconfig`
- **Location**: `system/priv-app/RokidSysConfig`
- **UID**: `android.uid.system` (shared)
- **Persistent**: Yes
- **Direct-boot aware**: Yes
- **Key components**:
  - `ConfigService` - Exported bound service for system configuration
- **Notes**: Minimal app that exposes system configuration to other Rokid components. Compiled against SDK 32 (Android 12). Uses non-SDK APIs.

### RokidOtaUpgrade

- **Package**: `com.rokid.glass.ota`
- **Location**: `product/app/RokidOtaUpgrade`
- **UID**: `android.uid.system` (shared)
- **Persistent**: Yes
- **Key components**:
  - `OtaService` - Background OTA management service
  - `CheckService` - OTA update check service
  - `DownloadService` - OTA download service
  - Multiple activities for upgrade flow: NeedUpgradeActivity, UpgradePercentActivity, UpgradeFailedActivity, NeedRebootActivity, NetCheckingActivity, NetCheckErrorActivity, AllowMobileDownloadActivity, DeviceConditionCheckActivity, UpdatePowerErrorActivity, UnNeedUpgradeActivity
- **Permissions**: Internet, storage, WiFi, location, network state, wake lock, system alert window
- **Notes**: Full OTA pipeline with network checking, download progress, error handling, and reboot prompts. Landscape orientation enforced. Uses Rokid Glass UI library for auto-sizing. Compiled against SDK 33 (Android 13).

### RokidScreenRecord

- **Package**: `com.rokid.os.master.screenstream`
- **Location**: `product/app/RokidScreenRecord`
- **UID**: `android.uid.system` (shared)
- **Key components**:
  - `ScreenRecordService` - Foreground service for screen recording
  - `ScreenRecordReceiver` - Broadcast receiver for intents:
    - `com.rokid.yodaos.action.SCREENRECORD_ON`
    - `com.rokid.yodaos.action.SCREENRECORD_OFF`
  - `BootCompleteReceiver` - Starts on boot
- **Permissions**: WiFi, network state, record audio, media projection, system alert window, internet
- **Notes**: Screen capture and streaming service. Triggered via broadcast intents from other Rokid components. Compiled against SDK 33 (Android 13).

### RokidSpriteAssistServer

- **Package**: `com.rokid.os.sprite.assistserver`
- **Location**: `product/app/RokidSpriteAssistServer`
- **UID**: `android.uid.system` (shared)
- **Persistent**: Yes
- **Key components**:
  - `MasterAssistService` - Main orchestration service
  - `SpriteWifiService` - WiFi management service
  - `InstructService` - Voice/gesture instruction processing service
  - `PaymentService` - Payment integration service (bridges to Alipay, JD, WeChat Pay)
  - `WebServerService` - Built-in HTTP/FTP web server for file access
  - `NsdService` - Network Service Discovery for FTP
  - `RokidBluetoothService` - Bluetooth management (stopWithTask=false, runs independently)
  - `SystemFuncService` - System-level function service
  - `SpriteMediaService` - Media playback/control service
  - `TtsService` - Text-to-speech engine (runs in separate `:tts` process)
  - `SDKService` (com.jd.jr.joygoclient) - JD Finance SDK service (bundled)
  - `MultiSpProvider` - Shared preferences content provider
  - `BootReceiver` - Starts services on boot
- **Permissions**: Extensive -- Bluetooth (all), camera, audio, WiFi, phone state, storage, USB, inject events, settings, sensors (accelerometer, gyroscope), network tethering, scheduling
- **Queries**: `com.tencent.glasswxpay.glassapp` (WeChat Pay), `com.iap.mobile.ar_pay` (AntPay)
- **Native libraries**: Requires `libOpenCL.so`
- **Notes**: This is the central system service for the Rokid Sprite platform. It manages nearly all device functionality: Bluetooth pairing, WiFi provisioning, payment integration, TTS, media control, camera operations, a built-in web server, and instruction processing. Uses Room database, CameraX, and Rokid Glass UI libraries. Compiled against SDK 34 (Android 14).

### RokidSpriteLauncher

- **Package**: `com.rokid.os.sprite.launcher`
- **Location**: `product/app/RokidSpriteLauncher`
- **UID**: `android.uid.system` (shared)
- **Key components**:
  - `SpriteMainActivity` - Home screen (MAIN/LAUNCHER/HOME intent categories, also LEANBACK_LAUNCHER)
  - Feature pages:
    - `CameraPageActivity` - Camera capture (separate task affinity)
    - `StorageImageShowActivity` - Photo gallery viewer
    - `AudioPageActivity` - Audio recording/playback
    - `ChatPageActivity` - AI chat interface
    - `TranslatePageActivity` - Real-time translation
    - `WordTipsPageActivity` - Word/vocabulary tips overlay
    - `NavigationPageActivity` - Navigation (domestic)
    - `NavigationOverseaPageActivity` - Navigation (overseas)
    - `MusicPageActivity` - Music player
    - `PaymentPageActivity` - Payment (links to payment apps)
    - `CaexpoPageActivity` - CAEXPO (China-ASEAN Expo) feature page
  - Settings pages:
    - `SettingPageActivity` - Main settings
    - `SettingSystemInfoActivity` - System info
    - `SettingSoundEffectsActivity` - Sound effects
    - `SettingBluetoothActivity` - Bluetooth settings
    - `BluetoothRepairActivity` - Bluetooth repair/troubleshooting
    - `SettingBrightnessActivity` - Display brightness
    - `SettingVolumeActivity` - Volume control
    - `SystemResetActivity` - Factory reset
  - `TestUiActivity` - UI testing (development)
- **Permissions**: Camera, audio, storage, Bluetooth, WiFi, install/delete packages, wake lock, phone state, shortcuts, force stop packages
- **Notes**: The main user-facing launcher application. Provides the home screen and all major feature entry points. Uses CameraX for camera features, Room database for persistence. No touch required (leanback launcher). Compiled against SDK 34 (Android 14).

## Rokid Resource Overlays

The following RROs in the `product/overlay` partition customize AOSP framework and WiFi behavior for Rokid devices:

| Overlay Name              | Package                                  | Target                |
|---------------------------|------------------------------------------|-----------------------|
| FrameworksResCommon_Rkd   | android.overlay.common.rkd              | Android framework     |
| WifiResCommon_Rkd         | com.android.wifi.resources.overlay.rkd  | WiFi service          |
| WifiStackResCommon_Rkd    | com.android.networkstack.overlay.rkd    | Network stack / WiFi  |
