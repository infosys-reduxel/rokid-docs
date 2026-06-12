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
  last_checked: 2026-06-12
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing (scraped 2026-06-12) shows CXR-L 1.0.3 updated 2026-06-02; map returned ar.rokid.com/master (out-of-scope YodaOS-Master) and ar.rokid.com/sprite (in-scope). ar.rokid.com/sprite returns only SPA wrapper — content at developerdoc.rokid.com/sprite. No change from 2026-06-11.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-12
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M portal still shows v1.1.0 / 0.0.5-SNAPSHOT, 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    SDK landing page scraped 2026-06-12 (fresh, non-cached). /sdk shows CXR-L 1.0.3 changelog inline — unchanged from 2026-06-11. /sprite FAQ unchanged from 2026-06-08 (all 8 Q&A present). SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope). Links array empty — all navigation via JS onclick. Deep-link "查看文档" buttons route to custom.rokid.com CMS.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-12
  notes: |
    Hosts rendered SDK docs as JS-rendered SPA via Aliyun OSS. URL format confirmed: /pc/cn/<pageId>.html for standalone pages, /pc/cn/index.html?documentId=<id> for SPA nav. Workspace statuses as of 2026-06-12:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机开发 (shared)
      - /pc/cn/9d9dea4799ca4dd2a1176fedb075b6f2.html = CXR-M 简介 (v1.0.1) — LIVE, content matches local cxr-m/intro.md
      - /pc/cn/index.html?documentId=4e088caa11e84b97b381a145bbb93379 = CXR-M SDK接入 (v0.0.5-SNAPSHOT) — LIVE but severely stale vs local cxr-m/sdk-integration.md (which has 1.2.2)
      - /pc/cn/13083daf77dd40bf84cf5c59711e987a.html = 眼镜端裸机开发指南 (v0.0.1) — LIVE, content matches local cxr-baremetal/development-guide.md
      - /pc/cn/2786298057084a82b170bf725aef6b5d.html?documentId=d0db23be629d4b09be197bd0dd931298 = 设计规范 (v1.0, placeholder) — LIVE
      - 4e088caa11e84b97b381a145bbb93379.html (bare ID without /pc/cn/) = NoSuchKey — the bare-ID URL format is dead; use /pc/cn/index.html?documentId= instead
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
  last_checked: 2026-06-12
  last_known_version: |
    client-l 1.0.3 (release; metadata lastUpdated 20260601112841 — UNCHANGED from 2026-06-11)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — UNCHANGED from 2026-06-11)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — UNCHANGED from 2026-06-11)
    Artifact group com/rokid/cxr: client, client-extend, client-l, client-m, cxr-service-bridge, sdk (unchanged)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browsed via /service/rest/repository/browse/maven-public/com/rokid/cxr/. All three tracked artifacts unchanged since last check. Maven client-m 1.2.2 still leads the portal (which shows 0.0.5-SNAPSHOT). No new artifact IDs in the group. client-m new releases since portal docs: 1.1.1, 1.2.1, 1.2.2 (all reverse-engineered in local cxr-m/release-notes.md).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-12
  notes: Verified org (id 57519491). Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). GitHub API returned no output on 2026-06-12 (likely API rate limit without auth token). Last confirmed 2026-06-11 — no in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-12
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-11 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority. GitHub API unavailable without auth 2026-06-12.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-12
  notes: Verified org (id 25831739). Only 2 public repos: BroadcastServiceDemo (last commit 2017-06-19 — a 9-year-old OpenCV sample with no Rokid Glasses relevance). GitHub API returned no output 2026-06-12 (likely rate limit without auth). Last confirmed 2026-06-11 — no actionable content. May have private repos not visible.
