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
  last_checked: 2026-07-04
  last_known_version: CXR-L 1.0.4 shown on portal (developerdoc.rokid.com); Maven now has client-l 1.1.0 as of 2026-07-02 (see maven.rokid.com entry)
  notes: |
    Reconfirmed 2026-07-04. firecrawl map returned 39 URLs, all Rokid Store app-catalog detail
    pages (/detail?appId=...) for Rokid Station/Air apps — out-of-scope hardware, no CXR SDK doc
    URLs surfaced via map (SPA client routes aren't indexed). Same pattern as 2026-07-03. No new
    in-scope content discovered here; real Sprite/CXR doc surfaces remain at developerdoc.rokid.com
    (registered) and open.rokid.com (unregistered, see bottom section).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-04
  last_known_version: CXR-L 1.0.4 (portal changelog, dated 2026-06-29 — UNCHANGED this run, portal has NOT yet published anything for Maven's new client-l 1.1.0). CXR-M tab and 眼镜端裸机开发 tab content still not independently reverified this run (JS-tab click continues to fail via Firecrawl actions — tried `text=CXR-M SDK` and `text=CXR-M` selectors, both "Element not found"; known SPA limitation, unchanged from prior runs).
  notes: |
    CONFIRMED 2026-07-04: /sdk default (CXR-L) tab content byte-for-byte matches the 2026-07-03
    capture — still only the v1.0.4 changelog (dated 2026.06.29), device-control APIs
    (setGlassBrightness/setGlassVolume, range 0-15), "设备控制" chapter, iOS parity APIs. No portal
    mention of client-l 1.1.0 (which Maven shows released 2026-07-02 — see maven.rokid.com entry;
    this is a genuine version-lag gap, not yet a portal content problem since portal simply hasn't
    published a changelog yet).
    RECONFIRMED content drift on /sprite FAQ (still present 2026-07-04, unchanged from 2026-07-03
    finding): fresh scrape shows only 4 short Q&As, all CXR-M/CXR-S focused (e.g. "CXR-M SDK 的主要
    功能包含哪些？", "CXR-M SDK 目前支持哪些设备使用？"), matching the OLDER FAQ content that local
    yodaos/docs/sprite-overview.md's own changelog note says was "replaced" as of 2026-06-08 with an
    8-Q&A CXR-L/bare-metal-focused set. Device-spec table and Developer Toolkit section on /sprite
    still match local content exactly (no drift there). Flagged as content drift needing human
    judgment (possible upstream rollback or A/B-served variant), not auto-applied — carried forward
    unresolved for the second consecutive run.
    Map still returns only /sdk, /sprite, root (3-4 URLs). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-04
  notes: |
    STILL BROKEN as of 2026-07-04 (re-verified with cache bypass --max-age 0 to rule out stale
    Firecrawl cache; got fresh RequestIds this run, confirming a live re-check, not a cached replay).
    Both previously valid workspace hashes still return OSS NoSuchKey:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey (RequestId
      6A48790128E0123738EF6D74 this run)
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey (RequestId 6A487904F5E96E3030F58467
      this run)
    Source remains unreachable; new workspace hashes still unknown. Not usable as a monitor
    candidate.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-04
  notes: |
    Reconfirmed 2026-07-04 via direct scrape of root: serves the identical AIUI/AI Agent
    rokid-developer homepage as ar.rokid.com. No in-scope Sprite/AR Glasses content on the live
    root; "Docs" CTA for Rokid Glasses links to open.rokid.com/sdk?lang=en (unregistered source).
    Not re-mapped this run (map output for this host was already flagged unverified/stale-looking
    in the 2026-07-03 note); low priority, GitBook content is legacy Speech/HomeBase (out of
    Sprite/CXR scope).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-04
  last_known_version: |
    client-l 1.1.0 — NEW RELEASE, published 2026-07-02 (metadata lastUpdated 20260702091606).
    IMPORTANT: an initial scrape without cache-bypass returned a STALE cached maven-metadata.xml
    (lastUpdated 20260625070819, release=1.0.4) that would have under-reported this. Re-scraping
    with --max-age 0 surfaced the true current metadata (release=1.0.4 -> now release=1.1.0).
    Confirmed via direct browse listing: https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/client-l/
    shows a real 1.1.0 folder with client-l-1.1.0.aar (1,286,574 bytes, modified 2026-07-02
    10:18:18Z), .module, and .pom files — not a snapshot placeholder.
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — reconfirmed unchanged, cache-bypassed)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — reconfirmed unchanged, cache-bypassed)
  notes: |
    Re-verified 2026-07-04 by scraping maven-metadata.xml for all three artifacts, WITH --max-age 0
    cache bypass after noticing the first pass returned identical-looking content to 2026-07-03
    (a red flag). This surfaced a genuinely NEW release: client-l 1.1.0, uploaded 2026-07-02,
    09:16-11:31 UTC — AAR size jumped from 70,543 bytes (1.0.4) to 1,286,574 bytes (+1724%), a much
    larger jump than any prior client-l point release, suggesting substantial new functionality (not
    just a device-control patch like 1.0.3->1.0.4). NO changelog published on developerdoc.rokid.com
    yet (portal /sdk CXR-L tab still shows only 1.0.4, dated 2026-06-29). Local cxr-l/release-notes.md
    already carries a forward-looking note (added prior cycle) flagging "Maven also lists a newer
    client-l:1.1.0 release (uploaded 2026-07-02) with no changelog published yet — not documented
    below" — so local docs anticipated this, but the actual 1.1.0 API surface is still undocumented
    pending either an official changelog or a binary-diff pass against the now-available 1.1.0 AAR.
    RECOMMENDATION: cache-bypass (--max-age 0) should be standard practice for all future
    maven-metadata.xml checks — the previous 2026-07-03 registry note claiming "byte-identical,
    UNCHANGED" for client-l was itself based on stale cached content and should not have been
    trusted as confirmation of no change.
    client-m and cxr-service-bridge genuinely unchanged (confirmed via cache-bypassed re-fetch, not
    just first-pass cache hit).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-04
  notes: Reconfirmed 2026-07-04 (25 URLs mapped). Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, issues list only). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-04
  notes: Reconfirmed 2026-07-04 (72 URLs mapped, same count as 2026-07-03). Same repo set as before — glass-docs (stale, 2020, Glass 1/2 era, not Sprite), UXR-docs (out of scope), remainder is Speech/OpenVoice/CloudApp/skill-kit/misc repos unrelated to Sprite/AR Glasses. No in-scope content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-04
  notes: Reconfirmed 2026-07-04. Map returned only the org root URL (1 URL) — no individual repos surfaced via map, same as 2026-07-03. No actionable content this run. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval. Only a quick map-level reconfirmation was done this run per Leader's request; NOT scraped for content (out of the 8-source registry; scout does not silently extend source list).
  notes: |
    Quick reconfirm 2026-07-04: map returned 8 URLs (root, /?lang=en, /?lang=cn, /sdk?lang=zh,
    /sdk?lang=en, /sprite?lang=zh, /academy, /master?lang=en) — same page set as the 2026-06-28
    discovery, no new page slugs. Root /?lang=cn now renders an "AIUI开启AI时代新未来" (AIUI: new
    future of the AI era) homepage headline — same page family as developer.rokid.com/ar.rokid.com,
    reinforcing this is the live canonical portal. NOT independently re-scraped for CXR-L/Sprite
    content this run (would require user approval to treat as an action source). Still linked as
    the "Docs" CTA target from both ar.rokid.com and developer.rokid.com's Rokid Glasses hardware
    cards. YodaOS-Master content remains at open.rokid.com/master — out of scope, skipped, not
    scraped.
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com — awaiting
    user decision.
