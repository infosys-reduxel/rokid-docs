# Sources & Provenance

This documentation is **community-maintained** and assembled from a mix of
upstream Rokid developer material and **reverse engineering** of shipped
firmware and SDK artifacts. This page records where the information comes from
so readers can verify and trace it. It is not affiliated with or endorsed by
Rokid.

> Every documentation page also shows a short provenance line in its footer
> linking to the relevant upstream entry point and to its Markdown source on
> GitHub.

## How to read provenance

- **Upstream portal docs** — translated/adapted from Rokid's Chinese-language
  developer sites. The exact rendered pages on those sites are JavaScript-
  generated and live behind unstable per-document hashes, so we link the
  canonical **entry point** rather than a deep link that would rot.
- **Reverse-engineered material** — derived from decompiled APKs/JARs
  (APKtool + JADX) and firmware dumps. These pages state the **firmware** or
  **SDK version** they were derived from, per the repository convention.

## Firmware baseline

Reverse-engineered YodaOS content is derived from the following shipped build
(see [yodaos/docs/overview.md](yodaos/docs/overview.md) for the full table):

| Property | Value |
|---|---|
| Build ID | `SKQ1.240613.001` |
| Build fingerprint | `Rokid/glasses/glasses:12/SKQ1.240613.001/1.12.009-20260109-150201:user/release-keys` |
| OS | YodaOS (Android 12, API 32) |
| Board | Qualcomm Neo (QSSI, Go-edition) |

## Upstream sources

The following upstream sources are monitored for changes. Versions noted are the
last known as of **2026-05-29**.

| Source | Kind | Covers | Notes |
|---|---|---|---|
| [ar.rokid.com](https://ar.rokid.com) | Developer portal | CXR-M/S/L, YodaOS | Top-level portal. `/sdk` iframe-wraps `developerdoc.rokid.com`. |
| [developerdoc.rokid.com/sdk](https://developerdoc.rokid.com/sdk?lang=zh) | Developer portal | CXR-M/S/L, baremetal | Actual SDK landing. Tabs: CXR-L / CXR-M / 眼镜端裸机开发 (baremetal), under the YodaOS-Sprite top tab. |
| [developerdoc.rokid.com/sprite](https://developerdoc.rokid.com/sprite) | Developer portal | YodaOS-Sprite | YodaOS-Sprite open API / SDK index. |
| [maven.rokid.com](https://maven.rokid.com/repository/maven-public/) | Maven | CXR-M/S/L | Canonical "what shipped" for SDK AAR/JAR versions. Maven leads the portal landing pages by ~1 minor. |
| [github.com/Rokid-AR](https://github.com/Rokid-AR) | GitHub org | CXR-M/S/L | AR Glasses sample-app / SDK demos. |
| [github.com/RokidGlass](https://github.com/RokidGlass) | GitHub org | CXR / YodaOS / HW | "Rokid Glass Developer Docs and SDK" (older Glass 2 overlap TBD). |
| [github.com/rokid](https://github.com/rokid) | GitHub org | YodaOS / HW | Official org; mostly Speech/OpenVoice. |

### Last known SDK versions (upstream)

| Artifact | Maven (shipped) | Portal landing |
|---|---|---|
| `com.rokid.cxr:client-m` | 1.2.1 | 1.1.0 |
| `com.rokid.cxr:client-l` | 1.0.2 | 1.0.1 |
| `com.rokid.cxr:cxr-service-bridge` | 1.0 | — |
| 眼镜端裸机开发 (baremetal) | — | 0.0.1 (2026-03-01) |

> The canonical machine-readable registry that drives upstream monitoring lives
> at `.claude/agents/rokid-sources.md` in this repository.

## Reporting source issues

Found a mis-attributed source, a stale version pin, or content that needs a
citation? Please open an issue or PR — see
[CONTRIBUTING.md](CONTRIBUTING.md).
