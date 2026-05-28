# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository nature

This is a **documentation-only repository** — Markdown plus Git LFS-tracked decompiled artifacts. There is no build system, test suite, package manifest, or lint config. Standard commands like `npm`, `make`, or `pytest` do not apply. "Running the project" means rendering Markdown locally or on GitHub.

The content is reverse-engineered/authored documentation for the **Rokid AR Glasses** platform — YodaOS (Android 12 / Qualcomm Neo) plus the **CXR SDK** suite. Treat the repo as a single source of truth that consolidates information otherwise scattered across Chinese-language sites, SDK JARs, and firmware dumps.

## Top-level layout

The repo splits into two halves with very different handling:

1. **Authored Markdown** — what humans read and edit:
   - `cxr-m/` — CXR-M SDK (mobile companion, Android/iOS)
   - `cxr-s/` — CXR-S SDK (on-device, runs on YodaOS-Sprite)
   - `cxr-l/` — CXR-L SDK (standalone apps replacing default Rokid apps)
   - `yodaos/docs/` — YodaOS platform docs (`apps/`, `development/`, `hardware/`, `kernel/`, `platform/`, `system/`, `vendor/`)

2. **Decompiled / firmware artifacts** — tracked with Git LFS, generally read-only references:
   - `yodaos/DECOMPILED/` — decompiled system partitions (`system/`, `system_ext/`, `vendor/`, `product/`, `odm/`, `apex/`)
   - `yodaos/DECOMPILED-APPS/product/app/<App>/{apktool,jadx}/` — per-app APKtool + JADX output
   - `yodaos/COMPILED - .../` — compiled OTA firmware
   - `yodaos/DUMPED/` — raw firmware dumps
   - `cxr-{m,s,l}/decompiled/` — decompiled SDK sources

   `.gitattributes` configures LFS for `yodaos/DECOMPILED*/**`, `yodaos/COMPILED*/**`, `yodaos/DUMPED/**`, `cxr-*/decompiled/**`, and `*.bin`/`*.fw`/`*.img`/`*.zip`. Don't commit large binaries outside these paths.

## Architectural mental model (for understanding cross-doc references)

The CXR SDK triple is the central abstraction the docs revolve around. Future questions will frequently hinge on which SDK does what:

- **CXR-M** (mobile) — `com.rokid.cxr:client-m`. Public API is `CxrApi` singleton; layered controllers (`BluetoothController`, `WifiController`, `FileController`, `AudioController`) sit over a native `CXRSocketProtocol` framing layer. Talks to glasses over **BLE GATT + classic Bluetooth socket** (control) and **Wi-Fi Direct on port 8848** (bulk/file).
- **CXR-S** (on-device) — `com.rokid.cxr:cxr-service-bridge`. Runs inside glasses apps; subscribes to messages from mobile and sends structured `Caps` payloads or `byte[]` back.
- **CXR-L** (standalone) — `com.rokid.cxr:client-l`. `CXRLink extends ExternalAppClient`, binds to `IMediaStreamService` via **Android AIDL** to integrate with the on-device AI app (`com.rokid.sprite.aiapp`). Used when *replacing* default Rokid apps rather than augmenting them.
- **CXRService** (system app, `com.rokid.cxrservice`) — the glasses-side bridge process. CXR-S clients talk through it.

The shared wire format across all three SDKs is **Caps** — binary serialization supporting booleans, int32/int64, float/double, strings, byte[], and nested Caps. When editing any SDK doc, cross-references to Caps should land in `cxr-s/data-structure.md`.

YodaOS itself is **Android 12 on Qualcomm QSSI (Neo board), Go-edition (low-RAM), Treble-compliant, Virtual A/B OTA**. The most-referenced facts (build fingerprint, OEM variant table, boot chain stages, Speech SDK on the NXP RT600 co-processor) all live in `yodaos/docs/overview.md`, `yodaos/docs/hardware/product-variants.md`, `yodaos/docs/system/boot-chain.md`, and `yodaos/docs/platform/speech-sdk.md`. The top-level `README.md` is a curated index into these — keep it in sync when adding new docs.

## Conventions specific to this repo

From `CONTRIBUTING.md` and observed patterns:

- File names: lowercase-kebab-case `.md` (e.g. `manage-device-connection.md`).
- ATX headers (`#`/`##`/`###`); fenced code blocks with language tags (` ```java`, ` ```kotlin`, ` ```xml`).
- Tables for structured data (specs, API params, OEM variants, firmware versions) — the existing docs lean heavily on tables, match that style.
- Use **relative links** between docs (e.g. `[boot chain](../system/boot-chain.md)`).
- When information comes from reverse engineering, say so, and note the **firmware version** or **SDK version** it was derived from. Decompiled findings should reference full Java package paths (`com.rokid.cxr...`).
- Line endings are normalized to LF for `*.md` via `.gitattributes`.

## Working in this repo

- Edit Markdown directly with the Edit tool; prefer editing existing files over creating new ones. New top-level sections should be raised in an issue first per `CONTRIBUTING.md`.
- The top-level `README.md` contains the master Documentation Index and Quick Reference table. When adding/renaming/moving any doc under `cxr-*/` or `yodaos/docs/`, update the corresponding entry in `README.md` so the index doesn't rot.
- Don't fabricate API signatures, version numbers, or hardware specs — the value of this repo is accuracy. If unsure, leave a TODO marker or check the relevant decompiled source under `yodaos/DECOMPILED-APPS/` or `cxr-*/decompiled/`. Note that LFS-tracked files may appear as pointer stubs unless LFS content is fetched (`git lfs pull`).
- Don't commit `.idea/`, `.vscode/`, build artifacts (`*.class`, `*.jar`, `*.apk`, `*.aab`, `build/`, `out/`), or `node_modules/` — all in `.gitignore`.
