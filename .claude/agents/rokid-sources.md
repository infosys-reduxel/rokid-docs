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
  last_checked: 2026-06-13
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing (scraped 2026-06-13) shows CXR-L 1.0.3 updated 2026-06-02; unchanged from 2026-06-11 check. ar.rokid.com/master (out-of-scope YodaOS-Master) and ar.rokid.com/sprite (in-scope). YodaOS-Master tab skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-13
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M portal still shows v1.1.0 doc page / 0.0.5-SNAPSHOT dependency (portal lags Maven 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    SDK landing page scraped 2026-06-13. /sdk shows CXR-L 1.0.3 changelog inline — unchanged from 2026-06-11. /sprite FAQ unchanged (all 8 Q&A present). SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope). CXR-M portal still does not publish official changelog beyond v1.1.0. Deep-link "查看文档" buttons route to custom.rokid.com CMS.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-13
  notes: |
    Hosts rendered SDK docs as JS-rendered SPA via Aliyun OSS. URL format: /pc/cn/<pageId>.html for standalone pages, /pc/cn/index.html?documentId=<id> for SPA nav. Workspace statuses as of 2026-06-13:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机开发 (shared)
      - /pc/cn/9d9dea4799ca4dd2a1176fedb075b6f2.html = CXR-M 简介 (v1.0.1) — LIVE, content matches local cxr-m/intro.md
      - /pc/cn/index.html?documentId=4e088caa11e84b97b381a145bbb93379 = CXR-M SDK接入 (v0.0.5-SNAPSHOT) — LIVE but severely stale vs local cxr-m/sdk-integration.md (which has 1.2.2); old bare .html URL for this page returns 404 NoSuchKey
      - /pc/cn/13083daf77dd40bf84cf5c59711e987a.html = 眼镜端裸机开发指南 (v0.0.1) — JS-rendered skeleton only on 2026-06-13 scrape
      - /pc/cn/2786298057084a82b170bf725aef6b5d.html?documentId=d0db23be629d4b09be197bd0dd931298 = 设计规范 (v1.0, placeholder) — JS-rendered skeleton only
    - 84feb39f8ef141b0ad0326f902ab881f = CXR-L
      - /pc/cn/9adcfb07939846e5945e79dfbd923f63.html = CXR-L SDK 简介 — returns JS skeleton only (版本\n文档); content not readable via Firecrawl scrape

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-12
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Scrape 2026-06-12 returns only SVG/logo (heavily JS-rendered). No change from 2026-06-11. Low coverage overlap with AR-focused scope.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-13
  last_known_version: |
    client-l 1.0.3 (release; metadata lastUpdated Thu Jun 11 09:50:16 Z 2026 — unchanged from 2026-06-11 check)
    client-m 1.2.2 (release; metadata lastUpdated Tue Jun 09 03:05:17 Z 2026 — unchanged from 2026-06-11 check)
    cxr-service-bridge 1.0 (release; metadata lastUpdated Tue Jun 02 05:20:12 Z 2026 — unchanged)
    client (artifact): SNAPSHOT-only (0.0.4/0.0.7/0.0.8/1.0.0/1.2.X SNAPSHOTs; no release); metadata lastUpdated Wed Jun 03 02:29:19 Z 2026
    client-extend (artifact): SNAPSHOT-only (0.0.2-SNAPSHOT); metadata lastUpdated Wed Jun 03 02:29:20 Z 2026
    sdk (artifact): SNAPSHOT-only (1.0.0-SNAPSHOT); metadata lastUpdated Wed Jun 03 02:27:15 Z 2026
    com/rokid/cxr/ group confirmed: client, client-extend, client-l, client-m, cxr-service-bridge, sdk (6 artifacts total; client/client-extend/sdk are SNAPSHOT-only, no releases)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browse path: https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/. Three new SNAPSHOT-only artifacts confirmed (client, client-extend, sdk) — all SNAPSHOT-only, no releases, internal/pre-release. No new release versions for any artifact since 2026-06-11. Maven versions still lead portal: client-m Maven 1.2.2 vs portal showing 1.1.0 (no official changelog gap persists). client-m new releases since portal docs: 1.1.1, 1.2.1, 1.2.2 (all reverse-engineered in local cxr-m/release-notes.md).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-13
  notes: Verified org. Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK, archived) and glass2-docs (out-of-scope Glass 2 / older hardware, last commit May 7 2023). No in-scope Sprite/AR Glasses content. No change from 2026-06-11.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-12
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-12 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-13
  notes: Verified org. 2 public repos: BroadcastServiceDemo (last commit Jun 19 2017) and openCV3_demo (last commit Feb 17 2017). Both ancient Java demos with no Rokid Glasses relevance. No change from 2026-06-11. No actionable content.
