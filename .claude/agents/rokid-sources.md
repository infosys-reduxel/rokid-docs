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
  last_checked: 2026-07-12
  last_known_version: CXR-L 1.0.4 (portal-published, 2026-06-29); Maven now has 1.1.0 (see maven.rokid.com entry)
  notes: |
    Verified live 2026-07-12. Root/`/all`/`/detail?appId=` map now resolves to the Rokid
    Station App Store (TV/Station app listings — NewTV, QQ Music, VLC, etc.) — this is
    OUT OF SCOPE hardware (Rokid Station family / Master ecosystem), not Sprite/CXR doc
    content. Skipped, not actionable.
    `/doc` and `/sprite` both render the new AIUI-focused developer homepage (English),
    with "Docs" links pointing to `open.rokid.com/sdk?lang=en` and `open.rokid.com/sprite?lang=en`.
    YodaOS-Master card/link present on this homepage — skipped (out of scope), Master
    docs at `open.rokid.com/master`.
    Confirms ar.rokid.com is no longer the doc-content host; open.rokid.com is (see
    unregistered-source note below). No in-scope content changed here since last check.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-12
  last_known_version: CXR-L 1.0.4 (portal-published changelog, updated 2026.06.29); CXR-M section not re-verified this run (tab content is JS-hidden, see notes)
  notes: |
    Verified live 2026-07-12. Map still returns the same 3 URLs (/sdk, /sprite, root).
    /sdk (CXR-L tab, default-active) scraped successfully: version 1.0.4, updated
    2026-06-29, changelog now includes an OFFICIAL entry for v1.0.4 describing
    setGlassBrightness/setGlassVolume and GlassInfo.brightness/sound — this content
    was previously only in our repo as a "provisional, reconstructed from binary diff"
    note (see cxr-l/api-reference.md). Official confirmation is now live upstream.
    CXR-M and 眼镜端裸机开发 tabs are separate client-side tabs not captured by a
    single scrape (raw HTML has no server-rendered text — full React SPA); did not
    re-verify their content this run, no fresh evidence either way.
    /sprite FAQ + spec table re-scraped and diffed field-by-field against
    yodaos/docs/sprite-overview.md (dimensions, IPX4, Wi-Fi 6, BT 5.3, 32GB ROM,
    210mAh, IMX681 camera specs, 1500 nits) — EXACT MATCH, no drift.
    YodaOS-Master tab still present on /sdk and /sprite — skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    PARTIALLY RECOVERED as of 2026-07-12, still not usable as a content source. `firecrawl map`
    now returns 16 real workspace URLs (both known hashes 57e35cd3ae294d16b1b8fc8dcbb1b7c7
    and 84feb39f8ef141b0ad0326f902ab881f resolve, plus two previously-unseen hashes
    c88be4bcde4c42c0b8b53409e1fa1701 and 323825adf4914c21be8a0d5fe7b8a9e5), with real
    page titles in the map metadata (e.g. "Rokid Glasses 设备连接与管理" matching
    cxr-m/device-connection.md). This is an improvement over the 2026-06-28 NoSuchKey
    failure. HOWEVER: `firecrawl scrape` (incl. with --wait-for 4000) on individual
    detail pages under both known hashes returns only an empty JS-app shell ("版本" /
    "文档" placeholder text, no body content) — the workspace document body itself
    still does not render via Firecrawl. Treat any content under this host as UNVERIFIED
    this run; do not infer document content from map titles alone.
    Note: hash c88be4bcde4c42c0b8b53409e1fa1701 map entries reference UXR2.0 / plane
    detection / RKCamera content — that is spatial-computing SDK material, OUT OF SCOPE,
    skipped regardless of future accessibility.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    Verified live 2026-07-12 — this is NOT a redirect right now; `firecrawl map` returned
    16 real GitBook URLs under /docs/rokidos-linux-docs/, /docs/rokid-homebase-docs/,
    /docs/2-RokidDocument/, /docs/5-enableVoice/, /docs/8-app/, etc. All content is the
    legacy Rokid smart-speaker / voice-skill platform (若琪 Skills Kit, AVS, HomeBase
    smart-home bridge, old rokidos-linux dev board) — none of it is Sprite/AR-Glasses
    content, all OUT OF SCOPE for this repo. No action items. Low priority, unchanged
    disposition from prior check (prior note said "redirects to open.rokid.com" — that
    was evidently not consistently true; this run it served the GitBook content directly).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-12
  last_known_version: |
    client-l 1.1.0 (release; maven-metadata.xml lastUpdated 2026-07-02T09:16:06Z — NEW since last check;
      previous known release was 1.0.4 lastUpdated 2026-06-25. 1.1.X-SNAPSHOT now in development.)
    client-m 1.2.2 (release; lastUpdated 20260608030211 — unchanged since last check)
    cxr-service-bridge 1.0 (release; lastUpdated 20260522063622 — unchanged since last check)
  notes: |
    Verified live 2026-07-12 via direct fetch of maven-metadata.xml for all three artifacts.
    client-l 1.1.0 is a CONCRETE, freshly-observed new release (full version list confirms
    1.0.4 -> 1.1.0 progression, with 1.1.X-SNAPSHOT now the active dev branch). Neither
    developerdoc.rokid.com/sdk nor open.rokid.com/sdk (both scraped fresh this run) show
    a changelog beyond v1.0.4 (portal-published 2026-06-29) — official changelog for 1.1.0
    has NOT been published yet as of this check. This is the same "Maven leads portal"
    pattern seen with 1.0.4 previously.
    client-m and cxr-service-bridge confirmed unchanged — local cxr-m docs already
    correctly pin 1.2.2 (verified via grep of cxr-m/sdk-integration.md and
    cxr-m/release-notes.md), so no action needed for CXR-M this cycle.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-12
  notes: Re-verified live 2026-07-12. Still only 2 public repos: UXR-docs (out-of-scope spatial-computing SDK) and glass2-docs (out-of-scope Glass 2 hardware, issues from 2020-2021 era). No in-scope Sprite/CXR content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-12
  notes: Re-verified live 2026-07-12. Map returned ~25 repos this time (NextForum, mingutils, RokidMobileSDK{iOS,Android}Demo, CloudAppClient, blacksiren, RokidVoiceAIDemo, better_jieba, community, NewsDemo, docs, skill-java, UXR-docs [out of scope], rokidos-cli, node-webworker, RokidSDK-Swift, rokidos-www, ELMo-chinese, glass-docs, tts-demo, RokidPhone). All are legacy Rokid smart-speaker / voice-platform / old Glass-1-2 repos, none are Sprite/AR-Glasses content. No action items.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-12
  notes: Re-verified live 2026-07-12 — map returned empty links array (0 URLs) again, same as prior check. No actionable content. May have private repos not visible to Firecrawl.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Re-verified live 2026-07-12 (still pending approval, NOT added to registry).
    Map returns 8 URLs: root, /?lang=cn, /sdk?lang=zh, /sdk?lang=en, /sprite?lang=zh,
    /sprite?lang=en, /academy, /master?lang=cn.
    /sdk?lang=zh scraped fresh (with --wait-for 3000) and diffed against
    developerdoc.rokid.com/sdk: same CXR-L version (1.0.4, updated 2026-06-29) and
    same official v1.0.4 changelog content; open.rokid.com's version additionally
    includes the full V1.0.3 / V1.0.1 changelog history in one page (developerdoc's
    default-tab scrape only surfaced V1.0.4 + V1.0.1). Confirms both hosts serve the
    same backend content — open.rokid.com is the more complete public-facing mirror.
    /master is YodaOS-Master content — skipped, out of scope, not scraped for content.
    /academy ("乐奇学院") still mixes in-scope CXR-L/Glasses courses with out-of-scope
    UXR3.0/AIUI courses — not scraped this run (low priority, no evidence of change).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com/
    developerdoc.rokid.com — still requires explicit user sign-off before Scout treats
    it as a first-class registry source.
