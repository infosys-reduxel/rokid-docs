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
  last_checked: 2026-08-09
  last_known_version: CXR-L 1.0.4 portal-displayed / 1.1.0 actual on Maven (see maven entry below)
  notes: |
    React SPA. As of 2026-06-28 the /sdk route now renders the open.rokid.com developer
    homepage (AIUI/AI Agent-focused) rather than the CXR SDK changelog it previously showed.
    Real Sprite/CXR doc surfaces have migrated to open.rokid.com (see new-sources note below).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope). Unchanged as of 2026-08-09 check.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-09
  last_known_version: CXR-L 1.0.4 (portal, unchanged since 2026-06-25), CXR-M 1.1.0 (portal lags Maven 1.2.2, unchanged), 眼镜端裸机开发 1.0.0 (2026-06-05 — CHANGED from 0.0.1, see below)
  notes: |
    SDK landing page re-scraped 2026-08-09. /sdk table now shows: CXR-L 最新版本 1.0.4 (still
    lagging Maven's 1.1.0 — see maven entry, P1 actioned this cycle), CXR-M 1.1.0 (lags Maven
    1.2.2, unchanged), 眼镜端裸机开发 (bare-metal) 1.0.0 "更新于 2026.06.05" — this is a real
    version bump from the 0.0.1 (2026-03-01) baseline cxr-baremetal/development-guide.md was
    translated from (P1 flagged this cycle). No detail-page route found for the bare-metal doc
    (custom.rokid.com, which hosted it, remains broken — see below); development-guide.md was
    annotated with a "known stale" note rather than rewritten, since no reachable source has the
    1.0.0 content. /sprite FAQ content matches local sprite-overview.md exactly except the
    client-l version pin (updated this cycle to 1.1.0). Map returns 3 URLs: /sdk, /sprite, root
    (no /sitemap.xml this time). YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-09
  notes: |
    Still BROKEN as of 2026-08-09 (re-verified; same failure mode as 2026-06-28 — Firecrawl
    "All scraping engines failed to retrieve content from this URL"). Both previously valid
    workspace hashes returned OSS NoSuchKey as of 2026-06-28:
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M / CXR-S / 眼镜端裸机开发): NoSuchKey 6A408F766EB57F3436AE9E82
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L): NoSuchKey 6A408F1CC38F5534389093AE
    The OSS bucket (rokid-ar-platform.oss-cn-hangzhou.aliyuncs.com) no longer contains these
    workspace keys. Detail docs appear to have been migrated. New workspace hashes unknown.
    This is why the bare-metal 1.0.0 doc-version bump (see developerdoc.rokid.com entry) could
    not be actioned beyond a stale-version flag — no reachable source has the actual content.
    Monitor candidate removed — source is not reachable.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-09
  notes: developer.rokid.com redirects to open.rokid.com (unchanged since 2026-06-29 check). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority; not re-crawled in depth this cycle.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-09
  last_known_version: |
    client-l 1.1.0 (release; metadata lastUpdated 20260718072455; artifact file timestamps read
    2026-07-02; CHANGED from 1.0.4 — actioned as P1 this cycle, see cxr-l/release-notes.md)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — release version
    string unchanged at "1.0", but lastUpdated moved from 20260522063622 to 20260728074326;
    likely a rebuild/republish of the same 1.0 release, not a new version — not actioned)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: 1.1.0 confirmed as a genuine new release (verified 2026-08-09) — binary diff of
    the 1.0.4 vs 1.1.0 AAR shows a major additive change: AAR grew from 70,543 to 1,286,574
    bytes; classes.jar grew from 66 to 160 classes (no removals); a new `com.rokid.cxr.session`
    Kotlin/coroutines package was added (CxrSession/CxrSessionManager — a new higher-level
    session API, additive alongside the existing CXRLink API); the previously-separate
    `cxr-service-bridge` classes (Caps, CXRServiceBridge, CXRSocketProtocol) are now bundled
    directly into client-l with native .so backing instead of being a POM dependency; a
    proguard.txt consumer-rules file was added, headed "CXR-L SDK v1.1.0 — 消费者混淆规则",
    confirming both the version number and that com.rokid.cxr.session is the intended public
    API. Full writeup and provenance in cxr-l/release-notes.md and cxr-l/api-reference.md.
    Portal (developerdoc.rokid.com/sdk) still shows 1.0.4 as of 2026-08-09 — no official
    changelog published for 1.1.0. Flagged for human confirmation: whether bundling the
    service-bridge classes into client-l signals a deeper CXR-M/CXR-L/CXR-S transport
    convergence, or is just an implementation-sharing detail — not resolvable from a binary
    diff alone.
    client-m: 1.2.2 unchanged since 2026-06-09. Portal still on 1.1.0.
    cxr-service-bridge: release version string still "1.0"; lastUpdated metadata timestamp
    moved (2026-05-22 → 2026-07-28) but the <versions> list is unchanged (1.0-SNAPSHOT, 1.0) —
    treated as a rebuild/republish, not a new version. Not actioned.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-09
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items. Unchanged since 2026-06-30.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-09
  notes: Verified org (id 19773259). Official "Rokid" org. Re-checked 2026-08-09 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority. Unchanged since 2026-06-30.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-09
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) again on 2026-08-09 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible. Unchanged since 2026-06-30.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Discovered 2026-06-28; still unregistered as of 2026-08-09 (no interactive user available
    this cycle — unattended run — so not auto-approved, per registry rules). ar.rokid.com/sdk
    now renders a new AIUI-focused homepage that links to open.rokid.com/sdk and
    open.rokid.com/sprite as the canonical developer portal. open.rokid.com/sprite?lang=zh
    re-scraped 2026-08-09 — content still matches developerdoc.rokid.com/sprite exactly (same
    hardware spec table, same FAQ). open.rokid.com/sdk?lang=zh returns only SPA shell (no
    per-SDK changelog content client-side-rendered in a Firecrawl-visible way).
    /academy hosts the "乐奇学院" (Rokid Academy) learning-platform / course-catalog page —
    checked 2026-08-09, lists "Glasses 开发课程" and "CXR-L SDK开发课程" (in scope) alongside
    UXR 3.0 and AIUI courses (out of scope, skipped); all course content links out to
    t.rokid.com short-links to an external video platform, not inline documentation — not
    translatable doc content, informational only. YodaOS-Master content linked from
    open.rokid.com/master — skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com.
