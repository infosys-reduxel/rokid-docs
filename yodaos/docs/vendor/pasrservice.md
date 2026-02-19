# pasrservice (PASR Memory Manager)

**Package:** `com.qti.pasrservice`
**APK:** `pasrservice.apk`
**Min SDK:** 32 | **Target SDK:** 33
**Process:** `.pasr` (persistent)

### Purpose

PASR = Partial Array Self-Refresh. This is a Qualcomm DDR memory power-saving service. It takes DDR memory segments offline when the screen is off or device is idle, and brings them back online when the screen turns on. This reduces power draw from unused RAM regions.

Controlled by system properties:
- `vendor.pasr.enabled` -- must be `"true"` for the service to do anything
- `vendor.pasr.activemode.enabled` -- enables "active mode" (offline segments one-by-one instead of all at once)
- `vendor.pasr.screenoff_offline_trigger_sec` -- delay in seconds before offlining memory after screen off (0 = no timer)

### Permissions

- `android.permission.WAKE_LOCK`
- `android.permission.DEVICE_POWER`
- `android.permission.RECEIVE_BOOT_COMPLETED`

### Uses Library

- `vendor.qti.memory.pasrmanager.V1_0.IPasrManager` -- HIDL HAL interface for memory segment management

### Exported Components

None exported. All components are internal (`exported="false"`).

### Components

| Type | Name | Exported | Notes |
|------|------|----------|-------|
| Receiver | `PASRReceiver` | No | Boot receiver. Starts PASRService on `BOOT_COMPLETED`. |
| Service | `PASRService` | No | Main service. Registers DozeReceiver and PASRAlarmReceiver dynamically. |

### Intent Filters

**Static (manifest):**
- `android.intent.action.BOOT_COMPLETED` (PASRReceiver)

**Dynamic (registered in code):**
- `android.os.action.DEVICE_IDLE_MODE_CHANGED`
- `android.intent.action.SCREEN_ON`
- `android.intent.action.SCREEN_OFF`
- `quic.alarms.SCREEN_OFF_ALARM` (internal alarm action)

### Behavior Summary

1. On boot: PASRReceiver starts PASRService
2. DozeReceiver listens for screen on/off and doze mode changes, forwards to PASRService
3. On screen off: Takes memory segments offline via IPasrManager HAL (`attemptOffline`)
4. On device idle (deep doze): Enters critical state, takes ALL segments offline (`attemptOfflineAll`)
5. On screen on: Exits critical state, brings memory segments back online (`attemptOnline`/`attemptOnlineAll`)
6. Optional alarm-based delay before offlining (configurable via system property)

### HIDL HAL Interface: `IPasrManager`

Service name: `"pasrhal"`
Interface: `vendor.qti.memory.pasrmanager@1.0::IPasrManager`

Key methods:
- `attemptOffline(int src, int priority)` -- take one segment offline
- `attemptOnline(int src, int priority)` -- bring one segment online
- `attemptOfflineAll(int src, int priority)` -- take all segments offline
- `attemptOnlineAll(int src, int priority)` -- bring all segments online
- `getOnlineCount()` / `getOfflineCount()` -- get segment counts
- `getPasrInfo()` -- returns `PasrInfo { ddr_size, granule, num_blocks, min_free_mem }`
- `stateEnter(int src, int priority, byte flag)` -- enter critical state
- `stateExit(int src)` -- exit critical state

Enums:
- `PasrState`: MEMORY_ONLINE(0), MEMORY_OFFLINE(1), MEMORY_UNKNOWN(2)
- `PasrStatus`: ERROR(-10), INCOMPLETE_ONLINE(-2), INCOMPLETE_OFFLINE(-1), OFFLINE(1), ONLINE(2), SUCCESS(3)
- `PasrPriority`: NONE(0), LOW(1), CRITICAL(2)
- `PasrSrc`: PSI(0), POWER(1)

### Smali/Java Structure

```
com.qti.pasrservice/
  PASRReceiver          -- BroadcastReceiver, auto-starts service on boot
  PASRService           -- Main service, orchestrates memory online/offline
  PASRAlarmReceiver     -- Handles screen-off timer alarm
  DozeReceiver          -- Forwards screen/doze intents to PASRService
  R                     -- Resources

vendor.qti.memory.pasrmanager.V1_0/
  IPasrManager          -- HIDL interface (with Proxy and Stub)
  PasrInfo              -- Data class: ddr_size, granule, num_blocks, min_free_mem
  PasrState             -- Enum: MEMORY_ONLINE, MEMORY_OFFLINE, MEMORY_UNKNOWN
  PasrStatus            -- Enum: ERROR, INCOMPLETE_ONLINE/OFFLINE, OFFLINE, ONLINE, SUCCESS
  PasrPriority          -- Enum: NONE, LOW, CRITICAL
  PasrSrc               -- Enum: PSI, POWER

android.hidl.base/     -- Standard HIDL base classes
android.hidl.manager/  -- Standard HIDL service manager
```

### Development Relevance

Low. This is a background memory optimization service. It does not expose any usable APIs to third-party apps. It communicates with a vendor HIDL HAL that is not accessible from app processes.
