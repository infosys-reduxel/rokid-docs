# AI Interaction

## 1. Monitoring AI events on the glasses

The CXR-M SDK allows you to configure AiEventListenerlisteners to receive AI events from Glasses.

```kotlin
private val aiEventListener by lazy {
    object : AiEventListener{
        override fun onAiKeyDown() {
            Log.i("AiModel", "When Ai Started from Glasses")
        }

        override fun onAiKeyUp() {
            //TODO("callback for feature")
        }

        override fun onAiExit() {
            Log.i("AiModel", "When Ai Exited from Glasses")
        }

    }
}

fun setAiListener(set: Boolean){
    if (set){
        CxrApi.getInstance().setAiEventListener(aiEventListener)
    }else{
        CxrApi.getInstance().setAiEventListener(null)
    }
}
```

## 2. Send an Exit event to the glasses.

The mobile device can proactively send an exit AI event public CxrStatus sendExitEvent()to exit the AI ​​process on the glasses.

```kotlin
fun exitAi(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().sendExitEvent()
}
```

## 3. Send ASR content to the glasses

After receiving the ASR result, the mobile device can public CxrStatus sendAsrContent(String content)push the content to the glasses. If the ASR result is empty, public CxrStatus notifyAsrNone()a notification must also be sent to the glasses, along with notifications public CxrStatus notifyAsrError()for any ASR error. Finally, a notification must be sent to the glasses when the ASR recognition process is complete public CxrStatus notifyAsrEnd().

```kotlin
enum class ASRStatus{
    SUCCESS,
    FAIL,
    END
}
fun sendAsrResult(result: String?, status: ASRStatus){
    when(status){
        ASRStatus.SUCCESS -> {
            if (TextUtils.isEmpty(result)){
                CxrApi.getInstance().notifyAsrNone()
            }else{
                CxrApi.getInstance().sendAsrContent(result)
            }
        }
        ASRStatus.FAIL -> {
            CxrApi.getInstance().notifyAsrError()
        }
        ASRStatus.END -> {
            CxrApi.getInstance().notifyAsrEnd()
        }
    }
}
```

## 4. Camera Operations in the AI ​​Process

During the AI ​​process, if you need to obtain real-time images from Glasses' camera, you can first open public CxrStatus openGlassCamera(int width, int height)the camera (optional), then take public CxrStatus takeGlassPhoto(int width, int height, PhotoResultCallback callback)a picture, and PhotoResultCallbackobtain the shooting results.

```kotlin
private val photoResultCallback by lazy { 
    object : PhotoResultCallback{
        override fun onPhotoResult(
            p0: ValueUtil.CxrStatus?,
            p1: ByteArray?
        ) {
            if (p0 == ValueUtil.CxrStatus.REQUEST_SUCCEED){
                Log.i("AiModel", "Photo Result : ${p1?.size}")
            }else{
                Log.i("AiModel", "Photo Result Error: $p0")
            }
        }
    }
}

fun aiOpenCamera(){
    CxrApi.getInstance().openGlassCamera(640, 480)
}

fun aiTakePhoto(){
    CxrApi.getInstance().takeGlassPhoto(640,480, photoResultCallback)
}
```

## 5. AI return results in the AI ​​process

After the ASR results and images are passed to the AI, the AI ​​returns the results. Typically, the application will choose to use TTS (Text-to-Speech) for voice playback. public CxrStatus sendTtsContent(String content)The AI ​​results can be sent to the glasses, and fun CxrStatus notifyTtsAudioFinished()the glasses can be notified that the TTS playback has ended.

```kotlin
fun sendTTS(text: String){
    CxrApi.getInstance().sendTtsContent(text)
}

fun notifyTTSEnd(){
    CxrApi.getInstance().notifyTtsAudioFinished()
}
```

## 6. AI Process Error Handling

No network:public CxrStatus notifyNoNetwork()
AI request failed:public CxrStatus notifyAiError()
