# Hardware Feature Flags

These features are declared via XML files in `vendor/etc/permissions/`, `system/system/etc/permissions/`, and `system_ext/etc/permissions/`. They determine what hardware capabilities `PackageManager.hasSystemFeature()` reports.

### Core Features (handheld_core_hardware.xml)

| Feature | Notes |
|---|---|
| android.hardware.audio.output | Audio output support |
| android.hardware.camera | Basic camera |
| android.hardware.location | Location services |
| android.hardware.location.network | Network-based location |
| android.hardware.sensor.compass | Magnetometer |
| android.hardware.sensor.accelerometer | Accelerometer |
| android.hardware.bluetooth | Bluetooth Classic |
| android.hardware.touchscreen | Touchscreen input |
| android.hardware.microphone | Microphone |
| android.hardware.screen.portrait | Portrait orientation |
| android.hardware.screen.landscape | Landscape orientation |
| android.hardware.security.model.compatible | Android security model compliance |
| android.software.connectionservice | ConnectionService telephony API |
| android.software.voice_recognizers | Voice recognizers (notLowRam) |
| android.software.home_screen | Home screen launcher |
| android.software.input_methods | Input method framework |
| android.software.picture_in_picture | PiP mode (notLowRam) |
| android.software.activities_on_secondary_displays | Multi-display activities (notLowRam) |
| android.software.companion_device_setup | Companion device pairing |
| android.software.cant_save_state | Heavy-weight process support |
| android.software.managed_users | Managed user profiles (notLowRam) |
| android.software.controls | Device controls API |

### Vendor-Declared Features (Individual XMLs)

| Feature | Source File |
|---|---|
| android.hardware.audio.low_latency | android.hardware.audio.low_latency.xml |
| android.hardware.audio.pro | android.hardware.audio.pro.xml |
| android.hardware.bluetooth | android.hardware.bluetooth.xml |
| android.hardware.bluetooth_le | android.hardware.bluetooth_le.xml |
| android.hardware.camera.any | android.hardware.camera.front.xml |
| android.hardware.camera.front | android.hardware.camera.front.xml |
| android.hardware.camera.concurrent | android.hardware.camera.concurrent.xml |
| android.hardware.camera.flash-autofocus | android.hardware.camera.flash-autofocus.xml |
| android.hardware.camera.full | android.hardware.camera.full.xml |
| android.hardware.camera.raw | android.hardware.camera.raw.xml |
| android.hardware.hardware_keystore (v100) | android.hardware.hardware_keystore.xml |
| android.hardware.keystore.app_attest_key | android.hardware.keystore.app_attest_key.xml |
| android.hardware.location.gps | android.hardware.location.gps.xml |
| android.hardware.opengles.aep | android.hardware.opengles.aep.xml |
| android.hardware.sensor.accelerometer | android.hardware.sensor.accelerometer.xml |
| android.hardware.sensor.compass | android.hardware.sensor.compass.xml |
| android.hardware.sensor.gyroscope | android.hardware.sensor.gyroscope.xml |
| android.hardware.sensor.light | android.hardware.sensor.light.xml |
| android.hardware.sensor.proximity | android.hardware.sensor.proximity.xml |
| android.hardware.touchscreen.multitouch.jazzhand | android.hardware.touchscreen.multitouch.jazzhand.xml |
| android.hardware.usb.accessory | android.hardware.usb.accessory.xml |
| android.hardware.usb.host | android.hardware.usb.host.xml |
| android.hardware.vulkan.compute-0 | android.hardware.vulkan.compute-0.xml |
| android.hardware.vulkan.level-1 | android.hardware.vulkan.level-1.xml |
| android.hardware.vulkan.version-1_1 | android.hardware.vulkan.version-1_1.xml |
| android.hardware.wifi | android.hardware.wifi.xml |
| android.hardware.wifi.aware | android.hardware.wifi.aware.xml |
| android.hardware.wifi.direct | android.hardware.wifi.direct.xml |
| android.hardware.wifi.passpoint | android.hardware.wifi.passpoint.xml |
| android.hardware.wifi.rtt | android.hardware.wifi.rtt.xml |
| android.software.device_id_attestation | android.software.device_id_attestation.xml |
| android.software.midi | android.software.midi.xml |
| android.software.opengles.deqp.level (v132449025 = 2021-03-01) | android.software.opengles.deqp.level.xml |
| android.software.sip.voip | android.software.sip.voip.xml |
| android.software.verified_boot | android.software.verified_boot.xml |
| android.software.vulkan.deqp.level (v132449025 = 2021-03-01) | android.software.vulkan.deqp.level.xml |
| android.software.webview | system: android.software.webview.xml |

### Features Summary by Category

**Camera**: camera, camera.any, camera.front, camera.full, camera.raw, camera.flash-autofocus, camera.concurrent

**Sensors**: sensor.accelerometer, sensor.compass, sensor.gyroscope, sensor.light, sensor.proximity

**Connectivity**: bluetooth, bluetooth_le, wifi, wifi.aware, wifi.direct, wifi.passpoint, wifi.rtt, location, location.network, location.gps

**Graphics**: opengles.aep (OpenGL ES 3.2 = 196610), vulkan.compute-0, vulkan.level-1, vulkan.version-1_1, Adreno GPU

**Audio**: audio.output, audio.low_latency, audio.pro, microphone, software.midi

**Security**: hardware_keystore (v100), keystore.app_attest_key, device_id_attestation, verified_boot

**Input**: touchscreen, touchscreen.multitouch.jazzhand (5+ pointers)

**Notable absences**: No `android.hardware.telephony` feature flag found despite telephony config being present in build.prop. No `android.hardware.nfc` feature. No `android.hardware.screen.landscape.only` (both portrait and landscape are declared). No `android.software.app_widgets` (commented out in handheld_core_hardware.xml). No `android.software.secure_lock_screen` (commented out).
