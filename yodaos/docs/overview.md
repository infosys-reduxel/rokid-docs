# YodaOS Overview

YodaOS is Rokid's custom operating system for their AR smart glasses product line. It is built on top of Android 12 (API level 32) using the Qualcomm QSSI (Qualcomm Single System Image) architecture, with the Qualcomm "neo" board platform. The firmware is built using Qualcomm's "Go" (low-RAM) configuration, indicating the device is resource-constrained.

## Build Information

| Property | Value |
|---|---|
| Build fingerprint | `Rokid/glasses/glasses:12/SKQ1.240613.001/1.12.009-20260109-150201:user/release-keys` |
| Android version | 12 |
| SDK level | 32 |
| Build ID | SKQ1.240613.001 |
| Security patch | 2024-07-05 |
| Build date | Fri Jan 9 11:38:41 CST 2026 |
| Build type | user (production release) |
| Build flavor | qssi_lite-user |
| Build host | hzfd-xr-build04 |
| Build user | jenkins |
| Version incremental | 1.12.009-20260109-150201 |
| VNDK version | 32 |
| Treble enabled | true |
| A/B OTA | true (virtual A/B) |
| Low RAM mode | true |
| Board platform | neo |
| Product model | RG-glasses |
| Brand | Rokid |
| Manufacturer | Rokid (vendor) / QUALCOMM (system) |
| Default locale | zh-CN |
| Default timezone | Asia/Shanghai |
| OTA URL | https://ota.rokid.com |
| OTA check API | /v1/extended/ota/check |

## CPU and Architecture

| Property | Value |
|---|---|
| Primary ABI | arm64-v8a |
| Supported ABIs | arm64-v8a, armeabi-v7a, armeabi |
| CPU variant (64-bit) | kryo300 |
| CPU variant (32-bit) | cortex-a75 |
| Zygote | zygote64_32 (64-bit primary, 32-bit secondary) |
| SoC manufacturer | QTI (Qualcomm) |

## Graphics and Display

| Property | Value |
|---|---|
| OpenGL ES version | 3.2 (0x30002) |
| Vulkan HAL | adreno |
| EGL HAL | adreno |
| Color management | true |
| Wide color display | false |
| HDR display | false |
| Background blur | supported |
| Display rotation | ROTATION_LEFT (default) |

## Device Model Variants

The firmware supports multiple OEM product variants, differentiated at boot time by `ro.boot.devicetypeid` (a UUID burned into the device). Each variant receives a unique OEM ID, model designation, authentication keys, and display panel configuration.

The OEM ID scheme uses a numbering convention: Rokid-branded products use 10x series IDs, Bolon-branded products (co-branded line) use 20x series IDs.

| OEM ID | Model | Description | Has Display | Boot Animation |
|---|---|---|---|---|
| 101 | RV101 | Rokid Glasses (with display) | Yes | bootanimation_101.zip |
| 102 | RV102 | Rokid Glasses - Carrier edition (with display) | Yes | bootanimation_101.zip |
| 103 | RV203 | Rokid Ai Glasses (no display) | No | - |
| 104 | RV101 | Rokid Glasses - State Gift edition (with display) | Yes | bootanimation_104.zip |
| 105 | RV101 | Rokid Glasses - Overseas edition (with display) | Yes | bootanimation_101.zip |
| 106 | RV101 | Rokid Glasses - Leqi Smart Glasses (mainland, with display) | Yes | bootanimation_101.zip |
| 201 | RV201 | Bolon Ai Glasses (no display) | No | - |
| 202 | RV202 | Bolon Ai Glasses - Carrier edition (no display) | No | - |
| 203 | RV201 | Bolon Ai Glasses - Semi-transparent gray celebrity custom (no display) | No | - |

The `ro.boot.glassesWithPanel` and `ro.boot.Panel` properties indicate whether the variant includes a display panel (1 = yes, 0 = no). Each variant also receives unique `ro.boot.key` and `ro.boot.secret` values used for device authentication with Rokid cloud services.

The CXR (Communication eXperience Runtime) service configuration in `/system/etc/cxr-service.json` identifies two primary product lines for Bluetooth MFI (Made For iPhone) pairing:
- "Rokid Glasses" (model_id RV101)
- "Bolon Ai Glasses" (model_id RV201)

## Partition Layout

The firmware uses dynamic A/B partitions with virtual A/B support. The following partitions were extracted:

### system

The main Android system partition. Contains the AOSP framework, core system binaries, and standard Android apps. Located at `system/system/` (double-nested due to system-as-root).

Key contents:
- `build.prop` - Primary system build properties
- `app/` - Non-privileged system apps (Bluetooth, CXRService, CameraExtensionsProxy, etc.)
- `priv-app/` - Privileged system apps (RokidSysConfig, SettingsProvider, Shell, etc.)
- `bin/` - Core system binaries
- `framework/` - Android framework JARs
- `etc/` - System configuration, init RC files, SELinux policies
- `xbin/` - Extended system binaries (bootconfigs, backupkeys - Rokid-specific)
- `etc/cxr-service.json` - CXR Bluetooth communication service configuration
- `etc/ant_rokid_model_v1.3.bin` - Rokid AEC (Acoustic Echo Cancellation) model
- `etc/ant_rokid_record_model.bin` - Rokid audio recording model

### vendor

Hardware-specific partition containing Qualcomm BSP, HAL implementations, vendor-specific drivers, and Rokid customizations.

Key contents:
- `build.prop` - Vendor build properties including Rokid-specific OTA and product model settings
- `etc/init/hw/` - Vendor init RC files including `init.rokid.rc` and `init.rokid_oem_define.rc`
- `etc/init/` - HAL service RC files (camera, audio, Bluetooth, sensors, display, etc.)
- `bin/` - Vendor binaries (GateServiced, Qualcomm daemons, HAL implementations)
- `firmware/` - Device firmware blobs
- `lib/`, `lib64/` - Vendor shared libraries
- `dsp/` - DSP firmware
- `gpu/` - GPU firmware
- `overlay/` - Runtime resource overlays
- `rfs/` - Remote filesystem for modem
- `odm/`, `odm_dlkm/` - Nested ODM and ODM dynamic loadable kernel modules

### product

Product-specific partition for Rokid applications and customizations.

Key contents:
- `app/` - Product apps: Camera2, RokidOtaUpgrade, RokidScreenRecord, RokidSpriteAssistServer, RokidSpriteLauncher, Alipay, AntPay, JdPay, webview
- `priv-app/` - Privileged product apps: OneTimeInitializer, QtiSoundRecorder, SettingsIntelligence
- `media/` - Boot animations per OEM variant (bootanimation_101.zip, bootanimation_104.zip, etc.)
- `overlay/` - Product resource overlays

### odm

ODM (Original Design Manufacturer) partition. Minimal content in this firmware -- contains only build properties and a ueventd.rc file for device node permissions.

### system_ext

System extension partition for Qualcomm and carrier-specific additions.

Key contents:
- `app/`, `priv-app/` - Extended system apps
- `bin/` - Extended binaries (mmi, mmi_diag, bt_logger, init.qti.qspa.sh)
- `factoryV2/` - Factory test resources
- `etc/init/` - Init RC files for QTI Bluetooth logger, QSPA, and perfservice
- `etc/selinux/` - Extended SELinux policies (includes Rokid property context definitions)

### modem

Modem firmware partition. Contains raw firmware images and version information. Not a traditional filesystem -- holds baseband processor firmware blobs.

### vendor_dlkm

Vendor Dynamic Loadable Kernel Modules partition. Contains kernel modules (`lib/modules/`) and build properties. Used for loading vendor-specific kernel drivers at runtime without modifying the vendor partition.

## Key Customizations Over Stock Android

1. **OEM variant system**: Boot-time device differentiation via `ro.boot.devicetypeid` UUIDs, enabling a single firmware image to support multiple product SKUs with different display, authentication, and branding configurations.

2. **CXR Bluetooth stack**: Custom communication runtime (`cxr-service.json`) for bidirectional Bluetooth connectivity with companion phones, supporting both Android and iPhone (via MFI) with configurable MTU sizes and buffer configurations.

3. **RT600 co-processor**: SPI-connected RT600 co-processor for audio processing (AEC/codec), loaded at boot completion via `rt600_spiboot.ko` kernel module. Exposes firmware update and AEC control via sysfs.

4. **Sensor bridge**: Custom `sensor_bridge` service that bridges hardware sensor data within the glasses system.

5. **GateServiced**: Rokid gate service daemon that starts on boot completion -- likely manages the device unlock/activation gate.

6. **Camera LED indicator**: Hardware camera privacy indicator -- when camera session opens (`vendor.rkd.camera.session_open=1`), a white LED is illuminated; when closed, it turns off.

7. **Proximity and hall sensors**: Dedicated init property triggers for proximity sensor mode (standard/sensitive) and hall effect sensor management, used for wear detection.

8. **Slider input**: Hardware slider control with tuning support for up to 6 positions (0-5), exposed via init properties.

9. **Display brightness sync**: Brightness values are synced to XBL (eXtensible Boot Loader) via `vendor.rkd.display.brightness` for boot-stage display control.

10. **Low RAM optimizations**: Android Go defaults are applied (ro.config.low_ram=true), with aggressive memory management, smaller heap sizes, and dexopt tuning for constrained resources.

11. **Payment integrations**: Alipay, AntPay, and JdPay are pre-installed as product apps, indicating Chinese market payment functionality built into the glasses.

12. **Custom boot animations**: Per-OEM-variant boot animations stored in `/product/media/`, selected at boot time based on device type ID.

13. **Serial number and MFI setup**: Custom init commands `set_serialno` and `set_mfi` run during filesystem mount to configure device identity and MFI (Made For iPhone) authentication from persistent storage.

14. **Face biometrics HAL**: The device includes `android.hardware.biometrics.face@1.0-service.face`, indicating hardware face recognition capability at the system level.
