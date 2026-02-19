# RokidSpriteAssistServer

Package: `com.rokid.os.sprite.assistserver`
Version: `0.2.4` (code `2411`)
Shared UID: `android.uid.system` (runs as system app)
Compiled SDK: 34 (Android 14)
Application class: `com.rokid.os.sprite.assist.MainApplication`
Persistent: true (auto-restarts on kill)

The central system service app for Rokid AR glasses. Acts as the orchestration layer between the glasses hardware, the launcher UI (`com.rokid.os.sprite.launcher`), the companion phone app (via Bluetooth), and all third-party/first-party apps. Manages voice commands, camera/media, Bluetooth phone bridging, payments, file transfer, TTS, and system-level functions.

Written primarily in Kotlin, with some Java components (AIDL interfaces, payment SDKs, Bluetooth management). Uses OkHttp for network, Gson/FastJSON for serialization, AndServer for HTTP serving, and ONNX models for on-device TTS.

---

## Architecture Overview

### Startup Sequence

`MainApplication.onCreate()` is the entry point. On the main process only, it:

1. Initializes system properties (TTS flags, sync time)
2. Calls `initBasicMainApp()` -- sets up USB mic listener, sound player, shutdown receiver, storage manager
3. Calls `initByCreate()` -- placeholder for future init
4. Starts all services in order:
   - TTS Service (separate `:tts` process, only if local TTS enabled)
   - SystemFuncService
   - MasterAssistService
   - InstructService
   - WebServerService
   - SpriteMediaService
   - PaymentService
   - RokidBluetoothService

All services register themselves back into `BasicApplication` via setter methods (e.g., `setInstructServer(this)`, `setBluetoothServer(this)`), creating a centralized service registry that any component can access via `BasicApplication.getBasicInstance().getXxxServer()`.

### Inter-Service Communication

Services communicate through several mechanisms:

1. **Direct method calls** via the `BasicApplication` service registry -- the most common pattern
2. **AIDL Binder IPC** for cross-process communication (MasterAssistService, TtsService)
3. **Broadcast intents** for system-level events and external control
4. **Handler message queues** within each service for thread-safe async operations

### Client-Server Model

External apps (like the launcher) bind to `MasterAssistService` via AIDL. The protocol:

1. Client calls `registerClient(packageName, IAssistClient)` on the `IAssistServer` binder
2. Server tracks clients via `ClientManager`
3. Server pushes messages to clients via `IAssistClient.onMessageReceive(AssistMessage)`
4. Client sends commands via `IAssistServer.controlMsgJson(packageName, jsonCommand)`
5. Client can also submit QR code bitmaps for scanning via `scanQrCodeBitmap()` / `scanQrCodeBmList()`

---

## Manifest Components

### Permissions

The app requests extensive system-level permissions:

**Storage**: `WRITE_EXTERNAL_STORAGE`, `READ_EXTERNAL_STORAGE`, `MANAGE_EXTERNAL_STORAGE`, `READ_MEDIA_IMAGES`
**Network**: `INTERNET`, `ACCESS_NETWORK_STATE`, `ACCESS_WIFI_STATE`, `CHANGE_WIFI_STATE`, `CHANGE_WIFI_MULTICAST_STATE`, `CHANGE_NETWORK_STATE`, `TETHER_PRIVILEGED`
**Bluetooth**: `BLUETOOTH`, `BLUETOOTH_ADMIN`, `BLUETOOTH_CONNECT`, `BLUETOOTH_PRIVILEGED`, `BLUETOOTH_ADVERTISE`, `BLUETOOTH_SCAN`
**Phone**: `READ_PHONE_STATE`, `READ_PRIVILEGED_PHONE_STATE`, `MODIFY_PHONE_STATE`, `ANSWER_PHONE_CALLS`
**Camera/Audio**: `CAMERA`, `RECORD_AUDIO`, `MEDIA_CONTENT_CONTROL`
**System**: `SYSTEM_ALERT_WINDOW`, `INJECT_EVENTS`, `WRITE_SETTINGS`, `READ_LOGS`, `WAKE_LOCK`, `SCHEDULE_EXACT_ALARM`, `RECEIVE_BOOT_COMPLETED`
**Location**: `ACCESS_COARSE_LOCATION` (max SDK 32), `ACCESS_FINE_LOCATION` (max SDK 32), `NEARBY_WIFI_DEVICES`
**Custom**: `glass.wxpay.service.permission`

**Required hardware features**: `android.hardware.sensor.accelerometer`, `android.hardware.sensor.gyroscope`

**Native library requirement**: `libOpenCL.so` (required)

### Custom Permission

`com.rokid.os.sprite.assistserver.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` (signature-level) -- used to protect dynamically registered receivers from external access.

### Queried Packages

- `com.tencent.glasswxpay.glassapp` (WeChat Pay glass app)
- `com.iap.mobile.ar_pay` (AR payment app)

---

## Services

### 1. MasterAssistService

**Class**: `com.rokid.os.sprite.assist.MasterAssistService`
**Exported**: true
**Intent filter action**: `com.rokid.os.sprite.assist.MasterAssistService`
**Implements**: `IAssistUnityServer`

The central command router. This is the primary bound service that other apps on the glasses interact with. It exposes the `IAssistServer` AIDL interface for IPC.

**AIDL Interface -- IAssistServer** (descriptor: `com.rokid.os.sprite.assist.server.IAssistServer`):

| Method | Description |
|--------|-------------|
| `registerClient(String packageName, IAssistClient client)` | Register as a client to receive messages |
| `unRegisterClient(String packageName)` | Unregister client |
| `controlMsgJson(String packageName, String json)` | Send a command as JSON |
| `scanQrCodeBitmap(Bitmap bitmap)` | Submit a single bitmap for QR code scanning |
| `scanQrCodeBmList(List<Bitmap> list)` | Submit multiple bitmaps for QR code scanning |

**AIDL Interface -- IAssistClient** (descriptor: `com.rokid.os.sprite.assist.client.IAssistClient`):

| Method | Description |
|--------|-------------|
| `onRegisterResult(RegisterResult result)` | Registration confirmation callback |
| `onMessageReceive(AssistMessage message)` | Receive a message from the server; returns boolean |
| `onDataReceive(String type, String key, byte[] data)` | Receive binary data from the server |

**Binding from external app**:
```java
Intent intent = new Intent("com.rokid.os.sprite.assist.MasterAssistService");
intent.setPackage("com.rokid.os.sprite.assistserver");
bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
```

**Command routing**: When a client sends `controlMsgJson(pkg, json)`, the JSON is parsed for a `cmd` field. The handler dispatches to the appropriate subsystem based on the command type. Supported server commands (from `AssistServerCmd`):

| Command Constant | String Value | Routes To |
|-----------------|--------------|-----------|
| `SERVER_CMD_REPORT_POINT_JSON` | `cmd_report_point_json` | BuryPointServer |
| `SERVER_CMD_REPORT_POINT_OBJ` | `cmd_report_point_obj` | BuryPointServer |
| `SERVER_CMD_ANALYSIS_QRCODE` | `cmd_analysis_qrcode` | QrCodeServer |
| `SERVER_CMD_CLOSE_ANALYSIS_QRCODE_UI` | `cmd_close_analysis_qrcode_ui` | QrCodeServer |
| `SERVER_CMD_CONTROL_REMOTE_WEB_SERVER` | `cmd_control_remote_web_server` | RemoteWebServer |
| `SERVER_CMD_GET_REMOTE_WEB_SERVER_STATE` | `cmd_get_remote_web_server_state` | RemoteWebServer |
| `SERVER_CMD_TAKE_PICTURE` | `cmd_take_picture` | SpriteMediaServer |
| `SERVER_CMD_START_VIDEO_RECORD` | `cmd_start_video_record` | SpriteMediaServer |
| `SERVER_CMD_STOP_VIDEO_RECORD` | `cmd_stop_video_record` | SpriteMediaServer |
| `SERVER_CMD_START_AUDIO_RECORD` | `cmd_start_audio_record` | SpriteMediaServer |
| `SERVER_CMD_STOP_AUDIO_RECORD` | `cmd_stop_audio_record` | SpriteMediaServer |
| `SERVER_CMD_PHONE_ACCEPT` | `cmd_phone_accept` | BluetoothServer |
| `SERVER_CMD_PHONE_REJECT` | `cmd_phone_reject` | BluetoothServer |
| `SERVER_CMD_PHONE_TERMINATE` | `cmd_phone_terminate` | BluetoothServer |
| `SERVER_CMD_PHONE_GATT_SEND_DATA` | `cmd_phone_gatt_send_data` | BluetoothServer |
| `SERVER_CMD_PHONE_AI_MODEL_OPERATE` | `cmd_phone_ai_model_operate` | BluetoothServer |
| `SERVER_CMD_PHONE_SYSTEM_APP_MODULE` | `cmd_phone_system_app_module` | BluetoothServer |
| `SERVER_CMD_SCENE_STATUS_CHANGE` | `CMD_SCENE_STATUS_CHANGE` | InstructServer |
| `SERVER_CMD_GET_BLE_STATUS` | `cmd_get_ble_status` | BluetoothServer |
| `SERVER_CMD_SETTING_MUSIC_STATUS` | `cmd_setting_music_status` | BluetoothServer |
| `SERVER_CMD_RESET_BLE_STATUS` | `cmd_reset_ble_status` | BluetoothServer |
| `SERVER_CMD_START_AUDIO_STREAM` | `cmd_start_audio_stream` | BluetoothServer |
| `SERVER_CMD_PLAY_TTS` | `cmd_play_tts` | TtsUtil |
| `SERVER_CMD_KEY_EVENT` | `cmd_key_event` | BluetoothServer |
| `SERVER_CMD_TOGGLE_MUTE` | `cmd_toggle_mute` | BluetoothServer |
| `SERVER_CMD_GET_MEMO` | `cmd_get_memo` | SystemFuncServer |
| `SERVER_CMD_GET_JOURNEY` | `cmd_get_journey` | SystemFuncServer |
| `SERVER_CMD_LAUNCHER_MIX_SCENE` | `cmd_launcher_mix_scene` | BluetoothServer |
| `SERVER_CMD_LAUNCHER_EXIT_MIX_SCENE` | `cmd_launcher_exit_mix_scene` | BluetoothServer |

**Client response commands** (sent from server to clients via `AssistClientCmd`):

| Constant | String Value |
|----------|--------------|
| `CLIENT_RESULT_TAKE_PICTURE` | `result_take_picture` |
| `CLIENT_RESULT_VIDEO_RECORD` | `result_video_record` |
| `CLIENT_RESULT_AUDIO_RECORD` | `result_audio_record` |
| `CLIENT_CMD_BLUETOOTH_STATUS` | `cmd_bluetooth_status` |
| `CLIENT_CMD_BLUETOOTH_PHONE_STATUS` | `cmd_bluetooth_phone_status` |
| `CLIENT_CMD_BLUETOOTH_MUSIC_STATUS` | `cmd_bluetooth_music_status` |
| `CLIENT_CMD_BLUETOOTH_GATT_STATUS` | `cmd_bluetooth_gatt_status` |
| `CLIENT_CMD_BLUETOOTH_GATT_TRANSLATE_RESULT` | `cmd_bluetooth_gatt_translate_result` |
| `CLIENT_CMD_BLUETOOTH_GATT_NORMAL_RESULT` | `cmd_bluetooth_gatt_normal_result` |
| `CLIENT_CMD_BLUETOOTH_AI_RESULT` | `cmd_bluetooth_ai_result` |
| `CLIENT_CMD_BLUETOOTH_ARTC_RESULT` | `cmd_bluetooth_artc_result` |
| `CLIENT_CMD_NOTIFY_SCENE_STATUS` | `cmd_notify_scene_status` |
| `CLIENT_CMD_CONTROL_SCENE_OR_APP` | `cmd_control_scene_or_app` |
| `CLIENT_CMD_REMOTE_WEB_SERVER_STATE` | `cmd_remote_web_server_state` |
| `CLIENT_CMD_REMOTE_FILE_RECEIVE_START` | `cmd_remote_file_receive_start` |
| `CLIENT_CMD_REMOTE_FILE_RECEIVE_SUCCESS` | `cmd_remote_file_receive_success` |
| `CLIENT_CMD_SYSTEM_APP_SETTINGS` | `cmd_system_app_settings` |
| `CLIENT_CMD_SYSTEM_QUIT_APP` | `cmd_system_quit_app` |
| `CLIENT_CMD_GET_LAUNCHER_SCENE` | `cmd_get_launcher_scene` |
| `CLIENT_CMD_HANDLE_MSG_RESULT` | `cmd_handle_msg_result` |
| `CLIENT_CMD_HANDLE_DATA_POINT` | `cmd_handle_data_point` |
| `CLIENT_CMD_CLIENT_REGISTER_SUCCESS` | `cmd_client_register_success` |

**Additional capabilities**:
- Photo occlusion detection via `Occulusion.Detect(path)` native library -- returns float score, threshold < 0.1 means not occluded
- OTA management via `SpriteOtaManager` -- start/stop/check OTA updates
- Listens for network connectivity changes to trigger periodic sync (every 6 hours)

---

### 2. SpriteWifiService

**Class**: `com.rokid.os.sprite.wifi.SpriteWifiService`
**Exported**: true
**Intent filter action**: `com.rokid.os.sprite.assist.wifi.SpriteWifiService`
**Implements**: `ISpriteWifiServer`

A lightweight WiFi management service. Currently appears to be mostly a skeleton/placeholder. Runs its own `Assist_Wifi_Thread` handler thread. Registers itself into `BasicApplication.setSpriteWifiServer()`.

The `glassTakeOnChange()` callback is present but empty -- no WiFi-specific behavior on take-on/take-off yet.

---

### 3. InstructService

**Class**: `com.rokid.os.sprite.assist.instruct.InstructService`
**Exported**: true
**Intent filter action**: `com.rokid.os.sprite.assist.instruct.InstructService`
**Implements**: `IInstructServer`

The voice command and scene management brain. Handles both online (cloud NLU) and offline (on-device speech recognition) voice commands. Manages "scenes" -- distinct UI/functional states like video recording, translation, AI chat, payment, etc.

**Core sub-managers**:
- `OffLineManager` -- local speech recognition using `RtInstructSdk` (on-device wake word + command detection)
- `OnLineManager` -- cloud-based natural language understanding
- `SceneCoreManager` -- scene lifecycle management (open/close/status tracking)
- `MobileAssistantManager` -- voice recognition control relay to companion phone
- `FunctionKeyReceiver` -- physical button event handling

**Key methods**:
- `handleOnLineInstruct(json)` -- process cloud NLU result
- `handleOffLineInstruct(cmdId, frameStart, frameDuration)` -- process local speech command
- `updateSceneStatus(sceneKey, value, callType)` -- update a scene's running state
- `controlSceneAndPage(controlData)` -- open/close scenes and pages
- `cancelAllScene(ignoreList, byInstruct)` -- close all active scenes
- `playTtsContent(useLocalTts, content)` -- play TTS either locally or via phone relay
- `changeSpeechLanguage(language)` -- switch speech recognition language
- `changeSpeechScene(audioSceneId)` -- change audio scene for translation
- `startVoiceRecognition(callType)` / `stopVoiceRecognition(callType)` -- control voice recognition on phone
- `sendMsgToMobile(cmd, key, param)` -- send GATT message to phone via Bluetooth
- `sendSceneStatusToMobileOrLauncher(toMobile, toLauncher)` -- sync scene state

**Settings integration**: Listens for `settings_voice_control` changes via `RKSettingsManager`. When set to `"close"`, offline voice commands are ignored.

**Translation support**: Dedicated translation scene handling with bidirectional speech scene ID negotiation between glasses and phone app.

---

### 4. PaymentService

**Class**: `com.rokid.os.sprite.payment.PaymentService`
**Exported**: true
**Intent filter action**: `com.rokid.os.sprite.assist.payment.PaymentService`
**Implements**: `IPaymentServer`

Manages payment integrations on the glasses. Supports multiple payment providers:

- `AliPayCenter` -- Alipay integration
- `AntPayCenter` -- Ant Financial payment
- `WeiXinPayCenter` -- WeChat Pay integration
- `JdPayCenter` -- JD.com payment
- `CityGuideCenter` -- city transportation/guide card payments

Uses `PaymentProxyManager` for proxying payment requests between glasses and phone when needed (via CXR Bluetooth bridge with `PayCmdHelper`).

**Key methods**:
- `initAllSdk(initPayScene, initParam, scene)` -- initialize payment SDKs
- `finishPaymentByUser(scene, byUser, tts, error)` -- complete/cancel a payment
- `onPaymentSceneChange(scene, open, callType)` -- track payment scene state
- `requestDeviceProxy(reqCap, bytes)` / `responseDeviceProxy(resCap, bytes)` -- proxy payment data
- `onBluetoothConnectChange(connect, device)` -- handle Bluetooth connection changes for payment relay
- `glassTakeOnChange(takeOn)` -- react to glasses wear state during payments

---

### 5. WebServerService

**Class**: `com.rokid.glass.webserver.WebServerService`
**Exported**: true
**Intent filter action**: `com.rokid.glass.webserver.er.WebServerService`
**Implements**: `IRemoteWebServer`

HTTP file transfer server running on the glasses. Uses the [AndServer](https://github.com/yanzhenjie/AndServer) library. Allows remote file management between a computer/phone browser and the glasses over WiFi.

**Configuration** (from `WebServerConfig`):
- HTTP port: **8848**
- Transfer directory: `/sdcard/Download/`
- Spatial photos directory: `/sdcard/SpatialPhotos/`
- Settings toggle: `rokid_er_remote_transmission` in Android Global Settings (1 = enabled, 0 = disabled)

**Supported media types**: `image`, `video`, `3d_video`, `audio`, `document`, `apk`, `subtitle`, `playlist`

**Behavior**:
- Reads the global setting `rokid_er_remote_transmission` to decide if the server should be running
- Controlled externally via `WebCmdReceiver` broadcast receiver
- Uses `WakeLockUtil` to keep the device awake while serving files
- Cleans up `.tmp` files in the transfer directory on startup
- Notifies clients (via `AssistClientCmd.CLIENT_CMD_REMOTE_WEB_SERVER_STATE`) of server state changes

**External control**:
```bash
# Check if remote transmission is enabled
adb shell settings get global rokid_er_remote_transmission

# Enable remote transmission
adb shell settings put global rokid_er_remote_transmission 1

# Disable remote transmission
adb shell settings put global rokid_er_remote_transmission 0
```

**AndServer registration config**: The asset file `com.rokid.glass.webserver.andserver` is used by AndServer's annotation processor for auto-registration of controllers/interceptors.

---

### 6. NsdService

**Class**: `com.rokid.glass.webserver.ftp.NsdService`
**Exported**: true
**No intent filter** (started programmatically)

Network Service Discovery (NSD/mDNS) registration service. Registers the glasses as an FTP service (`_ftp._tcp.`) on the local network so companion apps can discover them automatically.

**Broadcast actions** (for external control):
- `com.rokid.os.master.assist.web.start` -- start NSD registration
- `com.rokid.os.master.assist.web.stop` -- stop NSD registration

Uses the service name derived from `Build.getSerial()`. Currently configured as `USE_NDS = false` in `WebServerService`, meaning NSD is not actively used by default.

---

### 7. RokidBluetoothService

**Class**: `com.rokid.sprite.bluetooth.RokidBluetoothService`
**Exported**: true
**No intent filter** (started programmatically via `startService`)
**`stopWithTask`**: false (survives task removal)
**Implements**: `IBluetoothServer`

The largest and most complex service. Manages all Bluetooth communication between the glasses and the companion phone, including:

**Core sub-managers**:
- `CXRServiceManager` -- CXR-M SDK bridge for Rokid's proprietary Bluetooth protocol
- `PhoneCallManager` -- HFP call handling (accept, reject, terminate)
- `PhoneA2dpManager` -- A2DP audio streaming (music playback control)
- `PhoneDialogManager` -- call UI display on glasses
- `BluetoothStateManager` -- connection state tracking
- `BluetoothPBAPManager` -- PBAP phonebook sync from phone
- `BluetoothAIManager` -- AI model operations via Bluetooth
- `RokidAIManager` -- Rokid AI assistant integration
- `RokidGuideManager` -- user guide functionality
- `RokidTouchManager` -- touch/gesture input handling
- `ScreenStatusManager` -- screen state coordination
- `LauncherStatusManager` -- launcher state syncing
- `NtfCmdHelper` -- notification command handling
- `WifiP2pServerManager` -- WiFi Direct for high-bandwidth transfer

**Key IBluetoothServer methods**:
- `acceptCall()`, `rejectCall()`, `terminateCall()` -- phone call control
- `sendGattMessage(json)` -- send data to phone via BLE GATT
- `controlWifiP2p(open, close)` -- manage WiFi Direct connections
- `gattConnectStatus()` -- query BLE GATT connection state
- `SyncBluetoothStatus()` -- push current BT state to launcher
- `resetBluetoothStatus()` -- reset BT connection state
- `setMusicStatus(status)` -- set music playback state
- `toggleMute()` -- toggle microphone mute
- `sendKeyEvent(key, data)` -- relay key events to phone
- `aiModelOperate(data)` -- trigger AI model operations on phone
- `addAPPModule(data)` -- add system app module via phone
- `startAudioStream(type, function, data)` -- start audio streaming session
- `updateMediaCount(scene)` -- notify phone of new media capture
- `scenesInfo(data)` / `exitScenesInfo(data)` -- send AR/MR scene info to phone

**Broadcast receivers**: Listens for `ACTION_CALL_CHANGED` (`android.bluetooth.headsetclient.profile.action.AG_CALL_CHANGED`) and various Bluetooth state change intents. Also has `RokidDoorReceiver` (glasses open/close detection) and `RokidTouchReceiver` (touch input).

---

### 8. SystemFuncService

**Class**: `com.rokid.os.sprite.assist.system.SystemFuncService`
**Exported**: true
**Intent filter action**: `com.rokid.os.sprite.assist.system.SystemFuncService`
**Implements**: `ISystemFuncServer`

Manages system-level functions: volume, brightness, battery, scheduling, memos, journeys, notifications, and store demo mode.

**Core sub-managers**:
- `StatusChangeManager` -- monitors and reacts to system status changes (battery, connectivity, etc.)
- `NotifyMsgManager` -- manages notification messages on the glasses display
- `ScheduleManager` -- schedule/alarm management
- `StoreDemoManager` -- retail store demo mode

**Key ISystemFuncServer methods**:
- `getVolumeSpecified()` / `setVolumeSpecified(value)` -- volume control
- `getBrightnessSpecified()` / `setBrightnessSpecified(value)` -- brightness control
- `getBatteryLevel()` -- get current battery percentage
- `showModeChangeDialog(show, typeIn)` -- display mode change dialog overlay
- `showSystemMsg(systemMsg)` / `showMobileMsg(mobileMsg)` -- display system/mobile messages
- `syncSchedule(json)` / `addSchedule(json)` / `removeSchedule(json)` -- schedule management
- `syncMemo(json)` / `clearMemo()` / `getMemo()` -- memo management
- `syncJourney(json)` / `addJourney(json)` / `removeJourney(json)` / `getJourney()` -- journey management
- `glassTakeOnChange(takeOn)` -- react to glasses wear state
- `notifySceneStatusChanged()` -- notify scene state changes

**Low storage handling**: Listens for storage warnings via `RokidStorageManager.ILowStorageListener`. When storage is low, automatically stops video recording and mix recording scenes with a TTS warning.

**Language management**: Tracks `settings_language` setting and applies locale changes.

---

### 9. SpriteMediaService

**Class**: `com.rokid.os.sprite.assist.media.SpriteMediaService`
**Exported**: true
**Intent filter action**: `com.rokid.os.sprite.assist.media.SpriteMediaService`
**Extends**: `LifecycleService`
**Implements**: `ISpriteMediaServer`

Manages all camera and audio recording operations on the glasses.

**Core sub-managers**:
- `CameraFuncManager` -- camera open/close, photo capture, video recording
- `AudioFuncManager` -- audio recording start/stop
- `ArPictureManager` -- AR photo capture
- `MixRecordManager` -- mixed reality (AR overlay) screen recording

**Message constants**:
- `MSG_TAKE_PICTURE` = 102
- `MSG_START_VIDEO_RECORD` = 103
- `MSG_STOP_VIDEO_RECORD` = 104
- `MSG_START_AUDIO_RECORD` = 105
- `MSG_STOP_AUDIO_RECORD` = 106

**Key ISpriteMediaServer methods**:
- `takePicture(openType)` -- take a photo; `openType` specifies the trigger source
- `videoRecord(start, openType)` -- start/stop video recording
- `audioRecord(start, openType)` -- start/stop audio recording
- `isVideoRecording()` / `isAudioRecording()` -- query recording state
- `setAudioRecordLocation(location)` -- set GPS location metadata for recordings
- `onPlaySound(soundId)` -- play a sound effect (shutter, etc.)
- `getCurrentLocation()` / `updateLocation(location)` -- manage GPS for media EXIF
- `startArPicture()` / `stopArPicture()` -- AR photo session
- `startMixRecord()` / `stopMixRecord()` -- AR screen recording

**Camera picture types** (from `CameraData`):
- `TYPE_PICTURE_AI_ASSIST` -- AI assistant-triggered photo
- `TYPE_PICTURE_AR_MIX` -- AR mixed reality photo
- `TYPE_VIDEO_MIX_SCREEN` -- AR screen recording video

**Environment checks before recording**:
- Storage space check: refuses if SD card storage is insufficient
- Battery check for video: refuses if battery <= 10% (regular video) or <= 20% (AR screen recording), with TTS warning

**Media file paths**:
- Videos: `/sdcard/Movies/Camera/`
- Audio recordings: `/sdcard/Recordings/`
- On startup, cleans up orphaned `.tmp` files in both directories

**Results** are broadcast back to clients via `AssistClientCmd.CLIENT_RESULT_TAKE_PICTURE`, `CLIENT_RESULT_VIDEO_RECORD`, `CLIENT_RESULT_AUDIO_RECORD`.

---

### 10. TtsService

**Class**: `com.rokid.os.sprite.tts.TtsService`
**Exported**: true
**Process**: `:tts` (separate process from main app)
**Intent filter action**: `com.rokid.os.sprite.tts.TTS_SERVICE`

On-device text-to-speech engine using Rokid's custom `TtsNative` JNI library with ONNX neural network models. Runs in its own process for isolation.

**AIDL Interface -- ITtsServer** (descriptor: `com.rokid.os.sprite.tts.ITtsServer`):

| Method | Description |
|--------|-------------|
| `playTtsMsg(String msg, String uuid, ITtsListener listener)` | Speak a text message with unique ID |
| `stopTtsPlay(String uuid)` | Stop playback of a specific TTS message by UUID |
| `updateTtsParam(String param)` | Update TTS parameters (commits to SharedPrefs, then exits process to restart with new params) |

**AIDL Interface -- ITtsListener** (descriptor: `com.rokid.os.sprite.tts.ITtsListener`):

| Method | Description |
|--------|-------------|
| `onTtsStart(String uuid)` | Called when TTS playback starts |
| `onTtsStop(String uuid)` | Called when TTS playback finishes |

**Binding from another app**:
```java
Intent intent = new Intent("com.rokid.os.sprite.tts.TTS_SERVICE");
intent.setPackage("com.rokid.os.sprite.assistserver");
bindService(intent, connection, Context.BIND_AUTO_CREATE);
```

**TTS engine details**:
- Native library version: `2.1.3`
- Uses ONNX models for acoustic model + vocoder (FP16 precision)
- Two voice models available (directories `models/1/` and `models/2/`), each with:
  - `rokid_tts_acoustic_1_fp16.onnx`, `rokid_tts_acoustic_2_fp16.onnx`
  - `rokid_tts_vocoder_1_fp16.onnx`, `rokid_tts_vocoder_2_fp16.onnx`
- Uses Jieba for Chinese text segmentation
- Has text normalization (TN) FST models for both Chinese and English
- Supports traditional-to-simplified Chinese conversion
- Audio output: `StreamAudioPlayer` with streaming PCM playback
- Optional PCM debug dump: enabled via system prop `persist.rkd.local.tts.pcm.save.enable=true`

**Boot TTS**: On first boot (when `KEY_SHUTDOWN_FLAG` is true and `KEY_BOOT_TTS_FLAG` is false), plays a power-on greeting TTS message.

**System property flags**:
- `persist.rkd.local.tts.pcm.save.enable` -- enable PCM audio dump for debugging
- `vendor.rkd.need.restore.volume` -- restore default volume after TTS
- TTS init/boot/connect/shutdown flags managed via `TtsUtil` keys

**Queue behavior**: If multiple TTS messages arrive while one is playing, intermediate messages are skipped (only the latest is played). The current playback is stopped before starting the new one.

---

### 11. SDKService (JD JoyGo Client)

**Class**: `com.jd.jr.joygoclient.SDKService`
**Exported**: true
**Intent filter action**: `com.jd.jr.joygoclient.SDKService`

JD.com's JoyGo payment SDK service. Embedded as a library within the app. Provides an AIDL binder (`ISDKService`) for JD payment operations.

**AIDL Interface -- ISDKService**:

| Method | Description |
|--------|-------------|
| `registerCallback(IServiceCallback callback)` | Register for JD payment callbacks |
| `unregisterCallback(IServiceCallback callback)` | Unregister callback |
| `sendMessageToSDK(String message, byte[] data)` | Send a message/command to the JD payment SDK |

Communication is relayed to/from the `JoyGoClient` singleton. On unbind, sends an `ONDESTROY` Bluetooth-type message to clean up.

---

## Content Providers

### 1. MultiSpProvider

**Class**: `com.rokid.os.sprite.assist.engine.MultiSpProvider`
**Authority**: `com.rokid.glass.er.assistserver.MultiSpProvider`
**Exported**: true

A cross-process SharedPreferences provider. Allows other apps to read/write key-value pairs stored in a SharedPreferences file named `multi_sp_provider`. This is the primary mechanism for sharing simple configuration data between system apps on the glasses.

**URI format**: `content://com.rokid.glass.er.assistserver.MultiSpProvider/<operation>/<key>`

**Supported operations via URI path**:

| Operation | Method | URI Path | Description |
|-----------|--------|----------|-------------|
| Get all | `query()` | `/get_all` | Returns all key-value pairs as a MatrixCursor with columns `cursor_name`, `cursor_type`, `cursor_value` |
| Get value | `getType()` | `/<type>/<key>` | Returns the string representation of the value |
| Check exists | `getType()` | `/contain/<key>` | Returns `"true"` or `"false"` |
| Insert/Update | `insert()` | `/<type>/<key>` | ContentValues must contain `"value"` key |
| Delete key | `delete()` | `/remove/<key>` | Removes a specific key |
| Clear all | `delete()` | `/clean` | Clears all preferences |

**Supported types**: `string`, `boolean`, `int`, `long`, `float`

**Example usage from another app**:
```java
// Read a string value
String authority = "com.rokid.glass.er.assistserver.MultiSpProvider";
Uri uri = Uri.parse("content://" + authority + "/string/my_key");
String value = getContentResolver().getType(uri);

// Write a string value
ContentValues cv = new ContentValues();
cv.put("value", "my_value");
Uri insertUri = Uri.parse("content://" + authority + "/string/my_key");
getContentResolver().insert(insertUri, cv);
```

### 2. FileProvider

**Class**: `androidx.core.content.FileProvider`
**Authority**: `com.rokid.os.sprite.assistserver.provider`
**Exported**: false
**Grant URI permissions**: true

Standard Android FileProvider for securely sharing files with other apps. Configured via `@xml/provider_paths`.

### 3. InitializationProvider

**Class**: `androidx.startup.InitializationProvider`
**Authority**: `com.rokid.os.sprite.assistserver.androidx-startup`
**Exported**: false

AndroidX Startup library provider. Initializes:
- `EmojiCompatInitializer`
- `ProcessLifecycleInitializer`
- `ProfileInstallerInitializer`

---

## Broadcast Receivers

### BootReceiver

**Class**: `com.rokid.os.sprite.basic.receiver.BootReceiver`
**Exported**: true
**Action**: `android.intent.action.BOOT_COMPLETED`

Receives the boot completed broadcast. Since the app is marked `persistent=true`, it should already be running. The receiver logs the boot event but does not start any additional services (they are already started in `MainApplication.onCreate()`).

### CommandReceiver (Dynamic)

**Class**: `com.rokid.os.sprite.assist.receiver.CommandReceiver`
**Registered dynamically** in `MasterAssistService.onCreate()`
**Action**: `com.rokid.os.master.assist.server.cmd`

Receives external commands via broadcast. Used for ADB-driven control and inter-app communication.

**Command extras** (in the broadcast Intent):
- `cmd_type` -- the command type string
- Additional extras depend on the command

**Supported commands** (from `AssistCommand`):

| Command | Extra Parameters | Description |
|---------|-----------------|-------------|
| `change_env` | `env` = `"product"` / `"test"` / `"dev"` | Switch server environment |
| `open_wifi_p2p` | -- | Open WiFi Direct connection |
| `close_wifi_p2p` | -- | Close WiFi Direct connection |
| `control_scene` | `scene`, `open` = `"true"/"false"` | Open or close a scene |
| `setting_change` | `value` | Apply a settings change |
| `mobile_assistant` | `open` = `"true"/"false"` | Start/stop voice recognition |
| `start_server` | -- | Start assist server |
| `stop_server` | -- | Stop assist server |

**ADB usage examples**:
```bash
# Change environment to test
adb shell am broadcast -a com.rokid.os.master.assist.server.cmd \
  --es cmd_type change_env --es env test

# Open a scene
adb shell am broadcast -a com.rokid.os.master.assist.server.cmd \
  --es cmd_type control_scene --es scene translate --es open true

# Start mobile assistant voice recognition
adb shell am broadcast -a com.rokid.os.master.assist.server.cmd \
  --es cmd_type mobile_assistant --es open true
```

### ProfileInstallReceiver

**Class**: `androidx.profileinstaller.ProfileInstallReceiver`
**Exported**: true
**Permission required**: `android.permission.DUMP`

AndroidX baseline profile installer receiver. Supports actions: `INSTALL_PROFILE`, `SKIP_FILE`, `SAVE_PROFILE`, `BENCHMARK_OPERATION`.

---

## Assets

### TTS Engine Resources (`resource/`)

The TTS engine's neural network models and configuration files:

| Path | Description |
|------|-------------|
| `resource/front.conf` | TTS frontend configuration |
| `resource/model.conf` | TTS model configuration |
| `resource/models/1/rokid_tts_acoustic_*.onnx` | Voice 1 acoustic ONNX models (FP16) |
| `resource/models/1/rokid_tts_vocoder_*.onnx` | Voice 1 vocoder ONNX models (FP16) |
| `resource/models/2/rokid_tts_acoustic_*.onnx` | Voice 2 acoustic ONNX models (FP16) |
| `resource/models/2/rokid_tts_vocoder_*.onnx` | Voice 2 vocoder ONNX models (FP16) |
| `resource/phone_id_map_1.txt` / `phone_id_map_2.txt` | Phoneme-to-ID mappings per voice |
| `resource/tone_id_map_1.txt` / `tone_id_map_2.txt` | Tone-to-ID mappings per voice |
| `resource/word2phone.dict_1.bin` / `word2phone.dict_2.bin` | Word-to-phoneme dictionaries |
| `resource/tn/zh_tn_tagger.fst` / `zh_tn_verbalizer.fst` | Chinese text normalization FSTs |
| `resource/tn/en_tn_tagger.fst` / `en_tn_verbalizer.fst` | English text normalization FSTs |
| `resource/dict/jieba/` | Jieba Chinese segmentation dictionary and models |
| `resource/dict/tranditional_to_simplified/` | Traditional-to-simplified Chinese conversion table |

### Other Assets

| File | Description |
|------|-------------|
| `model/best_sz256.mnn` | MNN neural network model (likely for photo occlusion detection, used by `Occulusion.Detect()`) |
| `rfm_3dof.conf` | Head nod/shake detection configuration (gyroscope thresholds: nod=1.0 rad/s, shake=1.2 rad/s peak) |
| `cncity.txt` | Chinese city list (for location/city guide features) |
| `nearx_bs.jpg` | NearX branding/splash image |
| `phone_ringtone_sedna.ogg` | Phone ringtone audio file for incoming call alerts |
| `com.rokid.glass.webserver.andserver` | AndServer auto-registration config |
| `dexopt/baseline.prof` / `baseline.profm` | ART baseline profile for performance optimization |

---

## Key API Endpoints (for external apps)

### Binding to MasterAssistService (primary IPC)

```
Action: com.rokid.os.sprite.assist.MasterAssistService
Package: com.rokid.os.sprite.assistserver
Interface: com.rokid.os.sprite.assist.server.IAssistServer
```

### Binding to TtsService (text-to-speech)

```
Action: com.rokid.os.sprite.tts.TTS_SERVICE
Package: com.rokid.os.sprite.assistserver
Interface: com.rokid.os.sprite.tts.ITtsServer
```

### Binding to SDKService (JD payment)

```
Action: com.jd.jr.joygoclient.SDKService
Package: com.rokid.os.sprite.assistserver
Interface: com.jd.jr.joygoclient.ISDKService
```

### Content Provider (cross-process settings)

```
Authority: com.rokid.glass.er.assistserver.MultiSpProvider
```

### Broadcast command receiver

```
Action: com.rokid.os.master.assist.server.cmd
```

---

## Environment Configuration

The app supports three server environments:

| Environment | Host URL |
|------------|----------|
| `product` | `https://rokid-content-pro.rokid.com` |
| `test` | `https://rokid-content-test.rokid.com` |
| `dev` | `https://rokid-content-dev.rokid.com` |

Switch via broadcast:
```bash
adb shell am broadcast -a com.rokid.os.master.assist.server.cmd \
  --es cmd_type change_env --es env test
```

---

## System Properties Used

| Property | Purpose |
|----------|---------|
| `persist.rkd.local.tts.pcm.save.enable` | Enable TTS PCM audio dump |
| `vendor.rkd.need.restore.volume` | Flag to restore default volume after TTS |
| `ro.config.media_vol_default` | Default media volume level |
| `persist.rokid.display.mode.enable` | Display mode enabled flag |
| `ro.boot.glassesWithPanel` | Glasses have built-in display panel |

TTS-related properties (managed via `TtsUtil` / `DeviceUtil`):
- `KEY_TTS_INIT_FLAG` -- `"0"` = not initialized, `"1"` = ready
- `KEY_BOOT_TTS_FLAG` -- `"false"` = boot TTS not yet played, `"true"` = played
- `KEY_SHUTDOWN_FLAG` -- tracks if device was cleanly shut down
- `KEY_APP_CONNECT_FLAG` -- `"1"` = app just connected, triggers connect TTS
- `KEY_SYNC_TIME_FLAG` -- time sync state

---

## Network Configuration

The app uses OkHttp with:
- TLS (custom SSL socket factory, accepts all certificates)
- 10-second connect and read timeouts
- 20MB disk cache at `ExternalCacheDir/OkhttpCache`
- Connection pool with 0 idle connections (no keep-alive)
- Custom interceptors: `CacheInterceptor`, `AuthenticationInterceptor`
- Request headers: `Accept-Language`, `systemVersionName`, `RequestId` (UUID), `DateTime`
- Network security config: `@xml/network_security_config`
