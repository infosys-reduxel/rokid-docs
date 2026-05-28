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
  last_checked: 2026-05-28
  last_known_version: CXR-L 1.0.1 (2026-05-07)
  notes: React SPA. Map yields ~44 URLs (mostly /detail?appId=... AR Store entries). Doc surfaces are /sdk, /sprite, /master. /sprite scraped empty — needs firecrawl-interact for SPA hydration. /master describes YodaOS-Master (Station 2 / Station Pro), UXR3.0 + JSAR SDKs — uncovered locally.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-05-28
  notes: Legacy Rokid developer portal (mostly Speech/HomeBase GitBook). Map returned only 1 URL — site is heavily JS-rendered and /sitemap.xml returns 301. Real surface is developer.rokid.com/docs/rokid-homebase-docs/v2/. Low coverage overlap with this repo's AR-focused scope; keep for periodic checks via monitor.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-05-28
  notes: Public Maven for CXR SDK JARs/AARs. Map returned 0 URLs (Maven exposes directory listings, not a link-graph). For change detection, scrape specific groupId paths (e.g. /com/rokid/cxr/client-l/, /client-m/, /cxr-service-bridge/) and diff against local docs' pinned versions. Local cxr-l references v0.0.1; upstream /sdk advertises v1.0.1.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-05-28
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Blog → https://rokidglass.github.io/glass2-docs/. Likely older Glass 2 docs — overlap with current AR Glasses TBD.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-05-28
  notes: Verified org (id 19773259). Official "Rokid" org, located ZheJiang, blog points to developer.rokid.com. Mostly Speech/OpenVoice repos — limited overlap with AR Glasses scope.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-05-28
  notes: Verified org (id 25831739). Only 2 public repos — most likely the AR Glasses sample-app / SDK demo home. High signal-to-noise for this repo.
