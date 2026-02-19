# RokidSysConfig

Privileged system service that manages hardware-level configuration on Rokid AR glasses: LED indicators for battery/Bluetooth status, proximity sensor monitoring for glasses on/off and leg fold/spread detection, device name synchronization with Bluetooth, and auto-shutdown on leg fold timeout.

## Package Info

| Field | Value |
|-------|-------|
| Package | `com.rokid.sysconfig` |
| App Label | `RokidSystemConfig` |
| Version | 1.0 (versionCode 1) |
| SDK | minSdk 32, targetSdk 32 (Android 12L) |
| Shared UID | `android.uid.system` (runs as the system user) |
| Persistent | Yes (system keeps the process alive) |
| Direct Boot Aware | Yes (starts before user unlocks device) |
| Uses Non-SDK API | Yes (accesses hidden/internal Android APIs) |
| Native Libs | Not extracted (`extractNativeLibs=false`) |
| Framework | Uses framework resource ID 1 (standard Android) |

## Manifest

The manifest is minimal. No permissions are declared because the app runs with `android.uid.system` shared UID, which inherits all system-level permissions automatically.

### Components

| Type | Class | Exported | Description |
|------|-------|----------|-------------|
| Application | `.SysCfgApp` | - | Application subclass; starts ConfigService on creation |
| Service | `.ConfigService` | Yes | Main service managing all settings modules |

There are no activities, content providers, or broadcast receivers declared in the manifest. All broadcast receivers are registered dynamically at runtime.

## Architecture

### Boot Flow

1. System starts the app (persistent + direct boot aware).
2. `SysCfgApp.onCreate()` fires and calls `startService(ConfigService.class)`.
3. `ConfigService.onCreate()` creates a background `HandlerThread` ("SysCfg-Thread") and initializes settings modules.
4. Each module gets `onStart()` called immediately.
5. When `USER_UNLOCKED` broadcast arrives, each module gets `onUserUnlocked()`.
6. When `BOOT_COMPLETED` broadcast arrives, each module gets `onBootCompleted()`.
7. When `CONFIGURATION_CHANGED` broadcast arrives, each module gets `onUpdate()`.

### Settings Module System

All configuration subsystems extend the abstract `Settings` base class:

```
Settings (abstract)
  +-- DeviceNameSettings      -- Bluetooth/device name sync
  +-- PsensorObserver.S       -- Proximity sensor wrapper (conditional)
  +-- BluetoothLightSettings  -- Bluetooth status LED control
  +-- BatteryLightSettings    -- Battery/charging LED control
  +-- ShutdownSettings        -- Auto-shutdown on leg fold (conditional)
```

The `Settings` base class defines lifecycle hooks:

| Method | When Called |
|--------|------------|
| `onStart()` | Immediately after module creation |
| `onUserUnlocked()` | When `USER_UNLOCKED` intent received |
| `onBootCompleted()` | When `BOOT_COMPLETED` intent received |
| `onUpdate()` | When `CONFIGURATION_CHANGED` intent received |
| `onGlassesLegStateChanged(boolean isSpread)` | When proximity sensor detects leg fold/spread change |
| `onRelease()` | When ConfigService is destroyed |
| `dump(PrintWriter, String[])` | When service is dumped via `dumpsys` |

### Conditional Modules

Two modules are conditionally loaded based on system properties:

- **PsensorObserver**: Loaded only if `persist.rkd.enablePsensor` is `true` (default: true). When disabled, the service force-sets `vendor.rkd.glasses.is_spread=1` and `vendor.rkd.glasses.is_take_on=1` (treats glasses as always open and worn).
- **ShutdownSettings**: Loaded only if `persist.rkd.leg.fold_timeout.enable` is `true` (default: true).

## Class Analysis

### SysCfgApp

Application subclass. The only thing it does is start `ConfigService` in `onCreate()`. Since the app is marked `persistent`, the system keeps it running.

Source: `com/rokid/sysconfig/SysCfgApp.java`

### ConfigService

The central service that orchestrates all settings modules.

Source: `com/rokid/sysconfig/ConfigService.java`

**Key behavior:**

- Creates a dedicated `HandlerThread` named `"SysCfg-Thread"` for background work.
- Registers broadcast receivers for `USER_UNLOCKED`, `CONFIGURATION_CHANGED`, and `BOOT_COMPLETED`.
- Instantiates and manages all `Settings` subclasses.
- Listens for leg state changes from `PsensorObserver` and propagates them to all modules via `onGlassesLegStateChanged()`.
- Returns `START_STICKY` (value 1) from `onStartCommand()`, ensuring Android restarts the service if killed.
- The `dump()` method restricts access to UID <= 2000 (system/root/shell processes only).

**Broadcast receivers (runtime-registered):**

| Intent Action | Handler |
|---------------|---------|
| `android.intent.action.USER_UNLOCKED` | Sets `mIsSystemReady=true`, calls `onUserUnlocked()` on all modules |
| `android.intent.action.BOOT_COMPLETED` | Sets `mIsSystemReady=true`, calls `onBootCompleted()` on all modules (priority 1) |
| `android.intent.action.CONFIGURATION_CHANGED` | Calls `onUpdate()` on all modules |

### Constants

Source: `com/rokid/sysconfig/Constants.java`

Single constant class:

```
I2C_DIR = SystemProperties.get("ro.boot.i2c.device_node", "")
```

This is the I2C device node path used by both `PsensorObserver` and `ShutdownSettings` to interact with the glasses hardware. The value comes from the boot-time property `ro.boot.i2c.device_node`, typically something like `/sys/devices/platform/soc/a90000.i2c/i2c-2/2-0008`.

### DeviceNameSettings

Synchronizes the Android device name (`Settings.Global "device_name"`) with the Bluetooth adapter name.

Source: `com/rokid/sysconfig/DeviceNameSettings.java`

**Mechanism:**

1. On start, reads the current device name from `Settings.Global.getString("device_name")` and the Bluetooth adapter name via `BluetoothAdapter.getName()`.
2. If they differ, sets the Bluetooth name to match the device name via `BluetoothAdapter.setName()`.
3. Registers a `ContentObserver` on the `device_name` setting (after user unlock or boot complete).
4. Whenever `device_name` changes, re-syncs the Bluetooth adapter name.
5. Also listens for `BLUETOOTH_STATE_CHANGED` -- when Bluetooth turns on (state 12 = `STATE_ON`), re-syncs in case the name was lost.

**System settings accessed:**

| Setting | Type | Usage |
|---------|------|-------|
| `Settings.Global "device_name"` | Read | Source of truth for the device's display name |

### PsensorObserver

Proximity sensor (psensor) observer that monitors the glasses' physical state: whether the legs/temples are spread open or folded, and whether the glasses are being worn (taken on) or not.

Source: `com/rokid/sysconfig/PsensorObserver.java`

**Hardware interface:**

- Uses Linux `UEventObserver` to monitor kernel uevent changes at an extcon (external connector) sysfs path.
- The extcon path is determined by hardware ID:
  - If `ro.boot.i2c.device_node` is set and starts with `/sys`: uses that path + `/extcon`
  - If hardware ID >= 2: `devices/platform/soc/a90000.i2c/i2c-2/2-0008/extcon`
  - Otherwise: `devices/platform/soc/988000.i2c/i2c-0/0-0008/extcon`
- Reads extcon state file for two key signals:
  - `DOCK=1` -- glasses legs are spread open (unfolded)
  - `JIG=1` -- glasses are being worn (taken on / placed on head)

**State tracking and system properties:**

| State | System Property | Values |
|-------|----------------|--------|
| Legs spread/folded | `vendor.rkd.glasses.is_spread` | `"1"` = spread (open), `"0"` = folded |
| Glasses worn/off | `vendor.rkd.glasses.is_take_on` | `"1"` = worn, `"0"` = taken off |

**Broadcast intents sent:**

| Intent Action | Extra Key | Extra Values | When |
|---------------|-----------|--------------|------|
| `com.rokid.sprite.ACTION_LEG_STATUS_CHANGED` | `glasses_leg_state` | `"1"` (spread) / `"0"` (folded) | Leg state changes |
| `com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED` | `glasses_take_state` | `"1"` (on) / `"0"` (off) | Wear state changes |

Both broadcasts are sent with `FLAG_RECEIVER_INCLUDE_BACKGROUND` (0x40000000) to reach background receivers.

**Audio/wake effects:**

- When glasses are put on: wakes the device via `PowerManager.wakeUp()` with reason `"psensor"`, and plays sound effect 18.
- When glasses are taken off: plays sound effect 56.

**Dump command for debug:**

The psensor supports runtime debug toggling via the dump interface:
```
dumpsys activity service com.rokid.sysconfig/.ConfigService psensor debug 1   # enable
dumpsys activity service com.rokid.sysconfig/.ConfigService psensor debug 0   # disable
```

**Inner class `S`:**

A thin `Settings` adapter that bridges the `Settings` lifecycle to `PsensorObserver.start()` / `stop()` calls. Added to ConfigService's settings list so it receives the standard lifecycle events.

### BluetoothLightSettings

Controls the LED indicator based on Bluetooth connection state.

Source: `com/rokid/sysconfig/BluetoothLightSettings.java`

**Bluetooth events monitored:**

| Broadcast | Behavior |
|-----------|----------|
| `CONNECTION_STATE_CHANGED` (state 2 = connected) | Internal state -> `STATE_CONNECT` (3) |
| `CONNECTION_STATE_CHANGED` (other states) | Internal state -> `STATE_DISCONNECT` (4) |
| `STATE_CHANGED` (state 13 = turning off) | Internal state -> `STATE_OFF` (5) |
| `SCAN_MODE_CHANGED` (mode 23 = connectable+discoverable) | Internal state -> `STATE_BOND` (1) |
| `SCAN_MODE_CHANGED` (other modes) | Internal state -> `STATE_BOND_NONE` (2) |
| `BOND_STATE_CHANGED` | Logged only, no LED action |

**LED events via `LightsCtrl` (light type 4 = Bluetooth):**

| Internal State | LED Action |
|----------------|------------|
| `STATE_BOND` (1) -- discoverable | `sendEvent(4, 4010)` -- pairing indicator on |
| `STATE_BOND_NONE` (2) -- not discoverable | `cancelEvent(4, 4010)` -- pairing indicator off |
| `STATE_CONNECT` (3) -- connected | `cancelEvent(4, 4010)`, then `sendEvent(4, 4011)` -- connected indicator on (suppressed if glasses have panel and are worn) |
| `STATE_DISCONNECT` (4) -- disconnected | `cancelEvent(4, 4011)` -- connected indicator off |
| `STATE_OFF` (5) -- BT turning off | Cancel both 4010 and 4011 -- all BT LEDs off |

**Note on panel-equipped glasses:** When the glasses model has a display panel (`DeviceUtils.isGlasseWithPanel()`) and the user is wearing them (`DeviceUtils.isGlassTakeOn()`), the Bluetooth connected LED (4011) is suppressed to avoid visual distraction.

### BatteryLightSettings

Controls the LED indicator based on battery and charging state.

Source: `com/rokid/sysconfig/BatteryLightSettings.java`

**Events monitored:**

| Broadcast | Behavior |
|-----------|----------|
| `ACTION_POWER_CONNECTED` | State -> 1 (charging started), plugged = 7 (USB+AC+wireless) |
| `ACTION_POWER_DISCONNECTED` | State -> 2 (unplugged), plugged = 0 |
| `BATTERY_CHANGED` | State -> 3 (charging, level < 100) or 4 (fully charged, level >= 100) |
| `com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED` | Tracks whether glasses are worn (panel models only) |

**LED events via `LightsCtrl` (light type 1 = Battery):**

| Condition | LED Action |
|-----------|------------|
| Charging, not worn, level < 100 | `cancelEvent(1, 1013)`, `sendEvent(1, 1014)` -- charging indicator |
| Charging, not worn, level = 100 | `cancelEvent(1, 1014)`, `sendEvent(1, 1013)` -- fully charged indicator |
| Charging but glasses worn (take on) | Cancel both 1013 and 1014 -- suppress LEDs while wearing |
| Not charging | Cancel both 1013 and 1014 -- all battery LEDs off |

**Delayed registration:** The `BATTERY_CHANGED` broadcast receiver is not registered until the system is ready (`sys.user.0.ce_available` is true or `BOOT_COMPLETED` fires). `POWER_CONNECTED`/`POWER_DISCONNECTED` are registered immediately.

### ShutdownSettings

Auto-shutdown when the glasses legs are folded for a configurable timeout period while not charging.

Source: `com/rokid/sysconfig/ShutdownSettings.java`

**Shutdown logic:**

1. When legs fold (`onGlassesLegStateChanged(false)`) and the glasses are not charging, an `AlarmManager` alarm is set.
2. The alarm delay is `timeout / 1.75` milliseconds (the division by 1.75 is an adjustment factor, possibly to account for alarm imprecision or staged wake behavior).
3. When the alarm fires (action `com.rokid.glasses.legs.ACTION_SHUTDOWN_TIMEOUT`), the `TimeoutTask` runs:
   - Acquires a `PARTIAL_WAKE_LOCK` named `"leg_hold_timeout"`.
   - Checks charging state one more time.
   - If still not charging: writes `"1"` to the `auto_startup` file in the I2C device node directory, then calls `PowerManager.shutdown(false, "leg_fold_timeout", false)`.
4. If charging begins while the alarm is pending, the alarm is cancelled.
5. If legs spread again while the alarm is pending, the alarm is cancelled and re-evaluated.

**Timeout configuration:**

| Setting | Type | Key | Default |
|---------|------|-----|---------|
| Shutdown timeout | `Settings.System` | `rkd_shutdown_timeout` | -1 (maps to 1,200,000 ms = 20 minutes) |

Timeout value interpretation:
- `-1` (or any negative): 1,200,000 ms (20 minutes)
- `0`: Disabled (no auto-shutdown)
- `> 0`: Value in minutes, converted to milliseconds (`value * 60 * 1000`)

A `ContentObserver` watches for changes to `rkd_shutdown_timeout` and re-evaluates the shutdown timer immediately when the user changes the setting.

**Auto-startup file:** Before shutting down, writes `"1"` to `{I2C_DIR}/auto_startup`. This likely tells the hardware MCU to automatically power the glasses back on when the legs are spread again.

**Broadcast intents:**

| Intent Action | Direction | Purpose |
|---------------|-----------|---------|
| `android.intent.action.ACTION_POWER_CONNECTED` | Received | Cancel pending shutdown alarm |
| `android.intent.action.ACTION_POWER_DISCONNECTED` | Received | Re-evaluate shutdown based on current leg state |
| `com.rokid.glasses.legs.ACTION_SHUTDOWN_TIMEOUT` | Internal (self-sent via PendingIntent) | Trigger the actual shutdown |

## System Properties

### Read by this app

| Property | Type | Default | Usage |
|----------|------|---------|-------|
| `ro.boot.i2c.device_node` | String | `""` | I2C device node path for hardware access |
| `persist.rkd.enablePsensor` | Boolean | `true` | Enable/disable proximity sensor module |
| `persist.rkd.leg.fold_timeout.enable` | Boolean | `true` | Enable/disable auto-shutdown on leg fold |
| `sys.user.0.ce_available` | Boolean | `false` | Whether credential-encrypted storage is available (system readiness indicator) |

### Written by this app

| Property | Values | Usage |
|----------|--------|-------|
| `vendor.rkd.glasses.is_spread` | `"1"` / `"0"` | Current leg spread state (1 = open, 0 = folded) |
| `vendor.rkd.glasses.is_take_on` | `"1"` / `"0"` | Current wear state (1 = worn, 0 = off) |

## Android Settings

| Setting | Provider | Key | Type | Usage |
|---------|----------|-----|------|-------|
| Device name | `Settings.Global` | `device_name` | String | Read by DeviceNameSettings for BT sync |
| Shutdown timeout | `Settings.System` | `rkd_shutdown_timeout` | Int (minutes) | Read by ShutdownSettings (-1=20min default, 0=disabled) |

## Broadcast Intents

### Sent

| Action | Extras | Sender |
|--------|--------|--------|
| `com.rokid.sprite.ACTION_LEG_STATUS_CHANGED` | `glasses_leg_state`: `"1"`/`"0"` | PsensorObserver |
| `com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED` | `glasses_take_state`: `"1"`/`"0"` | PsensorObserver |

### Received

| Action | Handler |
|--------|---------|
| `android.intent.action.USER_UNLOCKED` | ConfigService |
| `android.intent.action.BOOT_COMPLETED` | ConfigService |
| `android.intent.action.CONFIGURATION_CHANGED` | ConfigService |
| `android.intent.action.ACTION_POWER_CONNECTED` | BatteryLightSettings, ShutdownSettings |
| `android.intent.action.ACTION_POWER_DISCONNECTED` | BatteryLightSettings, ShutdownSettings |
| `android.intent.action.BATTERY_CHANGED` | BatteryLightSettings |
| `android.bluetooth.adapter.action.CONNECTION_STATE_CHANGED` | BluetoothLightSettings |
| `android.bluetooth.device.action.BOND_STATE_CHANGED` | BluetoothLightSettings |
| `android.bluetooth.adapter.action.STATE_CHANGED` | BluetoothLightSettings, DeviceNameSettings |
| `android.bluetooth.adapter.action.SCAN_MODE_CHANGED` | BluetoothLightSettings |
| `com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED` | BatteryLightSettings (panel models) |
| `com.rokid.glasses.legs.ACTION_SHUTDOWN_TIMEOUT` | ShutdownSettings (self-delivered alarm) |

## LightsCtrl Event Reference

All LED control goes through `com.rokid.light.LightsCtrl` static methods.

| Light Type | Event ID | Meaning |
|------------|----------|---------|
| 1 (Battery) | 1013 | Battery fully charged indicator |
| 1 (Battery) | 1014 | Battery charging indicator |
| 4 (Bluetooth) | 4010 | Bluetooth pairing/discoverable indicator |
| 4 (Bluetooth) | 4011 | Bluetooth connected indicator |

## Hardware Dependencies

- **`com.rokid.hardware.DeviceUtils`**: Provides `getHwid()` (hardware revision), `isGlasseWithPanel()` (has display panel), `isGlassTakeOn()` (current wear state), `isGlassLegSpread()` (current leg state).
- **`com.rokid.light.LightsCtrl`**: LED control via `sendEvent(type, eventId)` and `cancelEvent(type, eventId)`.
- **Extcon subsystem**: Linux external connector framework at `/sys/{SUB_DIR}/extcon/*/state` -- provides `DOCK=` and `JIG=` signals from the proximity/hall sensor.
- **I2C device node**: `{I2C_DIR}/auto_startup` file for hardware MCU auto-start configuration before shutdown.

## Interacting with ConfigService

### Starting the service

The service starts automatically on boot. To start it manually:

```bash
adb shell am startservice com.rokid.sysconfig/.ConfigService
```

### Dumping state

Since the service is exported and supports `dump()`:

```bash
adb shell dumpsys activity service com.rokid.sysconfig/.ConfigService
```

This outputs the current state of all settings modules (device name, battery, Bluetooth, psensor, shutdown status). Only accessible from UID <= 2000 (root or shell).

### Toggling psensor debug

```bash
adb shell dumpsys activity service com.rokid.sysconfig/.ConfigService psensor debug 1
adb shell dumpsys activity service com.rokid.sysconfig/.ConfigService psensor debug 0
```

### Changing shutdown timeout

```bash
# Set to 10 minutes
adb shell settings put system rkd_shutdown_timeout 10

# Disable auto-shutdown
adb shell settings put system rkd_shutdown_timeout 0

# Reset to default (20 minutes)
adb shell settings delete system rkd_shutdown_timeout
```

### Disabling psensor entirely

```bash
adb shell setprop persist.rkd.enablePsensor false
# Requires app restart (reboot or force-stop + restart)
```

### Reading current glasses state

```bash
adb shell getprop vendor.rkd.glasses.is_spread    # 1=open, 0=folded
adb shell getprop vendor.rkd.glasses.is_take_on    # 1=worn, 0=off
```

## Resources

The app has minimal resources:

| Resource | Description |
|----------|-------------|
| `drawable/ic_launcher` | App icon (hdpi, mdpi) |
| `string/app_name` | "RokidSystemConfig" |
| `dimen/volume_dialog_tap_target_size` | 42dp (likely leftover from a volume dialog that was removed or planned) |
| `style/VolumeUI` | Dark floating theme extending `Theme.DeviceDefault.Settings.Dark.NoActionBar` (also likely vestigial) |

The `VolumeUI` style and `volume_dialog_tap_target_size` dimension suggest this app may have originally contained or been planned to contain a volume UI overlay, but no such component exists in the current codebase.
