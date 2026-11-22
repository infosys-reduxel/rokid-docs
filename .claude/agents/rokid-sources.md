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
  last_checked: 2026-11-22
  last_known_version: CXR-L 1.1.2 (see developerdoc.rokid.com entry; this host only redirects)
  notes: |
    2026-11-22: scrape returns HTTP 200 and resolves to https://open.rokid.com/ (unregistered,
    see below). Map returns 0 links. No in-scope content is served from this host itself.
    YodaOS-Master link present on the landing page; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-11-22
  last_known_version: CXR-L 1.1.2 (2026-09-08), CXR-M 1.1.0 (2026-04-01, business-contact only), 眼镜端裸机开发 1.0.0 (2026-06-05)
  notes: |
    2026-11-22: map returns /sdk, /sprite, /sitemap.xml. /sdk scraped live (HTTP 200).
    The per-SDK changelog ("更新内容") and quick-start panels are collapsed by default; they
    only appear in a scrape that clicks every button[aria-expanded="false"] first.
    "查看文档" buttons open t.rokid.com short links that redirect to custom.rokid.com:
      CXR-L       https://t.rokid.com/rwwiot2x
      CXR-M       https://t.rokid.com/enfmqud
      眼镜端裸机开发 https://t.rokid.com/fwgr4uj5
    /sprite scraped live: the FAQ now shows 4 Q&As about CXR-M / CXR-S, while local
    yodaos/docs/sprite-overview.md carries 8 Q&As about CXR-L / bare-metal from the
    2026-06-08 fetch. Not actioned; needs a human decision on which FAQ is canonical.
    YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-22
  last_known_version: CXR-L doc set 1.1.2, CXR-M doc set 1.1.0, 眼镜端裸机开发 doc set 1.0.0
  notes: |
    REACHABLE again as of 2026-11-22 (was NoSuchKey on 2026-06-28). Entry pages, all HTTP 200:
    - CXR-L (workspace 84feb39f8ef141b0ad0326f902ab881f):
      /pc/cn/fa92e9303bc848e6b4a20c062f2c792b.html — "版本1.1.2". Sidebar: 简介, 快速开始,
      开发流程与状态机, 术语与缩写, 功能开发 (Android: SDK 导入, 鉴权, 连接与会话, 设备控制,
      眼镜端自定义View, 眼镜端自定义应用, 音频, 拍照, 自定义指令, 眼镜端按键与系统广播; iOS;
      眼镜端 — the last two groups not expanded), 版本历史
      (?documentId=2b189da5908f4e48b2d1b749a38888a5).
    - CXR-M / CXR-S (workspace 57e35cd3ae294d16b1b8fc8dcbb1b7c7):
      /pc/cn/0ae2153f5e32433e9f7ce43a6b5c922a.html — "版本1.1.0". Sidebar: 简介, 历史版本.
    - 眼镜端裸机开发 (NEW workspace ff28c865a9634876be98cbc293588460):
      /pc/cn/index.html — "版本1.0.0". Sidebar: 简介, 快速开始, Sample工程与页面说明,
      眼镜UI设计规范, 功能开发 (按键与佩戴折叠, 原始音频, 拍照, 录像, 相机预览描边, IMU与传感器).
    The site is an SPA with an ant-tree sidebar; sub-pages are reached by clicking the tree
    node (.ant-tree-node-content-wrapper[title="..."]) after ~7 s render wait, which sets
    ?documentId=<hash>. Only 简介 (CXR-L, CXR-M, bare-metal) and CXR-L 版本历史 were read on
    2026-11-22; every other sub-page is untranslated backlog.
    A further workspace c88be4bcde4c42c0b8b53409e1fa1701 is linked from open.rokid.com; its
    subject was not identified and it was not followed.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-11-22
  notes: 2026-11-22 map succeeded but returned 0 links (consistent with the 2026-06-29 finding that the host redirects to open.rokid.com). No page content retrieved from this host. No in-scope Sprite/AR Glasses content surfaced. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-22
  last_known_version: |
    client-l 1.1.2 (release; latest 1.2.X-SNAPSHOT; metadata lastUpdated 20261105064636)
    client-m 1.2.2 (release and latest; metadata lastUpdated 20261109123245 — version unchanged, timestamp moved)
    cxr-service-bridge 1.9 (release and latest; metadata lastUpdated 20261120093221 — was 1.0; versions 1.1–1.9 now listed, no changelog found)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    Firecrawl returns the XML with tags stripped; read latest/release from element order.
    client-l: portal and Maven agree on 1.1.2 as of 2026-11-22.
    client-m: 1.2.2; portal still on 1.1.0 and now business-contact only.
    cxr-service-bridge: 1.9 is undocumented on the portal; local cxr-s docs still pin
    1.0-20260522.063600-105. Not actioned (no upstream changelog or doc to translate).
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-11-22
  notes: Verified org (id 57519491). 2026-11-22 map returned 8 URLs, all under glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-11-22
  notes: Verified org (id 19773259). 2026-11-22 map returned 29 URLs — Speech/OpenVoice/mobile-SDK/CloudApp repos, glass-docs (old Glass 1/Glass 2 era), UXR-docs (out of scope). No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-11-22
  notes: Verified org (id 25831739). 2026-11-22 map returned 7 URLs across 2 repos (openCV3_demo, BroadcastServiceDemo). No actionable content. May have private repos not visible.

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
