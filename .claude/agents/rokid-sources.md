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
  last_checked: 2026-09-23
  last_known_version: CXR-L 1.1.2 (per developerdoc.rokid.com/sdk, 2026-09-08); Maven confirms client-l release 1.1.2
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope). Not re-crawled this cycle — see
    developerdoc.rokid.com entry below for the live check; behavior unchanged since 2026-06-28.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-23
  last_known_version: CXR-L 1.1.2 (2026-09-08), CXR-M 1.1.0 (portal lags Maven 1.2.2, unchanged), 眼镜端裸机开发 1.0.0 (2026-06-05)
  notes: |
    SDK landing page re-scraped 2026-09-23 with --wait-for to render the client-side SDK cards
    (static scrape without wait only returns the nav shell). /sdk now shows CXR-L 1.1.2 —
    major session-management rewrite (CxrSessionManager / SessionConfig / ISessionLifecycleCbk,
    5-state machine), full changelog retrieved via the in-page "更新内容" accordion
    (button[aria-controls*='CXR-L SDK'][aria-controls*='changelog-panel']). Reconciled against
    this repo's earlier binary-diff v1.0.4 entry — official v1.0.4 changelog only credits
    volume/brightness controls; the ICXRSessionCbk material found via binary diff was preview
    code, superseded by v1.1.2's ISessionLifecycleCbk. 眼镜端裸机开发 (bare-metal) card shows
    v1.0.0 (2026-06-05, doc restructure + new sample project), up from the 0.0.1 baseline this
    repo had pinned — repo docs updated. CXR-M card unchanged (1.1.0, business-gated, updated
    2026-04-01) — Maven still ahead at 1.2.2, no new drift since last cycle. /sprite FAQ content
    changed structurally: upstream replaced the detailed CXR-L FAQ with a short CXR-M/CXR-S
    overview FAQ (4 Q&As) — repo's sprite-overview.md updated to match. Hardware spec table on
    /sprite re-verified unchanged. Map still returns 4 URLs: /sdk, /sprite, /sitemap.xml, root.
    YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-30
  notes: |
    BROKEN as of 2026-06-28. Both previously valid workspace hashes return OSS NoSuchKey:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey 6A408F766EB57F3436AE9E82
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey 6A408F1CC38F5534389093AE
    The OSS bucket (rokid-ar-platform.oss-cn-hangzhou.aliyuncs.com) no longer contains these
    workspace keys. Detail docs appear to have been migrated. New workspace hashes unknown.
    Root path returns HTTP 415. Monitor candidate removed — source is not reachable.
    Not re-checked 2026-09-23 (previously confirmed unreachable; no indication it has returned;
    cxr-s/design-spec.md and cxr-baremetal/*.md still cite it as their (now-dead) origin).

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority. Not re-checked 2026-09-23 (low priority, no signal of change).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-23
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20260910022017 — new release found 2026-09-23,
    up from 1.0.4 on 2026-06-28; official changelog now published, see developerdoc.rokid.com entry)
    client-m 1.2.2 (release; metadata lastUpdated 20260902061433 — version unchanged since
    2026-06-08, metadata re-timestamped only; portal still shows 1.1.0)
    cxr-service-bridge 1.4 (release; metadata lastUpdated 20260922073949 — new release found
    2026-09-23, up from 1.0 on 2026-06-30; versions 1.1/1.2/1.3/1.4 all published in between;
    no official changelog located for any of them)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>. Verified via
    WebFetch of the raw maven-metadata.xml (narrow manifest endpoint, permitted per scout.md
    boundaries) on 2026-09-23.
    client-l: 1.1.2 confirmed both via maven-metadata.xml and the official developerdoc.rokid.com
    changelog (published 2026-09-08). Repo docs updated this cycle (P1).
    client-m: 1.2.2 unchanged. Portal still on 1.1.0. No action.
    cxr-service-bridge: jumped 1.0 -> 1.4. No official changelog surfaced anywhere (no dedicated
    CXR-S card on the SDK landing page; custom.rokid.com, the prior detail-doc host, is broken).
    Repo version pins updated (P1) with an explicit "no changelog available" caveat — do not
    treat the version bump as implying any specific API change without further evidence.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-23
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Re-checked 2026-09-23 — still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-23
  notes: Verified org (id 19773259). Official "Rokid" org. Re-checked 2026-09-23 — repo list unchanged from 2026-06-29 check (glass-docs last commit still 2020-07-13; UXR-docs out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-23
  notes: Verified org (id 25831739). Re-checked 2026-09-23 — map now returns 1 URL (BroadcastServiceDemo/gradlew, the same long-inactive 2017 repo previously noted); no new repos. No actionable content. May have private repos not visible.

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
