# Message Subscription
The CXR-S SDK provides two message subscription modes, allowing glasses-side applications to receive messages from mobile devices:

Regular Message Subscription: One-way message reception
Reply-Enabled Message Subscription: Message reception with the ability to send responses
## 1. Regular Message Subscription
Developers can subscribe to glasses-side messages using the subscribe(String name, MsgCallback cb) method provided by the CXR-S SDK. When the mobile end sends a message, the SDK notifies the developer through the onReceive(String name, Caps args, byte[] data) callback in the MsgCallback interface, which includes the message name, structured parameters (Caps), and optional binary data.

### 1.1 Subscription Method

```kotlin
int subscribe(String name, MsgCallback cb)
```

Parameters:

name: The name of the message to subscribe to (must match the message name sent by the mobile end)
cb: The implementation of the message callback interface
Return Values:

0: Subscription successful
-1: Parameter error
-2: Duplicate subscription
### 1.2 Callback Interface

```java
interface MsgCallback {
    void onReceive(String name, Caps args, byte[] value);
}
```

Callback Parameters:

name: Message name
args: Message parameters (in Caps format)
value: Additional binary data, which may be
### 1.3 Example Code

```kotlin
private val cxrBridge = CXRServiceBridge()
// Implement the callback interface
private val msgCallback = object : CXRServiceBridge.MsgCallback {
    override fun onReceive(name: String, args: Caps, value: ByteArray?) {
        Log.i("MessageSubscribe", "Received name: $name, args: ${args.size()}, value: ${value}")
    }
}
 
// Call subscribe to subscribe to regular messages
cxrBridge.subscribe("glass_test", msgCallback)
```

## 2. Reply-Enabled Message Subscription
Developers can subscribe to messages that support replies using the subscribe(String name, MsgReplyCallback cb) method provided by the CXR-S SDK. When the mobile end sends a message, the SDK notifies the developer through the onReceive(String name, Caps args, byte[] value, Reply reply) callback in the MsgReplyCallback interface.

### 2.1 Subscription Method

```kotlin
int subscribe(String name, MsgReplyCallback cb)
```

Parameter Descriptions:

name: The name of the message to subscribe to (must be agreed upon with the mobile end)
cb: An instance implementing the MsgReplyCallback interface
Return Values:

0: Subscription successful
-1: Parameter error
-2: Duplicate subscription
### 2.2 Callback Interface

```java
interface MsgReplyCallback {
    void onReceive(String name, Caps args, byte[] value, Reply reply);
}
```

Callback Parameters:

name: Message name
args: Structured parameters (in Caps format)
value: Optional binary data, which may be
reply: A Reply object used to send a response to the mobile end
Developers can return response data using the reply.end(Caps ret) method.

### 2.3 Example Code

```kotlin
private val cxrBridge = CXRServiceBridge()
// Implement the callback interface
public val replyCallback = object : CXRServiceBridge.MsgReplyCallback {
    override fun onReceive(name: String, args: Caps, value: ByteArray?, reply: Reply?) {
        // Receive the message
        Log.d("MessageSubscribe", "Received name: $name, args: ${args.size()}, value: ${value}")
        // Construct the reply message
        val replyArgs = Caps()
        replyArgs.write("Received Message and Reply")
        // Send the reply message
        reply?.end(replyArgs)
    }
}
// Call subscribe to subscribe to reply-enabled messages
cxrBridge.subscribe("glass_test", replyCallback)
```
