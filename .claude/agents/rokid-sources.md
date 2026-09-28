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
  last_checked: 2026-09-28
  last_known_version: CXR-L 1.1.2 (portal-adjacent developerdoc.rokid.com/sdk confirmed; Maven matches at 1.1.2)
  notes: |
    Not re-crawled directly this cycle (developerdoc.rokid.com is the confirmed real render
    target — see below). Previously-noted redirect to open.rokid.com's AIUI-focused homepage
    unchanged as of last direct check (2026-06-28). YodaOS-Master tab remains; skipped
    (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-09-28
  last_known_version: CXR-L 1.1.2 (portal "最新版本 1.1.2", updated 2026.09.08; matches Maven release), CXR-M 1.1.0 (portal card unchanged; Maven has 1.2.2 — same gap as previous cycles), 眼镜端裸机开发 1.0.0 (portal card, updated 2026.06.05)
  notes: |
    Re-scraped 2026-09-28 via Firecrawl (`/sdk`). Card now shows CXR-L at "最新版本 1.1.2" with a
    collapsed "▶ 更新内容" (changelog) accordion per SDK card. Firecrawl scrape/actions/query
    attempts on this pass could not surface the accordion's actual content (query mode returned a
    clearly wrong/hallucinated answer — "商务对接" — for a public, actively-maintained SDK; discarded).
    So there is still no scraped official changelog text for CXR-L 1.1.0/1.1.1/1.1.2 — those were
    documented from Maven AAR binary diffs instead (see cxr-l/release-notes.md).
    `firecrawl map` still returns only 3 URLs (/sdk, /sprite?lang=zh, root) — no separate
    changelog page discovered. The page's link list (via `-f links` scrape) now includes
    `https://x-docs.rokid.com/docs/` — a previously-unregistered documentation site branded
    "Rokid Sprite Enterprise" with terminal-sdk (glasses/phone), openapi, and downloads sections
    referencing a "glass3.open.sdk" artifact and "Rokid Glass3" hardware. This looks like a
    genuinely new SDK family/hardware line, not a CXR-M/S/L variant — flagged under "New sources
    discovered" below, NOT treated as actionable/in-scope this cycle pending explicit sign-off.
    YodaOS-Master tab skipped (out of scope), as in prior cycles.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-28
  notes: |
    STILL BROKEN as of 2026-09-28 (re-checked). The CXR-L workspace hash
    (84feb39f8ef141b0ad0326f902ab881f) still returns OSS NoSuchKey (RequestId
    6AB9DA3A3748B73632D03F72 on this check, differs from prior RequestIds as expected for a
    per-request S3-style error, but the Key and Code are identical). No new workspace hashes
    discovered. Root path not re-tested this cycle. Monitor candidate remains removed —
    source is not reachable.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-09-28
  last_known_version: |
    client-l 1.1.2 (release; AAR last-modified 2026-08-28. 1.1.0 uploaded 2026-07-02, 1.1.1 uploaded 2026-08-14. Big jump from previously-known 1.0.4 — three releases landed since last check. No official changelog published; documented via binary diff + javap in cxr-l/release-notes.md)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged since last check)
    cxr-service-bridge 1.4 (release; AAR last-modified 2026-09-22. 1.1 uploaded 2026-09-17, 1.2 uploaded 2026-09-17, 1.3 uploaded 2026-09-18. Jump from previously-known 1.0 — four releases landed. Documented via binary diff + javap in cxr-s/release-notes.md, new file this cycle)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: jumped 1.0.4 -> 1.1.2 (three releases). v1.1.0 introduced a new
    com.rokid.cxr.session.* package (CxrSession/CxrSessionManager) alongside the existing
    CXRLink hierarchy — verified via javap, not fabricated. Portal (developerdoc.rokid.com/sdk)
    now shows "最新版本 1.1.2" matching Maven; still no scrapeable official changelog text.
    client-m: 1.2.2 unchanged since 2026-06-09 — no action this cycle.
    cxr-service-bridge: jumped 1.0 -> 1.4 (four releases). v1.1 added local audio-record
    streaming (new ICXRService AIDL interface, AudioRecordHelper) and a breaking constructor
    signature change (now requires Context). v1.3 added cancelMessage(int). v1.4 changed
    openAudioRecord's signature again. Portal has never published a cxr-service-bridge
    changelog in any cycle to date; documented via binary diff only.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-29 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-30
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) on 2026-06-29 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://x-docs.rokid.com/docs/
  kind: developer-portal
  covers: (unclear — see notes; likely a new SDK family, not cxr-m/s/l)
  status: UNREGISTERED — requires user approval; possible NEW TOP-LEVEL SECTION, do not action without sign-off
  notes: |
    Discovered 2026-09-28 via the link list (`-f links` scrape) of developerdoc.rokid.com/sdk.
    Branded "Rokid Sprite Enterprise" ("Rokid AI 企业版"). `firecrawl map` (limit 200) returned
    20+ URLs including /docs/terminal-sdk (glasses + phone SDK docs), /docs/openapi (a REST
    "user management" API with AK/SK auth), /docs/downloads (demo apps, requires enterprise
    login), and /docs/skills/version.json which reports
    `"sdk": {"glasses": "com.rokid.security:glass3.open.sdk:2.2.0-E ..."}` and
    `"changelogVersion": "V2.2.0-E (2026-8-6)"`.
    Artifact group `com.rokid.security` and product name "Rokid Glass3" do not match any
    artifact/hardware in this registry (client-m/client-l/cxr-service-bridge under
    com.rokid.cxr; RV101/RV102/RV203 hardware). This looks like a distinct enterprise/B2B SDK
    and possibly a distinct hardware revision ("Glass3"), not a documented CXR-M/S/L release.
    Per CLAUDE.md / CONTRIBUTING.md, a wholly new top-level SDK family requires explicit human
    sign-off before any translation or repo structure changes — NOT actioned this cycle.
    No content from this source was scraped beyond the map/version.json metadata above; no
    page content was translated or committed.

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
