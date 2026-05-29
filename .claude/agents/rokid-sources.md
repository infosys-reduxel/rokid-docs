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
  last_checked: 2026-05-29
  last_known_version: CXR-L 1.0.1 (2026-05-07 per landing page; Maven has 1.0.2)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing shows three sub-tabs (CXR-L / CXR-M / 眼镜端裸机开发) under YodaOS-Sprite top-tab. Landing-page versions lag Maven by 1 minor.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-05-29
  last_known_version: CXR-L 1.0.1 (2026-05-07), CXR-M 1.1.0 (2026-04-01), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: The actual SDK landing page (iframe target of ar.rokid.com/sdk). Tabs scraped 2026-05-29 — CXR-L unchanged from 2026-05-28 capture. CTA "查看文档" buttons resolve to custom.rokid.com/prod/rokid_web/<workspaceHash>/pc/cn/<pageHash>.html. SPA — requires firecrawl-interact for tab clicks. Sister surface developerdoc.rokid.com/sprite renders YodaOS-Sprite open API/SDK index.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-05-29
  notes: |
    Hosts the actual rendered SDK docs as JS-rendered single-page apps with per-doc left-nav. Workspace hashes:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机开发 (shared)
      - page 13083daf77dd40bf84cf5c59711e987a = Rokid Glasses 裸机开发指南 (landing)
        siblings: 按键开发说明, 录音开发说明
    - 84feb39f8ef141b0ad0326f902ab881f = CXR-L
      - page 9adcfb07939846e5945e79dfbd923f63 = CXR-L SDK 简介 (landing)
        nav: 简介 / 快速开始 / 开发流程与状态机 / 术语与缩写 / 功能开发 / 版本历史
    Pages are JS-rendered — `firecrawl scrape --only-main-content` returns only "版本X.Y.Z\n文档". Must use firecrawl-interact for content extraction (expensive). Good monitor candidate to detect doc edits.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-05-28
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Map returned only 1 URL — site is heavily JS-rendered and /sitemap.xml returns 301. Real surface is developer.rokid.com/docs/rokid-homebase-docs/v2/. Low coverage overlap with this repo's AR-focused scope; keep for periodic checks via monitor.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-05-29
  last_known_version: |
    client-l 1.0.2 (release; aar 2026-05-19, metadata 2026-05-28)
    client-m 1.2.1 (release; metadata 2026-04-17)
    cxr-service-bridge 1.0 (release; metadata 2026-05-22; uses snapshot fingerprint 1.0-20260212.103714-88)
    com/rokid/cxr/ group also contains: client, client-extend, sdk (unexamined — flag as discovery items)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/ (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/). maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>. Per-artifact dirs list all versions and their .aar/.pom upload times. **Maven versions lead the developerdoc.rokid.com SDK landing page by ~1 minor** as of 2026-05-29 (client-l Maven 1.0.2 vs landing 1.0.1; client-m Maven 1.2.1 vs landing 1.1.0). Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-05-28
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Blog → https://rokidglass.github.io/glass2-docs/. Likely older Glass 2 docs — overlap with current AR Glasses TBD.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-05-28
  notes: Verified org (id 19773259). Official "Rokid" org, located ZheJiang, blog points to developer.rokid.com. Mostly Speech/OpenVoice repos — limited overlap with AR Glasses scope.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-05-28
  notes: Verified org (id 25831739). Only 2 public repos — most likely the AR Glasses sample-app / SDK demo home. High signal-to-noise for this repo.
