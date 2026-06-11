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
  last_checked: 2026-06-11
  last_known_version: CXR-L 1.0.3 (2026-06-02 per landing page and Maven)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing shows three sub-tabs (CXR-L / CXR-M / 眼镜端裸机开发) under YodaOS-Sprite top-tab. ar.rokid.com itself returns SVG logo data — use developerdoc.rokid.com directly. Landing-page versions now in sync with Maven for CXR-L 1.0.3 as of 2026-06-11.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-11
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M 1.1.0 (2026-04-01; Maven has 1.2.2 undocumented), 眼镜端裸機開發 0.0.1 (2026-03-01)
  notes: |
    The actual SDK landing page (iframe target of ar.rokid.com/sdk). Scraped 2026-06-11 — CXR-L now shows 1.0.3 (updated from 1.0.1 on 2026-05-29 baseline). CXR-M tab still shows 1.1.0 — Rokid has not published 1.2.x changelog. developerdoc.rokid.com/sprite renders YodaOS-Sprite overview with full Q&A including CXR-L 1.0.3 prereqs and bare-metal dev guidance. map returns only 3 URLs (SPA) — scrape /sdk and /sprite directly.
    YodaOS-Master tab present on /sdk — out of scope, skipped.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-11
  notes: |
    Hosts the actual rendered SDK docs as JS-rendered single-page apps with per-doc left-nav. Workspace hashes:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机开发 (shared)
      - page 13083daf77dd40bf84cf5c59711e987a = Rokid Glasses 裸机开发指南 (landing) v0.0.1
        siblings: 按键开发说明, 录音开发说明
      - NEW page 9d9dea4799ca4dd2a1176fedb075b6f2 = unknown (JS-rendered only; returns only "版本/文档" stub)
      - page 2786298057084a82b170bf725aef6b5d?documentId=d0db23be... = 设计规范 (Design spec; nav: 简介/开发环境介绍/SDK接入/功能开发/设计规范; "上一篇: 数据结构" = CXR-S section)
      - index.html?documentId=4e088caa11e84b97b381a145bbb93379 = CXR-M SDK 接入 (v0.0.5 snapshot — old doc base)
    - 84feb39f8ef141b0ad0326f902ab881f = CXR-L
      - page 9adcfb07939846e5945e79dfbd923f63 = CXR-L SDK 简介 (landing); JS-rendered stub only
    Pages are JS-rendered — `firecrawl scrape --only-main-content` returns only "版本X.Y.Z\n文档" or nav + content (inconsistent). Use developerdoc.rokid.com/sdk and /sprite for authoritative changelogs. Monitor candidate.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Map returned only 1 URL — site is heavily JS-rendered and /sitemap.xml returns 301. Real surface is developer.rokid.com/docs/rokid-homebase-docs/v2/. Low coverage overlap with this repo's AR-focused scope; keep for periodic checks via monitor.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-11
  last_known_version: |
    client-l 1.0.3 (release; aar 2026-06-02, metadata Tue Jun 02 05:20:12 Z 2026)
    client-m 1.2.2 (release; metadata Tue Jun 09 03:05:17 Z 2026) — UNCHANGED since 2026-06-09
    cxr-service-bridge 1.0 (release; latest snapshot 1.0-20260522.063600-105, metadata 2026-05-22)
    com/rokid/cxr/ group also contains: client, client-extend, sdk (unexamined — flag as discovery items)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/ (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/). maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>. Per-artifact dirs list all versions and their .aar/.pom upload times. Maven and landing page now in sync for CXR-L (both 1.0.3) as of 2026-06-11. client-m Maven still leads portal by ~1 minor (1.2.2 vs documented 1.1.0). cxr-service-bridge snapshot at 1.0-20260522.063600-105 (build 105 of 1.0-SNAPSHOT).

- url: https://github.com/RokidGlass
  kind: github
  covers: hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: |
    Verified org (id 57519491). Only 2 public repos: glass2-docs (Documentation for RokidGlass 2 Project — Glass 2 hardware, OUT OF SCOPE) and UXR-docs (archived, spatial computing, OUT OF SCOPE). No in-scope content found. Map returned Glass2 issue URLs only. Org is low priority for future checks.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: |
    Verified org (id 19773259). Shows 73 public repos. Top repos: rokid/docs (Speech open platform, 104 stars, updated Mar 2026 — Voice/HomeBase scope, not AR Glasses), armazpro-module-sdk-sample. Mostly Speech/OpenVoice repos — limited overlap with AR Glasses scope. No CXR/Sprite content found in visible listings.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-11
  notes: |
    Verified org (id 25831739). Only 2 public repos: openCV3_demo (Java, last commit 9 years ago — 2017) and BroadcastServiceDemo (Java, last commit Jun 2017). Both are stale sample apps with no documentation content relevant to current CXR SDK. Map returned empty URL list. No action items.
