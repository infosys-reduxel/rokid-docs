# Rokid Upstream Source Registry

Scout reads this file to know which upstream documentation sources to monitor. Leader and Translator read it for source lookup.

## Schema (per entry)

```
- url: <fully-qualified URL>
  kind: developer-portal | github | release-notes | ota | sdk-maven | other
  covers: cxr-m | cxr-s | cxr-l | yodaos | hardware
  monitor_id: <Firecrawl monitor ID if registered, else empty>
  last_checked: YYYY-MM-DD
  last_known_version: X.Y.Z   # optional
  notes: <one line>
```

## Sources

- url: https://ar.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-16
  last_known_version: CXR-L 1.0.4 (portal changelog table; Maven has 1.1.1)
  notes: |
    React SPA "AIUI: The Next Frontier" developer homepage. Scraped 2026-08-16: SDK selection
    table under Rokid Glasses / YodaOS-Sprite unchanged in structure. CXR-L card links to
    a `t.rokid.com/uwxdzi51` shortlink resolving to `custom.rokid.com/.../84feb39f.../.../663f...html`
    (CXR-L doc workspace — see custom.rokid.com entry below; workspace now returns HTTP 200,
    no longer NoSuchKey, but is a client-rendered SPA shell). Bare-metal card now links to a
    NEW dedicated workspace `custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/`
    (previously shared workspace `57e35cd3...` with CXR-M/CXR-S) — sidebar shows "Version 1.0.0".
    YodaOS-Master tab remains; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-16
  last_known_version: CXR-L 1.0.4 (2026.06.25 per portal table; Maven at 1.1.1 since 2026-08-15), CXR-M 1.1.0 (portal lags Maven 1.2.2, unchanged), 眼镜端裸机开发 1.0.0 (2026.06.05 — up from last-recorded 0.0.1)
  notes: |
    /sdk scraped 2026-08-16 (also mirrored byte-for-byte at open.rokid.com/sdk?lang=zh):
    SDK comparison table now shows CXR-L 最新版本 1.0.4 (更新于 2026.06.25) — portal has NOT
    caught up to Maven's 1.1.0/1.1.1. 眼镜端裸机开发 now shows 最新版本 1.0.0 (更新于 2026.06.05) —
    this is a real, confirmed version bump from the previously recorded 0.0.1 (2026-03-01)
    baseline that local cxr-baremetal/ docs were translated from; flagged P1, see cxr-l and
    cxr-baremetal below for repo actions. /sprite hardware-spec table and FAQ intro text
    byte-compared against local yodaos/docs/sprite-overview.md — no drift. YodaOS-Master tab
    skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-16
  notes: |
    RECOVERED as of 2026-08-16 — no longer returns OSS NoSuchKey for the two previously-broken
    workspace roots. Both now return HTTP 200 via `pc/us/<pageId>.html` paths reached through
    the ar.rokid.com homepage card links:
    - CXR-L (84feb39f8ef141b0ad0326f902ab881f): `.../pc/us/663f26766e7348059905815bc022e1f7.html`
      resolves to a React SPA shell showing "Version 1.0.4" in the header nav after a
      --wait-for 4000ms scrape, but the document body never left its Ant Design loading-spinner
      state (`ant-spin-spinning`) in repeated attempts — chapter content is not retrievable via
      Firecrawl's markdown/HTML scrape modes. Confirms portal CXR-L changelog is still on 1.0.4.
    - CXR-M/CXR-S/裸机开发 workspace (57e35cd3ae294d16b1b8fc8dcbb1b7c7): not re-probed this cycle;
      bare-metal docs appear to have moved OUT of this shared workspace into a new dedicated one.
    - NEW bare-metal workspace (ff28c865a9634876be98cbc293588460): `.../pc/us/index.html` returns
      HTTP 200, header nav shows "Version 1.0.0" + "Document"/"Introduction" sidebar items after
      --wait-for 4000ms, but body is likewise stuck on the loading spinner — content not retrievable.
    Root cause appears to be client-side data fetching (XHR/fetch to a JSON content API) that
    Firecrawl's headless render does not wait long enough for, or that requires additional
    interaction/auth Firecrawl does not perform. Chapter-level content for CXR-L 1.1.x and
    bare-metal 1.0.0 remains structurally P0-adjacent (version confirmed, content unconfirmed).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-16
  notes: Not re-scraped this cycle (low priority, confirmed redirecting to open.rokid.com as of 2026-06-29; no in-scope content in prior checks). Registry `last_checked` advanced on the strength of the equivalent open.rokid.com/ar.rokid.com homepage scrape performed this cycle, which is the confirmed redirect target.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-16
  last_known_version: |
    client-l 1.1.1 (release; maven-metadata.xml lastUpdated 20260814092031 — NEW since last check 2026-06-30, was 1.0.4. AAR uploaded 2026-08-15. Intermediate 1.1.0 uploaded 2026-07-02, superseded — see notes.)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — lastUpdated timestamp advanced but <release>/<latest> both still "1.0" and the AAR file itself was not reuploaded per Nexus browse listing; treated as a non-event, no content change)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: 1.1.1 is the new stable release (verified 2026-08-16 from maven-metadata.xml and the
    per-version Nexus browse listing showing client-l-1.1.1.aar uploaded Sat Aug 15 07:06:05 UTC 2026,
    171,369 bytes). 1.1.0 (uploaded 2026-07-02, 1,286,574 bytes) sits between 1.0.4 and 1.1.1 in the
    <versions> list and was downloaded + bytecode-inspected (javap) for this cycle: it introduces a
    new `com.rokid.cxr.session` package (coroutine/StateFlow-based CxrSession API) alongside massive
    AAR bloat from bundled native JNI libs (libcaps.so, libcxr-bridge-jni.so, libcxr-sock-proto-jni.so,
    libflora-cli.so, libmutils.so for arm64-v8a/armeabi-v7a) and inlined Caps/CXRServiceBridge/
    CXRSocketProtocol/RLog classes normally supplied via cxr-service-bridge. 1.1.1 removes all of that
    native/inlined bloat (back to depending on cxr-service-bridge normally) while RETAINING the new
    com.rokid.cxr.session package unchanged — appears to be a "1.1.0 was an experimental/broken build,
    1.1.1 is the corrected release" situation. The original com.rokid.cxr.link.CXRLink API is
    byte-identical across 1.0.4/1.1.0/1.1.1 — fully additive, no breaking changes detected.
    <latest> tag is 1.2.X-SNAPSHOT — active 1.2.x development ongoing, no stable cut yet.
    Portal changelog (developerdoc.rokid.com/sdk) still shows 1.0.4 — two minor releases behind Maven.
    client-m: 1.2.2 unchanged since 2026-06-09. Portal still on 1.1.0.
    cxr-service-bridge: 1.0 unchanged (metadata lastUpdated touched 2026-07-28 but no new AAR upload
    per Nexus browse listing — not treated as an actionable change).
    Use Maven as the canonical "what's actually shipped" source. New POM dependencies confirmed for
    client-l 1.1.1 via direct .pom fetch: kotlin-stdlib 1.9.0 (was 1.6.0), cxr-service-bridge
    1.0-20260715.121510-107 (was 1.0-20260522.063600-105), NEW kotlinx-coroutines-android 1.9.0.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-16
  notes: Verified org (id 57519491) via Firecrawl map 2026-08-16. Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-16
  notes: Verified org (id 19773259) via Firecrawl map 2026-08-16. Top results include NextForum, mingutils, RokidMobileSDK{i,A}ndroidDemo (legacy pre-CXR SDK, not Sprite), CloudAppClient, RokidVoiceAI*, glass-docs (old Glass 1/2 era, not Sprite), UXR-docs (out of scope). No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-16
  notes: Verified org (id 25831739) via Firecrawl map 2026-08-16. Map again returned empty links array (0 URLs), consistent with prior checks. No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Discovered 2026-06-28. ar.rokid.com/sdk now renders a new AIUI-focused homepage that
    links to open.rokid.com/sdk and open.rokid.com/sprite as the canonical developer portal.
    open.rokid.com/sprite?lang=zh was scraped 2026-06-28 — content matches developerdoc.rokid.com/sprite
    exactly (same FAQ, same spec table). open.rokid.com/sdk?lang=zh returns only SPA shell.
    Map returned 4 URLs: root, /sdk?lang=zh, /sprite?lang=zh, /academy.
    Re-confirmed 2026-08-16: open.rokid.com/sdk?lang=zh content is now byte-for-byte identical
    to developerdoc.rokid.com/sdk (same SDK comparison table, same CXR-L 1.0.4 / bare-metal 1.0.0
    figures) — still treated as a mirror, not scraped as an independent primary source. Still
    UNREGISTERED pending user approval; no action taken beyond using it as a cross-check.
    /academy hosts the "乐奇学院" (Rokid Academy) learning platform with CXR-L and Glasses
    development courses (in scope) alongside UXR 3.0 (out of scope) and AIUI courses.
    YodaOS-Master content linked from open.rokid.com/master — skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com.
