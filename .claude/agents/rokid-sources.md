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
  last_checked: 2026-06-30
  last_known_version: CXR-L 1.0.3 (portal still shows 1.0.3 as of 2026-06-28; Maven has 1.1.2 as of 2026-09-04)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope).
    2026-09-04: could not re-verify — this environment's egress proxy rejects the CONNECT
    to ar.rokid.com:443 (org policy), so this cycle's check relied on Maven + GitHub only.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-04
  last_known_version: CXR-L 1.0.3 (2026-06-02, portal), CXR-M 1.1.0 (portal lags Maven 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    SDK landing page scraped 2026-06-28. /sdk still shows CXR-L 1.0.3 changelog; /sprite
    FAQ content matches local sprite-overview.md exactly — no drift detected. Map returns
    4 URLs: /sdk, /sprite, /sitemap.xml, root. YodaOS-Master tab skipped (out of scope).
    Portal may be the iframe-serve layer still; open.rokid.com is the new public-facing surface.
    2026-09-04: root/sdk reachable via plain curl (200) but is a pure client-rendered React
    SPA (`<div id="root"></div>` + JS bundle only) — no server-side content to scrape without
    a JS-rendering fetch tool. Could not confirm whether the portal changelog has caught up
    to Maven client-l 1.1.0/1.1.1/1.1.2 (verified separately via maven-metadata.xml).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-04
  notes: |
    BROKEN as of 2026-06-28 (OSS NoSuchKey on both known workspace hashes). 2026-09-04: no
    longer broken at the transport level — both the root path and the previously-NoSuchKey
    84feb39f8ef141b0ad0326f902ab881f (CXR-L) hash now return HTTP 200 — but content is a
    client-rendered React SPA shell with no server-side markup, so the actual changelog body
    is still not retrievable from this environment. Same limitation as developerdoc.rokid.com
    above. Did not re-check the 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M/CXR-S) hash this cycle.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: |
    As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical
    content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at
    developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content
    surfaced. Low priority.
    2026-09-04: could not re-verify — this environment's egress proxy rejects the CONNECT to
    developer.rokid.com:443 (org policy).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-04
  last_known_version: |
    client-l 1.1.2 (release; AAR Last-Modified 2026-08-28; metadata lastUpdated 20260828083628 —
    jumped from 1.0.4 via 1.1.0 (2026-07-02) and 1.1.1 (2026-08-14); no official Rokid changelog
    published for any of 1.1.0/1.1.1/1.1.2 as of 2026-09-04 — see cxr-l/release-notes.md for the
    binary-diff-derived writeup, actioned this cycle as P1)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged; note
    client-l 1.1.1+ depends on a newer *snapshot* build 1.0-20260715.121510-107, not the 1.0
    release artifact tracked here)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" as an HTML page, but direct artifact/
    metadata file downloads under /repository/maven-public/... work fine — that's how the
    AARs and maven-metadata.xml were fetched this cycle).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: 1.1.2 confirmed 2026-09-04 (release directories 1.1.0/1.1.1/1.1.2 all present;
    maven-metadata.xml <release> = 1.1.2). Downloaded and diffed all four AARs (1.0.4, 1.1.0,
    1.1.1, 1.1.2): v1.1.0 adds a new additive `com.rokid.cxr.session` coroutine/StateFlow API
    (~94 classes) alongside the unchanged classic `CXRLink` API, plus temporarily bundles
    native .so libs and cxr-service-bridge classes directly in the AAR; v1.1.1 reverts that
    packaging (re-declares cxr-service-bridge as a dependency, adds ProGuard); v1.1.2 is
    byte-identical to v1.1.1 at the class level (patch-only). Portal (developerdoc.rokid.com,
    custom.rokid.com) still shows/implies 1.0.3-era content but is unreadable as a JS SPA from
    this environment — could not confirm an official write-up exists. Actioned as P1
    (docs/cxr-l/release-notes.md, docs/cxr-l/api-reference.md, README.md, sprite-overview.md
    updated 2026-09-04; all new content explicitly marked provisional/binary-diff).
    client-m: 1.2.2 unchanged since 2026-06-09. Portal still on 1.1.0.
    cxr-service-bridge: 1.0 release unchanged since 2026-05-22 (see snapshot note above).
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-04
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, last updated 2023-05-07). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-04
  notes: Verified org (id 19773259). Official "Rokid" org, 73 total repos. Two repos updated since last check — armazpro-module-sdk-sample (Aug 2026; Armaz Pro is Master-family hardware, out of scope) and mapi-demo-outer (unrelated legacy demo) — neither is in-scope Sprite/AR Glasses content. glass-docs and UXR-docs unchanged (out of scope / stale). No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-04
  notes: Verified org (id 25831739). Still only 2 inactive repos visible (BroadcastServiceDemo from 2017, openCV3_demo from 2017). No actionable content. May have private repos not visible.

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
    2026-09-04: still unregistered (no user approval on record). Could not attempt a check
    this cycle regardless — this environment's egress proxy rejects the CONNECT to
    open.rokid.com:443 (org policy), both via WebFetch (EGRESS_BLOCKED) and curl (403).
