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
  last_checked: 2026-09-06
  last_known_version: CXR-L 1.0.3 (portal still shows 1.0.3 as of 2026-06-28; Maven has 1.1.2)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope).
    2026-09-06 re-check: map now returns a single unrelated /detail?appId=... link (not a
    Sprite/CXR surface). No in-scope content found here; developerdoc.rokid.com remains the
    working registered source for SDK version indicators.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-06
  last_known_version: CXR-L 1.0.4 (shown on portal, "更新于 2026.06.25"; Maven has 1.1.2), CXR-M 1.1.0 (portal lags Maven 1.2.2), 眼镜端裸机开发 1.0.0 ("更新于 2026.06.05", up from 0.0.1)
  notes: |
    SDK landing page re-scraped 2026-09-06 (live, --only-main-content). Card versions now
    read: CXR-L "最新版本 1.0.4"; CXR-M "最新版本 1.1.0"; 眼镜端裸机开发 "最新版本 1.0.0"
    (was recorded as 0.0.1/2026-03-01 in the previous cycle — confirmed live version-string
    bump, actioned as P1 in cxr-baremetal/development-guide.md, but underlying content is
    unreachable — see custom.rokid.com entry below). /sprite content still SPA-shell only
    without JS execution; map still returns 4 URLs (/sdk, /sdk?lang=zh, /sprite?lang=zh,
    root). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-06
  notes: |
    STILL BROKEN as of 2026-09-06 (re-verified live). Workspace hash
    57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发) returns OSS NoSuchKey
    6A9CD89D1D107438325D6A79 — same failure mode as the 2026-06-28 check, now confirmed
    stable/long-lived rather than transient. The OSS bucket
    (rokid-ar-platform.oss-cn-hangzhou.aliyuncs.com) still does not contain this workspace
    key. This blocks translating the confirmed CXR-L 1.1.0/1.1.1/1.1.2 and 眼镜端裸机开发
    1.0.0 version bumps from an official source — both were instead handled via binary-diff
    (CXR-L) or left as an unverifiable documentation-gap note (bare-metal). Root path returns
    HTTP 415. Monitor candidate remains removed — source is not reachable.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-06
  notes: Re-checked 2026-09-06 (live map). Still GitBook-era Speech/HomeBase/skill-review content (/voice, /docs/8-app/..., /docs/rokid-homebase-docs/v2/..., /docs/4-TermsAndAgreements/...). No in-scope Sprite/AR Glasses content surfaced. Low priority, unchanged from prior cycle.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-06
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20260828083628 — 3 new releases since last check: 1.1.0 uploaded 2026-07-02, 1.1.1 uploaded 2026-08-14, 1.1.2 uploaded 2026-08-28. Portal still shows 1.0.4. Actioned as P1 via binary diff in cxr-l/release-notes.md.)
    client-m 1.2.2 (release; metadata lastUpdated 20260902061433 — version unchanged since 2026-06-09, but metadata touched 2026-09-02; no new version string appeared in <versions>, no action taken)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326, up from 20260522063622 — release/latest tag unchanged at "1.0"; a new dependency snapshot 1.0-20260715.121510-107 appeared as a transitive dependency of client-l 1.1.1/1.1.2, but the published "1.0" release pointer itself did not change; no cxr-s doc version pin needs updating)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: verified 2026-09-06 via maven-metadata.xml + HTTP Last-Modified headers on each
    AAR (1.1.0: 2026-07-02, 1.1.1: 2026-08-14, 1.1.2: 2026-08-28). Portal still shows 1.0.4.
    Binary-diffed all four versions (1.0.4/1.1.0/1.1.1/1.1.2) via classes.jar + javap -p;
    findings written to cxr-l/release-notes.md.
    client-m: 1.2.2 unchanged since 2026-06-09 (metadata lastUpdated bump only). Portal still on 1.1.0.
    cxr-service-bridge: release pointer unchanged at 1.0; underlying snapshot rebuilt 2026-07-15.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-06
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Re-checked 2026-09-06 — still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-06
  notes: Verified org (id 19773259). Official "Rokid" org. Re-checked 2026-09-06 — still mostly Speech/OpenVoice/NextForum/mingutils repos, plus UXR-docs (out of scope). No in-scope Sprite/AR Glasses content found. Low priority, unchanged from prior cycle.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-06
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) again on 2026-09-06 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible.

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
    2026-09-06 (unattended cycle): left unregistered and not re-scraped — registering a new
    source requires explicit user approval per scout.md, which unattended runs cannot obtain.
    Still recommended for a human to approve on the next interactive cycle.
