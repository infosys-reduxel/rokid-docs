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

## Process note (2026-07-24 Scout run)

At the start of this run, `.claude/agents/rokid-sources.md` already had an UNCOMMITTED working-tree modification (git diff HEAD showed +107/-8 lines) that had bumped every source's `last_checked` to 2026-07-24 and added extremely detailed "RE-VERIFIED 2026-07-24" notes for all 8 sources -- BEFORE this Scout run made a single tool call. This Scout has no record of producing that content. Per the verified-upstream-or-abort rule, content not produced by an actual Firecrawl/WebFetch call *in this run* cannot be treated as freshly verified, regardless of how plausible it reads or how it got there (crashed/uncommitted prior attempt vs. something else -- origin was not determined and is out of scope for Scout to investigate further). That uncommitted draft was DISCARDED and is NOT reflected below. Everything below dated 2026-07-24 was produced by live Firecrawl/WebFetch calls actually made in this run; see the accompanying report to the Leader for full tool-call evidence. Flagging this discrepancy to the Leader as a process/data-integrity item worth investigating (e.g. was another Scout invocation started and abandoned without committing?).

**Root cause identified (orchestrator note, added after commit `7a9723756`):** not tampering. This cycle's orchestration accidentally spawned three concurrent Scout runs against this same branch (a coordination bug on the orchestrator's part, not a Scout error) -- the "mystery" uncommitted draft this run discarded was legitimate work from a sibling Scout run in the same cycle, and the commit that later appeared under this filename (`7a9723756`) landed via a file-write race between two sibling Scouts: its commit *message* summarizes one sibling's findings (including a browse-listing byte/timestamp check that ruled the two Maven `lastUpdated` bumps below out as re-indexing noise), but the file *content* that actually got committed is this run's more cautious, unconfirmed "WATCH item" version, because this run's write landed last. Net effect: no fabricated or inaccurate content reached the repo either way -- one sibling did strictly more verification (an actual browse-listing scrape) than this run did, and that verification is restored into `last_known_version` below in a follow-up commit so it isn't lost. Recommend the orchestrator avoid spawning multiple concurrent Scout instances against the same branch in one cycle.

## Sources

- url: https://ar.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-03
  last_known_version: (no CXR doc content hosted here anymore; see notes)
  notes: |
    RE-VERIFIED 2026-08-03 (this Scout run -- fresh live `firecrawl map` of
    root, limit 20, no cache): still only Rokid Store OTT/game-app
    marketplace detail pages (appId=...), no CXR-M/S/L content. Unchanged
    from every check since 2026-06-28. Evidence:
    .firecrawl/ar-rokid-map-20260803.json.
    RE-VERIFIED 2026-07-29 (this Scout run -- fresh live `firecrawl map` of
    root, limit 20, plus a cache-bypassed scrape of /sdk with --max-age 0,
    retried once after an initial attempt errored with a transport-level
    ECONNRESET, not a site error): map unchanged -- Rokid Store OTT/game-app
    marketplace detail pages (appId=...) only, no CXR-M/S/L content. Retried
    scrape of /sdk confirms sourceURL ar.rokid.com/sdk -> finalURL
    https://open.rokid.com/ (statusCode 200), rendering the identical
    "AIUI: The Next Frontier" homepage, same hardware cards, same Docs link
    to open.rokid.com/sdk?lang=en. No change from the 2026-07-25 baseline.
    Evidence: .firecrawl/ar-rokid-map-20260729.json,
    .firecrawl/ar-rokid-sdk-fresh-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run -- fresh live `firecrawl map` of
    root, limit 50, plus a cache-bypassed scrape of /sdk with --max-age 0):
    map unchanged -- Rokid Store OTT/game-app marketplace detail pages
    (appId=...) plus generic /doc link, no CXR-M/S/L content hosted here.
    Direct scrape of /sdk confirms sourceURL ar.rokid.com/sdk -> finalURL
    https://open.rokid.com/ (statusCode 200, cacheState absent/null
    on this --max-age 0 attempt, i.e. genuinely fresh, not cache-served), rendering the identical "AIUI: The Next
    Frontier" homepage with the same hardware cards (Rokid Glasses /
    Rokid AR Lite / Rokid AR Studio) and the same Docs link to
    open.rokid.com/sdk?lang=en. No change from the 2026-07-24 baseline.
    Evidence: .firecrawl/ar-rokid-map-20260725.json,
    .firecrawl/ar-rokid-sdk-fresh-20260725.json.
    RE-VERIFIED 2026-07-24 (this Scout run -- live `firecrawl map` of root,
    limit 50, no cache; plus a direct scrape of /sdk): map returns the same
    long-standing pattern -- Rokid Store OTT/game-app marketplace detail
    pages (appId=...) plus a generic /doc link, no CXR-M/S/L content
    directly hosted here. Direct scrape of https://ar.rokid.com/sdk
    confirms it 301/redirects: sourceURL ar.rokid.com/sdk -> finalURL
    https://open.rokid.com/ (statusCode 200, contentType text/html),
    rendering the "Rokid Open Platform" / "AIUI: The Next Frontier"
    homepage. No change from prior cycles.
    RE-VERIFIED 2026-07-21 (live `firecrawl map` of root, limit 500, plus
    scrape of /doc): same pattern as every check since 2026-07-10 -- Rokid
    Store app-catalog detail pages, generic /doc and /?lang=en links, no
    CXR-M/S/L content directly hosted here. No change.
    RE-VERIFIED 2026-07-18 (live `firecrawl map` of root, limit 20, fresh
    fetch, no cache): still returns Rokid Store OTT/game-app marketplace
    detail pages (appId=...) plus a generic /doc link and /?lang=en (EN
    homepage). Same pattern as every check since 2026-07-10. No CXR-M/S/L
    content directly hosted here. No change.
    RE-VERIFIED 2026-07-16 (live scrape of /sdk and /sprite, en locale):
    both routes render the identical "AIUI: The Next Frontier" homepage
    (byte-identical to each other -- confirmed via diff). Rokid Glasses
    hardware card still links Docs to open.rokid.com/sdk?lang=en. YodaOS-
    Master, Rokid AR Lite, Rokid AR Studio (Station 2 / Station Pro family)
    cards explicitly present -- skipped, out of scope, not clicked through.
    No CXR-M/S/L content directly hosted here. No change since 2026-07-14.
    PRIOR CYCLE (2026-07-14): re-verified, no change since 2026-07-12/07-10.
    Root path maps to a Rokid Store / OTT app marketplace (out of scope);
    /doc (now /sdk) renders the AIUI homepage. Real Sprite/CXR doc surfaces
    migrated to open.rokid.com as of 2026-06-28.
    RE-VERIFIED 2026-07-17: fresh `firecrawl map` of root (limit 20)
    returns Rokid Store app-marketplace detail pages (appId=... , all
    out-of-scope OTT/game apps) -- same pattern as prior checks, no
    in-scope CXR content surfaced. No change.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-21
  last_known_version: |
    RE-VERIFIED 2026-08-21 (this run -- `firecrawl scrape --only-main-content
    --wait-for 5000 --max-age 0` of /sdk successfully rendered the SPA this
    time; an earlier attempt this same session without --wait-for returned
    only the shell, confirming --wait-for is required for this route -- see
    the /sdk-shell note under the "New sources discovered" open.rokid.com
    entry for the same behavior there): "SDK 选型速览" table unchanged --
    CXR-L 公开 1.0.4 (更新于 2026.06.25), CXR-M 商务合作 1.1.0 (更新于
    2026.04.01), 眼镜端裸机开发 公开 1.0.0 (更新于 2026.06.05). The portal
    changelog for CXR-L remains pinned at 1.0.4 and has NOT caught up to
    Maven's 1.1.1 (or even 1.1.0) -- confirms Maven continues to lead this
    portal by 2 releases for CXR-L; no official changelog for 1.1.x exists
    on this surface as of today. Bare-metal 1.0.0 badge matches the local
    doc version already recorded in cxr-baremetal/development-guide.md.
    RE-VERIFIED 2026-08-03 (this Scout run -- fresh `firecrawl scrape
    --only-main-content --wait-for 5000 --max-age 0` of /sdk): "SDK 选型
    速览" table + card badges read directly: CXR-L 公开 1.0.4 (更新于
    2026.06.25), CXR-M 商务合作 1.1.0 (更新于 2026.04.01), 眼镜端裸机开发
    公开 1.0.0 (更新于 2026.06.05) -- all three UNCHANGED from the
    2026-07-29 baseline. Cross-checked against fresh Maven
    client-l/client-m maven-metadata.xml this run: client-l release still
    1.1.0/lastUpdated 20260718072455, client-m still 1.2.2/lastUpdated
    20260608030211 -- both unchanged, no new tagged release. Bare-metal P1
    (local cxr-baremetal/*.md pinned "v0.0.1 (2026-03-01)" vs upstream
    1.0.0/2026-06-05) ACTIONED THIS CYCLE: the 简介 (intro) sub-page's real
    body content was successfully retrieved this run (see custom.rokid.com
    entry below) and translated into cxr-baremetal/development-guide.md,
    which now cites this v1.0.0 source and bumps its Doc version line.
    key-broadcasts.md and audio-recording.md remain pinned to v0.0.1 --
    their corresponding sub-pages (按键与佩戴和折叠, 原始音频) were not
    retrieved this run (same click-through blocker as prior cycles) so they
    were left unchanged rather than guessed. Evidence:
    .firecrawl/developerdoc-sdk-20260803.md.
    RE-VERIFIED 2026-07-29 (this Scout run -- fresh live `firecrawl map`,
    limit 50, plus a cache-bypassed `--max-age 0` scrape of /sdk): map still
    returns 3 URLs (root, /sdk, /sitemap.xml), unchanged shape. First /sdk
    scrape attempt (default wait) rendered only the emoji legend line, not
    the full "SDK 选型速览" table -- a rendering-timing miss, not a content
    change; a retry with --wait-for 5000 successfully rendered the full
    comparison table + card badges: CXR-L 公开 1.0.4 (更新于 2026.06.25),
    CXR-M 商务合作 1.1.0 (更新于 2026.04.01), 眼镜端裸机开发 公开 1.0.0
    (更新于 2026.06.05) -- ALL THREE UNCHANGED from the 2026-07-25 baseline.
    Cross-checked against Maven client-l maven-metadata.xml fetched fresh
    this run: release/latest still 1.1.0, lastUpdated still 20260718072455
    -- no new tagged release, portal/Maven agreement unchanged. Bare-metal
    P1 (local cxr-baremetal/*.md pinned "Doc version: v0.0.1 (2026-03-01)")
    RE-CONFIRMED STILL OPEN via direct grep of local files this run.
    Evidence: .firecrawl/developerdoc-map-20260729.json,
    .firecrawl/developerdoc-sdk-fresh2-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run -- fresh live `firecrawl map`,
    limit 50, plus cache-bypassed `--max-age 0` scrapes of /sdk and
    /sprite): map still returns exactly 3 URLs (root, /sdk, /sprite),
    unchanged shape. /sdk "SDK 选型速览" comparison table + card badges
    read directly off a genuinely fresh fetch (cacheState null on the
    --max-age 0 attempt, confirming this bypassed any cache): CXR-L 公开
    1.0.4 (更新于 2026.06.25), CXR-M 商务合作 1.1.0 (更新于 2026.04.01),
    眼镜端裸机开发 公开 1.0.0 (更新于 2026.06.05) -- ALL THREE UNCHANGED
    from the 2026-07-24 baseline. Cross-checked against Maven client-l
    maven-metadata.xml fetched fresh this run (see maven.rokid.com entry):
    release/latest still 1.1.0, no new tagged release -- portal and Maven
    remain in agreement, no new gap opened. Bare-metal P1 (local cxr-
    baremetal/*.md pinned "Doc version: v0.0.1 (2026-03-01)") RE-CONFIRMED
    STILL OPEN via direct grep of local files this run. Evidence:
    .firecrawl/developerdoc-map-20260725.json,
    .firecrawl/developerdoc-sdk-fresh-20260725.json.
    RE-VERIFIED 2026-07-24 (this Scout run -- fresh `firecrawl map` +
    direct scrapes of /sdk and /sprite, no cache): map returns 3 URLs
    (root, /sdk, /sdk?lang=zh) -- unchanged shape. /sdk "SDK 选型速览"
    comparison table + card badges read directly: CXR-L 公开 1.0.4
    (更新于 2026.06.25), CXR-M 商务合作 1.1.0 (更新于 2026.04.01),
    眼镜端裸机开发 公开 1.0.0 (更新于 2026.06.05) -- all three UNCHANGED
    from the 2026-07-21 baseline. Cross-checked against Maven (see
    maven.rokid.com entry): portal still lags Maven's client-l 1.1.0
    tag by version-string (both now say 1.1.0, so actually caught up on
    the number, though no official CXR-L 1.1.0 changelog prose was
    found on this page -- the 1.1.0 badge appears to be the SAME 1.1.0
    number already tracked locally via binary-diff in
    cxr-l/release-notes.md, not a newer bump).
    RE-VERIFIED 2026-07-21 (fresh `firecrawl map` + scrape of /sdk and
    /sprite): CXR-L 1.0.4, CXR-M 1.1.0, 眼镜端裸机开发 1.0.0 -- all three
    UNCHANGED from 2026-07-18. Portal still lags Maven's client-l 1.1.0
    release (see maven.rokid.com entry) -- known, already-documented gap,
    no new action.
    RE-VERIFIED 2026-07-18: CXR-L 1.0.4 (updated 2026.06.25), CXR-M 1.1.0
    (updated 2026.04.01), 眼镜端裸机开发 1.0.0 (updated 2026.06.05) -- all
    three UNCHANGED from 2026-07-17, read directly off the /sdk page's new
    "SDK 选型速览" comparison table + card badges (see notes: SITE UI
    REBUILT). Bare-metal P1 (local pinned v0.0.1 2026-03-01) RE-CONFIRMED
    STILL OPEN.
    CXR-L 1.0.4 (official changelog, unchanged, re-verified 2026-07-16).
    CXR-M 1.1.0 (official changelog dated 2026-04-01, re-verified 2026-07-16
      via actions-click on the CXR-M tab -- no drift from cxr-m/intro.md).
    眼镜端裸机开发 (bare-metal) 1.0.0, updated 2026-06-05 (official
      changelog "重构文档" + new sample project, RE-CONFIRMED 2026-07-16
      via actions-click, selector
      "div.src-view-sdk-index-module__middle_title_div--2D3AR span:nth-child(3)".
      Local cxr-baremetal/ docs still pinned to v0.0.1 (2026-03-01) --
      STALE, P1 action item, open since 2026-07-14, still open.
      TRANSLATION ATTEMPTED 2026-07-16, BLOCKED: the actual restructured
      doc body lives behind a pure client-side React SPA at
      https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/13083daf77dd40bf84cf5c59711e987a.html
      -- WebFetch only returns the SPA shell/<title>; the JS bundle
      (static.rokidcdn.com/.../main.*.chunk.js), r.jina.ai and
      web.archive.org render-fallbacks, and open.rokid.com were all
      blocked by this session's egress proxy policy (403/connect_rejected)
      when the Translator tried them as workarounds. custom.rokid.com
      itself is a static OSS bucket with no queryable API. This P1 cannot
      be completed by an unattended run under current egress policy --
      needs either an actions-based Firecrawl scrape (as used
      successfully for the developerdoc.rokid.com/sdk changelog tabs) or
      a human/attended session with broader egress to fetch the real
      restructured content. Do not re-attempt via WebFetch/plain-fetch
      workarounds next cycle -- go straight to an actions-based Firecrawl
      scrape of the custom.rokid.com URL above, clicking through the
      in-page nav tree, or flag for human retrieval.)
  notes: |
    RE-VERIFIED 2026-08-03 (this Scout run): did not re-scrape /sprite this
    cycle (time-boxed; no drift has been found there in 5+ consecutive
    cycles). x-docs.rokid.com was spot-checked again this run (see "New
    sources discovered" below) -- content unchanged from 2026-07-25/07-29,
    still pending human scope triage, not registered, not translated.
    BARE-METAL P1 PARTIALLY ACTIONED THIS CYCLE: see last_known_version
    above and the custom.rokid.com entry below.
    RE-VERIFIED 2026-07-29 (this Scout run): direct cache-bypassed
    (--max-age 0) scrape of /sprite re-diffed word-for-word against
    yodaos/docs/sprite-overview.md: hardware spec table identical (SoC
    高通AR1, RAM 2GB, ROM 32GB, 210mAh battery, SONY IMX681 camera, FOV 30°,
    1500 nits, 480x640) and the FAQ section STILL shows the SAME 4
    CXR-M/CXR-S Q&As -- CONFIRMED NO CHANGE from 2026-07-25. x-docs.rokid.com
    was independently re-scraped this run (see "New sources discovered"
    below) and STILL renders the same substantial "Rokid Sprite Enterprise
    SDK" content first found live 2026-07-25 -- status unchanged, still
    pending human triage, not registered. Evidence:
    .firecrawl/developerdoc-sprite-fresh-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run): direct cache-bypassed
    (--max-age 0) scrape of /sprite re-diffed word-for-word against
    yodaos/docs/sprite-overview.md: hardware spec table identical, and
    the FAQ section STILL shows the SAME 4 CXR-M/CXR-S Q&As (not the
    7-Q&A CXR-L variant the local doc's own note describes) -- CONFIRMED
    NO CHANGE from 2026-07-24, this is a genuinely fresh independent
    re-read, not a carry-forward. rawHtml of /sdk re-checked for the
    "Rokid Glasses3" hardware card noted in prior cycles: CONFIRMED STILL
    PRESENT this run (grep of rawHtml finds both "Rokid Glasses3" and
    "x-docs.rokid.com" strings in the guide-menu-dropdown, alongside the
    YodaOS-Master entry) -- see x-docs.rokid.com note under "New sources
    discovered" below for a SIGNIFICANT status change found there this
    cycle (real SDK doc content now live, vs. previously undetermined).
    Evidence: .firecrawl/developerdoc-sprite-fresh-20260725.json,
    .firecrawl/developerdoc-sdk-rawhtml-20260725.html.
    RE-VERIFIED 2026-07-24 (this Scout run): direct scrape of /sprite
    (--only-main-content, live, no cache) re-diffed word-for-word against
    yodaos/docs/sprite-overview.md: hardware spec table identical (SoC,
    RAM/ROM, battery, camera, FOV, brightness, resolution all match) and
    the FAQ section still shows the SAME 4 CXR-M/CXR-S Q&As (CXR-M
    capabilities, CXR-M Android-only device support, CXR-S capabilities,
    no developer-mode requirement for CXR-M-only apps) -- CONFIRMED NO
    DRIFT, independently re-verified by this run (full scrape saved at
    .firecrawl/developerdoc-sprite.json). Did not observe the "Rokid
    Glasses3"/x-docs.rokid.com hardware card mentioned in prior cycles'
    notes on this run's /sdk scrape -- NOT independently confirmed
    present or absent this cycle (only --only-main-content markdown was
    captured, not rawHtml, so a card that requires rawHtml/different
    rendering could have been missed; do not treat its absence here as
    evidence it was removed).
    RE-VERIFIED 2026-07-18 (fresh live fetches, no cache reuse throughout):
    SITE UI REBUILT since 2026-07-17: /sdk no longer uses the old 3-tab
    layout (the "middle_title_div--2D3AR" tab selectors used on 07-16/07-17
    no longer exist in the DOM and a fresh actions-click against them fails
    with "Element not found"). The page now shows a single-view "SDK 选型
    速览" comparison table (运行位置/需要 AI App/公开获取/平台/最新版本 rows
    for all 3 SDKs side by side) followed by 3 expandable cards (CXR-L,
    CXR-M, 眼镜端裸机开发), each with its own version badge, "更新于" date,
    and a "查看文档 →" button -- no tab-click needed to see version numbers
    anymore. This is a genuine site restructuring, not just noise.
    Versions read directly off the new card badges: CXR-L 公开 1.0.4
    (更新于 2026.06.25), CXR-M 商务合作 1.1.0 (更新于 2026.04.01, "不可与
    Rokid AI App 在同一设备并行使用"), 眼镜端裸机开发 公开 1.0.0 (更新于
    2026.06.05) -- all three UNCHANGED from 2026-07-17. Bare-metal P1
    (cxr-baremetal/*.md still pinned "v0.0.1 (2026-03-01)") RE-CONFIRMED
    open via direct grep of local files this run.
    "查看文档 →" BUTTON TARGET RE-CAPTURED (independent reproduction of the
    07-17 finding, via a fresh executeJavascript action that patches
    window.open and clicks the button inside the 眼镜端裸机开发 card): still
    resolves to https://t.rokid.com/fwgr4uj5, which -- confirmed by directly
    scraping that short-link's rawHtml today -- redirects to
    window.websiteId="ff28c865a9634876be98cbc293588460", the SAME workspace
    hash found 07-17 (not a new one). This is a reproducible, non-one-off
    finding now (2 independent captures on 2 different days via 2 different
    click techniques).
    NEW HARDWARE CARD FOUND (not present in 07-16/07-17 rawHtml checks):
    a 4th hardware entry "Rokid Glasses3" linking to "YodaOS-Sprite
    Enterprise" at https://x-docs.rokid.com/docs/ -- a previously-unseen
    subdomain, NOT one of the 8 approved registry sources. NOT scraped
    further this run (no AskUserQuestion available in this unattended
    cycle). Flagged under "New sources discovered" below for human
    approval; do not treat as verified upstream content yet.
    /sprite FAQ RE-VERIFIED LIVE TODAY (fresh scrape, --wait-for 5000, no
    cache): live page shows the SAME 4 CXR-M/CXR-S Q&As word-for-word as
    local commit 2499a79c3's resynced yodaos/docs/sprite-overview.md FAQ
    section (CXR-M capabilities, CXR-M Android-only device support, CXR-S
    capabilities, no developer-mode requirement for CXR-M-only apps). The
    P2 drift flagged 2026-07-17 and actioned via commit 2499a79c3 IS NOW
    RESOLVED -- local matches live exactly as of today. NO FURTHER ACTION
    NEEDED on the FAQ item.
    /sprite device-spec table re-diffed against yodaos/docs/sprite-
    overview.md -- still exact match, no drift.
    YodaOS-Master tab/card remains present; skipped, out of scope.
    RE-VERIFIED 2026-07-16 (map: 3 URLs -- root, /sdk, /sprite, no
    /sitemap.xml this cycle, consistent with the 07-10/07-12 pattern, not
    the 07-14 one; confirms this is scrape-timing noise, not a real site
    change -- do not action).
    /sdk CXR-L tab (plain scrape, --only-main-content): byte-identical
    v1.0.4 changelog (2026-06-29) to 2026-07-14 check. No v1.1.0 changelog
    published yet despite Maven client-l:1.1.0 live since 2026-07-02 --
    already documented via binary-diff in cxr-l/release-notes.md, no
    further action.
    /sdk CXR-M tab (actions-click, selector
    "div...middle_title_div--2D3AR span:nth-child(2)"): tops out at
    v1.1.0 (2026-04-01), V1.1.0/V1.0.9/V1.0.4/V1.0.1 entries all present,
    matches cxr-m/intro.md exactly. No drift.
    /sdk 眼镜端裸机开发 tab (actions-click, span:nth-child(3), FIRST
    successful click today after an initial `text=...` selector attempt
    failed with "Element not found" -- fixed by inspecting rawHtml and
    using the real CSS class/nth-child selector): confirmed V1.0.0
    (2026-06-05) "重构文档"/"新增Sample工程源码" supersedes V0.0.1
    (2026-03-01). Local docs (cxr-baremetal/development-guide.md,
    key-broadcasts.md, audio-recording.md) still say "Doc version: v0.0.1
    (2026-03-01)" -- STALE, P1, carried forward from 2026-07-14, RE-
    CONFIRMED with fresh evidence today.
    /sprite spec table re-diffed against yodaos/docs/sprite-overview.md --
    still exact match, no drift.
    YodaOS-Master tab remains present on /sdk; skipped, out of scope.
    PRIOR CYCLE (2026-07-14): first-ever successful actions-click on all
    3 tabs; established the v1.0.0 bare-metal finding and CXR-M no-drift
    finding reconfirmed above.
    RE-VERIFIED 2026-07-17 (fresh map + plain scrape, --wait-for 5000/4000,
    no cache reuse): map still 3 URLs (root, /sdk, /sprite), unchanged.
    /sdk tab: CXR-L 1.0.4, CXR-M 1.1.0, 眼镜端裸机开发 1.0.0 -- all
    unchanged from 2026-07-16. Bare-metal P1 (cxr-baremetal/*.md still
    pinned "Doc version: v0.0.1 (2026-03-01)" vs upstream 1.0.0/2026-06-05)
    RE-CONFIRMED via direct grep of local files this run -- still open.
    CONCRETE EVIDENCE FOUND for the "/sprite FAQ" drift flagged in prior
    cycles: live scrape of /sprite's "Q&A 常见问题答疑" section today shows
    ONLY 4 short CXR-M/CXR-S-focused questions (CXR-M capabilities, CXR-M
    device support [Android-only], CXR-S capabilities, whether developer
    mode is required for CXR-M-only apps). This is COMPLETELY DIFFERENT
    from yodaos/docs/sprite-overview.md's FAQ section (7 Q&As, all
    CXR-L/bare-metal-focused: CXR-L capabilities, CustomView vs CustomApp,
    link-ready/session construction, environment prerequisites, custom
    commands, bare-metal comparison, bare-metal capabilities, install/
    debug). Zero topic overlap between the two. Local doc's own note
    (sprite-overview.md line 78) claims "the earlier CXR-M/CXR-S Q&As have
    been replaced by" the CXR-L content, dated 2026-06-08 -- but the LIVE
    page today serves the CXR-M/CXR-S variant, i.e. the opposite of what
    the local doc describes. This is genuine, concrete, structural content
    drift (P2), not vague noise -- see .firecrawl/developerdoc-sprite-
    current.md for full scrape evidence. NOTE: this specific "/sprite FAQ"
    item is NOT mentioned anywhere in this registry's notes for 2026-07-14
    or 2026-07-16 (those cycles only checked the /sprite device-spec
    table, not the FAQ Q&A block, and reported "still exact match, no
    drift" for the spec table only) -- Scout cannot corroborate a claim of
    "4 consecutive cycles flagged, never actioned" for the FAQ block
    specifically from this file's history; that framing did not originate
    from this registry's own notes. Flagging as newly-evidenced P2 as of
    today regardless of the disputed cycle count.
    BARE-METAL P1 -- RETRIEVAL BREAKTHROUGH 2026-07-17 (corrected from an
    earlier same-cycle false negative): a second attempt this cycle
    confirmed real v1.0.0 body content for 眼镜端裸机开发 IS retrievable --
    via the Firecrawl REST API called directly (curl, through the
    required proxy) plus an executeJavascript click-through of the
    doc's antd Tree nav (window.open patched to capture the target URL).
    The "查看文档" link now resolves through https://t.rokid.com/fwgr4uj5
    to a NEW custom.rokid.com workspace,
    https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/cn/index.html
    -- superseding the old 57e35cd3ae294d16b1b8fc8dcbb1b7c7-hosted page
    this doc previously cited. This is a genuine site rebuild (matches
    "重构文档"), not just a version-string bump. Confirmed 8 sub-pages:
    简介, 快速开始, Sample工程与页面说明, 眼镜UI设计规范, 按键与佩戴折叠,
    原始音频, 拍照, 录像, IMU与传感器. Only 3 map to existing local docs
    (简介+快速开始 -> development-guide.md, 按键与佩戴折叠 ->
    key-broadcasts.md, 原始音频 -> audio-recording.md); the other 4 --
    Sample工程与页面说明, 眼镜UI设计规范, 拍照, 录像, IMU与传感器 -- have
    NO local doc and are NEW P0 CANDIDATES for a future cycle. 按键与
    佩戴折叠 also appears to have grown scope beyond the current
    key-broadcasts.md (wear/fold-state broadcasts, triple-click
    Bluetooth pairing, extra-long-press) -- may need a rename/scope
    discussion rather than a straight refresh. Translation itself is
    STILL BLOCKED, but for a different reason than reported earlier same
    day: the `baoyu-translate` skill's SKILL.md is not installed in this
    environment (only .baoyu-skills/baoyu-translate/EXTEND.md, the
    glossary config, is present) -- a session/tooling setup gap, not a
    content-availability gap. Raw Chinese source was staged in that
    session's ephemeral scratchpad and intentionally not committed (repo
    policy against committing untranslated upstream copies); it will
    need to be re-fetched once the skill gap is fixed. See the
    REFRESH-PENDING comments in cxr-baremetal/*.md for the corrected
    per-file notes.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-21
  notes: |
    RE-VERIFIED 2026-08-21 (this run -- direct scrape of the CXR-L workspace,
    84feb39f8ef141b0ad0326f902ab881f/pc/cn/): still returns OSS NoSuchKey
    (RequestId 6A87C0622851783339216D77, HTTP 404). Not re-checked this
    cycle: the ff28c865/bare-metal workspace (see 2026-08-03 note below,
    still the most recent check for that workspace).
    RE-VERIFIED 2026-08-03 (this Scout run -- direct scrape of the ff28c865
    workspace's 简介 (intro) page, `--format rawHtml`): first attempt at
    the default `--wait-for` (implicitly lower) rendered only the ~3.5KB SPA
    shell, same rendering-timing miss as most prior cycles; a retry with
    `--wait-for 12000` rendered the FULL 14KB article body, word-for-word
    IDENTICAL to the 2026-07-18/07-25/07-29 findings (定位/运行环境/系统与
    约束/设备能力概览/建议阅读顺序/文档索引 sections, GlassesBareDevSample
    sample, minSdk 31/targetSdk 36) -- NO CONTENT DRIFT, but this is the
    FIRST cycle this content was actually LANDED in the repo: translated
    into cxr-baremetal/development-guide.md's new Scope/Runtime
    Environment/System and Constraints/Device Capability
    Overview/Suggested Reading Order/Sample Project sections, with the Doc
    version line bumped v0.0.1 (2026-03-01) -> v1.0.0 (2026-06-05). Did NOT
    re-attempt click-through to the 7 sibling sub-pages (快速开始, Sample
    工程与页面说明, 眼镜UI设计规范, 按键与佩戴和折叠, 原始音频, 拍照, 录像,
    IMU与传感器) this cycle -- every documented technique across ~6 prior
    cycles (plain scrape at various --wait-for, curl+executeJavascript,
    dispatchEvent MouseEvent sequences on the antd Tree nav) has returned
    only the identical 简介 body, and the "下一篇" (next) link at the bottom
    of the rendered HTML has no href (pure JS click handler) -- treating
    this as still-unsolved via non-interactive Firecrawl rather than
    re-running already-falsified techniques. key-broadcasts.md and
    audio-recording.md remain pinned to v0.0.1 pending retrieval of their
    specific sub-pages. Evidence: .firecrawl/custom-ff28c865-intro-20260803.html
    (SPA-shell miss), .firecrawl/custom-ff28c865-intro-retry-20260803.html
    (full body, 14183 bytes), .firecrawl/custom-ff28c865-map-20260803.json
    (map still only surfaces the 简介 URL, same as every prior cycle).
    RE-VERIFIED 2026-07-29 (this Scout run -- fresh, uncached fetches): root
    path https://custom.rokid.com/prod/rokid_web/ STILL returns HTTP 415
    Unsupported Media Type (cacheState "miss") -- unchanged. Fresh
    `firecrawl map` of custom.rokid.com root returned 24 links; the
    57e35cd3.../c88be4bc.../323825ad... hashes are all still present with
    the same in-scope/out-of-scope split as 07-25, PLUS one hash not
    explicitly enumerated in the 07-25 baseline note text --
    e4df4078436f44c286f36d06ab2c5404 (single mobile/cn link, generic "Rokid
    AR Platform" title only) -- NOT independently scraped or scope-
    determined this run; flagging as a low-priority WATCH item, most likely
    map crawl-coverage variance (this source has repeatedly shown link-count
    variance across cycles) rather than a new site. NOTABLE: the bare-metal
    workspace hash ff28c865a9634876be98cbc293588460 did NOT appear at all in
    today's 24-link map (present in every prior cycle's map) -- however its
    known intro-page URL is still directly reachable (confirmed by 3 direct
    scrape attempts below), so this is treated as map-coverage variance, not
    evidence the workspace was removed. BARE-METAL 简介 (intro) PAGE: 3
    direct scrape attempts this run (default, --wait-for 9000, --wait-for
    12000, all --max-age 0) each returned only a small (~3.5KB) SPA-shell
    response with the loading spinner still active -- FULL ARTICLE BODY DID
    NOT RENDER this cycle, unlike 2026-07-25's successful 14KB capture.
    COULD NOT independently word-for-word reconfirm the article body this
    run -- treat as "could not verify body content this cycle," not as
    confirmed-unchanged or confirmed-changed. The on-screen "版本" badge DID
    render on the 3rd attempt (--wait-for 12000) and reads "版本1.0.0",
    matching developerdoc.rokid.com/sdk's card badge -- this RESOLVES the
    2026-07-25 WATCH item (whether the visible badge says 1.0.0 or something
    else related to the separate `window.relatedVersion = "1.0.6"` JS
    variable, which is again present in this run's rawHtml and again NOT
    treated as an SDK version). The 5 candidate no-local-doc P0 sub-pages
    (Sample工程与页面说明, 眼镜UI设计规范, 拍照, 录像, IMU与传感器) are
    RE-CONFIRMED still absent from cxr-baremetal/ -- unchanged standing item,
    not re-actioned. Evidence: .firecrawl/custom-root-20260729.json,
    .firecrawl/custom-map-20260729.json,
    .firecrawl/custom-ff28c865-intro-fresh-20260729.html (attempt 1),
    .firecrawl/custom-ff28c865-intro-fresh2-20260729.html (attempt 2, 9s),
    .firecrawl/custom-ff28c865-intro-fresh3-20260729.html (attempt 3, 12s,
    badge captured).
    RE-VERIFIED 2026-07-25 (this Scout run -- fresh, uncached fetches):
    root path https://custom.rokid.com/prod/rokid_web/ STILL returns HTTP
    415 Unsupported Media Type (cacheState "miss", contentType
    application/octet-stream) -- unchanged, genuinely re-fetched (not
    cached). Fresh `firecrawl map` of custom.rokid.com root returned 24
    links across the SAME 4 workspace hashes as the 2026-07-24 baseline:
    ff28c865a9634876be98cbc293588460 (bare-metal), 57e35cd3ae294d16b1b8fc8dcbb1b7c7
    (CXR-M/CXR-S), c88be4bcde4c42c0b8b53409e1fa1701 (UXR2.0/UXR3.0/JSAR,
    out of scope, not scraped further), 323825adf4914c21be8a0d5fe7b8a9e5
    (mixed-scope nav page, not clicked through). 84feb39f8ef141b0ad0326f902ab881f
    (CXR-L-specific hash) STILL absent from today's map -- consistent
    every cycle since 2026-07-10, still treated as non-actionable.
    BARE-METAL 简介 (intro) PAGE RE-VERIFIED LIVE TODAY: a fresh
    `firecrawl scrape --format rawHtml --wait-for 6000 --max-age 0` of
    https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/cn/a38e99d9ab364a13abd506f6cfed2856.html
    rendered the full 14KB article body (title "Rokid Glasses 裸机开发简介")
    -- content word-for-word matches the 2026-07-18 finding: 定位/运行环境/
    系统与约束/设备能力概览/建议阅读顺序/文档索引 sections, same suggested
    reading order (简介→快速开始→UI规范→Sample工程→按键与佩戴折叠→原始音频
    →拍照→录像→IMU与传感器), NO CONTENT DRIFT. ONE AMBIGUOUS SIGNAL FOUND:
    this run's rawHtml contains a JS variable `window.relatedVersion =
    "1.0.6"` that was NOT reported in any prior cycle's notes for this
    page (prior cycles report a "版本1.0.0" on-screen badge, which was NOT
    independently reproduced by this run's rawHtml capture -- the visible
    "版本" label element was found but its value is populated client-side
    and wasn't captured in static rawHtml this time). Do NOT treat
    "relatedVersion 1.0.6" as a confirmed SDK version bump -- it may be an
    unrelated internal doc-editor/schema version, not the 眼镜端裸机开发
    SDK version (which developerdoc.rokid.com/sdk's card badge, re-verified
    fresh this same run, still shows as 1.0.0). Flagging as a WATCH item
    for next cycle: try to re-capture the on-screen "版本" badge value with
    a longer --wait-for or a --actions click, to confirm whether it says
    1.0.0 (matching the SDK card) or something else. The 5 candidate
    no-local-doc P0 sub-pages (Sample工程与页面说明, 眼镜UI设计规范, 拍照,
    录像, IMU与传感器) are RE-CONFIRMED still absent from cxr-baremetal/ --
    see report. Evidence: .firecrawl/custom-root-20260725.json,
    .firecrawl/custom-map-20260725.json,
    .firecrawl/custom-ff28c865-intro-fresh-20260725.html.
    RE-VERIFIED 2026-07-24 (this Scout run): root path
    https://custom.rokid.com/prod/rokid_web/ STILL returns HTTP 415
    Unsupported Media Type (confirmed live, contentType
    application/octet-stream) -- unchanged.
    `firecrawl map` of https://custom.rokid.com (limit 50) surfaced 4
    distinct workspace hashes:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M/CXR-S, in-scope): CONFIRMED
      ALIVE AGAIN -- map returned real page titles ("Rokid Glasses 设备
      连接与管理", "设备连接管理") with statusCode 200 on direct scrape,
      REVERSING the 2026-06-30 baseline note that this hash returned
      NoSuchKey. HOWEVER, this run's own direct scrapes of two of its
      pages (--only-main-content, --wait-for up to 5000ms) rendered only
      the antd-Tree-nav SPA shell ("版本 ... 文档", ~35 bytes), not the
      article body -- so CXR-M/CXR-S body-content drift could NOT be
      confirmed or denied by this run; treat as "could not verify body
      content," not as "no drift."
    - ff28c865a9634876be98cbc293588460 (bare-metal workspace -- a THIRD
      hash not part of the registry's originally-tracked pair): CONFIRMED
      ALIVE and its 简介 (intro) page's real article body WAS
      successfully retrieved this run via a plain `firecrawl scrape
      --wait-for 3000` (no click-through/actions needed for this specific
      URL). Content: "Rokid Glasses 裸机开发简介", 版本1.0.0 -- matches
      developerdoc.rokid.com/sdk's 眼镜端裸机开发 badge (1.0.0, 更新于
      2026.06.05) exactly, no version bump. Confirmed document map: 简介,
      快速开始, Sample工程与页面说明, 眼镜UI设计规范, 功能开发 (>按键与
      佩戴折叠, 原始音频, 拍照, 录像, IMU与传感器). Sibling sub-pages
      (everything except 简介) were NOT independently fetched this run --
      their specific documentId/URL values are not exposed by `map` and
      require in-page client-side navigation this run's scrape attempts
      did not perform. Local repo maps: 简介+快速开始 -> development-
      guide.md, 按键与佩戴折叠 -> key-broadcasts.md, 原始音频 ->
      audio-recording.md (all exist). Sample工程与页面说明, 眼镜UI设计
      规范, 拍照, 录像, IMU与传感器 have NO local doc -- P0 (see report).
    - c88be4bcde4c42c0b8b53409e1fa1701: confirmed present in the map,
      content is entirely UXR2.0/UXR3.0/JSAR (spatial-computing SDKs) --
      OUT OF SCOPE per CLAUDE.md, correctly NOT scraped further.
    - 323825adf4914c21be8a0d5fe7b8a9e5: generic multi-product nav page
      mixing Rokid Glasses (in-scope) with YodaOS-Master/AR Lite/AR
      Studio (out of scope) -- not clicked through further.
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L-specific hash): searched
      for directly by hash and by "CXR-L" keyword this run -- NOT found
      in either search's results. Could not confirm current live status
      one way or the other this cycle; treat as "could not verify," not
      as confirmed-resolved.
    RE-VERIFIED 2026-07-21: `firecrawl map` (limit 500, 29 URLs) and 5 scrapes
    across the 3 known workspace hashes -- device-connection, CXR-S intro,
    CXR-L intro, and the ff28c865 bare-metal set all match already-documented
    local content, no drift beyond what 2026-07-17/07-18 already logged.
    CORRECTION to a same-day process note: earlier in this session, a Scout
    invocation flagged this file as possibly "tampered" because it saw
    content about a client-l 1.1.0 POM/dependency change that it hadn't
    itself produced via a tool call. That content was NOT an injection --
    it matches this repo's own commit ac1138a74 (2026-07-04), which already
    documented the 1.1.0 binary diff. The false alarm traces to this
    session running multiple overlapping Scout/leader agent invocations
    against a stale local branch (built off an old `main` rather than this
    branch's actual tip) while recovering from an earlier stall; each
    lacked the other's context. Independently re-verified today via direct
    `curl` + `unzip` + `diff` of the real client-l 1.0.4/1.1.0 AAR and POM
    from maven.rokid.com (bypassing all agent output): AAR size 70,543 ->
    1,286,574 bytes, classes.jar 66 -> 160 classes, 5 new JNI .so libs per
    ABI, and the POM drops the cxr-service-bridge dependency in favor of
    kotlinx-coroutines-android:1.6.4 -- all exactly matching what
    cxr-l/release-notes.md and cxr-l/api-reference.md already document.
    No content correction needed; this is a process note for future
    cycles: don't re-launch a fresh Scout without checking whether one is
    already in flight on this branch.
    MAJOR UPDATE 2026-07-18 (fresh live fetches, --max-age 0 where
    supported, no cache reuse):
    BARE-METAL 简介 (intro) BODY CONTENT RE-CONFIRMED RETRIEVABLE, and via
    a SIMPLER, more reliable method than 07-17's curl+executeJavascript
    approach: a plain `firecrawl scrape --format rawHtml --wait-for 6000`
    of
    https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/cn/a38e99d9ab364a13abd506f6cfed2856.html
    renders the FULL real article (14KB rawHtml incl. antd Tree nav AND
    the "#docInfo" article body) -- no click-through or executeJavascript
    needed for this specific page. (Note: `--only-main-content` / plain
    markdown format still fails to extract the body -- markdown/readability
    conversion drops it even though rawHtml has it; use rawHtml + manual
    HTML parsing for this site.) Confirmed real Chinese content, title
    "Rokid Glasses 裸机开发简介": covers 定位 (scope: on-glasses standard
    Android dev, no phone-side SDK needed), 运行环境 (YodaOS-Sprite Android
    12 API 31 / Go, app minSdk 31, Sample targetSdk 36, Android Studio,
    debug via dedicated dev line + Rokid AI App ADB toggle), 系统与约束
    (Android Go constraints, 480x640 screen), 设备能力概览 (table mapping
    480x640 display/按键+触控板/8-channel raw audio/拍照录像/6-axis IMU to
    their respective doc sections), and 建议阅读顺序 (suggested reading
    order: 简介→快速开始→UI规范→Sample工程→按键与佩戴折叠→原始音频→拍照→
    录像→IMU与传感器). This is genuine v1.0.0 upstream content -- full text
    saved at .firecrawl/custom-ff28c865-rawhtml-0718.html (and a repeat
    capture at .firecrawl/custom-ff28c865-quickstart3-0718.html, which
    despite the filename is the SAME 简介 content -- see below). Maps to
    cxr-baremetal/development-guide.md's topic.
    CLICK-THROUGH TO SIBLING PAGES STILL BLOCKED: 3 independent
    executeJavascript techniques were tried against the antd Tree node
    titled "快速开始" (plain .click(), a dispatchEvent MouseEvent sequence
    on the title span, and the same dispatchEvent sequence on the parent
    .ant-tree-treenode) -- ALL THREE returned the IDENTICAL 简介 content
    with zero navigation, confirmed by diffing the resulting rawHTML's
    <h1> each time. This means only the tree's default/first-loaded node
    is retrievable via non-interactive Firecrawl scraping this cycle; true
    multi-node navigation (needed for 快速开始 and the 5 no-local-doc P0
    candidates: Sample工程与页面说明, 眼镜UI设计规范, 拍照, 录像, IMU与
    传感器) remains UNSOLVED via the firecrawl-cli's `--actions`/
    executeJavascript interface. Do not claim these 5 pages' content was
    retrieved -- it was NOT, only 简介's content is confirmed live text.
    AI 流程交互 PAGE RETRIEVED FOR THE FIRST TIME, WITH A CORRECTION TO THE
    07-17 FINDING: raising --wait-for to 10000 (vs. 6000/unspecified in
    prior attempts) finally rendered real body content for
    https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/index.html?documentId=4e088caa11e84b97b381a145bbb93379
    (rawHtml jumped from ~3.5KB SPA-shell-only to 13.7KB with full
    #docInfo body -- see .firecrawl/custom-57e35cd3-aiflow-rawhtml3-0718.html).
    HOWEVER the rendered content is NOT "AI flow interaction" as the
    07-17 title label claimed -- nowhere does "AI流程交互" appear in the
    live-rendered page. The actual content is a CXR-M SDK integration
    guide (配置Maven仓库/依赖导入/权限申请 -- Maven repo config, dependency
    import, permission declarations), filed under the workspace's "SDK接入"
    top-level nav category (breadcrumb: prev="简介", next="设备连接"), and
    it references ANCIENT client-m SNAPSHOT builds
    ("com.rokid.cxr:client-m:0.0.3-20250310.072635-3" and
    "...0.0.5-20250415.064355-3", i.e. March/April 2025 -- over a year
    stale vs. today's date and vs. Maven's current client-m release/latest
    1.2.2). Local cxr-m/sdk-integration.md ALREADY documents this exact
    Maven-config/dependency/permission flow using the CURRENT
    client-m:1.2.2 -- so this page is NOT a P0 content gap, it is a STALE/
    ORPHANED page on Rokid's own site that is already superseded by local
    docs. RECOMMENDATION: close out the "AI流程交互 P0 candidate" opened
    2026-07-17 -- downgrade to "no action, page content already covered
    and upstream itself is more stale than local," and correct the title
    label (it is not an AI-flow-interaction page). The doc-version badge
    "0.0.5" seen 07-17 does match what's on this page, confirming this is
    the same page, just mislabeled/misdescribed previously, not a
    different/moved page.
    `firecrawl map` of custom.rokid.com root today surfaced a RICHER link
    set than 07-17's "16 links, same 3 hashes" -- now includes explicit
    documentId-based URLs for BOTH ff28c865... (bare-metal) and 57e35cd3...
    (CXR-M/CXR-S) workspaces, plus 2 pages that render with REAL titles in
    the map's own metadata (proving they're not stuck on the SPA shell):
    "Rokid Glasses 设备连接与管理" at .../57e35cd3.../9d9dea4799ca4dd2a1176fedb075b6f2.html
    and "设备连接管理" at .../57e35cd3.../2786298057084a82b170bf725aef6b5d.html
    -- both already corroborated against cxr-m/device-connection.md with
    no drift (per the 07-16 finding); their continued presence + real
    titles today reconfirms that finding, no new action. By contrast, the
    AI流程交互/SDK接入 URL's own map entry still shows the generic
    "Rokid AR Platform" title/description (not "SDK接入" or its real
    heading) -- map-time rendering is inconsistent with a longer-wait
    direct scrape for that specific page, worth remembering for future
    cycles (use --wait-for 10000, not the default map crawl, to actually
    read that page's content).
    84feb39f8ef141b0ad0326f902ab881f (CXR-L-specific hash) STILL absent
    from today's map -- consistent every cycle since 2026-07-10, still
    treated as resolved/non-actionable (content believed consolidated into
    developerdoc.rokid.com/sdk's CXR-L card, confirmed live at 1.0.4
    today).
    RE-VERIFIED 2026-07-16. Root path still returns HTTP 415 Unsupported
    Media Type (confirmed live, statusCode 415, contentType
    application/octet-stream). `firecrawl map` of the root succeeds: 25
    links across the same 3 workspace hashes as 2026-07-10 through 07-14
    (57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M/CXR-S in-scope;
    c88be4bcde4c42c0b8b53409e1fa1701 = UXR2.0/UXR3.0/JSAR, OUT OF SCOPE,
    not scraped further; 323825adf4914c21be8a0d5fe7b8a9e5 = mixed-scope nav
    page, only the Rokid Glasses entry point is in-scope, not clicked
    through). The CXR-L-specific hash 84feb39f8ef141b0ad0326f902ab881f is
    STILL absent from the map (consistent every cycle since 2026-07-10) --
    working theory is its content consolidated into
    developerdoc.rokid.com/sdk's CXR-L tab, which is confirmed live and
    actively updated (v1.0.4). Treat as resolved/non-actionable unless a
    future cycle finds evidence otherwise.
    BREAKTHROUGH 2026-07-16: successfully rendered real body content from
    a 57e35cd3... page for the first time (all prior cycles only got an
    SPA-shell placeholder). Used `--wait-for 6000` (no actions needed this
    time) on
    https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/9d9dea4799ca4dd2a1176fedb075b6f2.html
    -- page shows "版本1.0.1", covers CXR SDK/Glasses architecture, device
    connection & management, custom scene interaction, and "Rokid Assist
    Service" (录音/拍照/录像/飞传 = recording/photo/video/file-transfer).
    Diffed against cxr-m/device-connection.md and cxr-m/intro.md (which
    already documents "Rokid Assist Service" incl. Feichuan/file transfer)
    -- CONTENT MATCHES, no drift, no action item. This closes out the
    "NOT independently content-verified" gap noted in prior cycles for
    this hash.
    PRIOR CYCLES (2026-06-30 through 07-14): status progressed from fully
    BROKEN (06-30, NoSuchKey on both known workspace hashes) to PARTIALLY
    REACHABLE (07-10 onward, map succeeds, body content unrenderable until
    today's --wait-for 6000 breakthrough).
    RE-VERIFIED 2026-07-17: fresh `firecrawl map` of root, 16 links,
    same 3 workspace hashes (57e35cd3.../84feb39f absent/c88be4bc out-
    of-scope/323825ad mixed-nav) as every check since 2026-07-10. No
    change.
    NEW CANDIDATE P0 FOUND 2026-07-17: a page within the 57e35cd3...
    (CXR-M/CXR-S, in-scope) workspace titled "Rokid Glasses AI 流程交互"
    (AI flow interaction), doc version 0.0.5, at
    https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/index.html?documentId=4e088caa11e84b97b381a145bbb93379
    -- no local doc exists for this topic under cxr-m/ or cxr-s/. NOT
    ACTIONABLE THIS CYCLE: scrape only rendered the SPA-shell nav
    (section headers 简介/SDK接入/功能开发/版本历史), not the article
    body, even with extended --wait-for; see .firecrawl/custom-ai-flow-
    current.md and .firecrawl/custom-ai-flow-retry.md. Same rendering
    blocker as the bare-metal doc body (needs an actions-based
    click-through scrape, or human/attended retrieval). DO NOT
    fabricate this page's content. Carry forward as an open P0 for the
    next cycle; try actions-click on the in-page nav tree next time.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-29
  notes: |
    RE-VERIFIED 2026-07-29 (this Scout run -- cache-bypassed --max-age 0
    scrape of root, plus a fresh live `firecrawl map`, limit 50): direct
    scrape confirms sourceURL developer.rokid.com redirects to finalURL
    https://open.rokid.com/ (statusCode 200), rendering the identical
    "AIUI: The Next Frontier" homepage; Rokid Glasses hardware card links
    Docs to open.rokid.com/sdk?lang=en; YodaOS-Master / AR Lite / AR Studio
    cards present, explicitly skipped as out of scope. Fresh map returned
    17 links, same legacy GitBook paths as every check since 2026-07-16
    (rokidos-linux-docs, rokid-homebase-docs, skill/, 2-RokidDocument,
    8-app/alliance, 5-enableVoice/rokid-vsvy-sdk-docs, etc.) -- all
    out-of-scope legacy voice-assistant/HomeBase/Skill-platform content. No
    new paths, no change. Evidence: .firecrawl/developer-rokid-20260729.json,
    .firecrawl/developer-rokid-map-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run -- cache-bypassed --max-age 0
    scrape of root, plus a fresh live `firecrawl map`, limit 50): direct
    scrape confirms sourceURL developer.rokid.com redirects to finalURL
    https://open.rokid.com/ (statusCode 200, cacheState null -- genuinely
    fresh), rendering the identical "AIUI: The Next Frontier" homepage;
    Rokid Glasses hardware card links Docs to open.rokid.com/sdk?lang=en;
    YodaOS-Master / AR Lite / AR Studio cards present, explicitly skipped
    as out of scope. Fresh map returned 17 links, same legacy GitBook
    paths as every check since 2026-07-16 (rokidos-linux-docs, rokid-
    homebase-docs, skill/, 2-RokidDocument, 8-app/alliance, 5-enableVoice/
    rokid-vsvy-sdk-docs, etc.) -- all out-of-scope legacy voice-assistant/
    HomeBase/Skill-platform content. No new paths, no change. Evidence:
    .firecrawl/developer-rokid-20260725.json,
    .firecrawl/developer-rokid-map-20260725.json.
    RE-VERIFIED 2026-07-24 (this Scout run): direct scrape
    (--only-main-content --json) confirms sourceURL developer.rokid.com
    redirects: finalURL https://open.rokid.com/ (statusCode 200,
    cacheState hit), rendering the "Rokid Open Platform" / "AIUI: The
    Next Frontier" homepage. Rokid Glasses hardware card links Docs to
    open.rokid.com/sdk?lang=en; YodaOS-Master, Rokid AR Lite, Rokid AR
    Studio, and AIUI Studio cards present and explicitly skipped as out
    of scope. Did NOT re-run the legacy-GitBook `firecrawl map` this
    cycle (time-boxed) -- no evidence of change, but that specific check
    was not repeated today; carry forward the 2026-07-21 map finding as
    unconfirmed-but-presumed-unchanged rather than re-verified.
    RE-VERIFIED 2026-07-21 (fresh `firecrawl map`): same 15 legacy GitBook
    paths as prior cycles (rokidos-linux-docs, rokid-homebase-docs, skill/,
    2-RokidDocument, etc.) -- legacy Rokid Skill/Voice smart-speaker
    platform, different product family from YodaOS-Sprite. No in-scope
    content, no change.
    RE-VERIFIED 2026-07-18 (fresh direct scrape --only-main-content
    --wait-for 3000, plus fresh map --limit 20, no cache): direct scrape
    still renders the identical "AIUI: The Next Frontier" homepage (EN
    locale this run) -- Rokid Glasses hardware card links Docs to
    open.rokid.com/sdk?lang=en; YodaOS-Master, Rokid AR Lite, Rokid AR
    Studio cards present, skipped as out of scope. Fresh map returned the
    same 16 legacy GitBook paths as 07-17 (rokidos-linux-docs, rokid-
    homebase-docs, 8-app/alliance, 2-RokidDocument/1-SkillsKit, etc.) --
    all out-of-scope legacy voice-assistant/HomeBase/Skill content. No
    change, no action.
    RE-VERIFIED 2026-07-16. Direct scrape of the root (--wait-for 3000)
    renders the identical "AIUI: The Next Frontier" homepage as
    ar.rokid.com/sdk and developer.rokid.com's prior checks -- byte-for-
    byte same hardware cards, same Docs links to open.rokid.com/sdk?lang=en.
    YodaOS-Master / AR Lite / AR Studio cards present -- skipped, out of
    scope. NOTE: a `firecrawl map` of this same root (as opposed to a
    direct scrape) surfaces a *different* set of URLs -- 15 legacy GitBook
    pages under /docs/... (skill development, rokidos-linux-docs,
    rokid-homebase-docs, etc.) for Rokid's older voice-assistant/speaker
    platform. These are old indexed paths that still resolve directly but
    are not reachable by navigating the current SPA UI; map discovers them
    via crawl/sitemap traversal rather than rendering. All content found
    is legacy Speech/HomeBase/Skill product docs -- OUT OF SCOPE (not
    Sprite/Glasses/CXR). No in-scope content. No action items. Low
    priority as a distinct source (mirrors developerdoc.rokid.com /
    open.rokid.com for the in-scope surface).
    RE-VERIFIED 2026-07-17: fresh map, 16 URLs, same legacy GitBook
    paths (rokidos-linux-docs, rokid-homebase-docs, skill/, docs/2-
    RokidDocument etc.). All out-of-scope legacy voice-assistant
    content. No change, no action.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-21
  last_known_version: |
    RE-VERIFIED 2026-08-21 (this run -- direct curl of maven-metadata.xml
    and per-version .pom files for client-l 1.0.4/1.1.0/1.1.1, client-m,
    and cxr-service-bridge; also re-downloaded and re-diffed all three
    client-l AARs independently, plus a fresh javap -p pass against
    client-l:1.1.1's classes.jar): NO NEW RELEASE since the 2026-08-19
    check. client-l: <release> still 1.1.1, <lastUpdated> still
    20260814092031 (2026-08-14); AAR still 171,369 bytes, still lacks the
    5 bundled native .so libs and the com.rokid.cxr root package, still
    depends on cxr-service-bridge:1.0-20260715.121510-107. client-m:
    release/latest still 1.2.2, lastUpdated still 20260608030211.
    cxr-service-bridge: release/latest still "1.0", lastUpdated still
    20260728074326 -- unchanged since 08-19, corroborating (not yet
    independently proven) the prior cycle's guess that this build is the
    same 1.0-20260715.121510-107 now pinned by client-l:1.1.1. One
    correction from this cycle's re-verification: the CxrSession API's
    SessionConfig write-up in cxr-l/api-reference.md was missing its 7th
    constructor field, viewIconData: String? (getViewIconData(), sitting
    between viewData and glassesActivityName) -- confirmed present via
    javap -p against the live 1.1.1 classes.jar and added. No other
    discrepancies found in a full re-check of the CxrSession API section
    against the current AAR. Everything else already documented in the
    2026-08-19 entry below remains accurate and was not re-actioned.
    RE-VERIFIED 2026-08-19 (this run -- direct curl of maven-metadata.xml
    for all 3 artifacts; no Firecrawl-class JS-rendering tool available this
    cycle, so the browse UI wasn't used, only the raw metadata XML and .pom
    files, fetched directly): client-l: <release> moved 1.1.0 -> 1.1.1,
    <latest> is now 1.2.X-SNAPSHOT, <lastUpdated> moved 20260718072455 ->
    20260814092031 (2026-08-14) -- a NEW tagged release since the 08-03
    baseline. AAR-embedded BuildConfig.BUILD_TIME confirms "2026-08-14
    15:43:24". Downloaded and diffed client-l-1.1.0.aar (1,286,574 bytes)
    vs client-l-1.1.1.aar (171,369 bytes, -86.7%) plus both .pom files:
    1.1.1 removes the com.rokid.cxr root package (Caps/CXRSocketProtocol/
    CXRServiceBridge/RLog) and all 5 native .so libs that 1.1.0 had
    erroneously bundled (see this file's prior entry below), and restores
    cxr-service-bridge as an external POM dependency (now pinned to
    1.0-20260715.121510-107, still release "1.0"). kotlin-stdlib bumped
    1.6.0->1.9.0, kotlinx-coroutines-android bumped 1.6.4->1.9.0. The
    com.rokid.cxr.session API added in 1.1.0 is unchanged (re-confirmed via
    javap -p). ACTIONED AS P1 THIS CYCLE -- see cxr-l/release-notes.md and
    cxr-l/api-reference.md (both already had a 1.1.0 write-up from a prior
    cycle; this cycle added a superseding 1.1.1 entry rather than
    duplicating the 1.1.0 analysis). client-m: release/latest STILL 1.2.2,
    lastUpdated STILL 20260608030211 -- unchanged since 2026-06-09, portal
    still on 1.1.0 per developerdoc.rokid.com entry. cxr-service-bridge
    (top-level artifact metadata, not the client-l dependency pin above):
    release/latest STILL 1.0, but lastUpdated is now 20260728074326
    (2026-07-28) -- newer than the 08-03 baseline's most recent noted bump;
    consistent with the established re-indexing/checksum-noise pattern for
    this artifact UNLESS it corresponds to the same 1.0-20260715.121510-107
    build now referenced by client-l:1.1.1's .pom (plausible but not
    independently re-verified via browse-listing byte diff this cycle --
    time-boxed). Not treated as a P1 on its own (no release version bump).
    RE-VERIFIED 2026-08-03 (this Scout run -- Firecrawl scrape of
    maven-metadata.xml for client-l and client-m; cxr-service-bridge not
    re-checked this cycle, low priority given its lastUpdated churn has
    been repeatedly ruled out as re-indexing noise): client-l
    release=1.1.0, latest=1.1.X-SNAPSHOT, lastUpdated=20260718072455 --
    IDENTICAL to 07-29, no new tagged release since 07-18. client-m
    release/latest STILL 1.2.2, lastUpdated STILL 20260608030211 --
    unchanged. Both already fully documented locally
    (cxr-l/release-notes.md covers 1.1.0 via binary-diff; cxr-m/intro.md
    documents through 1.1.0, the gap to Maven's 1.2.2 already noted). No P1
    action needed. Evidence: .firecrawl/maven-client-l-20260803.md,
    .firecrawl/maven-client-m-20260803.md.
    RE-VERIFIED 2026-07-29 (this Scout run -- WebFetch verbatim-XML prompts
    for all 3 artifacts): client-l: release=1.1.0, latest=1.1.X-SNAPSHOT,
    lastUpdated=20260718072455 -- IDENTICAL to 07-25, no new tagged release.
    client-m: release/latest STILL 1.2.2, lastUpdated STILL 20260608030211
    -- fully unchanged. cxr-service-bridge: release/latest STILL 1.0, but
    `lastUpdated` MOVED to 20260727100234 (2026-07-27) from the 07-25
    baseline's 20260723084719 (2026-07-23) -- a NEW bump within this check
    window. INVESTIGATED via a fresh browse-listing scrape of
    .../cxr-service-bridge/1.0/: cxr-service-bridge-1.0.aar itself is
    BYTE-IDENTICAL to every prior cycle's known state (Last Modified Thu Dec
    25 12:54:20 Z 2025, 1,076,548 bytes) -- only checksum sidecars
    (.aar.sha256 Jul 11, .module.md5/.pom.md5 May 14, .module Jun 02) show
    recent timestamps. CONFIRMED re-indexing/checksum-regeneration noise,
    consistent with the same pattern already established 2026-07-16 through
    07-24 -- NOT a real artifact change, NOT actionable, NOT a P1. NO NEW
    RELEASES for any of the 3 artifacts since 2026-07-25.
    RE-VERIFIED 2026-07-25 (this Scout run -- WebFetch of maven-metadata.xml
    for all 3 artifacts, verbatim-XML prompt used for client-l after an
    initial summarized fetch mis-stated release/latest and required a
    second, verbatim-quote fetch to correct -- see below):
    client-l: verbatim raw XML confirms <release>1.1.0</release>,
    <latest>1.1.X-SNAPSHOT</latest>, <lastUpdated>20260718072455</lastUpdated>
    (2026-07-18) -- IDENTICAL to the 2026-07-24 baseline, i.e. NO new
    tagged release since 07-18. (Note: a first-pass WebFetch summary of
    this same URL today incorrectly reported "release: 1.0.0" -- this was
    an AI-summarization error by the fetch tool, corrected by re-fetching
    with a prompt demanding a verbatim tag-by-tag quote of the raw XML;
    the corrected/verified value is release=1.1.0, matching every prior
    cycle. Flagging so a future cycle doesn't get misled by a similar
    summarization slip -- always ask for verbatim XML on this file.)
    client-m: release/latest STILL 1.2.2, lastUpdated STILL
    20260608030211 -- fully unchanged.
    cxr-service-bridge: release/latest STILL 1.0, versions still just
    [1.0-SNAPSHOT, 1.0], lastUpdated STILL 20260723084719 (2026-07-23) --
    UNCHANGED from the value already recorded in the 2026-07-24 baseline,
    i.e. no further bump since yesterday's check. Consistent with
    the already-established re-indexing/checksum-noise pattern for this
    artifact (see 2026-07-16/07-21 investigations) -- NOT independently
    re-verified via browse-listing byte/timestamp diff this run (time-
    boxed); treat the "no real artifact change" conclusion as carried
    forward from the prior confirmed investigation, not re-proven today.
    NO NEW RELEASES for any of the 3 artifacts since 2026-07-24.
    RE-VERIFIED 2026-07-24 (this Scout run -- WebFetch of maven-metadata.xml
    for all 3 artifacts; narrow-manifest exception per Scout policy):
    client-l: release/latest STILL 1.1.0. `lastUpdated` MOVED to
    20260718072455 (2026-07-18) from the previously-recorded
    20260702091606. The full <versions> list still tops out at 1.1.0,
    with a 1.1.X-SNAPSHOT entry also present -- most consistent with a
    SNAPSHOT rebuild on the 1.1.x line, not a new tagged release. This
    run did NOT independently verify (via .aar Last-Modified/size/hash
    diff) whether the 1.1.0 release artifact itself changed -- flagged
    as a WATCH item at the time.
    cxr-service-bridge: release/latest STILL 1.0. `lastUpdated` MOVED to
    20260723084719 (2026-07-23, the day before this check) from a
    previously-recorded 20260715121541. Same caveat as client-l: this
    run did not pull the .aar browse-listing itself -- flagged as a
    WATCH item at the time.
    BOTH WATCH ITEMS RESOLVED same-day by a sibling Scout run in this
    cycle (see Process note above re: 3 concurrent runs): a browse-listing
    scrape of https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/client-l/1.1.0/
    confirms client-l-1.1.0.aar is byte-identical to its previously-known
    state (Last Modified Thu Jul 02 10:18:18 Z 2026, 1,286,574 bytes) --
    only .sha1 (Jul 08) / .sha256 (Jul 11) checksum sidecars carry newer
    timestamps. Likewise for cxr-service-bridge-1.0.aar (Last Modified
    Thu Dec 25 12:54:20 Z 2025, 1,076,548 bytes, unchanged). Both
    `lastUpdated` bumps are confirmed repository re-indexing/checksum-
    regeneration noise, consistent with the same pattern already
    established for cxr-service-bridge on 2026-07-16/07-21 -- NOT real
    artifact changes, NOT actionable. No P1.
    client-m: release/latest STILL 1.2.2, lastUpdated STILL
    20260608030211 -- fully unchanged, no ambiguity.
    RE-VERIFIED 2026-07-21 (WebFetch of maven-metadata.xml for all 3
    artifacts): client-l release/latest still 1.1.0, lastUpdated
    20260702091606 -- unchanged, already fully documented via binary-diff
    (re-confirmed independently today via direct AAR/POM download+diff --
    see custom.rokid.com entry above). client-m release/latest still
    1.2.2, lastUpdated 20260608030211 -- unchanged. cxr-service-bridge
    release/latest still 1.0, lastUpdated 20260715121541 -- unchanged
    since 07-17/07-18, already ruled out as re-indexing noise.
    RE-VERIFIED 2026-07-18 (fresh Firecrawl scrape of all 3
    maven-metadata.xml files, no cache): client-l release/latest STILL
    1.1.0, lastUpdated STILL 20260702091606 -- unchanged. client-m
    release/latest STILL 1.2.2, lastUpdated STILL 20260608030211 --
    unchanged. cxr-service-bridge release/latest STILL 1.0, lastUpdated
    STILL 20260715121541 -- unchanged from yesterday (i.e. no further
    bump since the 07-17 re-indexing-noise finding; directly re-fetched
    and diffed today, not just carried forward). NO NEW RELEASES for any
    of the 3 artifacts since 2026-07-17.
    client-l: release/latest = 1.1.0, lastUpdated 20260702091606
      (2026-07-02) -- UNCHANGED, re-confirmed via fresh maven-metadata.xml
      fetch 2026-07-16. Still no official changelog published (developerdoc
      /sdk CXR-L tab confirmed still tops out at 1.0.4/2026-06-29). Already
      documented provisionally via binary-diff in cxr-l/release-notes.md
      and cxr-l/api-reference.md -- no further action.
    client-m: release/latest = 1.2.2, lastUpdated 20260608030211 -- UNCHANGED,
      re-verified 2026-07-16. Portal CXR-M tab confirmed still only
      documents through v1.1.0/2026-04-01 -- gap already noted in
      cxr-m/intro.md, no new action.
    cxr-service-bridge: release/latest = 1.0. maven-metadata.xml
      `lastUpdated` jumped to 20260715121541 (2026-07-15, YESTERDAY) from
      the previously-known 20260522063622 -- but the actual
      cxr-service-bridge-1.0.aar file itself is UNCHANGED: browse listing
      shows "Last Modified: Thu Dec 25 12:54:20 Z 2025", same 1,076,548-
      byte size as always known. Only checksum sidecar files
      (.aar.sha256, .module.md5, etc.) show recent modification
      timestamps (May 14, Jul 11, ...) -- this is repository re-indexing /
      checksum regeneration noise, NOT a real artifact change. VERIFIED,
      NOT ACTIONABLE (explicitly checked and ruled out this run, not just
      assumed).
  notes: |
    RE-VERIFIED 2026-07-29 (this Scout run): browse-listing path WAS
    re-checked this cycle (unlike 07-24/07-25) specifically to investigate
    the cxr-service-bridge lastUpdated bump -- see last_known_version above
    for the confirmed re-indexing-noise finding. Evidence:
    .firecrawl/maven-csb-browse-20260729.md.
    RE-VERIFIED 2026-07-25 (this Scout run): direct browse path not
    re-checked this cycle either (WebFetch of maven-metadata.xml used per
    the narrow-manifest exception, consistent with policy). See
    last_known_version above for the corrected client-l release=1.1.0
    finding (an initial WebFetch summarization error was caught and fixed
    by re-fetching with a verbatim-XML prompt).
    RE-VERIFIED 2026-07-24 (this Scout run): direct browse path
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    not re-checked this cycle (WebFetch of maven-metadata.xml was used
    instead, per the narrow-manifest exception). See last_known_version
    above for the two lastUpdated-timestamp WATCH items opened today
    (client-l 07-18, cxr-service-bridge 07-23) -- both since RESOLVED by a
    sibling Scout run's browse-listing check the same day: confirmed
    re-indexing/checksum noise, not real artifact changes.
    RE-VERIFIED 2026-07-16 (Firecrawl scrape of maven-metadata.xml for
    all three artifacts, plus a browse-listing scrape of
    cxr-service-bridge/1.0/ to investigate the lastUpdated jump). No new
    releases since 2026-07-14 for client-l/client-m. cxr-service-bridge
    lastUpdated bump investigated and ruled out as real content change
    (see last_known_version above).
    Direct browse path:
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable"; maven-metadata.xml
    under each artifact path is the canonical machine-readable source).
    RE-VERIFIED 2026-07-17: fresh scrape of client-l and client-m
    maven-metadata.xml. client-l release/latest still 1.1.0,
    lastUpdated 20260702091606 -- unchanged. client-m release/latest
    still 1.2.2, lastUpdated 20260608030211 -- unchanged. No new
    releases. (cxr-service-bridge not re-checked today; no change
    expected, low priority to re-verify given 07-16 findings already
    ruled out the lastUpdated jump as re-indexing noise.)

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-21
  notes: |
    RE-VERIFIED 2026-08-21 (this run -- fresh live map, limit 50, no cache):
    still exactly 2 public repos, UXR-docs and glass2-docs, both out of
    scope. No new repos, no change.
    RE-VERIFIED 2026-08-03 (this Scout run -- fresh live map, limit 50, no
    cache, 22 links): still exactly 2 public repos, UXR-docs and
    glass2-docs, both out of scope. No new repos, no change. Evidence:
    .firecrawl/gh-rokidglass-20260803.json.
    RE-VERIFIED 2026-07-29 (this Scout run -- fresh live map, limit 50, no
    cache, 25 links): still exactly 2 public repos, UXR-docs and
    glass2-docs, both out of scope. No new repos, no change. Evidence:
    .firecrawl/gh-rokidglass-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run -- fresh live map, limit 50, no
    cache, 25 links): still exactly 2 public repos, UXR-docs and
    glass2-docs, both out of scope (spatial-computing SDK / older Glass 2
    hardware+Issues respectively). No new repos, no change. Evidence:
    .firecrawl/gh-rokidglass-20260725.json.
    RE-VERIFIED 2026-07-24 (this Scout run -- fresh live map, limit 50, no
    cache): still exactly 2 public repos, UXR-docs (out-of-scope spatial-
    computing SDK) and glass2-docs (out-of-scope Glass 2/older hardware,
    many old GitHub Issues threads e.g. #177, #53, #80, #201, #168,
    #213, #84, #44, #224, #162, #98, #204, #176, #63, #83, #209, #66,
    #174, #88, #57, #35). No in-scope Sprite/AR Glasses/CXR content. No
    new repos, no change.
    RE-VERIFIED 2026-07-21 (fresh live map): still exactly 2 public repos,
    UXR-docs and glass2-docs, both out of scope. No change.
    RE-VERIFIED 2026-07-18 (fresh live map, limit 50, no cache): still
    exactly 2 public repos, UXR-docs (out-of-scope spatial-computing SDK)
    and glass2-docs (out-of-scope Glass 2 / older hardware, GitHub Issues
    threads e.g. #177, #53, #80, #201...). No in-scope Sprite/AR Glasses/
    CXR content. No new repos, no change since 2026-07-17.
    RE-VERIFIED 2026-07-16 (live map, 27 links / same 2 repos). Still only
    2 public repos: UXR-docs (out-of-scope spatial-computing SDK) and
    glass2-docs (out-of-scope Glass 2 / older hardware, mostly old GitHub
    Issues threads). No in-scope Sprite/AR Glasses content. No change
    since 2026-07-14. No action items.
    RE-VERIFIED 2026-07-17: fresh map, still exactly UXR-docs and
    glass2-docs, both out of scope. No change.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-29
  notes: |
    RE-VERIFIED 2026-07-29 (this Scout run -- fresh live map, limit 50, no
    cache): 41 links returned. Same long-standing legacy voice/speech/
    skill-platform repo set as prior cycles, PLUS several repo names not
    explicitly enumerated in the 07-24/07-25 baseline text --
    RokidVoiceAISDK, egg-raven, cloudapp-engine, Cocoapods-Specs,
    rokid-openvoice-websocket. All are legacy voice/speech/CloudApp-protocol
    tooling by name (consistent with the rest of this org's out-of-scope
    content) and their appearance/disappearance across cycles matches this
    source's already-documented crawl-coverage variance pattern (28/40/41/
    32/etc. links across different cycles) -- not independently confirmed as
    newly-created repos (no per-repo creation-date check performed this
    run), but naming and context make new-and-in-scope highly unlikely. No
    in-scope Sprite/CXR content found. No action. Evidence:
    .firecrawl/gh-rokid-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run -- fresh live map, limit 50, no
    cache): 40 links returned; same long-standing legacy repo set as
    prior cycles PLUS one repo not explicitly named in any prior cycle's
    enumeration: `RokidPhone` (surfaced via a /pulls sub-URL in the map).
    Independently scraped https://github.com/rokid/RokidPhone directly to
    check if this is a genuinely new repo: it is NOT new -- Public repo, 1
    branch (master), 0 tags, single commit "初始化 智能语音识别SDK的Demo"
    ("init smart speech-recognition SDK demo") dated Nov 29, 2017 (9 years
    old). This is 9-year-old legacy speech-recognition demo code, pre-
    dating YodaOS-Sprite entirely -- OUT OF SCOPE, and its appearance here
    (vs. absence from 07-24's enumeration) is map crawl-coverage variance
    (already a documented pattern for this source, e.g. 28/40/41/32 links
    across different cycles), NOT a newly-created repo. No action. All
    other repos in today's map match the established out-of-scope legacy
    voice/speech/skill-platform / pre-Sprite Glass set. No in-scope
    content found. Evidence: .firecrawl/gh-rokid-20260725.json,
    .firecrawl/gh-rokidphone-20260725.json.
    RE-VERIFIED 2026-07-24 (this Scout run -- fresh live map, limit 50, no
    cache): 32 links returned; same long-standing repo set (NextForum,
    mingutils, RokidMobileSDKiOS/AndroidDemo, CloudAppClient,
    blacksiren, RokidVoiceAIDemo, better_jieba, community, NewsDemo,
    docs, skill-java, UXR-docs [out of scope], rokidos-cli,
    node-webworker, RokidSDK-Swift, rokidos-www, ELMo-chinese,
    glass-docs [out of scope, legacy 2020-era Glass project], tts-demo,
    mapi-demo-outer, native-system-docs, speech-python-demo,
    node-http-bypass). No NEW repositories. All content remains legacy
    voice/speech/skill-platform or pre-Sprite Glass, out of scope. No
    in-scope Sprite/CXR content found. No action.
    RE-VERIFIED 2026-07-21 (fresh live map): same legacy Skill/Voice/Glass-1-2
    era repo set as prior cycles, all out of scope. No in-scope content, no
    change.
    RE-VERIFIED 2026-07-18 (fresh live map, limit 50, no cache): 41 links
    returned, same repo set as 07-17/07-16 (NextForum, mingutils,
    RokidMobileSDKiOS/AndroidDemo, CloudAppClient, blacksiren,
    RokidVoiceAIDemo, better_jieba, community, NewsDemo, docs, skill-java,
    UXR-docs [out of scope], rokidos-cli, node-webworker, RokidSDK-Swift,
    rokidos-www, ELMo-chinese, glass-docs [out of scope, legacy], tts-demo,
    mapi-demo-outer, native-system-docs, speech-python-demo, node-http-
    bypass, plus issue/PR sub-URLs) -- no NEW repositories, same deeper
    crawl-coverage pattern as 07-17. All content remains legacy voice/
    speech/skill-platform, out of scope. No action.
    RE-VERIFIED 2026-07-16 (live map, 28 links / distinct repos matches
    the 2026-07-14 enumeration: NextForum, mingutils, RokidMobileSDKiOS/
    AndroidDemo, CloudAppClient, blacksiren, RokidVoiceAIDemo,
    better_jieba, community, NewsDemo, docs, skill-java, UXR-docs (out of
    scope), rokidos-cli, node-webworker, RokidSDK-Swift, rokidos-www,
    ELMo-chinese, glass-docs (legacy pre-Sprite 2020-era Glass project,
    out of scope), etc.). No in-scope Sprite/AR Glasses/CXR content found.
    No action items. No change since 2026-07-14.
    RE-VERIFIED 2026-07-17: fresh map returned 40 links (vs 28 on
    07-16) -- same repo set plus additional issue/PR/release sub-URLs
    (e.g. rokid/docs/issues, rokid/tts-demo, rokid/mapi-demo-outer,
    rokid/native-system-docs) surfaced this pass; no NEW repositories,
    just deeper crawl coverage of existing ones. All content remains
    legacy voice/speech/skill-platform, out of scope. No action.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-21
  notes: |
    RE-VERIFIED 2026-08-21 (this run -- fresh live map, limit 50, no cache):
    still an empty links array (0 URLs), consistent with every check since
    2026-06-30. No actionable content.
    RE-VERIFIED 2026-08-03 (this Scout run -- fresh live map, limit 50, no
    cache): still an empty links array (0 URLs), consistent with every
    check since 2026-06-30. No actionable content. Evidence:
    .firecrawl/gh-rokidar-20260803.json.
    RE-VERIFIED 2026-07-29 (this Scout run -- fresh live map, limit 50, no
    cache): still an empty links array (0 URLs), consistent with every
    check since 2026-06-30. No actionable content. Evidence:
    .firecrawl/gh-rokidar-20260729.json.
    RE-VERIFIED 2026-07-25 (this Scout run -- fresh live map, limit 50, no
    cache): still an empty links array (0 URLs), consistent with every
    check since 2026-06-30. No actionable content. May have private repos
    not visible to this API key. Evidence: .firecrawl/gh-rokidar-20260725.json.
    RE-VERIFIED 2026-07-24 (this Scout run -- fresh live map, limit 50, no
    cache): still an empty links array (0 URLs), consistent with every
    check since 2026-06-30. No actionable content. May have private
    repos not visible to this API key.
    RE-VERIFIED 2026-07-21 (fresh live map): still an empty links array (0
    URLs), consistent with every check since 2026-06-30. No action.
    RE-VERIFIED 2026-07-18 (fresh live map, limit 50, no cache): still an
    empty links array (0 URLs), consistent with every check since
    2026-06-30. No actionable content. May have private repos not visible
    to this API key.
    RE-VERIFIED 2026-07-16 (live map): map again returned an empty links
    array (0 URLs), consistent with every check since 2026-06-30. No
    actionable content. May have private repos not visible to this API
    key.
    RE-VERIFIED 2026-07-17: fresh map, still 0 URLs. No change.

## New sources discovered (pending user approval to add to registry)

- url: https://x-docs.rokid.com/docs/
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l (unverified -- see notes)
  status: UNREGISTERED -- requires user approval. This cycle (2026-07-25)
    a single light spot-check scrape of the root /docs/ landing page was
    performed (consistent with the "just note if status/content changed"
    instruction for this cycle, not a registration or a deep crawl) --
    see notes below for a SIGNIFICANT status change. Do not crawl deeper
    or register until a human explicitly approves via AskUserQuestion.
  notes: |
    RE-VERIFIED 2026-08-03 (this Scout run -- single cache-bypassed
    --max-age 0 scrape of https://x-docs.rokid.com/docs/, per the
    established "just note if status/content changed" pattern -- NOT
    crawled deeper, NOT registered, NOT translated): content UNCHANGED from
    the 07-25/07-29 findings. Still renders the "Rokid Sprite Enterprise
    SDK" landing page for "Rokid Glass3 企业版" with 快速开始/API 参考/代码
    示例/FAQ nav. Scope remains undetermined by this unattended run (no
    AskUserQuestion available) -- "Rokid Glass3" is not one of the
    CLAUDE.md-listed in-scope hardware models (RV101/RV102/RV203/OEM
    variants/Rokid AI Glasses) by name, but could plausibly be a newer
    Sprite-line model or a distinct enterprise product; this ambiguity is
    surfaced to the Leader/human for triage, consistent with the "wholly
    new top-level section needs sign-off" rule -- not resolved by Scout.
    Evidence: .firecrawl/x-docs-spot-20260803.md.
    RE-VERIFIED 2026-07-29 (this Scout run -- single cache-bypassed
    --max-age 0 scrape of https://x-docs.rokid.com/docs/, per the "just note
    if status/content changed" instruction for this cycle -- NOT crawled
    deeper, NOT registered): content UNCHANGED from the 07-25 status-change
    finding. Still renders the same substantial "Rokid Sprite Enterprise
    SDK" landing page: title, tagline referencing "Rokid Glass3 enterprise
    edition", and the same 开始接入/API 参考/代码示例/下载 AI Skill nav plus
    3-step flow (跑通 Demo / 接入 SDK / 按能力开发). Re-confirmed generator
    metadata: VitePress v2.0.0-alpha.12. Scope determination is STILL NOT
    made by Scout (no AskUserQuestion available in this unattended cycle,
    consistent with the no-silent-extension rule) -- this remains a
    standing item for human triage, not a new discovery this cycle, just a
    reconfirmation that the rich content persists (i.e. it was not a
    one-off render). Evidence: .firecrawl/x-docs-spot-20260729.json.
    STATUS CHANGE FOUND 2026-07-25 (this Scout run -- single cache-bypassed
    scrape of https://x-docs.rokid.com/docs/, --max-age 0): this page now
    renders REAL, substantial content, not just an undetermined landing
    page as in prior cycles. Title: "Rokid Sprite Enterprise SDK". Tagline
    (translated): "Terminal SDK documentation for the Rokid Glass3
    enterprise edition. Run the demo first, then look up APIs per device
    side, then complete business integration by capability." Visible nav/
    sections: 开始接入 (Get Started), API 参考 (API Reference), 代码示例
    (Code Samples), 下载 AI Skill (Download AI Skill), plus a 3-step flow
    (跑通 Demo / 接入 SDK / 按能力开发 covering device connection, messaging,
    media, voice AI, vision recognition). This is a MUCH richer, live SDK
    documentation site than what was visible in prior cycles (which only
    found an ambiguous hardware-card link, content undetermined). Re-
    confirmed via a rawHtml check of developerdoc.rokid.com/sdk today that
    the "Rokid Glasses3" hardware card (linking here) is STILL present
    there, alongside YodaOS-Master, in the same guide-menu-dropdown.
    SCOPE STILL UNDETERMINED BY SCOUT (correctly, per the no-silent-
    extension rule): "Rokid Glass3" / "Sprite Enterprise" could plausibly
    be an enterprise/OEM variant of the in-scope YodaOS-Sprite line (would
    make this in-scope per CLAUDE.md's "OEM variants" language), OR a
    distinct enterprise-tier product family adjacent to Master/ER (would
    stay out of scope). The name "Sprite Enterprise" leans toward in-scope
    (shares the "Sprite" branding, not "Master"), but this is Scout's
    inference, not a verified determination -- do NOT translate or commit
    any of this content until a human explicitly triages and approves.
    This finding is materially different from every prior cycle's
    "STILL PENDING, not scraped" note and should be prioritized for human
    review THIS cycle. Evidence: .firecrawl/x-docs-spot-20260725.json.
    Discovered 2026-07-18: a 4th hardware card, "Rokid Glasses3", appeared
    on developerdoc.rokid.com/sdk (not present in 07-16/07-17 rawHtml
    checks of the same page) linking to "YodaOS-Sprite Enterprise" at
    https://x-docs.rokid.com/docs/. Unclear whether "YodaOS-Sprite
    Enterprise" is in-scope (an enterprise/OEM variant of the in-scope
    YodaOS-Sprite covered by this repo) or a distinct out-of-scope product
    line -- NOT determined this run, no content fetched. Flag for human
    triage: if in-scope, this could be a new registry source; if it turns
    out to be an enterprise-tier Master/ER-adjacent product, it stays out
    of scope per CLAUDE.md. Needs explicit AskUserQuestion approval before
    any future Scout cycle scrapes it.

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED -- requires user approval. NOT independently
    scraped this run (out of the 8 approved registry sources); its
    existence and role as the new canonical portal is corroborated
    indirectly via live scrapes of developer.rokid.com and ar.rokid.com,
    both of which link Rokid Glasses docs to open.rokid.com/sdk?lang=en.
  notes: |
    RE-VERIFIED 2026-08-21 -- STILL NOT independently registered or crawled
    as a registry source this run, per the no-silent-extension rule (no
    AskUserQuestion available in this unattended cycle). One light spot-check
    scrape of https://open.rokid.com/sprite?lang=zh (no --wait-for, --max-age
    default) rendered the FULL hydrated page this time -- content still
    word-for-word matches developerdoc.rokid.com/sprite and
    yodaos/docs/sprite-overview.md (same 4-item CXR-M/CXR-S FAQ, same
    hardware spec table, same CXR-M business-cooperation gating notice). A
    companion spot-check of https://open.rokid.com/sdk?lang=zh without
    --wait-for returned only the bare SPA shell (consistent with the
    equivalent developerdoc.rokid.com/sdk behavior documented above -- this
    route needs `--wait-for 5000`+ to hydrate; not retried with that flag
    this cycle since /sdk is outside the 8 approved sources). Recommend the
    user decide on registering open.rokid.com -- it has now mirrored
    developerdoc.rokid.com/sprite exactly across every check since
    2026-06-28 (nearly 2 months) with zero independent drift.
    RE-VERIFIED 2026-07-29 -- NOT independently registered or crawled as a
    registry source this run, per the no-silent-extension rule (no
    AskUserQuestion available in this unattended cycle). One light spot-
    check scrape of https://open.rokid.com/sprite?lang=zh was performed
    (--max-age 0, cache-bypassed): this attempt rendered only the bare SPA
    shell ("Rokid AR Platform", no body content) rather than the full
    hydrated page the 07-25 check captured -- a rendering-timing miss for
    this specific attempt, NOT evidence of drift (07-25's word-for-word
    diff against developerdoc.rokid.com/sprite already established the
    mirror relationship; this run could not re-confirm or contradict it).
    Both ar.rokid.com and developer.rokid.com's root redirects were
    re-verified today as still landing on open.rokid.com (see their
    entries above). Recommend the user decide on registering open.rokid.com
    given it has now been open for many cycles. Evidence:
    .firecrawl/open-rokid-sprite-spot-20260729.json.
    STILL PENDING 2026-07-25 -- NOT independently registered or crawled as
    a registry source this run, per the no-silent-extension rule (no
    AskUserQuestion available in this unattended cycle). One light spot-
    check scrape of https://open.rokid.com/sprite?lang=zh was performed
    (--max-age 0, cache-bypassed) and its content re-diffed against
    developerdoc.rokid.com/sprite's fresh scrape from today: WORD-FOR-WORD
    IDENTICAL (same hardware spec table, same CXR-L/CXR-M/CXR-S cards,
    same 4-item CXR-M/CXR-S FAQ) -- confirms open.rokid.com continues to
    mirror developerdoc.rokid.com exactly, no independent drift. Both
    ar.rokid.com and developer.rokid.com's root redirects were re-verified
    today as still landing on open.rokid.com. Recommend the user decide on
    registering open.rokid.com given it has now been open for many cycles.
    Evidence: .firecrawl/open-rokid-sprite-spot-20260725.json.
    STILL PENDING 2026-07-24 -- NOT independently registered or crawled
    as a source this run, per the no-silent-extension rule (no
    AskUserQuestion available in this unattended cycle). Its role as the
    landing target for both ar.rokid.com and developer.rokid.com was
    RE-CORROBORATED today via this run's own direct scrapes (both
    redirect to https://open.rokid.com/, confirmed via sourceURL/finalURL
    metadata). As a one-off spot check (not a registered-source crawl),
    this run also scraped https://open.rokid.com/sprite?lang=zh directly
    and confirmed its content matches developerdoc.rokid.com/sprite and
    yodaos/docs/sprite-overview.md exactly (same 4 FAQ Q&As, same
    hardware spec table) -- consistent with the 2026-06-28 finding.
    Recommend the user decide on registering open.rokid.com given it has
    now been open for many cycles and both redirects/mirrors continue to
    point to it.
    STILL PENDING 2026-07-21 -- not scraped, per the no-silent-extension
    rule; unchanged since 07-18. Recommend the user decide on registering
    it, and on x-docs.rokid.com/docs/ below, given both have now been open
    for multiple cycles.
    RE-CORROBORATED 2026-07-18 via fresh live scrapes of developer.rokid.com
    and ar.rokid.com's /sdk hardware cards, and via developerdoc.rokid.com's
    header nav (both "SDK" and the YodaOS-Sprite/YodaOS-Master product
    links now point to open.rokid.com/sdk?lang=zh, open.rokid.com/sprite,
    open.rokid.com/master, open.rokid.com/academy). Still NOT independently
    scraped this run -- remains outside the 8 approved registry sources;
    this is an unattended scheduled run, no AskUserQuestion tool used.
    Discovered 2026-06-28, corroborated again 2026-07-10, 07-12, 07-14,
    and 2026-07-16 (this run) via developer.rokid.com and ar.rokid.com,
    both of which still serve the "AIUI: The Next Frontier" homepage and
    link Docs to open.rokid.com/sdk / /sprite as of today. Still NOT
    independently scraped this run -- remains outside the 8 approved
    registry sources. This is an unattended scheduled run: no
    AskUserQuestion tool was available/used, so no approval was sought or
    granted. Continues to be flagged for the next attended cycle so a
    human can explicitly approve or decline adding it.
    Last direct check (2026-06-28) found open.rokid.com/sprite?lang=zh
    content matched developerdoc.rokid.com/sprite exactly; open.rokid.com
    /sdk?lang=zh returned only an SPA shell. /academy hosts "乐奇学院"
    (Rokid Academy) with CXR-L / Glasses courses (in scope) alongside
    UXR 3.0 and AIUI courses (out of scope). YodaOS-Master content linked
    from open.rokid.com/master -- would be skipped if this source is ever
    approved. Suggest registering open.rokid.com as the canonical
    replacement for ar.rokid.com; requires explicit user sign-off before
    Scout treats it as a registry source.
