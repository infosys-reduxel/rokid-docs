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
  last_checked: 2026-07-07
  last_known_version: CXR-L 1.0.4 shown on portal (developerdoc.rokid.com); Maven still has client-l 1.1.0 as of 2026-07-02, unchanged since last cycle (see maven.rokid.com entry)
  notes: |
    Reconfirmed live 2026-07-07 (firecrawl map --limit 60): 36 URLs, same count as 2026-07-05.
    Still /doc, /?lang=en, and the Rokid Store app-catalog detail pages (/detail?appId=...) for
    Rokid Station/Air apps (out-of-scope hardware), plus a few /all?type=... category-listing
    pages already present in prior counts. No structural change, no new in-scope surface. Real
    Sprite/CXR doc surfaces remain at developerdoc.rokid.com (registered) and open.rokid.com
    (unregistered, see bottom section).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-07
  last_known_version: CXR-L 1.0.4 (portal changelog, dated 2026-06-29 — UNCHANGED this run, portal still has NOT published anything for Maven's client-l 1.1.0, now 5 days stale relative to the 2026-07-02 Maven upload). CXR-M tab and 眼镜端裸机开发 tab content still not independently reverified this run (JS-tab click continues to fail via Firecrawl actions; known SPA limitation, unchanged from prior runs).
  notes: |
    RE-VERIFIED LIVE 2026-07-07 with --only-main-content --max-age 0 --wait-for 3000 (per Leader's
    explicit instruction this cycle, given the SPA's rendering-timing sensitivity noted 2026-07-05).
    /sdk default (CXR-L) tab content is IDENTICAL to the 2026-07-05 capture — still only the v1.0.4
    changelog (dated 2026.06.29), same 7-item V1.0.4 changelog text (setGlassBrightness/
    setGlassVolume, "设备控制" chapter, iOS parity APIs) plus the V1.0.1 initial-release entry.
    NO portal mention of client-l 1.1.0 anywhere on /sdk. This is now the THIRD consecutive
    confirmed-live cycle (07-05, 07-07 — note 07-06 was not run per the registry's own
    last_checked gap) where Maven has 1.1.0 and the portal changelog has not caught up — genuine,
    persistent version-lag gap, not a portal bug (portal simply hasn't published a changelog yet).

    RE-VERIFIED LIVE the /sprite FAQ drift with --max-age 0 --wait-for 3000 as instructed. Fresh
    scrape STILL shows the SAME 4 short Q&As, all CXR-M/CXR-S focused ("CXR-M SDK 的主要功能包含
    哪些？", "CXR-M SDK 目前支持哪些设备使用？", "CXR-S SDK 的主要功能包含哪些？", "仅使用 CXR-M SDK
    开发移动端应用时，需要打开 Rokid Glasses 的开发者权限吗？"). This continues to NOT match what
    local yodaos/docs/sprite-overview.md's own changelog note claims was captured live on
    2026-06-08 (an 8-Q&A CXR-L/bare-metal-focused set). Device spec table (Rokid Glasses:
    143x44x160.5mm, Qualcomm AR1, 2GB/32GB, SONY IMX681 etc.) and Developer Toolkit section on
    /sprite still match local content exactly (no drift there).
    PERSISTENT ACROSS MULTIPLE CONSECUTIVE VERIFIED-LIVE RUNS (2026-07-03, 07-04, 07-05, and now
    07-07) — reported plainly per Leader's instruction, NOT resolved here. This needs human
    judgment: either (a) the 2026-06-08 capture that produced the local 8-Q&A text was wrong or
    came from a different source/session, or (b) upstream genuinely rolled back the FAQ and never
    restored the CXR-L/bare-metal set. Map still returns only /sdk, /sprite, root (3-4 URLs).
    YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-07
  notes: |
    STILL BROKEN, re-verified live 2026-07-07 (cache bypass --max-age 0). Both previously valid
    workspace hashes still return OSS NoSuchKey with fresh RequestIds this run (confirming a live
    re-check, not a cached replay):
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey (RequestId
      6A4C6D31393D1B363370E107 this run)
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey (RequestId 6A4C6D34D1170E3734588E86
      this run)
    Source remains unreachable; new workspace hashes still unknown. Not usable as a monitor
    candidate. Fourth consecutive run confirming this (2026-07-03, 07-04, 07-05, 07-07).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-07
  notes: |
    Reconfirmed live 2026-07-07 (firecrawl map --limit 100): 19 URLs, same count and same page set
    as 2026-07-05 — rokidos-linux-docs, rokid-homebase-docs, Skills Kit (2-RokidDocument/
    3-ApiReference/4-Terms/5-enableVoice/8-app), contact-us — a legacy GitBook doc tree for
    RokidOS-Linux, HomeBase, and the Speech Skills platform. None relate to YodaOS-Sprite,
    CXR-M/S/L, or Rokid Glasses hardware. No action items, no change from prior cycles.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-07
  last_known_version: |
    client-l 1.1.0 — UNCHANGED since 2026-07-02 (maven-metadata.xml lastUpdated still
    20260702091606, re-verified live this run with --max-age 0; browse-listing directory index at
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/client-l/
    still lists 1.1.0 as the highest version folder present, maven-metadata.xml file timestamp
    still "Thu Jul 02 10:17:54 Z 2026", 776 bytes — no new 1.1.1+ folder). client-m and
    cxr-service-bridge not independently re-scraped this run (out of scope for this cycle's
    specific ask, which was client-l only); last known values from 2026-07-05 unchanged per
    registry baseline (client-m 1.2.2, cxr-service-bridge 1.0) — NOT re-verified live this cycle.
  notes: |
    Re-verified LIVE 2026-07-07 for client-l specifically (per Leader's explicit ask): re-scraped
    both maven-metadata.xml (--max-age 0) and the browse-listing HTML index for
    com/rokid/cxr/client-l/ — highest folder present is still 1.1.0, no 1.1.1 or newer. Cross-
    checked against developerdoc.rokid.com/sdk (see that entry): portal STILL has not published an
    official changelog for 1.1.0 — CXR-L tab still shows only v1.0.4 dated 2026-06-29. Local
    cxr-l/release-notes.md's provisional/binary-diff-derived v1.1.0 section (added 2026-07-04,
    commit ac1138a74) therefore remains correctly labeled "provisional pending official
    confirmation" — no update needed to local docs this cycle, nothing changed upstream on either
    axis (no new Maven version, no new portal changelog).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-07
  notes: Reconfirmed live 2026-07-07 (25 URLs mapped, same count and same set as 2026-07-05). Still only 2 public repos, UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, issues list only). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-07
  notes: Reconfirmed live 2026-07-07 (72 URLs mapped vs 73 on 2026-07-05 — trivial issue-page churn, not a repo change; repo-name extraction confirms the same ~33-repo set as before: glass-docs, UXR-docs, CloudAppClient, RokidMobileSDKAndroidDemo/iOSDemo, RokidSDK-Swift, rokid-openvoice-*, skill/speech/tts demos, docs, community, etc.). Same conclusion as prior cycles — legacy Speech/OpenVoice/CloudApp/skill-kit/misc repos, glass-docs is stale 2020-era Glass 1/2 (not Sprite), UXR-docs is out of scope. No in-scope content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-07
  notes: |
    Reconfirmed live 2026-07-07, but with a NEW result this cycle: firecrawl map returned 0 URLs
    (previous 3 cycles all returned exactly 1, the org root). Because that's a meaningful change in
    map behavior (not just a count wobble), did a follow-up direct scrape of the org root page
    (--max-age 0) to make sure this wasn't a silent detection failure — the scrape succeeded (200,
    real rendered content, not an error page) and surfaced 2 public repos that map alone had not
    been surfacing: openCV3_demo (Java, last updated Feb 2017) and BroadcastServiceDemo (Java, last
    updated Jun 2017). Both are ~9-year-old legacy demos unrelated to CXR-M/S/L or Sprite; org page
    states "no public members." No in-scope content. Net: org is reachable and unchanged in
    substance; the map-vs-scrape URL-count discrepancy looks like a Firecrawl map crawl-depth/JS
    quirk on this specific org page, not a real content change — flagging so future cycles don't
    misread "map returned 0" as "org vanished."

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval. Only a quick map-level reconfirmation was done this run per Leader's request; NOT scraped for content (out of the 8-source registry; scout does not silently extend source list).
  notes: |
    Quick reconfirm 2026-07-07 (map, --limit 20, no content scrape): returned 9 URLs this run —
    root, /sitemap.xml, /sprite?lang=zh, /sdk?lang=zh, /academy, /sdk?lang=en, /?lang=en, /?lang=cn,
    /master?lang=en. That is 3 MORE distinct URL slugs than the 6 seen on 2026-07-05
    (/sitemap.xml, /sprite?lang=zh, /sdk?lang=zh are new-to-map this cycle). Notably
    /sprite?lang=zh suggests this unregistered host may carry its own Sprite-specific doc surface
    distinct from developerdoc.rokid.com/sprite — potentially relevant to the in-scope
    YodaOS-Sprite/CXR-M/S/L scope this repo tracks. Per Leader's instruction this was a map-level
    check ONLY — no content was scraped, no workspace hash logged, nothing added to the repo.
    Flagging the new slugs for Leader/user attention; registration status UNCHANGED — still
    awaiting explicit user decision on whether to register open.rokid.com. YodaOS-Master content
    remains at open.rokid.com/master — out of scope, skipped, not scraped.
