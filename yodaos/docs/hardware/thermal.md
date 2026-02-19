# Thermal Management

Source: `vendor/etc/thermal-engine.conf`, `vendor/etc/init/init_thermal-engine-v2.rc`, `vendor/etc/display/thermallevel_to_fps.xml`.

### 4.1 Thermal Engine

Binary: `vendor/bin/thermal-engine-v2` (Qualcomm thermal engine v2).
HAL: `android.hardware.thermal@2.0-service.qti-v2`.

Init config (`init_thermal-engine-v2.rc`):
```
service thermal-engine /vendor/bin/thermal-engine-v2
    class main
    user root
    group root
    socket thermal-send-client stream 0660 system oem_2907
    socket thermal-recv-client stream 0660 system oem_2907
    socket thermal-recv-passive-client stream 0660 system oem_2907
    socket thermal-send-rule stream 0660 system oem_2907
```

Restarts on `sys.boot_completed=1`.

The `thermal-engine.conf` file is **empty by default** -- thermal zones, trip points, and cooling device rules are likely compiled into the thermal engine binary or loaded from device tree rather than this file.

### 4.2 Thermal-Related Kernel Modules

From `vendor_dlkm/lib/modules/`:
- `qcom_tsens.ko` -- Qualcomm Temperature Sensor driver
- `thermal_pause.ko` -- Thermal pause support
- `qti_cpufreq_cdev.ko` -- CPU frequency cooling device
- `qti_devfreq_cdev.ko` -- Device frequency cooling device
- `qti_qmi_cdev.ko` -- QMI-based cooling device
- `qti_userspace_cdev.ko` -- Userspace cooling device
- `ddr_cdev.ko` -- DDR cooling device
- `cpu_hotplug.ko` -- CPU hotplug for thermal management

### 4.3 Thermal-Display Interaction

FPS throttling by thermal level (see section 3.4):
- Level 1-2: 144 Hz (no throttling)
- Level 3-4: 120 Hz
- Level 5-7: 90 Hz
- Level 8-10: 60 Hz (maximum throttling)

### 4.4 Thermal Client Library

`libthermalclient.so` available in both `vendor/lib64/` and `vendor/lib/` for applications to register thermal callbacks and query thermal state.
