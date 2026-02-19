Data Interaction
1. Send data to the glasses.
Before sending data to the glasses, ensure that the device is connected via Bluetooth.

The glasses client can public CxrStatus sendStream(CxrStreamType type, byte[] stream, String fileName, SendStatusCallback: cb)send streaming data to the glasses client via a method. Successful data transmission can be detected in the callback SendStatusCallbackof the interface fun onSendStreamSucceed(), and fun onSendFailed(p0: ValueUtil.CxrSendErrorCode?)the reason for failure can be retrieved within the callback.

private val streamCallback = object : SendStatusCallback {
    override fun onSendSucceed() {
        Log.i("StreamModel", "onSendSucceed")
    }

    override fun onSendFailed(p0: ValueUtil.CxrSendErrorCode?) {
        Log.i("StreamModel", "onSendFailed")
    }
}

fun sendStream(type: ValueUtil.CxrStreamType, stream: ByteArray, fileName: String): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendStream(type, stream, fileName, streamCallback)
}
2. Read unsynchronized media files on the glasses.
To read media files that are not synchronized on the glasses, the device must be in Bluetooth connection mode.

The method can be used pbulic CxrStatus getUnsyncNum()to retrieve unsynchronized media files on the glasses. The number of unsynchronized media files can be returned in the callback UnsyncNumResultCallbackof the interface .fun onUnsyncNumResult(p0: ValueUtil.CxrStatus?,p1: Int,p2: Int,p3: Int)

private val unSyncNumResultCallback = object : UnsyncNumResultCallback{
    override fun onUnsyncNumResult(
        p0: ValueUtil.CxrStatus?,
        p1: Int,
        p2: Int,
        p3: Int
    ) {
        Log.i("SyncFiles", "onUnSyncNumResult: status $p0 audio num $p1, picture num $p2, video num $p3")
    }

}

fun getUnSyncNum(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().getUnsyncNum(unSyncNumResultCallback)
}
3. Media file update on the monitoring glasses.
MediaFilesUpdateListenerYou can obtain updates to media files on the glasses by setting up an interface.

private val mediaFilesUpdateListener = object : MediaFilesUpdateListener{
    override fun onMediaFilesUpdated() {
        Log.i("SyncFiles", "onMediaFilesUpdated")
    }
}

fun setMediaFilesUpdateListener(set: Boolean) {
    if (set) {
        CxrApi.getInstance().setMediaFilesUpdateListener(mediaFilesUpdateListener)
    } else {
        CxrApi.getInstance().setMediaFilesUpdateListener(null)
    }
}
4. Synchronize media files
Synchronizing media files from the glasses requires a Wi-Fi communication module. Please initialize the Wi-Fi communication module before synchronization.

public boolean startSync(String savePath, CxrMediaType[] types, SyncStatusCallbackcallback)You can start synchronizing media files using this method.

in:

savaPathFile storage location (please note file management permissions)

typesIt CxrMediaTypesupports simultaneous synchronization of multiple types.

CxrMediaType:
AUDIO: Audio files
PICTURE: Image file
VIDEO: Video file
ALL: All files
callback: CxrSyncCallback, synchronous callback

CxrSyncCallback:
fun onSyncStart()Callback when synchronization begins
fun onSingleFileSynced(fileName: String?)Callback when a single file <fileName> is successfully synchronized.
fun onSyncFailed()Callback when synchronization fails
fun onSyncFinished()Callback upon synchronization completion
Return value: Returns true if the request is successful, and false if the request fails.

Example:

private var isSyncing = false

private val cxrSyncCallback by lazy {
    object : SyncStatusCallback {
        override fun onSyncStart() {
            isSyncing = true
            Toast.makeText(this@MainActivity, "Sync Start", Toast.LENGTH_SHORT).show()
        }

        override fun onSingleFileSynced(p0: String?) {
            Toast.makeText(this@MainActivity, "Single File Synced: $p0", Toast.LENGTH_SHORT).show()
        }

        override fun onSyncFailed() {
            isSyncing = false
            Toast.makeText(this@MainActivity, "Sync Failed", Toast.LENGTH_SHORT).show()
        }

        override fun onSyncFinished() {
            isSyncing = false
            Toast.makeText(this@MainActivity, "Sync Finished", Toast.LENGTH_SHORT).show()
        }
    }
}

/**
 * Sync Media Files
 * @param savePath Save Path
 * @param mediaType Media Type
 * @return [Boolean]: true: Sync Media Files Request Success, false: Sync Media Files Request Failed
 */
private fun syncMediaFiles(savePath: String, mediaTypes: Array<ValueUtil.CxrMediaType>): Boolean {
    if (isSyncing) return false
    // Send Media Files
    return CxrApi.getInstance().startSync(savePath, mediaTypes, cxrSyncCallback)
}
You can also use public boolean syncSingleFile(String savePath, CxrMediaType type, String filePath, SyncStatusCallback callback)the interface to synchronize a single media file of a specified type.

in:

filePath: Path to the file on the glasses
private fun syncSingleFiles(savePath: String, mediaType: ValueUtil.CxrMediaType, filePath: String): Boolean {
    return CxrApi.getInstance().syncSingleFile(savePath, mediaType, filePath, cxrSyncCallback)
}
stopSync()Synchronization can be stopped using methods during the synchronization process.

private fun stopSync() {
    CxrApi.getInstance().stopSync()
}
5. Recording function
Audio recorded on YodaOS-Sprite can AudioStreamListenerbe listened to via an interface.

private val audioStreamListener by lazy {
    object : AudioStreamListener {
        override fun onStartAudioStream(p0: Int, p1: String?) {
            Log.i("AudioModel", "onStartAudioStream code type: $p0, streamType: $p1")
        }

        override fun onAudioStream(p0: ByteArray?, p1: Int, p2: Int) {
            Log.i("AudioModel", "AudioStream：${p0?.size} data offset: $p1, data length: $p2")
        }

    }
}

fun setAudioStreamListener(set: Boolean) {
    if (set) {
        CxrApi.getInstance().setAudioStreamListener(audioStreamListener)
    } else {
        CxrApi.getInstance().setAudioStreamListener(null)
    }
}
In addition to monitoring the audio stream during the AI ​​process, the CXR-M SDK can also public CxrStatus openAudioRecord(int codecType, String streamType)notify the glasses to start recording and public CxrStatus closeAudioRecord(String streamType)stop recording via methods.
