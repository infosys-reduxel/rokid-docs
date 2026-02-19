# RokidOtaUpgrade

System OTA update application for Rokid AR glasses and station devices. Handles firmware version checking, image downloading, A/B partition updating, and device reboot.

## Package Info

| Field | Value |
|---|---|
| Package | `com.rokid.glass.ota` |
| Version | 2.9.0 (code 2903) |
| Min SDK | 24 (Android 7.0) |
| Target SDK | 33 (Android 13) |
| Shared UID | `android.uid.system` (runs as system app) |
| Persistent | Yes (auto-restarts if killed) |
| Build Channel | `master` |
| Native Libs | `libpl_droidsonroids_gif.so` (arm64-v8a, armeabi-v7a) |

## Permissions

### Standard Android Permissions

| Permission | Purpose |
|---|---|
| `INTERNET` | Communicate with OTA server and download firmware images |
| `READ_EXTERNAL_STORAGE` | Read downloaded OTA packages |
| `WRITE_EXTERNAL_STORAGE` | Save downloaded OTA packages to `/sdcard/` |
| `MANAGE_EXTERNAL_STORAGE` | Full filesystem access for OTA file management |
| `RECEIVE_BOOT_COMPLETED` | Start OTA services on device boot |
| `SYSTEM_ALERT_WINDOW` | Display system-level OTA dialogs over other apps |
| `READ_SYNC_SETTINGS` | Read sync configuration |
| `WRITE_SYNC_SETTINGS` | Write sync configuration |
| `WRITE_SETTINGS` | Modify system settings (OTA state persistence) |
| `WAKE_LOCK` | Keep CPU awake during download and update operations |
| `ACCESS_WIFI_STATE` | Check WiFi connectivity status |
| `CHANGE_WIFI_STATE` | Programmatically connect to WiFi networks (sprite OTA) |
| `ACCESS_FINE_LOCATION` | Required for WiFi scanning on newer Android |
| `ACCESS_NETWORK_STATE` | Monitor network connectivity changes |
| `CHANGE_NETWORK_STATE` | Manage network connections |
| `NEARBY_WIFI_DEVICES` | WiFi device discovery (neverForLocation) |

### Custom Permissions

| Permission | Purpose |
|---|---|
| `com.rokid.ai.glass.provider.read` | Read data from Rokid AI glass content provider |
| `com.rokid.glass.ota.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` | Signature-level permission for internal broadcast receivers |

### Hardware Features

- `android.hardware.screen.landscape` - Landscape-only display (AR glasses form factor)

## System Properties

The app reads these Android system properties at initialization (`DeviceProp.init()`):

| Property | Field | Purpose |
|---|---|---|
| `ro.boot.key` | `DeviceProp.key` | Device authentication key for OTA API |
| `ro.boot.secret` | `DeviceProp.secret` | Device authentication secret for signing requests |
| `ro.rokid.device_type_id` | `DeviceProp.deviceTypeId` | Device type identifier |
| `ro.rokid.device_id` | `DeviceProp.deviceId` | Unique device identifier |
| `ro.rokid.ota.check_url` | `DeviceProp.checkUrl` | Base URL for OTA check server |
| `ro.rokid.ota.check_api` | `DeviceProp.checkApi` | API path appended to check_url |
| `ro.rokid.master.version` | (Station2 only) | System version for Station2 devices |
| `ro.product.model` | (detection) | Identifies device type; `RG-glasses` = Sprite glasses |
| `ro.rokid.product.generation` | (detection) | `RG-station2` or `RG-stationPro` for station devices |
| `ro.boot.glassesWithPanel` | (detection) | `1` = Sprite device with display panel |
| `ro.build.date.utc` | (policy) | Build timestamp, used for 7-day-old check |

The hardcoded constants are:
- `DeviceProp.version` = `"1.0"` (API version)
- `DeviceProp.service` = `"ota"` (service identifier)

## Architecture Overview

The app uses a state machine architecture with four state engines managed by `DataCenterEngine`, coordinated through broadcast-based messaging between services and UI components.

### Device Type Detection

The app supports three device channels and three hardware types:

**Channels** (set via `APK_CHANNEL` meta-data, hardcoded to `master` in this build):
- `master` - Standalone glasses/station mode (default)
- `erbox` - ER Station mode (companion station device)
- `common` - Generic mode with full-screen activities

**Hardware types** (detected from system properties):
- **Sprite** (`RG-glasses`) - AR glasses connected via USB to a station
- **Station2** (`RG-station2`) - Second-gen docking station
- **StationPro** (`RG-stationPro`) - Pro docking station

The channel determines UI behavior: `common` uses full-screen Activities, `erbox` uses system Dialog overlays, and `master` uses broadcast-based messaging to external UI (Unity integration for Sprite).

## Application Lifecycle

`OtaApplication` (extends `BasicApplication`) is the entry point. On `onCreate()`:

1. Initializes logging with version name, log level 1
2. Calls `OtaConfig.initConfig("master")` to set the channel
3. Creates the `DataCenterEngine` (state machine hub)
4. Initializes `VoiceInstruction` (voice command support)
5. Registers a listener for `reboot_success` on `RebootState` that broadcasts `com.ota.success.reboot`
6. Calls `DeviceUtil.initConfig()` to cache the device type
7. Starts three services sequentially:
   - `CheckService` - OTA version check worker
   - `DownloadService` - Firmware image download worker
   - `OtaService` - Main orchestrator service

## State Machine

`DataCenterEngine` manages four independent state machines. Each state extends `BaseState`, which persists state to `Settings.Global` and supports listener-based notifications.

### WifiConnectState

Persisted key: `ota_wifi_connect_state_key`

States:
- `normal` - Idle (initial)
- `wifi_connecting` - Attempting WiFi connection (Sprite mode)
- `wifi_connect_success` - WiFi connected successfully
- `wifi_connect_failed` - WiFi connection failed

### CheckState

Persisted key: `ota_check_state_key`

States:
- `normal` - Idle (initial)
- `net_need_check` - Network check needed, triggers server query
- `net_checking` - Actively checking server
- `net_check_end` - Server check complete (carries `CheckResult` with success/failure)
- `device_checking` - Verifying device conditions (power, storage)
- `device_check_success` - Device conditions met, proceed to download
- `device_check_failed` - Device conditions not met

`CheckResult` types:
- `TYPE_NORMAL` (0) - Standard result
- `TYPE_RUNNING_NET_ERROR` (10) - Network error during check
- `TYPE_RUNNING_SN_CLOUD_ERROR` (20) - Cloud/SN error

### UpgradeState

Persisted key: `ota_upgrade_state_key`

States:
- `normal` - Idle (initial)
- `need_download` - Ready to download firmware
- `downloading` - Download in progress (carries `UpgradeResult` with percent)
- `download_success` - Download and MD5 verification complete
- `download_failed` - Download failed (carries `UpgradeResult` with error info)
- `updating` - A/B update in progress (carries `UpgradeResult` with percent)
- `update_success` - Update complete, reboot needed
- `update_failed` - Update failed

`UpgradeResult` types:
- `TYPE_NORMAL` (0) - General
- `TYPE_DOWNLOAD` (1) - Download phase
- `TYPE_PRE_UPDATE` (2) - Pre-update verification phase
- `TYPE_REAL_UPDATE` (3) - Actual partition write phase

### RebootState

Persisted key: `ota_reboot_state_key`

States:
- `normal` - Idle
- `reboot_success` - Device has rebooted successfully after an update

Unlike other states, `RebootState.initStateBySave()` restores from persisted state (to detect post-reboot success across app restarts).

### Trigger Modes

The `DataCenterEngine` tracks whether the current OTA flow was initiated automatically or by user:
- `auto` - Triggered by `TimingManager` periodic check
- `user` - Triggered by user action or external command

This affects behavior: auto-mode force updates skip UI prompts, auto-mode non-force updates respect ignored versions, and network error dialogs are only shown in user mode.

## OTA Flow

### 1. Check for Updates

**Entry points:**
- `TimingManager` automatic timer (first check 5 minutes after boot, then every 1 hour)
- Network reconnection or screen-on events (2-minute delay, throttled to 1 hour between checks)
- External broadcast: `android.intent.customize.action.JUDGE_BEFORE_CHECK`
- Sprite command: `sprite_cmd_start_ota` with `OtaParam` JSON

**CheckService.handleCheckOtaVersion():**

1. Clears previous check info via `DownloadUriProp.clearAllCheckInfo()`
2. Constructs the check URL: `DeviceProp.checkUrl + DeviceProp.checkApi`
3. Generates an MD5 signature:
   ```
   MD5("key={key}&device_type_id={deviceTypeId}&device_id={deviceId}&service=ota&version=1.0&time={epoch_seconds}&secret={secret}")
   ```
4. Builds an Authorization header:
   ```
   version=1.0;time={epoch_seconds};sign={md5};key={key};device_type_id={deviceTypeId};device_id={deviceId};service=ota
   ```
5. Sends a POST request with JSON body:
   ```json
   {
     "version": "{current_system_version}",
     "osType": "",
     "cpuType": ""
   }
   ```
   - Content-Type: `application/json;charset=utf-8`
   - Connect timeout: 6 seconds
   - Read timeout: 10 seconds
6. Retries up to 4 times on failure (1 second between retries)

**Server response parsing** (`unPackageResponse`):

If `code == "OK"`:
- Extracts: `imageUrl`, `isForceUpdate`, `version`, `checksum`, `changelog`
- Checks for differential upgrade (same base version prefix) and skips if so
- On Sprite devices, forces `isForceUpdate = true`
- Stores all fields in `DownloadUriProp` (SharedPreferences `ota_sp`)
- Then fetches the image size by issuing a separate HTTP request to `imageUrl` and reading `Content-Length`

If `code == "NO_IMAGE"`:
- No update available, treated as success (no new version)

Otherwise:
- Treated as error with the `errorMessage` from response

### 2. Device Condition Check

After a new version is found, the system checks device conditions before downloading:

**Conditions checked:**
- Network connectivity (WiFi or any network)
- Battery level:
  - Default: >= 15% OR charging
  - Sprite/ER Station/Master: >= 30% (charging) or >= 70% (not charging)
- Storage: available space >= image_size + 1 GB (1073741824 bytes)

**Force update behavior:**
- If `isForceUpdate == true` AND auto mode AND WiFi connected: skip user prompt, proceed directly to download
- If conditions not met during force update: show error toast and cancel all processes

**Non-force update behavior:**
- If user-initiated: show upgrade prompt dialog
- If auto-initiated: show upgrade prompt unless the version is in the ignore list
- If mobile data only: show "Allow mobile download?" dialog

### 3. Download

**DownloadService.handleDownload():**

1. Determines download path from `DownloadUriProp.DOWNLOAD_PATH` (typically `/sdcard/`)
2. Download URL from `DownloadUriProp.DOWNLOAD_URL`
3. Temporary filename: `{version}.tmp`
4. Final filename: `update.zip`
5. If `update.zip` already exists with correct MD5: skip download
6. Supports resumable downloads (HTTP Range requests):
   - Sets `Range: bytes={offset}-` header
   - Expects HTTP 206 response
   - Writes to file via `RandomAccessFile.seek()` for resume
7. Reports progress via `UpgradeState.VALUE_DOWNLOADING` with percentage
8. On completion:
   - Verifies MD5 checksum (3 retries, 1 second between)
   - Renames `.tmp` to `update.zip`
9. Retries entire download up to 3 times on failure (1 second between)
10. Acquires a `PARTIAL_WAKE_LOCK` during download to prevent CPU sleep

**Download error codes:**
- 10400 - General download failure
- 10401 - Cannot create temp file
- 10402 - Server does not support Range requests
- 10403 - Downloaded more than 100% (size mismatch)
- 10404 - Cannot rename .tmp to .zip
- 10410 - MD5 checksum mismatch
- 10411 - Download stopped by user

**Battery safety:** On Sprite devices, if battery drops below 10% while not charging, the download is automatically stopped.

### 4. Update (A/B Partition)

**SystemUpdateManager.update():**

Uses Android's `UpdateEngine` API (A/B seamless updates):

1. Creates `MyUpdateEngine` wrapper (custom UpdateEngine implementation)
2. Registers `UpdateEngineCallback` for progress and completion
3. Calls `abUpdate("/sdcard/update.zip", callback)` (decompilation partially failed for this method - it parses `payload.bin` and `payload_properties.txt` from the ZIP)
4. Progress reporting through UpdateEngine status codes:
   - Status 3: Download/pre-update phase (50% weight on Sprite)
   - Status 4: Verification phase (Sprite: maps to 50-100% range)
   - Status 5: Final update write phase
5. On `onPayloadApplicationComplete(0)`: Update success, deletes `update.zip`, sets state to `update_success`
6. On non-zero result: Update failure with error code 10501

### 5. Reboot

After successful update:
- State transitions to `update_success` (maps to "need reboot")
- UI shows reboot prompt
- If force update: may auto-reboot (Sprite shows reboot command immediately)
- Reboot executed via `PowerManager.reboot(null)`
- After reboot, `RebootState` persisted value `reboot_success` is detected on next `initStateBySave()`
- Success broadcast: `com.ota.success.reboot`
- ER Station: writes `rokid_er_user_ota_notify_success = true` to Settings.Global, shows success notification on next OtaService.onCreate()

## Services

### OtaService

The main orchestrator service. Not bound (returns null from `onBind()`), runs indefinitely.

**Responsibilities:**
- Registers all state change listeners on `DataCenterEngine`
- Manages `SystemUpdateManager`, `TimingManager`, `WifiConnector`
- Registers broadcast receivers for commands, network, screen, battery, USB
- Coordinates the entire OTA flow through state transitions
- Manages a `WakeLock` (tag: `Ota::OtaWakelockTag`) for download/update phases

### CheckService

Background service running on `Ota_Check_Thread` HandlerThread.

**Responsibilities:**
- Executes HTTP POST to OTA server to check for new firmware
- Parses response JSON
- Fetches image size from download URL
- Reports results via `CheckState` transitions

### DownloadService

Background service running on `Ota_Download_Thread` HandlerThread.

**Responsibilities:**
- Downloads firmware image with resume support
- Verifies MD5 checksum
- Reports progress percentages
- Supports user-initiated stop

## Activities

All activities are landscape-only, exported, and use `singleTask` launch mode. They serve as full-screen UI for the `common` channel.

| Activity | Purpose |
|---|---|
| `AllowMobileDownloadActivity` | Asks user to confirm download over mobile data |
| `DeviceConditionCheckActivity` | Shows device condition checklist (WiFi, battery, storage) |
| `NeedRebootActivity` | Prompts user to reboot after successful update |
| `NeedUpgradeActivity` | Shows new version info and asks user to upgrade |
| `NetCheckErrorActivity` | Displays network check error with retry option |
| `NetCheckingActivity` | Shows "checking for updates" spinner |
| `UnNeedUpgradeActivity` | Informs user the system is already up to date |
| `UpdatePowerErrorActivity` | Shows insufficient battery error |
| `UpgradeFailedActivity` | Displays upgrade failure with error details |
| `UpgradePercentActivity` | Shows download/update progress with percentage |

For `erbox` channel, equivalent `erbox/` dialog classes are used instead (system-level `Dialog` windows via `SYSTEM_ALERT_WINDOW`):
`DeviceConditionCheckDialog`, `NeedRebootDialog`, `NeedUpgradeDialog`, `NetCheckErrorDialog`, `NetCheckingDialog`, `UnNeedUpgradeDialog`, `UpdatePowerErrorDialog`, `UpgradeFailedDialog`, `UpgradePercentDialog`.

## Broadcast Intents

### Received (CommandReceiver)

| Action | Purpose |
|---|---|
| `android.intent.customize.action.JUDGE_BEFORE_CHECK` | External trigger to check for OTA updates |
| `android.intent.customize.action.JUDGE_BEFORE_UPDATE` | External trigger to apply downloaded update |
| `com.rokid.glass.ota.DATA_CMD_ACTION` | Direct state engine control (type, cmd, info extras) |
| `com.rokid.glass.ota.NORMAL_CMD_ACTION` | Normal command dispatch (see commands below) |

**NORMAL_CMD_ACTION commands** (via `normal_cmd` extra):

| Command | Purpose |
|---|---|
| `device_reboot` | Reboot the device immediately |
| `get_update_info` | Request current update info broadcast |
| `ignore_new_version` | Mark current available version as ignored |
| `sprite_cmd_start_ota` | Start OTA with Sprite parameters (JSON `OtaParam` in `cmd_param`) |
| `sprite_cmd_stop_ota` | Stop ongoing OTA download |
| `sprite_check_status` | Query current OTA process status |
| `sprite_auto_config` | Auto OTA configuration (stub, not implemented) |
| `check_need_reboot` | Check if device needs reboot after update |

### Received (System Broadcasts)

| Action | Receiver | Purpose |
|---|---|---|
| `android.net.conn.CONNECTIVITY_CHANGE` | `NetStatusReceiver` | Network state changes (WiFi/mobile/ethernet) |
| `android.intent.action.SCREEN_ON` | `ScreenStatusReceiver` | Screen turned on |
| `android.intent.action.SCREEN_OFF` | `ScreenStatusReceiver` | Screen turned off |
| `android.intent.action.BATTERY_CHANGED` | `BatteryStatusReceiver` | Battery level and charging state changes |
| `android.hardware.usb.action.USB_DEVICE_ATTACHED` | `UsbMicUtil` | USB device connected (Sprite glasses via USB) |
| `android.hardware.usb.action.USB_DEVICE_DETACHED` | `UsbMicUtil` | USB device disconnected |

### Sent

| Action | Purpose |
|---|---|
| `com.ota.success.reboot` | Broadcast after successful OTA reboot |
| `android.intent.action.ERNotification` | ER Station notification (content, imagePath, type extras) |
| `com.rokid.glass.ota.msg.control` | Internal OTA message bus (local or global broadcast) |

### Internal Message Bus (OtaMsgSender/OtaMsgReceiver)

Action: `com.rokid.glass.ota.msg.control`

Uses `LocalBroadcastManager` when not in `master` channel, global broadcast otherwise.

Commands (via `msg_cmd` extra):

| Command | Purpose |
|---|---|
| `cmd_update_ota_process_info` | Send full OTA process status snapshot |
| `cmd_update_new_version_info` | Send new version details (URL, version, checksum, changelog, size) |
| `cmd_show_device_condition_check` | Show device condition check UI |
| `cmd_show_update_power_error` | Show power error UI |
| `cmd_show_un_need_upgrade` | Show "already up to date" UI |
| `cmd_show_net_connect_error` | Show network error UI |
| `cmd_show_net_checking` | Show "checking" spinner UI |
| `cmd_dismiss_net_checking` | Dismiss "checking" spinner |
| `cmd_show_need_upgrade` | Show "new version available" prompt |
| `cmd_dismiss_all_ota_layer` | Dismiss all OTA UI overlays |
| `cmd_show_upgrade_percent_ui` | Show/update progress percent UI |
| `cmd_dismiss_upgrade_percent_ui` | Dismiss progress UI |
| `cmd_show_upgrade_failed` | Show upgrade failure UI |
| `cmd_show_need_reboot` | Show reboot prompt UI |
| `cmd_show_allow_mobile_download` | Show mobile data download prompt |
| `cmd_ota_status_msg_info` | Show OTA status toast message |

## AIDL Interface

`IOtaCheckService` is an AIDL interface (descriptor: `com.rokid.glass.ota.IOtaCheckService`) exposing a single method:

```java
String checkOtaState()
```

This allows other system apps to query the current OTA state via IPC.

## Sprite (Glasses) Integration

When the device is detected as Sprite (`ro.product.model == RG-glasses`):

- `SpriteOtaUnity` initializes OTA with a Unity message bridge
- UI messages are forwarded to Unity via `UnityMessageManager.sendUnityMessage()` as JSON
- OTA parameters include `wifiSsid` and `wifiPassword` for glasses to connect to WiFi
- `WifiConnector` programmatically connects to WiFi using `WifiManager.addNetwork()`
- Battery thresholds are elevated: 30% (charging), 70% (not charging)
- `isForceUpdate` is always set to `true` for Sprite devices
- USB connection detection via `UsbMicUtil` (vendor ID 1234, product IDs 5677/5678/5679)
- System version has `-150` to `-151` suffix replacement logic for non-panel Sprite devices

## OTA Parameters (OtaParam)

JSON structure sent via Sprite commands or parsed from server response:

```json
{
  "imageUrl": "https://...",
  "version": "x.y.z-build",
  "checksum": "md5hex",
  "changelog": "...",
  "isForceUpdate": true,
  "imageSize": 123456789,
  "wifiSsid": "NetworkName",
  "wifiPassword": "password"
}
```

`wifiSsid` and `wifiPassword` are only used in Sprite mode for automated WiFi connection.

## Settings.Global Keys

| Key | Purpose |
|---|---|
| `ota_check_state_key` | Persisted CheckState value |
| `ota_upgrade_state_key` | Persisted UpgradeState value |
| `ota_reboot_state_key` | Persisted RebootState value |
| `ota_wifi_connect_state_key` | Persisted WifiConnectState value |
| `rokid_er_user_ota_has_new_version` | ER Station flag: new version available |
| `rokid_er_user_ota_ignore_update` | ER Station flag: user disabled auto OTA |
| `rokid_er_user_ota_notify_success` | ER Station flag: need to show success notification |

## SharedPreferences

File: `ota_sp` (private mode)

Keys in `DownloadUriProp`:

| Key | Type | Purpose |
|---|---|---|
| `download_url` | String | Firmware image download URL |
| `download_path` | String | Local download directory path |
| `download_version` | String | Firmware version string |
| `check_sum` | String | MD5 hex checksum of firmware image |
| `change_log` | String | Version changelog text |
| `force_update` | Boolean | Whether this is a forced update |
| `image_size` | Long | Firmware image size in bytes |
| `ignore_upgrade_version` | String | Version string the user chose to ignore |
| `wifi_ssid` | String | WiFi SSID for Sprite connection |
| `wifi_password` | String | WiFi password for Sprite connection |

## Timing and Thresholds

| Constant | Value | Purpose |
|---|---|---|
| First auto-check delay | 5 minutes (300,000 ms) | Delay after boot before first OTA check |
| Periodic check interval | 1 hour (3,600,000 ms) | Interval between automatic OTA checks |
| Network/screen event delay | 2 minutes (120,000 ms) | Delay after net reconnect/screen-on before check |
| Message dedup interval | 6 hours (21,600,000 ms) | Min time between ER Station notification messages |
| Check throttle | 1 hour | Min time between event-triggered checks |
| HTTP connect timeout | 6 seconds | TCP connection timeout |
| HTTP read timeout | 10 seconds | Response read timeout |
| Download retry count | 3 | Max download attempts |
| Check retry count | 4 | Max OTA check attempts |
| WiFi connect timeout | 10 seconds | Max WiFi connection polls (10 x 1s) |
| Default min battery | 15% | Min battery for non-Sprite devices |
| Sprite/Station min battery (charging) | 30% | Min battery when charging |
| Sprite min battery (not charging) | 70% | Min battery when not charging |
| Min free storage | image_size + 1 GB | Required free space |
| 7-day build age check | 604,800 seconds | `PolicyManager.isSevenDayAgo()` |

## Error Codes

| Code | Context | Meaning |
|---|---|---|
| 10100 | Check | General exception during check |
| 10101 | Check | HTTP error stream response |
| 10102 | Check | Non-200 HTTP response code |
| 10103 | Check | WiFi/network connection error |
| 10110 | Check | JSON parse exception |
| 10111 | Check | Server returned error code (not NO_IMAGE) |
| 10112 | Check | SharedPreferences commit failure |
| 10120 | Check | Exception getting image size |
| 10121 | Check | Non-200 response for image size request |
| 10122 | Check | Image size <= 0 |
| 10123 | Check | SharedPreferences commit failure for image size |
| 10202 | Device | Insufficient battery for force update (user) |
| 10203 | Update | Insufficient battery for update after download |
| 10310 | Device | Insufficient storage for force update |
| 10400 | Download | General download failure / WiFi error |
| 10401 | Download | Cannot create temp file |
| 10402 | Download | Server does not support Range requests (HTTP 206) |
| 10403 | Download | Downloaded data exceeds expected size |
| 10404 | Download | Cannot rename .tmp to .zip |
| 10405 | Download | Force update download error |
| 10410 | Download | MD5 checksum mismatch |
| 10411 | Download | Download stopped by user |
| 10501 | Update | A/B UpdateEngine failure (includes engine error code) |
| -99999 | Check | NO_IMAGE - no update available (internal, not a real error) |

## File System Paths

| Path | Purpose |
|---|---|
| `/sdcard/update.zip` | Final downloaded OTA package |
| `/sdcard/{version}.tmp` | Temporary download file (resumable) |
| `/etc/ota.prop` | OTA configuration properties file |
| `/system/build.prop` | System build properties (build date) |

## Key Files (Decompiled Source)

| File | Role |
|---|---|
| `OtaApplication.java` | Application entry point, starts all services |
| `OtaService.java` | Main orchestrator, all state listeners and receivers |
| `OtaConfig.java` | Channel configuration and broadcast action constants |
| `DataCenterEngine.java` | State machine hub with event listeners |
| `BaseState.java` | Abstract state with persistence and observer pattern |
| `CheckState.java` | Version check state machine |
| `UpgradeState.java` | Download and update state machine |
| `RebootState.java` | Post-reboot state detection |
| `WifiConnectState.java` | WiFi connection state for Sprite |
| `CheckService.java` | HTTP client for OTA server check |
| `DownloadService.java` | Resumable firmware image downloader |
| `SystemUpdateManager.java` | A/B partition update via UpdateEngine |
| `TimingManager.java` | Periodic and event-triggered check scheduler |
| `PolicyManager.java` | Device condition policies (battery, storage, network) |
| `DeviceUtil.java` | Device type detection and reboot |
| `CommandReceiver.java` | External command broadcast receiver |
| `OtaMsgSender.java` | Internal OTA UI message dispatcher |
| `OtaMsgReceiver.java` | Internal OTA UI message listener |
| `OtaUiControlManager.java` | Routes messages to Activities or Dialogs |
| `SpriteOtaUnity.java` | Sprite/glasses Unity bridge |
| `SpriteMsgConverter.java` | Converts OTA messages to Unity JSON |
| `WifiConnector.java` | Programmatic WiFi connection for Sprite |
| `OtaCmdSender.java` | Static helper for sending OTA command broadcasts |
| `SettingWR.java` | Settings.Global read/write helper for ER Station |
| `MD5Utils.java` | MD5 hashing for signature generation and file verification |
| `DeviceProp.java` | System property loader for OTA credentials |
| `IOtaCheckService.java` | AIDL interface for cross-process OTA state query |
