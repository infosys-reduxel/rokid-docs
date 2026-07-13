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
  last_checked: 2026-07-13
  last_known_version: CXR-L 1.0.4 shown in official portal changelog at open.rokid.com/sdk (Maven now has 1.1.0, released 2026-07-02 — see maven.rokid.com entry)
  notes: |
    Re-verified 2026-07-13. ar.rokid.com/map now returns almost entirely Rokid Store app-catalog
    URLs (Rokid Station/Air apps — out-of-scope hardware) with no /sdk or /sprite links surfaced
    by map. Direct scrape of both https://ar.rokid.com/sdk and https://ar.rokid.com/sprite
    confirms both still resolve and both render the same "AIUI: The Next Frontier" developer
    homepage (rokid-developer-homepage bundle v1.1.0) that links out to open.rokid.com/sdk and
    open.rokid.com/sprite as the real doc targets — same behavior already noted 2026-06-28, not
    a new change. YodaOS-Master card present on this homepage; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-07-13
  last_known_version: CXR-L 1.0.4 (portal at open.rokid.com; developerdoc.rokid.com/sdk itself now returns SPA-shell only — see notes), CXR-M 1.1.0 (portal lags Maven 1.2.2, unchanged), 眼镜端裸机开发 0.0.1 (not re-verified this cycle)
  notes: |
    Re-verified 2026-07-13. Map now returns only 3 URLs (root, /sdk, /sprite) — /sitemap.xml no
    longer listed (was 4 URLs on 2026-06-30). --only-main-content scrape of /sdk now returns only
    nav/footer chrome (SPA shell, no rendered changelog) — content that used to render here has
    effectively moved to open.rokid.com/sdk, which DOES now serve full rendered changelog markdown
    (see open.rokid.com entry below). /sprite content still renders fully and matches local
    yodaos/docs/sprite-overview.md structurally (About text, spec table, Developer Toolkit
    sections, FAQ) — no drift detected on this page. YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-13
  notes: |
    STILL BROKEN as of 2026-07-13, unchanged from 2026-06-30 finding. Root path returns HTTP 415
    Unsupported Media Type (contentType application/octet-stream). Re-tested with the CXR-L
    workspace hash as a query param (?workspace=84feb39f8ef141b0ad0326f902ab881f) — same HTTP 415.
    Source remains unreachable via Firecrawl scrape; no content retrieved this cycle either.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-13
  notes: |
    ANOMALY 2026-07-13: previous check (2026-06-30) recorded this domain as redirecting to
    open.rokid.com with identical content. This cycle, both firecrawl map and a direct scrape of
    the root URL returned the OLD GitBook-style content directly (rokidos-linux-docs,
    rokid-homebase-docs, skill-kit docs, AVS 方案) with NO redirect to open.rokid.com observed.
    This content is entirely legacy Rokid smart-speaker / voice-skill material (RokidOS-Linux,
    HomeBase, AVS) — not YodaOS-Sprite, not Rokid Glasses, and not one of the CLAUDE.md-listed
    out-of-scope families either; it is simply a different, older, unrelated product line and is
    treated as out of repo scope. No in-scope Sprite/AR Glasses content found. Flagging the
    redirect-vs-no-redirect flip for awareness only; no action needed.

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-13
  last_known_version: |
    client-l 1.1.0 (release; metadata lastUpdated 20260702091606 — NEW release since last check;
      previous known release was 1.0.4 lastUpdated 20260625070819)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260522063622 — unchanged)
  notes: |
    Re-verified 2026-07-13 via WebFetch of maven-metadata.xml (narrow-manifest exception, not a
    full Firecrawl crawl — appropriate per scout policy). client-l has a NEW release, 1.1.0,
    uploaded 2026-07-02 (11 days before this check) — not yet reflected in cxr-l/release-notes.md
    or cxr-l/api-reference.md (both top out at 1.0.4) nor in the official portal changelog at
    open.rokid.com/sdk (which as of this check still shows 1.0.4 as latest, dated 2026.06.29).
    This is a genuine stale-version gap — see P1 in this cycle's report. client-m and
    cxr-service-bridge unchanged since last check.

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-07-13
  notes: Re-verified 2026-07-13, unchanged. Still only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware, plus its issue tracker). No in-scope Sprite/AR Glasses content. No action items.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-07-13
  notes: Re-verified 2026-07-13, unchanged. Map surfaces mostly Speech/OpenVoice/CloudApp/skill-kit repos (old smart-speaker product, out of scope) plus glass-docs (old Glass 1 era, last active ~2020) and UXR-docs (out of scope). No in-scope Sprite/AR Glasses content found.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-07-13
  notes: Re-verified 2026-07-13, unchanged. Map still returns empty links array (0 URLs). No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Re-checked 2026-07-13 (still not added to registry — Scout does not self-approve new sources).
    Map now returns 8 URLs (was 4 on 2026-06-28): root, /sdk?lang=zh, /sprite?lang=zh, /academy,
    /?lang=cn, /master?lang=cn, /sprite?lang=en, /sdk?lang=en. Notably, /sdk?lang=zh previously
    returned only an SPA shell (2026-06-28 note) but as of this check it renders the FULL official
    CXR-L changelog inline (v1.0.4 dated 2026.06.29, v1.0.3, v1.0.1 entries) — this is now the best
    live source for the official CXR-L changelog and should be preferred over developerdoc.rokid.com
    for that content once/if registered. /sprite?lang=zh content verified to match local
    yodaos/docs/sprite-overview.md structurally, no drift. /academy hosts 乐奇学院 (Rokid Academy)
    with CXR-L/Glasses courses (in scope) alongside UXR 3.0 and AIUI courses (out of scope) —
    click-through not performed, per scope rules. YodaOS-Master content at /master?lang=cn —
    skipped (out of scope), not scraped. Suggest registering open.rokid.com as the canonical
    replacement for ar.rokid.com / developerdoc.rokid.com; the Leader/user should confirm.
