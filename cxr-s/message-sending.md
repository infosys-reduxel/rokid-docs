# Message Sending

The CXR-S SDK provides two message-sending methods, allowing glasses-side applications to send data to connected mobile devices:

Basic Message Sending: Sending structured data (in Caps format)
Binary Message Sending: Sending structured data + binary content

## 1. Basic Message Sending

### 1.1 Interface Definition

```kotlin
int sendMessage(String name, Caps args);
```

Parameter Descriptions:

name: Message name (must be agreed upon with the mobile end)
args: Structured message parameters (Caps object)
Return Values:

0: Sending successful
-1: Parameter error
-3: Internal error

### 1.2 Example Code

```kotlin
// Assuming CXRServiceBridge is initialized and available
val cxrServiceBridge = CXRServiceBridge()
 
fun sendExampleMessage() {
    // 1. Create a Caps object and populate it with data
    val args = Caps()
    args.write("send_message")  // Write a string message
    args.writeUInt32(5)        // Write an unsigned 32-bit integer parameter (example value)
 
    // 2. Call sendMessage to send the message (simplified interface)
    val result = cxrServiceBridge.sendMessage(
        "message_channel",  // Message channel name (defined according to the actual protocol)
        args                // Parameter object
    )
 
    // 3. Handle the sending result
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
int sendMessage(String name, Caps args, byte[] data, int offset, int size);
```

Parameter Descriptions:

name: Message name (must be agreed upon with the mobile end)
args: Structured message parameters (Caps object)
data: Binary data array
offset: Starting offset of the data
size: Length of the data to be sent
Return Values:

0: Sending successful
-1: Parameter error
-3: Internal error

### 2.2 Example Code

```kotlin
// Assuming CXRServiceBridge is initialized and available
val cxrServiceBridge = CXRServiceBridge()
 
fun sendExampleMessage() {
    // 1. Create a Caps object and populate it with data
    val args = Caps()
    args.write("send_message")  // Write a string message
    args.writeUInt32(5)        // Write an unsigned 32-bit integer parameter (example value)
 
    // 2. Prepare binary data to be sent (example: empty data)
    val data = byteArrayOf()  // Replace with the data to be sent in actual scenarios
    val offset = 0            // Data offset
    val size = data.size      // Data size
     
    // 3. Call sendMessage to send the data
    val result = cxrServiceBridge.sendMessage(
        "message_channel",  // Message channel name (defined according to the actual protocol)
        args,               // Parameter object
        data,               // Binary data
        offset,             // Starting offset of the data
        size                // Data length
    )
     
    // 4. Handle the sending result
    if (result == 0) { 
        Log.d("send_message", "Send message successful")
    } else {
        Log.d("send_message", "Send message error: $result")
    }
}
```
