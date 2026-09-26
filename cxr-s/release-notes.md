# CXR-S SDK Release Notes

> **Provisional — not an official Rokid changelog.** Unlike CXR-L and CXR-M, `com.rokid.cxr:cxr-service-bridge` has **no changelog surface at all** on `developerdoc.rokid.com/sdk` — it is not one of the three cards on the SDK picker (CXR-L, CXR-M, 眼镜端裸机开发/bare-metal). This entire document is reconstructed from a direct binary diff of the `cxr-service-bridge` `.aar`/`.pom`/Gradle-module files for versions `1.0` through `1.4`, downloaded fresh from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/cxr-service-bridge/` on 2026-09-26. `maven-metadata.xml` at that path reports `<release>1.4</release>`, `<versions>` listing `1.0-SNAPSHOT, 1.0, 1.1, 1.2, 1.3, 1.4`, and `<lastUpdated>20260922073949</lastUpdated>` (2026-09-22). This repo's own docs (`cxr-s/sdk-import.md`) previously pinned `1.0-20260522.063600-105`; the plain `1.0` release coordinate resolves to a **byte-identical** artifact (same 1,076,548-byte AAR, browse-listing `Last Modified: Thu Dec 25 12:54:20 Z 2025`), so `1.0` is used as the baseline below rather than the timestamped snapshot build. Tooling used: `curl` (direct download, bypassing the agent's normal fetch tools), `unzip`, `diff`, `md5sum`, and `javap -p`/`javap -c -constants` (bytecode-level disassembly of the AAR's `classes.jar`, which is a plain JVM jar, not a `.dex`) — no `jadx`/`apktool`/native disassembler was used, so native-library **method bodies** were not reverse engineered, only their JNI-exported entry points (inferred from the Java `native` method declarations they back) and file sizes/hashes.
>
> `cxr-s/decompiled/` in this repo contains a small hand-picked decompile of `com.rokid.cxr.{Caps,CXRServiceBridge,CXRSocketProtocol,ReplyImpl,RLog,BuildConfig}`, but every file there is currently an un-fetched Git LFS pointer stub in this environment (`git lfs` is not installed here), so it could not be cross-checked. Note for a future pass: that reference set already includes `RLog.java`, a class that — per the diff below — was only **added** in v1.1, meaning that decompiled snapshot was taken from v1.1 or later, not from the `1.0-20260522.063600-105` build the surrounding prose in `sdk-import.md` cited. This inconsistency was not resolved here (LFS content unavailable) and is flagged for whoever next has LFS access.

## What this artifact is

`cxr-service-bridge` is the on-device (CXR-S) half of the shared Caps/wire-protocol transport described in this repo's `CLAUDE.md`: it ships the `Caps` binary serialization format, the native `CXRSocketProtocol` BLE/Bluetooth-socket framing layer, and `CXRServiceBridge`, the message-subscribe/send API that `cxr-s/message-sending.md`, `cxr-s/message-subscription.md`, and `cxr-s/data-structure.md` document. It has **zero declared Maven dependencies** in every version 1.0–1.4 — both the `.pom` (which only carries the `<packaging>aar</packaging>` marker per version) and the richer Gradle `.module` metadata list no `dependencies` block at all; the SDK is fully self-contained (its own native libraries plus a small pure-Java surface).

## Maven publication timeline (browse-listing `Last Modified`, all times UTC)

| Version | AAR size | Last Modified (Maven browse listing) |
|---------|----------|----------------------------------------|
| 1.0 | 1,076,548 bytes | Thu Dec 25 12:54:20 2025 |
| 1.1 | 1,136,671 bytes | Sat Sep 19 03:04:43 2026 |
| 1.2 | 1,136,682 bytes | Sat Sep 19 03:04:47 2026 |
| 1.3 | 1,137,916 bytes | Fri Sep 18 11:07:20 2026 |
| 1.4 | 1,138,221 bytes | Wed Sep 23 03:52:35 2026 |

Note the out-of-order timestamps: 1.3's artifact carries an *earlier* `Last Modified` (Sep 18) than 1.1/1.2 (Sep 19, four seconds apart from each other), and 1.4 was published four days later (Sep 23). This reads as several small releases batched through a build/publish pipeline in quick succession rather than a steady one-a-week cadence; no explanation for the ordering is available without an official changelog, so it is reported here as observed fact only, not interpreted further.

## v1.4 — Maven `Last Modified` 2026-09-23 (provisional binary-diff reconstruction — no official changelog)

**Theme: extends the raw on-device audio-record API added in v1.1 with a third parameter (buffer/frame-size configuration) and a voice-activity-detection helper. No changes to `Caps`, `sendMessage`, or `subscribe`.**

Real (non-cosmetic) findings, confirmed via `javap -p` diff of v1.3 vs v1.4 `classes.jar` (27 classes in both):

- **`CXRServiceBridge.openAudioRecord` gains a third `int` parameter**: `openAudioRecord(String, int, int, AudioRecordCallback)` (was `openAudioRecord(String, int, AudioRecordCallback)` in v1.1–v1.3). The same third-`int` parameter is added in lockstep to `AudioRecordHelper.openAudioRecord`, and to the underlying AIDL contract `com.rokid.cxrservice.ICXRService.openAudioRecord(String, int, int): ParcelFileDescriptor` (interface, `Stub`, `Stub.Proxy`, and `Default` all updated identically) — i.e. this is a real, coordinated protocol change to the on-device audio-record IPC call, not just an app-level convenience overload. The meaning of the new parameter is not documented anywhere in the bundled artifacts (no javadoc/param names survive in bytecode); do not guess its semantics.
- **`CXRServiceBridge.AudioRecordCallback.onData` gains a third `int` parameter**: `onData(byte[], int, int)` (was `onData(byte[], int)` since the v1.2 type change below). This is a **breaking change for implementors** of `AudioRecordCallback` — any existing implementation must add the third parameter.
- **New private native method `AudioRecordHelper.getVad(long)`** — returns `int`. Reads as a voice-activity-detection query against the native audio-record handle; not otherwise documented.
- **Native libraries changed on both ABIs**: `libcxr-bridge-jni.so` (arm64-v8a: 119,576 → 119,744 bytes) and `libcxr-sock-proto-jni.so` (arm64-v8a: 662,488 → 662,552 bytes) both hash-differ from v1.3. `libcaps.so`, `libflora-cli.so`, and `libmutils.so` are byte-identical to v1.1 (unchanged since the v1.0→v1.1 jump — see below).

**Confirmed unchanged from v1.0 through v1.4**: `Caps` (all read/write methods, `serialize`/`parse`/`fromBytes`), `CXRServiceBridge.sendMessage` (both overloads), `CXRServiceBridge.subscribe` (both `MsgCallback` and `MsgReplyCallback` overloads), `MsgCallback.onReceive`, `MsgReplyCallback.onReceive`, `Reply.end` — i.e. every API this repo's `cxr-s/data-structure.md`, `cxr-s/message-sending.md`, and `cxr-s/message-subscription.md` document is byte-for-byte identical in signature across all five versions. No update to those three files is needed for this version step.

## v1.3 — Maven `Last Modified` 2026-09-18 (provisional binary-diff reconstruction — no official changelog)

**Theme: adds a way to cancel an in-flight sent message. No changes to `Caps`, raw audio-record signatures, or `subscribe`.**

- **New public native method on `CXRServiceBridge`: `void cancelMessage(int)`.** Added between `sendMessage(String, Caps, byte[], int, int)` and `disconnectCXRDevice()` in the class's member order. No overload of `sendMessage` returns an `int` handle/id that would obviously feed this call (the existing `sendMessage` overloads return `0`/`-1`/`-3` result codes, not request ids, per `cxr-s/message-sending.md`), so the exact id space `cancelMessage` expects is not established from bytecode alone — flagged rather than guessed. `cxr-s/message-sending.md` is not updated for this, since the existing two `sendMessage` overloads it documents are unchanged; this is a purely additive method with an undocumented calling convention.
- **Native libraries changed on both ABIs**: `libcxr-bridge-jni.so` (arm64-v8a: 118,872 → 119,576 bytes) and `libcxr-sock-proto-jni.so` (arm64-v8a: 659,768 → 662,488 bytes) both hash-differ from v1.2 — consistent with `cancelMessage`'s native counterpart living in `libcxr-bridge-jni.so`. `libcaps.so`, `libflora-cli.so`, `libmutils.so` unchanged.
- Class inventory (27 classes) is otherwise identical to v1.2 — confirmed via `javap -p` diff.

## v1.2 — Maven `Last Modified` 2026-09-19 (provisional binary-diff reconstruction — no official changelog)

**Theme: a real, breaking type change to the raw audio-record data callback — `short[]` becomes `byte[]`. Everything else, including `Caps`/`sendMessage`/`subscribe`, is untouched.**

- **`AudioRecordHelper.audioData` field, `AudioRecordHelper.readAudioData(long, ...)`, and `CXRServiceBridge.AudioRecordCallback.onData(...)`'s first parameter all change from `short[]` to `byte[]`.** This is a source- and binary-incompatible change for any integrator using the v1.1 raw audio-record API (`openAudioRecord`/`AudioRecordCallback`) — code written against v1.1's `short[]`-typed callback will not compile against v1.2+. No `cxr-s/*.md` file in this repo documents the raw audio-record API (it is not part of `data-structure.md`, `message-sending.md`, or `message-subscription.md`'s scope), so no existing doc needs correcting, but this is noted here in case a future doc is written covering `AudioRecordHelper`/`CXRServiceBridge.AudioRecordCallback`.
- **Native library `libcxr-bridge-jni.so` changed on both ABIs** (arm64-v8a: 111,632 → 118,872 bytes — the biggest single-version jump in this JNI library's size across the whole 1.0–1.4 range), consistent with the audio-buffer type change requiring new native marshalling code.
- **`libcxr-sock-proto-jni.so` is byte-identical to v1.1** on both ABIs (arm64-v8a md5 `432accea7bdb0ffe4927ff98017be1a9` in both) — the socket/framing native layer did not change in this release, only the audio-record path did.
- `libcaps.so`, `libflora-cli.so`, `libmutils.so` unchanged from v1.1.
- Class inventory (27 classes) otherwise identical to v1.1 — confirmed via `javap -p` diff; every method signature besides the three `short[]`→`byte[]` sites above is unchanged.

## v1.1 — Maven `Last Modified` 2026-09-19 (provisional binary-diff reconstruction — no official changelog)

> **This is by far the largest jump in the artifact's history.** Class count grew from 16 to 27 (`classes.jar` 15,276 → 28,369 bytes), the AAR grew from 1,076,548 to 1,136,671 bytes, and every native library except `libcaps.so`'s companion set gained real new code. Diffed against v1.0 (the version this repo's `cxr-s/*.md` docs were written against, and the coordinate `cxr-s/sdk-import.md` cited before this update).

### Breaking change: `CXRServiceBridge` constructor now requires a `Context`

**`public CXRServiceBridge()` (v1.0) is replaced by `public CXRServiceBridge(android.content.Context)` (v1.1+).** The no-argument constructor is gone entirely — this is a source-breaking change for every existing integration, including the sample code already committed in this repo:

- `cxr-s/message-sending.md`, `cxr-s/message-subscription.md`, and `cxr-s/manage-device-connection.md` all instantiate `CXRServiceBridge()` with no arguments. **These three files are updated by this changelog pass** to show the `Context`-taking constructor, since this is a concretely verified, unambiguous signature change — not a guess about behavior.
- The `Context` parameter is almost certainly needed for the new `AudioRecordHelper` (see below), which is an `android.content.ServiceConnection` that binds to a system service and therefore needs a `Context` — but this repo cannot confirm that specific causal link from bytecode alone; it is offered as the most likely explanation, not a documented fact.

### New: on-device raw audio-record subsystem

v1.1 adds an entire new subsystem for capturing raw audio via IPC to a system service:

- **New class `com.rokid.cxr.AudioRecordHelper`** (`implements android.content.ServiceConnection`) — manages binding to a new AIDL service and exposes `start()`, `openAudioRecord(String, int, AudioRecordCallback)`, `closeAudioRecord(String)`, `readAudioData(long, short[])` (native), `getAudioChannels(long)` (native).
- **New AIDL interface `com.rokid.cxrservice.ICXRService`** (plus `ICXRService.Stub`, `ICXRService.Stub.Proxy`, `ICXRService.Default`) — `openAudioRecord(String, int): ParcelFileDescriptor` and `closeAudioRecord(String)`, both `throws RemoteException`. This is a new system-service contract for `com.rokid.cxrservice` (the glasses-side CXRService bridge process named in this repo's `CLAUDE.md` and documented in `yodaos/docs/apps/cxr-service.md`) — `AudioRecordHelper` binds to it and hands back a `ParcelFileDescriptor` for raw audio.
- **`CXRServiceBridge` gains `openAudioRecord(String, int, AudioRecordCallback): boolean` and `closeAudioRecord(String)`**, delegating to the new `AudioRecordHelper`.
- **New interface `CXRServiceBridge.AudioRecordCallback`**: `onStart(int)`, `onData(short[], int)`, `onStop()`.
- **New class `CXRServiceBridge.AudioRecordParam`**: `denoiseMode: int`, `rokidDtlnAEC: boolean`, `rokidBF: boolean` (denoise/AEC/beamforming toggles for the audio path — field names only, no further semantics recoverable from bytecode).
- **New class `com.rokid.cxr.RLog`**: thin native-backed logging shim (`v`/`d`/`i`/`w`/`e`, backed by a private native `nativeWrite(int, String, String)`).

None of this new subsystem is currently documented anywhere in `cxr-s/*.md` — it is out of scope for this changelog pass to write new API docs for it (that would need to be its own doc, e.g. a future `cxr-s/audio-recording.md`, and is flagged here as a documentation gap rather than actioned).

### Breaking change: `StatusListener` gains `onAudioNoise(float)`

**`CXRServiceBridge.StatusListener` gains a fifth abstract method, `onAudioNoise(float)`**, alongside the four already documented in `cxr-s/manage-device-connection.md` (`onConnected`, `onDisconnected`, `onConnecting`, `onARTCStatus`, `onRokidAccountChanged` — note `onConnecting` and `onRokidAccountChanged` were already present in v1.0's decompile but are not shown in that doc's simplified example). Any class implementing `StatusListener` directly (rather than via an interface with default methods — Kotlin `object : StatusListener { ... }` requires every abstract member) must now also implement `onAudioNoise(float)`. **`cxr-s/manage-device-connection.md` is updated by this changelog pass** with a note about this new required override.

### `CXRSocketProtocol` (native BLE/Bluetooth-socket framing layer) — substantially expanded

This class is the same "native `CXRSocketProtocol` framing layer" referenced under CXR-M in this repo's `CLAUDE.md` — `cxr-service-bridge` ships the on-device (CXR-S) build of it. v1.1's version gains, relative to v1.0:

| Change | Detail |
|--------|--------|
| `run(...)` signature changed | v1.0: `run(BluetoothSocket, UUID, Callback, boolean, boolean)`. v1.1: `run(BluetoothSocket, Callback, Parameter)` — the loose booleans/UUID are replaced by a new `CXRSocketProtocol.Parameter` class (`async: boolean`, `customInfo: String`). |
| `openAudioRecord(...)` signature changed | v1.0: `openAudioRecord(int, String, Caps)`. v1.1: `openAudioRecord(int, int, String, AudioRecordParam)` — now takes the new `CXRSocketProtocol.AudioRecordParam` (`denoiseMode`, `rokidDtlnAEC`, `rokidBF` — same shape as `CXRServiceBridge.AudioRecordParam`, a separate class in a separate outer class). |
| `Callback` interface grew | v1.0 had 7 methods (`onResponse`, `onNotify`, `onReceived`, `onStartAudioStream`, `onAudioStream`, `onARTCFrame`, `onDisconnect`). v1.1 has 11: the same 7 (several with changed parameter lists — e.g. `onAudioStream` gains a trailing `long`, `onARTCFrame` gains a trailing `long`) plus 4 new ones: `onAudioStreamFinish(int)`, `onActiveStatus(int, String, String)`, `onClientList(ClientInfo[])`, `onRemoveClientResult(int, String)`. **Breaking change for implementors of `CXRSocketProtocol.Callback`.** |
| New class `CXRSocketProtocol.ClientInfo` | `mac: String`, `customInfo: String`, `status: int`. |
| New public methods | `sendMessageLP(String, Caps)`, `clearMessageLP()`, `startPlayAudio(int, int, float, int, int, int)`, `startAudioStream(int, int, String, Caps)`, `writeAudioStream(int, byte[], int, int)`, `finishAudioStream(int)`, `cancelAudioStream(int)`, `active(String)`, `getClientList(int): ClientInfo[]`, `fetchClientList()`, `removeClient(String, int): int`. |
| Removed | The old `AudioStream` inner class (v1.0's `openAudioStream(...)`-returned handle, with its own `write`/`finish`/native start-write-finish trio) is gone, replaced by the flatter `startAudioStream`/`writeAudioStream`/`finishAudioStream`/`cancelAudioStream` method group directly on `CXRSocketProtocol`. |

This is the transport underneath `CXRServiceBridge`'s own `sendMessage`/`subscribe`/`startAudioStream` (whose *own* public surface is unchanged, per the "Confirmed unchanged" list in the v1.4 section above) — `CXRSocketProtocol` is not itself part of any documented public entry point in `cxr-s/*.md`, so no doc in this repo needs updating for this table, but it is recorded here for anyone extending `cxr-s/data-structure.md` or writing a lower-level transport doc later.

### Toolchain / packaging changes

- **AAR manifest `package` attribute changed** from `com.rokid.cxr` to `com.rokid.cxr.servicebridge`, and **`targetSdkVersion` moved from 32 to 33** (`minSdkVersion` stays `28` in both). `BuildConfig` moved in lockstep from `com.rokid.cxr.BuildConfig` to `com.rokid.cxr.servicebridge.BuildConfig` (same pattern already documented for `client-l` in `cxr-l/release-notes.md` — an AAR-internal namespace/manifest attribute, not a change to any public `com.rokid.cxr.*` API package).
- `proguard.txt` is empty (0 bytes) in both v1.0 and v1.1 — no consumer ProGuard/R8 rules are bundled by this artifact in either version.
- `R.txt` and `aar-metadata.properties` (`minCompileSdk=1`, `minAndroidGradlePluginVersion=1.0.0`) are unchanged.
- No dependency changes: neither v1.0 nor v1.1's `.pom`/`.module` declares any dependency (see "What this artifact is" above) — this holds for every version through 1.4.

### Native libraries — all five files changed (both ABIs)

| Library | v1.0 (arm64-v8a) | v1.1 (arm64-v8a) | Changed? |
|---------|------------------|------------------|----------|
| `libcaps.so` | 106,640 bytes | 106,640 bytes | Hash changed (`6726e8a9...` → `2755f866...`) despite identical size — recompiled, not just resized. |
| `libcxr-bridge-jni.so` | 111,632 bytes | 118,872 bytes | Changed (grew, backs the new `AudioRecordHelper`/`CXRServiceBridge` audio-record natives). |
| `libcxr-sock-proto-jni.so` | 604,656 bytes | 659,768 bytes | Changed (grew, backs the expanded `CXRSocketProtocol` surface above). |
| `libflora-cli.so` | 157,192 bytes | 157,280 bytes | Changed (small growth). |
| `libmutils.so` | 417,296 bytes | 432,360 bytes | Changed (grew). |

Every native library was rebuilt for v1.1; `libcaps.so`, `libflora-cli.so`, and `libmutils.so` then remain byte-identical from v1.1 all the way through v1.4 (re-confirmed by md5sum at each subsequent version above) — only `libcxr-bridge-jni.so` and (less consistently) `libcxr-sock-proto-jni.so` continued to change in v1.2–v1.4.

## v1.0 — baseline (currently/previously documented version)

`com.rokid.cxr:cxr-service-bridge:1.0` is the version this repo's `cxr-s/*.md` docs (`data-structure.md`, `message-sending.md`, `message-subscription.md`, `manage-device-connection.md`) were originally written against, previously cited in `cxr-s/sdk-import.md` as the timestamped snapshot coordinate `1.0-20260522.063600-105`. The plain `1.0` release coordinate resolves to a byte-identical AAR (1,076,548 bytes; see the publication-timeline table above), so this changelog treats `1.0` as the stable name for that same build.

**Class inventory (16 classes):** `BuildConfig`, `Caps` (+ `Caps.Value`, `Caps.Binary`, `Caps.IncorrectTypeException`, `Caps$1`), `CXRServiceBridge` (+ `MsgCallback`, `MsgReplyCallback`, `Reply`, `StatusListener`), `CXRSocketProtocol` (+ `Callback`, `AudioStream`, `CXRSocketProtocol$1`), `ReplyImpl`. No `RLog`, no `AudioRecordHelper`, no `com.rokid.cxrservice.ICXRService` — all three were added in v1.1 (see above).

Everything this repo's `cxr-s/data-structure.md`, `message-sending.md`, and `message-subscription.md` document (`Caps`'s full read/write API, `CXRServiceBridge.sendMessage`/`subscribe`, `MsgCallback`/`MsgReplyCallback`/`Reply`) matches the v1.0 decompile exactly — those three docs needed no correction from this changelog pass beyond the v1.1 constructor change noted above (which affects `message-sending.md`/`message-subscription.md`'s example code, not the documented API tables themselves).

## Summary table

| Version | Class count | AAR size | `Caps`/`sendMessage`/`subscribe` API | Notable change |
|---------|-------------|----------|----------------------------------------|-----------------|
| 1.0 | 16 | 1,076,548 B | Baseline (documented in this repo) | — |
| 1.1 | 27 | 1,136,671 B | Unchanged | **Breaking:** `CXRServiceBridge()` → `CXRServiceBridge(Context)`; new raw audio-record subsystem; `StatusListener.onAudioNoise` added; `CXRSocketProtocol` overhauled |
| 1.2 | 27 | 1,136,682 B | Unchanged | Breaking: `AudioRecordCallback.onData`/`readAudioData`/`audioData` change `short[]` → `byte[]` |
| 1.3 | 27 | 1,137,916 B | Unchanged | New `CXRServiceBridge.cancelMessage(int)` |
| 1.4 | 27 | 1,138,221 B | Unchanged | `openAudioRecord`/`onData`/`ICXRService.openAudioRecord` gain a 3rd `int` param; new `getVad(long)` |

## Practical takeaway for integrators

Everything this repo currently documents in `cxr-s/data-structure.md`, `cxr-s/message-sending.md`, and `cxr-s/message-subscription.md` — the `Caps` serialization API and the `sendMessage`/`subscribe`/`MsgCallback`/`MsgReplyCallback`/`Reply` message channel — is **unchanged from v1.0 through v1.4**. The only change that affects those docs' example code is the v1.1 constructor signature change (`CXRServiceBridge()` → `CXRServiceBridge(Context)`), which is corrected in `cxr-s/manage-device-connection.md`, `cxr-s/message-sending.md`, and `cxr-s/message-subscription.md` as part of this update. Everything else new in 1.1–1.4 — raw on-device audio recording (`AudioRecordHelper`, `CXRServiceBridge.AudioRecordCallback`/`AudioRecordParam`, the `com.rokid.cxrservice.ICXRService` AIDL contract), `cancelMessage`, and the expanded low-level `CXRSocketProtocol` transport — is **not yet documented anywhere in this repo** and would need a new doc (e.g. `cxr-s/audio-recording.md`) to cover properly; this changelog only records what changed, not how to use the new surface, since no official documentation or working sample exists to confirm intended usage.

**New integrations should target `1.4`** (the current Maven `release`) rather than the previously-cited `1.0-20260522.063600-105` snapshot coordinate.
