# Rokid Upstream Source Registry

Scout reads this file to know which upstream documentation sources to monitor. Leader and Translator read it for source exchange.

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
  last_checked: 2026-06-27
  last_known_version: CXR-L 1.0.3 (2026-06-02 per portal; Maven at 1.0.4 since 2026-06-25)
  notes: |
    React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh. 2026-06-27 scrape confirmed
    portal still shows CXR-L 1.0.3 (2026-06-02); has not updated to reflect Maven 1.0.4.
    CXR-M section replaced with business-contact gating notice (Glasses.BD@rokid.com).
    YodaOS-Sprite section unchanged (8 FAQ items). YodaOS-Master tab present — out of scope, skipped.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-27
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M access-gated, 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    Scraped 2026-06-27 via /sdk and /sprite paths. /sdk: CXR-L 1.0.3 changelog inline (unchanged vs
    2026-06-11). CXR-M notice added: SDK now access-gated via Glasses.BD@rokid.com, no public version
    shown. /sprite: 8 FAQ questions unchanged. Baremetal section description unchanged.
    SPA with YodaOS-Sprite and YodaOS-Master tabs; Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-27
  notes: |
    Hosts rendered SDK docs as OSS-backed static HTML. 2026-06-27 check:
    Workspace hashes unchanged: CXR-M/CXR-S/裸机开发 = 57e35cd3ae294d16b1b8fc8dcbb1b7c7;
    CXR-L = 84feb39f8ef141b0ad0326f902ab881f.
    CXR-L workspace now has TWO page IDs: 9adcfb07939846e5945e79dfbd923f63 (intro, labeled v1.0.1) and
    NEW 595e6de0d5e143739168774d7571dd38 (SPA shell only — JS-rendered; documentId params: quickStart,
    devFlow/devflow, featureDev/featuredev, glossary, versionHistory). The new page appears to be the
    v1.0.3 rewrite mentioned in the SDK changelog (CXR-S integration sections merged in).
    CXR-M SDK integration page 4e088caa11e84b97b381a145bbb93379 returns NoSuchKey (deleted from OSS).
    CXR-M current pages: 9d9dea4799ca4dd2a1176fedb075b6f2 (intro, v1.0.1), index.html (SDK接入 v0.0.5-SNAPSHOT —
    severely stale). Design-spec page 2786298057084a82b170bf725aef6b5d now shows only nav skeleton (v1.0 placeholder).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-27
  notes: |
    Scrape 2026-06-27 returns the same "AIUI: The Next Frontier" developer homepage as last check
    (developerhomepage asset path 1.0.4 — same static asset version as prior cycle). No new in-scope
    AR Glasses/Sprite content. YodaOS-Sprite and YodaOS-Master both listed as OS sections; Master skipped
    (out of scope). Community counter changed from 33,000+ to 35,000+ developers — cosmetic only.
    Legacy Rokid GitBook (developer.rokid.com/docs/rokid-homebase-docs/) not rescraped; no changes expected.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-27
  last_known_version: |
    client-l 1.0.4 (release; lastUpdated 20260625070819 — NEW since last check 2026-06-11, was 1.0.3)
    client-l 1.1.X-SNAPSHOT (snapshot only; lastUpdated 20260625070819)
    client-m 1.2.2 (release; lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; lastUpdated 20260522063622 — unchanged)
    client (artifact): SNAPSHOT-only, latest 1.2.X-SNAPSHOT; lastUpdated 20260417064920 (no release)
    client-extend (artifact): SNAPSHOT-only, 0.0.2-SNAPSHOT; lastUpdated 20250214072609 (stale, no release)
  notes: |
    Maven map returned 5 artifacts under com/rokid/cxr/: client-l, client-m, cxr-service-bridge,
    client (SNAPSHOT-only pre-release predecessor), client-extend (SNAPSHOT-only, last updated 2025-02-14 — dormant).
    Key finding: client-l 1.0.4 released 2026-06-25 (lastUpdated 20260625070819). This is a NEW release
    vs last check (2026-06-11 showed 1.0.3). The portal (ar.rokid.com/sdk, custom.rokid.com CXR-L intro)
    has NOT updated to show 1.0.4. Also 1.1.X-SNAPSHOT build exists as of 2026-06-25 — active development
    on 1.1.x line. client and client-extend are internal/pre-release artifacts with no stable releases;
    not tracked separately.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-27
  notes: |
    Verified via GitHub API 2026-06-27. Only 2 public repos: UXR-docs (out-of-scope spatial SDK,
    last updated 2026-03-09) and glass2-docs (out-of-scope Glass 2, last updated 2026-02-08).
    No new in-scope repos. No actionable content for this repo.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-27
  notes: |
    Verified via GitHub API 2026-06-27. 50+ public repos. Most recently updated potentially relevant:
    docs (Speech platform, updated_at 2026-06-21 — metadata touch only; last commit 2022-09-15; out of scope),
    RokidMobileSDKAndroidDemo (updated_at 2026-06-04 — metadata touch; last commit 2019-07-10; old pre-CXR
    Rokid Mobile SDK v1.10.x, not CXR; out of scope),
    RokidMobileSDKiOSDemo (updated_at 2026-05-31 — metadata touch; last commit 2019-08-13; out of scope).
    glass-docs (last commit 2020-07-13, old Glass 1/2 era, out of scope).
    No in-scope Sprite/AR Glasses content.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-27
  notes: |
    Verified via GitHub API 2026-06-27. Only 2 public repos: BroadcastServiceDemo (last commit 2017-06-19)
    and openCV3_demo (last commit 2017-02-17). Both are 9-year-old samples with no Rokid Glasses / Sprite
    relevance. No actionable content for this repo.
