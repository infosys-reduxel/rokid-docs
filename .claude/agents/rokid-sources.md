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
  last_checked: 2026-09-10
  last_known_version: CXR-L 1.1.2 (per linked developerdoc.rokid.com/sdk, updated 2026-09-08)
  notes: |
    Re-verified 2026-09-10 (firecrawl map + scrape, HTTP 200). Still fully pivoted to the
    AIUI/AI Agent homepage; map returns only 1 URL (the arStore detail page). /sdk and /sprite
    both render the same AIUI homepage, not CXR content. Discovered new outbound links on /sdk:
    open.rokid.com/sprite, open.rokid.com/master, and a NEW custom.rokid.com workspace hash
    c88be4bcde4c42c0b8b53409e1fa1701 (scraped — also returns OSS NoSuchKey, i.e. dead link into
    the same broken CMS bucket as the old hashes). YodaOS-Master tab remains; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-10
  last_known_version: CXR-L 1.1.2 (public, updated 2026.09.08), CXR-M 1.1.0 (business-gated, unchanged), 眼镜端裸机开发/bare-metal 1.0.0 (public, updated 2026.06.05)
  notes: |
    Re-verified 2026-09-10 (firecrawl map + scrape, HTTP 200). Map still returns 3 URLs (/, /sdk,
    /sprite?lang=zh). /sdk now shows a "SDK 选型速览" comparison table: CXR-L public 1.1.2
    (🟢 active), CXR-M business-only 1.1.0 (🔒), 裸机开发/bare-metal public 1.0.0 (🟢). This is a
    JUMP from the last-checked CXR-L 1.0.3/1.0.4 baseline — versions 1.1.0, 1.1.1, 1.1.2 are all
    now live and undocumented locally (see P1 in latest scout report). Bare-metal doc also now
    explicitly versioned 1.0.0 / updated 2026-06-05, vs local cxr-baremetal/ pinned at v0.0.1
    (2026-03-01). /sprite FAQ + hardware spec table still matches local content verbatim — no
    drift there. "查看文档" (View Documentation) buttons are JS-routed, no static href captured;
    could not confirm exact target URL for CXR-L/bare-metal detail pages from this domain alone —
    combine with custom.rokid.com (broken) or the new x-docs.rokid.com finding. YodaOS-Master tab
    skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-10
  notes: |
    STILL BROKEN as of 2026-09-10. Root path: "All scraping engines failed to retrieve content"
    (index, fire-engine chrome-cdp, fire-engine retry chrome-cdp all failed — non-2xx/unreachable).
    New workspace hash c88be4bcde4c42c0b8b53409e1fa1701 (found linked from ar.rokid.com/sdk
    2026-09-10) also returns OSS NoSuchKey (RequestId 6AA21FA1EEC74232368A05DF) from the same
    rokid-ar-platform.oss-cn-hangzhou.aliyuncs.com bucket. Source remains unreachable / not a
    valid detection channel. Treat any finding sourced only from this domain as unverifiable.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-10
  notes: |
    Re-verified 2026-09-10 (firecrawl scrape, HTTP 200). Content still matches the AIUI-focused
    open.rokid.com homepage (contains "AIUI", "Master", "Sprite" nav references identical to
    ar.rokid.com). No in-scope Sprite/AR Glasses content surfaced beyond what's already covered
    via developerdoc.rokid.com / open.rokid.com. Low priority, no action.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-10
  last_known_version: |
    client-l: release 1.1.2, latest 1.2.X-SNAPSHOT, lastUpdated 20260828083628 (2026-08-28).
      Full version history now includes 1.0.4 -> 1.1.0 -> 1.1.1 -> 1.1.2 -> 1.2.x (in dev).
    client-m: release 1.2.2, latest 1.2.2, lastUpdated 20260902061433 (2026-09-02, metadata
      touched but version unchanged since last check).
    cxr-service-bridge: release 1.0, lastUpdated 20260728074326 (2026-07-28, version unchanged).
    NEW (not part of com.rokid.cxr groupId, not yet in scope pending Leader/user triage):
    com.rokid.security:glass3.open.sdk — release 2.5.1-P, latest 2.6.5-P-SNAPSHOT,
      lastUpdated 20260909072258 (2026-09-09, i.e. yesterday relative to this check).
    com.rokid.security:phone.sdk — release 2.5.1-P, latest 2.6.4-P-SNAPSHOT,
      lastUpdated 20260909140033 (2026-09-09).
  notes: |
    Re-verified 2026-09-10 via direct maven-metadata.xml scrapes (all HTTP 200/valid XML).
    client-l is 2 full releases ahead of the local docs (1.0.4 documented -> Maven/portal now
    at 1.1.2). client-m/cxr-service-bridge version-stable, only lastUpdated timestamps moved.
    MAJOR FINDING: discovered an entirely new artifact family under groupId com.rokid.security
    (glass3.open.sdk = glasses-side, phone.sdk = phone-side) that is under near-daily active
    development (-E and -P release trains, dozens of point releases since ~2.1.0). This appears
    to correspond to the new "Glass3 SDK" documented at x-docs.rokid.com (see New sources
    discovered) and targets the same "Rokid RG-glasses" hardware this repo already documents.
    Not added to `covers:` for this registry entry pending explicit user decision on whether/how
    to bring com.rokid.security into scope alongside com.rokid.cxr.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-09-10
  notes: |
    Re-verified 2026-09-10 (firecrawl map, HTTP 200, org id 57519491 unchanged). Still only 2
    public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope
    Glass 2 hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-09-10
  notes: |
    Re-verified 2026-09-10 (firecrawl map, HTTP 200, org id 19773259 unchanged). Map surfaced
    additional legacy repos not previously logged: mobile-sdk-android-docs, mobile-sdk-ios-docs
    (last commit ~2019 per scrape), native-system-docs (last commit 2018). All are pre-Sprite-era
    smart-speaker/robot SDK docs (same vintage as RokidVoiceAIDemo, rokid-openvoice-sdk,
    CloudAppClient in this org) — out of scope, not Rokid Glasses/Sprite hardware. glass-docs and
    UXR-docs remain out of scope as previously noted. No in-scope content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-10
  notes: |
    Re-verified 2026-09-10 (org id 25831739 unchanged). `map` again returned an empty links
    array — known limitation of this org's GitHub listing for the map tool. A direct `scrape`
    of the org page (HTTP 200) surfaced 4 repos: BroadcastServiceDemo, openCV3, openCV3_demo,
    people — all last updated 2017 (Jun 19, 2017 / Feb 17, 2017 per scraped dates). No in-scope
    content. May have private repos not visible to this key.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Re-confirmed reachable 2026-09-10 (map returns 8 URLs: root, /?lang=cn, /sdk?lang=en,
    /sprite?lang=en, /master?lang=en, /academy, and 2 /detail?appId=... app-store pages).
    open.rokid.com/sprite?lang=en content still matches developerdoc.rokid.com/sprite verbatim
    (same FAQ, same RG-glasses hardware spec table: Qualcomm AR1, 2GB RAM, 32GB ROM, etc.).
    open.rokid.com/sprite now explicitly labels CXR-M as "CXR-M SDK (Business Cooperation)" —
    contact Glasses.BD@rokid.com, not publicly downloadable — matching developerdoc's gating.
    YodaOS-Master content still linked from open.rokid.com/master — skipped (out of scope).
    Still suggest registering open.rokid.com as a canonical read surface alongside
    developerdoc.rokid.com (both serve equivalent Sprite content; neither exposes real CXR-L/
    bare-metal detail-page URLs via static links — those are JS-routed to custom.rokid.com
    (broken) or possibly x-docs.rokid.com, see below).

- url: https://x-docs.rokid.com
  kind: developer-portal
  covers: cxr-m (candidate), cxr-s (candidate), hardware
  status: UNREGISTERED — requires explicit user approval before adding to scope
  notes: |
    NEWLY DISCOVERED 2026-09-10 via firecrawl map/scrape (HTTP 200 throughout, ~45 URLs mapped
    under /docs). This is a full VitePress documentation site titled "Rokid Sprite Enterprise
    SDK" / "Rokid Glass3 SDK", structured as a Phone SDK + a Glasses SDK, with getting-started,
    API reference, code samples, FAQ, and demo-guide sections in both zh and en.
    Evidence it targets the SAME hardware this repo already documents (not a new product line):
    the Quick Start doc states "Android Studio should show a device such as `Rokid RG-glasses`"
    — RG-glasses is the exact Model value in this repo's README.md Device Specifications table.
    "Glass3" appears to be Rokid's internal/marketing codename for this generation of Rokid
    Glasses, NOT the out-of-scope "Glass 2" (Master/ER family) — needs explicit confirmation but
    strongly implied by the RG-glasses device match and the Qualcomm-AR1-class hardware context.
    The SDK itself is a different artifact family entirely: Maven groupId com.rokid.security,
    artifacts glass3.open.sdk (glasses-side) and phone.sdk (phone-side), both real and under
    active near-daily development (see maven.rokid.com entry above) — architecturally distinct
    from com.rokid.cxr's CxrApi/CXRLink/cxr-service-bridge model documented in cxr-m/cxr-s/cxr-l.
    Sample API surface (IMobileEngine, EngineParam, AK/SK auth via Glasses.BD@rokid.com,
    NetServiceType cloud services) shares no method/class names with the current CXR-M docs.
    Full page inventory captured in .firecrawl/x-docs-map.json (45 URLs) including:
    terminal-sdk/getting-started, terminal-sdk/api-reference (separate phone-side and
    glasses-side API doc pages), terminal-sdk/glasses, terminal-sdk/phone, terminal-sdk/capabilities
    (UI design spec), downloads/{demo-guide,samples,apps}, faq/{bluetooth,p2p troubleshooting},
    openapi/{device-management,remote-collaboration,AI-large-model-onboarding,ApiKey}, and a large
    code-samples tree (device-connection, message-transfer, media, vision, voice-ai, system) under
    both zh (代码示例/) and en (en/代码示例/) paths.
    This is the single largest open item from this scout run: either (a) Rokid has begun
    migrating/rebranding CXR-M+CXR-S into a unified "Glass3 SDK" at x-docs.rokid.com, or (b) this
    is a parallel enterprise-only SDK track sold alongside the public CXR-L SDK. Recommend Leader/
    user decide whether to bring x-docs.rokid.com into scope (would likely need new top-level
    coverage, not just edits to existing cxr-m/cxr-s files) before any translation work proceeds
    on it. Evidence files: .firecrawl/x-docs.rokid.com_docs_en_.md,
    .firecrawl/x-docs.rokid.com_docs_en_terminal-sdk_getting-started__E5_BF_AB_E9_80_9F_E5_BC_80_E5_A7_8B.html.md,
    .firecrawl/x-docs.rokid.com_docs_en_terminal-sdk_glasses.md, .firecrawl/x-docs-phone-api.md.
