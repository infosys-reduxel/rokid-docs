# CXR-S SDK Release Notes

> Source: `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/cxr-service-bridge/` (Maven metadata + AAR binary diff, checked 2026-09-28). No official Rokid changelog for `cxr-service-bridge` has been found on `developerdoc.rokid.com/sdk` or `ar.rokid.com/sdk` as of this pass — the entries below are **reverse-inferred from binary diffs** of the published AARs (class-list diff plus `javap` decompilation of changed/added classes). They describe the *observable* public-API delta only; they do not capture intent, deprecation notes, or behavior that doesn't show up in the class file. Treat them as a best-effort developer aid until Rokid publishes an authoritative changelog.

The CXR-S SDK (`com.rokid.cxr:cxr-service-bridge`) is the on-device counterpart that runs inside glasses-side apps on YodaOS-Sprite. It exposes the same `CXRServiceBridge` / `CXRSocketProtocol` / `Caps` classes documented in [data-structure.md](data-structure.md), [message-sending.md](message-sending.md), and [message-subscription.md](message-subscription.md); those pages were derived from `cxr-service-bridge:1.0` and remain valid for the message/Caps surface. This page tracks version-over-version changes to the bridge class itself.

Method to reproduce (run any time):

```sh
for V in 1.0 1.1 1.2 1.3 1.4; do
  curl -O "https://maven.rokid.com/repository/maven-public/com/rokid/cxr/cxr-service-bridge/$V/cxr-service-bridge-$V.aar"
done
# Unpack AAR -> classes.jar -> .class files
# Diff class lists and javap signatures across the versions.
```

## Changelog

### v1.4 — uploaded 2026-09-22 (inferred from binary diff)

AAR size: 1,138,221 bytes (+0.03% vs v1.3).

- **Breaking change:** `openAudioRecord` gained a parameter — signature changed from `openAudioRecord(String, int, AudioRecordCallback)` (v1.1–v1.3) to `openAudioRecord(String, int, int, AudioRecordCallback)`. The meaning of the new `int` parameter is not recoverable from bytecode alone; Rokid has not published an accompanying guide.

### v1.3 — uploaded 2026-09-18 (inferred from binary diff)

AAR size: 1,137,916 bytes (+0.11% vs v1.2).

- **New method:** `native void cancelMessage(int)` on `CXRServiceBridge` — cancels an in-flight message by the ID previously returned from `sendMessage`.

### v1.2 — uploaded 2026-09-17 (inferred from binary diff)

AAR size: 1,136,682 bytes (+0.001% vs v1.1). No observable class-list or method-signature changes vs v1.1 — treat as a maintenance/patch release.

### v1.1 — uploaded 2026-09-17 (inferred from binary diff)

> **This is the first release with public-API changes since the `1.0` baseline** that [data-structure.md](data-structure.md), [sdk-import.md](sdk-import.md), and [design-spec.md](design-spec.md) were derived from. AAR size: 1,136,671 bytes vs 1,076,548 bytes for v1.0 (+5.6%).

**Theme: local audio-record streaming via a new system-service AIDL interface, plus device/BT-pairing signature changes.**

**Breaking changes on `com.rokid.cxr.CXRServiceBridge`:**

| Method | v1.0 | v1.1+ |
|--------|------|-------|
| Constructor | `CXRServiceBridge()` | `CXRServiceBridge(Context)` — now requires an Android `Context`. |
| `startAudioStream` | `startAudioStream(int, String, Caps)` | `startAudioStream(int, int, String, AudioRecordParam)` — gains an `int` parameter and replaces the raw `Caps` parameter with the new `AudioRecordParam` class. |
| `startBTPairing` | `startBTPairing()` | `startBTPairing(int)` — gains an `int` parameter (purpose not recoverable from bytecode). |

**New public methods on `CXRServiceBridge`:**

- `native void appLaunch()` — no-argument native call, likely signals app-launch lifecycle to the bridge.
- `boolean openAudioRecord(String, int, int, AudioRecordCallback)` — opens a local audio-record stream identified by a `String` key (initial v1.1–v1.3 signature was the 3-argument `openAudioRecord(String, int, AudioRecordCallback)`; see v1.4 above for the 4-argument change).
- `void closeAudioRecord(String)` — closes a previously opened record stream.
- New private callback hook: `onAudioNoise(float)`.

**New public classes:**

- `com.rokid.cxr.CXRServiceBridge$AudioRecordCallback` — callback interface: `onStart(int)`, `onData(byte[], int, int)`, `onStop()`.
- `com.rokid.cxr.CXRServiceBridge$AudioRecordParam` — public fields `denoiseMode: int`, `rokidDtlnAEC: boolean`, `rokidBF: boolean`; has both a no-arg and a 3-arg constructor.
- `com.rokid.cxr.AudioRecordHelper` — a `ServiceConnection` implementation that binds to the new `ICXRService` AIDL service (below) to route audio-record requests. Exposes `start()`, `openAudioRecord(String, int, int, AudioRecordCallback)`, `closeAudioRecord(String)`, plus native helpers `readAudioData`, `getAudioChannels`, `createAudioQueue`, `destroyAudioQueue`, `getVad`.
- `com.rokid.cxr.AudioRecordHelper$AudioRecordInfo` — internal record-session bookkeeping (native pointer, callback, buffered audio data, channel count).
- `com.rokid.cxrservice.ICXRService` — a new AIDL interface (`extends android.os.IInterface`) with exactly two methods: `ParcelFileDescriptor openAudioRecord(String, int, int)` and `void closeAudioRecord(String)`. This is the system-side counterpart that `AudioRecordHelper` binds to — i.e. the actual audio pipe is now brokered through a `ParcelFileDescriptor` handed back by the `CXRService` system app (see [yodaos/docs/apps/cxr-service.md](../yodaos/docs/apps/cxr-service.md)) rather than delivered purely over the existing `Caps`/socket channel.
- `com.rokid.cxr.RLog` — internal logging helper (class present but not decompiled in detail here).
- `com.rokid.cxr.CXRSocketProtocol$AudioRecordParam`, `com.rokid.cxr.CXRSocketProtocol$ClientInfo` (public fields `mac: String`, `customInfo: String`, `status: int`), `com.rokid.cxr.CXRSocketProtocol$Parameter` (public fields `async: boolean`, `customInfo: String`) — new nested types on the wire-protocol class; likely parameter/metadata carriers for the audio-record and multi-client flows above.

**Removed/renamed:**

- Top-level `com.rokid.cxr.BuildConfig` was renamed to `com.rokid.cxr.servicebridge.BuildConfig`.
- `com.rokid.cxr.CXRSocketProtocol$1` and `com.rokid.cxr.CXRSocketProtocol$AudioStream` (present in v1.0) are gone from the class list in v1.1+.

<!-- Version-specific dependency/pom deltas were not captured for cxr-service-bridge 1.0–1.4 (no external Maven dependencies beyond the AOSP/Kotlin toolchain were observed in the AARs' pom.xml files during this pass). -->
