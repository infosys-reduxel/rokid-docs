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
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing (scraped 2026-06-14) still shows CXR-L 1.0.3 updated 2026-06-02 — unchanged from 2026-06-11. Returns only SPA chrome shell ("Rokid AR Platform") when scraped directly; real content at developerdoc.rokid.com/sprite.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-14
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M 1.1.0 (portal lags Maven; Maven has 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    Scraped 2026-06-14 via /sdk and /sprite paths. /sdk still shows CXR-L 1.0.3 (2026-06-02) — unchanged from 2026-06-11. /sprite FAQ still has same 8 Q&As as 2026-06-08 refresh — no new content. CXR-M portal lag (shows v1.1.0; Maven has 1.2.2) is a known long-standing gap. SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope). Out-of-scope content noted: YodaOS-Master tab present on /sprite and /sdk.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-14
  notes: |
    BLOCKED — Both workspace hashes now returning Aliyun OSS NoSuchKey errors as of 2026-06-14.
    Workspace 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): all page URLs
    return NoSuchKey (pages 9d9dea47..., 4e088caa..., 13083daf... all confirmed 404 via OSS).
    Workspace 84feb39f8ef141b0ad0326f902ab881f (CXR-L): page 9adcfb07... also returns NoSuchKey.
    Root URL https://custom.rokid.com/prod/rokid_web/ returns HTTP 415. Last confirmed readable:
    baremetal-landing-live.md (date unknown, earlier cycle). Detection status: FAILED / BLOCKED.
    If content migrated, new workspace hashes need to be discovered via developerdoc.rokid.com click-through.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-14
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Not re-scraped this cycle (credit conservation; 83 credits remaining). Prior check (2026-06-11) returned only SVG/logo. Low coverage overlap with this repo's AR-focused scope. No actionable in-scope content found to date.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-14
  last_known_version: |
    client-l 1.0.3 (release; metadata lastUpdated 20260611075349 — unchanged from 2026-06-11)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged from 2026-06-11)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Maven metadata checked 2026-06-14 via direct XML fetch. All three release artifacts unchanged since 2026-06-11: client-l stable at 1.0.3, client-m stable at 1.2.2, cxr-service-bridge stable at 1.0. No new releases. com/rokid/cxr/ group also has client, client-extend, sdk (SNAPSHOT-only, internal). Local docs are aligned: cxr-l/release-notes.md and api-reference.md pin 1.0.3; cxr-m/sdk-integration.md pins 1.2.2; cxr-s/sdk-import.md pins 1.0-20260522.063600-105. No version drift detected.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-14
  notes: Verified 2026-06-14 via Firecrawl scrape. Still only 2 public repos: glass2-docs (last updated May 7, 2023 — JavaScript, out-of-scope Glass 2) and UXR-docs (archived, Sep 9, 2021 — out-of-scope UXR SDK). No new repos. No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-14
  notes: Scraped 2026-06-14. 73 public repos. Most recently updated in-scope candidates: docs (Speech/HomeBase, HTML, updated Mar 28 2026 — not AR scope), armazpro-module-sdk-sample (Mar 13 2026 — unclear scope; "Module SDK" for ArmAzPro, not Sprite). glass-docs visible (old Glass 1/Glass 2 era). No new Sprite/CXR repos detected. No action items.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-14
  notes: Verified 2026-06-14 via Firecrawl scrape. Still only 2 public repos: BroadcastServiceDemo (updated Jun 19, 2017) and openCV3_demo (updated Feb 17, 2017). Both are decade-old samples with no Rokid Glasses/Sprite relevance. No new repos. No action items.
