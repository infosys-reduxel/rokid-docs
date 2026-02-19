# Init Services

### Rokid-Specific Services

| Service | Binary | Class | User | Description |
|---|---|---|---|---|
| `bootconfigs` | `/system/xbin/bootconfigs` | - (oneshot, disabled) | root | Rokid boot configuration setup, started during post-fs |
| `backup_keys` | `/system/xbin/backupkeys` | - (oneshot, disabled) | root | Backs up device keys, started on boot_completed |
| `GateServiced` | `/vendor/bin/GateServiced` | - (disabled) | root | Rokid gate/activation service, started on boot_completed |
| `sensor_bridge` | `/system/bin/sensor_bridge` | - | root | Bridges hardware sensor data between subsystems |

### Qualcomm Platform Services (vendor/etc/init/hw/)

| Service | Binary | Class | Description |
|---|---|---|---|
| `vendor.qrtr-ns` | `/vendor/bin/qrtr-ns` | core | Qualcomm IPC Router name service |
| `irsc_util` | `/vendor/bin/irsc_util` | core | Security config utility |
| `vendor.pd_mapper` | `/vendor/bin/pd-mapper` | core | Protection domain mapper for subsystem restart |
| `vendor.mdm_helper` | `/vendor/bin/mdm_helper` | core | Modem helper daemon |
| `vendor.ssr_setup` | `/system/vendor/bin/ssr_setup` | - (oneshot) | Subsystem restart setup |
| `vendor.ss_ramdump` | `/system/vendor/bin/subsystem_ramdump` | main | Subsystem ramdump collector |
| `qcomsysd` | `/system/vendor/bin/qcom-system-daemon` | main | Qualcomm system daemon (enabled via persist.vendor.qcomsysd.enabled) |
| `qcom-c_main-sh` | `/vendor/bin/init.class_main.sh` | main | Main class initialization script |
| `qcom-sh` | `/vendor/bin/init.qcom.sh` | late_start | Qualcomm init script |
| `qcom-post-boot` | `/vendor/bin/init.qcom.post_boot.sh` | late_start | Post-boot performance tuning, started on boot_completed |
| `kernel-boot` | `/vendor/bin/init.qti.kernel.sh` | core | Kernel boot-time configuration |
| `kernel-post-boot` | `/vendor/bin/init.kernel.post_boot.sh` | core | Kernel post-boot tuning (CPU governors, scheduler, etc.) |
| `vendor.modprobe` | `/vendor/bin/vendor_modprobe.sh` | main | Loads vendor kernel modules |
| `hibernate.resume` | `/vendor/bin/init.hibernate.resume.sh` | main | Hibernate resume handler |
| `vendor.msm_irqbalance` | `/vendor/bin/msm_irqbalance` | core | IRQ load balancer |

### Connectivity Services

| Service | Binary | Class | Description |
|---|---|---|---|
| `wpa_supplicant` | `/vendor/bin/hw/wpa_supplicant` | main | WiFi supplicant (disabled, started on demand) |
| `vendor.wigig_supplicant` | `/vendor/bin/hw/wpa_supplicant` | main | WiGig (60GHz) supplicant |
| `cnss-daemon` | `/system/vendor/bin/cnss-daemon` | late_start | CNSS (Connectivity Network Sub-System) daemon |
| `wifi_qos_daemon` | `/vendor/bin/wifi_qos_daemon` | late_start | WiFi QoS management |
| `wifi-crda` | `/vendor/bin/init.crda.sh` | late_start | Central Regulatory Domain Agent for WiFi |
| `hostapd_fst` | `/vendor/bin/hw/hostapd` | main | WiFi hotspot daemon |
| `vendor.atfwd` | `/vendor/bin/ATFWD-daemon` | late_start | AT command forwarding daemon |
| `ptt_socket_app` | `/system/vendor/bin/ptt_socket_app` | main | Production test tool socket app (WiFi) |
| `dhcpcd_wlan0` | `/system/bin/dhcpcd` | late_start | DHCP client for wlan0 |
| `ssgqmigd` | `/vendor/bin/ssgqmigd` | late_start | Secure services QMI gateway |
| `mlid` | `/vendor/bin/mlid` | late_start | Location ID service |
| `vendor.cnss_diag` | `/system/vendor/bin/cnss_diag` | main | CNSS diagnostics (Helium WiFi) |

### HAL Services (vendor/etc/init/)

| Service | Binary | Class | Description |
|---|---|---|---|
| `vendor.audio-hal` | `android.hardware.audio.service_64` | hal | 64-bit audio HAL service |
| `vendor.face-default` | `android.hardware.biometrics.face@1.0-service.face` | hal | Face biometrics HAL |
| `vendor.bluetooth` | `android.hardware.bluetooth@1.0-service-qti-lazy` | hal | Qualcomm Bluetooth HAL (lazy start) |
| `vendor.boot-hal` | `android.hardware.boot@1.2-service` | hal | Boot control HAL for A/B OTA |
| `vendor.cas-hal` | `android.hardware.cas@1.2-service-lazy` | hal | Conditional Access System HAL |
| `vendor.clearkey-hal` | `android.hardware.drm@1.4-service-lazy.clearkey` | hal | ClearKey DRM HAL |
| `vendor.gatekeeper` | `android.hardware.gatekeeper@1.0-service-qti` | hal | Qualcomm gatekeeper HAL |
| `vendor.health` | `android.hardware.health@2.1-service` | hal | Device health HAL |
| `vendor.keymaster` | `android.hardware.keymaster@3.0-service-qti` | hal | Qualcomm keymaster HAL |
| `vendor.keymint` | `android.hardware.security.keymint-service-qti` | hal | KeyMint HAL (newer keymaster) |
| `vendor.omx` | `android.hardware.media.omx@1.0-service` | hal | OpenMAX media HAL |
| `vendor.power` | `android.hardware.power-service` | hal | Power HAL |
| `vendor.sensors` | `android.hardware.sensors@2.1-service-multihal` | hal | Multi-HAL sensors service |
| `vendor.thermal` | `android.hardware.thermal@2.0-service.qti-v2` | hal | Qualcomm thermal HAL v2 |
| `vendor.usb-gadget` | `android.hardware.usb.gadget@1.2-service-qti` | hal | USB gadget HAL |
| `vendor.usb` | `android.hardware.usb@1.2-service-qti` | hal | USB HAL |
| `vendor.wifi-hal` | `android.hardware.wifi@1.0-service` | hal | WiFi HAL |
| `vendor.wifi-supplicant-hal` | `android.hardware.wifi.supplicant-service` | hal | WiFi supplicant AIDL HAL |
| `vendor.camera-provider-2-7` | `vendor.qti.camera.provider@2.7-service_64` | hal | Qualcomm camera provider HAL (v2.4-2.7) |
| `vendor.display-allocator` | `vendor.qti.hardware.display.allocator-service` | hal | Display buffer allocator (gralloc) |
| `vendor.display-composer` | `vendor.qti.hardware.display.composer-service` | hal | HWC (Hardware Composer) |
| `vendor.display-demura` | `vendor.qti.hardware.display.demura-service` | hal | Display demura (uniformity correction) |
| `vendor.lights` | `vendor.qti.hardware.lights.service` | hal | LED/lights HAL |
| `vendor.vibrator` | `vendor.qti.hardware.vibrator.service` | hal | Vibrator HAL |
| `vendor.perf-hal` | `vendor.qti.hardware.perf-hal-service` | hal | Qualcomm performance HAL |
| `vendor.dsp` | `vendor.qti.hardware.dsp@1.0-service` | hal | DSP service |
| `vendor.qseecom` | `vendor.qti.hardware.qseecom@1.0-service` | hal | Qualcomm Secure Execution Environment |
| `vendor.wifilearner` | `vendor.qti.hardware.wifi.wifilearner@1.0-service` | hal | WiFi learning/optimization HAL |
| `vendor.c2-media` | `vendor.qti.media.c2@1.0-service` | hal | Codec2 media service |
| `vendor.pasrmanager` | `vendor.qti.memory.pasrmanager@1.0-service` | hal | Partial Active/Self-Refresh memory manager |
| `vendor.psiclient` | `vendor.qti.psiclient@1.0-service` | hal | PSI (Pressure Stall Information) client |
| `vendor.AGMIPC` | `vendor.qti.hardware.AGMIPC@1.0-service` | hal | Audio-Graphics-Modem IPC |
| `vendor.alarm` | `vendor.qti.hardware.alarm@1.0-service` | hal | Alarm HAL |
| `vendor.limits` | `vendor.qti.hardware.limits-service` | hal | Hardware limits HAL |
| `powerstate` | `vendor.qti.hardware.power.powerstateservice@1.0-service` | hal | Power state tracking HAL |

### System Services (system/etc/init/)

| Service | Binary | Class | Description |
|---|---|---|---|
| `ueventd` | `/system/bin/ueventd` | core | Device node manager (critical) |
| `servicemanager` | `/system/bin/servicemanager` | core | Binder service manager |
| `hwservicemanager` | `/system/bin/hwservicemanager` | core | HIDL service manager |
| `surfaceflinger` | `/system/bin/surfaceflinger` | core animation | Display compositor, includes VR display sockets |
| `lmkd` | `/system/bin/lmkd` | core | Low Memory Killer daemon |
| `logd` | `/system/bin/logd` | core | Android log daemon |
| `vold` | `/system/bin/vold` | core | Volume daemon (storage management) |
| `netd` | `/system/bin/netd` | main | Network daemon |
| `zygote` | `/system/bin/app_process64` | main | App process launcher (64-bit primary) |
| `zygote_secondary` | `/system/bin/app_process32` | main | App process launcher (32-bit secondary) |
| `audioserver` | `/system/bin/audioserver` | core | Audio framework server |
| `cameraserver` | `/system/bin/cameraserver` | main | Camera framework server |
| `installd` | `/system/bin/installd` | main | Package installer daemon |
| `storaged` | `/system/bin/storaged` | main | Storage health monitoring |
| `tombstoned` | `/system/bin/tombstoned` | core | Crash dump collector |
| `statsd` | `/system/bin/statsd` | main | Statistics daemon |
| `update_engine` | `/system/bin/update_engine` | main | A/B OTA update engine |
| `apexd` | `/system/bin/apexd` | core | APEX module daemon |
| `charger` | `/system/bin/charger` | charger | Charger mode UI |
| `console` | `/system/bin/sh` | core | Debug console (only on debuggable builds) |

### Qualcomm-Specific Platform Services

| Service | Binary | Class | Description |
|---|---|---|---|
| `qseecomd` | per qseecomd.rc | core | Qualcomm Secure Execution Environment Communication daemon |
| `time_daemon` | `/vendor/bin/time_daemon` | main | Time synchronization daemon (started when file encryption is active) |
| `vendor.rmt_storage` | per vendor.qti.rmt_storage.rc | core | Remote storage service for modem |
| `vendor.tftp` | per vendor.qti.tftp.rc | core | TFTP service for firmware loading |
| `vendor.adsprpcd` | per vendor.qti.adsprpc-guestos-service.rc | - | ADSP RPC daemon |
| `vendor.audio-adsprpc` | per vendor.qti.audio-adsprpc-service.rc | - | Audio ADSP RPC service |
| `vendor.cdsprpc` | per vendor.qti.cdsprpc-service.rc | - | Compute DSP RPC daemon |
| `vendor.sensors.qti` | per vendor.sensors.qti.rc | - | Qualcomm sensors daemon |
| `vendor.sensors.sscrpcd` | per vendor.sensors.sscrpcd.rc | - | Sensor SLPI RPC daemon |
| `poweropt-service` | `/vendor/bin/poweropt-service` | hal | Qualcomm power optimization service |
| `qguard` | `/vendor/bin/qguard` | late_start | Qualcomm process guard (watchdog with KILL capability) |
| `vmmgr` | `/vendor/bin/vmmgr` | core | Virtual machine manager (disabled) |
| `vndservicemanager` | per vndservicemanager.rc | core | Vendor binder service manager |
| `vendor.ril-daemon` | `/vendor/bin/hw/rild` | main | Radio interface layer daemon |
| `qdcmss` | per qdcmss.rc | - | Display color management service |
| `init_thermal-engine-v2` | per init_thermal-engine-v2.rc | - | Thermal engine v2 |
| `vendor.wlan.lowirpcd` | per vendor.wlan.lowirpcd.rc | - | WLAN low-power IPC daemon |
| `vendor_flash_recovery` | per vendor_flash_recovery.rc | - | Recovery partition flasher |

### System Extension Services (system_ext/etc/init/)

| Service | Binary | Class | Description |
|---|---|---|---|
| `vendor.bt_logger` | `/system_ext/bin/bt_logger` | late_start | Bluetooth HCI logger (started via vendor.bluetooth.startbtlogger property) |
| `vendor_qspa` | `/system_ext/bin/init.qti.qspa.sh` | main | Qualcomm System Performance Advisor setup |
| `perfservice` | per perfservice.rc | - | Performance management service |

### Product Services (product/etc/init/)

| Service | Binary | Class | Description |
|---|---|---|---|
| `vendor_sys_qti_display` | `/product/bin/init.qti.display.sh` | main | Display initialization script (runs on post-fs-data) |

### Factory Test Services (init.qcom.factory.rc)

| Service | Binary | Description |
|---|---|---|
| `fastmmi` | `/system_ext/bin/mmi` | Factory MMI (Man-Machine Interface) test framework |
| `vendor.mmid` | `/vendor/bin/mmid` | Vendor MMI daemon |
| `mmi_diag` | `/system_ext/bin/mmi_diag` | MMI diagnostics service |
| `vendor.audio_tc*` | `/vendor/bin/mm-audio-ftm` | Audio factory test cases (multiple test case IDs) |
