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
  last_checked: 2026-08-30
  last_known_version: CXR-L 1.1.2 on Maven / 1.0.4 on official docs (see custom.rokid.com and maven entries)
  notes: |
    React SPA, re-verified 2026-08-30 (scrape succeeded, real content retrieved). Root page is
    now an AIUI-first homepage. SDK grid links out to: CXR-L -> https://t.rokid.com/uwxdzi51
    (short link, resolves to custom.rokid.com workspace 84feb39f8ef141b0ad0326f902ab881f);
    CXR-S -> custom.rokid.com workspace 57e35cd3ae294d16b1b8fc8dcbb1b7c7; bare-metal (BMP) ->
    custom.rokid.com workspace ff28c865a9634876be98cbc293588460 (NEW, previously BMP shared
    the CXR-S/CXR-M hash — now split into its own workspace). NOTABLE: the SDK grid on this
    page no longer has a CXR-M card at all (see open.rokid.com/sprite note below — CXR-M is
    now business-cooperation-only, not publicly documented). ar.rokid.com/sprite ->
    open.rokid.com/sprite is confirmed as the canonical Sprite content (identical to
    developerdoc.rokid.com/sprite). YodaOS-Master tab remains; skipped (out of scope). Also
    links out to UXR3.0/MRTK3/XRI/RNO/RUO/JSAR (all out of scope, all under shared workspace
    c88be4bcde4c42c0b8b53409e1fa1701) — noted, not scraped further.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-30
  last_known_version: same SPA shell as ar.rokid.com / open.rokid.com — see those entries
  notes: |
    Re-verified 2026-08-30. Map returns 3 URLs (/sprite?lang=zh, /sprite, /sdk). Scraping
    /sdk returns only the bilingual-glossary SPA shell (same as open.rokid.com/sdk) —
    confirms developerdoc.rokid.com and open.rokid.com serve the identical React app/build.
    No independent content; treat as a mirror of open.rokid.com going forward.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-30
  last_known_version: CXR-S "Brief" doc content unchanged vs local cxr-s/brief.md (verbatim match). CXR-L doc portal shows "Version 1.0.4" (Maven is ahead at 1.1.2 — see maven entry). Bare-metal (BMP) doc substantially rewritten, now in English, workspace ff28c865a9634876be98cbc293588460.
  notes: |
    CORRECTION to 2026-06-30 "BROKEN" verdict: re-tested 2026-08-30 with --wait-for 10000
    and the SAME two workspace hashes previously reported as NoSuchKey now return full
    content successfully:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/us/3fe1c87b945245bf8b6c50393f4da7b6.html (CXR-S
      Brief) — content verified verbatim-identical to local cxr-s/brief.md. No drift.
    - 84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html (CXR-L
      Introduction, reached via https://t.rokid.com/uwxdzi51) — NOTE this is a *different*
      docId than the one cited in local cxr-l/intro.md (9adcfb07939846e5945e79dfbd923f63),
      i.e. the doc was reorganized/replaced, not just edited in place. New content adds a
      "Device control (brightness/volume)" capability with documented range 0-15 for both —
      this officially confirms values that cxr-l/api-reference.md currently lists as
      "undocumented (inferred from binary diff)". Sample project renamed
      CXRLSample -> RenewCXRLSample (com.rokid.renewcxrlsample), requires Rokid AI App >= 1.9.0.
    - ff28c865a9634876be98cbc293588460/pc/us/index.html (bare-metal dev, NEW/separate
      workspace) — full rewrite in English vs local's Chinese v0.0.1 (2026-03-01) docs.
      New structure: Introduction, Quick Start, Glasses UI Design Guidelines (Bare Metal),
      GlassesBareDevSample Project and Pages, Keys/Wear/Fold Events, Raw Audio, Photo
      Capture, Video Recording, IMU and Sensors. Sample app now GlassesBareDevSample
      (com.rokid.glassesbaredevsample), minSdk 31, targetSdk 36. Confirmed via actions-click
      that "Quick Start" subpage also renders real content (not just the landing page).
    ROOT CAUSE of prior false "BROKEN": the SPA shows a loading spinner for several seconds
    before content resolves; a short/no --wait-for scrape captures only the spinner shell
    (looks like empty/broken content, NOT an actual NoSuchKey error this cycle). Going
    forward, always scrape custom.rokid.com pages with --wait-for >= 8000-10000ms before
    concluding content is missing or broken.
    Root path (https://custom.rokid.com/prod/rokid_web/ with no workspace/docId) still fails
    outright (all Firecrawl engines failed) — that part of the prior BROKEN note stands.
    Individual tree subsections (Development Environment, SDK Import, Function Development
    for CXR-S; Version History for CXR-L) were not individually resolved this cycle — the
    SPA uses JS-only tree navigation with no crawlable hrefs, so each subpage needs a
    dedicated actions-click scrape (as done for BMP Quick Start) rather than map/crawl.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-30
  notes: Re-verified 2026-08-30 — confirmed redirects to the same AIUI-focused ar.rokid.com/open.rokid.com homepage (identical footer links: SDK -> developerdoc.rokid.com/sdk, YodaOS-Master-Docs -> open.rokid.com/master, YodaOS-Sprite-Docs -> open.rokid.com/sprite). No independent or in-scope content. Low priority, unchanged from prior cycle.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-30
  last_known_version: |
    client-l: release 1.1.2 (versions list also shows 1.1.0, 1.1.1 as intermediate releases
      beyond the previously-tracked 1.0.4; metadata lastUpdated 20260828083628, i.e. 2 days
      before this check). THREE new releases since last cycle's known-latest of 1.0.4.
    client-m: release 1.2.2 (unchanged version number; metadata lastUpdated timestamp moved
      to 20260826091529 from 20260608030211 — likely repo reindex, not a new build; verify
      AAR hash if this needs to be ruled out definitively)
    cxr-service-bridge: release 1.0 (unchanged; lastUpdated 20260728074326, likely reindex)
  notes: |
    maven-metadata.xml fetched directly (rawHtml format) and parsed 2026-08-30 for all three
    artifacts at https://maven.rokid.com/repository/maven-public/com/rokid/cxr/<artifact>/maven-metadata.xml.
    client-l is now 3 releases ahead of what local cxr-l/release-notes.md documents (only
    goes up to v1.0.4, itself only provisionally reconstructed from a binary diff since no
    official changelog existed at the time). The official custom.rokid.com doc portal is
    still showing "Version 1.0.4" as of this check (see custom.rokid.com entry) — so Maven
    leads the official docs by 3 releases (1.1.0, 1.1.1, 1.1.2), same lag pattern as before.
    Use Maven as the canonical "what's actually shipped" source; official docs remain the
    authoritative source for behavior/API description once published.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-30
  notes: Re-verified 2026-08-30 — unchanged. Still exactly 2 public repos: glass2-docs (out-of-scope Glass 2) and UXR-docs (out-of-scope, public archive). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-30
  notes: Re-verified 2026-08-30. Pinned/listed repos include community, docs, glass-docs (73 total repos in org). Checked github.com/rokid/docs directly: latest commit is from 2022-09-15 ("更新tts http接口错误"), pure Speech/TTS HTTP API content, not Sprite/Glasses. glass-docs still old Glass 1/2 era. No in-scope content found. Low priority, unchanged verdict from prior cycle.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-30
  notes: Re-verified 2026-08-30 via direct scrape (map previously returned 0 links; scrape now shows content directly). 2 public repos: openCV3_demo and BroadcastServiceDemo (both old/inactive, BroadcastServiceDemo dates to 2017). No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval. Re-checked 2026-08-30, current state reported below; NOT promoted without explicit approval.
  notes: |
    Re-verified 2026-08-30. Map still returns the same small SPA-route set (root, /sitemap.xml,
    /sprite?lang=en, /sprite?lang=zh, /sdk?lang=zh, /sdk?lang=en, /academy, root AIUI page,
    /master, and 3 /detail?appId=... Rokid Store pages). open.rokid.com/sprite?lang=zh content
    reconfirmed byte-for-byte consistent with what ar.rokid.com and developerdoc.rokid.com
    link to — this remains the de facto canonical Sprite/CXR landing page, still unregistered.
    NEW THIS CYCLE: /sprite now explicitly documents that CXR-M SDK is no longer publicly
    available — "如需获取 CXR-M SDK、文档与技术支持，请联系商务合作：Glasses.BD@rokid.com...
    该 SDK 不在开发者站点公开提供" (CXR-M SDK access/docs/support now require contacting
    business development directly; the SDK is explicitly stated to not be publicly available
    on the developer site). This is a policy/access change, not merely a missing page —
    flagged as an action item in this cycle's report regardless of open.rokid.com's
    unregistered status, since the same statement is also visible on ar.rokid.com (a
    registered source).
    /academy ("乐奇学院") re-checked — leans heavily spatial-computing/AIUI branded ("空间计算
    时代与开发者共同生长"); in-scope CXR-L/Glasses course content is still mixed in with
    out-of-scope UXR3.0/AIUI course content per prior note. Skipped per out-of-scope policy;
    no full course catalog was scraped this cycle (would need JS interaction to enumerate).
    YodaOS-Master content at /master — skipped (out of scope).
    Recommendation unchanged: promote open.rokid.com to the registry as the canonical
    replacement/mirror-source for ar.rokid.com and developerdoc.rokid.com — pending user
    approval.
