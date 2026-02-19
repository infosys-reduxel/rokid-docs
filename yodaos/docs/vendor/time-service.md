# TimeService (Qualcomm Time Sync)

**Package:** `com.qualcomm.timeservice`
**APK:** `TimeService.apk`
**Min SDK:** 31 | **Target SDK:** 33

### Purpose

Synchronizes Android user-set time to the Qualcomm modem/baseband time via a local Unix socket (`time_genoff`). When the user changes the system time, this receiver propagates it to the modem's `ATS_USER` time base, so the baseband clock stays in sync.

### Permissions

- `android.permission.WAKE_LOCK`

### Exported Components

None exported (`exported="false"`).

### Components

| Type | Name | Exported | Notes |
|------|------|----------|-------|
| Receiver | `TimeServiceBroadcastReceiver` | No | Direct boot aware. Listens for TIME_SET. |

### Intent Filters

- `android.intent.action.TIME_SET`

### Behavior Summary

1. When the system time is changed, Android broadcasts `TIME_SET`
2. TimeServiceBroadcastReceiver acquires a wake lock
3. Opens local socket `time_genoff` (abstract namespace)
4. Sends a 24-byte payload: `{base=2 (ATS_USER), unit=1 (MSEC), operation=0 (T_SET), result=-1, value=currentTimeMillis}`
5. Reads 28-byte response, checks result code at offset 24
6. Releases wake lock

### Socket Protocol

Socket: `time_genoff` (LocalSocket, abstract namespace)

**Send (24 bytes):**
| Offset | Size | Field | Value |
|--------|------|-------|-------|
| 0 | 4 | base | 2 (ATS_USER_BASE) |
| 4 | 4 | unit | 1 (TIME_MSEC_UNIT) |
| 8 | 4 | operation | 0 (T_SET_OPERATION) |
| 12 | 4 | result | -1 (initial) |
| 16 | 8 | value | System.currentTimeMillis() |

**Receive (28 bytes):**
| Offset | Size | Field |
|--------|------|-------|
| 24 | 4 | result code (0+ = success, negative = error) |

All integers are little-endian.

### Smali/Java Structure

```
com.qualcomm.timeservice/
  TimeServiceBroadcastReceiver           -- Main receiver, handles TIME_SET
  TimeServiceBroadcastReceiver$SendData  -- Inner class for socket message encoding
  R                                      -- Resources
```

### Development Relevance

Low. This is a modem time sync service. It uses a Unix socket only accessible to system processes. The `time_genoff` socket is served by the Qualcomm time daemon. Third-party apps cannot interact with it.
