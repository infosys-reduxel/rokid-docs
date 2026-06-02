# Manage Device Connection

_Source: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/ (Chinese, fetched 2026-05-29)._

Before reading this chapter, ensure you have thoroughly understood the [SDK Import](sdk-import.md) chapter.

## Monitor Mobile Connection Status

The CXR-S SDK can monitor the connection status from the mobile end by setting up a `StatusListener`.

```kotlin
private val TAG = "StatusListener"
private val cxrBridge = CXRServiceBridge()

// Connection status listener
private val statusListener by lazy {
    object : CXRServiceBridge.StatusListener {
        // Callback for successful connection
        override fun onConnected(name: String, type: Int) {
            Log.d(TAG, "Connected to $name $type")
        }
        // Callback for disconnection
        override fun onDisconnected() {
            Log.d(TAG, "Disconnected")
        }
        // ARTC status update
        override fun onARTCStatus(health: Float, reset: Boolean) {
            Log.d(TAG, "ARTC Status: Health: ${(health * 100).toInt()}%")
        }
    }
}

fun setStatusListener() {
    cxrBridge.setStatusListener(statusListener)
}
```

### Callback Parameters

**`onConnected(name, type)`**

| Parameter | Type | Description |
|---|---|---|
| `name` | `String` | Device name |
| `type` | `Int` | Device type: `1` = Android, `2` = iPhone, `3` = Unknown |

**`onARTCStatus(health, reset)`**

| Parameter | Type | Description |
|---|---|---|
| `health` | `Float` | Percentage of successfully sent ARTC frames (0.0 – 1.0) |
| `reset` | `Boolean` | Whether a frame-queue reset has occurred |
