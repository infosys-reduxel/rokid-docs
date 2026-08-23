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
  last_checked: 2026-08-23
  last_known_version: CXR-L 1.0.4 (portal version table still shows 1.0.4 as of 2026-08-23; Maven release is 1.1.1)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope). Not re-scraped directly this cycle —
    superseded by the developerdoc.rokid.com and open.rokid.com checks below.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-23
  last_known_version: CXR-L 1.0.4 (portal, unchanged since 2026-06-25; Maven release is 1.1.1 as of 2026-08-14), CXR-M 1.1.0 (portal lags Maven 1.2.2, unchanged), 眼镜端裸机开发 1.0.0 (unchanged)
  notes: |
    /sdk re-scraped 2026-08-23 (--only-main-content). Version table unchanged: CXR-L 1.0.4,
    CXR-M 1.1.0, bare-metal 1.0.0. CXR-L card "updated" timestamp still 2026.06.25 — portal has
    NOT been refreshed to reflect Maven's 1.1.1 release. Changelog sections ("更新内容") render
    collapsed/empty in the static scrape (client-side accordion; content not accessible without
    JS interaction) — this is why the CXR-L 1.1.0/1.1.1 API surface was reconstructed via binary
    diff of Maven AARs instead (see cxr-l/release-notes.md v1.1.1 entry). YodaOS-Master tab
    skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-30
  notes: |
    BROKEN as of 2026-06-28. Both previously valid workspace hashes return OSS NoSuchKey:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey 6A408F766EB57F3436AE9E82
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey 6A408F1CC38F5534389093AE
    The OSS bucket (rokid-ar-platform.oss-cn-hangzhou.aliyuncs.com) no longer contains these
    workspace keys. Detail docs appear to have been migrated. New workspace hashes unknown.
    Root path returns HTTP 415. Monitor candidate removed — source is not reachable.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-23
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Not re-verified directly this cycle (redirect behavior unlikely to have changed); low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-23
  last_known_version: |
    client-l 1.1.1 (release; metadata lastUpdated 20260814092031 — new release 1.1.1 found 2026-08-23, confirmed via raw maven-metadata.xml; portal still shows 1.0.4, no official changelog published. Documented in cxr-l/release-notes.md and cxr-l/api-reference.md via binary diff. Intermediate 1.1.0 release (2026-07-02, bundled stray JNI libs) superseded by 1.1.1 — not separately documented.)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged since last cycle)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — lastUpdated timestamp advanced from 20260522063622 but <release> version unchanged at 1.0; likely a metadata-only republish or minor patch within the 1.0 line, no content diff performed since client-l's own cxr-service-bridge dependency pin already advanced to 1.0-20260715.121510-107)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: 1.1.1 is the new release (verified 2026-08-23 via WebFetch of raw maven-metadata.xml
    and confirmed via HTTP HEAD Last-Modified: 2026-08-14). Binary diff of 1.0.4 vs 1.1.1 AARs
    (downloaded with curl, unzipped, javap'd) shows a new com.rokid.cxr.session coroutine-based
    API; the legacy com.rokid.cxr.link.CXRLink API is byte-identical to 1.0.4. Full breakdown
    landed in cxr-l/release-notes.md and cxr-l/api-reference.md this cycle (P1).
    client-m: 1.2.2 unchanged since 2026-06-09. Portal still on 1.1.0.
    cxr-service-bridge: <release> still 1.0 but lastUpdated advanced 2026-07-28; not independently
    actionable (no version-number signal), but its content is folded into client-l's dependency bump.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-23
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). Re-mapped 2026-08-23 — unchanged. No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-23
  notes: Verified org (id 19773259). Official "Rokid" org. Re-mapped 2026-08-23 — same repo list as prior check (NextForum, mingutils, RokidMobileSDK*Demo, CloudAppClient, glass-docs, UXR-docs, mostly Speech/OpenVoice/CloudApp repos). glass-docs still old Glass 1/Glass 2 era content, not Sprite. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-23
  notes: Verified org (id 25831739). Map still returns empty links array (0 URLs) as of 2026-08-23. No actionable content. May have private repos not visible.

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
    Re-checked 2026-08-23: map now returns 10 URLs (root, /sitemap.xml, /sprite?lang=en [NEW —
    English translation now live], /sdk?lang=zh, /sdk?lang=en [NEW], /academy, /?lang=cn
    [AIUI Studio landing — out of scope, different product], /master?lang=en, /master?lang=cn
    [both out of scope], and 3 /detail?appId=... Rokid Store app-listing pages [out of scope —
    consumer app store, not developer docs]. /sprite?lang=zh and /sdk?lang=zh re-scraped
    2026-08-23 — content still matches developerdoc.rokid.com exactly (CXR-L 1.0.4 still shown
    in the version table on both portals; no drift). No action beyond what is already captured
    under the developerdoc.rokid.com / maven.rokid.com entries above.
    /academy hosts the "乐奇学院" (Rokid Academy) learning platform with CXR-L and Glasses
    development courses (in scope) alongside UXR 3.0 (out of scope) and AIUI courses. Not
    scraped in depth this cycle — still pending registration decision.
    YodaOS-Master content linked from open.rokid.com/master — skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com.
