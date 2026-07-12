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
  last_known_version: (no CXR doc content hosted here anymore; see notes)
  notes: |
    RE-VERIFIED 2026-07-12, no change since 2026-07-10 check below (Station
    App Store on root, AIUI homepage on /doc, YodaOS-Master card present,
    skipped).
    VERIFIED LIVE 2026-07-10 (map of root + scrape of /doc). Root path
    (https://ar.rokid.com) now maps to a Rokid Store / OTT app marketplace
    (appId=... detail pages for TV apps and games) -- not a docs portal.
    /doc route renders the same "AIUI: The Next Frontier" developer homepage
    now also served at developer.rokid.com and open.rokid.com, with a
    "Rokid Glasses" hardware card linking Docs to open.rokid.com/sdk?lang=en.
    YodaOS-Master card is present and explicit on this page -- skipped,
    out of scope, not clicked through. No CXR-M/S/L content is directly
    hosted at ar.rokid.com anymore; confirms the 2026-06-28 finding that
    real Sprite/CXR doc surfaces have migrated to open.rokid.com.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-12
  last_known_version: CXR-L 1.0.4 (official, portal updated 2026-06-29). CXR-M / 眼镜端裸机开发 tabs NOT re-verified this run (see notes).
  notes: |
    RE-VERIFIED 2026-07-12: /sdk still tops out at the official v1.0.4
    changelog (2026-06-29); no v1.1.0 changelog published yet. /sprite
    spec table diffed field-by-field against yodaos/docs/sprite-overview.md
    -- exact match, no drift. No change since 2026-07-10 check below.
    VERIFIED LIVE 2026-07-10. Map returns 3 URLs (root, /sdk, /sprite) --
    /sitemap.xml no longer listed (was 4 URLs as of 2026-06-30).
    /sdk scraped live: CXR-L SDK now shows an OFFICIAL v1.0.4 changelog,
    updated 2026-06-29 (previously only a provisional binary-diff
    reconstruction existed, sourced 2026-06-25). Official changelog text:
    Android client-l upgraded to 1.0.4; new setGlassBrightness(level) /
    setGlassVolume(level) API, level range 0...15 (now documented -- local
    docs previously said "range undocumented"); GlassInfo brightness/sound
    fields exposed via onGlassDeviceInfo; new "Device Control" chapter for
    both Android and iOS; Android Sample updated to v1.0.4; iOS RGCxrClient
    gains setBrightness/getBrightness/setVolume/getVolume, RGCxrDeviceInfo
    gains brightness/sound fields; iOS docs/sample version-aligned to 1.0.4.
    This iOS parity and the value-range detail are NOT in local docs -- see
    P1 action item.
    /sprite scraped live: CXR-S Q&A and Rokid Glasses hardware spec table
    (SoC Qualcomm AR1, 2GB RAM, 32GB ROM, 210mAh, Sony IMX681 12MP camera,
    1500 nits, 480x640 waveguide res, IPX4, BT5.3, WiFi6) -- content
    consistent with local docs' architecture description but this
    consolidated spec table does not appear to exist in
    yodaos/docs/hardware/product-variants.md.
    NOT VERIFIED THIS RUN: the CXR-M SDK and 眼镜端裸机开发 tabs on /sdk.
    This is a tab-based SPA; plain scrape only renders the default-active
    CXR-L tab body. Confirming CXR-M/裸机开发 content requires an
    actions-based scrape (click tab) next cycle -- do not treat prior
    registry values for those tabs as re-verified.
    YodaOS-Master tab remains present; skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    RE-VERIFIED 2026-07-12: same partially-reachable state as 2026-07-10
    (map succeeds, individual page scrapes still only return empty SPA-shell
    placeholders). No new evidence this run; still unverified as a content
    source.
    STATUS CHANGED from BROKEN (2026-06-30) to PARTIALLY REACHABLE, verified
    live 2026-07-10. Root path (https://custom.rokid.com/prod/rokid_web/)
    still returns HTTP 415 Unsupported Media Type when scraped directly
    (confirmed live, statusCode 415, contentType application/octet-stream --
    same failure mode as before). HOWEVER a live `firecrawl map` of that same
    root URL now SUCCEEDS and returns 23 real content-page URLs across 3
    workspace hashes (previously all workspace keys 404'd with OSS
    NoSuchKey errors):
      - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (matches the previously-known
        CXR-M/CXR-S/眼镜端裸机开发 hash) -- page titles include "Rokid
        Glasses 设备连接与管理", "设备连接管理", "Rokid Glasses AI
        流程交互", "版本". IN SCOPE. Body content did NOT fully render via
        plain scrape or scrape with --wait-for 6000ms (only "版本1.0" /
        "版本0.0.5" SPA-shell placeholders were captured, not real
        document body) -- this SPA needs actions-based scraping (click
        into doc tree) to retrieve real content; NOT independently
        content-verified this run, only URL/title-verified.
      - c88be4bcde4c42c0b8b53409e1fa1701 (NEW hash, not previously known)
        -- titles include "UXR3.0 SDK: Opening a New Chapter in YodaOS"
        and "双目渲染、手势交互、平面检测等能力" (binocular rendering /
        gesture interaction / plane detection). OUT OF SCOPE (UXR3.0
        spatial-computing SDK) -- explicitly skipped, not scraped further,
        no workspace hash logged as an in-scope source.
      - 323825adf4914c21be8a0d5fe7b8a9e5 (NEW hash) -- appears to be a
        top-level nav/landing page listing Rokid Glasses, YodaOS-Sprite
        AIUI Studio, Rokid AR Lite, YodaOS-Master, Rokid AR Studio, Rokid
        Glass3. MIXED scope -- only the Rokid Glasses entry point is
        in-scope; AR Lite/AR Studio/Master/Glass3 are out of scope and
        were not clicked through.
      - The previously-known CXR-L-specific hash
        84feb39f8ef141b0ad0326f902ab881f did NOT appear in this map.
        Unknown whether its content merged into 57e35cd3... or moved
        elsewhere -- unconfirmed, flag for next cycle.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    RE-VERIFIED 2026-07-12: still no in-scope content hosted directly here.
    No change since 2026-07-10 check below.
    VERIFIED LIVE 2026-07-10 (direct scrape). Confirms this domain now
    serves the identical "AIUI: The Next Frontier" homepage content as
    open.rokid.com and ar.rokid.com/doc (same markup, same asset hashes).
    "Rokid Glasses" hardware card links Docs to open.rokid.com/sdk?lang=en.
    YodaOS-Master card explicitly present -- skipped, out of scope, not
    clicked through. No in-scope Sprite/AR Glasses doc content hosted
    directly at this domain; low priority as a distinct source (mirrors
    developerdoc.rokid.com / open.rokid.com).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-12
  last_known_version: |
    client-l 1.1.0 (release; metadata lastUpdated 20260702091606 / 2026-07-02
      -- unchanged since 2026-07-10 check, re-confirmed via fresh
      maven-metadata.xml fetch 2026-07-12. Still no official changelog
      published (developerdoc.rokid.com/sdk tops out at 1.0.4 / 2026-06-29).
      Already documented provisionally via binary-diff in this branch's
      cxr-l/release-notes.md and cxr-l/api-reference.md -- no further action.)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 -- unchanged,
      re-verified via fresh maven-metadata.xml fetch 2026-07-12)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 --
      unchanged, re-verified via fresh maven-metadata.xml fetch 2026-07-12)
  notes: |
    RE-VERIFIED 2026-07-12: no new releases since 2026-07-10 check below.
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/
    for human browsing, but maven-metadata.xml under each artifact path is
    directly fetchable and is the canonical machine-readable source).
    client-l: 1.1.0 is a brand-new, previously-unseen release as of this
    run (2026-07-10) -- portal/local docs still only cover up through 1.0.4.
    No binary diff of 1.1.0 has been performed yet.
    client-m: 1.2.2 unchanged since 2026-06-08. Portal CXR-M tab was not
    re-verified this run (tab-based SPA, see developerdoc.rokid.com note) --
    do not assume portal still shows 1.1.0 without a fresh check next cycle.
    cxr-service-bridge: 1.0 unchanged since 2026-05-22.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    RE-VERIFIED 2026-07-12: no change since 2026-07-10 check below.
    VERIFIED LIVE 2026-07-10 (map). Still only 2 public repos: UXR-docs
    (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope
    Glass 2 / older hardware). No in-scope Sprite/AR Glasses content.
    Unchanged from prior check. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    RE-VERIFIED 2026-07-12: no change since 2026-07-10 check below.
    VERIFIED LIVE 2026-07-10 (map). Same repo set as before: native-system-docs,
    NextForum, mingutils, RokidMobileSDKiOSDemo, RokidMobileSDKAndroidDemo,
    CloudAppClient, blacksiren, DeepFaceLab, and others -- Speech/OpenVoice/
    CloudApp/forum-tooling repos, plus an unrelated DeepFaceLab mirror. No
    in-scope Sprite/AR Glasses content found. Unchanged. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-12
  notes: |
    RE-VERIFIED 2026-07-12: no change since 2026-07-10 check below.
    VERIFIED LIVE 2026-07-10 (map). Map again returned an empty links array
    (0 URLs), consistent with the 2026-06-30 check. No actionable content.
    May have private repos not visible to this API key.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED -- requires user approval. NOT independently
    scraped this run (out of the 8 approved registry sources); its
    existence and role as the new canonical portal is corroborated
    indirectly via live scrapes of developer.rokid.com and ar.rokid.com/doc,
    both of which link Rokid Glasses docs to open.rokid.com/sdk?lang=en.
  notes: |
    Discovered 2026-06-28, corroborated again 2026-07-10 via developer.rokid.com
    and ar.rokid.com/doc (both now serve the same "AIUI: The Next Frontier"
    homepage and link Docs to open.rokid.com/sdk / /sprite). Prior direct
    check (2026-06-28) found open.rokid.com/sprite?lang=zh content matches
    developerdoc.rokid.com/sprite exactly; open.rokid.com/sdk?lang=zh
    previously returned only SPA shell. /academy hosts "乐奇学院" (Rokid
    Academy) with CXR-L / Glasses courses (in scope) alongside UXR 3.0 and
    AIUI courses (out of scope). YodaOS-Master content linked from
    open.rokid.com/master -- skipped (out of scope). Suggest registering
    open.rokid.com as the canonical replacement for ar.rokid.com; requires
    explicit user sign-off before Scout treats it as a registry source.
