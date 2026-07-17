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
  last_checked: 2026-07-17
  last_known_version: (no CXR doc content hosted here anymore; see notes)
  notes: |
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
  last_checked: 2026-07-17
  last_known_version: |
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
  last_checked: 2026-07-17
  notes: |
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
  last_checked: 2026-07-17
  notes: |
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
  last_checked: 2026-07-17
  last_known_version: |
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
  last_checked: 2026-07-17
  notes: |
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
  last_checked: 2026-07-17
  notes: |
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
  last_checked: 2026-07-17
  notes: |
    RE-VERIFIED 2026-07-16 (live map): map again returned an empty links
    array (0 URLs), consistent with every check since 2026-06-30. No
    actionable content. May have private repos not visible to this API
    key.
    RE-VERIFIED 2026-07-17: fresh map, still 0 URLs. No change.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED -- requires user approval. NOT independently
    scraped this run (out of the 8 approved registry sources); its
    existence and role as the new canonical portal is corroborated
    indirectly via live scrapes of developer.rokid.com and ar.rokid.com,
    both of which link Rokid Glasses docs to open.rokid.com/sdk?lang=en.
  notes: |
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
