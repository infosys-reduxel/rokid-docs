# Manage Device Connection
Before reading this chapter, please ensure that you have thoroughly understood the content of the “SDK Import” chapter.

## Monitor Mobile Connection Status
The CXR-S SDK can monitor the connection status from the mobile end by setting up a StatusListener.

```kotlin
private val TAG = "StatusListener"
private val cxrBridge = CXRServiceBridge(context)
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
        // Audio noise level update (new in SDK v1.1+, required override — see release-notes.md)
        override fun onAudioNoise(level: Float) {
            Log.d(TAG, "Audio noise level: $level")
        }
    }
}
fun setStatusListener() {
    cxrBridge.setStatusListener(statusListener)
}
```

Parameter Descriptions:

name: Device name
type: Device type (1 - Android, 2 - iPhone, 3 - Unknown)
health: Percentage of successfully sent ARTC frames (0.0 - 1.0)
reset: Whether a frame queue reset has occurred
level: Audio noise level reported by the glasses

> **SDK v1.1+ note:** `CXRServiceBridge` now takes a `Context` argument in its constructor (`CXRServiceBridge(context)`), replacing the no-argument constructor from SDK v1.0. `StatusListener` also gained the `onAudioNoise(Float)` callback in v1.1 — any implementation must override it. Both changes are binary-diff-confirmed, not from an official changelog — see [release-notes.md](release-notes.md).
