# System Properties for Development

### Rokid-Specific Properties

Source: `vendor/build.prop` and `vendor/etc/init/hw/init.rokid.rc`

#### Build Identity

| Property | Value | Description |
|---|---|---|
| ro.product.vendor.brand | Rokid | Brand name |
| ro.product.vendor.device | glasses | Device codename |
| ro.product.vendor.model | RG-glasses | Device model |
| ro.rokid.product.model | RG-glasses | Rokid product model |
| ro.board.platform | neo | SoC platform codename |
| ro.product.board | neo | Board name |

#### OTA Configuration

| Property | Value |
|---|---|
| ro.rokid.ota.check_url | https://ota.rokid.com |
| ro.rokid.ota.check_api | /v1/extended/ota/check |

#### OEM Variant Properties (set at boot via init.rokid_oem_define.rc)

These are set dynamically based on `ro.boot.devicetypeid`:

| Property | Description |
|---|---|
| ro.product.rokid.oem.model | Hardware model identifier (RV101, RV102, RV201, RV202, RV203) |
| ro.product.rokid.oem.id | Numeric OEM ID (101-106 for Rokid, 201-203 for Bolon) |
| ro.boot.glassesWithPanel | 1 = has display panel, 0 = no display |
| ro.boot.Panel | Same as glassesWithPanel |
| ro.boot.key | Device authentication key (UUID) |
| ro.boot.secret | Device authentication secret (UUID) |
| persist.sys.customanim.boot | Path to custom boot animation ZIP |

**OEM Variants Table:**

| devicetypeid | OEM Model | OEM ID | Display | Description |
|---|---|---|---|---|
| B4BED4DBF1F94F88BF22F79A4D70A7C7 | RV101 | 101 | Yes | Rokid Glasses (standard) |
| FC9345A910B242C892B482F957543898 | RV102 | 102 | Yes | Rokid Glasses (carrier) |
| 983D4ED4D5BF47F3BCEF94DA550427C4 | RV203 | 103 | No | Rokid AI Glasses (no display) |
| 1FCC23EF2E6246BFABDD8B0065CBA8DE | RV101 | 104 | Yes | Rokid Glasses (national gift edition) |
| F5D9FC7C181C4F9CB7DCAD33FA4FE3B8 | RV101 | 105 | Yes | Rokid Glasses (overseas) |
| 696A1B92A9A0450B8D6FC13C65950FDA | RV101 | 106 | Yes | Rokid Glasses (Leqi branded, mainland China) |
| F817FC4A9BCF4D07B2F0B336A35F6EB5 | RV201 | 201 | No | Bolon AI Glasses |
| F9714DB407C14470B40A27D8883F164E | RV202 | 202 | No | Bolon AI Glasses (carrier) |
| E0BE10854F7042DD9CA0FC46AA62A447 | RV201 | 203 | No | Bolon AI Glasses (celebrity custom, translucent grey) |

#### Rokid Debug Properties (init.rokid.rc)

These properties are reactive -- setting them triggers hardware writes via sysfs at `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/`:

| Property | Values | Effect |
|---|---|---|
| rokid.debug.uart | 0, 1 | Enable/disable UART debug console |
| rokid.debug.enforce_hall | 0, 1 | Force hall sensor behavior |
| rokid.debug.enforce_psensor | 0, 1 | Force proximity sensor behavior |
| rokid.debug.proximity_tuning | 0, 1 | Enable/disable proximity sensor tuning mode |
| rokid.debug.slider_tuning | 0, 1 | Enable/disable slider tuning mode |
| rokid.debug.slider_nu | 0-5 | Set slider number index |
| rokid.psensor.mode | standard, sensitive | Proximity sensor sensitivity (writes thres_pct: 0 or 3) |

#### Rokid Vendor Properties (init.rokid.rc)

| Property | Description |
|---|---|
| vendor.rkd.camera.session_open | 0/1 -- triggers white LED on/off when camera opens/closes |
| vendor.rkd.display.brightness | Display brightness value, synced to logfs |
| vendor.rkd.display.brightness_xbl | XBL (bootloader) brightness value, synced to logfs |

### Build and Platform Properties

| Property | Value | Description |
|---|---|---|
| ro.build.version.sdk | 32 | Android SDK version |
| ro.build.version.release | 12 | Android release version |
| ro.build.version.security_patch | 2024-07-05 | Security patch date |
| ro.vendor.build.version.incremental | 1.12.009-20260109-150201 | Firmware version |
| ro.product.first_api_level | 31 | First shipped API level |
| ro.bionic.arch | arm64 | Primary CPU architecture |
| ro.bionic.cpu_variant | kryo300 | CPU variant |
| ro.bionic.2nd_arch | arm | Secondary architecture |
| ro.bionic.2nd_cpu_variant | cortex-a75 | Secondary CPU variant |
| ro.config.low_ram | true | Low-RAM device flag (Go edition optimizations) |
| ro.vndk.version | 32 | VNDK version |
| ro.build.characteristics | nosdcard | No SD card slot |
| ro.debuggable | 0 | Production build (not debuggable) |
| ro.adb.secure | 1 | ADB authentication required |
| ro.product.locale | zh-CN | Default locale |
| persist.sys.timezone | Asia/Shanghai | Default timezone |

### Graphics Properties

| Property | Value | Description |
|---|---|---|
| ro.opengles.version | 196610 | OpenGL ES 3.2 (0x30002) |
| ro.hardware.vulkan | adreno | Vulkan implementation |
| ro.hardware.egl | adreno | EGL implementation |
| graphics.gpu.profiler.support | true | GPU profiling enabled |
| ro.surface_flinger.use_color_management | true | Color management active |
| ro.surface_flinger.max_virtual_display_dimension | 4096 | Max virtual display size |
| ro.surface_flinger.supports_background_blur | 1 | Background blur supported |
| ro.surface_flinger.has_wide_color_display | false | No wide color gamut display |
| ro.surface_flinger.has_HDR_display | false | No HDR display |
| ro.minui.default_rotation | ROTATION_LEFT | Default display rotation |
| vendor.display.enable_display_extensions | 1 | Display extensions enabled |
| vendor.display.enable_rounded_corner | 1 | Rounded corners enabled |

### Camera Properties

| Property | Value | Description |
|---|---|---|
| ro.camera.enableCamera1MaxZsl | 1 | Camera1 API max ZSL enabled |
| ro.camera.tuningVersion | 4.1.2 | Camera tuning version |
| vendor.camera.aux.packagelist | org.codeaurora.snapcam | Packages allowed to use aux camera |
| persist.vendor.camera.privapp.list | org.codeaurora.snapcam | Camera privileged app list |
| vendor.debug.camera.3Adebug | (trigger) | Setting to 1 creates 3A debug capture file |

### Audio Properties

| Property | Value | Description |
|---|---|---|
| ro.vendor.audio.sdk.fluencetype | none | Fluence noise cancellation type |
| aaudio.mmap_policy | 2 | AAudio MMAP policy (exclusive) |
| aaudio.mmap_exclusive_policy | 2 | AAudio exclusive MMAP |
| af.fast_track_multiplier | 1 | Fast audio track multiplier |
| ro.audio.flinger_standbytime_ms | 2000 | AudioFlinger standby timeout |
| vendor.audio.hal.boot.timeout.ms | 20000 | Audio HAL boot timeout |
| persist.vendor.bt.a2dp_offload_cap | sbc-aptx-aptxtws-aptxhd-aac-ldac | BT A2DP offload codecs |
| ro.bluetooth.a2dp_offload.supported | true | A2DP hardware offload |
| vendor.audio.feature.concurrent_capture.enable | true | Concurrent audio capture |

### Bluetooth Properties

| Property | Value |
|---|---|
| ro.vendor.bt.bdaddr_path | /mnt/vendor/persist/quec_bt_mac |
| ro.bluetooth.library_name | libbluetooth_qti.so |
| persist.vendor.bluetooth.hfp_client | true |
| persist.vendor.bluetooth.split_a2dp_sink | true |
| bluetooth.profile.pbap.client.enabled | true |
| bluetooth.profile.a2dp.sink.enabled | true |
| bluetooth.profile.hfp.hf.enabled | true |
| bluetooth.profile.map.client.enabled | true |

### Debug and Performance Properties

| Property | Value | Description |
|---|---|---|
| debug.sf.hw | 0 | SurfaceFlinger HW composer debug |
| debug.egl.hw | 0 | EGL hardware debug |
| debug.sf.latch_unsignaled | 1 | Latch unsignaled buffers |
| debug.sf.enable_hwc_vds | 1 | HWC virtual display support |
| debug.sf.enable_advanced_sf_phase_offset | 1 | Advanced phase offset |
| debug.sf.enable_gl_backpressure | 1 | GL backpressure |
| debug.stagefright.ccodec | 4 | Codec2 preference level (vendor) |
| debug.stagefright.c2inputsurface | -1 | C2 input surface setting |
| debug.c2.use_dmabufheaps | 1 | DMA-buf heap usage for Codec2 |
| persist.debug.coresight.config | stm-events | CoreSight trace config |
| persist.debug.wfd.enable | 1 | WiFi Display enabled |
| ro.vendor.perf-hal.ver | 2.3 | Perf HAL version |
| ro.vendor.extension_library | libqti-perfd-client.so | Perf extension library |
