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
  last_checked: 2026-06-25
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page; portal not yet updated to Maven 1.0.4)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh. Scraped 2026-06-23 — /sprite returns full YodaOS-Sprite FAQ (8 Q&As) and hardware spec table matching local sprite-overview.md. No structural drift from local copy. New nav items discovered: "高校挑战赛" (university challenge, out-of-scope marketing page) and "灵珠" (rizon.rokid.com — AI platform for Rokid Glasses, potentially in-scope, not yet in registry). YodaOS-Master tab skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-25
  last_known_version: CXR-L 1.0.3 (2026-06-02 per portal; Maven now at 1.0.4 as of 2026-06-18), CXR-M portal gated (商务合作), 眼镜端裸机开发 stable
  notes: |
    Scraped 2026-06-22 via /sdk and /sprite paths. /sdk still shows CXR-L 1.0.3 (2026-06-02); portal does NOT yet reflect Maven 1.0.4 (released 2026-06-18). CXR-M notice changed: portal now prominently shows "如需获取 CXR-M SDK、文档与技术支持，请联系商务合作：Glasses.BD@rokid.com" — explicitly gated, no longer a standard public doc page. /sprite FAQ unchanged at 8 questions, same headings as 2026-06-08 local copy; no structural drift detected. Map returned 3 URLs: developerdoc.rokid.com, /sdk, /sprite. YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-25
  notes: |
    Scraped 2026-06-23 via direct .html URLs (restored access). Map returned 6 CXR-M/S workspace URLs (57e35cd3ae294d16b1b8fc8dcbb1b7c7) and 1 CXR-L URL (84feb39f8ef141b0ad0326f902ab881f). New workspace 323825adf4914c21be8a0d5fe7b8a9e5 discovered — contains only marketing/challenge pages and UXR2.0 content (all out of scope). Content verified: CXR-M intro (9d9dea4799ca4dd2a1176fedb075b6f2.html) = v1.0.1 matches local cxr-m/intro.md; CXR-S intro (2786298057084a82b170bf725aef6b5d.html) = v1.0 matches local cxr-s/brief.md; CXR-L intro (84feb39f8ef141b0ad0326f902ab881f) = v1.0.1 skeleton (JS-rendered, old URL format now returns NoSuchKey — content at new /pc/cn/ path). Note: ?documentId= sub-pages remain JS-rendered skeletons. Workspace hashes: CXR-L = 84feb39f8ef141b0ad0326f902ab881f (reassigned to CXR-S content); CXR-M / CXR-S / 眼镜端裸机开发 (shared) = 57e35cd3ae294d16b1b8fc8dcbb1b7c7. CXR-L portal entry point is now solely developerdoc.rokid.com/sdk (still shows 1.0.3 as of 2026-06-25).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-25
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Scrape 2026-06-22 returns only SVG/logo (heavily JS-rendered). Map returned only 1 URL. No in-scope AR Glasses / Sprite content. Monitoring continues as low-priority sanity check.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-25
  last_known_version: |
    client-l 1.0.4 (release; maven-metadata.xml lastUpdated 20260618075010 — NEW since 2026-06-11 check, was 1.0.3)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged from 2026-06-11 check)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browse path: https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/. Key change vs 2026-06-11: client-l advanced to 1.0.4 (uploaded 2026-06-18; maven-metadata lastUpdated 20260618075010). Portal (developerdoc.rokid.com/sdk) still shows 1.0.3 as of 2026-06-22 — portal lags Maven by 1 minor. Re-verified 2026-06-23: client-l browse shows 1.0.4 as newest entry; metadata lastUpdated unchanged at 20260618075010 — no new release since 2026-06-18. client-m still at 1.2.2 (metadata unchanged; portal still shows 1.1.0 — major lag). cxr-service-bridge still at 1.0 (metadata 20260522063622 — unchanged).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-24
  notes: Verified org (id 57519491). Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK, archived) and glass2-docs (out-of-scope Glass 2, last updated May 2023). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-24
  notes: Verified org (id 19773259). Official "Rokid" org. 73 public repos; checked 2026-06-22 — no new in-scope repos. Most recently updated in-scope candidates: glass-docs (last commit Apr 2023, old Glass 1/Glass 2 era). docs repo (last commit Mar 2026) covers Speech/HomeBase platform — not AR Glasses / Sprite. No actionable content for this repo.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-24
  notes: Verified org (id 25831739). Now shows 2 public repos (previously 1 confirmed): BroadcastServiceDemo (last commit Feb 2017) and openCV3_demo (last commit Feb 2017). Both are 9-year-old OpenCV samples with no Rokid Glasses / Sprite relevance. No actionable content.
