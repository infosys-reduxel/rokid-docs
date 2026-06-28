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
  last_checked: 2026-06-28
  last_known_version: CXR-L 1.0.3 (portal still shows 1.0.3 as of 2026-06-28; Maven has 1.0.4)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-28
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M 1.1.0 (portal lags Maven 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    SDK landing page scraped 2026-06-28. /sdk still shows CXR-L 1.0.3 changelog; /sprite
    FAQ content matches local sprite-overview.md exactly — no drift detected. Map returns
    4 URLs: /sdk, /sprite, /sitemap.xml, root. YodaOS-Master tab skipped (out of scope).
    Portal may be the iframe-serve layer still; open.rokid.com is the new public-facing surface.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-28
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
  last_checked: 2026-06-11
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Scrape 2026-06-11 returns only SVG/logo (heavily JS-rendered). Map returned only 1 URL. Real surface is developer.rokid.com/docs/rokid-homebase-docs/v2/. Low coverage overlap with this repo's AR-focused scope; keep for periodic checks via monitor.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-28
  last_known_version: |
    client-l 1.0.4 (release; metadata lastUpdated 20260625070819 — new release 1.0.4 found 2026-06-28; portal still shows 1.0.3, official changelog not yet published)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: 1.0.4 is the new release (verified 2026-06-28 from maven-metadata.xml
    lastUpdated 20260625070819). Portal still shows 1.0.3 changelog. Official changelog pending.
    client-m: 1.2.2 unchanged since 2026-06-09. Portal still on 1.1.0.
    cxr-service-bridge: 1.0 unchanged since 2026-05-22.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-11 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-11
  notes: Verified org (id 25831739). Only 2 public repos: BroadcastServiceDemo (last commit 2017-06-19 — a 9-year-old OpenCV sample with no Rokid Glasses relevance) and repo listing showed no new repos as of 2026-06-11. No actionable content for this repo. May have private repos not visible.

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
