# Message Subscription

_Source: https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/ (Chinese, fetched 2026-05-29)._

The CXR-S SDK provides two message subscription modes, allowing glasses-side applications to receive messages from mobile devices:

- **Regular Message Subscription** — one-way message reception.
- **Reply-Enabled Message Subscription** — message reception with the ability to send responses.

## 1. Regular Message Subscription

Developers can subscribe to messages using the `subscribe(String name, MsgCallback cb)` method. When the mobile end sends a message, the SDK delivers it via the `onReceive(String name, Caps args, byte[] data)` callback, which includes the message name, structured parameters (Caps), and optional binary data.

### 1.1 Subscription Method

```kotlin
int subscribe(String name, MsgCallback cb)
```

Parameters:

| Parameter | Description |
|---|---|
| `name` | Name of the message to subscribe to (must match the message name sent by the mobile end) |
| `cb` | Implementation of the `MsgCallback` interface |

Return values:

| Value | Meaning |
|---|---|
| `0` | Subscription successful |
| `-1` | Parameter error |
| `-2` | Duplicate subscription |

### 1.2 Callback Interface

```java
interface MsgCallback {
    void onReceive(String name, Caps args, byte[] value);
}
```

Callback parameters:

| Parameter | Description |
|---|---|
| `name` | Message name |
| `args` | Message parameters (Caps format) |
| `value` | Additional binary data (may be `null`) |

### 1.3 Example Code

```kotlin
private val cxrBridge = CXRServiceBridge()

// Implement the callback interface
private val msgCallback = object : CXRServiceBridge.MsgCallback {
    override fun onReceive(name: String, args: Caps, value: ByteArray?) {
        Log.i("MessageSubscribe", "Received name: $name, args: ${args.size()}, value: $value")
    }
}

// Subscribe to regular messages
cxrBridge.subscribe("glass_test", msgCallback)
```

## 2. Reply-Enabled Message Subscription

Developers can subscribe to messages that support replies using the `subscribe(String name, MsgReplyCallback cb)` method. When the mobile end sends a message, the SDK delivers it via the `onReceive(String name, Caps args, byte[] value, Reply reply)` callback.

### 2.1 Subscription Method

```kotlin
int subscribe(String name, MsgReplyCallback cb)
```

Parameters:

| Parameter | Description |
|---|---|
| `name` | Name of the message to subscribe to (must be agreed upon with the mobile end) |
| `cb` | Instance implementing `MsgReplyCallback` |

Return values:

| Value | Meaning |
|---|---|
| `0` | Subscription successful |
| `-1` | Parameter error |
| `-2` | Duplicate subscription |

### 2.2 Callback Interface

```java
interface MsgReplyCallback {
    void onReceive(String name, Caps args, byte[] value, Reply reply);
}
```

Callback parameters:

| Parameter | Description |
|---|---|
| `name` | Message name |
| `args` | Structured parameters (Caps format) |
| `value` | Optional binary data (may be `null`) |
| `reply` | A `Reply` object used to send a response to the mobile end |

Use `reply.end(Caps ret)` to return response data to the mobile end.

### 2.3 Example Code

```kotlin
private val cxrBridge = CXRServiceBridge()

// Implement the callback interface
val replyCallback = object : CXRServiceBridge.MsgReplyCallback {
    override fun onReceive(name: String, args: Caps, value: ByteArray?, reply: Reply?) {
        // Receive the message
        Log.d("MessageSubscribe", "Received name: $name, args: ${args.size()}, value: $value")
        // Construct the reply
        val replyArgs = Caps()
        replyArgs.write("Received Message and Reply")
        // Send the reply
        reply?.end(replyArgs)
    }
}

// Subscribe to reply-enabled messages
cxrBridge.subscribe("glass_test", replyCallback)
```
