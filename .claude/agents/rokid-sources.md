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
  last_checked: 2026-07-05
  last_known_version: CXR-L 1.0.4 shown on portal (developerdoc.rokid.com); Maven still has client-l 1.1.0 as of 2026-07-02, unchanged since last cycle (see maven.rokid.com entry)
  notes: |
    Reconfirmed 2026-07-05. firecrawl map returned 36 URLs (39 on 2026-07-04 — normal app-catalog
    churn, not a structural change), all Rokid Store app-catalog detail pages (/detail?appId=...)
    for Rokid Station/Air apps — out-of-scope hardware. New URL `/doc` surfaced this run (not seen
    2026-07-03/07-04); scraped it directly — it is the same English "AIUI: The Next Frontier" /
    hardware-card homepage already documented under developer.rokid.com (Rokid Glasses card's
    "Docs" CTA points to open.rokid.com/sdk?lang=en). Not a new in-scope surface, just a previously
    unindexed route to already-known content. Real Sprite/CXR doc surfaces remain at
    developerdoc.rokid.com (registered) and open.rokid.com (unregistered, see bottom section).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-05
  last_known_version: CXR-L 1.0.4 (portal changelog, dated 2026-06-29 — UNCHANGED this run, portal has still NOT published anything for Maven's client-l 1.1.0, now 3 days stale relative to the 2026-07-02 Maven upload). CXR-M tab and 眼镜端裸机开发 tab content still not independently reverified this run (JS-tab click continues to fail via Firecrawl actions; known SPA limitation, unchanged from prior runs).
  notes: |
    CONFIRMED 2026-07-05 with --max-age 0 --wait-for 3000 (a plain scrape without --wait-for
    returned only unrendered nav chrome this run — see caveat below): /sdk default (CXR-L) tab
    content is IDENTICAL to the 2026-07-04 capture — still only the v1.0.4 changelog (dated
    2026.06.29), same 7-item V1.0.4 changelog text (setGlassBrightness/setGlassVolume, "设备控制"
    chapter, iOS parity APIs) plus the V1.0.1 initial-release entry. No portal mention of client-l
    1.1.0 (Maven-side, unchanged — see maven.rokid.com entry). Genuine version-lag gap persists,
    not yet a portal content problem since portal simply hasn't published a changelog.
    CAVEAT (tooling note, not a content finding): a first-pass scrape of /sdk with --max-age 0 but
    no --wait-for returned only the SPA's unrendered header/footer chrome (nav links to
    open.rokid.com, no CXR-L body text) — this looks like a client-render timing issue, not a
    content change or redirect (metadata confirmed sourceURL=url=https://developerdoc.rokid.com/sdk,
    statusCode 200, no redirect). Re-running with --wait-for 3000 recovered the full expected
    content byte-for-byte matching prior runs. RECOMMENDATION: always pair --max-age 0 with
    --wait-for >=2000 on this SPA host going forward, or a future cycle could misreport this as
    "content vanished" when it was actually just an unrendered capture.
    RECONFIRMED content drift on /sprite FAQ (still present 2026-07-05, unchanged from 2026-07-03
    and 2026-07-04 findings): fresh scrape (--max-age 0 --wait-for 3000) shows the SAME 4 short
    Q&As, all CXR-M/CXR-S focused ("CXR-M SDK 的主要功能包含哪些？", "CXR-M SDK 目前支持哪些设备使用？",
    "CXR-S SDK 的主要功能包含哪些？", "仅使用 CXR-M SDK 开发移动端应用时，需要打开 Rokid Glasses 的开发者权限吗？"),
    still matching the OLDER FAQ content that local yodaos/docs/sprite-overview.md's own changelog
    note claims was "replaced" as of 2026-06-08 with an 8-Q&A CXR-L/bare-metal-focused set. Device
    spec table (Rokid Glasses: 143x44x160.5mm, Qualcomm AR1, 2GB/32GB, SONY IMX681 etc.) and
    Developer Toolkit section on /sprite still match local content exactly (no drift there).
    UNRESOLVED for the THIRD consecutive run (2026-07-03, 07-04, 07-05) — this is now a
    persistent discrepancy between what local docs claim was captured live on 2026-06-08 and
    what is actually live today, not a transient blip. Needs human judgment: either (a) the
    2026-06-08 capture that produced the local 8-Q&A text was wrong/from a different source, or
    (b) upstream genuinely rolled back the FAQ and never restored the CXR-L/bare-metal set.
    Map still returns only /sdk, /sprite, root (3-4 URLs). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-05
  notes: |
    STILL BROKEN as of 2026-07-05 (re-verified with cache bypass --max-age 0). Both previously
    valid workspace hashes still return OSS NoSuchKey with fresh RequestIds this run (confirming a
    live re-check, not a cached replay):
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey (RequestId
      6A49CA44565BBE3533505011 this run)
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey (RequestId 6A49CA4614E7143631853FFD
      this run)
    Source remains unreachable; new workspace hashes still unknown. Not usable as a monitor
    candidate. Third consecutive run confirming this (2026-07-03, 07-04, 07-05).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-05
  notes: |
    Reconfirmed 2026-07-05 via direct scrape of root (--max-age 0 --wait-for 2000) AND, new this
    run, a full firecrawl map (--limit 100): map returned 19 URLs, all under /docs or /skill —
    rokidos-linux-docs, rokid-homebase-docs, Skills Kit (2-RokidDocument/3-ApiReference/4-Terms/
    5-enableVoice/8-app), contact-us — a legacy GitBook doc tree for RokidOS-Linux, HomeBase, and
    the Speech Skills platform. None of these 19 URLs relate to YodaOS-Sprite, CXR-M/S/L, or Rokid
    Glasses hardware; confirms the 2026-07-04 note's assessment that this content is out of scope.
    Root page content unchanged: same AIUI/AI Agent homepage as ar.rokid.com, "Docs" CTA for Rokid
    Glasses still points to open.rokid.com/sdk?lang=en (unregistered source). Root also mentions
    "YodaOS-Sprite runs on Rokid Glasses and Bolon AI Glasses" — Bolon AI Glasses is already
    documented locally (yodaos/docs/hardware/product-variants.md, overview.md,
    sprite-overview.md, system-properties.md) as an existing OEM variant, not a new finding.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-05
  last_known_version: |
    client-l 1.1.0 — UNCHANGED since 2026-07-02 (metadata lastUpdated still 20260702091606,
    reconfirmed with --max-age 0 this run; no 1.1.1 or newer folder in the browse listing — highest
    version folder is still 1.1.0). client-m 1.2.2 (release; metadata lastUpdated still
    20260608030211 — reconfirmed unchanged, cache-bypassed). cxr-service-bridge 1.0 (release;
    metadata lastUpdated still 20260522063622 — reconfirmed unchanged, cache-bypassed).
  notes: |
    Re-verified 2026-07-05 by re-scraping maven-metadata.xml for all three artifacts with
    --max-age 0, plus a fresh browse-listing scrape of
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/client-l/ —
    highest folder present is still 1.1.0 (maven-metadata.xml itself last modified Thu Jul 02
    10:17:54 Z 2026, 776 bytes — unchanged from the 2026-07-04 finding). NO new client-l release
    since 2026-07-02; NO changelog published on developerdoc.rokid.com yet (portal /sdk CXR-L tab
    still shows only 1.0.4, dated 2026-06-29 — see developerdoc.rokid.com entry above). Local
    cxr-l/release-notes.md already carries a full binary-diff-derived v1.1.0 section (added
    2026-07-04 in commit ac1138a74), clearly labeled as provisional/unofficial pending a real
    changelog — no update needed to local docs this cycle since nothing new appeared upstream.
    client-m and cxr-service-bridge genuinely unchanged (confirmed via cache-bypassed re-fetch).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-05
  notes: Reconfirmed 2026-07-05 (25 URLs mapped, same count as 2026-07-04). Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, issues list only). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-05
  notes: Reconfirmed 2026-07-05 (73 URLs mapped vs 72 on 2026-07-04 — trivial issue-page churn, not a new repo). Same repo set as before — glass-docs (stale, 2020, Glass 1/2 era, not Sprite), UXR-docs (out of scope), remainder is Speech/OpenVoice/CloudApp/skill-kit/misc repos unrelated to Sprite/AR Glasses. Spot-checked RokidMobileSDKAndroidDemo (17 stars, 2 branches) directly this run since the name suggested possible CXR-M relevance — confirmed it is a 7-9 year old legacy Rokid Speech/voice-assistant mobile demo (last commit Aug 2019, "更新sdk到1.10.1"), unrelated to the com.rokid.cxr:client-m Maven artifact. No in-scope content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-05
  notes: Reconfirmed 2026-07-05. Map returned only the org root URL (1 URL) — no individual repos surfaced via map, same as 2026-07-03 and 2026-07-04. No actionable content this run. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval. Only a quick map-level reconfirmation was done this run per Leader's request; NOT scraped for content (out of the 8-source registry; scout does not silently extend source list).
  notes: |
    Quick reconfirm 2026-07-05 (map, --limit 20, no content scrape): returned 6 of the previously
    seen URLs (root, /academy, /?lang=en, /?lang=cn, /master?lang=en, /sdk?lang=en) — same page
    family as the 2026-06-28 discovery and 2026-07-04 recheck, no new page slugs observed, no
    material change to registration status. Root /?lang=cn still renders the "AIUI开启AI时代新未来"
    homepage headline. Still linked as the "Docs" CTA target from ar.rokid.com/doc and
    developer.rokid.com's Rokid Glasses hardware card. YodaOS-Master content remains at
    open.rokid.com/master — out of scope, skipped, not scraped.
    Registration status UNCHANGED — still awaiting explicit user decision on whether to register
    open.rokid.com (possibly as the canonical replacement for ar.rokid.com). No action taken.
