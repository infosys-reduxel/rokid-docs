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
  last_checked: 2026-06-26
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page; portal not yet updated to Maven 1.0.4)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}. Scraped 2026-06-26 — landing shows YodaOS-Sprite section (in-scope) and YodaOS-Master section (skipped). YodaOS-Sprite description now explicitly reads "YodaOS-Sprite runs on Rokid Glasses and Bolon AI Glasses." — Bolon AI Glasses newly surfaced in upstream landing copy. No new SDK-specific pages detected. YodaOS-Master tab skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-26
  last_known_version: CXR-L 1.0.3 (2026-06-02 per portal; Maven now at 1.0.4 as of 2026-06-18), CXR-M portal gated (商务合作), 眼镜端裸机开发 stable
  notes: |
    Scraped 2026-06-26 via /sdk and /sprite paths. /sdk still shows CXR-L 1.0.3 (2026-06-02); portal has NOT updated to reflect Maven 1.0.4 (released 2026-06-18; metadata 20260625070819). CXR-M notice retained: "如需获取 CXR-M SDK、文档与技术支持，请联系商务合作：Glasses.BD@rokid.com" — explicitly access-gated. /sprite FAQ unchanged at 8 questions vs 2026-06-11 scrape (byte-identical diff). Map returned 3 URLs: developerdoc.rokid.com, /sdk, /sprite (SPA). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-26
  notes: |
    Live URL scheme (confirmed 2026-06-23): /{workspaceId}/pc/cn/{pageId}.html — e.g. /57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/9d9dea4799ca4dd2a1176fedb075b6f2.html. Workspace hashes: CXR-M / CXR-S / 眼镜端裸机开发 (shared) = 57e35cd3ae294d16b1b8fc8dcbb1b7c7; CXR-L = 84feb39f8ef141b0ad0326f902ab881f. 2026-06-26 check: Firecrawl returned HTTP 415 (Unsupported Media Type) for all 3 page-ID URLs tried — content unverifiable this cycle. WebFetch returns 200 with JS skeleton only (SPA pages require browser rendering). Four CXR-L structural-gap pages (快速开始, 开发流程与状态机, 术语与缩写, 功能开发) remain unresolved due to JS rendering requirement.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-26
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Scrape 2026-06-26 returns only SPA shell / marketing content — identical to 2026-06-11 check. No in-scope AR Glasses / Sprite content. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-26
  last_known_version: |
    client-l 1.0.4 (release; maven-metadata.xml lastUpdated 20260625070819 — NEW vs 2026-06-11 baseline of 1.0.3)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged from 2026-06-11)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browse path: https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/. Key change vs 2026-06-11: client-l advanced to 1.0.4 (maven-metadata.xml release=1.0.4, lastUpdated 20260625070819 — June 25, 2026). Portal (developerdoc.rokid.com/sdk) still shows 1.0.3 as of 2026-06-26. Local repo already has v1.0.4 entries in cxr-l/release-notes.md and cxr-l/api-reference.md (from prior binary-diff work on 2026-06-25). client-m still at 1.2.2 (unchanged). cxr-service-bridge still at 1.0 (unchanged). Local cxr-m files already reference 1.2.2.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-26
  notes: Verified 2026-06-26. Only 2 public repos: UXR-docs (out-of-scope, last push Sep 2021) and glass2-docs (out-of-scope Glass 2, last push May 2023). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-26
  notes: Verified 2026-06-26. Official "Rokid" org. Public repos: glass-docs (last commit Apr 2023, old Glass 1/Glass 2 era — not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. No action items.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-26
  notes: Verified 2026-06-26. Only 2 public repos: BroadcastServiceDemo (pushed 2017-06-19) and openCV3_demo (pushed 2017-02-17). Both are 9-year-old samples with no Rokid Glasses / Sprite relevance. No actionable content. May have private repos not visible.
