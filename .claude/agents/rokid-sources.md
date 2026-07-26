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
  last_checked: 2026-07-26
  last_known_version: CXR-L 1.0.4 shown on portal chain (see developerdoc.rokid.com below); Maven now has 1.1.0
  notes: |
    Re-verified 2026-07-26. Both root (/) and /doc now render the same English
    "AIUI: The Next Frontier" homepage (React SPA, static.rokidcdn.com/prod/rokid-developer-homepage
    build 1.1.0). Rokid Glasses card links "Docs" to https://open.rokid.com/sdk?lang=en, confirming
    open.rokid.com is now the Rokid-designated canonical developer-docs entry point (still
    UNREGISTERED pending user approval — see "New sources discovered" below). YodaOS-Master card
    remains alongside YodaOS-Sprite; skipped/out of scope per repo policy. No app-store /detail?appId=
    content is in scope (TV/Store apps, not Sprite/CXR docs).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-26
  last_known_version: CXR-L 1.0.4 (portal, updated 2026.06.25); CXR-M 1.1.0 (portal, updated 2026.04.01, business-gated); 眼镜端裸机开发 (bare-metal) 1.0.0 (portal, updated 2026.06.05)
  notes: |
    Re-verified 2026-07-26. /sdk NOW RENDERS full SDK selection content (previously only an SPA
    shell) with a comparison table: CXR-L SDK (Android/iOS) 公开 1.0.4 🟢活跃维护 (更新于 2026.06.25);
    CXR-M SDK 商务合作 1.1.0 🔒需商务对接 (更新于 2026.04.01, contact Glasses.BD@rokid.com); 眼镜端裸机开发
    公开 1.0.0 🟢活跃维护 (更新于 2026.06.05). The bare-metal version number (1.0.0) is NEW information —
    local cxr-baremetal/*.md docs are pinned "Doc version: v0.0.1 (2026-03-01)", a major-version gap.
    /sprite hardware-spec table and FAQ text still match yodaos/docs/sprite-overview.md exactly (word-
    for-word device spec table) — no drift there. Map still returns only 3 URLs (/sdk, /sprite, root).
    YodaOS-Master tab still present at top, skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-26
  notes: |
    Re-verified 2026-07-26 — STILL BROKEN. Both workspace hashes return OSS NoSuchKey on fresh scrape:
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey 6A6579DA47C61736378FF317
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M/CXR-S/裸机开发): NoSuchKey 6A6579DFA2FF263833BFC4B4
    No change from 2026-06-28 finding. Source remains unreachable; detail docs presumed migrated to
    developerdoc.rokid.com/open.rokid.com. Not usable for change detection.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-26
  notes: |
    Re-verified 2026-07-26. Root (/) now serves byte-for-byte the same AIUI homepage as ar.rokid.com
    (confirmed via scrape — identical markdown). The legacy GitBook path /docs returned "504 Gateway
    Time-out (nginx)" on live scrape this cycle (firecrawl map still lists old GitBook URLs like
    /docs/rokidos-linux-docs/..., /docs/5-enableVoice/rokid-vsvy-sdk-docs/..., but a direct scrape of
    /docs itself timed out with 504 — could not confirm live content this cycle). All GitBook content
    found via map is Speech SDK / RokidOS-Linux / Skill-platform material — out of scope for this repo
    regardless of reachability. No in-scope Sprite/AR Glasses content. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-26
  last_known_version: |
    client-l 1.1.0 (release; maven-metadata lastUpdated 20260718072455 — NEW release since last
      cycle, verified 2026-07-26; AAR client-l-1.1.0.aar uploaded 2026-07-02; portal/README/local
      docs all still pinned at 1.0.4)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged since last cycle)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260723084719 — timestamp bumped from
      20260522063622 but <release>/<latest> version number is still "1.0"; looks like a republish
      of the same version, not a new release. Not actionable as a version-drift finding.)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Browse path:
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>; verified via
    WebFetch (narrow manifest URL exception) 2026-07-26.
    client-l: 1.1.0 is a new stable release beyond the previously tracked 1.0.4 — confirmed present
    both in the directory listing (client-l/1.1.0/) and in maven-metadata.xml's <release> tag.
    client-m: 1.2.2 unchanged since 2026-06-09.
    cxr-service-bridge: version unchanged at 1.0, but Nexus re-touched the metadata file on
    2026-07-23 (no version bump) — informational only.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-26
  notes: Re-verified 2026-07-26 (fresh map, 25 URLs). Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-26
  notes: Re-verified 2026-07-26 (fresh map, 40 URLs). Same repo set as prior cycle — glass-docs (2020-era, pre-Sprite), UXR-docs (out of scope), Speech/OpenVoice/CloudApp/skill-* repos. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-26
  notes: Re-verified 2026-07-26. Map returned empty links array again; direct org-page scrape confirms only 2 inactive repos (BroadcastServiceDemo updated 2017-06-19, openCV3_demo updated 2017-02-17). No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Discovered 2026-06-28, re-confirmed as the canonical target 2026-07-26 (ar.rokid.com and
    developer.rokid.com both now link their "Docs" CTAs to open.rokid.com/sdk?lang=en). NOT
    crawled this cycle — no user approval obtained, and this is an unattended scheduled run with
    no user available to ask via AskUserQuestion. Prior notes (2026-06-28): open.rokid.com/sprite
    content matched developerdoc.rokid.com/sprite exactly; open.rokid.com/sdk was an SPA shell;
    /academy hosts 乐奇学院 (Rokid Academy) with CXR-L/Glasses courses (in scope) alongside UXR 3.0
    and AIUI courses (out of scope); YodaOS-Master content at /master (skipped, out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com — needs
    explicit user sign-off before Scout crawls it further.
