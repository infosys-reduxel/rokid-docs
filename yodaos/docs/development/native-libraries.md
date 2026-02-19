# Native Library Catalog

### Rokid-Specific Libraries

| Library | Path | Description |
|---|---|---|
| libRokidAudioEffects.so | vendor/lib64/ (or lib/) | Rokid custom audio effects processing |

Note: Unlike many OEM devices, this firmware contains very few Rokid-branded native libraries. Most hardware interaction happens through Qualcomm HALs and the I2C sysfs interface at `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/`.

### Camera Libraries (Qualcomm CamX)

| Library | Description |
|---|---|
| camera.qcom.so | Qualcomm camera HAL implementation |
| libcamerapostproc.so | Camera post-processing service |
| libcamera2ndk_vendor.so | Camera2 NDK vendor interface |
| libcamera_nn_stub.so / libcamera_nn_skel.so | Camera neural network stubs (DSP offload) |
| camx.device@3.7-impl.so | CamX device HAL v3.7 |
| camx.provider@2.7-legacy.so | CamX camera provider v2.7 |
| libcamxfdalgo.so | CamX face detection algorithm |
| libcamxfacialfeatures.so | CamX facial features detection |
| libcamxfdengine.so | CamX face detection engine |
| libcamxstatscore.so | CamX 3A stats core |
| com.qti.sensor.imx681.so | IMX681 camera sensor driver |
| com.qti.sensor.imx681_evk.so | IMX681 EVK variant |
| com.qti.sensor.imx681_evt.so | IMX681 EVT variant |

### GPU / Graphics Libraries

| Library | Description |
|---|---|
| vulkan.adreno.so | Vulkan ICD (Adreno) |
| libEGL_adreno.so | EGL implementation |
| libGLESv1_CM_adreno.so | OpenGL ES 1.x |
| libGLESv2_adreno.so | OpenGL ES 2.0/3.x |
| libOpenCL_adreno.so / libOpenCL.so | OpenCL compute |
| libadreno_utils.so | Adreno utility functions |
| libadreno_compiler_cl.so | Adreno CL shader compiler |
| libllvm-qcom.so | LLVM backend for Adreno |

### DSP Libraries (Hexagon)

| Library | Description |
|---|---|
| libadsprpc.so | ADSP FastRPC client |
| libcdsprpc.so | CDSP FastRPC client |
| libsdsprpc.so | SDSP FastRPC client |
| libmdsprpc.so | MDSP FastRPC client |
| libfastcvopt.so | FastCV optimized library |
| libfastcvadsp.so | FastCV ADSP offload |
| libmctfengine_skel.so / libmctfengine_stub.so | MCTF engine (DSP) |

### Bluetooth Libraries

| Library | Description |
|---|---|
| android.hardware.bluetooth@1.0-impl-qti.so | BT HAL implementation |
| audio.bluetooth_qti.default.so | QTI BT audio |
| lib_bt_aptx.so | aptX codec |
| libbtnv.so | BT NV items |
| libbt-hidlclient.so | BT HIDL client |
| libbtsinkpal.so | BT sink PAL |
| libbthost_if_sink.so | BT host sink interface |

### Sensor Libraries

| Library | Description |
|---|---|
| sensors.ssc.so | Sensors SSC HAL |
| libsnsapi.so / libsnsapi-full.so | Sensor API client |
| libsensorslog.so | Sensor logging |
| libssc.so | Sensor Service Core |

### Biometrics

| Library | Description |
|---|---|
| android.hardware.biometrics.face-V1-ndk_platform.so | Face biometric HAL AIDL binding |
| android.hardware.biometrics.common-V1-ndk_platform.so | Common biometric types |

The face biometric HAL service runs as `android.hardware.biometrics.face@1.0-service.face` under user `system` in group `camera system`.

### Performance

| Library | Description |
|---|---|
| libqti-perfd.so | QTI performance daemon library |
| libqti-perfd-client.so | Performance daemon client (ro.vendor.extension_library) |
| libperfconfig.so | Performance configuration |
| libperfgluelayer.so | Performance glue layer |
| libperfioctl.so | Performance ioctl interface |
