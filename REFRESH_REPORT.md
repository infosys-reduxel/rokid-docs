# Scheduled Refresh Report — 2026-06-19

**Status: BLOCKED — Firecrawl credits exhausted**

## What happened

The unattended documentation refresh run on 2026-06-19 could not complete because the Firecrawl account has **0 of 1,000 credits remaining**.

The Scout attempted to crawl all 8 upstream sources listed in `.claude/agents/rokid-sources.md`. Every source returned a "no credits" failure — no upstream pages were actually read. Per the Leader hard-stop conditions (upstream unverified → abort), no translations, commits, or pushes to main were performed.

## Sources that could not be checked

| Source | Kind | Covers | Last checked |
|--------|------|--------|--------------|
| https://ar.rokid.com | developer-portal | cxr-m, cxr-s, cxr-l, yodaos | 2026-06-11 |
| https://developerdoc.rokid.com | developer-portal | cxr-m, cxr-s, cxr-l, yodaos | 2026-06-11 |
| https://custom.rokid.com/prod/rokid_web/ | release-notes | cxr-m, cxr-s, cxr-l | 2026-06-11 |
| https://developer.rokid.com | developer-portal | yodaos, hardware | 2026-06-11 |
| https://maven.rokid.com/repository/maven-public/ | sdk-maven | cxr-m, cxr-s, cxr-l | 2026-06-11 |
| https://github.com/RokidGlass | github | all | 2026-06-11 |
| https://github.com/rokid | github | yodaos, hardware | 2026-06-11 |
| https://github.com/Rokid-AR | github | cxr-m, cxr-s, cxr-l | 2026-06-11 |

All sources were last successfully verified on **2026-06-11** (8 days ago). The `last_checked` dates in the registry were **not updated** — they remain at 2026-06-11 to accurately reflect when content was last actually read.

## Action required

Top up Firecrawl credits at the Firecrawl dashboard, then re-trigger the scheduled refresh. Once credits are available the Scout will perform a full crawl and the refresh cycle will complete normally.

This file can be deleted once the credits issue is resolved.
