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
  last_checked: 2026-06-16
  last_known_version: CXR-L 1.0.3 (2026-06-02 per SDK landing page)
  notes: React SPA. Iframe-wraps developerdoc.rokid.com/{sdk,sprite}?lang=zh — that's where actual content renders. /sdk landing (scraped 2026-06-11) shows CXR-L 1.0.3 updated 2026-06-02; map returned ar.rokid.com/master (out-of-scope YodaOS-Master) and ar.rokid.com/sprite (in-scope). ar.rokid.com/sprite and ar.rokid.com/sdk both return only SPA shell ("Rokid AR Platform") to static fetchers — no content extracted 2026-06-16. Content verified at developerdoc.rokid.com/sdk and developerdoc.rokid.com/sprite instead.

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-06-16
  last_known_version: CXR-L 1.0.3 (2026-06-02), CXR-M 1.1.0 (portal lags Maven; Maven has 1.2.2), 眼镜端裸机开发 0.0.1 (2026-03-01)
  notes: |
    /sdk page scraped 2026-06-16: byte-for-byte identical to 2026-06-11 scrape. CXR-L 1.0.3 changelog (7 items) confirmed. CXR-M portal still shows 1.1.0; portal doc inaccessible via custom.rokid.com (NoSuchKey). /sprite page scraped 2026-06-16: byte-for-byte identical to 2026-06-11 scrape (105 lines, 8 Q&A items). YodaOS-Master tab skipped (out of scope). Notable gap still open: official v1.0.3 CXR-L changelog (items 1-7, including CustomView JSON Schema chapter and merged CXR-S chapters) has NOT been translated into local cxr-l/release-notes.md on main branch — the 2026-06-11 translation work remains in an unmerged PR (#16).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-16
  notes: |
    Hosts actual SDK docs as JS-rendered SPA on Alibaba OSS. As of 2026-06-16, individual page URLs for both workspaces return "NoSuchKey" from Alibaba OSS when Firecrawl fetches the path directly — suggesting workspace content has been rebuilt/rehashed. Static WebFetch hits the SPA shell only ("Rokid AR Platform"). Pages remain inaccessible to static fetchers. Known workspace hashes:
    - 84feb39f8ef141b0ad0326f902ab881f = CXR-L (page IDs now NoSuchKey as of 2026-06-16)
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 = CXR-M / CXR-S / 眼镜端裸机开发 (page IDs now NoSuchKey as of 2026-06-16)
    Content can only be read via human browser. Best monitoring approach: register Firecrawl monitor on developerdoc.rokid.com/sdk (the iframe entry point) rather than custom.rokid.com directly.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Scrape 2026-06-11 returns only SVG/logo (heavily JS-rendered). Map returned only 1 URL. Real surface is developer.rokid.com/docs/rokid-homebase-docs/v2/. Low coverage overlap with this repo's AR-focused scope; keep for periodic checks via monitor. Not re-scraped 2026-06-16 due to low priority and credit constraints (24 credits remaining).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-16
  last_known_version: |
    client-l 1.0.3 (release; metadata lastUpdated 20260611075349 — unchanged from 2026-06-11 check)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged from 2026-06-11 check)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Public Maven for CXR SDK JARs/AARs. Checked via Firecrawl scrape of maven-metadata.xml for each artifact 2026-06-16. All three artifacts unchanged from 2026-06-11 check: client-l 1.0.3 (lastUpdated 20260611075349), client-m 1.2.2 (lastUpdated 20260608030211), cxr-service-bridge 1.0 (lastUpdated 20260522063622). client-m 1.2.2 continues to lead the portal docs (portal still shows 1.1.0 as latest documented). No new artifacts released this cycle.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos as of 2026-06-11 check: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. Not re-checked 2026-06-16 due to credit constraints.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-06-11
  notes: Verified org (id 19773259). Official "Rokid" org. Checked 2026-06-11 — relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Not re-checked 2026-06-16 due to credit constraints.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-06-11
  notes: Verified org (id 25831739). Only 2 public repos as of 2026-06-11 check: BroadcastServiceDemo (last commit 2017-06-19 — a 9-year-old OpenCV sample with no Rokid Glasses relevance) and repo listing showed no new repos. No actionable content for this repo. May have private repos not visible. Not re-checked 2026-06-16 due to credit constraints.
