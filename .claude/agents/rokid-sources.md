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
  last_known_version: CXR-L 1.0.3 (portal unchanged; Maven at 1.0.4)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}. 2026-06-28 scrape unchanged vs 2026-06-27. YodaOS-Sprite description continues to read "YodaOS-Sprite runs on Rokid Glasses and Bolon AI Glasses." CXR-L portal version still 1.0.3 (2026-06-02). CXR-M remains access-gated. YodaOS-Master tab present but skipped (out of scope). No new in-scope content detected.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-28
  last_known_version: CXR-L 1.0.3 (2026-06-02 per portal; Maven at 1.0.4 since 2026-06-18)
  notes: |
    Scraped 2026-06-28 via /sdk and /sprite paths. /sdk still shows CXR-L 1.0.3 (2026-06-02); portal has NOT updated to reflect Maven 1.0.4. CXR-M notice retained: "如需获取 CXR-M SDK、文档与技术支持，请联系商务合作：Glasses.BD@rokid.com" — explicitly access-gated. /sprite FAQ unchanged at 8 questions (byte-identical count vs 2026-06-27). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-28
  notes: |
    Live URL scheme: /{workspaceId}/pc/cn/{pageId}.html. Workspace hashes: CXR-M / CXR-S / 眼镜端裸机开发 (shared) = 57e35cd3ae294d16b1b8fc8dcbb1b7c7; CXR-L = 84feb39f8ef141b0ad0326f902ab881f.
    2026-06-28 check: CXR-L workspace HTML shell: window.relatedVersion = "1.0.6" (confirmed on both page IDs — this was already noted in local cxr-l/intro.md as of fetch date 2026-06-20; NOT a new change vs 2026-06-27). The workspace serves multiple documentation version branches: page 9adcfb07939846e5945e79dfbd923f63 defaults to v1.0.1 view; page 595e6de0d5e143739168774d7571dd38 defaults to v1.0.3 view. Sidebar shows 6 chapters: 简介, 快速开始, 开发流程与状态机, 术语与缩写, 功能开发, 版本历史. Intro page content byte-identical to 2026-06-27 cached version. Chapter page IDs for 快速开始, 开发流程与状态机, 术语与缩写, 功能开发 remain unresolvable — SPA JS router navigation is not captured by Firecrawl; CDN JS bundle (static.rokidcdn.com) returns 403. documentId param approach yields 版本1.0.3 container without chapter body content. Four CXR-L structural-gap pages remain P0 missing from local repo.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-28
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). No new scrape this cycle (no changes expected per prior pattern). Last substantive check 2026-06-26 — identical to 2026-06-11. No in-scope AR Glasses / Sprite content. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-28
  last_known_version: |
    client-l 1.0.4 (release; maven-metadata.xml lastUpdated 20260625070819 — UNCHANGED vs 2026-06-27)
    client-l 1.1.X-SNAPSHOT build 1 (snapshot; lastUpdated 20260625070810 — single AAR build dated 2026-06-25T07:08:10Z; UNCHANGED vs 2026-06-27)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — UNCHANGED)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — UNCHANGED)
  notes: |
    2026-06-28 check: All three maven-metadata.xml fetched directly via curl (Maven XML is not behind CDN block). client-l stable release unchanged at 1.0.4 (lastUpdated 20260625070819 — same as 2026-06-27). 1.1.X-SNAPSHOT still at single build 1.1.X-20260625.070810-1 — no new snapshot builds since 2026-06-25. No 1.1.0 stable release cut. client-m unchanged at 1.2.2. cxr-service-bridge unchanged at 1.0. All metadata timestamps identical to yesterday. No action items.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-28
  notes: Verified 2026-06-28 via Firecrawl scrape. Still only 2 public repos: glass2-docs (last push May 7, 2023; Glass 2 — out of scope) and UXR-docs (archived, last push Sep 9, 2021; out of scope). No new repos. No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-28
  notes: Verified 2026-06-28 via Firecrawl scrape. 73 public repos total; page shows top 10 sorted by last updated. Most recently updated in-scope candidate: glass-docs (last push Apr 30, 2023 — still old Glass 1/Glass 2 era). rokid/docs (Speech platform) pushed Mar 28, 2026 — not in scope. rokid/armazpro-module-sdk-sample pushed Mar 13, 2026 — ArmAZ Pro Module SDK, not Sprite/CXR. No in-scope Sprite/AR Glasses content among visible repos. No action items.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-28
  notes: Verified 2026-06-28 via Firecrawl scrape. Still only 2 public repos: BroadcastServiceDemo (pushed Jun 19, 2017) and openCV3_demo (pushed Feb 17, 2017). Both are 9-year-old samples with no Rokid Glasses / Sprite relevance. No actionable content.
