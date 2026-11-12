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
  last_checked: 2026-11-12
  last_known_version: CXR-L 1.1.2 (SDK card, 2026-11-12)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope).
    2026-11-12: root (English homepage) links CXR-L -> t.rokid.com/uwxdzi51 (-> custom.rokid.com
    84feb39f.../pc/us/663f26766e7348059905815bc022e1f7.html, doc version 1.0.4), CXR-S ->
    custom.rokid.com 57e35cd3.../pc/us/3fe1c87b945245bf8b6c50393f4da7b6.html (doc version 1.0),
    bare-metal -> custom.rokid.com ff28c865a9634876be98cbc293588460/pc/us/index.html (NEW workspace,
    doc version 1.0.0). Master-side SDK cards (UXR3.0/MRTK3/XRI/Unreal OpenXR/Native OpenXR/JSAR) skipped.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-11-12
  last_known_version: CXR-L 1.1.2 (updated 2026.09.08, Android + iOS), CXR-M 1.1.0 (now 商务对接 / business-contact only, updated 2026.04.01), 眼镜端裸机开发 1.0.0 (updated 2026.06.05)
  notes: |
    SDK landing page scraped 2026-06-28. /sdk still shows CXR-L 1.0.3 changelog; /sprite
    FAQ content matches local sprite-overview.md exactly — no drift detected. Map returns
    4 URLs: /sdk, /sprite, /sitemap.xml, root. YodaOS-Master tab skipped (out of scope).
    Portal may be the iframe-serve layer still; open.rokid.com is the new public-facing surface.
    2026-11-12: /sdk restructured into an "SDK 选型速览" comparison table (CXR-L / CXR-M / 裸机开发);
    CXR-S no longer has its own card on this page. Content identical to open.rokid.com/sdk?lang=zh.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-12
  notes: |
    REACHABLE AGAIN as of 2026-11-12 (was BROKEN 2026-06-28 with OSS NoSuchKey). /pc/us/ (English)
    pages render; /pc/cn/ pages for CXR-L return NoSuchKey or an empty 1.0.1 shell. Workspaces:
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): intro 663f26766e7348059905815bc022e1f7.html, doc v1.0.4.
      Sidebar: Introduction, Quick Start, Development Flow & State-Machine, Terms and Abbreviations,
      Feature Development, Version History (SPA nav; sub-page URLs not exposed to map).
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-S): 3fe1c87b945245bf8b6c50393f4da7b6.html, doc v1.0 (body did not render).
    - ff28c865a9634876be98cbc293588460 (眼镜端裸机开发, NEW): index.html, doc v1.0.0 (body did not render).
    Master workspace c88be4bcde4c42c0b8b53409e1fa1701 linked from homepage — skipped (out of scope).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-11-12
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-12
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20261105064636; versions 1.1.0, 1.1.1, 1.1.2 added since 2026-06-30; latest = 1.2.X-SNAPSHOT)
    client-m 1.2.2 (release; metadata lastUpdated 20261109123245 — release unchanged)
    cxr-service-bridge 1.8 (release; metadata lastUpdated 20261030075904; versions 1.1-1.8 added since 2026-06-30)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    2026-11-12: client-l jumped 1.0.4 -> 1.1.2 and cxr-service-bridge 1.0 -> 1.8. No official
    changelog for either was retrievable this run (portal "更新内容" sections do not expand
    under scraping; doc-site Version History page not reachable via SPA navigation).
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-11-12
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-11-12
  notes: Verified org (id 19773259). Official "Rokid" org. 2026-11-12 - most recently updated repo is armazpro-module-sdk-sample (2026-08-14; not Sprite/CXR, ignored). Checked 2026-06-29 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-12
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) on 2026-06-29 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible.

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
