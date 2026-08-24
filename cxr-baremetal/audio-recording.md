# Raw Audio on Glasses (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=641107e820764397870d6da8a103da19> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0** (supersedes the v0.0.1 revision fetched 2026-05-29 from the now-migrated `57e35cd3ae294d16b1b8fc8dcbb1b7c7` workspace; implementation notes and the reference `AudioRecordViewModel` sample below are retained from that earlier decompiled-sample capture — the channel mask, sample rate, and channel layout are unchanged between the two revisions).

This page covers 8-channel microphone capture on Rokid Glasses from a bare-metal Android app. For the broader context, see the [Introduction to Bare-Metal Development on Rokid Glasses](./development-guide.md).

## Overview

The glasses support **8-channel** PCM capture via standard `android.media.AudioRecord`, using `AudioFormat.Builder.setChannelMask(...)` for the device channel mask.

| Parameter | Recommended value |
| --- | --- |
| ChannelMask | `0x6000FC` |
| Sample rate | 16000 Hz |
| Encoding | `ENCODING_PCM_16BIT` |
| Audio source | `MediaRecorder.AudioSource.MIC` |

## Channel layout

| Channel | Content |
| --- | --- |
| 0 / 1 | Processed algorithm audio (output of the on-device AEC / beamforming pipeline) |
| 2–5 | Four raw microphone channels (per-capsule, for custom DSP / beamforming) |
| 6 / 7 | Hardware echo reference (playback reference signal used by the AEC) |

## Prerequisites

- Runtime `RECORD_AUDIO` permission.
- Optional: listen for click broadcasts from the [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) chapter to toggle recording.

## Code sketch

```kotlin
val channelMask = 0x6000FC
val recorder = AudioRecord.Builder()
    .setAudioSource(MediaRecorder.AudioSource.MIC)
    .setAudioFormat(
        AudioFormat.Builder()
            .setSampleRate(16_000)
            .setChannelMask(channelMask)
            .setEncoding(AudioFormat.ENCODING_PCM_16BIT)
            .build()
    )
    .build()
recorder.startRecording()
// Background thread read(buffer) and write PCM file
```

## Practices

- Call `stop()` and `release()` when leaving the screen to avoid leaks.
- Mind storage paths and permissions on Android 10+ scoped storage.
- The output is **interleaved PCM** at 16 kHz, 16-bit, 8 channels — each frame is `8 channels × 2 bytes = 16 bytes`. To play back a raw `.pcm` capture (no WAV header), open it in a tool such as Audacity with the same parameters (16 kHz, 16-bit signed, 8 channels, interleaved). _(Carried over from the v0.0.1-era capture; not contradicted by the current doc.)_

## Sample

`GlassesBareDevSample` → **Raw audio** screen: click broadcast toggles recording; status area shows the PCM file path.

See also the [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) chapter.

## Legacy reference implementation (v0.0.1-era)

The following `AudioRecordViewModel` was captured from the v0.0.1-era decompiled sample (`com.rokid.cxrssdksamples`, superseded by `GlassesBareDevSample` in v1.0.0). It demonstrates the same `ChannelMask = 0x6000FC` / 16 kHz / 16-bit PCM configuration confirmed by the current doc above, plus a full read/write loop; kept for reference since the current chapter only gives a code sketch.

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
            priority = 100
        })
    }

    fun unregisterReceiver(activity: Activity) {
        keyReceiver.let {
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

## Related docs

- [Introduction to Bare-Metal Development on Rokid Glasses](./development-guide.md) — overview, dev environment, ADB enablement.
- [Keys, Wear Detection, and Fold Events](./key-broadcasts.md) — action-string list used to start/stop recording on a click.

<!-- TODO: Neither the current v1.0.0 doc nor the v0.0.1-era capture documents the per-channel sample format (signed vs. unsigned, byte order) beyond "16-bit PCM". Confirm against a firmware-side decompiled trace or capture a sample and inspect if exact byte order matters for your use case. -->
