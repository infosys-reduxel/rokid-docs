# CXRService

Glasses-side system service that manages all Bluetooth communication between Rokid AR glasses and companion devices (Android phones via RFCOMM, iPhones via BLE GATT and MFI/iAP2). Runs as a persistent, direct-boot-aware system app with `android.uid.system` shared UID.

## Package Info

| Field | Value |
|---|---|
| Package | `com.rokid.cxrservice` |
| Version | 1.103 (versionCode 32, versionName 12) |
| Min SDK | 32 (Android 12L) |
| Target SDK | 32 |
| Shared UID | `android.uid.system` |
| Persistent | Yes |
| Direct Boot Aware | Yes |
| Cleartext Traffic | Allowed |
| Uses Non-SDK API | Yes |
| Native Libraries | `libcxr_service_jni.so`, `libcaps-jni.so` |
| Config File | `system/etc/cxr-service.json` |

## Permissions

| Permission | Purpose |
|---|---|
| `android.permission.BLUETOOTH_CONNECT` | Bluetooth device connections |

The app also gets all permissions granted to `android.uid.system` implicitly (Bluetooth admin, system time, etc.).

## Exported Components

### CXRService (Service)

The single exported component. This is an Android `Service` (not an Activity).

```xml
<service android:name=".CXRService" android:enabled="true" android:exported="true">
    <intent-filter>
        <action android:name="com.rokid.cxrservice.CXRService"/>
    </intent-filter>
</service>
```

**Binding**: `onBind()` returns `null`. External apps cannot bind to this service via AIDL. All communication happens over Bluetooth or via the Flora IPC service (started natively via `startFloraService()`).

**Starting**: The service is started by `CXRServiceApp` (the Application subclass) when it receives the `android.intent.action.USER_UNLOCKED` broadcast. Since the app is `persistent=true`, the system keeps it alive.

## Broadcast Receivers (Internal)

These are registered dynamically in code, not in the manifest:

| Receiver | Action | Purpose |
|---|---|---|
| `btStatusReceiver` | `android.bluetooth.adapter.action.STATE_CHANGED` | Reacts to BT on/off. Stops socket services when BT turns off, restarts them when BT turns on. On first BT-on after boot, reconnects to last known iOS device. |
| `glassActionReceiver` | `android.intent.action.SCREEN_ON` | Reconnects to last known iOS device when glasses screen turns on. |
| `btNameReceiver` | `android.bluetooth.adapter.action.LOCAL_NAME_CHANGED` | Restarts BLE advertising when the glasses' Bluetooth name changes (so the new name appears in scans). |
| `receiver` (in CXRServiceApp) | `android.intent.action.USER_UNLOCKED` | Starts the CXRService after device unlock (direct boot). |

## System Properties

| Property | Access | Description |
|---|---|---|
| `rokid.cxr-service.version` | Write | Set to `"1.103"` on app startup. |
| `ro.boot.glassesWithPanel` | Read | `"1"` if glasses have a display panel. Affects MFI connection parameters. |
| `ro.serialno` | Read | Glass serial number, used in BLE advertising scan response data. |
| `ro.boot.serialno` | Read | Glass serial number, passed to native MFI connection code. |

## Architecture Overview

### Class Hierarchy

```
com.rokid.cxr/
    Caps                          -- Serialization format (Rokid's binary message format)
    Caps.Value                    -- Typed value container (int32/uint32/int64/uint64/float/double/string/binary/object)
    Caps.Binary                   -- Binary data wrapper
    Caps.IncorrectTypeException   -- Type mismatch error
    CXRSocketProtocol             -- Socket-level protocol over Bluetooth RFCOMM
    CXRSocketProtocol.Callback    -- Receive callbacks (messages, audio streams, ARTC frames)

com.rokid.cxrservice/
    CXRServiceApp                 -- Application subclass, sets version property, starts service on USER_UNLOCKED
    CXRService                    -- Main service: manages client connections, GATT server, auth, Flora IPC
    CXRService.ClientInfo         -- Per-client info (type, device, MAC)
    CXRService.ClientConnectionInfo -- Persisted connection info (type, MAC, name, account, service record)
    CXRService.XServerCallback    -- GATT server callback (read/write/connection/notification handlers)
    CXRService.GATTCommandHandler -- Interface for handling GATT command messages
    CXRService.PendingNotification -- Queued GATT notification
    CXRConfig                     -- Configuration loaded from system/etc/cxr-service.json (native)
    CXRConfig.CXR                 -- Core config: MTU sizes, storage paths, buffer sizes
    CXRConfig.CXRProto            -- Protocol buffer sizes
    CXRConfig.MFI                 -- MFI config: timeouts, manufacturer info, EAP apps, firmware version
    CXRConfig.EAP                 -- External Accessory Protocol app entry (iOS)
    CXRConfig.IdentificationInformation -- MFI device identification (model_id, name, product_plan_uid)
    CXRConfig.GATT                -- GATT notification timeout
    CXRConfig.Audio               -- Audio config: buffer sizes, AEC parameters
    CXRConfig.ANT_AEC_PARAM       -- ANT acoustic echo cancellation parameters
    CXRConfig.ROKID_AEC_PARAM     -- Rokid AEC parameters
    CXRConfig.ARTC                -- Audio real-time communication config
    BLEServiceManager             -- BLE GATT server and advertising management
    BTSocketService               -- RFCOMM Bluetooth server socket for Android clients
    MFISocketService              -- RFCOMM server socket specifically for MFI (iOS) clients
    MFIConnection                 -- Single MFI/iAP2 connection handler (read/write threads, native IAP2 service)
    MFIConnectionHelper           -- MFI connection lifecycle manager (connect/disconnect/reconnect tasks)
    MFIConnectionHelper.ConnectTask     -- Async RFCOMM connection attempt with timeout
    MFIConnectionHelper.BondedDeviceTask -- Scans bonded devices and initiates MFI connections
    MFIConnectionHelper.DisconnectTask  -- Async cleanup of MFI connections
    ClientConnectionState         -- Publishes connection state changes to native layer with 40s timeout
    SocketServiceCallback         -- Interface for socket accept/connect/disconnect events
    R                             -- Empty resources class
```

### Client Types

The service supports three client connection types:

| Type ID | Name | Transport | Description |
|---|---|---|---|
| 1 | `android` | Bluetooth RFCOMM | Android phone connects via classic Bluetooth SPP. |
| 2 | `iphone-gatt` | BLE GATT | iPhone connects via BLE GATT characteristics for data exchange. |
| 3 | `iphone-socket` | MFI/iAP2 over RFCOMM | iPhone connects via Apple MFI protocol over classic Bluetooth (External Accessory framework). |

Client types 2 and 3 are considered compatible (an iPhone can use either or both).

### Connection Flow

#### 1. Boot and Initialization

1. System starts `CXRServiceApp` (persistent app).
2. App registers receiver for `USER_UNLOCKED`.
3. On unlock, starts `CXRService`.
4. `CXRService.onCreate()`:
   - Loads config from `system/etc/cxr-service.json` via native `CXRConfig.loadConfig()`.
   - Detects Apple MFI coprocessor at `/sys/devices/platform/soc/988000.i2c/i2c-2/2-0010/acp_certificate`.
   - Initializes GATT command handlers.
   - Creates `BLEServiceManager` (opens GATT server, starts advertising thread).
   - Starts Flora IPC service (native, for inter-process communication with other glasses apps).
   - Starts Bluetooth socket services (RFCOMM for Android, MFI socket for iOS if MFI hardware present).
   - Loads previously saved client connection info from file.
   - Starts GATT notification queue thread.

#### 2. BLE Advertising

`BLEServiceManager` advertises a compound GATT service with UUID `00009100-0000-1000-8000-00805f9b34fb`.

The scan response includes the glass serial number under a service data UUID derived from `00009a01-0000-1000-8000-00805f9b34fb`.

Advertising settings:
- Mode: balanced (1)
- Connectable: true
- Timeout: 0 (indefinite)
- TX power level: high (2)

#### 3. GATT Service Structure

The compound GATT service contains six characteristics:

| Characteristic | UUID | Properties | Permissions | Purpose |
|---|---|---|---|---|
| iOS Connection | `00009300-0000-1000-8000-00805f9b34fb` | Read | Read | Returns MFI status, service record UUID, Rokid account, panel flag, MAC address (Caps-serialized). |
| RFCOMM Connection | `00009301-0000-1000-8000-00805f9b34fb` | Read | Read | Returns RFCOMM service record UUID, MAC address, Rokid account, panel flag, GATT version (Caps-serialized). |
| iOS Write | `00009201-0000-1000-8000-00805f9b34fb` | Write no response | Write | Auth packets and CXR protocol data from iPhone (GATT transport). |
| iOS Notify | `00009202-0000-1000-8000-00805f9b34fb` | Notify | Read | CXR protocol data sent to iPhone (GATT transport). |
| iOS MFI Pairing | `00009203-0000-1000-8000-00805f9b34fb` | Write no response | Write | MFI pairing initiation and GATT commands from iPhone. |
| iOS MFI Notify | `00009204-0000-1000-8000-00805f9b34fb` | Notify | Read | MFI-related notifications to iPhone. |

All UUIDs follow the Bluetooth Base UUID pattern: `0000XXYY-0000-1000-8000-00805f9b34fb` where XX is the group and YY is the index.

#### 4. Android Client Connection (Type 1)

1. Phone discovers glasses via BLE advertising.
2. Phone reads RFCOMM Connection characteristic to get the service record UUID.
3. Phone connects via RFCOMM using that UUID.
4. `BTSocketService` accepts the connection.
5. `CXRSocketProtocol` runs the auth handshake (native), then the protocol loop.
6. On successful auth, BLE advertising stops.

RFCOMM service name: `rokid.CXRTransfer`.

#### 5. iPhone GATT Connection (Type 2)

1. Phone discovers glasses via BLE advertising.
2. Phone reads iOS Connection characteristic.
3. Phone writes auth packets to iOS Write characteristic.
4. Auth is handled natively via `handleAuthPacket()`.
5. On success, `onClientConnected()` is called. The phone and glasses create a BLE bond.
6. Data is exchanged via iOS Write (phone to glasses) and iOS Notify (glasses to phone).

#### 6. iPhone MFI Connection (Type 3)

1. Phone discovers glasses via BLE advertising.
2. Phone writes to iOS MFI Pairing characteristic (single byte triggers classic BT pairing).
3. Glasses initiate classic Bluetooth bond with `createBond()`.
4. `MFIConnectionHelper.BondedDeviceTask` finds the bonded device and connects via RFCOMM to UUID `00000000-deca-fade-deca-deafdecacafe`.
5. `MFIConnection` starts read/write threads and the native iAP2 service.
6. Data is exchanged via the iAP2 protocol over the RFCOMM socket.

MFI server socket listens on UUID `00000000-deca-fade-deca-deafdecacaff`.

### GATT Commands

Commands are sent as Caps-serialized messages to the iOS MFI Pairing characteristic:

| Command ID | Name | Description |
|---|---|---|
| 0x1100 (4352) | `GATT_COMMAND_ASK_MFI_CONNECT` | iPhone asks glasses to initiate MFI classic BT connection. Payload: UUID string, optional MAC. Glasses respond with `GATT_COMMAND_ASK_MFI_CONNECT_RESP` (4353). |
| 0x1102 (4354) | `GATT_COMMAND_MFI_APP_SYNC_INFO` | Deprecated (logs "abandoned"). |

### CXR Protocol

Protocol version: 1.4

#### Auth Response (command 0x1005 / 4101)

Sent from glasses to client after auth:

```
Caps {
    uint32: 4101           // CXR_CMD_AUTH_RESP
    uint32: major_version  // 1
    uint32: minor_version  // 4
    int32:  result          // 0 = success, negative = error
}
```

Auth error codes:
- `0`: Success
- `-1`: Major version mismatch
- `-3`: No service record
- `-4`: Service record mismatch
- `-6`: Already connected / rejected

#### CXR Control Commands

Called from native code via `doCXRControl(command, param)`:

| Command | Description |
|---|---|
| 1 | Start BT pairing. `param & 1` = clear bonded devices. |
| 4 | Request app launch on iPhone (via MFI EAP). |
| 5 | Update MFI detection status. `param=0` disables, otherwise re-checks hardware. |

### CXRSocketProtocol

Bidirectional message protocol over Bluetooth sockets. Used for both RFCOMM (Android) and MFI connections.

**Callback interface** (`CXRSocketProtocol.Callback`):

| Method | Description |
|---|---|
| `onReceived(String, Caps, byte[])` | Named message received with optional binary payload. |
| `onNotify(String, Caps)` | Named notification received. |
| `onResponse(int, Caps)` | Response to a numbered request. |
| `onStartAudioStream(int, int, String, Caps)` | Audio stream starts. |
| `onAudioStream(int, byte[], int, int)` | Audio stream data chunk. |
| `onAudioStreamFinish(int)` | Audio stream ends. |
| `onARTCFrame(byte[], long)` | Real-time audio/communication frame. |
| `onDisconnect()` | Socket disconnected. |

**Methods for sending**:

| Method | Description |
|---|---|
| `request(int, String, Caps)` | Send a numbered request. |
| `send(String, Caps, byte[])` | Send a named message with binary data. |
| `openAudioRecord(int, int, String, int)` | Start audio recording on glasses microphone. |
| `closeAudioRecord(String)` | Stop audio recording (sends CXRControl request). |
| `startPlayAudio(int, int, float, int)` | Start audio playback stream. |
| `startAudioStream(int, int, String, Caps)` | Start a named audio stream. |
| `writeAudioStream(int, byte[], int, int)` | Write audio data to stream. |
| `finishAudioStream(int)` | Finish audio stream. |
| `cancelAudioStream(int)` | Cancel audio stream. |
| `changeRokidAccount(String)` | Update Rokid account on the connection. |

### Caps Serialization

`com.rokid.cxr.Caps` is Rokid's binary serialization format used for all structured data in the CXR protocol. It is an ordered sequence of typed values.

**Supported types**:

| Type Char | Java Type | Description |
|---|---|---|
| `V` | void/null | Void placeholder |
| `i` | int | Signed 32-bit integer |
| `u` | int | Unsigned 32-bit integer |
| `l` | long | Signed 64-bit integer |
| `k` | long | Unsigned 64-bit integer |
| `f` | float | 32-bit float |
| `d` | double | 64-bit float |
| `S` | String | UTF-8 string |
| `B` | byte[] | Binary blob |
| `O` | Caps | Nested Caps object |

**Usage pattern**:

```java
// Write
Caps caps = new Caps();
caps.writeUInt32(4101);
caps.write("hello");
caps.writeInt32(-1);
byte[] data = caps.serialize();  // native serialization

// Read
Caps caps = new Caps();
caps.parse(data);                // native deserialization
int cmd = caps.at(0).getInt();   // 4101
String msg = caps.at(1).getString(); // "hello"
int val = caps.at(2).getInt();   // -1
```

The native implementation is in `libcaps-jni.so`.

### Connection State Machine

`ClientConnectionState` manages and publishes connection state to the native layer via `clientConnectionStateChange()`:

| State | Meaning |
|---|---|
| 0 | Disconnected |
| 1 | Connecting (accepted but not yet authorized) |
| 2 | Connected (authorized) |

State 1 has a 40-second timeout. If authorization does not complete within 40 seconds, the state reverts to 0 (disconnected).

### Client Connection Persistence

Client connection info is saved to disk so the glasses can reconnect to the last known device after reboot.

**Serialized fields** (Caps format, versioned):

| Index | Field | Since Version |
|---|---|---|
| 0 | Version number (currently 4) | 1 |
| 1 | Client type (1/2/3) | 1 |
| 2 | Service record UUID string | 1 |
| 3 | Client name | 1 |
| 4 | Rokid account | 1 |
| 5 | Client Bluetooth MAC | 2 |
| 6 | Extra type | 4 |

Storage path is configured via `CXRConfig.cxr.client_connection_info_storage_path` with a fallback to `client_connection_info_storage_path_compatible`.

### MTU Configuration

Three MTU values are configured in `CXRConfig.cxr.mtu[]`, one per client type:

| Index | Client Type |
|---|---|
| 0 | Android (RFCOMM) |
| 1 | iPhone GATT |
| 2 | iPhone MFI socket |

### Apple MFI Hardware Detection

The service checks for an Apple MFI coprocessor at:
```
/sys/devices/platform/soc/988000.i2c/i2c-2/2-0010/acp_certificate
```

If this file exists, `mfi=1` and the MFI socket service is started. This allows iPhones to connect via the External Accessory framework.

### Flora IPC

`startFloraService()` is a native method that starts the Flora inter-process communication service. Flora is Rokid's IPC framework (based on pub/sub messaging) used by other glasses-side apps to communicate with the companion phone through CXRService. The Flora service is the bridge that allows system apps on the glasses to send/receive messages to/from the connected phone.

### Native Libraries

| Library | Purpose |
|---|---|
| `libcxr_service_jni.so` | Core CXR service logic: Flora service, protocol handling, auth, audio recording, MFI IAP2, connection state publishing, GATT characteristic write handling, configuration loading |
| `libcaps-jni.so` | Caps binary serialization/deserialization |
| `libcxr-sock-proto-jni.so` | CXRSocketProtocol native implementation (loaded on demand by `CXRSocketProtocol` when `loadLib=true`) |

### How to Interact with CXRService from Another App

CXRService does not expose a bindable AIDL interface. Interaction from other glasses-side apps happens through **Flora IPC** (native pub/sub messaging). The service starts a Flora agent via `startFloraService()` that other apps on the glasses can subscribe to and publish messages through.

From a companion phone app, the interaction path is:
1. Discover the glasses via BLE (look for service UUID `00009100-0000-1000-8000-00805f9b34fb`).
2. Read the RFCOMM Connection or iOS Connection characteristic to get the service record UUID.
3. Connect via RFCOMM (Android) or BLE GATT / MFI (iOS).
4. Use the CXR protocol (`CXRSocketProtocol`) to exchange `request`, `send`, and audio stream messages.

### Key UUIDs

| UUID | Purpose |
|---|---|
| `00009100-0000-1000-8000-00805f9b34fb` | BLE GATT compound service |
| `00009300-0000-1000-8000-00805f9b34fb` | iOS Connection characteristic (read) |
| `00009301-0000-1000-8000-00805f9b34fb` | RFCOMM Connection characteristic (read) |
| `00009201-0000-1000-8000-00805f9b34fb` | iOS Write characteristic |
| `00009202-0000-1000-8000-00805f9b34fb` | iOS Notify characteristic |
| `00009203-0000-1000-8000-00805f9b34fb` | iOS MFI Pairing characteristic |
| `00009204-0000-1000-8000-00805f9b34fb` | iOS MFI Notify characteristic |
| `00009a01-0000-1000-8000-00805f9b34fb` | BLE scan response service data (glass serial number) |
| `00000000-deca-fade-deca-deafdecacafe` | MFI RFCOMM target UUID (glasses connect to phone) |
| `00000000-deca-fade-deca-deafdecacaff` | MFI RFCOMM listen UUID (phone connects to glasses) |
| (random per session) | RFCOMM service record for Android SPP connection |
