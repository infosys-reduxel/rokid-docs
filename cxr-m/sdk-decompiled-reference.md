# CXR-M SDK Decompiled Reference

This document provides a comprehensive reference for the Rokid CXR-M SDK (`com.rokid.cxr:client-m`) based on decompilation of the SDK's .class files. It covers every class, interface, enum, callback, listener, and internal mechanism found in the SDK.

**SDK Version**: 1.0.8 (version code 108)
**SDK Build Time**: 2026-02-02 11:37:58 (extend layer) / 2026-02-02 11:37:12 (core layer)
**Min SDK**: 28 (Android 9 Pie)

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [CxrApi - Public API Surface](#cxrapi---public-api-surface)
3. [CxrController - Core Communication Layer](#cxrcontroller---core-communication-layer)
4. [BluetoothController - BLE and Socket Management](#bluetoothcontroller---ble-and-socket-management)
5. [WifiController - Wi-Fi P2P Connection](#wificontroller---wi-fi-p2p-connection)
6. [FileController - File Sync and APK Upload](#filecontroller---file-sync-and-apk-upload)
7. [AudioController - Communication Device Routing](#audiocontroller---communication-device-routing)
8. [CXRSocketProtocol - Low-Level Bluetooth Protocol](#cxrsocketprotocol---low-level-bluetooth-protocol)
9. [CXRServiceBridge - Glasses-Side Bridge (CXR-S)](#cxrservicebridge---glasses-side-bridge-cxr-s)
10. [Caps - Serialization Format](#caps---serialization-format)
11. [Enums](#enums)
12. [Callback Interfaces](#callback-interfaces)
13. [Listener Interfaces](#listener-interfaces)
14. [Info Classes (Data Models)](#info-classes-data-models)
15. [Sync / Retrofit Classes](#sync--retrofit-classes)
16. [Version Checking](#version-checking)
17. [Utility Classes](#utility-classes)
18. [Obfuscated Classes](#obfuscated-classes)
19. [Message Protocol - Channel Names and Commands](#message-protocol---channel-names-and-commands)
20. [SN Verification and AES Encryption](#sn-verification-and-aes-encryption)

---

## Architecture Overview

The SDK has a layered architecture:

```
CxrApi (com.rokid.cxr.client.extend.CxrApi)
  -- Public-facing singleton, holds all callbacks/listeners
  |
  +-- CxrController (com.rokid.cxr.client.controllers.CxrController)
  |     -- Core controller, manages request IDs and routing
  |     |
  |     +-- BluetoothController
  |     |     -- Manages BLE GATT + classic Bluetooth socket
  |     |     |
  |     |     +-- CXRSocketProtocol (native JNI layer)
  |     |           -- Handles framing, request/response, streaming over BT socket
  |     |
  |     +-- AudioController
  |           -- Routes audio to/from Bluetooth SCO device
  |
  +-- WifiController (com.rokid.cxr.client.extend.controllers.WifiController)
  |     -- Wi-Fi P2P (Wi-Fi Direct) peer discovery and connection
  |
  +-- FileController (com.rokid.cxr.client.extend.controllers.FileController)
        -- HTTP file sync and APK upload over Wi-Fi P2P (port 8848)
        |
        +-- RetrofitClient / RetrofitService
              -- Retrofit HTTP client for glasses HTTP server
```

All Bluetooth communication uses the **Caps** serialization format. Messages are sent through named **channels** (e.g., "Ai", "Dev", "Med", "Sys", "Trans", "ARTC", "Settings") with specific **command strings** as the first Caps field.

---

## CxrApi - Public API Surface

**Package**: `com.rokid.cxr.client.extend`
**Singleton**: `CxrApi.getInstance()`

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `Context` | Application context |
| `b` | `String` | Wi-Fi P2P address (IP of glasses) |
| `c` | `ApkStatusCallback` | APK install/uninstall status callback |
| `d` | `AudioSceneIdCallback` | Audio scene ID change callback |
| `e` | `BluetoothStatusCallback` | Bluetooth status callback |
| `f` | `PhotoPathCallback` | Photo path (URL) result callback |
| `g` | `PhotoResultCallback` | Photo binary result callback |
| `h` | `SendStatusCallback` | Stream send status callback |
| `i` | `SyncStatusCallback` | File sync status callback |
| `j` | `WifiP2PStatusCallback` | Wi-Fi P2P status callback |
| `k` | `AiEventListener` | AI key event listener |
| `l` | `ArtcListener` | ARTC streaming listener |
| `m` | `AudioStreamListener` | Audio stream listener |
| `n` | `BatteryLevelUpdateListener` | Battery level change listener |
| `o` | `BrightnessUpdateListener` | Brightness change listener |
| `p` | `CustomCmdListener` | Custom command listener |
| `q` | `CustomViewListener` | Custom view lifecycle listener |
| `r` | `MediaFilesUpdateListener` | Media files change listener |
| `s` | `SceneStatusUpdateListener` | Scene status change listener |
| `t` | `ScreenStatusUpdateListener` | Screen on/off listener |
| `u` | `TranslationListener` | Translation start/stop listener |
| `v` | `VolumeUpdateListener` | Volume change listener |
| `w` | `int` (volatile) | Request ID counter (incremented per request) |
| `x` | `int` (volatile) | AI session counter |
| `y` | `boolean` | First-connection flag (triggers time sync + SN check) |
| `z` | `byte[]` | SN encrypted content (for AES verification) |
| `A` | `String` | Client secret (used as AES key, dashes stripped) |
| `B` | `SceneStatusInfo` | Current scene status |
| `C` | `GlassInfo` | Cached glasses info |
| `D` | `CxrApi$a` | Internal CxrController.Callback adapter |
| `E` | `CxrApi$b` | Internal WifiController.Callback adapter |
| `F` | `CxrApi$c` | Internal FileController.Callback adapter |

### Bluetooth Methods

```java
// Initialize Bluetooth connection with a discovered BLE device
public void initBluetooth(Context context, BluetoothDevice device, BluetoothStatusCallback callback)

// Reconnect using previously stored UUID, MAC, and SN verification data
public void connectBluetooth(Context context, String socketUuid, String macAddress,
    BluetoothStatusCallback callback, byte[] snEncryptContent, String clientSecret)

// Check if Bluetooth communication is connected
public boolean isBluetoothConnected()

// Update the Rokid account on the glasses
public void updateRokidAccount(String rokidAccount)

// Disconnect and clean up Bluetooth
public void deinitBluetooth()

// Route phone audio to Bluetooth SCO device (glasses speaker)
public void setCommunicationDevice()

// Stop routing audio to Bluetooth SCO device
public void clearCommunicationDevice()
```

### Device Control Methods

```java
// Sync phone time/timezone to glasses
// Sends JSON: {"timeZone": "<tz_id>", "time": <millis>}
// Channel: "Dev", Command: "Dev_TimeUpdate"
public CxrStatus setGlassTime()

// Get glasses hardware/software info (async result via callback)
// Channel: "Dev", Command: "Dev_GetGlassInfo"
public CxrStatus getGlassInfo(GlassInfoResultCallback callback)

// Set brightness [0-15]
// Channel: "Dev", Command: "Dev_GlassBrightness", data: UInt32
public CxrStatus setGlassBrightness(int value)

// Set volume [0-15]
// Channel: "Dev", Command: "Dev_GlassSound", data: UInt32
public CxrStatus setGlassVolume(int value)

// Set photo capture resolution
// Channel: "Settings", Command: "Settings_Update"
// Data: JSON array [{key: "settings_photo_width", value: "N"}, {key: "settings_photo_height", value: "N"}]
public CxrStatus setPhotoParams(int width, int height)

// Set video recording parameters
// Channel: "Settings", Command: "Settings_Update"
// Keys: settings_video_duration, settings_video_fps, settings_video_width, settings_video_height, settings_video_duration_unit
public CxrStatus setVideoParams(int duration, int fps, int width, int height, int unit)

// Set sound effect value
// Channel: "Settings", Command: "Settings_Update", Key: "settings_sound_effect"
public CxrStatus setSoundEffect(String value)

// Set screen-off timeout in milliseconds
// Channel: "Settings", Command: "Settings_Update", Key: "settings_screen_offTimeout"
public CxrStatus setScreenOffTimeout(long value)

// Set power-off timeout
public CxrStatus setPowerOffTimeout(int timeout)

// Reboot glasses
public CxrStatus notifyGlassReboot()

// Shutdown glasses
public CxrStatus notifyGlassShutdown()

// Turn glasses screen off
public CxrStatus notifyGlassScreenOff()

// Check for glasses firmware update (async result via callback)
public CxrStatus checkGlassVersion(GlassVersionCallback callback)

// Get current cached SceneStatusInfo
public SceneStatusInfo getSceneStatusInfo()
```

### AI Interaction Methods

```java
// Send exit event to end AI session on glasses
// Channel: "Ai", Command: "Exit", data: aiSessionCount (Int32)
public CxrStatus sendExitEvent()

// Send ASR (speech recognition) content to glasses
// Channel: "Ai", Command: "ASR_Result", data: content (String)
public CxrStatus sendAsrContent(String content)

// Notify glasses no ASR result
// Channel: "Ai", Command: "ASR_None"
public CxrStatus notifyAsrNone()

// Notify glasses ASR recognition ended
// Channel: "Ai", Command: "ASR_End"
public CxrStatus notifyAsrEnd()

// Notify glasses ASR recognition error
// Channel: "Ai", Command: "ASR_Error"
public CxrStatus notifyAsrError()

// Notify glasses AI process started
// Channel: "Ai", Command: "Ai_Start"
public CxrStatus notifyAiStart()

// Send AI heartbeat with counter
// Channel: "Ai", Command: "Ai_Heartbeat", data: JSON {"count": N}
public CxrStatus sendAi_Heartbeat()

// Notify glasses of no network
// Channel: "Ai", Command: "NoNetwork"
public CxrStatus notifyNoNetwork()

// Notify glasses photo upload error
// Channel: "Ai", Command: "Pic_UploadError"
public CxrStatus notifyPicUploadError()

// Notify glasses AI request failed
// Channel: "Ai", Command: "Ai_Error"
public CxrStatus notifyAiError()

// Send TTS text to glasses display
// Channel: "Ai", Command: "TTS_Result", data: content (String)
public CxrStatus sendTtsContent(String content)

// Notify glasses TTS playback finished
// Channel: "Ai", Command: "TTS_AudioFinished"
public CxrStatus notifyTtsAudioFinished()
```

### Camera Methods

```java
// Open glasses camera with resolution and quality
// Channel: "Ai", Command: "Ai_OpenCamera"
// Data: JSON {"width": N, "height": N, "quality": N}
public CxrStatus openGlassCamera(int width, int height, int quality)

// Take photo and receive raw bytes via callback
// Channel: "Ai", Command: "Ai_TakePhoto"
// Data: JSON {"width": N, "height": N, "quality": N}
public CxrStatus takeGlassPhoto(int width, int height, int quality, PhotoResultCallback callback)

// Take photo and receive file URL path via callback
// Channel: "Med", Command: "Med_Take_Photo_Url"
// Data: JSON {"width": N, "height": N, "quality": N}
public CxrStatus takeGlassPhoto(int width, int height, int quality, PhotoPathCallback callback)

// Take photo from global context (outside AI process)
public CxrStatus takeGlassPhotoGlobal(int width, int height, int quality, PhotoResultCallback callback)
```

### Word Tips Methods

```java
// Configure text display area on glasses for word tips
// Channel: "Sys", Command: "WordTips_SetTvConfig"
// Data: JSON {textSize, lineSpacing, mode, contentSite: {startPointX, startPointY, width, height}}
public CxrStatus configWordTipsText(float textSize, float lineSpace, String mode,
    int startPointX, int startPointY, int width, int height)

// Send ASR content for word tips display
// Channel: "Sys", Command: "WordTips_AiContent", Data: JSON {"asr": "content"}
public CxrStatus sendWordTipsAsrContent(String content)
```

### Translation Methods

```java
// Configure translation text display area
// Channel: "Trans", Command: "Trans_SetTvConfig"
// Data: JSON {textSize, contentSite: {startPointX, startPointY, width, height}}
public CxrStatus configTranslationText(int textSize, int startPointX, int startPointY,
    int width, int height)

// Send translation content to glasses
// Channel: "Trans", Command: "Trans_Result"
// Data: JSON {id, subId, temporary, finished, result}
public CxrStatus sendTranslationContent(int id, int subId, boolean temporary,
    boolean finished, String content)
```

### ARTC (Audio-RTC) Methods

```java
// Stop ARTC stream
// Channel: "ARTC", Command: "OnStop"
public CxrStatus stopArtc()

// Configure ARTC video frame display
public CxrStatus configArtcFrame(int startPointX, int startPointY, int width, int height, int layer)

// Send speaking status during ARTC session
public CxrStatus sendArtcSpeakStatus(boolean isSpeaking)

// Send ASR content during ARTC session
public CxrStatus sendArtcAsrContent(String content, boolean isFinal, int id)

// Send TTS content during ARTC session
public CxrStatus sendArtcTtsContent(String content, boolean isFinal, int id)
```

### Global Message Methods

```java
// Send global message to glasses display
public CxrStatus sendGlobalMsgContent(int type, String content, boolean isShow)

// Send global toast to glasses display
public CxrStatus sendGlobalToastContent(int type, String content, boolean isShow)

// Send global TTS content
public CxrStatus sendGlobalTtsContent(String content)
```

### Custom View Methods

```java
// Send custom icons to glasses
public CxrStatus sendCustomViewIcons(List<IconInfo> icons)

// Send Lottie animation JSON to glasses
public CxrStatus sendCustomView_LottieAnimJson(String name, String jsonData)

// Control Lottie animation playback
public CxrStatus sendCustomView_LottieAnimControl(String name, LottieAnimControl control)

// Open custom view on glasses with JSON layout
public CxrStatus openCustomView(String jsonLayout)

// Update custom view content
public CxrStatus updateCustomView(String jsonData)

// Close custom view
public CxrStatus closeCustomView()
```

### Custom Command

```java
// Send arbitrary command with Caps data
public CxrStatus sendCustomCmd(String cmd, Caps args)
```

### Scene Control

```java
// Enable/disable a scene on glasses
public CxrStatus controlScene(CxrSceneType sceneType, boolean enable, String params)
```

### Audio Control

```java
// Open audio recording on glasses
public CxrStatus openAudioRecord(int codecType, String streamType)

// Close audio recording on glasses
public CxrStatus closeAudioRecord(String streamType)

// Change audio scene ID
public CxrStatus changeAudioSceneId(int sceneId, AudioSceneIdCallback callback)

// Set local TTS parameter JSON
public CxrStatus setLocalTtsParam(String paramJson)

// Set local TTS speech speed
public CxrStatus setLocalTtsSpeed(float speed)
```

### Streaming Methods

```java
// Send binary stream data to glasses (for WORD_TIPS type, adds file name)
// Channel: "Sys", Command: "WordTips_SendData" + JSON {"fileName": name}
public CxrStatus sendStream(CxrStreamType type, byte[] stream, String fileName,
    SendStatusCallback callback)
```

### Wi-Fi P2P Methods

```java
// Initialize Wi-Fi P2P connection
// Sends channel "Med", command "Sync_Start" with JSON {"type": "Android"}
public CxrStatus initWifiP2P(WifiP2PStatusCallback callback)

// Check Wi-Fi P2P connection status
public boolean isWifiP2PConnected()

// Disconnect Wi-Fi P2P
public CxrStatus deinitWifiP2P()
```

### File Sync Methods

```java
// Get count of unsynchronized media files
// Channel: "Med", Command: "UnSync_Count"
public CxrStatus getUnsyncNum(UnsyncNumResultCallback callback)

// Start syncing media files from glasses over Wi-Fi (requires Wi-Fi P2P connected)
public boolean startSync(String savePath, CxrMediaType[] types, SyncStatusCallback callback)

// Sync a single media file
public boolean syncSingleFile(String savePath, CxrMediaType type, String filePath,
    SyncStatusCallback callback)

// Stop syncing
public void stopSync()
```

### APK Management Methods

```java
// Upload APK file to glasses and install
// Uses HTTP upload to glasses HTTP server at http://<wifi_p2p_ip>:8848
public boolean startUploadApk(String wifiAddress, ApkStatusCallback callback)

// Cancel APK upload
public void stopUploadApk()

// Uninstall app from glasses by package name
public CxrStatus uninstallApk(String packageName, ApkStatusCallback callback)

// Open app on glasses
public CxrStatus openApp(RKAppInfo appInfo, ApkStatusCallback callback)
```

### Listener Setter Methods

```java
public void setAiEventListener(AiEventListener listener)
public void setArtcListener(ArtcListener listener)
public void setAudioStreamListener(AudioStreamListener listener)
public void setBatteryLevelUpdateListener(BatteryLevelUpdateListener listener)
public void setBrightnessUpdateListener(BrightnessUpdateListener listener)
public void setCustomCmdListener(CustomCmdListener listener)
public void setCustomViewListener(CustomViewListener listener)
public void setMediaFilesUpdateListener(MediaFilesUpdateListener listener)
public void setSceneStatusUpdateListener(SceneStatusUpdateListener listener)
public void setScreenStatusUpdateListener(ScreenStatusUpdateListener listener)
public void setTranslationListener(TranslationListener listener)
public void setVolumeUpdateListener(VolumeUpdateListener listener)
```

### Inner Class: LottieAnimControl (Enum)

```java
public enum CxrApi.LottieAnimControl {
    PLAY,
    PAUSE,
    RESUME,
    CANCEL,
    REVERSESPEED
}
```

---

## CxrController - Core Communication Layer

**Package**: `com.rokid.cxr.client.controllers`
**Singleton**: `CxrController.getInstance()`

The CxrController sits between CxrApi and BluetoothController. It manages:
- The callback chain from Bluetooth up to CxrApi
- Request ID to RequestCallback mapping
- Bluetooth connection state checks (logs "bluetooth not connected" if not)

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `CxrController.Callback` | Upstream callback (set by CxrApi) |
| `b` | `Context` | Application context |
| `c` | `Map<Integer, RequestCallback>` | Request ID to callback mapping |
| `d` | `CxrController$a` | Internal BluetoothController.Callback adapter |

### Methods

```java
public static CxrController getInstance()

public void setCallback(CxrController.Callback callback)
// When callback is non-null, creates new HashMap for request tracking
// When null, clears and nulls the HashMap

public void initBluetooth(Context context, BluetoothDevice device)
// Delegates to BluetoothController.init(context, device, internalCallback)
// If params null, calls callback.onStatusUpdate(BLUETOOTH_UNAVAILABLE, PARAM_INVALID)

public void connectBluetooth(Context context, String socketUuid, String macAddress)
// Delegates to BluetoothController.connect(context, socketUuid, macAddress, internalCallback)

public boolean isBluetoothConnected()
// Delegates to BluetoothController.isConnected()

public CxrStatus request(int reqId, String cmd, Caps args, RequestCallback callback)
// Determines CxrNotifyType: REQUEST if callback non-null, UNKNOWN if null
// Stores callback in map by reqId if present
// Delegates to BluetoothController.request(reqId, cmd, args, notifyType)

public CxrStatus sendStream(String cmd, Caps caps, byte[] stream)
// Delegates to BluetoothController.sendStream(cmd, caps, stream)

public CxrStatus openAudioRecord(int codec, String cmd, Caps args)
public CxrStatus closeAudioRecord(String cmd)

public void setCommunicationDevice()
// Delegates to AudioController.setCommunicationDevice(context)

public void clearCommunicationDevice()
// Delegates to AudioController.clearCommunicationDevice()

public CxrStatus updateRokidAccount(String account)
// Delegates to BluetoothController.updateRokidAccount(account)

public void deinitBluetooth()
// Calls BluetoothController.deinit(SUCCEED), nulls context
```

### CxrController.Callback Interface

```java
public interface CxrController.Callback {
    void onConnectionInfo(String socketUuid, String macAddress, String deviceName, int protocolVersion)
    void onStatusUpdate(CxrStatus status, CxrBluetoothErrorCode errorCode)
    void onValueUpdate(String cmd, Caps args)
    void onStartAudioStream(int codecType, String streamType, Caps args)
    void onAudioStream(byte[] data, int offset, int length)
    void onARTCFrame(byte[] frameData)
}
```

### CxrController.RequestCallback Interface

```java
public interface CxrController.RequestCallback {
    void onResponse(int reqId, CxrStatus status, String cmd, Caps responseArgs)
}
```

---

## BluetoothController - BLE and Socket Management

**Package**: `com.rokid.cxr.client.controllers`
**Singleton**: `BluetoothController.getInstance()`

Manages the dual-layer Bluetooth connection:
1. **BLE GATT** - Used for initial discovery and characteristic reading
2. **Classic Bluetooth Socket** - Used for data transfer via CXRSocketProtocol

### Static UUIDs

```java
// BLE Service/Characteristic UUIDs (stored as static final UUID v, w)
// Used for GATT service discovery and characteristic read/write
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `Context` (volatile) | Application context |
| `b` | `boolean` (volatile) | Connection state |
| `c` | `BluetoothDevice` (volatile) | Target BLE device |
| `d` | `CxrStatus` (volatile) | Current status |
| `e` | `BluetoothController.Callback` (volatile) | Upstream callback |
| `f` | `BluetoothGatt` (volatile) | BLE GATT connection |
| `g` | `BluetoothGattCharacteristic` (volatile) | Target characteristic |
| `h` | `int` (volatile) | MTU size |
| `i` | `boolean` (volatile) | Services discovered flag |
| `j` | `BluetoothAdapter` (volatile) | BT adapter |
| `k` | `BluetoothDevice` (volatile) | Classic BT device (for socket) |
| `l` | `BluetoothSocket` (volatile) | Classic BT socket |
| `m` | `CXRSocketProtocol` (volatile) | Protocol handler |
| `n` | `boolean` (volatile) | Deinit in progress flag |
| `o` | `String` (volatile) | Socket UUID string |
| `p` | `String` (volatile) | MAC address |
| `q` | `String` (volatile) | Device name |
| `r` | `int` (volatile) | Protocol version from socket |
| `s` | `int` (volatile) | Additional version info |
| `t` | `BluetoothController$a` | CXRSocketProtocol.Callback adapter |
| `u` | `BluetoothController$b` | BluetoothGattCallback implementation |

### Methods

```java
public static BluetoothController getInstance()

// Initialize with BLE device - starts GATT connection, discovers services,
// reads characteristic to get socket UUID, then connects classic BT socket
public void init(Context context, BluetoothDevice device, BluetoothController.Callback callback)

// Reconnect using known UUID and MAC (skips BLE discovery)
public void connect(Context context, String socketUuid, String macAddress,
    BluetoothController.Callback callback)

public boolean isConnected()

// Send a request over the BT socket protocol
public synchronized CxrStatus request(int reqId, String cmd, Caps args, CxrNotifyType notifyType)

// Send stream data (caps + binary payload) over BT socket
public CxrStatus sendStream(String cmd, Caps caps, byte[] stream)

// Audio record control
public CxrStatus openAudioRecord(int codec, String cmd, Caps args)
public CxrStatus closeAudioRecord(String cmd)

// Update Rokid account on connected glasses
public CxrStatus updateRokidAccount(String account)

// Disconnect and clean up everything
public synchronized void deinit(CxrBluetoothErrorCode errorCode)
```

### BluetoothController.Callback Interface

```java
public interface BluetoothController.Callback {
    void onConnectionInfo(String socketUuid, String macAddress, String deviceName, int protocolVersion)
    void onStatusUpdate(CxrStatus status, CxrBluetoothErrorCode errorCode)
    void onValueUpdate(int reqId, CxrStatus status, String cmd, Caps args, CxrNotifyType notifyType)
    void onStartAudioStream(int codecType, String streamType, Caps args)
    void onAudioStream(byte[] data, int offset, int length)
    void onARTCFrame(byte[] frameData)
}
```

### Connection Flow (init)

1. Store context, device, callback
2. Start BLE GATT connection to device (`connectGatt`)
3. `BluetoothGattCallback.onConnectionStateChange` -> if connected, discover services
4. `onServicesDiscovered` -> request MTU change, find target characteristic
5. `onMtuChanged` -> read characteristic value
6. `onCharacteristicRead` -> parse socket UUID from characteristic data
7. Get classic BT device by MAC, create RFCOMM socket with parsed UUID
8. Connect classic BT socket
9. Create and run `CXRSocketProtocol` over the socket
10. Protocol authenticates, then calls `onConnectionInfo` with UUID, MAC, device name, version
11. Callback propagates up: BluetoothController -> CxrController -> CxrApi -> BluetoothStatusCallback.onConnected()

### Connection Flow (connect/reconnect)

1. Store context, socketUuid, macAddress, callback
2. Get BluetoothAdapter, get remote device by MAC
3. Create RFCOMM socket with stored UUID
4. Connect socket
5. Run CXRSocketProtocol over socket
6. Same callback chain as init from step 10

---

## WifiController - Wi-Fi P2P Connection

**Package**: `com.rokid.cxr.client.extend.controllers`
**Singleton**: `WifiController.getInstance()`

Manages Wi-Fi Direct (P2P) connection to the glasses for high-bandwidth operations (file sync, APK upload).

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `Context` | Application context |
| `b` | `CxrStatus` | Current status |
| `c` | `WifiController.Callback` | Upstream callback |
| `d` | `String` | Device name to find |
| `e` | `Looper` | Handler looper |
| `f` | `Handler` | Handler for delayed operations |
| `g` | `WifiP2pManager` | Android P2P manager |
| `h` | `WifiP2pManager.Channel` | P2P channel |
| `i` | `WifiP2pDevice` | Found peer device |
| `j` | `WifiP2pInfo` | Connection info |
| `k` | `boolean` | Connect success flag (set by WifiController$a.onSuccess) |
| `l` | `boolean` | mHasInitiatedConnection - once true, ignores new peer lists |
| `m` | `Runnable` | Timeout runnable |
| `n` | `int` | Retry count |
| `o` | `boolean` | isConnected flag (set on WIFI_AVAILABLE, blocks new peer processing) |
| `p` | `WifiController$d` | BroadcastReceiver for P2P events |

### Methods

```java
public static WifiController getInstance()

// Initialize Wi-Fi P2P with device name to search for
public void init(Context context, String deviceName, WifiController.Callback callback)

// Check if P2P connection is established
public boolean isConnected()

// Disconnect and clean up
public synchronized CxrStatus deinit(CxrWifiErrorCode errorCode)
```

### WifiController.Callback Interface

```java
public interface WifiController.Callback {
    void onStatusUpdate(CxrStatus status, CxrWifiErrorCode errorCode)
    void onAddress(String ipAddress)  // Called with glasses IP when P2P connected
}
```

### Wi-Fi P2P Connection Flow

1. `init()` called with device name (from GlassInfo)
2. Get `WifiP2pManager` system service
3. Initialize P2P channel
4. Register BroadcastReceiver (`WifiController$d`) for P2P intents:
   - `WIFI_P2P_STATE_CHANGED_ACTION`
   - `WIFI_P2P_PEERS_CHANGED_ACTION`
   - `WIFI_P2P_CONNECTION_CHANGED_ACTION`
   - `WIFI_P2P_THIS_DEVICE_CHANGED_ACTION`
5. Start peer discovery via `WifiP2pManager.discoverPeers()`
6. On peers found, match by device name
7. Connect to matched peer via `WifiP2pManager.connect()`
8. On connection info available, extract group owner IP
9. Call `callback.onAddress(ipAddress)` - this IP is used as base URL for FileController
10. Call `callback.onStatusUpdate(WIFI_AVAILABLE, SUCCEED)`

### Inner Classes (Corrected from Bytecode)

- `WifiController$a` - ActionListener for `connect()` (method `g()`). On success: sets `k=true`. On failure: retries up to 3x via `i()`, then `deinit(WIFI_CONNECT_FAILED)`.
- `WifiController$b` - ActionListener for `removeGroup()` in `connectToDevice()` (when status IS 3). Both success and failure call `i()` (startConnect).
- `WifiController$c` - ActionListener for `stopPeerDiscovery()` in `handleWifiConnectionChanged`. Logs only, no retry logic.
- `WifiController$d` - BroadcastReceiver for P2P state changes. Routes to `handleWifiStateChanged`, `handleWifiPeersChanged` (`d()`), or `handleWifiConnectionChanged` (`a(NetworkInfo)`).
- `WifiController$e` - ActionListener for `discoverPeers()` in `connectP2pService()` (`b()`).
- `WifiController$f` - Singleton holder.

### Detailed Internal Flow (from Bytecode)

```
CxrApi.initWifiP2P(callback)
  -> sends "Sync_Start" via CxrController.request("Med", caps)
  -> glasses respond with device name
  -> CxrApi$a (response handler) calls WifiController.init(context, deviceName, internalCallback)

WifiController.init(context, deviceName, callback):
  1. deinit(SUCCEED)  -- cleans up previous session (sends Sync_Stop IF callback was set)
  2. Store context, deviceName, callback
  3. b() -- connectP2pService()

b() -- connectP2pService:
  1. Get WifiP2pManager system service
  2. Initialize WifiP2pManager.Channel
  3. Register BroadcastReceiver (WifiController$d) for:
     - WIFI_P2P_STATE_CHANGED_ACTION
     - WIFI_P2P_PEERS_CHANGED_ACTION
     - WIFI_P2P_CONNECTION_CHANGED_ACTION
  4. h() -- setupTimeout (15 seconds)
  5. discoverPeers()

On PEERS_CHANGED -> d() -> requestPeers -> a(WifiP2pDeviceList):
  1. Guard: if (o || l) -> "ignore new peers device list" -> return
     (o = isConnected, l = mHasInitiatedConnection)
  2. Iterate device list, match by deviceName (field d)
  3. If found: set i = device, set l = true, call c()

c() -- connectToDevice:
  1. Check i.status == 3 (AVAILABLE)
  2. If NOT available:
     - If n < 3: n++, post c() again in 1000ms
     - If n >= 3: deinit(WIFI_CONNECT_FAILED)
  3. If AVAILABLE:
     - n = 0
     - removeGroup(channel, WifiController$b)
     - WifiController$b calls i() on both success/failure

i() -- startConnect:
  1. post g() with 500ms delay

g() -- actual P2P connect:
  1. WifiP2pConfig: deviceAddress, WPS PBC (setup=0), groupOwnerIntent=15
  2. mgr.connect(channel, config, WifiController$a)
  3. WifiController$a.onSuccess: k = true
  4. WifiController$a.onFailure: retry via i() up to 3x, then deinit(WIFI_CONNECT_FAILED)

On CONNECTION_STATE_CHANGE -> a(NetworkInfo):
  1. If connected: stopPeerDiscovery, requestConnectionInfo
  2. onConnectionInfoAvailable -> a(WifiP2pInfo):
     - If groupFormed && groupOwnerAddress != null:
       cancelTimeout, callback.onAddress(ip), updateStatus(WIFI_AVAILABLE)
     - If groupOwnerAddress is null:
       deinit(WIFI_CONNECT_FAILED)

f() -- timeout (15s):
  1. deinit(WIFI_CONNECT_FAILED)

deinit(errorCode):
  1. cancelTimeout
  2. If callback != null: send "Sync_Stop" via CxrController
  3. cancelConnect + removeGroup on own channel
  4. Reset all fields, set status WIFI_UNAVAILABLE
```

### CRITICAL BUG: Stale P2P Device Status (Verified from Bytecode)

The SDK has a bug in `connectToDevice()` (method `c()`) that causes `WIFI_CONNECT_FAILED` when Android has stale persistent P2P groups from previous connection attempts.

**Verification status**: Fully confirmed by bytecode analysis of WifiController.class. Every claim below references specific bytecode offsets.

#### Root Cause

When `onPeersAvailable` (method `a(WifiP2pDeviceList)`) finds the matching device at offset 127-135, it:
1. Stores the `WifiP2pDevice` snapshot in field `i` (offset 135: `putfield i`)
2. Sets `l = true` (offset 170: `putfield l = iconst_1`)
3. Calls `c()` / connectToDevice (offset 173: `invokevirtual c()`)

The guard at the TOP of `onPeersAvailable` (offsets 50-63) checks:
```
50: getfield o    // isConnected flag
53: ifne 177      // if true, skip to "ignore"
57: getfield l    // mHasInitiatedConnection flag
60: ifeq 66       // if false, proceed to iterate
63: goto 177      // if true, skip to "ignore"
// offset 177: LogUtil.w("WifiController", "ignore new peers device list"); return;
```

Once `l=true`, ALL subsequent `PEERS_CHANGED` broadcasts are silently dropped. The `WifiP2pDevice` objects from `getDeviceList()` are Java object snapshots -- the `status` field in field `i` never updates unless a new peer list is processed, which is blocked by `l=true`.

The retry in `c()` (offset 63: `InvokeDynamic #2` confirmed as `REF_invokeVirtual WifiController.c:()V` from BootstrapMethods table) re-enters `c()` after 1000ms delay. It re-reads `i.status` (offsets 9-12) but gets the same stale value. After 3 retries (checked at offset 47: `if n >= 3`), it calls `deinit(WIFI_CONNECT_FAILED)`.

Field `i` is written in exactly 2 places in the entire class: `onPeersAvailable` (set to matched device) and `deinit()` (set to null). No other method updates it.

#### Reset Paths (Nuance)

There ARE paths that reset `l=false`:
- `onConnectionInfoAvailable` at offset 17: `putfield l = false` (fires when P2P actually connects)
- `handleWifiConnectionChanged` at offset 112: `putfield l = false` (fires on disconnect)
- `deinit()` at offset 226: `putfield l = false` (full reset)

However, in the failure case (device stuck as non-AVAILABLE), the P2P connection never forms, so `onConnectionInfoAvailable` never fires during the 3-second retry window. The bug window is: device found with stale status -> l=true -> 3 retries at 1s -> deinit(WIFI_CONNECT_FAILED). During these 3 seconds, any `PEERS_CHANGED` broadcasts that might report the device as now-AVAILABLE are dropped.

#### Trigger

Android's wpa_supplicant maintains a persistent P2P group database. After a failed WiFi Direct negotiation (common during development), the glasses' P2P device gets cached with status `INVITED` (1) or `FAILED` (2) instead of `AVAILABLE` (3). This status persists across app restarts and survives `cancelConnect()` + `removeGroup()` calls.

#### Symptom

`initWifiP2P` returns `REQUEST_SUCCEED`, the SDK discovers the glasses P2P device and sets `l=true`, but `connectToDevice()` finds `status != 3`, retries 3 times checking the same stale `WifiP2pDevice.status` value, then reports `WIFI_CONNECT_FAILED`.

#### Workaround

Before calling `CxrApi.initWifiP2P()`, delete ALL persistent P2P groups using the hidden `WifiP2pManager.deletePersistentGroup()` API via reflection:

```kotlin
fun cleanupP2pState(context: Context) {
    val mgr = context.getSystemService(Context.WIFI_P2P_SERVICE) as? WifiP2pManager ?: return
    val ch = mgr.initialize(context, Looper.getMainLooper(), null) ?: return

    mgr.stopPeerDiscovery(ch, null)
    mgr.cancelConnect(ch, null)
    mgr.removeGroup(ch, null)

    // Delete ALL persistent P2P groups (hidden API, clears wpa_supplicant cache)
    try {
        val method = WifiP2pManager::class.java.getMethod(
            "deletePersistentGroup",
            WifiP2pManager.Channel::class.java,
            Int::class.javaPrimitiveType,
            WifiP2pManager.ActionListener::class.java
        )
        for (netId in 0..31) {
            method.invoke(mgr, ch, netId, null)
        }
    } catch (_: Exception) {}

    // Wait 5+ seconds before calling CxrApi.initWifiP2P()
}
```

This ensures the phone discovers the glasses P2P device with a fresh `AVAILABLE` (3) status.

---

## FileController - File Sync and APK Upload

**Package**: `com.rokid.cxr.client.extend.controllers`
**Singleton**: `FileController.getInstance()`

Handles HTTP-based file operations with the glasses HTTP server running on **port 8848** over Wi-Fi P2P.

### Static Constants

```java
// Media paths on glasses filesystem
// Field `t` contains default media directory paths used for file listing
public static final String[] t  // Media type path templates
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `Context` | Application context |
| `b` | `String` | Save path on phone |
| `c` | `String[]` | Media paths to sync |
| `d` | `String` | Single file path filter |
| `e` | `boolean` | Download in progress |
| `f` | `FileController.Callback` | Download callback |
| `g` | `int` | Current download index |
| `h` | `boolean` | Fetch file list in progress |
| `i` | `Call<FileListResponse>` | File list Retrofit call |
| `j` | `boolean` | Download file in progress |
| `k` | `Call<ResponseBody>` | Download file Retrofit call |
| `l` | `boolean` | Report download in progress |
| `m` | `Call<ResponseBody>` | Report download Retrofit call |
| `n` | `boolean` | Delete file in progress |
| `o` | `Call<ResponseBody>` | Delete file Retrofit call |
| `p` | `boolean` | APK upload in progress |
| `q` | `ApkStatusCallback` | APK status callback |
| `r` | `Call<ResponseBody>` | Upload APK Retrofit call |
| `s` | `boolean` | Upload in progress |

### Methods

```java
public static FileController getInstance()

// Generate media directory paths for given media types
public static String[] generateMediaPaths(CxrMediaType[] types)

// Start downloading media files from glasses
// Sets up RetrofitClient base URL as http://<address>:8848
// Fetches file list, then downloads each file
public void startDownload(Context context, String savePath, CxrMediaType[] types,
    String singleFilePath, String address, FileController.Callback callback)

// Cancel ongoing downloads
public void stopDownload()

// Upload APK file to glasses
// Base URL: http://<address>:8848
// Uses multipart upload via RetrofitService.uploadFile()
// Content-Type: application/vnd.android.package-archive
public void startUploadApk(File apkFile, String address, ApkStatusCallback callback)

// Cancel APK upload
public void stopUploadApk()
```

### FileController.Callback Interface

```java
public interface FileController.Callback {
    void onDownloadStart()
    void onSingleFileDownloaded(String fileName)
    void onDownloadFailed()
    void onDownloadFinished()
}
```

### HTTP Server Details (Port 8848)

The glasses run an HTTP server on port 8848 (accessible via Wi-Fi P2P). The RetrofitService defines:

```java
public interface RetrofitService {
    // Get file listing from glasses
    Call<FileListResponse> getFileList(RequestBody body)

    // Report that a file was downloaded (so glasses can track)
    Call<ResponseBody> reportDownload(RequestBody body)

    // Download a file from glasses
    Call<ResponseBody> downloadFile(RequestBody body)

    // Delete a file on glasses
    Call<ResponseBody> deleteFile(RequestBody body)

    // Upload APK file to glasses (multipart)
    Call<ResponseBody> uploadFile(MultipartBody.Part file)
}
```

**Retrofit Client Configuration**:
- Base URL: `http://<glasses_ip>:8848`
- Connect timeout: 60 seconds
- Read timeout: 5 minutes
- Write timeout: 5 minutes
- No cache
- Header interceptor adds version "1.0" headers
- Uses Gson converter factory

### APK Upload Flow

1. `CxrApi.startUploadApk(wifiAddress, callback)` is called
2. Checks `isWifiP2PConnected()` (required)
3. `FileController.startUploadApk(file, address, callback)`
4. Sets RetrofitClient base URL to `http://<address>:8848`
5. Creates `RequestBody` with MIME type `application/vnd.android.package-archive`
6. Creates `MultipartBody.Part` with form name "file" and original filename
7. Calls `RetrofitService.uploadFile(part)`
8. On success: `ApkStatusCallback.onUploadApkSucceed()` followed by install callbacks
9. On failure: `ApkStatusCallback.onUploadApkFailed()`

### File Sync Flow

1. `CxrApi.startSync(savePath, types, callback)` is called
2. Checks `isWifiP2PConnected()` and validates params
3. Creates save directory if needed
4. `FileController.startDownload(context, savePath, types, null, address, internalCallback)`
5. `generateMediaPaths()` creates filesystem paths for requested media types
6. For each media path, calls `getFileList()` to get available files
7. For each file, calls `downloadFile()` and saves to local path
8. After each file: `reportDownload()` and optionally `deleteFile()`
9. Callbacks: `onDownloadStart()`, `onSingleFileDownloaded(name)`, `onDownloadFinished()`

---

## AudioController - Communication Device Routing

**Package**: `com.rokid.cxr.client.controllers`
**Singleton**: `AudioController.getInstance()`

Routes phone audio output/input to the Bluetooth SCO (glasses speaker/mic).

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `AudioManager` | Android AudioManager |

### Methods

```java
public static AudioController getInstance()

// Set glasses as communication device via AudioManager
public void setCommunicationDevice(Context context)

// Clear communication device routing
public void clearCommunicationDevice()
```

---

## CXRSocketProtocol - Low-Level Bluetooth Protocol

**Package**: `com.rokid.cxr`

Native JNI-backed class that handles the binary framing protocol over the Bluetooth classic socket. All data is serialized using the Caps format.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `a` | `String` | Tag for logging |
| `b` | `BluetoothSocket` | The BT socket |
| `c` | `InputStream` | Socket input stream |
| `d` | `OutputStream` | Socket output stream |
| `e` | `Runnable` | Read loop runnable |
| `f` | `long` | Native handle (JNI pointer) |
| `g` | `boolean` | Running flag |
| `h` | `boolean` | Connected flag |
| `i` | `CXRSocketProtocol.Callback` | Protocol callback |
| `j` | `int` | Version |

### Constructor

```java
public CXRSocketProtocol(String tag, boolean param, long timeout, int version)
```

### Methods

```java
// Start protocol over a Bluetooth socket
// Creates native protocol handler, starts read loop thread
// Authenticates with glasses using socket UUID
public boolean run(BluetoothSocket socket, UUID uuid,
    CXRSocketProtocol.Callback callback, boolean param1, boolean param2)

// Send a request (with request ID for response tracking)
public boolean request(int reqId, String cmd, Caps args)

// Send data with binary payload
public boolean send(String cmd, Caps caps, byte[] data)

// Audio record operations
public boolean openAudioRecord(int codec, String cmd, Caps args)
public boolean closeAudioRecord(String cmd)

// Update Rokid account
public void changeRokidAccount(String account)

// Get protocol version
public int version()
```

### Native Methods

```java
private native long nativeCreate(CXRSocketProtocol.Callback callback)
private native void nativeDestroy(long handle)
private native void nativeClose(long handle)
private native boolean nativeRequest(long handle, int reqId, String cmd, Caps args)
private native boolean nativeSend(long handle, String cmd, Caps caps, byte[] data)
private native int nativeHandleReadPacket(long handle, byte[] buffer, int offset, int length)
private native boolean nativeAuth(long handle, String uuid)
private native void nativeChangeRokidAccount(long handle, String account)
private boolean sendFragment(byte[] data, int offset, int length)
```

### CXRSocketProtocol.Callback Interface

```java
public interface CXRSocketProtocol.Callback {
    // Response to a request (matched by reqId)
    void onResponse(int reqId, Caps responseArgs)

    // Notification from glasses (no request ID)
    void onNotify(String cmd, Caps args)

    // Received data with binary payload
    void onReceived(String cmd, Caps args, byte[] data)

    // Audio stream started
    void onStartAudioStream(int codecType, String streamType, Caps args)

    // Audio stream data
    void onAudioStream(byte[] data, int offset, int length)

    // ARTC video frame received
    void onARTCFrame(byte[] frameData)

    // Socket disconnected
    void onDisconnect()
}
```

---

## CXRServiceBridge - Glasses-Side Bridge (CXR-S)

**Package**: `com.rokid.cxr`

This class is the glasses-side bridge for CXR-S SDK. It is included in the CXR-M SDK package but is intended for use on the glasses OS (YodaOS-Sprite).

### Constants

```java
public static final int EINVAL   // Invalid parameter error
public static final int EDUP     // Duplicate subscription error
public static final int EFAULT   // Internal fault error
public static final int EBUSY    // Resource busy error
```

### Methods

```java
// Set connection status listener
public synchronized void setStatusListener(StatusListener listener)

// Subscribe to named message channel (one-way)
public int subscribe(String channel, MsgCallback callback)

// Subscribe to named message channel (with reply capability)
public int subscribe(String channel, MsgReplyCallback callback)

// Send message on a channel
public native int sendMessage(String channel, Caps args)

// Send message with binary payload
public int sendMessage(String channel, Caps args, byte[] data)
public native int sendMessage(String channel, Caps args, byte[] data, int offset, int length)

// Disconnect the CXR device
public native void disconnectCXRDevice()

// Start audio streaming
public native int startAudioStream(int codecType, String streamType, Caps args)

// Stop audio streaming
public native void stopAudioStream(String streamType)

// Start Bluetooth pairing mode
public native void startBTPairing()

// Send ARTC video frame
public native int sendARTCFrame(byte[] data, int offset, int length, boolean isKeyFrame, long timestamp)
```

### CXRServiceBridge.StatusListener Interface

```java
public interface StatusListener {
    int DEVICE_TYPE_UNKNOWN = 0
    int DEVICE_TYPE_ANDROID = 1
    int DEVICE_TYPE_IOS = 2

    void onConnected(String deviceInfo, int deviceType)
    void onDisconnected()
    void onARTCStatus(float quality, boolean isHealthy)
    void onRokidAccountChanged(String account)
}
```

### CXRServiceBridge.MsgCallback Interface

```java
public interface MsgCallback {
    void onReceive(String channel, Caps args, byte[] data)
}
```

### CXRServiceBridge.MsgReplyCallback Interface

```java
public interface MsgReplyCallback {
    void onReceive(String channel, Caps args, byte[] data, Reply reply)
}
```

### CXRServiceBridge.Reply Interface

```java
public interface Reply {
    void end(Caps responseArgs)
}
```

---

## Caps - Serialization Format

**Package**: `com.rokid.cxr`

Binary serialization format used for all Bluetooth protocol messages. Supports ordered typed values, similar to a typed tuple.

### Constructor and Factory

```java
public Caps()
public static Caps fromBytes(byte[] data, int offset, int length)
public static Caps fromBytes(byte[] data)
```

### Write Methods (append values in order)

```java
public void write(boolean value)      // TYPE_VOID used for booleans
public void writeInt32(int value)     // Signed 32-bit integer
public void writeUInt32(int value)    // Unsigned 32-bit integer
public void writeInt64(long value)    // Signed 64-bit integer
public void writeUInt64(long value)   // Unsigned 64-bit integer
public void writeFloat(float value)   // 32-bit float
public void writeDouble(double value) // 64-bit double
public void write(String value)       // UTF-8 string
public void write(byte[] data)        // Binary blob
public void write(byte[] data, int offset, int length)  // Binary blob with range
public void write(Caps nested)        // Nested Caps object
public void writeVoid()               // Void/null marker
```

### Read Methods

```java
public int size()           // Number of values
public Caps.Value at(int index)  // Get value at index
public void clear()         // Clear all values
```

### Serialization (Native)

```java
public native byte[] serialize()
public native boolean parse(byte[] data, int offset, int length)
public boolean parse(byte[] data)
```

### Caps.Value Class

```java
public class Caps.Value {
    // Type constants
    public static char TYPE_VOID    // Boolean/void
    public static char TYPE_INT32   // Signed 32-bit int
    public static char TYPE_UINT32  // Unsigned 32-bit int
    public static char TYPE_INT64   // Signed 64-bit long
    public static char TYPE_UINT64  // Unsigned 64-bit long
    public static char TYPE_FLOAT   // 32-bit float
    public static char TYPE_DOUBLE  // 64-bit double
    public static char TYPE_STRING  // String
    public static char TYPE_BINARY  // Binary data
    public static char TYPE_OBJECT  // Nested Caps

    public char type()
    public Object getValueNoType()
    public int getInt()
    public long getLong()
    public float getFloat()
    public double getDouble()
    public String getString()
    public Caps.Binary getBinary()
    public Caps getObject()
}
```

### Caps.Binary Class

```java
public class Caps.Binary {
    public byte[] data
    public int offset
    public int length

    public Caps.Binary(byte[] data, int offset, int length)
}
```

### Caps.IncorrectTypeException

```java
public class Caps.IncorrectTypeException extends RuntimeException
// Thrown when calling a getter for the wrong type (e.g., getInt() on a String value)
```

### Typical Caps Message Structure

Most messages follow this pattern:
```
Caps message:
  [0] String: command name (e.g., "Dev_GetGlassInfo", "ASR_Result")
  [1] String: JSON payload (optional, depends on command)
```

For responses:
```
Caps response:
  [0] String: command name (echo of request)
  [1] String: JSON response body
```

---

## Enums

### CxrStatus

```java
public enum ValueUtil.CxrStatus {
    BLUETOOTH_AVAILABLE(0),
    BLUETOOTH_UNAVAILABLE(1),
    BLUETOOTH_INIT(2),
    WIFI_AVAILABLE(3),
    WIFI_UNAVAILABLE(4),
    WIFI_INIT(5),
    REQUEST_SUCCEED(6),
    REQUEST_FAILED(7),
    REQUEST_WAITING(8),
    RESPONSE_SUCCEED(9),
    RESPONSE_INVALID(10),
    RESPONSE_TIMEOUT(11);

    public int getStatus()
}
```

### CxrBluetoothErrorCode

```java
public enum ValueUtil.CxrBluetoothErrorCode {
    SUCCEED(0),
    PARAM_INVALID(1),
    BLE_CONNECT_FAILED(2),
    SOCKET_CONNECT_FAILED(3),
    SN_CHECK_FAILED(4),
    UNKNOWN(5);

    public int getErrorCode()
}
```

### CxrWifiErrorCode

```java
public enum ValueUtil.CxrWifiErrorCode {
    SUCCEED(0),
    WIFI_DISABLED(1),
    WIFI_CONNECT_FAILED(2),
    UNKNOWN(3);

    public int getErrorCode()
}
```

### CxrMediaType

```java
public enum ValueUtil.CxrMediaType {
    AUDIO(0),
    PICTURE(1),
    VIDEO(2),
    ALL(3);

    public int getType()
}
```

### CxrNotifyType

```java
public enum ValueUtil.CxrNotifyType {
    UNKNOWN(0),   // Fire-and-forget (no callback expected)
    REQUEST(1),   // Expects response (has RequestCallback)
    NOTIFY(2);    // Notification type

    public int getType()
}
```

### CxrSceneType

```java
public enum ValueUtil.CxrSceneType {
    AI_CHAT("ai_chat"),
    TRANSLATE("translate"),
    AUDIO_RECORD("audio_record"),
    VIDEO_RECORD("video_record"),
    WORD_TIPS("word_tips"),
    NAVIGATION("navigation");

    public String getSceneId()
}
```

### CxrSendErrorCode

```java
public enum ValueUtil.CxrSendErrorCode {
    UNKNOWN(0);

    public int getErrorCode()
}
```

### CxrStreamType

```java
public enum ValueUtil.CxrStreamType {
    WORD_TIPS(0);

    public int getType()
}
```

---

## Callback Interfaces

All in package `com.rokid.cxr.client.extend.callbacks`.

### ApkStatusCallback

```java
public interface ApkStatusCallback {
    void onUploadApkSucceed()
    void onUploadApkFailed()
    void onInstallApkSucceed()
    void onInstallApkFailed()
    void onUninstallApkSucceed()
    void onUninstallApkFailed()
    void onOpenAppSucceed()
    void onOpenAppFailed()
}
```

### AudioSceneIdCallback

```java
public interface AudioSceneIdCallback {
    void onAudioSceneId(int sceneId, boolean success)
}
```

### BluetoothStatusCallback

```java
public interface BluetoothStatusCallback {
    // Note: decompiled version has 4 params, not 2 as in original docs
    void onConnectionInfo(String socketUuid, String macAddress, String deviceName, int protocolVersion)
    void onConnected()
    void onDisconnected()
    void onFailed(CxrBluetoothErrorCode errorCode)
}
```

### GlassInfoResultCallback

```java
public interface GlassInfoResultCallback {
    void onGlassInfoResult(CxrStatus status, GlassInfo info)
}
```

### GlassVersionCallback

```java
public interface GlassVersionCallback {
    void onGlassVersion(boolean hasUpdate, String versionInfo)
}
```

### PhotoPathCallback

```java
public interface PhotoPathCallback {
    void onPhotoPath(CxrStatus status, String filePath)
}
```

### PhotoResultCallback

```java
public interface PhotoResultCallback {
    void onPhotoResult(CxrStatus status, byte[] photoData)
}
```

### SendStatusCallback

```java
public interface SendStatusCallback {
    void onSendSucceed()
    void onSendFailed(CxrSendErrorCode errorCode)
}
```

### SyncStatusCallback

```java
public interface SyncStatusCallback {
    void onSyncStart()
    void onSingleFileSynced(String fileName)
    void onSyncFailed()
    void onSyncFinished()
}
```

### UnsyncNumResultCallback

```java
public interface UnsyncNumResultCallback {
    // audioNum, pictureNum, videoNum
    void onUnsyncNumResult(CxrStatus status, int audioNum, int pictureNum, int videoNum)
}
```

### WifiListCallback

```java
public interface WifiListCallback {
    void onWifiList(CxrStatus status, List<RKWifiInfo> wifiList)
}
```

### WifiP2PStatusCallback

```java
public interface WifiP2PStatusCallback {
    void onConnected()
    void onDisconnected()
    void onFailed(CxrWifiErrorCode errorCode)
}
```

---

## Listener Interfaces

All in package `com.rokid.cxr.client.extend.listeners`.

### AiEventListener

```java
public interface AiEventListener {
    void onAiKeyDown()   // AI button pressed on glasses
    void onAiKeyUp()     // AI button released
    void onAiExit()      // AI session exited on glasses
}
```

### ArtcListener

```java
public interface ArtcListener {
    void onArtcStart()         // ARTC session started
    void onArtcStop()          // ARTC session stopped
    void onArtcFrame(byte[] frameData)  // ARTC video frame received
}
```

### AudioStreamListener

```java
public interface AudioStreamListener {
    void onStartAudioStream(int codecType, String streamType)
    void onAudioStream(byte[] data, int offset, int length)
}
```

### BatteryLevelUpdateListener

```java
public interface BatteryLevelUpdateListener {
    // Note: decompiled shows (int level, boolean isCharging), not (String) as in docs
    void onBatteryLevelUpdated(int level, boolean isCharging)
}
```

### BrightnessUpdateListener

```java
public interface BrightnessUpdateListener {
    // Note: decompiled shows (int brightness), not (String) as in docs
    void onBrightnessUpdated(int brightness)
}
```

### CustomCmdListener

```java
public interface CustomCmdListener {
    void onCustomCmd(String cmd, Caps args)
}
```

### CustomViewListener

```java
public interface CustomViewListener {
    void onIconsSent()         // Icons successfully sent to glasses
    void onOpened()            // Custom view opened
    void onOpenFailed(int errorCode)  // Custom view open failed
    void onUpdated()           // Custom view updated
    void onClosed()            // Custom view closed
}
```

### MediaFilesUpdateListener

```java
public interface MediaFilesUpdateListener {
    void onMediaFilesUpdated()
}
```

### SceneStatusUpdateListener

```java
public interface SceneStatusUpdateListener {
    void onSceneStatusUpdated(SceneStatusInfo info)
}
```

### ScreenStatusUpdateListener

```java
public interface ScreenStatusUpdateListener {
    void onScreenStatusUpdated(boolean isScreenOn)
}
```

### TranslationListener

```java
public interface TranslationListener {
    void onTranslationStart()
    void onTranslationStop()
}
```

### VolumeUpdateListener

```java
public interface VolumeUpdateListener {
    // Note: decompiled shows (int volume), not (String) as in docs
    void onVolumeUpdated(int volume)
}
```

---

## Info Classes (Data Models)

All in package `com.rokid.cxr.client.extend.infos`.

### GlassInfo

Complete glasses hardware and software information, deserialized from JSON via Gson.

```java
public class GlassInfo {
    String deviceName        // a - Glasses device name
    int    batteryLevel      // b - Battery percentage
    boolean isCharging       // c - Currently charging
    String devicePanel       // d - Display panel identifier
    int    brightness        // e - Current brightness [0-15]
    int    volume            // f - Current volume [0-15]
    String wearingStatus     // g - Whether glasses are being worn
    String deviceKey         // h - IoT device key
    String deviceSecret      // i - IoT device secret
    String deviceTypeId      // j - Device type identifier
    String deviceId          // k - Unique device identifier (SN)
    String deviceSeed        // l - Device seed value
    String otaCheckUrl       // m - OTA update check URL
    String otaCheckApi       // n - OTA API endpoint
    String assistVersionName // o - Assist service version name
    long   assistVersionCode // p - Assist service version code
    String systemVersion     // q - YodaOS system version

    // All fields have getters and setters
}
```

### IconInfo

Icon data for custom views on glasses.

```java
public class IconInfo {
    String name   // a - Icon name
    String data   // b - Icon data (base64 encoded image)

    public IconInfo(String name, String data)
}
```

### RKAppInfo

Application info for launching apps on glasses.

```java
public class RKAppInfo {
    String packageName   // a - Android package name
    String activityName  // b - Main activity class name

    public RKAppInfo(String packageName, String activityName)
}
```

### RKWifiInfo

Wi-Fi network info from glasses.

```java
public class RKWifiInfo {
    String name    // a - SSID
    int    signal  // b - Signal strength

    // Default constructor + getters/setters
}
```

### SceneStatusInfo

Status of all running scenes on the glasses.

```java
public class SceneStatusInfo {
    boolean isAiAssistRunning    // a
    boolean isAiChatRunning      // b
    boolean isAudioRecordRunning // c
    boolean isBrightnessRunning  // d
    boolean isHasDisplay         // e
    boolean isNavigationRunning  // f
    boolean isNotesRunning       // g
    boolean isOtaRunning         // h
    boolean isPaymentRunning     // i
    boolean isPhoneCallRunning   // j
    boolean isTranslateRunning   // k
    boolean isVideoRecordRunning // l
    boolean isWordTipsRunning    // m
    boolean isCustomViewRunning  // n

    // All fields have getters and setters
}
```

### ScheduleInfo

Calendar/reminder data for glasses.

```java
public class ScheduleInfo {
    long   id             // a - Schedule entry ID
    String title          // b - Schedule title
    String description    // c - Schedule description
    long   scheduleTime   // d - Scheduled timestamp (millis)
    long   reminderTime   // e - Reminder timestamp (millis)

    public ScheduleInfo(long id)
    public ScheduleInfo(long id, String title, String description, long scheduleTime, long reminderTime)
}
```

---

## Sync / Retrofit Classes

All in package `com.rokid.cxr.client.extend.sync`.

### RetrofitClient

Singleton HTTP client for communicating with the glasses HTTP server.

```java
public class RetrofitClient {
    RetrofitService a  // Retrofit service interface instance

    public static RetrofitClient getInstance()

    // Create text/plain request body
    public static RequestBody createPartFromString(String value)

    // Create application/vnd.android.package-archive request body for APK files
    public static RequestBody createPartFromApk(File file)

    // Set base URL (typically http://<glasses_ip>:8848)
    // Creates OkHttpClient with:
    //   - HttpLoggingInterceptor (level NONE)
    //   - HeaderInterceptor (version "1.0", "1.0")
    //   - Connect timeout: 60 seconds
    //   - Read timeout: 5 minutes
    //   - Write timeout: 5 minutes
    //   - No cache
    //   - Gson converter factory
    public void setBaseUrl(String baseUrl)

    public RetrofitService getService()
}
```

### RetrofitService

Retrofit interface defining all HTTP endpoints on the glasses server.

```java
public interface RetrofitService {
    Call<FileListResponse> getFileList(RequestBody body)
    Call<ResponseBody> reportDownload(RequestBody body)
    Call<ResponseBody> downloadFile(RequestBody body)
    Call<ResponseBody> deleteFile(RequestBody body)
    Call<ResponseBody> uploadFile(MultipartBody.Part file)
}
```

### HeaderInterceptor

OkHttp interceptor that adds version headers to requests.

```java
public class HeaderInterceptor implements Interceptor {
    String a  // version1
    String b  // version2

    public HeaderInterceptor(String version1, String version2)
    public Response intercept(Interceptor.Chain chain)
}
```

### BaseNetworkResponse

Base class for HTTP responses.

```java
public abstract class BaseNetworkResponse {
    int errorCode
    String errorMsg
    boolean isSuccess
}
```

### FileListResponse

Response from `getFileList()`.

```java
public class FileListResponse extends BaseNetworkResponse {
    List<FileData> data
}
```

### FileData

File metadata from glasses filesystem.

```java
public class FileData {
    final String absoluteFilePath
    final long createDate
    final String fileName
    final long fileSize
    final boolean isDir
    final String mimeType
    final long modifiedDate
    final String webFilePath      // URL path for downloading
    final List<FileData> childList // Sub-files if this is a directory
}
```

---

## Version Checking

### CheckUtil

**Package**: `com.rokid.cxr.client.extend.version`
**Singleton**: `CheckUtil.getInstance()`

Checks for glasses firmware updates using the OTA URL from GlassInfo.

```java
public class CheckUtil {
    PrintWriter a
    BufferedReader b
    HttpURLConnection c
    GlassInfo d

    public static CheckUtil getInstance()

    // Check glasses version against OTA server
    // Returns JSON string with imageUrl and version info
    // Version format regex: \d+\.\d+\.\d+-\d+-\d+
    public synchronized String checkGlassVersion(GlassInfo info)
}
```

### Md5Util

```java
public class Md5Util {
    public static String getMd5(String input)
}
```

---

## Utility Classes

### LogUtil

**Package**: `com.rokid.cxr.client.utils`

Logging utility wrapping Android Log with configurable log level.

```java
public class LogUtil {
    public static final int VERBOSE = 1
    public static final int DEBUG = 2
    public static final int INFO = 3
    public static final int WARN = 4
    public static final int ERROR = 5

    static int a  // Current log level (default: 3/INFO, set in CxrController constructor)

    public static void setLogLevel(int level)
    public static void v(String tag, String msg)
    public static void d(String tag, String msg)
    public static void i(String tag, String msg)
    public static void w(String tag, String msg)
    public static void e(String tag, String msg)
    public static String getStackTrace(Exception e)
}
```

### Constants

**Package**: `com.rokid.cxr.client.extend`

```java
public class Constants {
    public static final String MED          // Media channel name
    public static final String MED_SYNC_STOP  // Media sync stop command
}
```

### BuildConfig Classes

Three BuildConfig classes for the three library modules:

```java
// com.rokid.cxr.BuildConfig (core native layer)
public final class BuildConfig {
    boolean DEBUG
    String LIBRARY_PACKAGE_NAME
    String BUILD_TYPE
}

// com.rokid.cxr.client.BuildConfig (controller layer)
public final class BuildConfig {
    boolean DEBUG
    String LIBRARY_PACKAGE_NAME
    String BUILD_TYPE
    String BUILD_TIME          // "2026-02-02 11:37:12"
    String VERSION_CODE        // "108"
    String VERSION_NAME        // "1.0.8"
}

// com.rokid.cxr.client.extend.BuildConfig (extend/API layer)
public final class BuildConfig {
    boolean DEBUG
    String LIBRARY_PACKAGE_NAME
    String BUILD_TYPE
    String BUILD_TIME          // "2026-02-02 11:37:58"
    String VERSION_CODE        // "108"
    String VERSION_NAME        // "1.0.8"
}
```

---

## Obfuscated Classes

### a.a (StringBuilder Helper)

**Package**: `a`

Simple utility for creating StringBuilder with initial string.

```java
public final class a.a {
    // Creates new StringBuilder and appends the given prefix
    public static StringBuilder a(String prefix)
    // Equivalent to: new StringBuilder().append(prefix)
}
```

Used throughout the SDK for log message building.

### b.a (FileController Delete Callback)

**Package**: `b`

Retrofit callback for the `deleteFile()` call in FileController.

```java
public final class b.a implements Callback<ResponseBody> {
    FileController a  // Parent controller reference

    public b.a(FileController controller)

    // Handles delete response - logs result, resets `FileController.p` flag
    public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response)

    // Handles delete failure - logs error, resets `FileController.p` flag
    public void onFailure(Call<ResponseBody> call, Throwable t)
}
```

### c.a (StringBuilder Multi-Append Helper)

**Package**: `c`

Utility for building strings with separator characters.

```java
public final class c.a {
    // Appends: str1 + separator char + str2 to the StringBuilder
    public static StringBuilder a(StringBuilder sb, String str1, char separator, String str2)
}
```

---

## Message Protocol - Channel Names and Commands

All Bluetooth communication uses named channels with Caps-serialized messages. The first value in the Caps is always the command string.

### Channel: "Dev" (Device Control)

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `Dev_TimeUpdate` | Phone -> Glasses | JSON: `{timeZone, time}` | Sync time |
| `Dev_GetGlassInfo` | Phone -> Glasses | (none) | Request glasses info |
| `Dev_GlassBrightness` | Phone -> Glasses | UInt32: brightness | Set brightness |
| `Dev_GlassSound` | Phone -> Glasses | UInt32: volume | Set volume |

### Channel: "Ai" (AI Interaction)

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `Ai_Start` | Phone -> Glasses | (none) | Notify AI started |
| `Ai_Heartbeat` | Phone -> Glasses | JSON: `{count}` | AI keepalive |
| `ASR_Result` | Phone -> Glasses | String: text | ASR recognition result |
| `ASR_None` | Phone -> Glasses | (none) | No ASR result |
| `ASR_End` | Phone -> Glasses | (none) | ASR finished |
| `ASR_Error` | Phone -> Glasses | (none) | ASR error |
| `TTS_Result` | Phone -> Glasses | String: text | TTS content |
| `TTS_AudioFinished` | Phone -> Glasses | (none) | TTS playback done |
| `Ai_OpenCamera` | Phone -> Glasses | JSON: `{width, height, quality}` | Open camera |
| `Ai_TakePhoto` | Phone -> Glasses | JSON: `{width, height, quality}` | Take photo |
| `Exit` | Phone -> Glasses | Int32: sessionCount | Exit AI session |
| `NoNetwork` | Phone -> Glasses | (none) | No network available |
| `Pic_UploadError` | Phone -> Glasses | (none) | Photo upload failed |
| `Ai_Error` | Phone -> Glasses | (none) | AI request error |

### Channel: "Med" (Media)

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `UnSync_Count` | Phone -> Glasses | (none) | Get unsync file counts |
| `Sync_Start` | Phone -> Glasses | JSON: `{type: "Android"}` | Start Wi-Fi P2P sync |
| `Med_Take_Photo_Url` | Phone -> Glasses | JSON: `{width, height, quality}` | Take photo (URL result) |

### Channel: "Sys" (System)

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `WordTips_SetTvConfig` | Phone -> Glasses | JSON config | Configure word tips display |
| `WordTips_AiContent` | Phone -> Glasses | JSON: `{asr}` | Word tips content |
| `WordTips_SendData` | Phone -> Glasses | JSON + binary | Word tips file data |

### Channel: "Trans" (Translation)

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `Trans_SetTvConfig` | Phone -> Glasses | JSON config | Configure translation display |
| `Trans_Result` | Phone -> Glasses | JSON: `{id, subId, temporary, finished, result}` | Translation content |

### Channel: "Settings"

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `Settings_Update` | Phone -> Glasses | JSON array: `[{key, value}]` | Update settings |

Settings keys:
- `settings_photo_width` - Photo width
- `settings_photo_height` - Photo height
- `settings_video_duration` - Video duration
- `settings_video_fps` - Video FPS
- `settings_video_width` - Video width
- `settings_video_height` - Video height
- `settings_video_duration_unit` - Duration unit
- `settings_sound_effect` - Sound effect mode
- `settings_screen_offTimeout` - Screen off timeout (ms)

### Channel: "ARTC" (Audio-RTC)

| Command | Direction | Data | Description |
|---------|-----------|------|-------------|
| `OnStop` | Phone -> Glasses | (none) | Stop ARTC |

---

## SN Verification and AES Encryption

The SDK includes a device serial number (SN) verification mechanism used during Bluetooth reconnection.

### How it works

1. During `connectBluetooth()`, the caller provides `snEncryptContent` (byte[]) and `clientSecret` (String)
2. These are stored in CxrApi fields `z` (snEncryptContent) and `A` (clientSecret with dashes removed)
3. After Bluetooth connects and `getGlassInfo()` returns, the SDK performs SN verification:
   - Uses AES/CBC/PKCS5Padding decryption
   - Key: `clientSecret` bytes (dashes stripped)
   - IV: first 16 bytes of `clientSecret` bytes
   - Decrypts `snEncryptContent`
   - Checks if decrypted result contains the glasses `deviceId` (from GlassInfo)
4. If SN check passes: `BluetoothStatusCallback.onConnected()` is called
5. If SN check fails: `BluetoothStatusCallback.onFailed(SN_CHECK_FAILED)` is called, and Bluetooth is disconnected after 500ms delay

### First-connection behavior

On the first connection (via `initBluetooth()` with a BluetoothDevice), the SN check is skipped. The `y` flag is set to `true`, and after getGlassInfo returns, it:
1. Calls `setGlassTime()` to sync time
2. Sets `y = false`
3. Calls `onConnected()` directly without SN verification

The SN verification is only applied during `connectBluetooth()` reconnection flow.
