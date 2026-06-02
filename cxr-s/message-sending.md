# Message Sending

_Source: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/ (Chinese, fetched 2026-05-29)._

The CXR-S SDK provides two message-sending methods, allowing glasses-side applications to send data to connected mobile devices:

- **Basic Message Sending** — sends structured data (Caps format).
- **Binary Message Sending** — sends structured data together with a binary payload.

## 1. Basic Message Sending

### 1.1 Interface Definition

```kotlin
int sendMessage(String name, Caps args)
```

Parameters:

| Parameter | Description |
|---|---|
| `name` | Message name (must be agreed upon with the mobile end) |
| `args` | Structured message parameters (Caps object) |

Return values:

| Value | Meaning |
|---|---|
| `0` | Sending successful |
| `-1` | Parameter error |
| `-3` | Internal error |

### 1.2 Example Code

```kotlin
// Assuming CXRServiceBridge is initialized and available
val cxrServiceBridge = CXRServiceBridge()

fun sendExampleMessage() {
    // 1. Create a Caps object and populate it with data
    val args = Caps()
    args.write("send_message")  // Write a string message
    args.writeUInt32(5)         // Write an unsigned 32-bit integer (example value)

    // 2. Call sendMessage
    val result = cxrServiceBridge.sendMessage(
        "message_channel",  // Channel name (agreed with mobile end)
        args                // Parameter object
    )

    // 3. Handle the result
    if (result == 0) {
        Log.d("send_message", "Send message successful")
    } else {
        Log.d("send_message", "Send message error: $result")
    }
}
```

## 2. Binary Message Sending

### 2.1 Interface Definition

```kotlin
int sendMessage(String name, Caps args, byte[] data, int offset, int size)
```

Parameters:

| Parameter | Description |
|---|---|
| `name` | Message name (must be agreed upon with the mobile end) |
| `args` | Structured message parameters (Caps object) |
| `data` | Binary data array |
| `offset` | Starting offset within `data` |
| `size` | Number of bytes to send |

Return values:

| Value | Meaning |
|---|---|
| `0` | Sending successful |
| `-1` | Parameter error |
| `-3` | Internal error |

### 2.2 Example Code

```kotlin
// Assuming CXRServiceBridge is initialized and available
val cxrServiceBridge = CXRServiceBridge()

fun sendExampleMessage() {
    // 1. Create a Caps object and populate it with data
    val args = Caps()
    args.write("send_message")  // Write a string message
    args.writeUInt32(5)         // Write an unsigned 32-bit integer (example value)

    // 2. Prepare binary data (replace with actual data in production)
    val data = byteArrayOf()
    val offset = 0
    val size = data.size

    // 3. Call sendMessage with binary payload
    val result = cxrServiceBridge.sendMessage(
        "message_channel",  // Channel name (agreed with mobile end)
        args,               // Parameter object
        data,               // Binary data
        offset,             // Starting offset
        size                // Data length
    )

    // 4. Handle the result
    if (result == 0) {
        Log.d("send_message", "Send message successful")
    } else {
        Log.d("send_message", "Send message error: $result")
    }
}
```
