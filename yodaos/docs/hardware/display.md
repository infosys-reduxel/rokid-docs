# Display

Source: `vendor/etc/display/`, kernel module `panel-jbd-jbd4020-right.ko`.

### 3.1 Display Panel

Kernel driver: `panel-jbd-jbd4020-right.ko` (JBD JBD4020 Micro-LED panel, right eye).

The JBD4020 is a micro-LED display panel manufactured by Jade Bird Display (JBD), designed for AR/MR headsets.

### 3.2 Display Processing Unit (DPU)

Multiple DPU configurations present for different Qualcomm SDE revisions:

| Config File | DPU Version |
|-------------|-------------|
| `DPU660.xml` | SDE 6.6.0 |
| `DPU670.xml` | SDE 6.7.0 |
| `DPU720.xml` | SDE 7.2.0 |
| `DPU7__.xml` | SDE 7.x (generic) |
| `DPU820.xml` | SDE 8.2.0 |
| `DPU830.xml` | SDE 8.3.0 |
| `DPU860.xml` | SDE 8.6.0 |
| `DPU8__.xml` | SDE 8.x (generic) |
| `DPU9__.xml` | SDE 9.x (generic) |

DPU820 (likely the active config for Neo platform) key parameters:
- Max line buffer width: 15360 pixels (Param29)
- Pipe count parameters and bandwidth limits defined

### 3.3 Panel Calibration Data

QDCM (Qualcomm Display Calibration Manager) calibration files present for multiple panel types:

| Panel | Type | DSC |
|-------|------|-----|
| Visionox R66451 AMOLED | cmd mode / video mode | with / without DSC |
| Novatek NT36672E LCD | video mode | with / without DSC |
| Sharp 1080p | cmd mode | -- |
| Sharp 2K | cmd / video mode | QSync |
| Sharp 4K | cmd / video mode | DSC |
| Sharp QHD | cmd mode | -- |

Backlight calibration for Visionox R66451:
- Gamma: 1.0
- Minimum backlight: 1200
- Max backlight: 4095
- Max OS backlight: 8191
- HBM (High Brightness Mode): not supported

### 3.4 Refresh Rate and Thermal Throttling

From `vendor/etc/display/thermallevel_to_fps.xml`:

| Thermal Level | Max FPS |
|---------------|---------|
| 1-2 | 144 |
| 3-4 | 120 |
| 5-7 | 90 |
| 8-10 | 60 |

Supported refresh rates (from SurfaceFlinger offset config): 90, 120, 144, 180 Hz.

Advanced SurfaceFlinger offsets per refresh rate:
| FPS | SF Offset % (v1) | SF Offset % (v2) | SF Offset % (v3) |
|-----|-------------------|-------------------|-------------------|
| 90 | 36% | 45% | 36% |
| 120 | 48% | 48% | 48% |
| 144 | 40% | 40% | 57% |
| 180 | -- | 40% | -- |

### 3.5 SmoMo (Smooth Motion)

Config: `vendor/etc/smomo_setting.xml`.

Game mode FPS mapping:
- 100-1000 app FPS mapped to 120 Hz refresh, QSync enabled, bypass mode
- 52-68 app FPS mapped to 60 app FPS at 120 Hz refresh, QSync enabled

### 3.6 Display Brightness Sync

From `init.rokid.rc`:
- Camera LED indicator: white LED at `/sys/class/leds/white/brightness` (255 when camera open, 0 when closed)
- Display brightness synced to XBL (eXtensible Boot Loader) via `vendor.rkd.display.brightness` property, written to `/mnt/vendor/logfs/panel_bright` and `/mnt/vendor/logfs/system_bright`

### 3.7 Display Libraries

Key display libraries in `vendor/lib64/`:
- `libsdmcore.so` -- SDE/SDM display core
- `libsdmextension.so` -- SDM extensions
- `libsdmutils.so` -- SDM utilities
- `libsdm-color.so` / `libsdm-colormgr-algo.so` -- color management
- `libsnapdragoncolor-manager.so` / `libsnapdragoncolor-qdcm.so` -- Snapdragon color
- `libdisplayconfig.qti.so` -- display configuration
- `libdisplaydebug.so` -- display debugging
- `libdisplayqos.so` -- display QoS
- `libdisplayskuutils.so` -- SKU-specific display utilities
- `libdpps.so` -- Display Post-Processing Service
- `libdisp-aba.so` -- Adaptive Backlight Algorithm
- `libdigital-dimming.so` -- digital dimming
- `libbacklight-calib.so` -- backlight calibration
- `libhistogram.so` -- display histogram
- `libhdr10plus.so` -- HDR10+ support
- `libhdr_backlight_adapter.so` / `libhdr_tm.so` -- HDR tone mapping
- `libgame_enhance.so` -- game display enhancement
- `libgpu_tonemapper.so` -- GPU-based tone mapping
- `libsdedrm.so` -- DRM (Direct Rendering Manager)

DRM kernel module: `msm_drm.ko`.
