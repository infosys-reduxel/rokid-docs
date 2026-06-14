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
  last_checked: 2026-06-14
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing (scraped 2026-06-14) still shows CXR-L 1.0.3 updated 2026-06-02 — unchanged from 2026-06-11. Returns only SPA chrome shell ("Rokid AR Platform") when scraped directly; real content at developerdoc.rokid.com/sprite. ar.rokid.com/master (out-of-scope YodaOS-Master) skipped.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-14
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M 1.1.0 (portal lags Maven; Maven has 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    Scraped 2026-06-14 via /sdk and /sprite paths. /sdk still shows CXR-L 1.0.3 (2026-06-02) — unchanged from 2026-06-11. /sprite FAQ still has same 8 Q&As — no new content. CXR-M portal lag (shows v1.1.0; Maven has 1.2.2) is a known long-standing gap. SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope). Out-of-scope content noted: YodaOS-Master tab present on /sprite and /sdk.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-14
  notes: |
    BLOCKED as of 2026-06-14 — Both workspace hashes returning Aliyun OSS NoSuchKey errors.
    Workspace 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开発): all known page
    URLs return NoSuchKey (9d9dea47..., 4e088caa..., 13083daf... confirmed 404 via OSS scrape).
    Workspace 84feb39f8ef141b0ad0326f902ab881f (CXR-L): page 9adcfb07... also returns NoSuchKey.
    Root URL returns HTTP 415. Last successfully readable content: baremetal/cxr-m pages had
    JS-rendered skeleton only ("版本 X / 文档") on all prior checks; never yielded full article text.
    Content may have migrated — new workspace hashes need discovery via developerdoc.rokid.com
    click-through. Detection status: FAILED / BLOCKED for all workspace doc pages.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-14
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Not re-scraped 2026-06-14 (credit conservation). Prior check (2026-06-11) returned only SVG/logo (heavily JS-rendered). Low coverage overlap with AR-focused scope. No actionable in-scope content found to date.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-14
  last_known_version: |
    client-l 1.0.3 (release; metadata lastUpdated 20260611075349 — unchanged from 2026-06-11)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged from 2026-06-09)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
    com/rokid/cxr/ group: client-l, client-m, cxr-service-bridge (releases); client, client-extend, sdk (SNAPSHOT-only, internal)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browse path: /service/rest/repository/browse/maven-public/com/rokid/cxr/. Maven metadata XML scraped fresh 2026-06-14 — all three artifacts unchanged since 2026-06-11. client-m Maven 1.2.2 still leads the portal (portal shows v1.1.0). client-m version history confirmed: 0.0.8, 1.0.1–1.0.4, 1.0.8–1.0.9, 1.1.0–1.1.1, 1.2.1–1.2.2. Three SNAPSHOT-only artifacts (client, client-extend, sdk) — no releases, internal/pre-release only. Local docs aligned on all three release versions.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-14
  notes: Verified 2026-06-14 via Firecrawl scrape. Still only 2 public repos — glass2-docs (last updated May 7, 2023, JavaScript, out-of-scope Glass 2) and UXR-docs (archived Sep 9, 2021, out-of-scope UXR SDK). No new repos. No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-14
  notes: Verified 2026-06-14 via Firecrawl scrape. 73 public repos. Most recently updated in-scope candidates: docs (Speech/HomeBase, HTML, updated Mar 28 2026 — not AR scope), armazpro-module-sdk-sample (Mar 13 2026 — ArmAzPro scope, not Sprite). glass-docs still present (old Glass 1/Glass 2 era, last commit 2020-07-13). No new Sprite/CXR repos detected. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-14
  notes: Verified 2026-06-14 via Firecrawl scrape. 2 public repos — BroadcastServiceDemo (updated Jun 19, 2017) and openCV3_demo (updated Feb 17, 2017). Both are decade-old Java samples with no Rokid Glasses/Sprite relevance. No new repos. No action items. May have private repos not visible.
