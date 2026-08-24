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
  last_checked: 2026-08-24
  last_known_version: CXR-L 1.0.4 (portal card, unchanged); Maven has moved to 1.1.1 (undocumented, see maven.rokid.com entry)
  notes: |
    React SPA. Map still returns only the root URL (SPA shell) as of 2026-08-24 — unchanged
    from 2026-06-28 behavior. Real Sprite/CXR doc surfaces remain at developerdoc.rokid.com
    and the new custom.rokid.com CMS (now back online — see that entry below).
    YodaOS-Master tab remains; skipped (out of scope).

- url: https://developerdoc.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  monitor_id:
  last_checked: 2026-08-24
  last_known_version: CXR-L 1.0.4 (2026.06.25 per card), CXR-M 1.1.0 (portal lags Maven 1.2.2), 眼镜端裸机开发 1.0.0 (2026.06.05 per card)
  notes: |
    /sdk scraped 2026-08-24 with --wait-for 4000 (plain --only-main-content returns SPA shell
    only; needs a render wait). SDK selection table confirms: CXR-L 公开 1.0.4 (updated
    2026.06.25), CXR-M 商务合作 1.1.0 (business-only, updated 2026.04.01, unchanged), 眼镜端裸机开发
    公开 1.0.0 (updated 2026.06.05) — up from the 0.0.1 (2026-03-01) doc version previously
    tracked; matches the major bare-metal doc overhaul found on custom.rokid.com (see below).
    "展开全部内容" / "更新内容" expandable changelog sections did not render via Firecrawl
    click actions (selector text= not supported by this action engine version; [title=...]
    CSS selectors work on other pages). Map still returns the same 3 URLs as 2026-06-28.
    YodaOS-Master tab skipped (out of scope).

- url: https://custom.rokid.com/prod/rokid_web/
  kind: release-notes
  covers: cxr-m, cxr-s, cxr-l, cxr-baremetal
  monitor_id:
  last_checked: 2026-08-24
  notes: |
    BACK ONLINE as of 2026-08-24 (was BROKEN with OSS NoSuchKey as of 2026-06-28). Content
    is now served natively in **English** at /pc/us/ paths (previously Chinese-only /pc/cn/).
    Requires --wait-for 4000-13000ms (client-rendered SPA); per-chapter content is reached
    via Ant Design tree nav ([title="..."] CSS click selectors work) or documentId query
    params once known — no static <a href> per chapter, so map/crawl cannot discover the
    full page tree; must click through.
    Confirmed workspace hashes (2026-08-24):
    - 57e35cd3ae294d16b1b8fc8dcbb1b7c7 (CXR-M "Brief" + device-connection chapters, CN pc/cn/
      paths only found so far) — content verified byte-for-byte matching local cxr-m/intro.md
      and cxr-s/brief.md; no drift.
    - 84feb39f8ef141b0ad0326f902ab881f (CXR-L, EN pc/us/ + CN pc/cn/) — restructured into
      Introduction / Quick Start / Development Flow & State-Machine / Terms and Abbreviations /
      Feature Development (Android/iOS/Glasses subtrees, NOT yet crawled — large scope,
      deferred to a future cycle) / Version History. Version banner reads 1.0.4, matches
      Maven's last-documented release (Maven itself has moved to 1.1.0/1.1.1 undocumented).
    - ff28c865a9634876be98cbc293588460 (Bare-Metal dev, EN pc/us/ only) — NEW hash, previously
      shared under 57e35cd3.... Now a fully independent v1.0.0 workspace (was v0.0.1) with
      10 chapters: Introduction, Quick Start, Sample Project and Pages, Glasses UI Design
      Guidelines, and a "Features" subtree (Keys/Wear/Fold, Raw Audio, Photo, Video, Camera
      Preview Outlining [NEW topic], IMU and Sensors). Fully translated into
      cxr-baremetal/ this cycle (2026-08-24).
    - CXR-L hash 84feb39f... resolved via short-link https://t.rokid.com/uwxdzi51 (found on
      the new developer.rokid.com homepage's CXR-L card) — direct doc-portal URL was not
      otherwise discoverable via map/crawl.

- url: https://developer.rokid.com
  kind: developer-portal
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-24
  notes: |
    CHANGED as of 2026-08-24: no longer a plain redirect to open.rokid.com. Now serves its
    own redesigned English developer homepage (rokid-developer-homepage v1.2.0 static assets)
    with a "Products" section: Rokid Glasses (→ aiui-global.rokid.com, AIUI/AI Agent product —
    not in this repo's explicit in-scope list, treated as out of scope and skipped), Rokid AR
    (→ open.rokid.com/master, YodaOS-Master — out of scope, skipped), and a **NEW product,
    "Rokid Glass3"** (→ x-docs.rokid.com/docs/en/) not previously seen and not covered by
    CLAUDE.md's in-scope hardware list (RV101/RV102/RV203, Rokid AI Glasses). STOP CONDITION:
    per Leader instructions this is flagged as a possible wholly-new hardware line requiring
    human sign-off before any translation — x-docs.rokid.com was NOT scraped or actioned this
    cycle. The "Operating Systems" section correctly cross-links YodaOS-Sprite (CXR-L/CXR-S/
    bare-metal cards, matching custom.rokid.com hashes above) and YodaOS-Master (skipped).
    Also surfaces "AIUI Studio" (out of scope, not in in-scope list).

- url: https://maven.rokid.com/repository/maven-public/
  kind: sdk-maven
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-24
  last_known_version: |
    client-l 1.1.1 (release; metadata lastUpdated 20260814092031 — new releases 1.1.0 AND 1.1.1
      found 2026-08-24, jumping from the previously-tracked 1.0.4; latest tag shows
      1.2.X-SNAPSHOT in progress. Neither the developer portal (still shows 1.0.4) nor the
      custom.rokid.com Version History chapter (still tops out at v1.0.4) documents 1.1.0/1.1.1
      yet — no changelog available, so no entry was added to cxr-l/release-notes.md for these
      two releases this cycle; see the note added there instead)
    client-m 1.2.2 (release; metadata lastUpdated 20260608030211 — unchanged since 2026-06-09)
    cxr-service-bridge 1.0 (release; metadata lastUpdated 20260728074326 — version number
      unchanged, but republished/lastUpdated bumped from 20260522063622; likely an internal
      republish under the same version tag, not a new release)
  notes: |
    Public Maven for CXR SDK JARs/AARs. maven-metadata.xml under each artifact gives
    <release>, <latest>, <lastUpdated>; fetched directly via WebFetch/curl per Scout's
    narrow-manifest exception (Firecrawl not required for XML manifests).
    Use Maven as the canonical "what's actually shipped" source — it currently leads all
    documented CXR-L doc surfaces by two releases (1.1.0, 1.1.1).

- url: https://github.com/RokidGlass
  kind: github
  covers: cxr-m, cxr-s, cxr-l, yodaos, hardware
  monitor_id:
  last_checked: 2026-08-24
  notes: Verified org (id 57519491). "Rokid Glass Developer Docs and SDK". Only 2 public repos: UXR-docs (out-of-scope spatial computing SDK) and glass2-docs (out-of-scope Glass 2 / older hardware). No in-scope Sprite/AR Glasses content. No action items. Unchanged since 2026-06-30.

- url: https://github.com/rokid
  kind: github
  covers: yodaos, hardware
  monitor_id:
  last_checked: 2026-08-24
  notes: Verified org (id 19773259). Official "Rokid" org. Relevant repos: glass-docs (last commit 2020-07-13, old Glass 1/Glass 2 era content, not Sprite), UXR-docs (out of scope). Mostly Speech/OpenVoice/CloudApp repos. No in-scope Sprite/AR Glasses content found. Unchanged since 2026-06-30. Low priority.

- url: https://github.com/Rokid-AR
  kind: github
  covers: cxr-m, cxr-s, cxr-l
  monitor_id:
  last_checked: 2026-08-24
  notes: Verified org (id 25831739). Map returned empty links array (0 URLs) again on 2026-08-24. Previously confirmed 2 inactive repos (BroadcastServiceDemo from 2017). No actionable content. May have private repos not visible.

## New sources discovered (pending user approval to add to registry)

- url: https://open.rokid.com
  kind: developer-portal
  covers: cxr-m, cxr-s, cxr-l, yodaos
  status: UNREGISTERED — requires user approval
  notes: |
    Discovered 2026-06-28, not re-verified this cycle (2026-08-24) — deferred since
    developer.rokid.com's new homepage and the now-restored custom.rokid.com CMS already
    supplied enough signal for this cycle's actions. ar.rokid.com/sdk previously rendered a
    new AIUI-focused homepage linking to open.rokid.com/sdk and open.rokid.com/sprite.
    open.rokid.com/sprite?lang=zh previously matched developerdoc.rokid.com/sprite exactly.
    /academy hosts the "乐奇学院" (Rokid Academy) learning platform with CXR-L and Glasses
    development courses (in scope) alongside UXR 3.0 (out of scope) and AIUI courses.
    YodaOS-Master content linked from open.rokid.com/master — skipped (out of scope).
    Suggest registering open.rokid.com as the canonical replacement for ar.rokid.com;
    re-verify at next cycle.

- url: https://x-docs.rokid.com/docs/en/
  kind: developer-portal
  covers: (unknown — new hardware line, not yet in scope)
  status: UNREGISTERED — requires user approval AND explicit scope decision
  notes: |
    Discovered 2026-08-24 via developer.rokid.com's new "Products" section, linked under a
    product card labelled "Rokid Glass3". Not scraped or explored this cycle per the
    Leader's stop-condition rule for wholly-new hardware/product lines (CONTRIBUTING.md:
    new top-level sections need human sign-off first). CLAUDE.md's in-scope hardware list
    (RV101/RV102/RV203, Rokid AI Glasses) does not mention "Glass3" — unclear whether this
    is a new Sprite-line device (would be in scope) or a new product family entirely (would
    need an explicit scope decision, akin to Glass 2 being permanently out of scope). A
    human should visit x-docs.rokid.com/docs/en/ and decide before any Scout/Translator
    action is taken here.
