# RokidScreenRecord

System app for screen recording and live FLV streaming on Rokid AR glasses.

## Package Info

- **Package**: `com.rokid.os.master.screenstream`
- **Shared UID**: `android.uid.system` (runs as system process)
- **Target SDK**: 33 (Android 13)
- **Application class**: `ScreenStreamApplication`
- **Source**: `ScreenStreamApplication.java`, `ScreenRecordService.java`, `ScreenRecordReceiver.java`

## Permissions

| Permission | Purpose |
|---|---|
| `android.permission.ACCESS_WIFI_STATE` | Get device IP address for HTTP server binding |
| `android.permission.ACCESS_NETWORK_STATE` | Check network connectivity |
| `android.permission.RECORD_AUDIO` | Capture microphone audio alongside screen |
| `android.permission.SYSTEM_ALERT_WINDOW` | Show recording indicator overlay on glasses display |
| `android.permission.INTERNET` | Serve FLV stream over HTTP |
| `android.permission.MANAGE_MEDIA_PROJECTION` | Capture screen content via MediaProjection |
| `android.permission.FOREGROUND_SERVICE` | Run recording/streaming as foreground service |

## Components

### Receivers

**ScreenRecordReceiver** (exported)
Handles screen recording on/off via broadcast intents. Registered in manifest with intent filters for `SCREENRECORD_ON` and `SCREENRECORD_OFF`.

**BootCompleteReceiver** (exported)
Listens for `android.intent.action.BOOT_COMPLETED`. On boot, logs that it will set default stream status. The actual initialization happens in `ScreenStreamApplication.onCreate()`.

### Services

**ScreenRecordService**
Foreground service that manages the recording lifecycle. Registers with EventBus to receive `BusMessages`. When a `MESSAGE_ACTION_STREAMING_STOP` event arrives while the engine is in recording mode (status = 1), it stops the recording. Also listens for `ACTION_SHUTDOWN` to gracefully stop recording before device shutdown.

### Providers

**InitializationProvider** (`androidx.startup.InitializationProvider`)
Standard AndroidX startup provider for emoji and lifecycle initialization. Not relevant to screen recording functionality.

## Broadcast Intents API

### Screen Recording (to file)

#### Start Recording

```
Action: com.rokid.yodaos.action.SCREENRECORD_ON
```

**Extras:**

| Key | Type | Default | Description |
|---|---|---|---|
| `FileName` | String | null | Output filename for the recording |
| `MaxTime` | long | 600000 (10 min) | Maximum recording duration in milliseconds |

**Preconditions:**
- Engine status must be idle (0). If the engine is already running (recording or streaming), the intent is ignored.
- Free disk space must exceed 400 MB (0x19000000 bytes = 419,430,400). If insufficient, the recording does not start.

**adb example:**

```bash
# Start recording with default settings (max 10 min)
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_ON

# Start recording with custom filename and 5-minute max
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_ON \
  --es FileName "my_recording" \
  --el MaxTime 300000
```

#### Stop Recording

```
Action: com.rokid.yodaos.action.SCREENRECORD_OFF
```

**Extras:**

| Key | Type | Default | Description |
|---|---|---|---|
| `DeleteFile` | boolean | false | If true, deletes the recorded file after stopping |

**Preconditions:**
- Engine status must be 1 (recording mode). If the engine is in a different state, the intent is ignored.

**adb example:**

```bash
# Stop recording and keep the file
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_OFF

# Stop recording and delete the file
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_OFF \
  --ez DeleteFile true
```

### Screen Streaming (FLV over HTTP) -- Intent Constants

These intent action strings are defined in `Constraints.java` but are **not registered in the manifest** as intent filters. They are used internally or by other system components:

| Constant | Action String |
|---|---|
| `SCREEN_STREAM_ACTION_ON` | `com.rokid.yodaos.action.SCREENSTREAM_ON` |
| `SCREEN_STREAM_ACTION_OFF` | `com.rokid.yodaos.action.SCREENSTREAM_OFF` |
| `SCREEN_STREAM_ACTION_START` | `com.rokid.yodaos.action.SCREENSTREAM_START` |
| `SCREEN_STREAM_ACTION_STOP` | `com.rokid.yodaos.action.SCREENSTREAM_STOP` |
| `SCREEN_STREAM_ACTION_RESTART` | `com.rokid.yodaos.action.SCREENSTREAM_RESTART` |
| `SCREEN_STREAM_SERVICE_ACTION_ON` | `com.rokid.yodaos.action.SCREENSTREAM_SERVICE_ON` |
| `SCREEN_STREAM_SERVICE_ACTION_OFF` | `com.rokid.yodaos.action.SCREENSTREAM_SERVICE_OFF` |
| `LIVE_STREAM_ACTION_ON` | `com.rokid.yodaos.action.LIVE_STREAM_ON` |
| `LIVE_STREAM_ACTION_OFF` | `com.rokid.yodaos.action.LIVE_STREAM_OFF` |

### Settings / System Properties

The app reads and writes several system-level settings:

| Setting | Type | Purpose |
|---|---|---|
| `rokid_yodaos_screen_stream` | Settings.Global (int) | Stream status flag (0 = off) |
| `rokid_yodaos_screen_record` | Settings.Global (int) | Record status flag (0 = off) |
| `rokid_yodaos_screen_stream_switch` | Settings.Global (int) | Stream switch flag (0 = off) |
| `debug.rokid.screenrecord_state` | System property | Recording state ("0" = idle) |
| `persist.rokid.stream.state` | System property | Persistent stream state |
| `persist.rokid.stream.audio` | System property | Audio streaming enable flag |
| `persist.rokid.record.atw` | System property | Record from ATW (async timewarp) flag |
| `persist.rokid.mix.record` | System property | Mix record enable flag |
| `persist.rokid.mic.mix.record` | System property | Microphone mix record flag ("1" = enabled) |

All three Settings.Global values are reset to 0 on app startup in `ScreenStreamApplication.onCreate()`.

## FLV Live Streaming

### Architecture

The streaming pipeline works as follows:

1. **StreamEngine** captures the screen via `MediaProjection` / `VirtualDisplay` (or ATW surface for 3D content)
2. **VideoEncoder** encodes frames to H.264 (AVC) via Android `MediaCodec`
3. **AudioEncoder** encodes microphone audio to AAC (`audio/mp4a-latm`)
4. **FlvDecorator** wraps encoded H.264 + AAC data into FLV container format
5. **FlvDispatcher** (with `FlvStreamerThread`) takes FLV data from a `ConcurrentLinkedDeque` queue and writes it to all connected HTTP clients
6. **HttpServer** listens on port 8080 and serves the web viewer page and FLV stream

### HTTP Server

**Port**: 8080 (hardcoded in `Constraints.SERVER_PORT`)

**Bind address**: The device's WiFi IP address (obtained from `WifiManager.getConnectionInfo().getIpAddress()`)

**Socket timeout**: 10000 ms (10 seconds)

**Backlog**: 4 connections

### HTTP Routes

| Path | Response | Description |
|---|---|---|
| `/` | HTML page | Serves the FLV web viewer (`index.html` from assets). The placeholder `SCREEN_STREAM_ADDRESS` in the HTML is replaced with `http://<device-ip>:8080/screen_stream.flv` at runtime. |
| `/screen_stream.flv` | FLV stream | Raw FLV video stream. Client is added to the FlvDispatcher's client list. Response headers: `Content-Type: video/x-flv`, `Cache-Control: no-cache`, `Access-Control-Allow-Origin: *`. |
| `/favicon.ico` | PNG image | Serves the `favicon.png` from app assets. Response: `Content-Type: image/png`. Note: despite the `.ico` path, it returns PNG. |
| Any other path | 301 redirect | Redirects to `http://<device-ip>:8080/` |

### Streaming Auto-Start

Streaming is demand-driven. The flow is:

1. A browser connects to `http://<device-ip>:8080/`
2. The HTML page loads and `flv.js` requests `/screen_stream.flv`
3. `HttpServer` accepts the socket, `FlvDispatcher.addClient()` adds the client
4. `LocalServiceManager.addClient()` detects client count changed from 0 to 1
5. `OnClientSizeChangeListener` fires with `true`, posting `MESSAGE_ACTION_STREAMING_START` via EventBus
6. `ScreenRecordService.onMessageEvent()` receives the event and starts the stream engine

When the last client disconnects, `MESSAGE_ACTION_STREAMING_STOP` is posted and the engine stops.

### How to Access the Stream

1. Ensure the glasses are connected to WiFi
2. Find the glasses IP address (check WiFi settings or router DHCP leases)
3. Open a browser on any device on the same network
4. Navigate to `http://<glasses-ip>:8080/`
5. The web viewer loads and the stream starts automatically

Alternatively, access the raw FLV stream directly:

```bash
# Watch in VLC or mpv
mpv http://<glasses-ip>:8080/screen_stream.flv

# Record with ffmpeg
ffmpeg -i http://<glasses-ip>:8080/screen_stream.flv -c copy output.flv
```

## Web Viewer (index.html)

The app bundles a minimal HTML page in its assets that acts as the streaming viewer.

### Page Title

"Rokid screen cast" (displayed as Chinese text: "Rokid screen projection")

### How It Works

The page contains an embedded copy of **flv.js** (Bilibili's FLV player library) inlined in a `<script>` tag. On `window.onload`:

1. Reads the FLV stream URL from a hidden `<div id="J-flv-path">` element. The server replaces the placeholder text `SCREEN_STREAM_ADDRESS` with the actual URL `http://<device-ip>:8080/screen_stream.flv` before serving the page.
2. Creates an `flvjs.createPlayer()` with these settings:
   - `type: 'flv'`
   - `enableWorker: true` -- uses Web Worker for parsing
   - `enableStashBuffer: false` -- disables buffering for lower latency
   - `stashInitialSize: 128` -- minimal initial stash (128 bytes)
   - `fixAudioTimestampGap: true` -- fixes audio sync issues
3. Attaches the player to a `<video>` element, calls `load()` and `play()`.
4. The player starts **muted** (`flvPlayer.muted = true`).

### Latency Compensation

The viewer implements adaptive playback rate control via a `progress` event listener:

- Calculates `delta = buffered.end(0) - currentTime` (how far ahead the buffer is from current playback)
- If `delta > 5` seconds or `delta < 0`: **hard seek** to `buffered.end(0) - 1` (jump to near-live)
- If `delta > 2.0` seconds: set `playbackRate = 1.2` (speed up to catch up)
- If `delta <= 2.0` seconds: set `playbackRate = 1.0` (normal speed)

This keeps the viewer within ~2 seconds of the live feed.

### Assets

| File | Description |
|---|---|
| `index.html` | Web viewer page with embedded flv.js |
| `flv.js` | Bilibili's flv.js library (inlined in index.html, also present as separate file) |
| `favicon.png` | Favicon served at `/favicon.ico` |
| `awesomeface.png` | Smiley face image (purpose unclear, possibly unused or a test asset) |

## Encoding Configuration

### Video

| Parameter | Value | Notes |
|---|---|---|
| Codec | H.264 / AVC (`video/avc`) | Hardware-accelerated via MediaCodec |
| Width | 480 (0x1E0) | Landscape-oriented for glasses display |
| Height | 640 (0x280) | |
| Bitrate | 4,000,000 bps (~4 Mbps) | `0x3D0900` |
| Frame rate | 30 fps | `0x1E` |
| I-frame interval | 10 seconds | `0x0A` |
| Bitrate mode | CBR (constant bitrate, mode 2) | |
| Quality | 100 (0x64) | |
| Profile | High (0x08) | |
| Repeat frame delay | 100,000 us (100 ms) | `REPEAT_FRAME_DELAY_US = 0x186A0` |

For screen recording mode (`SCREEN_RECORD`), the color format is set to `0x7F420888` (likely `COLOR_FormatYUV420Flexible`), with color standard = 2, color range = 1, color transfer = 3.

### Audio

| Parameter | Value | Notes |
|---|---|---|
| Codec | AAC (`audio/mp4a-latm`) | |
| Sample rate | 16,000 Hz | `0x3E80` |
| Bit rate | 196,000 bps (~196 kbps) | `0x2FDA0` |
| Channel count | 1 (mono) | |
| Channel format | 16 (0x10) | `CHANNEL_IN_MONO` |
| Sample size | 4096 (0x1000) | Buffer size in samples |
| Bit width | 2 | 16-bit PCM |
| AAC profile | 2 | AAC-LC |
| Audio source | 1 | `MediaRecorder.AudioSource.MIC` |

Audio is always enabled for streaming (the `enableAudio()` method returns hardcoded `true`).

## Engine States

The `StreamEngine` operates as a state machine with the following states:

| State | Value | Description |
|---|---|---|
| `STREAM_ENGINE_IDLE` | 0 | Idle, no recording or streaming |
| `STREAM_ENGINE_FOR_RECORD` | 1 | Recording screen to file |
| `STREAM_ENGINE_FOR_STREAM` | 2 | FLV streaming over HTTP |
| `STREAM_ENGINE_FOR_LIVE_STREAM` | 3 | RTMP live streaming (external server) |

The `StreamEngineHandler` message codes for controlling the engine:

| Message | Code | Description |
|---|---|---|
| `STREAM_ENGINE_PREPARE` | 0x1001 | Initialize the engine with a stream type (1, 2, or 3). Defaults to type 2 (stream) if arg1 is 0. |
| `STREAM_ENGINE_START` | 0x1002 | Start encoding and streaming/recording |
| `STREAM_ENGINE_RESUME` | 0x1003 | Resume (logged but no-op in current implementation) |
| `STREAM_ENGINE_STOP` | 0x1004 | Stop encoding |
| `STREAM_ENGINE_PAUSE` | 0x1005 | Pause (logged but no-op in current implementation) |
| `STREAM_ENGINE_DESTROY` | 0x1006 | Destroy the engine and release all resources |
| `STREAM_ENGINE_ON_DRAW` | 0x1007 | Trigger a frame draw from the SurfaceTexture |

## Recording Details

### File Storage

Recordings are saved to the device's external storage Movies directory:
`<external-storage>/Android/data/com.rokid.os.master.screenstream/files/Movies/ScreenRecorder/`

### Recording Limits

- **Maximum duration**: 600,000 ms (10 minutes) by default, configurable via the `MaxTime` intent extra
- **Minimum free space**: 419,430,400 bytes (~400 MB) required to start recording
- **Delayed stop time**: 30,000 ms (30 seconds) -- `SCREEN_RECORD_DLELAY_STOP_TIME = 0x7530`

### Recording Format

Output is muxed via Android's `MediaMuxer` into MP4 container with:
- H.264 video at 480x640, 30fps, ~4 Mbps
- AAC audio at 16 kHz mono, ~196 kbps

### Recording Indicator

When recording, a `ScreenRecordDialog` overlay is displayed on the glasses' 3D display to indicate recording is active. The dialog includes a countdown timer showing remaining time.

## Display Modes

The app handles two display configurations:

| Mode | Width | Height | Constant Names |
|---|---|---|---|
| 2D | 1920 (0x780) | 1200 (0x4B0) | `TWOD_WIDTH`, `TWOD_HEIGHT` |
| 3D | 3840 (0xF00) | 1200 (0x4B0) | `THREED_WIDTH`, `THREED_HEIGHT` |

The recording resolution is always downscaled to 480x640 regardless of display mode.

## Product Model Detection

The app checks `ro.rokid.product.model` system property:
- `RG-station2` -- Station 2 model
- `RG-stationPro` -- Station Pro model

This affects certain behaviors related to mixed recording and display handling.

## Internal Architecture

### Key Classes

| Class | Role |
|---|---|
| `ScreenStreamApplication` | Application singleton. Creates `LocalServiceManager`, resets all status flags on startup. |
| `LocalServiceManager` | Central manager. Holds client queue, FLV data queue, camera queue, HTTP page template, recording state, and device info (WiFi IP, display manager, power manager). |
| `ScreenRecordService` | Foreground service. Subscribes to EventBus for start/stop events. Manages the recording lifecycle and UI indicator. |
| `ScreenRecordReceiver` | Broadcast receiver for `SCREENRECORD_ON`/`SCREENRECORD_OFF` intents. |
| `StreamEngine` | Singleton managing the `StreamEngineHandler` on a background HandlerThread. |
| `StreamEngineHandler` | Handler processing engine state messages. Initializes video/audio encoders, screen recorder, EGL environment, and output decorators. |
| `HttpServer` | TCP server on port 8080. Routes requests to the HTML page, FLV stream, or favicon. |
| `FlvDispatcher` | Manages connected FLV clients. Runs a `FlvStreamerThread` that polls the FLV data queue and writes to all clients. |
| `Client` | Represents a connected HTTP client socket. Sends FLV header and data frames, handles disconnection. |
| `FlvDecorator` | Wraps raw H.264/AAC encoded data into FLV tags. |
| `VideoEncoder` / `AudioEncoder` | MediaCodec wrappers for H.264 and AAC encoding. |
| `VideoOrder` / `AudioOrder` | Builder-pattern configuration objects for video/audio encoding parameters. |
| `RecordManager` | AIDL binder service providing `getEncodeSurface()` and `getSingleSurface()` for other system components to render into the recording pipeline. |
| `PropertyUtils` | Reads/writes Android system properties via reflection (`android.os.SystemProperties`). |

### EventBus Messages

| Message | Direction | Effect |
|---|---|---|
| `MESSAGE_ACTION_STREAMING_START` | `LocalServiceManager` -> `ScreenRecordService` | First HTTP client connected; start streaming engine |
| `MESSAGE_ACTION_STREAMING_STOP` | `LocalServiceManager` -> `ScreenRecordService` | Last HTTP client disconnected; stop streaming engine if in recording mode |

### IPC (AIDL)

`RecordManager` extends `IRecordManager.Stub`, providing a binder interface for other system apps:
- `getEncodeSurface(List<Surface>)` -- adds the encoder's input Surface to the provided list
- `getSingleSurface()` -- returns the encoder's input Surface directly

This allows other system components (such as the compositor or ATW renderer) to render directly into the recording pipeline without going through screen capture.

## Quick Reference

### Start FLV stream viewing from a computer

```bash
# 1. Get glasses IP (must be on WiFi)
adb shell ip addr show wlan0

# 2. Open in browser
# http://<ip>:8080/

# 3. Or use a media player
mpv http://<ip>:8080/screen_stream.flv
vlc http://<ip>:8080/screen_stream.flv
```

### Start/stop screen recording via adb

```bash
# Start
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_ON

# Stop
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_OFF

# Start with custom filename and 2-minute limit
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_ON \
  --es FileName "test_capture" \
  --el MaxTime 120000

# Stop and discard the file
adb shell am broadcast -a com.rokid.yodaos.action.SCREENRECORD_OFF \
  --ez DeleteFile true
```

### Check recording state

```bash
adb shell getprop debug.rokid.screenrecord_state
# "0" = idle, non-zero = recording
```

### Pull recorded files

```bash
adb pull /sdcard/Android/data/com.rokid.os.master.screenstream/files/Movies/ScreenRecorder/ .
```
