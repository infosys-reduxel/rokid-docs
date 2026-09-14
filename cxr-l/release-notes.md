# CXR-L SDK Release Notes

_Source: https://developerdoc.rokid.com/sdk (Chinese, fetched 2026-06-11; official Rokid changelog). The v1.0.4 device-control APIs are now confirmed by an official changelog (published 2026-06-29, fetched 2026-07-03); the v1.0.4 session-lifecycle additions remain a provisional binary-diff reconstruction not covered by the official text. `client-l:1.1.0` (uploaded to Maven 2026-07-02) still has no official changelog as of 2026-07-04 — see the v1.1.0 entry below for a binary-diff reconstruction. `client-l:1.1.1` (uploaded to Maven 2026-08-14) supersedes 1.1.0 and is documented below from a binary diff + `.pom` diff performed 2026-08-19; `developerdoc.rokid.com/sdk` was checked again on 2026-08-19 and still rendered only as a JavaScript SPA shell in this environment (no Firecrawl-class JS-rendering tool was available this cycle, unlike some earlier cycles — see `.claude/agents/rokid-sources.md`), so it is unknown whether an official v1.1.x changelog has since been published there. `client-l:1.1.2` (Maven `lastUpdated` 2026-09-10; `developerdoc.rokid.com/sdk`'s SDK-picker card shows "更新于 2026.09.08") is documented below from a binary diff + `.pom` diff performed 2026-09-13 — the dev-portal SDK picker only shows a version badge, not a changelog entry, so this remains provisional._

The CXR-L SDK (Android/iOS) is a developer toolkit for extending the scenarios of the Rokid AI app. The Rokid AI app establishes the connection to Rokid Glasses; developers integrate the CXR-L SDK into their own apps to access the Glasses' I/O capabilities — image, audio, display, and command channels — through the Rokid AI app.

## v1.1.2 — Maven `lastUpdated` 2026-09-10 (provisional binary-diff reconstruction — no official changelog)

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.1` and `client-l:1.1.2` AARs and their Maven `.pom` files (diffed 2026-09-13). AAR size: 171,307 bytes, essentially unchanged from the 171,369-byte v1.1.1 AAR (-62 bytes). `BuildConfig.BUILD_TIME` embedded in the 1.1.2 AAR reads `2026-08-28 16:36:21`; `maven-metadata.xml`'s `lastUpdated` (`20260910022017`) is a few weeks later, i.e. published to Maven roughly two weeks after it was built. `developerdoc.rokid.com/sdk`'s SDK-picker card independently shows CXR-L at "1.1.2" "更新于 2026.09.08" (updated 2026.09.08), consistent with a build-then-publish gap; no changelog prose accompanies the badge.

**Theme: packaging- and toolchain-only release. No documented public API surface changed.**

- **`.pom` diff**: only the `<version>` element changed (`1.1.1` → `1.1.2`). No dependency version changes — `cxr-service-bridge`, `kotlin-stdlib`, `gson`, and `kotlinx-coroutines-android` pins are all identical to v1.1.1.
- **`AndroidManifest.xml`**: byte-for-byte identical to v1.1.1 (`cmp` confirms no diff). Same `<queries>` block, same `minSdkVersion="28"`.
- **Toolchain bump, not a rewrite.** Every one of the AAR's ~230 `.class` files hash-differs from its v1.1.1 counterpart, but this is a compiler-version artifact, not evidence of a source change: `BuildConfig`'s embedded R8 marker moved from `{"backend":"cf",...,"version":"8.1.56"}` to `{"backend":"cf",...,"version":"8.2.33"}`. Confirmed via `javap -p` diffing every class: the only systematic difference is that Kotlin `object`/enum static initializers are now emitted as package-private `static {}` instead of `public static {}` (an R8 output convention, not a visibility change — static initializers are never externally invocable either way). This is the same "recompile with a newer toolchain" pattern already documented for the v1.0.4 → v1.1.0 and v1.1.0 → v1.1.1 transitions.
- **The entire v1.0.x `CXRLink`/`ExternalAppClient` callback API and the v1.1.0+ `com.rokid.cxr.session` API are unchanged at the class/method-signature level** — every type and method documented in [api-reference.md](api-reference.md) is present in v1.1.2 with an identical signature, re-confirmed via `javap -p`.

**Two real (non-cosmetic) binary-diff findings, both undocumented by any official changelog:**

1. **`GlassInfo.sn` field removed.** `com.rokid.cxr.link.utils.GlassInfo` (the struct behind `ICXRLinkCbk.onGlassDeviceInfo`) drops its `public java.lang.String sn` field in v1.1.2 — every other field (`deviceName`, `batteryLevel`, `sound`, `brightness`, `systemVersion`, `ischarging`, `wearingStatus`, `screenOn`) is unchanged. This is a breaking change for any integrator reading the device serial number off `GlassInfo`; no replacement field was added. See [api-reference.md](api-reference.md#glassinfo-v103) — updated to reflect the removal.
2. **`ExternalAppClient` drops 11 minified public methods.** The public (obfuscated, single/double-letter-named) overloads `a(String)`, `a`/`b(IDeviceStatusCallback)`, `a`/`b(IImageStreamCallback)`, `a`/`b(IAudioStreamCallback)`, `a`/`b(ICustomViewCallback)`, and `a`/`b(ICustomCmdCallback)` are gone from `com.rokid.sprite.aiapp.externalapp.example.ExternalAppClient` in v1.1.2. The internal `access$register*Callback`/`access$unregister*Callback` synthetic accessors they wrapped are still present and unchanged, and none of these `a`/`b` overloads were ever part of this repo's documented `ExternalAppClient`/`CXRLink` Public Methods table (they are not the same as, and sit alongside, the named `IMediaStreamService` AIDL methods documented in [api-reference.md](api-reference.md)). Reads as dead/duplicate obfuscated wrapper cleanup rather than a change to any capability this repo documents, but is noted here for completeness since it is a genuine public-method removal.

## v1.1.1 — published 2026-08-14 (provisional binary-diff reconstruction — no official changelog)

> **Provisional — not an official Rokid changelog.** Reconstructed from a binary diff of `client-l:1.1.0` and `client-l:1.1.1` AARs and their Maven `.pom` files (diffed 2026-08-19). AAR size: 171,369 bytes, **down** 86.7% from the 1,286,574-byte v1.1.0 AAR (though still +143% vs the 70,543-byte v1.0.4 baseline). `BuildConfig.BUILD_TIME` embedded in the 1.1.1 AAR reads `2026-08-14 15:43:24`, and `maven-metadata.xml`'s `lastUpdated` (`20260814092031`) agrees. `client-l:1.1.0` was live on Maven for roughly six weeks (2026-07-02 – 2026-08-14) before being superseded by this release.

**Theme: v1.1.1 fixes the v1.1.0 packaging defect described below — it drops the erroneously bundled `cxr-service-bridge` classes and native libraries and restores a normal external dependency — while keeping the new `com.rokid.cxr.session` API introduced in v1.1.0 fully intact.**

The v1.0.x `CXRLink`/`ExternalAppClient` API and the `com.rokid.cxr.session` API added in v1.1.0 (both documented above/below and in [api-reference.md](api-reference.md#cxrsession-api-v110)) are **unchanged** in v1.1.1 at the class/method level — every public type from v1.1.0's `com.rokid.cxr.session` package and every v1.0.x `CXRLink` method is still present with an identical signature. What changed is packaging only:

- **The bundled `com.rokid.cxr` root package is gone.** `com.rokid.cxr.Caps`, `com.rokid.cxr.CXRSocketProtocol`, `com.rokid.cxr.CXRServiceBridge`, and `com.rokid.cxr.RLog` — the `cxr-service-bridge` classes that v1.1.0 erroneously bundled directly into `client-l`'s own `classes.jar` (see the v1.1.0 entry below) — are removed from `classes.jar` in v1.1.1.
- **All five native `.so` libraries are removed** from both `jni/arm64-v8a/` and `jni/armeabi-v7a/` (`libcaps.so`, `libcxr-sock-proto-jni.so`, `libcxr-bridge-jni.so`, `libflora-cli.so`, `libmutils.so`). This alone accounts for the great majority of the AAR's size reduction.
- **`cxr-service-bridge` is restored as an external POM dependency** — `.pom` diff shows `client-l:1.1.1` now depends on `com.rokid.cxr:cxr-service-bridge:1.0-20260715.121510-107` (release version still `1.0`; this is a newer snapshot build than the `1.0-20260522.063600-105` build v1.0.4 depended on — see the `maven.rokid.com` entry in `.claude/agents/rokid-sources.md` for confirmation the published `cxr-service-bridge` *release* itself has not moved past `1.0`).
- **The AAR's own library package/namespace changed** from `com.rokid.cxr.client.extend` (v1.0.x and v1.1.0) to `com.rokid.cxr.link` (v1.1.1). This is the AAR's internal `BuildConfig`/manifest package attribute only — it does not affect the public `com.rokid.cxr.link.*` API package, which was already named `com.rokid.cxr.link` in earlier versions.
- `minSdkVersion` remains `28`; `AndroidManifest.xml` is otherwise unchanged (same `<queries>` block).

**Dependency changes vs v1.1.0:**

| Dependency | v1.1.0 | v1.1.1 |
|------------|--------|--------|
| `cxr-service-bridge` | *(bundled directly, not a POM dependency)* | `1.0-20260715.121510-107` (restored as an external dependency) |
| `kotlin-stdlib` | `1.6.0` | `1.9.0` |
| `gson` | `2.10.1` | `2.10.1` (unchanged) |
| `kotlinx-coroutines-android` | `1.6.4` | `1.9.0` |

**Practical takeaway for integrators:** apps that briefly integrated `client-l:1.1.0` were pulling in ~2.4 MB of native code across two ABIs and a fat `classes.jar`, none of which was needed unless they used the new `com.rokid.cxr.session` API's underlying transport directly. `client-l:1.1.1` is a strict size/packaging fix on top of the same API surface — **new integrations should target 1.1.1, not 1.1.0.**

## v1.1.0 — uploaded to Maven 2026-07-02 (provisional binary-diff reconstruction — no official changelog)

> **Superseded by v1.1.1 (published 2026-08-14, see above).** The packaging defect described in this entry — a bundled `cxr-service-bridge` (Java classes + 5 native `.so` libraries) inside the `client-l` AAR — was fixed in v1.1.1, which removed the bundling and restored `cxr-service-bridge` as a normal external dependency. The `com.rokid.cxr.session` API introduced in this release is unaffected and remains current in v1.1.1. This entry is retained for historical reference; **new integrations should use v1.1.1.**

> **This entire section is a provisional binary-diff reconstruction, not covered by any official changelog.** As of 2026-07-04, `developerdoc.rokid.com/sdk` (CXR-L tab) still only shows the changelog through v1.0.4 (dated 2026-06-29); no v1.1.0 entry exists there. Maven's `maven-metadata.xml` for `com.rokid.cxr:client-l` was fetched with cache bypass on 2026-07-04 and shows `release` moved from `1.0.4` to `1.1.0` and `lastUpdated` moved from `20260625070819` to `20260702091606` — i.e. the artifact itself was uploaded 2026-07-02, five days before this write-up. The findings below come from downloading and diffing `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/1.0.4/client-l-1.0.4.aar` (70,543 bytes) against `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/1.1.0/client-l-1.1.0.aar` (1,286,574 bytes) — a **+1,724 % size increase**, by far the largest jump of any point release in this SDK's history — using `unzip`, `javap -p`, `strings`, and `md5sum` (no `jadx`/`apktool`/dedicated Android decompiler was available in this environment; see the tooling note at the end of this entry).

**Theme: the SDK now bundles its own native CXR wire-protocol stack, and ships an entirely new coroutine/`StateFlow`-based `com.rokid.cxr.session` API alongside the existing callback-based `CXRLink`/`ExternalAppClient` API.**

### Where the +1,724 % actually comes from

Unzipping both AARs and comparing file trees shows the growth is *not* concentrated in one place — it's split roughly between newly-bundled native code and a genuinely larger Java/Kotlin surface:

| Component | v1.0.4 | v1.1.0 | Change |
|-----------|--------|--------|--------|
| `classes.jar` | 79,030 bytes | 203,274 bytes | +157 % (66 → 160 `.class` files) |
| `jni/arm64-v8a/*.so` (5 native libs) | — (none) | 1,466,648 bytes uncompressed | new |
| `jni/armeabi-v7a/*.so` (5 native libs) | — (none) | 957,120 bytes uncompressed | new |
| `proguard.txt` (consumer ProGuard/R8 rules) | — (none) | 2,037 bytes | new |
| `AndroidManifest.xml`, `R.txt`, `network_security_config.xml`, `aar-metadata.properties` | unchanged | unchanged (byte-identical) | none |

The AAR is a zip, so the on-disk 1,286,574-byte figure is compressed; the five native `.so` files alone total roughly 2.4 MB uncompressed across both ABIs, which is the dominant contributor to the size jump. The remaining growth is real new Java/Kotlin bytecode: `classes.jar` grew from 66 to 160 class files.

**Every one of the 66 classes present in v1.0.4 is still present in v1.1.0 with the same public API** (confirmed via `javap -p` diffing, see below) — this is a strictly additive release at the class-inventory level, not a rewrite. All 66 pre-existing `.class` files hash-differ from their v1.0.4 counterparts (`md5sum`), which is expected since the whole module was recompiled with a newer toolchain (new Kotlin metadata annotations were observed by version-diffing `javap` output — see the "Toolchain artifact, not an API change" note below), but line-for-line `javap -p` output for the SDK's core entry points (`CXRLink`, `ExternalAppClient`, `AuthorizationHelper`, `GlassInfo`, `CxrDefs`, etc.) is unchanged apart from the specific additions listed below.

### New: bundled native CXR wire-protocol stack (`com.rokid.cxr` root package)

v1.1.0 adds a `com.rokid.cxr` package (as opposed to the existing `com.rokid.cxr.link` / `com.rokid.cxr.session` packages) containing:

| Class | Kind | Notes |
|-------|------|-------|
| `com.rokid.cxr.Caps` | class | The Caps binary serialization format (see [cxr-s/data-structure.md](../cxr-s/data-structure.md)) — `writeInt32`/`writeInt64`/`write(String)`/`write(byte[])`/`write(Caps)` etc., plus **native** `serialize()`/`parse(byte[], int, int)`/`dump()`. |
| `com.rokid.cxr.CXRSocketProtocol` | class | Native-backed BLE/Bluetooth-socket framing layer — `run(BluetoothSocket, Callback, Parameter)`, `request(int, String, Caps)`, `send(String, Caps, byte[])`, `startAudioStream(...)`, `getClientList(int)`. This is architecturally the same "CXRSocketProtocol framing layer" referenced under CXR-M in this repo's CLAUDE.md — CXR-L 1.1.0 now embeds it directly rather than only depending on it transitively. |
| `com.rokid.cxr.CXRServiceBridge` | class | Native-backed message bridge — `sendMessage(String, Caps[, byte[]])`, `subscribe(...)`, `startAudioStream(...)`, `startBTPairing(int)`, `sendARTCFrame(...)`. |
| `com.rokid.cxr.RLog` | class | Thin native-backed logging shim (`v`/`d`/`i`/`w`/`e`). |
| `com.rokid.cxr.BuildConfig` | class | Standard AGP-generated build config (`DEBUG`, `LIBRARY_PACKAGE_NAME`, `BUILD_TYPE`). |

**This is not new code Rokid wrote for client-l** — it is the `cxr-service-bridge` module, previously consumed as an external transitive Maven dependency (`com.rokid.cxr:cxr-service-bridge:1.0-20260522.063600-105`, per the v1.0.3/v1.0.4 entries above). Two independent pieces of evidence confirm this:

1. **The POM dependency was removed.** Diffing `client-l-1.0.4.pom` against `client-l-1.1.0.pom`: the `cxr-service-bridge` dependency entry is gone in 1.1.0, while `kotlin-stdlib:1.6.0` and `gson:2.10.1` remain and a **new** `kotlinx-coroutines-android:1.6.4` (runtime scope) was added. See the dependency table below.
2. **Embedded debug path strings in the native libraries.** Running `strings` on `jni/arm64-v8a/libflora-cli.so` shows compile-time paths still baked into the binary: `/Users/ming/work/XR2/CXR/caps-java/cxr-service-bridge/src/main/cpp/flora/src/{adap.h,cli.cpp,cli.h}` — i.e. the object file was built from the `cxr-service-bridge` module's own `flora` C++ sources.

In short: v1.1.0 changes `cxr-service-bridge` from an external POM dependency into a **bundled, fat-AAR dependency** — its Java classes now ship inside `client-l`'s own `classes.jar`, and its five native libraries ship inside `client-l`'s own `jni/` folders. Consuming apps no longer need to resolve `cxr-service-bridge` separately, but they now pull in ~2.4 MB of native code (across two ABIs) whether or not they use the new APIs that need it.

**Native libraries added** (both `arm64-v8a` and `armeabi-v7a`; all stripped, built with Android's clang 18.0.2 per embedded compiler strings):

| Library | arm64-v8a size | armeabi-v7a size | JNI-bound Java class (by exported native methods) |
|---------|---------------|-------------------|----------------------------------------------------|
| `libcaps.so` | 106,640 bytes | 54,364 bytes | `com.rokid.cxr.Caps` (`serialize`, `parse`, `dump`) |
| `libcxr-sock-proto-jni.so` | 641,320 bytes | 476,916 bytes | `com.rokid.cxr.CXRSocketProtocol` (the largest of the five — framing/audio-stream natives) |
| `libcxr-bridge-jni.so` | 129,048 bytes | 71,076 bytes | `com.rokid.cxr.CXRServiceBridge` |
| `libflora-cli.so` | 157,280 bytes | 89,936 bytes | Rokid's internal "flora" pub/sub IPC client (linked into the bridge; contains embedded `cxr-service-bridge/.../flora/` source paths) |
| `libmutils.so` | 432,360 bytes | 264,828 bytes | Shared native utility library (exact symbol table not enumerated this pass — stripped binary, no exported-symbol analysis performed) |

`libmutils.so` and the internals of `libflora-cli.so`/`libcxr-sock-proto-jni.so` were **not decompiled** — this environment had no `jadx`, `apktool`, or native disassembler beyond `strings`/`file`, so their contents beyond size, architecture, and the JNI method names visible via `javap`/`strings` are not characterized here.

### New: `com.rokid.cxr.session` — a coroutine/`StateFlow`-based session API

Alongside the existing `CXRLink extends ExternalAppClient` callback API (which is untouched — see below), v1.1.0 adds a parallel, higher-level API surface in a new `com.rokid.cxr.session` package (18 non-synthetic public types, plus internal `CxrSessionImpl`/`CapabilityBroker` implementation classes and ~40 Kotlin coroutine-continuation classes generated for `suspend`-style internals). This is confirmed as genuine public API — not just internal implementation detail — because `v1.1.0`'s bundled `proguard.txt` (new in this release; see below) explicitly `-keep`s `com.rokid.cxr.session.CxrSession`, `CxrSessionManager`, `SessionConfig`, and the session enums/callbacks as `public`.

Entry point: `com.rokid.cxr.session.CxrSessionManager.Companion.getInstance(Context): CxrSessionManager`. See [api-reference.md](api-reference.md) for full decompiled signatures of `CxrSessionManager`, `CxrSession`, `SessionConfig`, `SessionResult<T>`, the session enums (`SessionType`, `SessionState`, `SessionErrorCode`, `CloseReason`, `PausedReason`, `TerminatingReason`, `AiInterceptMode`), the sealed `RokidAppStatus` hierarchy (`Compatible`/`NotInstalled`/`VersionTooLow`), and the five callback interfaces (`ISessionLifecycleCbk`, `IAudioCallback`, `IImageCallback`, `ICustomCmdSessionCallback`, `IGlassesEventListener`).

At a glance, this new API appears designed to replace the manual `configCXRSession`/`ICXRSessionCbk` dance from v1.0.4 with a single `CxrSession` object exposing a Kotlin `StateFlow<SessionState>`, typed `SessionResult<T>` return values (a `code`/`data`/`message` triple with `isSuccess`), and a richer state machine (`Idle → Starting → Started ⇄ Paused → Terminating`) than the four-value `CXRSessionState` enum from v1.0.4. Whether this new API is meant to **replace** `CXRLink`/`ExternalAppClient` going forward, or supplement it, is not stated anywhere in the bundled artifacts — no prose documentation for it has been found. Internally, `CxrSessionImpl` (package-private implementation of `CxrSession`) is implemented as a wrapper around the very same `ExternalAppClient` (it holds a `com.rokid.sprite.aiapp.externalapp.example.ExternalAppClient` field and delegates to it via a `CapabilityBroker` helper class), so the old and new APIs share one underlying connection.

### New public members on existing classes

- **`com.rokid.cxr.link.callbacks.ICXRLinkCbk` gains `onGlassLauncherResume()`.**

  > **Breaking change for implementors of `ICXRLinkCbk`.** Any class implementing this interface must now also implement `onGlassLauncherResume()`, in addition to the three methods added in v1.0.3 and the interface's original method. Add an empty stub if the behaviour is not needed.

- **`ExternalAppClient` (base class of `CXRLink`) gains `setCXRGlassAppCbk(IGlassAppCbk)`.** A public setter for the `IGlassAppCbk` callback (install/uninstall/open/stop/resume/query-app results). The `IGlassAppCbk` interface itself and the `appUploadAndInstall`/`appUninstall`/`appStart`/`appStop`/`appIsInstalled` methods that use it were already present in v1.0.4's decompile, but no public method to *register* the callback existed there — `setCXRGlassAppCbk` is the first way to wire it up via public API.

- **`com.rokid.sprite.aiapp.externalapp.auth.AuthorizationHelper` gains a public constant `minRokidAppRequired = 10090000`.** This is a plain `public static final int` (a real `ConstantValue`, confirmed via `javap -c -constants`), not a synthetic artifact. It appears to be the actual minimum Rokid AI app `versionCode` the SDK now checks against — a more precise, code-visible figure than the `100000` placeholder this repo's `api-reference.md` has been carrying (see `Notes` item 9 there), though this repo has not independently confirmed which of the two values `isRequiredRokidAppInstalled()` actually enforces at runtime without further reverse engineering.

### Toolchain artifact, not an API change (documented for transparency)

`javap -p` reports the `GlassPermission` enum's constructor as `(java.lang.String)` in v1.0.4 but as `(int, java.lang.String, java.lang.String)` in v1.1.0. This looks like a constructor signature change at first glance, but it is not one: the enum's three constants (`MICROPHONE`, `CAMERA`, `MEDIA`), their string values (`glass.permission.MICROPHONE` etc.), and the class's only declared field (`a: String`, exposed via `getPermission()`) are byte-for-byte identical in both versions' static initializers. The extra `int`/`String` parameters visible in the v1.1.0 disassembly are the standard implicit enum `(ordinal, name)` parameters that `javap` chose to display in this build but hid in the v1.0.4 build — most likely because the two AARs were compiled with different Kotlin/AGP toolchain versions (consistent with the wholesale recompilation of all 66 pre-existing classes noted above). No functional change to `GlassPermission` is claimed here.

### Dependency changes vs v1.0.4

| Dependency | v1.0.4 | v1.1.0 |
|------------|--------|--------|
| `cxr-service-bridge` | `1.0-20260522.063600-105` (external POM dependency) | **removed from POM** — bundled directly into `client-l`'s `classes.jar` and `jni/` (see above) |
| `kotlin-stdlib` | `1.6.0` | `1.6.0` (unchanged) |
| `gson` | `2.10.1` | `2.10.1` (unchanged) |
| `kotlinx-coroutines-android` | not a dependency | **`1.6.4`** (new, runtime scope) — required by the new `StateFlow`/`MutableStateFlow`-based `com.rokid.cxr.session` API |

> Source: `client-l-1.0.4.pom` vs `client-l-1.1.0.pom`, both fetched from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/` on 2026-07-04.

### AndroidManifest / packaging changes

- `AndroidManifest.xml` is **byte-identical** to v1.0.4 (`minSdkVersion="28"`, same `<queries>` block for `com.rokid.sprite.aiapp`/`com.rokid.sprite.global.aiapp`). No manifest changes in this release.
- **New `proguard.txt` bundled in the AAR** (2,037 bytes; absent from v1.0.4). Its comments are in Chinese and explicitly label it `# CXR-L SDK v1.1.0 — 消费者混淆规则` ("consumer obfuscation rules"), confirming the version number independently of the Maven metadata. It `-keep`s `Caps`'s core methods, all of `com.rokid.sprite.aiapp.externalapp.**` (the AIDL-generated interfaces), `GlassInfo`'s fields (for Gson), and — most usefully for this write-up — explicitly lists the `com.rokid.cxr.session.*` types that are meant to be public API (`CxrSession`, `CxrSessionManager`, `SessionConfig`, the session enums, `SessionResult`, `GlassesInfo`, `AuthResult`, and the five callback interfaces). It also strips debug/verbose `android.util.Log` calls via `-assumenosideeffects`.

### Tooling note

This entry was produced without `jadx`, `apktool`, or `dex2jar` (none were available in the sandbox this pass ran in). `classes.jar` inside an AAR is a plain JVM `.class` jar (not a `.dex`), so `javap -p` (bytecode-level disassembly, not full Java/Kotlin source decompilation) was sufficient to enumerate every public/protected/package/private member of every class and diff the two versions method-for-method. `unzip -l`, `md5sum`, `file`, and `strings` were used for the file-tree diff and native-library identification. No Kotlin/Java source-level decompiler was used, so method **bodies** (beyond what bytecode offsets/constant-pool comments reveal, e.g. the `minRokidAppRequired` constant value) were not reconstructed — the tables above are complete for signatures but do not describe internal control flow.

## v1.0.4 — published 2026-06-18 (official changelog published 2026-06-29)

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, fetched 2026-07-03; changelog dated 2026-06-29). The device-control APIs below are now confirmed by the official changelog. The session-lifecycle callbacks (`ICXRSessionCbk`, `CXRSessionReason`, `CXRSessionState`) remain a **provisional binary-diff reconstruction** — the official changelog does not mention them — from a diff of `client-l:1.0.3` and `client-l:1.0.4` AARs (downloaded 2026-06-25 from `https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/`). AAR size: 70,543 bytes vs 65,494 bytes for v1.0.3 (+7.7 %).

**Official changelog (Android + iOS):**

1. Android `client-l` upgraded to 1.0.4.
2. New device-control APIs: `setGlassBrightness(level)` / `setGlassVolume(level)`; **level range confirmed as 0–15**.
3. `GlassInfo` gains `brightness` / `sound` fields, delivered via the `onGlassDeviceInfo` callback. Note: the repo's own binary-diff record shows these two `GlassInfo` fields were already present since v1.0.3 (see the v1.0.3 entry below) — the official changelog appears to restate them as part of the new 1.0.4 "Device Control" chapter rather than introduce them fresh. The genuinely new part is the pair of *setter* APIs (`setGlassBrightness`/`setGlassVolume`).
4. New "设备控制" (Device Control) documentation chapter (Android + iOS), covering brightness/volume set and query.
5. Android sample archive updated to v1.0.4.
6. iOS `RGCxrClient` gains `setBrightness()` / `getBrightness()` / `setVolume()` / `getVolume()`; `RGCxrDeviceInfo` gains `brightness` / `sound` fields.
7. iOS documentation and sample version unified to v1.0.4 (previously pinned at v1.0.1 — see the v1.0.3 entry below).

**Theme: structured session lifecycle callbacks and direct device controls.**

**Additional technical findings (binary diff of v1.0.3 → v1.0.4 AAR) — provisional, not covered by the official changelog:**

**New interfaces:**

- `com.rokid.cxr.link.callbacks.ICXRSessionCbk` — session lifecycle callback interface.

  | Method | Parameter | Description |
  |--------|-----------|-------------|
  | `onSessionAvailable` | `CXRSessionReason` | Session became available (glasses and link ready for use). |
  | `onSessionStart` | `CXRSessionReason` | Session started (app's scene is now active). |
  | `onSessionPause` | `CXRSessionReason` | Session paused (e.g. OS overlay took over; scene suspended). |
  | `onSessionUnavailable` | `CXRSessionReason` | Session became unavailable (link disconnected or glasses idle). |

  > **Breaking change for implementors of `ICXRSessionCbk`.** Any class implementing this interface must provide all four methods.

**New enums:**

- `com.rokid.cxr.link.utils.CxrDefs$CXRSessionReason` — reason code passed to all `ICXRSessionCbk` callbacks.

  | Constant | Description |
  |----------|-------------|
  | `SESSION_GLASS_READY` | Glasses signalled ready state. |
  | `SESSION_GLASS_IDLE` | Glasses entered idle / standby. |
  | `SESSION_LINK_CONNECT` | CXR link connected. |
  | `SESSION_LINK_DISCONNECT` | CXR link disconnected. |
  | `SESSION_SCREEN_OFF` | Glasses display turned off. |
  | `SESSION_AI_START` | On-device AI session started. |
  | `SESSION_AI_STOP` | On-device AI session stopped. |
  | `SESSION_SCENE_TAKEOVER` | Another scene took over the display. |
  | `SESSION_OTHER` | Other / unspecified reason. |

- `com.rokid.cxr.link.utils.CxrDefs$CXRSessionState` — current session state, queryable via `getCXRSessionState()`.

  | Constant | Meaning |
  |----------|---------|
  | `SessionAvailable` | Session is available and ready. |
  | `SessionStart` | Session is active. |
  | `SessionPause` | Session is paused. |
  | `SessionUnavailable` | Session is unavailable. |

**New public methods on `ExternalAppClient` / `CXRLink`:**

- `boolean configCXRSession(CxrDefs.CXRSession, ICXRSessionCbk)` — 2-argument overload of the existing `configCXRSession(CXRSession)`. Registers a session lifecycle callback at the same time as configuring the session type. The 1-argument overload remains available.
- `CxrDefs.CXRSessionState getCXRSessionState()` — query the current session state.
- `boolean setGlassBrightness(int)` — set the glasses display brightness level programmatically. **Confirmed range: 0–15** (official changelog, 2026-06-29).
- `boolean setGlassVolume(int)` — set the glasses speaker volume level programmatically. **Confirmed range: 0–15** (official changelog, 2026-06-29).

**AndroidManifest change:**

`targetSdkVersion` attribute removed from the `<uses-sdk>` element in the AAR manifest (was `"28"`). The `minSdkVersion` remains `"28"`. This is an AAR-level declaration only; host apps are unaffected.

**Dependency changes vs v1.0.3:** None — `cxr-service-bridge:1.0-20260522.063600-105`, `kotlin-stdlib:1.6.0`, and `gson:2.10.1` are unchanged.

## v1.0.3 — published 2026-06-02

> Source: official Rokid changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, fetched 2026-06-11).

`com.rokid.cxr:client-l:1.0.3` was uploaded to Maven on 2026-06-02 (AAR size: 65,494 bytes vs 57,145 bytes for 1.0.2, +14.6 %).

**Official changelog:**

1. Android `client-l` upgraded to 1.0.3.
2. Required companion app: when integrating `client-l:1.0.3`, Rokid AI App (China mainland) must be ≥ 1.7.14.
3. Documentation v1.0.3 rewritten from a developer-integration perspective, with unified "session construction" (会话构建) terminology throughout.
4. Android on-device Custom View chapter supplemented with a CustomView JSON Schema (LinearLayout, TextView, ImageView, RelativeLayout).
5. On-device CXR-S integration documentation merged into the CXR-L doc: SDK import, custom app integration, custom commands, key and broadcast chapters.
6. New reference sample apps published with OSS download archive: mobile-side `RenewCXRLSample` (`com.rokid.renewcxrlsample`) and glasses-side `CXRSWithCXRLSample` (`com.rokid.cxrswithcxrl`).
7. iOS documentation and sample remain at v1.0.1 — the version of iOS-specific chapters follows each platform chapter's own timeline.

**Additional technical findings (binary diff of v1.0.2 → v1.0.3 AAR):**

**New class:**

- `com.rokid.cxr.link.utils.GlassInfo` — data class representing a snapshot of connected-glasses state.

  | Field | Type | Description |
  |-------|------|-------------|
  | `deviceName` | `String` | Advertised Bluetooth device name |
  | `batteryLevel` | `int` | Battery level (0–100) |
  | `sound` | `int` | Current speaker volume level |
  | `brightness` | `int` | Display brightness level |
  | `systemVersion` | `String` | Glasses firmware / OS version string |
  | `ischarging` | `boolean` | Whether the glasses are on charge |
  | `sn` | `String` | Device serial number |
  | `wearingStatus` | `String` | Wearing-state descriptor (raw; see `onGlassWearingStatus`) |

**New callbacks on `ICXRLinkCbk`:**

- `void onGlassDeviceInfo(GlassInfo info)` — fired when the SDK receives a device-state update from the glasses. Provides a structured snapshot instead of discrete per-field queries.
- `void onGlassWearingStatus(boolean isWearing)` — fired when the glasses detect a wearing / not-wearing transition (via proximity / IMU sensor).
- `void onGlassAiInterrupt(boolean interrupted)` — fired when an in-progress AI session on the glasses is interrupted (e.g. by a system event or OS overlay).

  > **Breaking change for implementors of `ICXRLinkCbk`.** Any class implementing this interface must now implement the three new methods. Add empty stubs if the behaviour is not needed.

**AndroidManifest change:**

The AAR's `<queries>` block now also declares `com.rokid.sprite.global.aiapp` (in addition to the existing `com.rokid.sprite.aiapp`). This suggests Rokid has introduced or renamed the on-device AI app package for a new hardware variant or region — the SDK will now resolve to either package name when binding the AIDL service.

**Dependency changes vs v1.0.2:**

| Dependency | v1.0.2 | v1.0.3 |
|------------|--------|--------|
| `cxr-service-bridge` | `1.0-20260212.103714-88` | `1.0-20260522.063600-105` |
| `kotlin-stdlib` | `2.1.0` | `1.6.0` |
| `gson` | `2.10.1` | `2.10.1` |

> **Note on kotlin-stdlib downgrade.** The Kotlin stdlib runtime dependency was downgraded from 2.1.0 to 1.6.0. Apps that relied on the transitive Kotlin 2.x stdlib should declare their own `kotlin-stdlib` dependency at the desired version to avoid being silently downgraded by dependency resolution.

## v1.0.2 — published 2026-05-20

> Source: official changelog at `https://developerdoc.rokid.com/sdk` (CXR-L tab, fetched 2026-06-06).

`com.rokid.cxr:client-l:1.0.2` was uploaded to Maven on 2026-05-19. Rokid published the official changelog on 2026-05-20.

**Android changes:**

1. Android `client-l` upgraded to 1.0.2.
2. **Auth API change:** `requestAuthorization` now requires a `GlassPermission` array (e.g. microphone, camera, media). If the user has already authorized, the call can return a `Pair` synchronously — parse the token directly from that.
3. **`sendCustomCmd` enhancement:** now accepts a `Caps` object directly (in addition to the existing form).
4. `CXRLSample` updated to reflect the above API changes.

**iOS changes (RGCxrClient 1.0.2):**

5. iOS `RGCxrClient` upgraded to 1.0.2 via CocoaPods; requires the Rokid specs source to be configured in your `Podfile`.
6. App startup: `CxrClient.initialize(mode:options:)` now explicitly distinguishes `customApp` / `customView` session modes.
7. Auth scopes changed from string constants to SDK permission enums (e.g. `.microphone`).
8. Most capability APIs now return `RGCxrClientError?` synchronously. `sendCustomCmd` sends without a completion callback; subscribe to events via `notifyEventPublisher`.
9. `ios_cxr_l_sample` updated to reflect the above API changes.

## v1.0.1 — 2026-05-07

1. Initial SDK release.
2. Support for obtaining authorization from the Rokid AI app.
3. Support for creating on-device custom View scenes.
4. Support for creating on-device custom app scenes.
5. Support for accessing on-device audio.
6. Support for capturing photos through the glasses.
7. Support for custom-command exchange with on-device custom apps.
