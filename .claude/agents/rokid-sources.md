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
  last_checked: 2026-09-07
  last_known_version: CXR-L 1.0.3 (portal still shows 1.0.3 as of 2026-06-28; Maven has 1.1.2)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope). Root now 301-redirects; not
    re-scraped in full on 2026-09-07 (developerdoc.rokid.com used instead, same content).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-07
  last_known_version: CXR-L 1.0.4 (portal, unchanged since 2026-06-25; Maven now at 1.1.2), CXR-M 1.1.0 (portal lags Maven 1.2.2, unchanged), 眼镜端裸机开发 1.0.0 (portal now shows 1.0.0, previously 0.0.1)
  notes: |
    /sdk re-scraped 2026-09-07: version-comparison table now shows CXR-L "1.0.4" (was
    "1.0.3" tile label, changelog body already covered 1.0.4), CXR-M "1.1.0" (unchanged),
    bare-metal "1.0.0" (was "0.0.1" as of 2026-03-01 — worth a follow-up P2 content-drift
    check on cxr-baremetal/ next cycle; not actioned this run since the "更新内容" detail
    panels render empty in a static scrape — no prose available to diff/translate).
    /sprite re-scraped 2026-09-07: hardware spec table (dimensions, IPX4, AR1, Wi-Fi 6,
    BT 5.3, 210 mAh, IMX681, FOV 30°, 1500 nits, 480x640) byte-for-byte matches
    yodaos/docs/sprite-overview.md — no drift. FAQ Q4 pin (client-l version number) was
    stale against Maven; corrected in this cycle. Map returns 4 URLs: /sdk, /sprite,
    /sdk?lang=en, root. YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-07
  notes: |
    Root path now returns HTTP 200 (was 415 as of 2026-06-28) but Firecrawl scrape still
    fails on all engines ("All scraping engines failed to retrieve content from this URL")
    — likely an API/JSON endpoint rather than a real HTML page, or still gated behind a
    workspace-hash path this registry doesn't have. Not usable as a source this cycle.
    Previously valid workspace hashes (57e35cd3ae294d16b1b8fc8dcbb1b7c7,
    84feb39f8ef141b0ad0326f902ab881f) not re-tested. Monitor candidate remains removed.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority. Not re-checked 2026-09-07 (egress proxy blocked direct connection this run; low priority, deferred).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-07
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20260828083628 — new releases 1.1.0 (2026-07-02),
    1.1.1 (2026-08-14), 1.1.2 (2026-08-28) found 2026-09-07, superseding the previously-recorded
    1.0.4; latest tag is 1.2.X-SNAPSHOT; portal still shows 1.0.4, official changelog not published)
    client-m 1.2.2 (release; metadata lastUpdated 20260902061433 — version unchanged since last
    check, though the artifact was rebuilt/republished 2026-09-02)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — version unchanged,
    but a new build 1.0-20260715.121510-107 was published and is now client-l 1.1.x's dependency)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: verified 2026-09-07 via maven-metadata.xml and per-version AAR/POM downloads.
    Binary-diffed 1.0.4 -> 1.1.0 -> 1.1.1 -> 1.1.2: v1.1.x adds a new com.rokid.cxr.session
    Kotlin coroutine API (CxrSessionManager/CxrSession) layered on the existing CXRLink;
    v1.1.0 anomalously bundled CXR-M/CXR-S native-bridge classes and .so libs, reverted in
    v1.1.1; v1.1.2 is a build-metadata-only republish of v1.1.1. Actioned this cycle — see
    cxr-l/release-notes.md and cxr-l/api-reference.md. Portal still shows 1.0.4; no official
    changelog published for any 1.1.x release.
    client-m: 1.2.2 unchanged since 2026-06-09 (rebuilt 2026-09-02, same version number, no
    action needed). Portal still on 1.1.0.
    cxr-service-bridge: version stays 1.0; latest build timestamp is now consumed by client-l
    1.1.x's POM. No standalone action needed.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-07
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-07
  notes: Verified org (id 19773259). Official "Rokid" org. Re-checked 2026-09-07 — same repo set as 2026-06-29 (glass-docs, UXR-docs, Speech/OpenVoice/CloudApp repos, native-system-docs, mobile-sdk-ios-docs, mobile-sdk-android-docs, etc). No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-07
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) again on 2026-09-07 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible.

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
    /academy hosts the "乐奇学院" (Rokid Academy) learning platform with CXR-L and Glasses
    development courses (in scope) alongside UXR 3.0 (out of scope) and AIUI courses.
    YodaOS-Master content linked from open.rokid.com/master — skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com.
    Re-checked 2026-09-07 (map only, not scraped further pending approval): root now serves
    the AIUI Studio agent-builder homepage rather than an /sdk link; /sprite?lang=en,
    /master (both cn/en), and /academy still present. Still unregistered — no action taken,
    per no-silent-extension policy.
