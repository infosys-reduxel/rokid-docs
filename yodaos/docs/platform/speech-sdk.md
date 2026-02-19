# YodaOS Speech SDK Documentation

Extracted from decompiled Rokid AR glasses firmware (Android 12 / YodaOS).

Source partition: `vendor/firmware/`

---

## Overview

The Speech SDK runs on an NXP RT600 co-processor connected to the main Qualcomm SoC via SPI. It handles all on-device speech processing: microphone input, noise reduction, acoustic echo cancellation (AEC), wake word detection (KWS -- keyword spotting), and command word recognition. The speech algorithms are powered by iFlytek (Xunfei) for the front-end audio processing pipeline, with Rokid's own KWS engine introduced starting from firmware version 5.0.0.

The firmware is identified as `sdk20-app-release` and ships in multiple variants covering different hardware revisions (EVT1, EVT2, DVT), acoustic modes (near-field, far-field, noise-reduction), and languages (Chinese, English).

---

## SDK Firmware Variants

### DVT Hardware (Current Production)

| Filename | Version | Build Date | Git Hash | Size |
|---|---|---|---|---|
| `sdk20-app-release-dvt-far-en.bin` | 3.0.4 | 2025-10-25 14:40 | 620cba3 | 1,365,792 bytes (1.30 MB) |
| `sdk20-app-release-dvt-far-zh.bin` | 3.0.4 | 2025-10-25 14:40 | 620cba3 | 1,366,368 bytes (1.30 MB) |
| `sdk20-app-release-dvt-near-en.bin` | 5.1.0 | 2025-10-25 14:51 | 0c92e5a | 1,441,416 bytes (1.37 MB) |
| `sdk20-app-release-dvt-near-zh.bin` | 3.3.0 | 2025-10-25 14:54 | 19cbda4 | 1,427,904 bytes (1.36 MB) |
| `sdk20-app-release-dvt-nr-en.bin` | 3.0.4 | 2025-10-25 14:45 | ef0836f | 1,338,144 bytes (1.28 MB) |
| `sdk20-app-release-dvt-nr-zh.bin` | 3.0.4 | 2025-10-25 14:45 | ef0836f | 1,338,720 bytes (1.28 MB) |

### EVT Hardware (Earlier Revisions, Chinese Only)

| Filename | Size | Notes |
|---|---|---|
| `sdk20-app-release-evt1-nr-zh.bin` | 1,340,712 bytes (1.28 MB) | EVT1, noise reduction, Chinese |
| `sdk20-app-release-evt2-far-zh.bin` | 1,368,392 bytes (1.30 MB) | EVT2, far-field, Chinese |
| `sdk20-app-release-evt2-near-zh.bin` | 1,368,392 bytes (1.30 MB) | EVT2, near-field, Chinese |
| `sdk20-app-release-evt2-nr-zh.bin` | 1,340,744 bytes (1.28 MB) | EVT2, noise reduction, Chinese |

No version header files exist for the EVT variants. They are likely legacy builds retained for hardware compatibility.

### Naming Convention

`sdk20-app-release-{hw_revision}-{acoustic_mode}-{language}.bin`

- **hw_revision**: `evt1`, `evt2`, `dvt`
- **acoustic_mode**: `near` (near-field), `far` (far-field), `nr` (noise reduction / omni-directional)
- **language**: `en` (English), `zh` (Chinese)

### Acoustic Modes

- **Near-field (`near`)**: Optimized for the wearer's own voice. Lower gain settings (12dB output gain as of v2.1.1). Starting from v5.0.0, uses iFlytek front-end + Rokid's own KWS engine.
- **Far-field (`far`)**: Captures voices from further away. Higher mic gain (+6dB added in v3.0.1). Balances suppression of wearer voice vs. downward pitch angles.
- **Noise reduction / omni (`nr`)**: General noise reduction mode. 10dB omni microphone gain boost added in v3.1.1.

---

## Version History

### Near-field English (CHANGELOG_5.1.0-release_near_en / CHANGELOG_5.1.2-release)

The near-field English firmware has diverged to a separate version track starting at 5.x, reflecting the switch to iFlytek front-end + Rokid KWS.

| Version | Date | Highlights |
|---|---|---|
| **5.1.2** | 2025-11-26 | Dynamically raise CM33 frequency to speed up DSP image decompression. Fix noise when switching between scenarios. |
| **5.1.1** | -- | Add SPI thread semaphore timeout to fix wake events not reaching AR1. Increase DMIC buffer count/length to fix audio pipeline stalls. |
| **5.1.0** | 2025-10-25 | Lower DSP clock to 230 MHz to fix recording anomalies on some units. |
| **5.0.0-beta** | 2025-10-15 | Add Rokid proprietary near-field English keyword wake. Marks the transition to iFlytek front-end + Rokid KWS architecture. |
| **3.1.1-beta** | 2025-09-24 | Omni microphone gain +10dB. |
| **3.1.0-beta** | 2025-09-17 | Update iFlytek to 2025-09-17 build. First version of deeply customized British/American English wake word resources. |
| **3.1.0-beta** | 2025-09-12 | Update iFlytek to 2025-09-06 build. Improve wake rate in noisy environments. |

### Near-field Chinese (CHANGELOG_3.3.0-release_near_zh)

| Version | Date | Highlights |
|---|---|---|
| **3.3.0-beta** | 2025-10-20 | Lower DSP clock to 230 MHz. Fix recording anomalies on certain devices. |
| **3.1.0-beta** | 2025-09-17 | Update iFlytek to 2025-09-17. English/American accent wake word resources v1. |
| **3.1.0-beta** | 2025-09-12 | Update iFlytek to 2025-09-06. Improve noisy environment wake rate. |

### Far-field (CHANGELOG_3.0.4-release_far)

| Version | Date | Highlights |
|---|---|---|
| **3.0.4-beta** | 2025-09-06 | Optimize RT600 sleep strategy for AR1 host frequent sleep/wake cycles. |
| **3.0.3-beta** | 2025-08-19 | Fix intermittent recording data anomalies. |
| **3.0.2-beta** | 2025-08-11 | Fix audio channel misalignment. |
| **3.0.1-beta** | 2025-08-08 | Far-field mic gain +6dB. Optimize AGC convergence time. |
| **3.0.0-beta** | 2025-07-25 | Support Chinese/English switching. |

### Noise Reduction (CHANGELOG_3.0.4-release_nr)

Same version history as far-field up to 3.0.4. Identical changelog content, same RT600 sleep optimization.

### Shared History (All Variants, v2.x)

| Version | Date | Highlights |
|---|---|---|
| **2.10.5-beta** | 2025-07-08 | Adjust DMA channel priority. Fix audio channel misalignment. |
| **2.10.4-beta** | 2025-06-28 | Fix channel misalignment. Set CM33 52 MHz, DSP 260 MHz. |
| **2.10.3-beta** | 2025-06-26 | Fix channel misalignment. |
| **2.10.2-beta** | 2025-06-23 | Fix channel misalignment. |
| **2.10.1-beta** | 2025-06-20 | Raise DSP clock to 250 MHz. |
| **2.10.0-beta** | 2025-06-19 | Update iFlytek algorithms (2025-06-19). Far-field model: balance wearer suppression vs. downward pitch. Lower DSP clock 250 -> 200 MHz. |
| **2.9.4-beta** | 2025-06-19 | Add DVT AEC on/off toggle. DSP 250 MHz. Switch from PINT interrupt to GPIO polling for AEC toggle reliability. Update iFlytek engine to 1.0.1.1002.3651. |
| **2.8.1-beta** | 2025-06-12 | Fix audio stuttering. Fix some command words not working. |
| **2.8.0-beta** | 2025-06-11 | Update iFlytek algorithms (2025-06-11). Reduce false wake-ups. Improve far-field performance. |
| **2.7.0-beta** | 2025-06-09 | Update iFlytek back-end to 2025-06-06. Disable non-near-field wake. Fix omni channel misalignment. |
| **2.6.2-beta** | 2025-06-03 | Fix conditional channel misalignment bug. |
| **2.6.1-beta** | 2025-05-27 | Fix duplicate KWS event reporting. Power optimization: disable unused module clocks. |
| **2.6.0-beta** | 2025-05-24 | DSP frequency down to 200 MHz. Mic and AEC-I2S frame length 10ms -> 16ms. Fix null pointer deadlock in mic/AEC-I2S components. Split iFlytek processing to reduce peak latency. |
| **2.5.0-beta** | 2025-05-22 | Update iFlytek Chinese front-end model to 2025-05-15. |
| **2.4.0-beta** | 2025-05-16 | Support RT600 sleep/wake functionality. |
| **2.3.1-beta** | 2025-05-11 | Fix interrupt anomaly during AR1 sleep/wake. |
| **2.2.1-beta** | 2025-05-09 | Upgrade iFlytek: front-end 1.0.1.1002.3546, back-end 2025-04-24. Standardize wake word / command word IDs. |
| **2.1.1-beta** | 2025-05-01 | Near-field output gain 24dB -> 12dB. |
| **2.1.1-beta** | 2025-04-29 | DVT hardware support. Fix microphone order. Disable oneshot mode. |
| **2.1.0-beta** | 2025-04-21 | Adjust noise reduction model firmware output gain. |
| **2.1.0-beta** | 2025-04-16 | Split firmware into near/far/omni variants based on iFlytek models. |
| **2.1.0-beta** | 2025-04-14 | Fix audio reference path data lagging behind microphone channels. |

---

## Language and Locale Support

Two languages are supported:

- **Chinese (zh)**: Primary language. All hardware revisions (EVT1, EVT2, DVT) have Chinese firmware. Models use iFlytek Chinese front-end.
- **English (en)**: DVT hardware only. English support added from v3.0.0 (language switching). Dedicated English wake word resources with British/American accent customization from v3.1.0. Rokid's own KWS engine for near-field English added in v5.0.0.

Language switching at runtime is supported since v3.0.0 (far/nr) and presumably the same for near-field.

---

## Model Files Inventory

### Speech SDK Firmware (vendor/firmware/)

| File | Size | Purpose |
|---|---|---|
| `rt600_sbl.bin` | 17,324 bytes (16.9 KB) | RT600 secondary bootloader. Loads the SDK firmware onto the NXP RT600 co-processor. |
| `filler_keywords_en.bin` | 3,216 bytes (3.1 KB) | English filler/keyword dictionary for keyword spotting. Contains phoneme or word patterns used as rejection (filler) models. |
| `filler_keywords_zh.bin` | 3,952 bytes (3.9 KB) | Chinese filler/keyword dictionary for keyword spotting. |
| `mlp_en.bin` | 99,824 bytes (97.5 KB) | English MLP (multi-layer perceptron) neural network model for wake word / keyword detection scoring. |
| `mlp_zh.bin` | 130,052 bytes (127.0 KB) | Chinese MLP neural network model for wake word / keyword detection scoring. |

### ACD (Audio Context Detection) Models (vendor/etc/models/acd/)

| File | Size | Purpose |
|---|---|---|
| `speech.eai` | 16,448 bytes (16.1 KB) | Audio context detection model for speech activity. Used by the Qualcomm ADSP to classify audio as speech. |
| `music.eai` | 16,824 bytes (16.4 KB) | Audio context detection model for music activity. |
| `event.eai` | 115,136 bytes (112.4 KB) | Audio context detection model for general audio events (alarms, alerts, environmental sounds). |

### ANT (Audio Neural Technology) Models (system/system/etc/)

| File | Size | Purpose |
|---|---|---|
| `ant_rokid_model_v1.3.bin` | 1,792,328 bytes (1.71 MB) | Rokid ANT speech/sound enhancement model v1.3. Used by the `libant_rokid_se.so` sound enhancement library. |
| `ant_rokid_record_model.bin` | 3,103,988 bytes (2.96 MB) | Rokid ANT recording model. Used by the `libant_rokid_record.so` recording processing library. |

### ANT Shared Libraries (system/system/lib64/)

| File | Size | Purpose |
|---|---|---|
| `libant_rokid_se.so` | 969,416 bytes (946.7 KB) | Rokid ANT sound enhancement library (64-bit). |
| `libant_rokid_record.so` | 697,224 bytes (680.9 KB) | Rokid ANT recording processing library (64-bit). |
| `com.dsi.ant@1.0.so` | -- | ANT HAL interface library v1.0. |

### Other Relevant Files (vendor/etc/)

| File | Size | Purpose |
|---|---|---|
| `CaliData_For_RokidGlasses.bin` | 256 bytes | Calibration data for Rokid glasses audio/sensor hardware. |

---

## Wake Word / Keyword Detection

The keyword spotting (KWS) system operates in two architectural generations:

### Generation 1: iFlytek Full Pipeline (v2.1.0 through v3.x)

- iFlytek front-end performs noise reduction, beamforming, AEC, and voice activity detection.
- iFlytek back-end performs wake word detection and command word recognition.
- `mlp_zh.bin` / `mlp_en.bin` are MLP scoring models used alongside the iFlytek engine.
- `filler_keywords_zh.bin` / `filler_keywords_en.bin` provide filler word patterns for rejection.

### Generation 2: iFlytek Front-end + Rokid KWS (v5.0.0+)

- iFlytek still handles front-end audio processing (noise reduction, AEC, beamforming).
- Rokid's own KWS engine replaces the iFlytek back-end for keyword detection.
- Currently only applies to near-field English (`dvt-near-en`), which runs v5.1.0+.
- Near-field Chinese remains on the v3.x iFlytek-only track.

### DSP Clock History

The RT600 DSP frequency has been tuned repeatedly for power/performance balance:

- v2.1.0: unspecified initial frequency
- v2.6.0: lowered to 200 MHz
- v2.10.0: lowered to 200 MHz (from 250 MHz)
- v2.10.1: raised to 250 MHz
- v2.10.4: CM33 52 MHz, DSP 260 MHz
- v3.0.4 cherry-pick: DSP set to 230 MHz
- v5.1.0 / v3.3.0: settled at 230 MHz

---

## Speech Recognition Configuration Notes

- The RT600 co-processor communicates with the AR1 main SoC via SPI (kernel module `rt600_spiboot.ko` handles the boot and communication link).
- AEC (Acoustic Echo Cancellation) is toggleable since v2.9.4, using GPIO polling rather than PINT interrupts for reliability.
- Microphone frame length was increased from 10ms to 16ms in v2.6.0 for processing stability.
- The RT600 supports sleep/wake since v2.4.0, optimized in v3.0.4 for the AR1 host's frequent short sleep cycles.
- Wake word events are reported to the AR1 host. Duplicate event reporting was fixed in v2.6.1. SPI semaphore timeout for event delivery was added in v5.1.1.
