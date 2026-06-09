# Rokid AR Glasses Platform Documentation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/buildwithfenna/rokid-docs/pulls)

Comprehensive technical documentation for the Rokid AR smart glasses ecosystem, covering the YodaOS operating system internals (built on Android 12/Qualcomm), hardware architecture, system services, and the CXR SDK suite (Mobile, On-Device, and Standalone) for developing companion and glasses-native applications.

> **Why does this exist?** Development documentation for Rokid glasses is scattered across Chinese-language sites, SDK JARs, and firmware dumps. This project aims to be the single, community-maintained source of truth for anyone building on the Rokid platform.

---

## Table of Contents

- [Overview](#overview)
- [Device Specifications](#device-specifications)
- [Product Variants](#product-variants)
- [Repository Structure](#repository-structure)
- [YodaOS](#yodaos)
  - [System Architecture](#system-architecture)
  - [Boot Chain](#boot-chain)
  - [Hardware Layer](#hardware-layer)
  - [Platform Services](#platform-services)
  - [System Applications](#system-applications)
  - [Development Reference](#development-reference)
- [CXR SDK Suite](#cxr-sdk-suite)
  - [CXR-M SDK (Mobile)](#cxr-m-sdk-mobile)
  - [CXR-S SDK (On-Device)](#cxr-s-sdk-on-device)
  - [CXR-L SDK (Standalone)](#cxr-l-sdk-standalone)
  - [SDK Communication Architecture](#sdk-communication-architecture)
- [Bare-Metal Development](#bare-metal-development)
- [Decompiled Sources](#decompiled-sources)
  - [Firmware](#firmware)
  - [System Applications](#decompiled-system-applications)
  - [Configuration Files](#configuration-files)
- [Documentation Index](#documentation-index)

---

## Overview

This repository contains reverse-engineered and authored documentation for the **Rokid AR Glasses** platform. It covers everything from low-level firmware and kernel modules to high-level SDK APIs for third-party developers. The documentation was generated through a combination of firmware decompilation (APKtool + JADX), system property extraction, and analysis of Rokid's CXR SDK artifacts.

The platform runs **YodaOS**, Rokid's custom operating system built on Android 12 with Qualcomm's QSSI (Qualcomm Single System Image) architecture. The glasses communicate with companion mobile apps via Bluetooth and Wi-Fi Direct using the **CXR protocol suite**.

---

## Device Specifications

| Property | Value |
|---|---|
| **Model** | RG-glasses |
| **Brand** | Rokid |
| **OS** | YodaOS (Android 12, SDK 32) |
| **Build ID** | SKQ1.240613.001 |
| **Board Platform** | Qualcomm Neo |
| **Primary ABI** | arm64-v8a |
| **CPU (64-bit)** | Kryo 300 |
| **CPU (32-bit)** | Cortex-A75 |
| **Display** | JBD JBD4020 Micro-LED (right eye) |
| **IMU** | InvenSense ICM-4x6xx (accel, gyro, motion detect, freefall, temp) |
| **Speech Co-processor** | NXP RT600 (iFlytek + Rokid KWS) |
| **GPU Firmware** | Adreno 620/621/650/740v3 |
| **Low RAM Mode** | Enabled |
| **A/B OTA** | Virtual A/B |
| **Treble** | Enabled |
| **Default Locale** | zh-CN |
| **Build Fingerprint** | `Rokid/glasses/glasses:12/SKQ1.240613.001/1.12.009-20260109-150201:user/release-keys` |

---

## Product Variants

| OEM ID | Model | Display | Description |
|--------|-------|---------|-------------|
| 101 | RV101 | Yes | Rokid Glasses (domestic) |
| 102 | RV102 | Yes | Rokid Glasses (carrier edition) |
| 103 | RV203 | No | Rokid AI Glasses (no display) |
| 104 | RV101 | Yes | Rokid Glasses (state gift edition) |
| 105 | RV101 | Yes | Rokid Glasses (overseas) |
| 106 | RV101 | Yes | Rokid Glasses (Leqi Smart brand) |
| 201 | RV201 | No | Bolon AI Glasses |
| 202 | RV202 | No | Bolon AI Glasses (carrier edition) |
| 203 | RV201 | No | Bolon AI Glasses (celebrity custom, tinted gray) |

Each variant is identified by a `devicetypeid` UUID set during boot, which configures authentication keys, display presence, and boot animations.

---

## Repository Structure

```
rokid-docs/
├── cxr-m/                          # CXR-M SDK documentation (Mobile companion)
│   ├── intro.md                    # SDK overview
│   ├── sdk-integration.md         # Integration guide
│   ├── device-connection.md        # Device connection management
│   ├── get-device-status.md        # Querying device status
│   ├── data-interaction.md         # Data interaction patterns
│   ├── ai-integration.md           # AI integration with Sprite OS
│   ├── release-notes.md            # SDK changelog (v1.0.1 -> v1.2.2)
│   └── sdk-decompiled-reference.md # Full decompiled SDK reference
│
├── cxr-s/                          # CXR-S SDK documentation (On-device)
│   ├── brief.md                    # SDK overview
│   ├── development-environment.md  # Dev environment setup
│   ├── sdk-import.md               # SDK import guide
│   ├── manage-device-connection.md # Connection management
│   ├── message-subscription.md     # Receiving messages from mobile
│   ├── message-sending.md          # Sending messages to mobile
│   └── data-structure.md           # Caps serialization format
│
├── cxr-l/                          # CXR-L SDK documentation (Standalone)
│   ├── api-reference.md            # Complete API reference
│   └── release-notes.md            # SDK changelog
│
├── cxr-baremetal/                  # Bare-metal Android dev on Rokid Glasses (no CXR SDK)
│   ├── development-guide.md        # Reserved interactions + dev environment
│   ├── key-broadcasts.md           # Hardware-button Intent broadcasts
│   └── audio-recording.md          # 8-channel mic recording (ChannelMask 0x6000FC)
│
└── yodaos/                         # YodaOS platform documentation
    ├── docs/
    │   ├── overview.md             # Build info, CPU, architecture
    │   ├── sprite-overview.md      # YodaOS-Sprite developer-portal landing
    │   ├── apps/                   # System application docs
    │   ├── development/            # Development reference (HAL, permissions, etc.)
    │   ├── hardware/               # Hardware specifications
    │   ├── kernel/                 # Kernel modules
    │   ├── platform/               # Platform services (Speech SDK, AOSP)
    │   ├── system/                 # System internals (boot, init, properties)
    │   └── vendor/                 # Vendor services
    ├── DECOMPILED/                  # Decompiled system partitions
    │   ├── system/                  # Framework, services, configs
    │   ├── system_ext/              # Extended system apps, factory scripts
    │   ├── vendor/                  # Vendor HALs, firmware, scripts
    │   ├── product/                 # Rokid product apps
    │   ├── odm/                     # ODM partition
    │   └── apex/                    # APEX modules
    └── DECOMPILED-APPS/             # Full APK decompilation output
        └── product/app/             # Per-app decompiled sources (smali, jadx, res)
```

---

## YodaOS

### System Architecture

YodaOS is built on **Android 12 (API 32)** using the **Qualcomm QSSI** build system with the **Neo** board platform. It uses Qualcomm's "Go" (low-RAM) configuration, indicating a resource-constrained device. Key architectural features:

- **Treble-compliant**: vendor/system separation via VNDK 32
- **Virtual A/B OTA**: seamless updates with rollback support
- **Low RAM optimizations**: Go-edition Android configuration
- **APEX modules**: modular system components (WiFi, tethering, permissions, media, ext services)
- **Partitions**: system, system_ext, vendor, product, odm, apex

> [Full build details](yodaos/docs/overview.md)

### Boot Chain

The boot sequence follows Android init with Qualcomm BSP and Rokid customizations layered on top:

1. **early-init** -- Mount tracefs, device symlinks, load kernel modules
2. **init** -- Start logd, servicemanager, hwservicemanager, lmkd
3. **fs** -- Mount filesystems, run Rokid identity setup (`set_serialno`, `set_mfi`)
4. **post-fs** -- Configure persist partition (SN, MAC, type ID files)
5. **post-fs-data** -- Create data directories, start tombstoned, apexd
6. **zygote-start** -- Start statsd, netd, zygote
7. **early-boot** -- Run Qualcomm early boot scripts, start sensor subsystems (ADSP/CDSP/SLPI/CVP)
8. **boot** -- Configure Bluetooth/network/sensors, start HALs, resolve OEM variant via `init.rokid_oem_define.rc`

> [Full boot chain documentation](yodaos/docs/system/boot-chain.md)

### Hardware Layer

| Component | Details | Documentation |
|-----------|---------|---------------|
| **Display** | JBD JBD4020 Micro-LED panel (right eye), Qualcomm DPU 8.2.0, QDCM calibration | [display.md](yodaos/docs/hardware/display.md) |
| **Sensors** | InvenSense ICM-4x6xx IMU (accel, gyro, motion/freefall detect, temp) via I3C | [sensors.md](yodaos/docs/hardware/sensors.md) |
| **Audio** | Qualcomm audio HAL with AEC/ANT models | [audio.md](yodaos/docs/hardware/audio.md) |
| **GPU** | Adreno 620/621/650/740v3 firmware variants | [firmware.md](yodaos/docs/hardware/firmware.md) |
| **Thermal** | Thermal management and throttling | [thermal.md](yodaos/docs/hardware/thermal.md) |
| **Power** | Power and performance profiles | [power-performance.md](yodaos/docs/hardware/power-performance.md) |
| **Native Libs** | Qualcomm, GPU, camera, audio library catalog | [native-libraries.md](yodaos/docs/hardware/native-libraries.md) |

> [Product variants and hardware revisions](yodaos/docs/hardware/product-variants.md)

### Platform Services

#### Speech SDK

The Speech SDK runs on an **NXP RT600 co-processor** connected to the main Qualcomm SoC via SPI. It handles:

- Microphone input and noise reduction
- Acoustic echo cancellation (AEC)
- Wake word detection (KWS -- keyword spotting)
- Command word recognition

Speech processing is powered by **iFlytek (Xunfei)** for the front-end audio pipeline, with **Rokid's own KWS engine** introduced from firmware v5.0.0. Multiple firmware variants cover different hardware revisions (EVT1/EVT2/DVT), acoustic modes (near-field, far-field, noise-reduction), and languages (Chinese, English).

> [Full Speech SDK documentation](yodaos/docs/platform/speech-sdk.md)

#### Other Platform Services

| Service | Documentation |
|---------|---------------|
| AOSP Components | [aosp-components.md](yodaos/docs/platform/aosp-components.md) |
| PASRService (vendor) | [pasrservice.md](yodaos/docs/vendor/pasrservice.md) |
| Time Service (vendor) | [time-service.md](yodaos/docs/vendor/time-service.md) |
| TrustZone Access | [trustzone-access.md](yodaos/docs/vendor/trustzone-access.md) |

### System Applications

| App | Package | Description | Documentation |
|-----|---------|-------------|---------------|
| **CXRService** | `com.rokid.cxrservice` | Core glasses-side CXR communication bridge (Bluetooth) | [cxr-service.md](yodaos/docs/apps/cxr-service.md) |
| **RokidSpriteLauncher** | `com.rokid.os.sprite.launcher` | Main home launcher (camera, gallery, audio, chat, translate, navigation, music, settings) | [sprite-launcher.md](yodaos/docs/apps/sprite-launcher.md) |
| **RokidSpriteAssistServer** | `com.rokid.os.sprite.assistserver` | Central service hub: Bluetooth, WiFi, payment, TTS, media, camera, web server | [sprite-assist.md](yodaos/docs/apps/sprite-assist.md) |
| **RokidSysConfig** | `com.rokid.sysconfig` | System configuration service | [sys-config.md](yodaos/docs/apps/sys-config.md) |
| **RokidOtaUpgrade** | `com.rokid.glass.ota` | OTA firmware update client with download, verification, and reboot | [ota-upgrade.md](yodaos/docs/apps/ota-upgrade.md) |
| **RokidScreenRecord** | `com.rokid.os.master.screenstream` | Screen recording/streaming via broadcast intents | [screen-record.md](yodaos/docs/apps/screen-record.md) |
| **Camera2** | -- | Camera2 API integration | [camera2.md](yodaos/docs/apps/camera2.md) |
| **Chinese Payment Apps** | Alipay, AntPay, JdPay | Payment/commerce integrations | [chinese-apps.md](yodaos/docs/apps/chinese-apps.md) |

> [Full apps overview](yodaos/docs/apps/overview.md)

### Development Reference

| Topic | Documentation |
|-------|---------------|
| HAL Interfaces | [hal-interfaces.md](yodaos/docs/development/hal-interfaces.md) |
| Hardware Features | [hardware-features.md](yodaos/docs/development/hardware-features.md) |
| Native Libraries | [native-libraries.md](yodaos/docs/development/native-libraries.md) |
| Permissions | [permissions.md](yodaos/docs/development/permissions.md) |
| System Interfaces | [system-interfaces.md](yodaos/docs/development/system-interfaces.md) |
| System Properties | [system-properties.md](yodaos/docs/development/system-properties.md) |
| Kernel Modules | [modules.md](yodaos/docs/kernel/modules.md) |
| Init Services | [init-services.md](yodaos/docs/system/init-services.md) |
| Hardware Interaction | [hardware-interaction.md](yodaos/docs/system/hardware-interaction.md) |

---

## CXR SDK Suite

The CXR (Connected XR) SDK suite enables developers to build applications for the Rokid AR Glasses ecosystem. It consists of three SDKs, each targeting a different runtime environment:

```
┌─────────────────────────────────────────────────────────────────┐
│                     ROKID AR GLASSES                            │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   CXR-S SDK  │    │   CXR-L SDK  │    │  CXRService  │      │
│  │  (on-device  │    │ (standalone  │    │   (system    │      │
│  │    bridge)   │    │    apps)     │    │   bridge)    │      │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘      │
│         │                   │                   │               │
│         └───────────────────┼───────────────────┘               │
│                             │                                   │
│                    Bluetooth / Wi-Fi Direct                     │
└─────────────────────────────┼───────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │    CXR-M SDK      │
                    │  (mobile phone)   │
                    └───────────────────┘
```

### CXR-M SDK (Mobile)

The **CXR-M SDK** is a developer toolkit for building **mobile companion applications** (Android/iOS) that communicate with Rokid Glasses.

| Property | Value |
|----------|-------|
| **Maven Artifact (documented)** | `com.rokid.cxr:client-m:1.1.0` (2026-04-01) |
| **Maven Artifact (latest, undocumented)** | `com.rokid.cxr:client-m:1.2.2` (2026-06-08) |
| **Min SDK** | 28 (Android 9) |
| **Repository** | `https://maven.rokid.com/repository/maven-public/` |
| **Distribution** | Not on the public developer site — contact `Glasses.BD@rokid.com` |

**Capabilities:**
- **AI Interaction** -- Integrate the AI workflow defined by YodaOS-Sprite (custom AI and custom AI workflows)
- **Hardware Info** -- Query glasses hardware information
- **Assist Services** -- File transfer, audio recording, photo capture via Rokid Assist Service
- **Device Connection** -- BLE GATT + classic Bluetooth socket + Wi-Fi Direct

**Architecture:**

| Layer | Class | Role |
|-------|-------|------|
| Public API | `CxrApi` | Singleton entry point with callbacks/listeners |
| Core | `CxrController` | Request routing, ID management |
| Bluetooth | `BluetoothController` | BLE GATT + classic socket management |
| Wi-Fi | `WifiController` | Wi-Fi P2P peer discovery and connection |
| File | `FileController` | HTTP file sync and APK upload (port 8848) |
| Audio | `AudioController` | Bluetooth SCO audio routing |
| Protocol | `CXRSocketProtocol` | Native JNI framing layer |
| Serialization | `Caps` | Binary serialization format |

**Documentation:**

| Document | Description |
|----------|-------------|
| [intro.md](cxr-m/intro.md) | SDK overview and capabilities |
| [sdk-integration.md](cxr-m/sdk-integration.md) | Integration guide |
| [device-connection.md](cxr-m/device-connection.md) | Connection management |
| [get-device-status.md](cxr-m/get-device-status.md) | Querying device status |
| [data-interaction.md](cxr-m/data-interaction.md) | Data interaction patterns |
| [ai-integration.md](cxr-m/ai-integration.md) | AI integration with Sprite OS |
| [release-notes.md](cxr-m/release-notes.md) | SDK changelog (v1.0.1 → v1.2.2) |
| [sdk-decompiled-reference.md](cxr-m/sdk-decompiled-reference.md) | Complete decompiled SDK reference (31 classes) |

### CXR-S SDK (On-Device)

The **CXR-S SDK** is the on-device development toolkit running on YodaOS-Sprite, enabling developers to build **applications that run directly on the glasses**. It provides access to the data channel and establishes two-way communication with the CXR-M SDK on mobile.

| Property | Value |
|----------|-------|
| **Maven Artifact** | `com.rokid.cxr:cxr-service-bridge:1.0` |
| **Classes** | 16 classes, 6 decompiled files |

**Capabilities:**
- **Connection Monitoring** -- Real-time Android/iOS connection/disconnection status
- **ARTC Health** -- Monitor data transmission health status
- **Message Subscription** -- Receive command messages from mobile
- **Message Sending** -- Send structured data (Caps) or binary streams (byte[]) to mobile
- **Bidirectional Communication** -- Full duplex mobile-to-glasses data channel

**Documentation:**

| Document | Description |
|----------|-------------|
| [brief.md](cxr-s/brief.md) | SDK overview |
| [development-environment.md](cxr-s/development-environment.md) | Dev environment setup |
| [sdk-import.md](cxr-s/sdk-import.md) | SDK import guide |
| [manage-device-connection.md](cxr-s/manage-device-connection.md) | Connection management |
| [message-subscription.md](cxr-s/message-subscription.md) | Receiving messages |
| [message-sending.md](cxr-s/message-sending.md) | Sending messages |
| [data-structure.md](cxr-s/data-structure.md) | Caps serialization format |

### CXR-L SDK (Standalone)

The **CXR-L SDK** is for building **standalone apps that replace the default Rokid apps entirely**. It communicates with the Rokid AI app service (`com.rokid.sprite.aiapp`) via AIDL bound service.

| Property | Value |
|----------|-------|
| **Maven Artifact (current decompile)** | `com.rokid.cxr:client-l:1.0.1` |
| **Maven Artifact (latest)** | `com.rokid.cxr:client-l:1.0.3` (2026-06-02) |
| **Size** | 65,494 bytes (1.0.3 AAR); 57,145 bytes (1.0.2 AAR) |
| **Min SDK** | 28 (1.0.1–1.0.2) / **31** (1.0.3+) |
| **Target SDK** | 28 |
| **Dependencies (1.0.3)** | cxr-service-bridge 1.0-20260522.063600-105, Kotlin stdlib 1.6.0, Gson 2.10.1 |
| **Repository** | `https://maven.rokid.com/repository/maven-public/` |

**Entry Point:**
```kotlin
class CXRLink(context: Context) : ExternalAppClient(context)
```

`CXRLink` extends `ExternalAppClient`, which binds to `IMediaStreamService` via Android AIDL for media streaming and AI app integration.

> [Full API reference](cxr-l/api-reference.md) · [Release notes](cxr-l/release-notes.md)

### SDK Communication Architecture

The three SDKs work together to form a complete communication stack:

| Path | Protocol | Use Case |
|------|----------|----------|
| **Mobile to Glasses** | BLE GATT + Classic Bluetooth Socket | Device discovery, pairing, control commands |
| **Mobile to Glasses** | Wi-Fi Direct (port 8848) | File transfer, APK upload, high-bandwidth data |
| **CXR-M to CXR-S** | Caps serialization over Bluetooth/Wi-Fi | Structured bidirectional messaging |
| **CXR-L to AI Service** | Android AIDL (IMediaStreamService) | On-device AI app integration |

**Caps Serialization Format** -- The shared binary serialization format used across all SDKs, supporting: booleans, integers (int32/int64), floats (float/double), strings, binary blobs (byte[]), and nested Caps objects.

---

## Bare-Metal Development

For apps that run directly on the glasses **without any CXR SDK** -- plain, side-loaded Android apps that handle their own input, audio, and camera. This is the track to use when you are *not* integrating with the on-device Rokid AI app or a mobile companion (for those, use the [CXR SDK Suite](#cxr-sdk-suite)).

| Property | Value |
|----------|-------|
| **Target** | YodaOS-Sprite (Android 12, Go-edition) |
| **Display viewport** | 480 × 640 px |
| **Connectivity** | ADB over the dedicated developer cable (the in-box charge cable is charge-only); enable ADB via the Rokid AI mobile app |
| **Doc version** | v0.0.1 (2026-03-01) |

YodaOS-Sprite reserves some interactions that bare-metal apps **cannot** override (long-press the touchpad to enter the AI module, double-click for back, top button for photo, long-press top button for video). All other buttons and touchpad gestures are delivered as system broadcast Intents.

**Documentation:**

| Document | Description |
|----------|-------------|
| [development-guide.md](cxr-baremetal/development-guide.md) | Reserved interactions, dev environment, and side-load workflow |
| [key-broadcasts.md](cxr-baremetal/key-broadcasts.md) | Hardware-button system Intents and the `BroadcastReceiver` pattern |
| [audio-recording.md](cxr-baremetal/audio-recording.md) | 8-channel mic recording (`ChannelMask = 0x6000FC`, 16 kHz, 16-bit PCM) |

---

## Decompiled Sources

### Firmware

The `yodaos/DECOMPILED/vendor/firmware/` directory contains firmware binaries and changelogs:

**GPU Firmware:**
- Adreno 620, 621, 650, 740v3 shader/microcode binaries

**Camera Firmware:**
- ICP (Image Control Processor) firmware

**Computer Vision:**
- EVA firmware (face detection, CV pipelines)
- VPU firmware (video processing)

**Speech Firmware (NXP RT600):**

| Variant | Version | Mode | Language |
|---------|---------|------|----------|
| dvt-far-en/zh | 3.0.4 | Far-field | English/Chinese |
| dvt-near-en | 5.1.0 | Near-field | English |
| dvt-near-zh | 3.3.0 | Near-field | Chinese |
| dvt-nr-en/zh | 3.0.4 | Noise reduction | English/Chinese |
| evt1-nr-zh | Legacy | Noise reduction | Chinese |
| evt2-far/near/nr-zh | Legacy | Various | Chinese |

**Firmware Changelogs:**
- [CHANGELOG_5.1.2-release](yodaos/DECOMPILED/vendor/firmware/CHANGELOG_5.1.2-release_202511262111.md) -- DSP frequency optimization, audio sync fixes
- [CHANGELOG_5.1.0-release_near_en](yodaos/DECOMPILED/vendor/firmware/CHANGELOG_5.1.0-release_near_en_202510251451.md) -- DSP optimization, recording improvements
- [CHANGELOG_3.3.0-release_near_zh](yodaos/DECOMPILED/vendor/firmware/CHANGELOG_3.3.0-release_near_zh_202510251433.md) -- Near-field Chinese variant
- [CHANGELOG_3.0.4-release_far](yodaos/DECOMPILED/vendor/firmware/CHANGELOG_3.0.4-release_far_202510251440.md) -- Far-field variant
- [CHANGELOG_3.0.4-release_nr](yodaos/DECOMPILED/vendor/firmware/CHANGELOG_3.0.4-release_nr_202510251445.md) -- Noise reduction variant

### Decompiled System Applications

Each application in `yodaos/DECOMPILED-APPS/` is decompiled using both APKtool and JADX:

```
<app>/
├── apktool/                # APKtool output
│   ├── AndroidManifest.xml # App manifest
│   ├── smali/              # Dalvik bytecode (classes)
│   ├── res/                # Resources (layouts, drawables, values)
│   ├── assets/             # Raw assets (Lottie animations, configs, sounds)
│   └── original/           # Original APK metadata
├── jadx/                   # JADX output (decompiled Java source)
└── apktool.yml             # APKtool config
```

**Decompiled apps include:**
- `RokidSpriteLauncher` -- Home launcher with Lottie animations, multi-page UI
- `RokidSpriteAssistServer` -- Central system service hub
- `RokidOtaUpgrade` -- OTA update client
- `RokidScreenRecord` -- Screen recording service
- `Camera2` -- Camera app
- `SettingsIntelligence` -- Search and settings indexing
- `Alipay`, `AntPay`, `JdPay` -- Chinese payment integrations

### Configuration Files

| File | Location | Description |
|------|----------|-------------|
| `cxr-service.json` | `DECOMPILED/system/system/etc/` | CXR service config (MTU, MFI, buffer sizes, AEC/ANT, ARTC) |
| `build.prop` | Multiple partitions | Build properties per partition |
| `cgroups.json` | `DECOMPILED/system/system/etc/` | Control group configuration |
| Sensor configs | `DECOMPILED/vendor/etc/sensors/config/` | IMU and sensor HAL configuration (JSON) |
| Display calibration | `DECOMPILED/vendor/etc/display/` | QDCM panel calibration data |
| Factory scripts | `DECOMPILED/system_ext/factoryV2/` | Hardware factory test scripts (WiFi, BT, battery, IMU, LED, sensors, etc.) |

---

## Documentation Index

### Quick Reference

| Looking for... | Go to |
|----------------|-------|
| Build info and device specs | [yodaos/docs/overview.md](yodaos/docs/overview.md) |
| Hardware variants (RV101, RV201, etc.) | [yodaos/docs/hardware/product-variants.md](yodaos/docs/hardware/product-variants.md) |
| Boot sequence | [yodaos/docs/system/boot-chain.md](yodaos/docs/system/boot-chain.md) |
| Display hardware | [yodaos/docs/hardware/display.md](yodaos/docs/hardware/display.md) |
| Sensor specs | [yodaos/docs/hardware/sensors.md](yodaos/docs/hardware/sensors.md) |
| Speech/voice processing | [yodaos/docs/platform/speech-sdk.md](yodaos/docs/platform/speech-sdk.md) |
| All Rokid apps | [yodaos/docs/apps/overview.md](yodaos/docs/apps/overview.md) |
| Android permissions | [yodaos/docs/development/permissions.md](yodaos/docs/development/permissions.md) |
| Mobile app development | [cxr-m/intro.md](cxr-m/intro.md) |
| On-device app development | [cxr-s/brief.md](cxr-s/brief.md) |
| Standalone app replacement | [cxr-l/api-reference.md](cxr-l/api-reference.md) |
| CXR-L SDK changelog | [cxr-l/release-notes.md](cxr-l/release-notes.md) |
| CXR-M SDK changelog | [cxr-m/release-notes.md](cxr-m/release-notes.md) |
| YodaOS-Sprite developer portal | [yodaos/docs/sprite-overview.md](yodaos/docs/sprite-overview.md) |
| Bare-metal Android dev on Glasses | [cxr-baremetal/development-guide.md](cxr-baremetal/development-guide.md) |
| Hardware-button broadcast Intents | [cxr-baremetal/key-broadcasts.md](cxr-baremetal/key-broadcasts.md) |
| 8-channel mic recording on Glasses | [cxr-baremetal/audio-recording.md](cxr-baremetal/audio-recording.md) |
| Full CXR-M SDK API | [cxr-m/sdk-decompiled-reference.md](cxr-m/sdk-decompiled-reference.md) |
| Caps data format | [cxr-s/data-structure.md](cxr-s/data-structure.md) |
| Firmware changelogs | [yodaos/DECOMPILED/vendor/firmware/](yodaos/DECOMPILED/vendor/firmware/) |

### All Documentation Files

**CXR-M SDK (Mobile)**
1. [intro.md](cxr-m/intro.md) -- SDK overview
2. [sdk-integration.md](cxr-m/sdk-integration.md) -- Integration guide
3. [device-connection.md](cxr-m/device-connection.md) -- Connection management
4. [get-device-status.md](cxr-m/get-device-status.md) -- Device status queries
5. [data-interaction.md](cxr-m/data-interaction.md) -- Data interaction
6. [ai-integration.md](cxr-m/ai-integration.md) -- AI integration
7. [release-notes.md](cxr-m/release-notes.md) -- SDK changelog
8. [sdk-decompiled-reference.md](cxr-m/sdk-decompiled-reference.md) -- Full decompiled reference

**CXR-S SDK (On-Device)**
1. [brief.md](cxr-s/brief.md) -- SDK overview
2. [development-environment.md](cxr-s/development-environment.md) -- Environment setup
3. [sdk-import.md](cxr-s/sdk-import.md) -- SDK import
4. [manage-device-connection.md](cxr-s/manage-device-connection.md) -- Connection management
5. [message-subscription.md](cxr-s/message-subscription.md) -- Receiving messages
6. [message-sending.md](cxr-s/message-sending.md) -- Sending messages
7. [data-structure.md](cxr-s/data-structure.md) -- Caps data structure

**CXR-L SDK (Standalone)**
1. [api-reference.md](cxr-l/api-reference.md) -- API reference
2. [release-notes.md](cxr-l/release-notes.md) -- SDK changelog

**Bare-metal Android Development on Rokid Glasses**
1. [development-guide.md](cxr-baremetal/development-guide.md) -- Reserved interactions + dev environment + side-load workflow
2. [key-broadcasts.md](cxr-baremetal/key-broadcasts.md) -- Hardware-button system Intents and `BroadcastReceiver` pattern
3. [audio-recording.md](cxr-baremetal/audio-recording.md) -- 8-channel mic recording (`ChannelMask = 0x6000FC`)

**YodaOS -- System**
1. [overview.md](yodaos/docs/overview.md) -- Build info and architecture
2. [sprite-overview.md](yodaos/docs/sprite-overview.md) -- YodaOS-Sprite developer-portal landing
3. [boot-chain.md](yodaos/docs/system/boot-chain.md) -- Boot sequence
4. [init-services.md](yodaos/docs/system/init-services.md) -- Init services
5. [system-properties.md](yodaos/docs/system/system-properties.md) -- System properties
6. [hardware-interaction.md](yodaos/docs/system/hardware-interaction.md) -- Hardware interaction

**YodaOS -- Hardware**
1. [product-variants.md](yodaos/docs/hardware/product-variants.md) -- Product variants
2. [display.md](yodaos/docs/hardware/display.md) -- Display panel
3. [sensors.md](yodaos/docs/hardware/sensors.md) -- Sensors (IMU)
4. [audio.md](yodaos/docs/hardware/audio.md) -- Audio
5. [firmware.md](yodaos/docs/hardware/firmware.md) -- GPU/camera/CV firmware
6. [thermal.md](yodaos/docs/hardware/thermal.md) -- Thermal management
7. [power-performance.md](yodaos/docs/hardware/power-performance.md) -- Power/performance
8. [native-libraries.md](yodaos/docs/hardware/native-libraries.md) -- Native libraries

**YodaOS -- Applications**
1. [overview.md](yodaos/docs/apps/overview.md) -- All Rokid apps
2. [cxr-service.md](yodaos/docs/apps/cxr-service.md) -- CXRService
3. [sprite-launcher.md](yodaos/docs/apps/sprite-launcher.md) -- Home launcher
4. [sprite-assist.md](yodaos/docs/apps/sprite-assist.md) -- Assist server
5. [sys-config.md](yodaos/docs/apps/sys-config.md) -- System config
6. [ota-upgrade.md](yodaos/docs/apps/ota-upgrade.md) -- OTA updates
7. [screen-record.md](yodaos/docs/apps/screen-record.md) -- Screen recording
8. [camera2.md](yodaos/docs/apps/camera2.md) -- Camera
9. [chinese-apps.md](yodaos/docs/apps/chinese-apps.md) -- Payment apps

**YodaOS -- Platform**
1. [speech-sdk.md](yodaos/docs/platform/speech-sdk.md) -- Speech SDK
2. [aosp-components.md](yodaos/docs/platform/aosp-components.md) -- AOSP components

**YodaOS -- Development**
1. [hal-interfaces.md](yodaos/docs/development/hal-interfaces.md) -- HAL interfaces
2. [hardware-features.md](yodaos/docs/development/hardware-features.md) -- Hardware features
3. [native-libraries.md](yodaos/docs/development/native-libraries.md) -- Native libraries
4. [permissions.md](yodaos/docs/development/permissions.md) -- Permissions
5. [system-interfaces.md](yodaos/docs/development/system-interfaces.md) -- System interfaces
6. [system-properties.md](yodaos/docs/development/system-properties.md) -- System properties

**YodaOS -- Kernel**
1. [modules.md](yodaos/docs/kernel/modules.md) -- Kernel modules

**YodaOS -- Vendor Services**
1. [pasrservice.md](yodaos/docs/vendor/pasrservice.md) -- PASRService
2. [time-service.md](yodaos/docs/vendor/time-service.md) -- Time service
3. [trustzone-access.md](yodaos/docs/vendor/trustzone-access.md) -- TrustZone access

---

## Contributing

This is a community-driven project and contributions are welcome! Whether you've found an error, have new SDK findings, want to add a tutorial, or can help translate Chinese documentation to English, we'd love your help.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on how to contribute.

### Areas that need help

- English translations of Chinese-language SDK docs and UI strings
- Getting started guides and tutorials for common tasks
- Code examples for CXR-M, CXR-S, and CXR-L SDK usage
- Troubleshooting guides based on real development experience
- Documentation for undocumented SDK features and system APIs
- Hardware findings from reverse engineering or hands-on development

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

## Disclaimer

This documentation is community-maintained and includes information obtained through reverse engineering. It is not affiliated with or endorsed by Rokid. Use at your own risk. Rokid and YodaOS are trademarks of Rokid Co., Ltd.
