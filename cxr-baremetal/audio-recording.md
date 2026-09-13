# Audio Recording (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/13083daf77dd40bf84cf5c59711e987a.html> (Chinese, fetched 2026-05-29)
>
> **Doc version: v0.0.1 (2026-03-01)**

<!-- REFRESH-PENDING (2026-07-17, corrected): Upstream index at https://developerdoc.rokid.com/sdk lists 眼镜端裸机开发 at v1.0.0 (updated 2026-06-05, "重构文档" + new sample project source). An earlier same-cycle attempt wrongly reported this as unretrievable (SPA shell only) — a second attempt confirmed the real v1.0.0 body content IS retrievable via the Firecrawl REST API called directly (curl, through the required proxy) plus an executeJavascript click-through of the site's antd Tree nav. The doc has also moved to a new custom.rokid.com workspace (ff28c865a9634876be98cbc293588460, superseding 57e35cd3ae294d16b1b8fc8dcbb1b7c7 — a genuine site rebuild, not just a version bump), with 8 sub-pages total; this file maps to 原始音频. (Sibling pages 简介/快速开始 → development-guide.md, 按键与佩戴折叠 → key-broadcasts.md; 4 more pages — Sample工程与页面说明, 眼镜UI设计规范, 拍照, 录像, IMU与传感器 — have no local doc yet, new P0 candidates.) The actual blocker is that the `baoyu-translate` skill's SKILL.md is not installed in this environment (only the EXTEND.md glossary is present) — translation was correctly not attempted inline per that agent's instructions, so no fabricated content landed here. Raw Chinese source was staged in that session's ephemeral scratchpad and was not committed (per repo policy against committing untranslated upstream copies) — it will need to be re-fetched once the skill gap is fixed. Do not bump the Doc version line above until real v1.0.0 content is translated and lands. UPDATE (2026-08-03): the sibling 简介 (intro) sub-page was successfully retrieved this cycle and landed in [development-guide.md](./development-guide.md), but 原始音频 itself is still not retrievable — every click-through technique tried across multiple cycles (plain scrape, curl+executeJavascript, dispatchEvent MouseEvent sequences on the antd Tree nav) returns only the 简介 body; the "next page" link in the rendered HTML has no `href` (pure JS click handler). Still open. -->

This page covers 8-channel microphone capture on Rokid Glasses from a bare-metal Android app. For the broader context (dev environment, ADB enablement, reserved system gestures), see the [Bare-Metal Development Guide](./development-guide.md). The reference implementation reuses the `KeyReceiver` from [Button Broadcasts](./key-broadcasts.md) to start and stop recording on a side-button click.

## Audio recording on Rokid Glasses

Rokid Glasses exposes an **8-channel audio recording configuration** (`ChannelMask` set to `0x6000FC`). Developers can read all 8 channels via Android's `AudioRecord` channel API at a **16 kHz** sample rate, **16-bit PCM**.

### Channel layout

| Channels | Source |
|----------|--------|
| **0, 1** | **Post-algorithm audio** — output of the on-device AEC / beamforming pipeline. Use these channels for downstream speech recognition or transcription. |
| **2, 3, 4, 5** | **Raw audio** from the 4 microphone capsules on Rokid Glasses. Use these for custom DSP, beamforming experiments, or per-capsule analysis. |
| **6, 7** | **Hardware echo reference** — playback reference signal used by the AEC. Useful when implementing your own echo cancellation in place of the system's. |

### Reference implementation

```kotlin
package com.rokid.cxrssdksamples.activities.audioRecord

import android.Manifest
import android.annotation.SuppressLint
import android.app.Activity
import android.content.IntentFilter
import android.media.AudioFormat
import android.media.AudioRecord
import android.media.MediaRecorder
import android.os.Environment
import android.util.Log
import androidx.annotation.RequiresPermission
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import com.rokid.cxrssdksamples.activities.keys.KeyReceiver
import com.rokid.cxrssdksamples.activities.keys.KeyReceiverListener
import com.rokid.cxrssdksamples.activities.keys.KeyType
import com.rokid.cxrssdksamples.default.CONSTANT
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import java.io.File
import java.io.FileOutputStream
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

interface PermissionNeed {
    fun needPermission()
}

@SuppressLint("MissingPermission")
class AudioRecordViewModel : ViewModel() {
    private val _isRecording = MutableStateFlow(false)
    val isRecording = _isRecording.asStateFlow()
    private val _isPreparing = MutableStateFlow(false)
    val isPreparing = _isPreparing.asStateFlow()

    var permissionNeed: PermissionNeed? = null

    private var keyReceiver: KeyReceiver = KeyReceiver().apply {
        listener = object : KeyReceiverListener {
            override fun onReceive(keyType: KeyType) {
                when (keyType) {
                    KeyType.CLICK -> {
                        Log.e("AudioRecordActivity", "Click")
                        if (!permissionGranted.value) {
                            permissionNeed?.needPermission()
                        } else {
                            if (_isRecording.value) {
                                _isRecording.value = false
                                stopRecording()
                            } else {
                                _isRecording.value = true
                                startAudioRecord()
                            }
                        }
                    }
                    KeyType.BUTTON_DOWN -> Log.e("AudioRecordActivity", "Button down")
                    KeyType.BUTTON_UP -> Log.e("AudioRecordActivity", "Button up")
                    KeyType.DOUBLE_CLICK -> Log.e("AudioRecordActivity", "Double click")
                    KeyType.AI_START -> Log.e("AudioRecordActivity", "AI start")
                    KeyType.LONG_PRESS -> Log.e("AudioRecordActivity", "Long press")
                    else -> Log.e("AudioRecordActivity", "Other key")
                }
            }
        }
    }

    private var recorder: AudioRecord? = null
    private var recordingThread: Thread? = null
    private var isRecordingActive = false

    private val _permissionGranted = MutableStateFlow(false)
    val permissionGranted = _permissionGranted.asStateFlow()

    companion object {
        private const val SAMPLE_RATE = 16000 // 16 kHz
        private const val CHANNEL_CONFIG = CONSTANT.AUDIO_CHANNEL // 0x6000FC
        private const val AUDIO_FORMAT = AudioFormat.ENCODING_PCM_16BIT // 16-bit
        private const val BUFFER_SIZE = 1024
    }

    @SuppressLint("UnspecifiedRegisterReceiverFlag")
    fun registerReceiver(activity: Activity) {
        activity.registerReceiver(keyReceiver, IntentFilter().apply {
            addAction(KeyType.CLICK.action)
            addAction(KeyType.BUTTON_DOWN.action)
            addAction(KeyType.BUTTON_UP.action)
            addAction(KeyType.DOUBLE_CLICK.action)
            addAction(KeyType.AI_START.action)
            addAction(KeyType.LONG_PRESS.action)
            priority = 100
        })
    }

    fun unregisterReceiver(activity: Activity) {
        keyReceiver?.let {
            activity.unregisterReceiver(it)
        }
    }

    fun permissionGranted(granted: Boolean) {
        _permissionGranted.value = granted
    }

    fun stopRecording() {
        isRecordingActive = false
        recordingThread?.join()
        recorder?.stop()
        recorder?.release()
        recorder = null
    }

    fun startAudioRecord() {
        if (recorder == null) {
            recorder = AudioRecord.Builder()
                .setAudioSource(MediaRecorder.AudioSource.MIC)
                .setAudioFormat(
                    AudioFormat.Builder()
                        .setSampleRate(SAMPLE_RATE)
                        .setChannelMask(CHANNEL_CONFIG)
                        .setEncoding(AUDIO_FORMAT)
                        .build()
                )
                .build()
        }
        recorder?.startRecording()
        isRecordingActive = true
        recordingThread = Thread { writeAudioDataToFile() }
        recordingThread?.start()
    }

    private fun writeAudioDataToFile() {
        val audioDir = File("/sdcard/Audio/")
        if (!audioDir.exists()) audioDir.mkdirs()

        val timeStamp = SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault()).format(Date())
        val fileName = "$timeStamp.pcm"
        val file = File(audioDir, fileName)

        Log.d("AudioRecordViewModel", "Saving audio to: ${file.absolutePath}")

        try {
            FileOutputStream(file).use { outputStream ->
                val buffer = ByteArray(BUFFER_SIZE)
                while (isRecordingActive) {
                    val read = recorder?.read(buffer, 0, BUFFER_SIZE) ?: 0
                    if (read > 0) outputStream.write(buffer, 0, read)
                }
            }
        } catch (e: Exception) {
            Log.e("AudioRecordViewModel", "Error writing audio data to file", e)
        }
    }
}
```

### Notes

- The output is **interleaved PCM** at 16 kHz, 16-bit, 8 channels — so each frame is `8 channels × 2 bytes = 16 bytes`. Writing `BUFFER_SIZE = 1024` bytes at a time corresponds to 64 frames per write (~4 ms of audio).
- The recording is written as raw `.pcm` (no WAV header) into `/sdcard/Audio/`. To play back, open it in a tool like Audacity with the same parameters (16 kHz, 16-bit signed, 8 channels, interleaved).
- The reference code only registers the 6 single-finger broadcast actions on `IntentFilter`. If you also need the two-finger or settings actions, mirror the full filter from [Button Broadcasts](./key-broadcasts.md).
- `MediaRecorder.AudioSource.MIC` is used; the channel mask is what unlocks the 8-channel layout. Other audio sources (e.g. `VOICE_RECOGNITION`) have not been verified to respect the same channel mask.

## Related docs

- [Bare-Metal Development Guide](./development-guide.md) — overview, dev environment, ADB enablement.
- [Button Broadcasts](./key-broadcasts.md) — `KeyType` enum and full action-string list used by the reference `KeyReceiver` above.

<!-- TODO: Source does not document the per-channel sample format (signed vs. unsigned, byte order). Confirm against a firmware-side decompiled trace or capture a sample and inspect. -->
