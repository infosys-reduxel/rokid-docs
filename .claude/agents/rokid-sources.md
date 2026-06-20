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
  last_checked: 2026-06-20
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page; portal not yet updated to 1.0.4)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. ar.rokid.com/sdk returns only SPA wrapper "Rokid AR Platform" (no readable content). ar.rokid.com/sprite returns full content (107 lines Markdown). Map returned 48 URLs: ar.rokid.com/master (out-of-scope YodaOS-Master, skipped), ar.rokid.com/sprite (in-scope), ar.rokid.com/sdk, ar.rokid.com/arStore, ar.rokid.com/control, and many /detail?appId= entries (app store, not doc content). Checked 2026-06-20 — no new in-scope URLs discovered vs last check.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-20
  last_known_version: CXR-L 1.0.3 (2026-06-02; portal not yet updated to Maven 1.0.4), CXR-M 1.1.0 (portal lags Maven; Maven has 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    SDK landing page (iframe target of ar.rokid.com/sdk) scraped 2026-06-20 via /sdk and /sprite paths. /sdk (developerdoc-sdk-current.md) shows CXR-L 1.0.3 changelog inline; portal has not updated to 1.0.4 despite Maven shipping it 2026-06-18. /sprite FAQ (107 lines) still references client-l:1.0.3 in Q4 — matches local sprite-overview.md content from 2026-06-08 scrape exactly. SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope). Map returned only 1 URL (developerdoc.rokid.com root — heavy SPA, not crawlable by URL map).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-20
  notes: |
    Hosts the actual rendered SDK docs as JS-rendered single-page apps with per-doc left-nav. Workspace hashes:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机開発 (shared)
      - page 13083daf77dd40bf84cf5c59711e987a = Rokid Glasses 裸机开发指南 (landing; version 0.0.1)
        siblings: 按键开发说明, 录音开发说明
      - page 9d9dea4799ca4dd2a1176fedb075b6f2 = CXR-M 简介 (version 1.0.1 portal label; scraped 2026-06-11; skeleton-only 2026-06-18)
      - documentId=4e088caa11e84b97b381a145bbb93379 = CXR-M SDK接入 (WAS: version 0.0.5-SNAPSHOT; REMOVED 2026-06-18 — Aliyun OSS returns NoSuchKey; page removed or migrated)
      - page 2786298057084a82b170bf725aef6b5d = 设计规范 (version 1.0 placeholder; no useful content)
    - 84feb39f8ef141b0ad0326f902ab881f = CXR-L
      - page 9adcfb07939846e5945e79dfbd923f63 = CXR-L SDK 简介 (简介/intro; FULL CONTENT accessible via .html URL — 80+ lines; capabilities table, prerequisite table, availability matrix, sample refs)
        nav: 简介 / 快速开始 / 开发流程与状态机 / 术语与缩写 / 功能开发 / 版本历史
        static .html URL: https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/cn/9adcfb07939846e5945e79dfbd923f63.html
        page IDs for nav siblings (快速开始 etc.) not yet resolved — run map/crawl from .html root to discover
    URL FORMAT: query-parameter form (?workspaceId=…&pageId=…) returns HTTP 415. Use static .html paths: /prod/rokid_web/{workspaceId}/pc/cn/{pageId}.html → HTTP 200 with full content.
    New workspace 323825adf4914c21be8a0d5fe7b8a9e5 appeared in map — confirmed nav/landing scaffold only (links to ar.rokid.com, developerdoc), not an SDK doc area.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-20
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Scrape 2026-06-20 returns SVG/logo only (heavily JS-rendered). Map returned only 1 URL. Real surface is developer.rokid.com/docs/rokid-homebase-docs/v2/. Low coverage overlap with this repo's AR-focused scope. No in-scope content detected; keep for periodic checks.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-20
  last_known_version: |
    client-l 1.0.4 (release; metadata lastUpdated 20260618075010 — NEW vs 2026-06-11 check which showed 1.0.3)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
    client (legacy/internal): only SNAPSHOT versions (latest 1.2.X-SNAPSHOT) — no release; not actionable
    client-extend (internal): only 0.0.2-SNAPSHOT — no release; not actionable
    sdk (internal): only 1.0.0-SNAPSHOT — no release; not actionable
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browse path: https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/. As of 2026-06-20 the /com/rokid/cxr/ index now lists SIX artifacts: client, client-extend, client-l, client-m, cxr-service-bridge, sdk. The three new vs. 2026-06-11 registry (client, client-extend, sdk) contain only SNAPSHOT/pre-release versions and are not publicly documented — likely internal build artifacts. KEY CHANGE: client-l jumped from 1.0.3 to 1.0.4 between 2026-06-11 and 2026-06-18 — local cxr-l/ files updated to reflect 1.0.4. client-m Maven still 1.2.2, portal still 1.1.0 (known lag, already noted in release-notes.md). Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-20
  notes: Verified org. Public repos: UXR-docs (out-of-scope spatial computing SDK, archived), glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items. Checked 2026-06-20 — no new repos vs 2026-06-11.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-20
  notes: Verified org. Pinned repos: community (issue tracker), docs (Rokid voice platform / Speech — not AR Glasses). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority. Checked 2026-06-20 — no new repos vs 2026-06-11.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-20
  notes: Verified org. Only 2 public repos: openCV3_demo and BroadcastServiceDemo (both old OpenCV/broadcast Java samples with no Rokid Glasses relevance). No actionable content. Map returned 0 URLs (empty org page for Firecrawl). Checked 2026-06-20 — no new repos vs 2026-06-11.
