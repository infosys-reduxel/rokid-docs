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
  last_checked: 2026-11-13
  last_known_version: n/a (no SDK version surfaced; see developerdoc.rokid.com)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope).
    2026-11-13: /sdk and /sprite both render the open.rokid.com homepage (Spatial Joy 2026 /
    AIUI banners); map returned 0 links. No in-scope SDK content on this host.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-11-13
  last_known_version: CXR-L 1.1.2 (2026-09-08), CXR-M 1.1.0 (2026-04-01; portal lags Maven 1.2.2), 眼镜端裸机开发 1.0.0 (2026-06-05)
  notes: |
    SDK landing page scraped 2026-06-28. /sdk still shows CXR-L 1.0.3 changelog; /sprite
    FAQ content matches local sprite-overview.md exactly — no drift detected. Map returns
    4 URLs: /sdk, /sprite, /sitemap.xml, root. YodaOS-Master tab skipped (out of scope).
    Portal may be the iframe-serve layer still; open.rokid.com is the new public-facing surface.
    2026-11-13: /sdk redesigned as three cards (CXR-L / CXR-M / 眼镜端裸机开发) with collapsed
    quick-start and changelog panels — a plain scrape misses them; expand every
    button[aria-expanded="false"] via a Firecrawl executeJavascript action first.
    CXR-L card: 1.1.2, changelog V1.1.2 (2026.09.08) and V1.0.4 (2026.06.25) now official.
    Bare-metal card: 1.0.0 (2026.06.05). CXR-M card: 1.1.0, marked business-contact only.
    "查看文档" buttons open t.rokid.com short links: rwwiot2x (CXR-L), enfmqud (CXR-M),
    fwgr4uj5 (bare-metal), which redirect to custom.rokid.com (see next entry).
    /sprite: spec table unchanged; FAQ rendered 4 CXR-M/CXR-S Q&As, unlike the CXR-L/bare-metal
    FAQ in local sprite-overview.md — unconfirmed whether a real change or a render variant.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-13
  notes: |
    WORKING again as of 2026-11-13 (was BROKEN 2026-06-28 with OSS NoSuchKey).
    Workspaces, reached via the t.rokid.com links on developerdoc.rokid.com/sdk:
    - CXR-L v1.1.2: 84feb39f8ef141b0ad0326f902ab881f/pc/cn/fa92e9303bc848e6b4a20c062f2c792b.html
      39-node tree: 简介 / 快速开始 / 开发流程与状态机 / 术语与缩写 / 功能开发 (Android x10,
      iOS x9, 眼镜端 [CXR-S] x9) / 版本历史. Full rewrite for the 1.1.x API.
    - CXR-M v1.1.0: 57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/0ae2153f5e32433e9f7ce43a6b5c922a.html
      Only 2 pages remain public (简介, 历史版本); detail chapters withdrawn.
    - 眼镜端裸机开发 v1.0.0: ff28c865a9634876be98cbc293588460/pc/cn/index.html (new workspace
      hash; no longer shares the CXR-M workspace). 10 pages: 简介 / 快速开始 / Sample工程与页面说明 /
      眼镜UI设计规范 / 按键与佩戴折叠 / 原始音频 / 拍照 / 录像 / 相机预览描边 / IMU与传感器.
    Pages are a client-rendered SPA: scrape needs ~9 s wait; individual pages are selected by
    clicking the ant-tree nav node (URL gains ?documentId=<id>). Root path not re-tested.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-11-13
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-13
  last_known_version: |
    client-l 1.1.2 (release; latest 1.2.X-SNAPSHOT; metadata lastUpdated 20261105064636)
    client-m 1.2.2 (release; metadata lastUpdated 20261109123245 — release unchanged)
    cxr-service-bridge 1.8 (release; metadata lastUpdated 20261030075904)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    2026-11-13 read of maven-metadata.xml:
    client-l: new releases 1.1.0, 1.1.1, 1.1.2 since 1.0.4. Portal now matches (1.1.2).
    client-m: release still 1.2.2 (lastUpdated moved; cause not determined). Portal on 1.1.0.
    cxr-service-bridge: new releases 1.1–1.8 (incl. 1.7.1–1.7.3) since 1.0. No official
    changelog found; the CXR-L doc's 眼镜端 SDK 导入 page still pins 1.0-20250519.061355-45.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-11-13
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items. 2026-11-13: still only UXR-docs (2023-05-07) and glass2-docs (2021-09-09).

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-11-13
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-29 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority. 2026-11-13 repo list (sorted by push): most recent are armazpro-module-sdk-sample (updated 2026-08-14; not inspected, scope undetermined) and docs (2026-03-28, legacy). Everything else last pushed 2023 or earlier.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-13
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) on 2026-06-29 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible. 2026-11-13: 2 public repos (BroadcastServiceDemo 2017-06-19, openCV3_demo 2017-02-17); nothing actionable.

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
