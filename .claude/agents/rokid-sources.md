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
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing (scraped 2026-06-11) shows CXR-L 1.0.3 updated 2026-06-02; map returned ar.rokid.com/master (out-of-scope YodaOS-Master) and ar.rokid.com/sprite (in-scope). ar.rokid.com/sprite returns only SPA wrapper — content at developerdoc.rokid.com/sprite.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-11
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M 1.1.0 (portal lags Maven; Maven has 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    SDK landing page (iframe target of ar.rokid.com/sdk) scraped 2026-06-11 via /sdk and /sprite paths. /sdk shows CXR-L 1.0.3 changelog inline; CXR-M portal still shows v1.1.0 doc page with 0.0.5-SNAPSHOT dependency example (severely lags Maven 1.2.2). /sprite FAQ matched local sprite-overview.md (fetched 2026-06-08) exactly — no drift detected. SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-11
  notes: |
    Hosts the actual rendered SDK docs as JS-rendered single-page apps with per-doc left-nav. Workspace hashes:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机开发 (shared)
      - page 13083daf77dd40bf84cf5c59711e987a = Rokid Glasses 裸机开发指南 (landing; version 0.0.1)
        siblings: 按键开发说明, 录音开发说明
      - page 9d9dea4799ca4dd2a1176fedb075b6f2 = CXR-M 简介 (version 1.0.1 portal label; scraped 2026-06-11)
      - documentId=4e088caa11e84b97b381a145bbb93379 = CXR-M SDK接入 (version 0.0.5-SNAPSHOT; severely stale)
      - page 2786298057084a82b170bf725aef6b5d = 设计规范 (version 1.0 placeholder; no useful content)
    - 84feb39f8ef141b0ad0326f902ab881f = CXR-L
      - page 9adcfb07939846e5945e79dfbd923f63 = CXR-L SDK 简介 (landing; JS-rendered — returns only "版本\n文档" skeleton)
        nav: 简介 / 快速开始 / 开发流程与状态机 / 术语与缩写 / 功能开发 / 版本历史
    Pages are JS-rendered — firecrawl scrape --only-main-content returns only skeleton for CXR-L intro page. Bare-metal and CXR-M intro pages do render content. Good monitor candidate.

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
  last_checked: 2026-06-11
  last_known_version: |
    client-l 1.0.3 (release; metadata lastUpdated 20260601112841 — unchanged from 2026-06-09 check)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — 1 new version 1.2.2 since last check 2026-06-09; was 1.2.2 already; no change)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
    com/rokid/cxr/ group confirmed: only client-l, client-m, cxr-service-bridge (map returned these 3; earlier notes mentioning client, client-extend, sdk unconfirmed — not visible in browse)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/ (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/). maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>. Maven versions lead the developerdoc.rokid.com SDK landing page by multiple minors: client-l Maven 1.0.3 vs portal showing 1.0.3 (now aligned); client-m Maven 1.2.2 vs portal still showing 1.1.0. Use Maven as the canonical "what's actually shipped" source.

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
