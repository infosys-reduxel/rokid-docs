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
  last_checked: 2026-08-31
  last_known_version: CXR-L 1.0.4 (portal changelog still shows 1.0.4 as of 2026-08-31; Maven has 1.1.2)
  notes: |
    React SPA. `firecrawl map` on the root now returns 0 URLs (SPA shell only, consistent with
    the 2026-06-28 finding that real content lives at developerdoc.rokid.com / open.rokid.com).
    ar.rokid.com/sprite still resolves but content is now at open.rokid.com/sprite.
    YodaOS-Master tab remains; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-31
  last_known_version: CXR-L 1.0.4 (2026-06-25, per /sdk changelog), CXR-M 1.1.0 (portal lags Maven 1.2.2), 眼镜端裸机开发 1.0.0 (2026-06-05)
  notes: |
    /sdk re-scraped 2026-08-31 — still shows CXR-L latest version "1.0.4" and CXR-M "1.1.0";
    matches Maven client-l release for 1.0.4 (portal has not caught up to 1.1.0/1.1.1/1.1.2 —
    see maven.rokid.com entry). /sprite FAQ section has CHANGED since the 2026-06-08 fetch:
    it reverted from the detailed CXR-L/bare-metal Q&A set to a short 4-question CXR-M/CXR-S
    FAQ (confirmed via direct scrape, also re-checked with explicit ?lang=zh — identical).
    Device specification table on /sprite is unchanged. yodaos/docs/sprite-overview.md FAQ
    section updated this cycle to match. Map now returns 3 URLs (/sprite, /sprite?lang=zh,
    /sitemap.xml) — /sdk and root no longer appear in the map despite still resolving directly.
    YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-31
  notes: |
    Still BROKEN as of 2026-08-31 (re-verified: scrape fails on all Firecrawl engines,
    "All scraping engines failed to retrieve content from this URL"). Consistent with the
    2026-06-28 finding of OSS NoSuchKey errors on both workspace hashes
    (57e35cd3ae294d16b1b8fc8dcbb1b7c7, 84feb39f8ef141b0ad0326f902ab881f). Source remains
    not reachable; not a monitor candidate.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-30
  notes: As of 2026-06-29, developer.rokid.com redirects to open.rokid.com (confirmed identical content to ar.rokid.com redirect). Legacy Speech/HomeBase GitBook content may still be at developer.rokid.com/docs/rokid-homebase-docs/v2/. No in-scope Sprite/AR Glasses content surfaced. Low priority. (Not re-checked 2026-08-31 — low priority, no change expected.)

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-31
  last_known_version: |
    client-l 1.1.2 (release; metadata lastUpdated 20260828083628 — new releases 1.1.0 (2026-07-02),
    1.1.1 (2026-08-14), 1.1.2 (2026-08-28) found 2026-08-31; portal still shows 1.0.4, official
    changelog not yet published. 1.1.0 appears to be a withdrawn/anomalous build — see
    cxr-l/release-notes.md. Documented this cycle via binary diff.)
    client-m 1.2.2 (release; metadata lastUpdated 20260826091529 — version number unchanged
    since 2026-06-09, but AAR re-uploaded 2026-08-26; downloaded and verified byte-for-byte
    identical to the previously-documented 1.2.2 AAR (sha256 c75c5d3f...; size 1,225,276 bytes
    matches). No content change — likely a Nexus re-index/republish. No action needed.)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — version unchanged
    since first observed 2026-05-22, metadata timestamp advanced; not independently verified
    this cycle since no client depends on a newer cxr-service-bridge release).
  notes: |
    Public Maven for CXR SDK JARs/AARs. Direct browse path is
    https://maven.rokid.com/service/rest/repository/browse/maven-public/com/rokid/cxr/
    (the /repository/ path returns "not browseable" page; use /service/rest/repository/browse/).
    maven-metadata.xml under each artifact gives <release>, <latest>, <lastUpdated>.
    client-l: 1.1.2 is now the release version (verified 2026-08-31). Portal still shows 1.0.4
    changelog. Official changelog pending for the whole 1.1.x line.
    client-m: 1.2.2 unchanged since 2026-06-09 (content-verified via AAR hash this cycle).
    Portal still on 1.1.0.
    cxr-service-bridge: 1.0 unchanged.
    Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-31
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items. Re-verified 2026-08-31, no change.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-31
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-29 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Low priority. Re-verified 2026-08-31, no change.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-31
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) again on 2026-08-31 check. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible.

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
