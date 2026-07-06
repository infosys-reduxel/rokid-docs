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
  last_checked: 2026-07-06
  last_known_version: N/A — no longer serves CXR SDK content
  notes: |
    CONFIRMED 2026-07-06: ar.rokid.com/sdk and ar.rokid.com/sprite now BOTH render the
    full open.rokid.com-style AIUI/English developer homepage (Rokid Glasses / AR Lite / AR
    Studio hardware cards, "Docs" links pointing to open.rokid.com/sdk, YodaOS-Sprite card
    linking to open.rokid.com/sprite). Neither route surfaces CXR-L/CXR-M/CXR-S changelog
    content directly any more — this is a full transition, not a partial one as of the
    2026-06-30 check. ar.rokid.com root `map` now only returns Rokid Store app-catalog URLs
    (TV/Air/Station apps, out of scope hardware). Not useful as a CXR content source going
    forward; developerdoc.rokid.com remains the working iframe-serve layer. YodaOS-Master
    card present on both pages — skipped (out of scope), noted only.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-06
  last_known_version: CXR-L 1.0.4 (official changelog published 2026-06-29), CXR-M 1.1.0 (portal; Maven is 1.2.2, unchanged gap), 眼镜端裸机开发 0.0.1 (tab exists, content not re-verified this cycle — see note)
  notes: |
    REACHABLE AND VERIFIED 2026-07-06. /sdk now shows an OFFICIAL CXR-L v1.0.4 changelog
    (published 2026-06-29) — this supersedes the "no official changelog, binary-diff only"
    state noted in the prior cycle. New content in the official changelog not previously
    known: value range 0...15 for setGlassBrightness/setGlassVolume (local docs said "range
    undocumented"), a new "设备控制" (device control) doc chapter for Android+iOS, iOS
    RGCxrClient additions (setBrightness/getBrightness/setVolume/getVolume) and RGCxrDeviceInfo
    brightness/sound fields, iOS docs+sample version aligned to v1.0.4. /sprite content is
    byte-identical to local sprite-overview.md and to open.rokid.com/sprite (same iframe
    backend) — no drift. Map still returns only /sdk, /sprite, /sitemap.xml, root — the
    "眼镜端裸机开发" tab is present in the /sdk page's tab bar (text confirmed in scrape) but
    static/JS-action scraping could not activate it this cycle (click-by-selector failed);
    its content was not re-verified — treat 0.0.1/2026-03-01 pin as unconfirmed, not stale.
    YodaOS-Master tab skipped (out of scope) per repo rules.

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-06
  notes: |
    STILL BROKEN as of 2026-07-06 (re-verified live). Root path returns HTTP 415
    "Unsupported Media Type" / contentType application/octet-stream via Firecrawl scrape —
    same failure mode as the 2026-06-30 check. Not re-tested at the per-section workspace
    hash level this cycle (already confirmed dead 2026-06-28/30). DETECTION FAILURE for
    this source — do not treat as verified-absent; treat as unreachable.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-06
  notes: |
    Re-verified 2026-07-06: still redirects/serves the same open.rokid.com AIUI homepage
    content as ar.rokid.com (200, content matches ar.rokid.com scrape byte-for-byte).
    No distinct in-scope Sprite/CXR content surfaced. Low priority, no action.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-06
  last_known_version: |
    client-l 1.1.0 (NEW release; maven-metadata.xml lastUpdated 20260702091606 — release
    field moved from 1.0.4 to 1.1.0; client-l-1.1.0.aar uploaded 2026-07-02 10:18 UTC,
    size 1,286,574 bytes vs 70,543 bytes for 1.0.4 — an ~18x size jump, unusually large for
    a point release; portal changelog at developerdoc.rokid.com/sdk still tops out at 1.0.4
    as of 2026-07-06, no 1.1.0 changelog published yet)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged since last cycle)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Verified live via maven-metadata.xml (client-l, client-m, cxr-service-bridge) plus a
    directory listing of client-l/1.1.0/ (client-l-1.1.0.aar, .module, .pom + .sha1 files,
    all dated 2026-07-02/07-04). This is a fresh, Maven-only lead: client-l has jumped a
    full minor version (1.0.4 -> 1.1.0) since the 2026-06-30 registry snapshot, and the
    AAR size jump is large enough to suggest more than an incremental API addition — flag
    for investigation once/if an official changelog appears. client-m and cxr-service-bridge
    both unchanged. Use Maven as the canonical "what's actually shipped" source.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-06
  notes: Re-verified 2026-07-06. Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, issue-tracker traffic only). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-06
  notes: Re-verified 2026-07-06. Same repo set as prior cycle (glass-docs last commit 2020-07-13, old Glass 1/2 era; UXR-docs out of scope; mostly Speech/OpenVoice/CloudApp/skill-platform repos). No in-scope Sprite/AR Glasses content found. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-06
  notes: Re-verified 2026-07-06 — map again returned an empty links array (0 URLs), consistent with the 2026-06-29/30 checks. No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Re-checked 2026-07-06 (still unregistered — this note is a status report only, not a
    registry entry). open.rokid.com/sprite?lang=zh (JS-rendered, --wait-for 4000ms) is
    byte-for-byte identical to developerdoc.rokid.com/sprite — same FAQ, same hardware spec
    table, same CDN asset paths (ar.rokidcdn.com/developerDoc/...). This confirms
    open.rokid.com/sprite is a front-end wrapper over the SAME developerdoc.rokid.com
    iframe backend, not an independent content source — everything reachable there is
    already covered by the registered developerdoc.rokid.com entry. open.rokid.com/sdk
    still returns only an SPA shell on static/JS-wait scrape (no content). /academy
    ("乐奇学院" Rokid Academy) surfaced a "CXR-L SDK开发课程" (Glasses mobile-app dev intro)
    and a "Glasses 开发课程" course card — in-scope but course-marketing content only, no
    new API/version facts beyond what developerdoc.rokid.com already documents. UXR 3.0 and
    AIUI courses also listed on the same page — out of scope, skipped, not translated.
    YodaOS-Master content at open.rokid.com/master — skipped (out of scope).
    No change to prior recommendation: still suggest open.rokid.com as a potential
    registry addition (front door to the same content ar.rokid.com used to serve), but
    since ar.rokid.com itself no longer serves CXR content at all, developerdoc.rokid.com
    is now the only registered source actually returning live CXR-L/CXR-S/CXR-M content —
    worth flagging to the user that the registry's primary "developer-portal" source for
    CXR content is effectively developerdoc.rokid.com alone.
