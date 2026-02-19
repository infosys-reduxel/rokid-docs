# Firmware

Source: `vendor/firmware/`.

### 6.1 GPU Firmware (Adreno)

| File | Description |
|------|-------------|
| `a620_zap.*` (b00-b02, elf, mbn, mdt) | Adreno 620 ZAP (Secure Access Protection) firmware, split-format |
| `a621_gmu.bin` | Adreno 621 GMU (Graphics Management Unit) firmware |
| `a650_sqe.fw` | Adreno 650 SQE (Shader/Queue Engine) microcode |
| `a740v3_zap.*` (b00-b02, elf, mbn, mdt) | Adreno 740v3 ZAP firmware, split-format |
| `a740v3_sqe.fw` | Adreno 740v3 SQE microcode |
| `gmu_gen70200.bin` | GMU firmware for Gen7.0.200 GPU |

GPU kernel module: `msm_kgsl.ko` (Kernel Graphics Support Layer).
GPU clock control: `gpucc-neo.ko`.

### 6.2 Camera Firmware (ICP)

| File | Description |
|------|-------------|
| `CAMERA_ICP.*` (b00-b20, elf, mbn, mdt) | Camera ICP (Image Control Processor) firmware, 21 segments |
| `CAMERA_ICP_170.elf` | ICP firmware variant for IPE/BPS v1.7.0 |
| `CAMERA_ICP_480.elf` | ICP firmware variant for IPE/BPS v4.8.0 |

Camera kernel module: `camera.ko`, clock control: `camcc-neo.ko`.

### 6.3 EVA Firmware (AI/Computer Vision)

| File | Description |
|------|-------------|
| `evass.*` (b01-b19, mbn, mdt) | EVA (Enhanced Video Analytics) subsystem firmware, 19 segments |

EVA kernel module: `msm-eva.ko`.
Face detection model: `vendor/etc/eva/facedetection/model3.dat`.
Libraries: `libeva.so`, `libeva_util.so`.

### 6.4 VPU Firmware (Video Processing)

| File | Description |
|------|-------------|
| `vpu20_4v.mbn` | VPU 2.0 firmware (signed) |
| `vpu20_4v_unsigned.mbn` | VPU 2.0 firmware (unsigned) |

Video kernel module: `msm_video.ko`, clock control: `videocc-neo.ko`.

### 6.5 Rokid MCU / Audio Firmware (RT600)

| File | Description |
|------|-------------|
| `rt600_sbl.bin` | NXP RT600 MCU Secondary Boot Loader firmware |
| `sdk20-app-release-dvt-near-en.bin` | Near-field English firmware v5.1.0 (2025-10-25) |
| `sdk20-app-release-dvt-near-zh.bin` | Near-field Chinese firmware v3.3.0 |
| `sdk20-app-release-dvt-far-en.bin` | Far-field English firmware v3.0.4 (2025-10-25) |
| `sdk20-app-release-dvt-far-zh.bin` | Far-field Chinese firmware v3.0.4 |
| `sdk20-app-release-dvt-nr-en.bin` | Noise reduction English firmware v3.0.4 (2025-10-25) |
| `sdk20-app-release-dvt-nr-zh.bin` | Noise reduction Chinese firmware v3.0.4 |
| `sdk20-app-release-evt1-nr-zh.bin` | EVT1 board NR Chinese firmware |
| `sdk20-app-release-evt2-far-zh.bin` | EVT2 board far-field Chinese firmware |
| `sdk20-app-release-evt2-near-zh.bin` | EVT2 board near-field Chinese firmware |
| `sdk20-app-release-evt2-nr-zh.bin` | EVT2 board NR Chinese firmware |
| `filler_keywords_en.bin` / `filler_keywords_zh.bin` | Keyword filler models (EN/ZH) |
| `mlp_en.bin` / `mlp_zh.bin` | MLP (Multi-Layer Perceptron) models (EN/ZH) |

RT600 kernel driver: `rt600_spiboot.ko` (SPI boot).
Calibration data: `vendor/etc/CaliData_For_RokidGlasses.bin`.

### 6.6 WLAN Firmware

| Path | Description |
|------|-------------|
| `wlan/qca_cld/kiwi/` | QCA Kiwi WLAN config + MAC binary |
| `wlan/qca_cld/kiwi_v2/` | QCA Kiwi v2 WLAN config + MAC binary |
| `wlanmdsp.otaupdate` | WLAN DSP OTA update |

WLAN kernel modules: `qca_cld3_kiwi.ko`, `qca_cld3_kiwi_v2.ko`.
Config symlink: `WCNSS_qcom_cfg.ini` -> `/vendor/etc/wifi/WCNSS_qcom_cfg.ini`.
