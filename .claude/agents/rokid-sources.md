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
  last_checked: 2026-09-01
  last_known_version: CXR-L 1.0.4 (portal changelog, confirmed live), CXR-M 1.1.0 (business-gated)
  notes: |
    React SPA landing/router page — still shows the AIUI/AI Agent-focused homepage (English
    default), unchanged in structure since 2026-06-28. "Products" grid links out to
    aiui-global.rokid.com (Rokid Glasses/AIUI), open.rokid.com/master (Rokid AR / Master),
    and — NEW as of 2026-09-01 — https://x-docs.rokid.com/docs/en/ under a "Rokid Glass3"
    product tile. x-docs.rokid.com is an undiscovered doc domain — see "New sources discovered"
    below; it is NOT the same as CXR-L/CXR-M/CXR-S. YodaOS-Sprite tile still links to
    open.rokid.com/sprite; CXR-L tile now links via a t.rokid.com short link
    (t.rokid.com/uwxdzi51 -> custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/us/663f26766e7348059905815bc022e1f7.html,
    an English-language render of the CXR-L intro). YodaOS-Master tab remains; skipped
    (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-01
  last_known_version: |
    CXR-L 1.0.4 (official portal changelog, updated 2026.06.25: "增加音量控制/增加亮度控制" —
    volume + brightness control; matches local reverse-inferred setGlassVolume/setGlassBrightness)
    CXR-M 1.1.0 (business-gated tile, unchanged, portal lags Maven 1.2.2)
    眼镜端裸机开发 (bare-metal) 1.0.0 (updated 2026.06.05 — MAJOR jump from previously known 0.0.1;
    official changelog: "1.重构文档 2.新增Sample工程源码" — doc restructure + new sample source)
  notes: |
    /sdk?lang=zh live-scraped 2026-09-01 with 6-8s render wait (SPA needs JS settle time).
    SDK-selection table + per-SDK changelog accordions confirmed via click-through (actions).
    /sprite live-scraped 2026-09-01: hardware spec table for Rokid Glasses matches local
    yodaos/docs/sprite-overview.md exactly (no drift). BUT the FAQ section now shows only 4
    generic Q&As (CXR-M/CXR-S basics) — the richer CXR-L-specific FAQ (link-ready vs session
    construction, CustomView vs CustomApp, etc.) documented locally is NOT present in the live
    render. Confirmed on TWO independent live fetches (developerdoc.rokid.com/sprite and
    open.rokid.com/sprite, byte-identical). Flagged as P2 content drift in latest report.
    CXR-L doc tree (via custom.rokid.com workspace 84feb39f...) has been restructured with new
    sibling sections not in local repo: Quick Start, Development Flow & State-Machine,
    Terminology & Abbreviations (seen in both CN and EN sidebar renders). Feature Development
    and Version History map to existing api-reference.md / release-notes.md.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-01
  notes: |
    RESTORED as of 2026-09-01 (was BROKEN/NoSuchKey as of 2026-06-28). Both previously-known
    workspace hashes are reachable again with real content (verified via Firecrawl scrape,
    --wait-for 8-18s needed for the SPA to hydrate; short waits return only the shell):
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / old bare-metal): confirmed serving
      CXR-S brief, CXR-M intro, and the OLD (pre-restructure) bare-metal dev-guide page
      (13083daf77dd40bf84cf5c59711e987a.html, still v0.0.1-era content, single page).
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): confirmed serving CXR-L intro (CN + a
      newly-observed EN documentId 663f26766e7348059905815bc022e1f7), content matches local
      cxr-l/intro.md almost verbatim — no drift on this specific page.
    NEW workspace hash discovered: ff28c865a9634876be98cbc293588460 — this is the
    RESTRUCTURED bare-metal doc site (v1.0.0, English-native), reached via the ar.rokid.com
    homepage's "裸机开发" tile. Confirmed sections: Introduction, Quick Start, Sample Project
    and Pages, Glasses UI Design Guidelines (Bare Metal), Features (Keys/Wear/Fold Events, Raw
    Audio, Photo Capture, Video Recording, IMU and Sensors). This supersedes the old
    57e35cd3.../13083daf... page and is NOT yet reflected in local cxr-baremetal/*.md (which
    still cites the old page/v0.0.1). Root path (`/prod/rokid_web/` with no workspace hash)
    still fails to render (all Firecrawl engines failed) — expected, not a new failure; do not
    use as a monitor target. SPA pages need `--wait-for 8000` or more, or a Firecrawl
    `--actions` click on collapsed changelog/nav elements, to render past the loading shell.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-01
  notes: Unchanged — still redirects to the same AIUI-focused homepage as ar.rokid.com (confirmed byte-similar via live scrape 2026-09-01). No in-scope Sprite/AR Glasses content surfaced. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-01
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20260828083628 — MAJOR: three new releases
    1.1.0/1.1.1/1.1.2 published since the last check found only 1.0.4. <latest> tag is now
    1.2.X-SNAPSHOT. Verified via WebFetch of maven-metadata.xml AND the Nexus browse UI:
    client-l-1.1.2.aar = 171,307 bytes, dated 2026-08-29 (vs 70,543 bytes for 1.0.4 — AAR
    size more than doubled, implying substantial new surface area). Portal (developerdoc.rokid.com/sdk)
    still shows 1.0.4 as "最新版本" — portal is now 3 patch releases behind Maven.)
    client-m 1.2.2 (release; metadata lastUpdated 20260826091529 — version unchanged, only
    the metadata timestamp moved; no new release)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — unchanged)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    IMPORTANT: fetch maven-metadata.xml and the Nexus browse UI via WebFetch, not curl —
    curl is out of policy for this agent even though the CLI works; only Firecrawl and the
    narrow-endpoint WebFetch exception are permitted.
    client-l 1.1.0/1.1.1/1.1.2 are undocumented anywhere locally or on the official portal —
    highest-priority P1 in the 2026-09-01 report.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-01
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No new repos, no in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-01
  notes: Verified org (id 19773259). Official "Rokid" org. Map unchanged — NextForum, mingutils, RokidMobileSDK demos, CloudAppClient, better_jieba, community, docs, skill-java: all legacy Speech/OpenVoice/CloudApp repos, not Sprite. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-01
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) again on 2026-09-01 check — consistent with 2026-06-29. No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Re-verified 2026-09-01: open.rokid.com/sprite?lang=zh is byte-identical to
    developerdoc.rokid.com/sprite; open.rokid.com/sdk?lang=zh is byte-identical to
    developerdoc.rokid.com/sdk?lang=zh (same SDK-selection table, same version numbers, same
    changelog accordions). This is confirmed to be the same live content as developerdoc.rokid.com,
    just served under a different hostname. /academy hosts the "乐奇学院" (Rokid Academy)
    learning-platform with video courses — "Glasses 开发课程", "CXR-L SDK开发课程", and
    "Rokid x GPASS 眼镜端智能体研发课程" are in-scope-adjacent (Glasses/CXR-L), alongside
    out-of-scope UXR 3.0 / XR / AIUI courses. This is course video content, not reference docs —
    no direct 1:1 mapping to repo file structure; flagged informationally only, not P0.
    Recommend still registering open.rokid.com as the canonical mirror of developerdoc.rokid.com.

- url: https://x-docs.rokid.com/docs/en/
  kind: developer-portal
  covers: UNKNOWN — new SDK family, not cxr-m/s/l
  status: UNREGISTERED — requires user approval — PROMINENT NEW-FAMILY FLAG
  notes: |
    Discovered 2026-09-01 via a "Rokid Glass3" product tile on the ar.rokid.com homepage.
    Site title: "Rokid Sprite Enterprise" / "Glass3 SDK". VitePress-based static doc site,
    fully bilingual (EN default at /docs/en/, presumably /docs/zh/ or similar for CN).
    Confirmed via live scrape (Glass3 SDK Overview + Quick Start pages):
      - Targets NEW hardware "Rokid Glass3 AI glasses" (own debug cable, Android Studio shows
        device string "Rokid RG-glasses") — not explicitly one of the models listed in
        CLAUDE.md's in-scope hardware table (RV101/RV102/RV203/Rokid AI Glasses).
      - Uses a DIFFERENT Maven groupId entirely: `com.rokid.security:glass3.open.sdk:2.2.0-E`
        (glasses side) and `com.rokid.security:phone.sdk:2.2.0-E` (phone side) — NOT
        `com.rokid.cxr:*`. Same Maven host (maven.rokid.com) but a parallel artifact tree.
      - Architecture closely parallels CXR-M/CXR-S (Phone SDK <-> Glasses SDK over
        Bluetooth + Wi-Fi P2P, messaging/file transfer, media capture, remote control) but
        adds native AI/vision capabilities not in the current CXR docs: ASR/TTS/AI Chat,
        offline voice commands, face detection, license-plate recognition, Cloud OpenAPI.
      - Site branding explicitly says "Rokid Sprite Enterprise", suggesting this runs on
        YodaOS-Sprite (in-scope OS) but as a distinct "Enterprise" edition/SDK line for
        different/newer hardware — relationship to CXR-M/S/L (supersede vs. parallel
        enterprise tier vs. different hardware family) is NOT yet determined.
    DO NOT silently fold into cxr-m/cxr-s/cxr-l or treat as P0 for those directories — this
    needs an explicit Leader/user scoping decision (new top-level dir vs. out-of-scope hardware
    per CLAUDE.md's Master/ER exclusions, which do not currently mention "Glass3"). See report
    2026-09-01 "New top-level section discovered" for full detail.
