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
  last_checked: 2026-09-21
  last_known_version: CXR-L 1.1.2 (via linked developerdoc.rokid.com table); root page links to custom.rokid.com detail docs directly
  notes: |
    Root page is now a fully rebuilt EN/CN landing page (title "AIUI: The Next Frontier"),
    verified via scrape 2026-09-21. Products section has 3 cards: "Rokid Glasses" -> Build Now
    -> https://aiui-global.rokid.com/ (AIUI platform, not CXR docs); "Rokid AR" -> Build Now ->
    https://open.rokid.com/master?lang=en (YodaOS-Master, OUT OF SCOPE, skipped); "Rokid Glass3"
    -> Build Now -> https://x-docs.rokid.com/docs/en/ (new/unclear hardware model, NOT yet
    in-scope per CLAUDE.md hardware list -- flagged under New sources discovered, needs human
    triage, not scraped further).
    Operating Systems section has a YodaOS-Sprite card (in scope) linking to
    https://open.rokid.com/sprite?lang=en (content verified identical to developerdoc.rokid.com/sprite),
    plus SDK cards with DIRECT deep links into custom.rokid.com detail pages, all verified live
    (HTTP 200) 2026-09-21, superseding the 2026-06-28 "BROKEN" status:
      - CXR-L card -> https://t.rokid.com/uwxdzi51 -> redirects to
        https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html
        (workspace 84feb39f... is LIVE again; page header reads "Version 1.0.4", but see
        developerdoc.rokid.com entry below for the authoritative 1.1.2 latest-version table.)
      - CXR-S card -> https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/us/3fe1c87b945245bf8b6c50393f4da7b6.html
        (workspace 57e35cd3... LIVE again; CN mirror page scraped shows "版本1.0" / v1.0, content
        matches local cxr-s/brief.md structurally -- no drift.)
      - Bare-Metal Development card -> https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html
        (BRAND NEW workspace hash, not in registry before. "Version 1.0.0", 7-chapter restructure.
        See custom.rokid.com entry below -- this is the P0 item.)
    YodaOS-Master card and its SDK tiles (UXR3.0, MRTK3, XRI, Rokid Unreal OpenXR Plugin, Rokid
    Native OpenXR SDK, JSAR) all present and correctly out of scope; skipped, not scraped further.
    "AIUI Studio" section is a separate AI-agent IDE product, out of scope (not CXR/YodaOS-Sprite).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-21
  last_known_version: CXR-L 1.1.2 (updated 2026.09.08); CXR-M 1.1.0 public / Maven actually at 1.2.2 (Business-gated, known gap); Glasses Bare Metal 1.0.0
  notes: |
    /sdk?lang=en scraped 2026-09-21 (.firecrawl/developerdoc-sdk.md). Comparison table now
    reads: CXR-L SDK "Latest Version 1.1.2" (Updated 2026.09.08, Public); CXR-M SDK "Latest
    Version 1.1.0" (Updated 2026.04.01, Business/contact-sales gated -- unchanged from prior
    check); Glasses Bare Metal "Latest Version 1.0.0" (up from the 0.0.1 last recorded
    2026-03-01 -- confirmed via corresponding custom.rokid.com content, see below).
    /sprite?lang=en scraped 2026-09-21 (.firecrawl/developerdoc-sprite.md) -- content matches
    local yodaos sprite docs and matches open.rokid.com/sprite byte-for-byte in structure
    (diff showed only a stray title line). No drift.
    Map returns 3 URLs: /sdk, /sdk?lang=en, root. YodaOS-Master tab still present, skipped
    (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-21
  last_known_version: CXR-L page header "1.0.4"; CXR-S page header "1.0"; Bare-Metal page header "1.0.0"
  notes: |
    RESTORED as of 2026-09-21 -- supersedes the 2026-06-28 "BROKEN / NoSuchKey" status.
    Root path (bare workspace listing) was NOT re-tested this run; only specific document
    paths were verified, all HTTP 200 with real content:
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): LIVE. Verified page
      .../pc/us/663f26766e7348059905815bc022e1f7.html (reached via https://t.rokid.com/uwxdzi51
      redirect) -- header reads "Version 1.0.4", Introduction chapter content captured in
      .firecrawl/t-rokid-cxrl.md. `firecrawl map` against the bare workspace root returned 0
      URLs (sitemap-based map does not enumerate this SPA's routes) -- direct scrape is the
      only reliable method for this workspace.
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / shared): LIVE. Map returned 5 URLs
      (.firecrawl/custom-shared-map.json); verified 2 pages by scrape -- CXR-S intro
      (.../3fe1c87b945245bf8b6c50393f4da7b6.html) and CXR-S 简介/SDK-与-Glasses chapter
      (.../2786298057084a82b170bf725aef6b5d.html, "版本1.0") -- content matches local
      cxr-s/brief.md, no drift detected.
    - ff28c865a9634876be98cbc293588460 (Bare-Metal, NEW hash, not previously registered):
      LIVE. .../pc/us/index.html, "Version 1.0.0". Full 7-chapter restructure: Introduction,
      Quick Start, Sample Project and Pages, Glasses UI Design Guidelines, Keys/Wear/Fold
      Events, Raw Audio, Photo Capture/Video Recording, IMU and Sensors. Captured in full in
      .firecrawl/custom-baremetal-new.md. This is a major content expansion vs. local
      cxr-baremetal/ (3 files, pinned to the old workspace 57e35cd3.../...13083daf...html at
      doc v0.0.1, fetched 2026-03-01) -- see Action items P0.
    Root path itself (https://custom.rokid.com/prod/rokid_web/ with no workspace hash) was
    NOT re-tested this run -- unknown if still HTTP 415. Old CXR-L hash 84feb39f... is
    confirmed alive again (was NoSuchKey on 2026-06-28); do not assume other stale hashes are
    dead without re-checking.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-21
  notes: Re-verified 2026-09-21 via scrape (.firecrawl/developer-rokid-root.md) -- still redirects to/mirrors the same ar.rokid.com "AIUI: The Next Frontier" landing content (identical Products/Operating Systems/AIUI Studio sections). No in-scope content found beyond what ar.rokid.com already surfaces. Low priority, no action.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-21
  last_known_version: |
    client-l: release 1.1.2 (maven-metadata.xml lastUpdated 20260910022017 / 2026-09-10).
      Version history includes 1.0.1, 1.0.2, 1.0.3, 1.0.4, 1.1.0, 1.1.1, 1.1.2 -- local repo
      release notes stop at 1.0.4 (2026-06-18). THREE releases undocumented (1.1.0, 1.1.1,
      1.1.2), including a minor version bump. See Action items P0.
    client-m: release 1.2.2 (maven-metadata.xml lastUpdated 20260902061433 / 2026-09-02 --
      version number unchanged from the 2026-06-30 check, but the metadata file itself was
      re-touched/re-published on 2026-09-02; treat as informational only, no version-number
      action needed). Portal (developerdoc.rokid.com/sdk) still shows 1.1.0 as latest public
      release -- this gap was already known/documented in cxr-m/release-notes.md.
    cxr-service-bridge: release 1.3 (maven-metadata.xml lastUpdated 20260918093440 /
      2026-09-18) -- UP from 1.0 (previously the only known version, lastUpdated
      2026-05-22). Version history now includes 1.0, 1.1, 1.2, 1.3. Local cxr-s/*.md pins
      cxr-service-bridge:1.0 explicitly (design-spec.md, sdk-import.md). NOTE: as of this run
      the live custom.rokid.com CXR-S doc pages STILL show "版本1.0" too -- Rokid's own docs
      have not caught up to the Maven artifact either, so this is lower urgency than the
      CXR-L gap, but flagged as P1 since 3 releases (1.1, 1.2, 1.3) are undocumented anywhere
      public.
  notes: |
    Public Maven for CXR SDK JARs/AARs. maven-metadata.xml fetched directly via
    firecrawl scrape for client-l, client-m, and cxr-service-bridge (all HTTP 200,
    .firecrawl/maven-*-metadata.xml). Firecrawl's markdown conversion strips XML tags from
    these responses (concatenates text nodes) -- version numbers were extracted by manual
    inspection of the token order (matches the standard <latest>/<release>/<versions>/
    <lastUpdated> maven-metadata.xml schema). Use Maven as the canonical "what's actually
    shipped" source; it continues to lead the dev portal.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-21
  notes: Re-verified 2026-09-21 (.firecrawl/gh-rokidglass-map.json). Still only UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 hardware) with a handful of glass2-docs issue threads. No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-21
  notes: Re-verified 2026-09-21 (.firecrawl/gh-rokid-map.json). Same repo set as before (glass-docs, UXR-docs, Speech/OpenVoice/CloudApp repos, NextForum, etc). No new Sprite/CXR-named repos surfaced. Low priority, no action.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-21
  notes: Re-verified 2026-09-21 (.firecrawl/gh-rokidar-map.json) -- map returned 1 URL (BroadcastServiceDemo, same inactive 2017 repo as before). No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval (still pending; not added this run, no human available to approve)
  notes: |
    Re-verified 2026-09-21. Map still returns the same 5 URLs: root, /sprite?lang=zh,
    /sdk?lang=en, /?lang=cn, /master?lang=zh. Scraped /sprite?lang=en and /sdk?lang=en this
    run -- content is confirmed identical to developerdoc.rokid.com/sprite and
    developerdoc.rokid.com/sdk respectively (same CXR-L 1.1.2 / CXR-M 1.1.0 / Bare Metal 1.0.0
    comparison table). /master content linked from here and from ar.rokid.com's "Rokid AR"
    card -- out of scope, skipped, not scraped.
    Still recommend registering open.rokid.com as the canonical replacement/mirror for
    ar.rokid.com and developerdoc.rokid.com's Sprite-relevant content -- pending explicit
    user/human approval per scout.md rules.

- url: https://x-docs.rokid.com
  kind: developer-portal
  covers: hardware (unclear — possibly out of scope)
  status: UNREGISTERED — requires user approval; NEWLY DISCOVERED 2026-09-21, not yet scraped
  notes: |
    Discovered 2026-09-21 as the "Build Now" target for the "Rokid Glass3" product card on
    ar.rokid.com's root landing page (link: https://x-docs.rokid.com/docs/en/). NOT scraped
    this run -- "Rokid Glass3" does not clearly match any hardware model named in
    CLAUDE.md's in-scope list (RV101/RV102/RV203/OEM variants, Rokid AI Glasses) or the
    out-of-scope list (Station 2/Pro, Max family, Air family, X-Craft, AR Studio, AR Lite,
    Glass 2). Needs human triage to determine (a) whether "Glass3" is a new/renamed in-scope
    Sprite-line product or an out-of-scope spatial-computing product, and (b) whether
    x-docs.rokid.com should be added as a ninth registered source. Do not scrape or register
    without that determination.
