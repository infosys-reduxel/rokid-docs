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
  last_checked: 2026-09-22
  last_known_version: CXR-L 1.1.2 (confirmed on developerdoc.rokid.com/sdk card, updated 2026.09.08; Maven matches)
  notes: |
    React SPA. Re-verified 2026-09-22: /sdk still renders the open.rokid.com/AIUI-style
    landing page (identical to 2026-06-28 finding), not the CXR SDK changelog. No in-scope
    content surfaced directly on this host this cycle; real changelog data came from
    developerdoc.rokid.com and maven.rokid.com instead (see those entries).
    YodaOS-Master tab remains; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-22
  last_known_version: CXR-L 1.1.2 (2026.09.08 per portal card), CXR-M 1.1.0 (still 商务对接/business-cooperation-only, unchanged), 眼镜端裸机开发 (bare-metal) 1.0.0 (2026.06.05 per portal card)
  notes: |
    /sdk scraped 2026-09-22 (`.firecrawl/developerdoc-sdk-current.md`). SDK selector cards now
    show: CXR-L 1.1.2 (公开/public, updated 2026.09.08), CXR-M 1.1.0 (商务合作/business-cooperation
    only — unchanged from 2026-06-23 finding), 眼镜端裸机开发 1.0.0 (公开/public, updated 2026.06.05).
    **Local docs are stale for two of these**: cxr-l/* pins client-l 1.0.4 (provisional,
    2026-06-25 binary diff); cxr-baremetal/* pins v0.0.1 (2026-03-01). Both are genuine P1 gaps.
    The portal's "▶ 更新内容" (changelog) and "查看文档" (view docs) sections are collapsed,
    client-side-rendered UI — a static Firecrawl scrape captures only the collapsed header, not
    the expanded changelog text. No deep-linkable doc URLs exist (map still returns only 4 URLs:
    root, /sdk, /sprite, and lang=en variants — no /sitemap.xml entry this cycle, that path now
    just serves the SPA shell). Faithfully documenting the CXR-L 1.1.0/1.1.1/1.1.2 and bare-metal
    1.0.0 changes requires downloading + decompiling the actual Maven artifacts (as was done for
    client-l 1.0.3/1.0.4) rather than scraping — flagged for a dedicated follow-up session, not
    attempted this cycle to avoid fabricating API-level detail.
    /sprite scraped 2026-09-22 — hardware spec table for Rokid Glasses verified **byte-for-byte
    identical** to `yodaos/docs/sprite-overview.md`; no drift. CXR-M FAQ wording also matches
    local docs (`cxr-m/intro.md` already documents the business-cooperation gating correctly).
    YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-22
  notes: |
    Re-checked 2026-09-22. Root path now returns HTTP 200 (was HTTP 415 as of 2026-06-28) but
    with an **empty body** — still no usable content. Both previously valid workspace hashes
    still return OSS NoSuchKey:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey (RequestId 6AB1F0A3EAC5D2353439E614)
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey (RequestId 6AB1F09FFBB19F3432DB0D63)
    Source remains unreachable for content purposes; no new workspace hashes discovered.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-22
  notes: Re-verified 2026-09-22 via Firecrawl scrape — still renders the AIUI/"Learn More" landing page (identical content to ar.rokid.com redirect target, js.rokid.com). No in-scope Sprite/AR Glasses content surfaced. Low priority, no change from 2026-06-29 finding.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-22
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20260910022017 — new release since last check; portal (developerdoc.rokid.com/sdk) confirms 1.1.2 as of 2026.09.08; local docs still pin 1.0.4; intermediate releases 1.1.0 and 1.1.1 also published on Maven but not yet reflected anywhere in this repo)
    client-m 1.2.2 (release; metadata lastUpdated 20260902061433 — version unchanged since 2026-06-09 despite a metadata republish timestamp bump; portal still shows 1.1.0/business-cooperation-only)
    cxr-service-bridge 1.3 (release; metadata lastUpdated 20260918093440 — new release since last check, was 1.0; this is CXR-S's on-device bridge dependency, not independently published on the developer portal; design-spec.md pins 1.0)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns a 303 redirect to the Nexus UI; use maven-metadata.xml under
    each artifact directly, e.g. https://maven.rokid.com/repository/maven-public/com/rokid/cxr/client-l/maven-metadata.xml).
    client-l: full <versions> list now includes 1.1.0, 1.1.1, 1.1.2 beyond the previously-known
    1.0.4 — three undocumented minor releases. Portal changelog confirms 1.1.2 is current but its
    change-log text is behind a collapsed JS widget Firecrawl's static scrape cannot expand.
    client-m: unchanged (1.2.2), consistent with unchanged business-cooperation-gated status.
    cxr-service-bridge: jumped 1.0 → 1.3 (versions 1.1, 1.2, 1.3 all new since last check,
    lastUpdated 2026-09-18 — only 4 days before this check). No public changelog source exists
    for this artifact; it underpins both cxr-l's CUSTOMAPP counterpart and CXR-S itself
    (see cxr-s/design-spec.md, currently pinned to 1.0).
    Use Maven as the canonical "what's actually shipped" source. All three version bumps above
    are genuine, live-verified P1 stale-pin findings, but none has been translated/reverse-engineered
    into repo content yet — see "Known gaps requiring follow-up" below.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-22
  notes: Verified org (id 57519491). Re-checked 2026-09-22 — still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, issues only). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-22
  notes: Verified org (id 19773259). Re-checked 2026-09-22 — same repo set as before (NextForum, mingutils, RokidMobileSDK*Demo, CloudAppClient, RokidVoiceAI*, better_jieba, rokid-speech, community, mapi-demo-outer, rokidos-www, node-http-bypass, docs). All Speech/Voice/CloudApp/legacy-mobile-SDK repos, not Sprite/AR Glasses. No in-scope content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-22
  notes: Verified org (id 25831739). Re-checked 2026-09-22 — map now returns 1 URL (BroadcastServiceDemo/gradlew, the same long-inactive 2017 demo repo as before). No actionable content. May have private repos not visible.

## Known gaps requiring follow-up (not actioned 2026-09-22)

Genuine, live-verified stale-version findings that were **not** translated/committed this cycle
because faithful documentation requires downloading and decompiling the actual artifacts (as was
done for client-l 1.0.3/1.0.4) rather than a quick scrape, and that work was judged too large/
error-prone to rush in an unattended cycle:

- **client-l 1.0.4 → 1.1.2** (`cxr-l/release-notes.md`, `cxr-l/api-reference.md`, `cxr-l/intro.md`,
  `yodaos/docs/sprite-overview.md` FAQ pin). Three undocumented releases (1.1.0, 1.1.1, 1.1.2).
  Confirmed via both Maven metadata and the developerdoc.rokid.com/sdk portal card
  (updated 2026.09.08). No official changelog text retrievable via static scrape.
- **眼镜端裸机开发 (bare-metal) 0.0.1 → 1.0.0** (`cxr-baremetal/*.md`). Major version jump
  (0.0.1 → 1.0.0) confirmed via developerdoc.rokid.com/sdk portal card (updated 2026.06.05).
  No changelog text retrievable via static scrape; likely substantial content changes given
  the version jump.
- **cxr-service-bridge 1.0 → 1.3** (`cxr-s/design-spec.md`, referenced in `cxr-l/api-reference.md`
  dependency table). Confirmed via Maven metadata only (lastUpdated 2026-09-18, 4 days before
  this check). No public changelog source exists for this artifact at all.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Discovered 2026-06-28. ar.rokid.com/sdk now renders a new AIUI-focused homepage that
    links to open.rokid.com/sdk and open.rokid.com/sprite as the canonical developer portal.
    open.rokid.com/sprite?lang=zh was scraped 2026-06-28 — content matches developerdoc.rokid.com/sprite
    exactly (same FAQ, same spec table). open.rokid.com/sdk?lang=zh returns only SPA shell.
    Map returned 4 URLs: root, /sdk?lang=zh, /sprite?lang=zh, /academy.
    /academy hosts the "乐奇学院" (Rokid Academy) learning platform with CXR-L and Glasses
    development courses (in scope) alongside UXR 3.0 (out of scope) and AIUI courses.
    YodaOS-Master content linked from open.rokid.com/master — skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com.
