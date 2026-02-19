# Native Libraries

Source: `vendor/lib64/`, `vendor/lib64/hw/`, `vendor/lib64/egl/`, `vendor/lib64/camera/`.

### 7.1 Rokid-Specific Libraries

| Library | Description |
|---------|-------------|
| `libRokidAudioEffects.so` | Rokid custom audio effects (vendor/lib64 and vendor/lib) |

### 7.2 GPU / Graphics Libraries (Adreno)

| Library | Description |
|---------|-------------|
| `libEGL_adreno.so` | Adreno EGL implementation |
| `libGLESv2_adreno.so` | Adreno OpenGL ES 2.0/3.x |
| `libGLESv1_CM_adreno.so` | Adreno OpenGL ES 1.x |
| `libOpenCL_adreno.so` / `libOpenCL.so` | Adreno OpenCL |
| `libq3dtools_adreno.so` / `libq3dtools_esx.so` | Adreno 3D tools |
| `libadreno_utils.so` | Adreno utilities |
| `libadreno_app_profiles.so` | App-specific GPU profiles |
| `libadreno_compiler_cl.so` | OpenCL compiler |
| `libllvm-qcom.so` / `libllvm-glnext.so` / `libllvm-qgl.so` | LLVM backends for GPU shaders |
| `libgpudataproducer.so` | GPU data producer |
| `vulkan.adreno.so` | Vulkan HAL (in hw/) |
| `eglSubDriverAndroid.so` | EGL sub-driver |
| `libVkLayer_ADRENO_qprofiler.so` | Vulkan profiler layer |

### 7.3 Camera Libraries

**HAL implementations** (vendor/lib64/hw/):
- `camera.qcom.so` -- Qualcomm camera HAL
- `com.qti.chi.override.so` -- CHI (Camera Hardware Interface) override

**CamX framework** (vendor/lib64/):
- `camx.device@3.2-3.7-impl.so` -- Camera device implementations (v3.2-3.7)
- `camx.provider@2.4-2.7-*.so` -- Camera provider implementations
- `libcamxcommonutils.so` / `libcamximageformatutils.so` / `libcamxexternalformatutils.so`
- `libcamxfacialfeatures.so` / `libcamxfdalgo.so` / `libcamxfdengine.so` -- Face detection
- `libcamxifestriping.so` -- IFE striping
- `libcamxstatscore.so` -- Stats core
- `libcamxswispiqmodule.so` -- SW ISP IQ module
- `libcamxswprocessalgo.so` -- SW processing algorithm
- `libcamxtintlessalgo.so` -- Tintless correction algorithm
- `libcamxsettingsmanager.so` -- Settings manager
- `libcamxhwnodecontext.so` -- HW node context

**Camera sensor drivers** (vendor/lib64/camera/):
- `com.qti.sensor.imx681.so` -- Sony IMX681 sensor driver
- `com.qti.sensor.imx681_evk.so` -- IMX681 EVK variant
- `com.qti.sensor.imx681_evt.so` -- IMX681 EVT variant
- `com.qti.sensormodule.sunny_imx681.bin` -- Sunny optics module for IMX681
- `com.qti.sensormodule.sunny_imx681_evk.bin` / `_evt.bin` -- EVK/EVT variants
- `com.qti.tuned.sunny_imx681.bin` -- Tuning data for Sunny IMX681

**Camera ISP components** (vendor/lib64/camera/components/):
- `com.qti.eisv2.so` / `com.qti.eisv3.so` -- Electronic Image Stabilization v2/v3
- `com.qti.node.eisv2.so` / `com.qti.node.eisv3.so` -- EIS processing nodes
- `com.qti.node.gpu.so` -- GPU processing node
- `com.qti.node.swaidenoiser.so` -- SW AI denoiser
- `com.qti.node.swmfnr.so` -- SW Multi-Frame Noise Reduction
- `com.qti.node.swmctf.so` -- SW MCTF (Motion Compensated Temporal Filter)
- `com.qti.node.swregistration.so` -- SW registration (frame alignment)
- `com.qti.node.remosaic.so` -- Remosaic processing
- `com.qti.node.fcv.so` -- FastCV node
- `com.qti.node.formatconversion.so` -- Format conversion
- `com.qti.node.gme.so` -- Global Motion Estimation
- `com.qti.node.stich.so` -- Stitching node
- `com.qti.node.swlsc.so` -- SW Lens Shading Correction
- `com.qti.node.swbestats.so` -- SW Bayer exposure stats
- `com.qti.node.swpdpc.so` -- SW Pixel Defect Correction

**Camera 3A stats** (vendor/lib64/camera/components/):
- `com.qti.stats.aec.so` / `com.qtistatic.stats.aec.so` -- Auto Exposure Control
- `com.qti.stats.awb.so` / `com.qtistatic.stats.awb.so` -- Auto White Balance
- `com.qtistatic.stats.af.so` -- Auto Focus
- `com.qtistatic.stats.pdlib.so` -- Phase Detection library
- `com.qti.stats.afd.so` -- Anti-Flicker Detection
- `com.qti.stats.asd.so` -- Auto Scene Detection
- `com.qti.stats.aecwrapper.so` / `com.qti.stats.aecxcore.so` -- AEC wrappers
- `com.qti.stats.awbwrapper.so` -- AWB wrapper
- `com.qti.stats.statsgenerator.so` -- Stats generator
- `com.qti.stats.common.so` -- Common stats

**Feature2 (Chi Feature) pipeline** (vendor/lib64/):
- `com.qti.feature2.generic.so` -- Generic feature pipeline
- `com.qti.feature2.rt.so` -- Real-time pipeline
- `com.qti.feature2.anchorsync.so` -- Anchor frame sync
- `com.qti.feature2.derivedoffline.so` -- Derived offline processing
- `com.qti.feature2.gs.aurora.so` -- Aurora GS (graphics) feature
- `com.qti.feature2.mcreprocrt.so` -- Multi-camera reprocessing RT
- `com.qti.feature2.mfsr.aurora.so` -- Multi-Frame Super Resolution (Aurora)
- `com.qti.feature2.rawproc.so` -- RAW processing
- `com.qti.chiusecaseselector.so` -- Use case selector
- `com.qti.settings.aurora.so` -- Aurora settings

**IPE/BPS striping:**
- `libipebpsstriping.so` / `libipebpsstriping170.so` / `libipebpsstriping480.so` -- Image Processing Engine / Bayer Processing Segment striping

### 7.4 HAL Implementations (vendor/lib64/hw/)

| Library | HAL |
|---------|-----|
| `audio.primary.neo.so` | Primary audio HAL (Neo platform) |
| `audio.bluetooth.default.so` / `audio.bluetooth_qti.default.so` | Bluetooth audio HAL |
| `audio.r_submix.default.so` | Remote submix HAL |
| `audio.usb.default.so` | USB audio HAL |
| `camera.qcom.so` | Camera HAL |
| `com.qti.chi.override.so` | CHI override HAL |
| `sound_trigger.primary.neo.so` | Sound trigger HAL (Neo) |
| `vulkan.adreno.so` | Vulkan HAL |
| `android.hardware.graphics.mapper@4.0-impl-qti-display.so` | Graphics mapper HAL |
| `android.hardware.bluetooth@1.0-impl-qti.so` | Bluetooth HAL |
| `android.hardware.boot@1.0-impl-1.2-qti.so` | Boot HAL |
| `android.hardware.gatekeeper@1.0-impl-qti.so` | Gatekeeper HAL |
| `android.hardware.health@2.0-impl-2.1-qti.so` | Health HAL |
| `android.hardware.soundtrigger@2.2-impl.so` / `@2.3-impl.so` | Sound trigger HAL |
| `power.default.so` | Power HAL |
| `vibrator.default.so` | Vibrator HAL |
| `gralloc.default.so` | Gralloc (graphics alloc) HAL |
| `vendor.qti.hardware.bluetooth_audio@2.0-impl.so` / `@2.1-impl.so` | QTI BT audio |
| `vendor.qti.hardware.bluetooth_sar@1.1-impl.so` | BT SAR HAL |
| `vendor.qti.hardware.btconfigstore@1.0-impl.so` / `@2.0-impl.so` | BT config store |
| `vendor.qti.hardware.alarm@1.0-impl.so` | Alarm HAL |
| `vendor.qti.hardware.qseecom@1.0-impl.so` | QSEE HAL |
| `vendor.qti.hardware.wifidisplaysession@1.0-impl.so` | WiFi Display |
| `vendor.qti.memory.pasrmanager@1.0-impl.so` | PASR memory manager |

### 7.5 OpenCV and Computer Vision

- `libopencv.so` / `libopencv3a.so` -- OpenCV libraries
- `libfastcvopt.so` / `libfastcvdsp_stub.so` -- Qualcomm FastCV
- `libcv_common.so` -- Common CV utilities
- `libcamera_nn_stub.so` -- Camera neural network stub
- `libmfGhostDetection.so` -- Multi-frame ghost detection

### 7.6 XR / MCX Libraries

- `com.qualcomm.mcx.distortionmapper.so` -- MCX distortion mapper
- `com.qualcomm.mcx.linearmapper.so` -- MCX linear mapper
- `com.qualcomm.mcx.policy.mfl.so` -- MCX MFL policy
- `com.qualcomm.mcx.policy.xr.so` -- MCX XR policy
- `com.qualcomm.qti.mcx.usecase.extension.so` -- MCX use case extension
- `libmcs.so` -- Multi-Camera Sync

### 7.7 Key Kernel Modules (Hardware Drivers)

From `vendor_dlkm/lib/modules/`, categorized by subsystem:

**Display:**
- `msm_drm.ko` -- MSM DRM driver
- `msm_ext_display.ko` -- External display
- `hdmi_dlkm.ko` -- HDMI
- `panel-jbd-jbd4020-right.ko` -- JBD micro-LED panel

**GPU:**
- `msm_kgsl.ko` -- Kernel Graphics Support Layer
- `gpucc-neo.ko` -- GPU clock controller

**Camera/Video:**
- `camera.ko` -- Camera subsystem
- `camcc-neo.ko` -- Camera clock controller
- `msm_video.ko` -- Video codec
- `videocc-neo.ko` -- Video clock controller
- `msm-eva.ko` -- Enhanced Video Analytics
- `synx-driver.ko` -- Camera sync framework

**Sensors/Input:**
- `aw2110x.ko` -- AW2110x touch/capacitive sensor
- `psoc_ts_drv_right.ko` -- PSoC touch sensor (right)

**Power/Charging:**
- `mp2724_charger.ko` -- MP2724 charger IC
- `cw221x_battery.ko` -- CW221x battery fuel gauge
- `fan49103.ko` -- FAN49103 voltage regulator
- `power_state.ko` -- Power state management
- `pmic_glink.ko` -- PMIC GLink communication

**Connectivity:**
- `qca_cld3_kiwi.ko` / `qca_cld3_kiwi_v2.ko` -- QCA WLAN (Kiwi/Kiwi v2)
- `btpower.ko` -- Bluetooth power
- `bt_fm_slim.ko` -- BT/FM SLIMbus
- `cnss2.ko` -- Connectivity subsystem v2

**MCU:**
- `rt600_spiboot.ko` -- NXP RT600 SPI boot driver
