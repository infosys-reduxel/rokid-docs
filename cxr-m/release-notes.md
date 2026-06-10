# CXR-M SDK Release Notes

> Source: https://ar.rokid.com/sdk (CXR-M section, Chinese, fetched 2026-05-29)

The CXR-M SDK is a developer toolkit for mobile apps that pair with Rokid Glasses. It supports stable connectivity, data communication, real-time audio and video, and custom scenes, and is meant to be used together with the on-device CXR-S SDK. **The CXR-M SDK is not publicly distributed on the Rokid developer site.** To request the SDK, documentation, or technical support, contact business partnerships at `Glasses.BD@rokid.com`.

Latest **publicly documented** release: **v1.1.0** (2026-04-01).

## Documentation gap (2026-05-29)

Maven publishes `com.rokid.cxr:client-m:1.2.2` as the current `<release>` and `<latest>` tag (lastUpdated 2026-06-09). Intermediate releases 1.1.1, 1.2.1, and 1.2.2 are all real, downloadable artifacts. Neither the dev portal at `https://ar.rokid.com/sdk` nor the canonical doc site at `https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/...` describes any release beyond v1.1.0.

The v1.2.2, v1.2.1, and v1.1.1 sections below are **reverse-inferred from binary diffs** of the AARs. They describe the *observable* public-API delta — names of new classes / methods / callbacks, breaking signature changes, JNI library size deltas — but they do not capture intent, deprecation notes, or behavioural changes that don't show up in the class file. Treat them as a best-effort developer aid until Rokid publishes the authoritative changelog.

Method to reproduce (run any time):

```sh
# Download
for V in 1.1.0 1.1.1 1.2.1 1.2.2; do
  curl -O "https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-m/$V/client-m-$V.aar"
done
# Unpack AAR -> classes.jar -> .class files
# Diff class lists and javap signatures across the versions.
```

<!-- TODO: replace the inferred sections below with Rokid's official changelog when it is published. -->

## Changelog

### v1.2.2 — uploaded 2026-06-09 (inferred from binary diff)

> **Provisional — not an official Rokid changelog.** Reconstructed from the public-API diff between `client-m:1.2.1` and `client-m:1.2.2` AARs (downloaded 2026-06-09 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-m/`). AAR size: 1,225,276 bytes (+0.5 % vs 1.2.1).

**Theme: structured audio-record parameters and multi-client disconnect.**

**New class:**

- `com.rokid.cxr.CXRSocketProtocol$AudioRecordParam` — parameter object replacing the raw `int` denoise-mode flag in `openAudioRecord`.

  | Field | Type | Description |
  |-------|------|-------------|
  | `denoiseMode` | `int` | Denoise algorithm mode (was the bare `int` in 1.2.1) |
  | `rokidDtlnAEC` | `boolean` | Enable Rokid DTLN acoustic echo cancellation |
  | `rokidBF` | `boolean` | Enable Rokid beamforming |

  Constructors: `AudioRecordParam()` (defaults) and `AudioRecordParam(int, boolean, boolean)`.

**API changes (breaking for callers of `openAudioRecord`):**

- `CXRSocketProtocol.openAudioRecord(int, int, String, int)` → `openAudioRecord(int, int, String, AudioRecordParam)`. The bare `int` denoise-mode argument is replaced by the new `AudioRecordParam` object.
- `CxrController.openAudioRecord(int, int, String, int)` → `openAudioRecord(int, int, String, AudioRecordParam)` (same substitution at the controller layer).
- `CxrApi.openAudioRecord(int, int, String, int)` → `openAudioRecord(int, int, String, AudioRecordParam)` (same at the public API layer).

> **Migrating from 1.2.1:** replace any `openAudioRecord(sampleRate, channelCount, tag, denoiseMode)` call with `openAudioRecord(sampleRate, channelCount, tag, new AudioRecordParam(denoiseMode, false, false))` to preserve previous behaviour. Set `rokidDtlnAEC` or `rokidBF` to `true` to opt into the new processing modes.

**New methods:**

- `CXRSocketProtocol.removeClient(String mac, int type)` — disconnect a specific connected client by MAC and connection type.
- `CxrController.removeClient(String mac, int type)` — same at the controller layer.
- `CxrApi.removeBluetoothClient(String mac)` — convenience wrapper on the public API surface.

**Obfuscated package rename (no public-API impact):**

The internal obfuscated packages were renamed from `a/`, `b/`, `c/` to `rapple/`, `rbanana/`, `rcherry/`. No public class names changed. Any tooling that relied on the old obfuscated class paths will need to rebuild.

**JNI library deltas (arm64-v8a):**

| Library | v1.2.1 | v1.2.2 | Delta |
|---------|--------|--------|-------|
| `libcxr-bridge-jni.so` | 112,672 B | 116,488 B | +3.4 % |
| `libcxr-sock-proto-jni.so` | 636,200 B | 641,320 B | +0.8 % |
| `libflora-cli.so` | 157,224 B | 157,280 B | +0.04 % |
| `libcaps.so` | 106,640 B | 106,640 B | unchanged |
| `libmutils.so` | 432,360 B | 432,360 B | unchanged |

**Unchanged vs v1.2.1:** `AndroidManifest.xml`, `R.txt`, `proguard.txt`, POM dependency set (Retrofit 2.9.0, OkHttp 4.9.3, Kotlin 2.1.0, Gson 2.10.1).

### v1.2.1 — uploaded 2026-03-26 (inferred from binary diff)

> **Provisional — not an official Rokid changelog.** Reconstructed from the public-API diff between `client-m:1.1.0` and `client-m:1.2.1` AARs (downloaded 2026-05-29 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-m/`).

**Theme: multi-client Bluetooth support.**

New classes:

- `com.rokid.cxr.client.extend.callbacks.BluetoothClientsInfoCallback` — interface; `void onBtClientsInfo(List<BtClientInfo>)`.
- `com.rokid.cxr.client.utils.ValueUtil.BtClientInfo` — data class with `String mac`, `String customInfo`, `ValueUtil.CxrStatus bluetoothStatus`.
- `com.rokid.cxr.CXRSocketProtocol.ClientInfo` — data class with `String mac`, `String customInfo`, `int status`.
- `com.rokid.cxr.CXRSocketProtocol.Parameter` — data class with `boolean async`, `String customInfo`.

New methods on `CxrController`:

- `void activeBluetoothConnect(String)` — activate a Bluetooth connection by client identifier.
- `void fetchClientList()` — request the connected-client list asynchronously.
- `List<BtClientInfo> getClientList(int)` — synchronous query.

New callbacks on `CxrController`:

- `void onStatusUpdateWithExtra(CxrStatus, CxrBluetoothErrorCode, String, String)` — richer status update than the existing `onStatusUpdate(...)`.
- `void onBtClientsInfo(List<BtClientInfo>)` — receives the connected-client list.

New native methods in `libcxr-sock-proto-jni.so`:

- `nativeActive(long, String)`
- `nativeGetClientList(long, int) → CXRSocketProtocol.ClientInfo[]`
- `nativeFetchClientList(long)`

The `libcxr-sock-proto-jni.so` library grew from 616,824 to 636,200 bytes (+3.1 %), consistent with the new native calls. `libcaps.so`, `libflora-cli.so`, `libmutils.so`, and `libcxr-bridge-jni.so` were not resized; `libcxr-bridge-jni.so` saw a 112-byte rebuild artifact.

**Breaking changes vs v1.1.0** (consumer code will not compile against 1.2.1 without edits):

- `CxrController.connectBluetooth(Context, String, String)` → `connectBluetooth(Context, String, String, String)`. The added 4th `String` is most likely a per-client `customInfo` identifier, matching the multi-client theme.
- `CXRSocketProtocol.run(BluetoothSocket, UUID, Callback, boolean, boolean)` → `run(BluetoothSocket, Callback, Parameter)`. The `UUID` and the two `boolean` flags are now bundled into the new `Parameter` class.
- `CXRSocketProtocol.setAuthPtr(long)` removed from the public surface — authentication state appears to be routed through `Parameter` or another internal path now.
- Private native `CXRSocketProtocol.nativeClose(long)` removed; `nativeHandleReadPacket` lost one `long` parameter.

**Unchanged across v1.1.0 → v1.2.1**: `AndroidManifest.xml`, `R.txt`, `proguard.txt`, POM dependency set (Retrofit 2.9.0 + converter-gson, OkHttp 4.9.3 + logging-interceptor, okio 2.8.0, Kotlin stdlib 2.1.0, Gson 2.10.1).

### v1.1.1 — uploaded 2026-04-10 (inferred from binary diff)

> **Provisional — not an official Rokid changelog.** Reconstructed from the diff between `client-m:1.1.0` and `client-m:1.1.1` AARs. This artifact was uploaded *after* v1.2.1, indicating Rokid maintained the 1.1.x line in parallel with the 1.2.x line — likely a backport patch for consumers that had not migrated yet.

- New: `WifiController.setWifiCallback(WifiController.Callback)` — register a Wi-Fi callback (a previously read-only state surface gained a subscription path).
- New: a 3-argument overload `startUploadApk(String, String, ApkStatusCallback)` for the APK upload flow.
- Internal obfuscated package layout was renamed from `a/`, `b/`, `c/` to `rapple/`, `rbanana/`, `rcherry/`. No public-API impact, but the ProGuard mapping is fresh — any tooling that relied on the old obfuscated paths will need to rebuild.
- `AndroidManifest.xml`, `R.txt`, `proguard.txt`, and all JNI libraries are byte-identical to v1.1.0 — no native code changes.

### v1.1.0 — 2026-04-01

1. Improved the voice-stream listener API.
2. Photo capture now returns the raw image straight from the on-glasses Camera.

### v1.0.9 — 2026-03-01

1. Real-time audio streaming.
2. Improved voice control.
3. Added an alternate media-file transfer scheme — the host app can now bring up the Wi-Fi P2P connection itself.
4. Added documentation for the CXR-S-side data API.

### v1.0.4 — 2025-12-25

1. Added SN-based access control.
2. Added a standalone photo-capture API that returns the captured image data.
3. Fixed an OOM issue when syncing large files over Wi-Fi P2P.
4. Fixed the issue where AI scenes exited after 6 seconds.
5. Added an API to subscribe to glasses screen-on / screen-off state updates.
6. Added an API to put the glasses display to sleep on demand.
7. Added an API to switch the microphone pickup field.
8. Added an API to set the on-device TTS voice.
9. Added an API to set the on-device TTS speech rate.

### v1.0.1 — 2025-08-25

1. Connect to the glasses.
2. Read and set glasses screen brightness.
3. Custom AI assistant scenes.
4. Custom translation scenes.
5. Custom teleprompter scenes.
6. Capture audio from the on-device microphones.
7. Configure and capture photos.
8. Configure video recording parameters.
9. Launch on-device scenes (AI assistant, translation, etc.) from the mobile app.
