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
  last_checked: 2026-07-03
  last_known_version: CXR-L 1.0.3 (portal still shows 1.0.3 as of 2026-06-28; Maven has 1.0.4)
  notes: |
    React SPA. Reconfirmed 2026-07-03: /sdk renders the AIUI/AI Agent-focused rokid-developer
    homepage, not the CXR SDK changelog. Homepage's "Hardware" section links Rokid Glasses ->
    open.rokid.com/sdk, and "Operating Systems" section links YodaOS-Sprite -> open.rokid.com/sprite
    (YodaOS-Master -> open.rokid.com/master, skipped, out of scope). firecrawl map returned 36 URLs,
    all Rokid Store app-catalog detail pages (/detail?appId=...) for Rokid Station/Air apps —
    out-of-scope hardware, no CXR SDK doc URLs surfaced via map (SPA client routes aren't indexed).
    Real Sprite/CXR doc surfaces remain migrated to open.rokid.com (see new-sources note below).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-03
  last_known_version: CXR-L 1.0.4 (official, published 2026-06-29 — CONFIRMED this run), CXR-M tab not reverified this run (last confirmed 1.1.0 portal vs Maven 1.2.2), 眼镜端裸机开发 not reverified this run (last known 0.0.1, 2026-03-01)
  notes: |
    SDK landing page (/sdk) is a tab-based SPA; firecrawl scrape only renders the default-active
    tab (CXR-L), CXR-M and 眼镜端裸机开发 tab content requires a JS click and was NOT captured this
    run. CONFIRMED 2026-07-03: /sdk CXR-L tab now shows an OFFICIAL v1.0.4 changelog dated
    2026.06.29 (previously, as of 2026-06-28/30, the portal still showed only the 1.0.3 changelog
    with 1.0.4 unpublished). New in the official changelog vs local docs' binary-diff-only
    "provisional" v1.0.4 entry: (1) confirms value range for setGlassBrightness/setGlassVolume is
    0...15 (local docs had "value range undocumented"); (2) documents iOS parity APIs not
    previously captured locally — RGCxrClient gains setBrightness/getBrightness/setVolume/getVolume,
    RGCxrDeviceInfo gains brightness/sound fields, iOS docs/sample now aligned to v1.0.4; (3) new
    "设备控制" (Device Control) chapter added for both Android and iOS; (4) Android sample zip
    updated to v1.0.4.
    CONTENT DRIFT on /sprite FAQ: fresh scrape 2026-07-03 shows only 4 short Q&As, all CXR-M/CXR-S
    focused (e.g. "CXR-M SDK 的主要功能包含哪些？", "CXR-M SDK 目前支持哪些设备使用？"), matching the
    OLDER FAQ content the local doc's own changelog note says was "replaced" as of 2026-06-08.
    Local yodaos/docs/sprite-overview.md FAQ section currently has 8 CXR-L/bare-metal-focused Q&As
    captured 2026-06-08. The device-spec table and Developer Toolkit section on /sprite still match
    local content exactly (no drift there). This FAQ reversion could be a genuine upstream rollback
    or an A/B-served variant — flagged as content drift needing human judgment, not auto-applied.
    Map still returns only /sdk, /sprite, root (3-4 URLs). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-03
  notes: |
    STILL BROKEN as of 2026-07-03 (re-verified, unchanged from 2026-06-28 finding). Both
    previously valid workspace hashes still return OSS NoSuchKey:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey (RequestId
      6A45D62ADA59D037305A7A73 this run)
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey (RequestId 6A45D62DDC817033390450AF
      this run)
    Root path still returns HTTP 415 Unsupported Media Type. Source remains unreachable; new
    workspace hashes still unknown. Not usable as a monitor candidate.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-03
  notes: |
    Reconfirmed 2026-07-03 via direct scrape of root: serves the identical AIUI/AI Agent
    rokid-developer homepage as ar.rokid.com (English-language render this time). No in-scope
    Sprite/AR Glasses content on the live root. CAUTION: firecrawl map returned 12 stale-looking
    URLs (old GitBook paths like /docs/rokid-homebase-docs/v2/, /docs/8-app/alliance/...) that
    are NOT reflected in the live scrape — these are likely stale entries in Firecrawl's crawl
    index rather than currently-served pages; treat map output for this host as unverified unless
    individually scraped. Low priority either way — GitBook content is legacy Speech/HomeBase
    (out of Sprite/CXR scope).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-03
  last_known_version: |
    client-l 1.0.4 (release; metadata lastUpdated 20260625070819 — UNCHANGED this run; now backed
    by an official portal changelog as of 2026-06-29, see developerdoc.rokid.com entry above)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Re-verified 2026-07-03 by scraping maven-metadata.xml for all three artifacts directly — all
    three lastUpdated timestamps are byte-identical to the 2026-06-30 check, confirming no new
    releases since. client-l's 1.0.4 release (already known from Maven since 2026-06-25) now has
    an official Rokid changelog published on developerdoc.rokid.com/sdk as of 2026-06-29.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-03
  notes: Reconfirmed 2026-07-03. Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-03
  notes: Reconfirmed 2026-07-03 (72 URLs mapped). Same repo set as before — glass-docs (stale, 2020, Glass 1/2 era, not Sprite), UXR-docs (out of scope), remainder is Speech/OpenVoice/CloudApp/skill-kit repos unrelated to Sprite/AR Glasses. No in-scope content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-03
  notes: Reconfirmed 2026-07-03. Map returned only the org root URL (1 URL) — no individual repos surfaced via map. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content this run. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval. NOT scraped this run (out of the 8-source registry; scout does not silently extend source list).
  notes: |
    Carried forward from 2026-06-28 discovery, NOT independently re-verified in this 2026-07-03
    run. ar.rokid.com/sdk and developer.rokid.com both link their "Docs" CTAs to
    open.rokid.com/sdk?lang=en and open.rokid.com/sprite?lang=en as of 2026-07-03, reinforcing
    that this is the canonical live developer-portal surface for in-scope content, but this run
    did not scrape open.rokid.com itself pending user approval.
    Prior notes (2026-06-28): open.rokid.com/sprite?lang=zh content matched developerdoc.rokid.com/sprite
    exactly. open.rokid.com/sdk?lang=zh returned only SPA shell. Map returned 4 URLs: root,
    /sdk?lang=zh, /sprite?lang=zh, /academy. /academy hosts "乐奇学院" (Rokid Academy) with
    CXR-L/Glasses courses (in scope) alongside UXR 3.0 and AIUI courses (out of scope).
    YodaOS-Master content linked from open.rokid.com/master — would be skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com — awaiting
    user decision.
