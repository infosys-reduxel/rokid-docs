# YodaOS Kernel Modules Documentation

Extracted from decompiled Rokid AR glasses firmware (Android 12 / YodaOS).

Source partition: `vendor_dlkm/lib/modules/`

Platform: Qualcomm Neo (QCS6490-class), ARM64

---

## Summary

155 kernel module entries in `modules.load`, covering 137 unique `.ko` files. Total loadable modules are organized by function below. The `modules.blocklist` explicitly prevents loading of 68 modules (mostly tuner drivers, test modules, and `rt600_spiboot.ko`).

Notable: `rt600_spiboot.ko` is blocklisted in the default load list, meaning the RT600 speech co-processor SPI boot is handled through an alternative mechanism or loaded conditionally.

---

## Display / GPU Drivers

| Module | Size | Description |
|---|---|---|
| `msm_drm.ko` | 4,366,320 (4.16 MB) | Qualcomm MSM Display/DRM driver. Main display subsystem driver for rendering pipeline, display output, and composition. Depends on `panel-jbd-jbd4020-right.ko`. |
| `msm_kgsl.ko` | 2,769,144 (2.64 MB) | Qualcomm Kernel Graphics Support Layer. GPU driver providing userspace interface to Adreno GPU hardware. |
| `msm_ext_display.ko` | 26,208 (25.6 KB) | External display framework module. Manages display outputs beyond the primary panel (HDMI, DP). |
| `hdmi_dlkm.ko` | 37,216 (36.3 KB) | HDMI audio/video output driver. |
| `panel-jbd-jbd4020-right.ko` | 45,800 (44.7 KB) | JBD JBD4020 micro-LED display panel driver (right eye). JBD is the micro-LED supplier for Rokid AR glasses. |
| `gpucc-neo.ko` | 34,568 (33.8 KB) | GPU clock controller for Neo platform. Manages Adreno GPU clock frequencies and power states. |
| `camcc-neo.ko` | 140,984 (137.7 KB) | Camera clock controller for Neo platform. |
| `videocc-neo.ko` | 33,976 (33.2 KB) | Video clock controller for Neo platform. Manages video encoder/decoder clocks. |
| `debugcc-neo.ko` | 79,472 (77.6 KB) | Debug clock controller for Neo platform. Provides clock measurement and debug interfaces. |

---

## Camera Drivers

| Module | Size | Description |
|---|---|---|
| `camera.ko` | 6,078,112 (5.80 MB) | Qualcomm camera subsystem driver. Largest module -- handles ISP, sensor interfaces, and camera pipeline. Depends on `synx-driver.ko` for synchronization. |
| `msm-eva.ko` | 930,376 (908.6 KB) | Qualcomm Enterprise Video Analytics. Compute Vision Accelerator (CVA/EVA) driver for on-device computer vision processing. |
| `synx-driver.ko` | 138,432 (135.2 KB) | Camera synchronization framework. Coordinates timing between camera, display, and compute pipelines. |
| `msm_video.ko` | 1,156,752 (1.10 MB) | Video codec (encoder/decoder) driver. Venus video hardware accelerator interface. |
| `msm-mmrm.ko` | 95,360 (93.1 KB) | Multimedia Resource Manager. Manages shared multimedia hardware resources (bandwidth, clock) across camera, video, and display. |
| `mmrm_test_module.ko` | 131,936 (128.8 KB) | MMRM test module. Blocklisted in production. |

---

## Audio Drivers

| Module | Size | Description |
|---|---|---|
| `machine_dlkm.ko` | 143,080 (139.7 KB) | Audio machine driver. Top-level ALSA sound card driver that ties together codec, platform, and DAI link configurations. |
| `lpass_cdc_dlkm.ko` | 117,680 (114.9 KB) | Low Power Audio Subsystem (LPASS) codec driver core. Base driver for the integrated audio codec. |
| `lpass_cdc_rx_macro_dlkm.ko` | 231,664 (226.2 KB) | LPASS RX (playback) macro driver. Handles audio output path digital processing. |
| `lpass_cdc_tx_macro_dlkm.ko` | 148,160 (144.7 KB) | LPASS TX (capture) macro driver. Handles microphone input path digital processing. |
| `lpass_cdc_va_macro_dlkm.ko` | 121,776 (118.9 KB) | LPASS Voice Activation macro driver. Handles always-on low-power voice detection hardware path. |
| `lpass_cdc_wsa_macro_dlkm.ko` | 140,848 (137.5 KB) | LPASS WSA (Wall Speaker Amplifier) macro driver. |
| `lpass_cdc_wsa2_macro_dlkm.ko` | 140,976 (137.7 KB) | LPASS WSA2 macro driver. Second WSA instance. |
| `wcd938x_dlkm.ko` | 248,816 (243.0 KB) | WCD9380/9385 audio codec driver. Qualcomm premium audio codec with multi-channel support. |
| `wcd938x_slave_dlkm.ko` | 27,096 (26.5 KB) | WCD938x SoundWire slave interface driver. |
| `wcd937x_dlkm.ko` | 190,760 (186.3 KB) | WCD9370/9375 audio codec driver. Qualcomm mid-tier audio codec. |
| `wcd937x_slave_dlkm.ko` | 14,584 (14.2 KB) | WCD937x SoundWire slave interface driver. |
| `wcd9xxx_dlkm.ko` | 58,968 (57.6 KB) | WCD9xxx common codec utilities. Shared code for WCD audio codec family. |
| `wcd_core_dlkm.ko` | 62,104 (60.6 KB) | WCD codec core framework. Base registration and regmap infrastructure for WCD codecs. |
| `mbhc_dlkm.ko` | 88,776 (86.7 KB) | Mechanical Button and Headset Controller. Detects headset insertion, button presses, and impedance. |
| `wsa883x_dlkm.ko` | 103,520 (101.1 KB) | WSA8830/8835 smart speaker amplifier driver. Class-D speaker amplifier with integrated DSP. |
| `swr_dlkm.ko` | 54,296 (53.0 KB) | SoundWire bus driver. Core bus infrastructure for Qualcomm SoundWire audio interconnect. |
| `swr_ctrl_dlkm.ko` | 123,960 (121.1 KB) | SoundWire controller driver. Master controller for SoundWire bus. |
| `swr_dmic_dlkm.ko` | 37,120 (36.3 KB) | SoundWire digital microphone driver. Interfaces DMIC arrays over SoundWire. |
| `swr_haptics_dlkm.ko` | 34,872 (34.1 KB) | SoundWire haptics driver. Drives haptic actuators over SoundWire bus. |
| `pinctrl_lpi_dlkm.ko` | 43,432 (42.4 KB) | Low Power Island pin controller for audio. Manages GPIO/pin mux for audio subsystem. |
| `snd_event_dlkm.ko` | 21,904 (21.4 KB) | Sound event notification driver. Coordinates audio subsystem state machine events. |
| `stub_dlkm.ko` | 11,864 (11.6 KB) | Audio stub/dummy codec driver. Placeholder codec used for testing or virtual DAI links. |
| `spf_core_dlkm.ko` | 33,648 (32.9 KB) | Sensor Processing Framework core. Audio DSP framework driver for ADSP communication. |
| `gpr_dlkm.ko` | 37,392 (36.5 KB) | Generic Packet Router for audio. Routes audio packets between kernel and ADSP. |
| `audio_pkt_dlkm.ko` | 40,408 (39.5 KB) | Audio packet driver. Manages audio data packet transport. |
| `audio_prm_dlkm.ko` | 24,264 (23.7 KB) | Audio Parameter Manager. Handles audio DSP parameter configuration. |
| `audpkt_ion_dlkm.ko` | 33,264 (32.5 KB) | Audio packet ION memory driver. Manages shared memory buffers between kernel and ADSP for audio data. |
| `q6_dlkm.ko` | 16,096 (15.7 KB) | Hexagon Q6 DSP audio interface driver. |
| `q6_notifier_dlkm.ko` | 33,512 (32.7 KB) | Q6 DSP notifier driver. Handles ADSP subsystem state change notifications (up/down/restart). |
| `q6_pdr_dlkm.ko` | 12,616 (12.3 KB) | Q6 Protection Domain Restart driver. Manages audio DSP protection domain lifecycle. |
| `adsp_loader_dlkm.ko` | 23,088 (22.5 KB) | ADSP firmware loader. Triggers loading of Audio DSP firmware image. |
| `slimbus.ko` | 56,744 (55.4 KB) | SLIMbus (Serial Low-power Inter-chip Media Bus) core driver. Legacy audio interconnect bus. |
| `slim-qcom-ngd-ctrl.ko` | 118,808 (116.0 KB) | Qualcomm SLIMbus NGD (Non-Generic Device) controller. SLIMbus master driver. |
| `bt_fm_slim.ko` | 47,880 (46.8 KB) | Bluetooth/FM over SLIMbus driver. Audio transport for BT and FM radio over SLIMbus. |

---

## Sensor Drivers

| Module | Size | Description |
|---|---|---|
| `psoc_ts_drv_right.ko` | 156,032 (152.4 KB) | PSoC touchpad/sensor driver (right side). Cypress/Infineon PSoC-based capacitive touch sensor on the right temple of the glasses. |
| `aw2110x.ko` | 66,776 (65.2 KB) | AWINIC AW2110x proximity/light sensor driver. Optical sensor for wear detection or ambient light sensing. |
| `qcom_tsens.ko` | 85,472 (83.5 KB) | Qualcomm thermal sensor driver. Reads on-die temperature sensors for thermal management. |

---

## Network / WiFi / Bluetooth

| Module | Size | Description |
|---|---|---|
| `qca_cld3_kiwi.ko` | 18,226,944 (17.38 MB) | Qualcomm WiFi driver (QCA CLD 3.0) for Kiwi chipset. Full WiFi stack -- the largest module in the system. |
| `qca_cld3_kiwi_v2.ko` | 18,332,616 (17.48 MB) | Qualcomm WiFi driver (QCA CLD 3.0) for Kiwi v2 chipset revision. |
| `cnss2.ko` | 950,616 (928.3 KB) | Connectivity Subsystem driver v2. Platform driver managing WiFi chip power, firmware download, and PCIe link. |
| `cnss_utils.ko` | 33,440 (32.7 KB) | CNSS utility functions. Shared helpers for connectivity subsystem. |
| `cnss_nl.ko` | 16,264 (15.9 KB) | CNSS netlink interface. Userspace communication channel for connectivity management. |
| `cnss_prealloc.ko` | 18,552 (18.1 KB) | CNSS memory pre-allocation. Pre-allocates memory pools for WiFi driver to avoid allocation failures. |
| `cnss_plat_ipc_qmi_svc.ko` | 58,160 (56.8 KB) | CNSS platform IPC QMI service. QMI-based inter-processor communication for connectivity. |
| `wlan_firmware_service.ko` | 83,568 (81.6 KB) | WLAN firmware service driver. Manages WiFi firmware loading and communication. |
| `icnss2.ko` | 422,824 (412.9 KB) | Integrated Connectivity Subsystem driver v2. Alternative connectivity platform driver (possibly for integrated WiFi). |
| `btpower.ko` | 57,760 (56.4 KB) | Bluetooth power management driver. Controls BT chip power sequencing and regulators. |

---

## USB / Peripheral

| Module | Size | Description |
|---|---|---|
| `phy-msm-snps-eusb2.ko` | 62,016 (60.6 KB) | Synopsys eUSB2 PHY driver. USB 2.0 physical layer for enhanced USB. |
| `phy-msm-ssusb-qmp.ko` | 33,928 (33.1 KB) | QMP SuperSpeed USB PHY driver. USB 3.x physical layer. |
| `usb_f_diag.ko` | 47,536 (46.4 KB) | USB function driver for diagnostics (DIAG). Exposes Qualcomm DIAG interface over USB. |
| `usb_f_qdss.ko` | 53,312 (52.1 KB) | USB function driver for QDSS (Qualcomm Debug Subsystem). Debug trace data over USB. |
| `repeater.ko` | 16,568 (16.2 KB) | USB signal repeater framework. |
| `repeater-i2c-eusb2.ko` | 20,328 (19.9 KB) | eUSB2 I2C repeater driver. Signal redriver for USB 2.0 over I2C-controlled repeater chip. |
| `eud.ko` | 34,192 (33.4 KB) | Embedded USB Debugger. Qualcomm EUD for debug access over USB. |

---

## Inter-Processor Communication (IPC)

| Module | Size | Description |
|---|---|---|
| `qcom_glink.ko` | 97,232 (94.9 KB) | Qualcomm G-Link transport core. Base IPC framework for communication between processors (apps, modem, ADSP, CDSP). |
| `qcom_glink_smem.ko` | 20,112 (19.6 KB) | G-Link shared memory transport. G-Link backend using shared memory regions. |
| `glink_pkt.ko` | 65,632 (64.1 KB) | G-Link packet driver. Provides character device interface for G-Link channels. |
| `glink_probe.ko` | 23,080 (22.5 KB) | G-Link probe driver. Handles discovery and enumeration of G-Link channels. |
| `qcom_smd.ko` | 44,816 (43.8 KB) | Qualcomm Shared Memory Driver. Legacy IPC mechanism using shared memory. |
| `smp2p.ko` | 34,848 (34.0 KB) | Shared Memory Point-to-Point. Provides state notification between processors via shared memory bits. |
| `smp2p_sleepstate.ko` | 14,448 (14.1 KB) | SMP2P sleep state driver. Communicates processor sleep states between subsystems. |
| `qcom_ipc_lite.ko` | 15,480 (15.1 KB) | Lightweight IPC driver. Simplified inter-processor communication interface. |
| `ipclite.ko` | 124,392 (121.5 KB) | IPC Lite core driver. Full implementation of lightweight IPC framework. |
| `pdr_interface.ko` | 30,760 (30.0 KB) | Protection Domain Restart interface. Manages subsystem restart notifications and coordination. |
| `rproc_qcom_common.ko` | 68,176 (66.6 KB) | Qualcomm remote processor common driver. Shared infrastructure for loading and managing remote processors (ADSP, CDSP, modem). |
| `qcom_q6v5.ko` | 26,632 (26.0 KB) | Hexagon Q6 v5 remoteproc driver base. Core driver for Hexagon DSP processor management. |
| `qcom_q6v5_pas.ko` | 82,736 (80.8 KB) | Q6v5 PAS (Peripheral Authentication Service) remoteproc driver. Loads and authenticates firmware for Hexagon DSP. |
| `qcom_sysmon.ko` | 44,360 (43.3 KB) | Qualcomm subsystem monitor. Monitors health and state of remote processors. |
| `pmic_glink.ko` | 39,408 (38.5 KB) | PMIC G-Link driver. IPC channel for PMIC (power management IC) communication with co-processors. |
| `qrtr-smd.ko` | 16,632 (16.2 KB) | QMI Router over SMD transport. Routes QMI messages over Shared Memory Device links. |
| `qrtr-mhi.ko` | 18,608 (18.2 KB) | QMI Router over MHI transport. Routes QMI messages over Modem Host Interface (PCIe). |

---

## PCIe / MHI (Modem Host Interface)

| Module | Size | Description |
|---|---|---|
| `pci-msm-drv.ko` | 480,088 (468.8 KB) | Qualcomm MSM PCIe root complex driver. Manages PCIe bus for external connectivity devices. |
| `mhi.ko` | 243,064 (237.4 KB) | Modem Host Interface core driver. MHI protocol stack for communication over PCIe with modem/WiFi chipsets. |
| `mhi_dev_uci.ko` | 47,104 (46.0 KB) | MHI device UCI (User Channel Interface). Provides userspace access to MHI channels. |
| `mhi_dev_netdev.ko` | 54,600 (53.3 KB) | MHI device network driver. Network interface for data transfer over MHI/PCIe. |
| `mhi_dev_dtr.ko` | 22,824 (22.3 KB) | MHI device DTR (Data Terminal Ready) signal driver. |

---

## SPI / I2C Bus Drivers

| Module | Size | Description |
|---|---|---|
| `spi-msm-geni.ko` | 99,152 (96.8 KB) | Qualcomm GENI-based SPI controller driver. |
| `spidev.ko` | 65,968 (64.4 KB) | Generic SPI device driver. Provides userspace SPI access via `/dev/spidevX.Y`. |
| `i2c-msm-geni.ko` | 112,504 (109.9 KB) | Qualcomm GENI-based I2C controller driver. |
| `qcom-i2c-pmic.ko` | 36,624 (35.8 KB) | I2C interface for Qualcomm PMIC. Accesses power management IC registers over I2C. |
| `gpi.ko` | 106,712 (104.2 KB) | Generic Peripheral Interface DMA engine driver. DMA engine for GENI-based peripherals (SPI, I2C, UART). |
| `bam_dma.ko` | 45,928 (44.9 KB) | Bus Access Manager DMA engine driver. Legacy DMA engine for BAM-based peripherals. |

---

## DMA / Bus Performance

| Module | Size | Description |
|---|---|---|
| `sps_drv.ko` | 491,112 (479.6 KB) | Smart Peripheral System driver. BAM (Bus Access Manager) pipe management infrastructure. |
| `bwmon.ko` | 77,040 (75.2 KB) | Bandwidth monitor driver. Monitors memory bus bandwidth usage for DVFS decisions. |
| `bwprof.ko` | 28,904 (28.2 KB) | Bandwidth profiler. Provides bandwidth utilization statistics. |
| `llcc_perfmon.ko` | 64,336 (62.8 KB) | Last Level Cache Controller performance monitor. Monitors LLCC hit/miss rates. Blocklisted by default. |
| `memlat.ko` | 84,952 (83.0 KB) | Memory latency monitor. Tracks memory access latency for governor decisions. |

---

## Power / Thermal Management

| Module | Size | Description |
|---|---|---|
| `qcom_lpm.ko` | 102,128 (99.7 KB) | Low Power Mode driver. Manages CPU cluster and system-wide low power states (C-states). |
| `qcom_aoss.ko` | 45,680 (44.6 KB) | Always-On Subsystem driver. Interface to the always-on processor that manages system-wide power state. |
| `power_state.ko` | 32,760 (32.0 KB) | Power state driver. Tracks and reports processor power state transitions. |
| `msm_performance.ko` | 87,616 (85.6 KB) | MSM performance driver. CPU frequency and scheduling optimization hooks. |
| `cpu_hotplug.ko` | 20,552 (20.1 KB) | CPU hotplug driver. Manages CPU core online/offline for power savings. |
| `thermal_pause.ko` | 28,320 (27.7 KB) | Thermal pause driver. Pauses CPU cores when thermal limits are reached. |
| `qti_cpufreq_cdev.ko` | 20,336 (19.9 KB) | CPU frequency cooling device. Throttles CPU frequency for thermal management. |
| `qti_devfreq_cdev.ko` | 17,712 (17.3 KB) | Device frequency cooling device. Throttles device frequencies (GPU, bus) for thermal management. |
| `qti_qmi_cdev.ko` | 30,888 (30.2 KB) | QMI cooling device. Sends thermal mitigation commands to remote processors via QMI. |
| `qti_userspace_cdev.ko` | 15,368 (15.0 KB) | Userspace cooling device. Allows userspace thermal management policies. |
| `ddr_cdev.ko` | 15,688 (15.3 KB) | DDR cooling device. Throttles memory frequency for thermal management. |
| `fan49103.ko` | 19,432 (19.0 KB) | Fairchild/ON Semi FAN49103 voltage regulator driver. Buck converter for dynamic voltage scaling. |
| `mp2724_charger.ko` | 36,248 (35.4 KB) | MPS MP2724 battery charger driver. Manages battery charging for the glasses. |
| `cw221x_battery.ko` | 30,464 (29.8 KB) | CellWise CW2215/CW2217 battery fuel gauge driver. Reports battery capacity, voltage, and health. |

---

## Sleep / Stats

| Module | Size | Description |
|---|---|---|
| `soc_sleep_stats.ko` | 35,824 (35.0 KB) | SoC sleep statistics driver. Tracks time spent in various SoC sleep states. |
| `subsystem_sleep_stats.ko` | 30,520 (29.8 KB) | Subsystem sleep statistics. Tracks sleep durations for individual subsystems (modem, ADSP, CDSP). |
| `sysmon_subsystem_stats.ko` | 38,512 (37.6 KB) | System monitor subsystem statistics. Aggregates subsystem health and performance data. |
| `sys_pm_vx.ko` | 35,680 (34.8 KB) | System power management statistics driver. Exposes detailed PM statistics. |
| `qcom_cpuss_sleep_stats.ko` | 26,040 (25.4 KB) | CPU subsystem sleep statistics. Tracks CPU cluster sleep residency. |

---

## Security / Crypto

| Module | Size | Description |
|---|---|---|
| `qseecom-mod.ko` | 734,688 (717.5 KB) | Qualcomm Secure Execution Environment Communication driver. Interface to TrustZone secure world for DRM, biometrics, and key storage. |
| `smcinvoke_mod.ko` | 294,104 (287.2 KB) | Secure Monitor Call invoke driver. Makes SMC calls to TrustZone for secure operations. |
| `qce50.ko` | 147,064 (143.6 KB) | Qualcomm Crypto Engine v5.0. Hardware-accelerated encryption/decryption (AES, SHA, etc.). |
| `qcedev-mod.ko` | 522,856 (510.6 KB) | Qualcomm Crypto Engine device driver. Userspace interface to hardware crypto acceleration. |
| `msm_rng.ko` | 24,632 (24.1 KB) | MSM hardware random number generator driver. |
| `nvmem_qfprom.ko` | 22,440 (21.9 KB) | QFPROM (Qualcomm Fuse Programmable Read-Only Memory) NVMEM driver. Reads hardware fuses for device identity, calibration, and security. |
| `tz_log.ko` | 52,736 (51.5 KB) | TrustZone log driver. Reads debug logs from the secure world. |

---

## DSP Loaders

| Module | Size | Description |
|---|---|---|
| `cdsp-loader.ko` | 17,864 (17.4 KB) | Compute DSP loader. Triggers firmware loading for the CDSP (used for compute-intensive tasks like ML inference). |
| `cdsprm.ko` | 63,000 (61.5 KB) | CDSP Resource Manager. Manages compute DSP resource allocation and thermal management. |
| `frpc-adsprpc.ko` | 832,384 (812.9 KB) | FastRPC ADSP Remote Procedure Call driver. Enables applications to offload processing to Hexagon ADSP/CDSP via FastRPC. |
| `mdt_loader.ko` | 19,848 (19.4 KB) | Meta-Data Table loader. Loads split firmware images (`.mdt` + `.bNN` files) for DSPs and other subsystems. |
| `qcom_pil_info.ko` | 16,664 (16.3 KB) | Peripheral Image Loader info driver. Provides metadata about loaded subsystem firmware images. |
| `qcom_ramdump.ko` | 24,640 (24.1 KB) | Ramdump driver. Captures memory dumps from remote processors on crashes for post-mortem analysis. |

---

## Rokid-Specific / Custom

| Module | Size | Description |
|---|---|---|
| `rt600_spiboot.ko` | 114,096 (111.4 KB) | NXP RT600 SPI boot driver. Loads speech SDK firmware onto the RT600 co-processor over SPI. **Blocklisted** in default module load list -- loaded conditionally by the speech service. |
| `panel-jbd-jbd4020-right.ko` | 45,800 (44.7 KB) | JBD JBD4020 micro-LED display panel driver (right side). Custom display panel used in Rokid AR glasses. |
| `psoc_ts_drv_right.ko` | 156,032 (152.4 KB) | Cypress PSoC touchpad/sensor driver (right temple). Touch/gesture input on the glasses frame. |
| `aw2110x.ko` | 66,776 (65.2 KB) | AWINIC AW2110x sensor driver. Proximity/light sensor on the glasses. |

---

## Debug / Trace (STM)

| Module | Size | Description |
|---|---|---|
| `stm_core.ko` | 67,872 (66.3 KB) | System Trace Macrocell core driver. ARM CoreSight STM for hardware-assisted debug tracing. |
| `stm_console.ko` | 11,800 (11.5 KB) | STM console driver. Routes kernel console output to STM trace. |
| `stm_ftrace.ko` | 13,040 (12.7 KB) | STM ftrace integration. Routes Linux ftrace events to STM hardware trace. |
| `stm_p_basic.ko` | 11,344 (11.1 KB) | STM basic protocol driver. |
| `stm_p_ost.ko` | 11,808 (11.5 KB) | STM OST (MIPI Original Stimulus Transport) protocol driver. |
| `stm_p_sys-t.ko` | 20,440 (20.0 KB) | STM SYS-T (MIPI System Software Trace) protocol driver. |
| `qdss_bridge.ko` | 43,840 (42.8 KB) | QDSS bridge driver. Routes Qualcomm Debug Subsystem trace data to USB or other transport. |
| `dcc_v2.ko` | 60,160 (58.8 KB) | Data Capture and Compare v2 driver. Hardware debug module that captures register snapshots on triggers. |
| `rdbg.ko` | 34,464 (33.7 KB) | Remote debugger driver. Provides debug access to remote processors. |
| `msm_sysstats.ko` | 40,464 (39.5 KB) | MSM system statistics driver. Exposes system-level performance counters and stats. |
| `boot_stats.ko` | 13,088 (12.8 KB) | Boot statistics driver. Records kernel boot timing milestones. |
| `rq_stats.ko` | 15,352 (15.0 KB) | Run queue statistics driver. Exposes CPU run queue depth for load monitoring. |
| `qcom_va_minidump.ko` | 37,936 (37.0 KB) | VA (Virtual Address) minidump driver. Registers memory regions for inclusion in crash minidumps. |
| `qcom_logbuf_vendor_hooks.ko` | 21,240 (20.7 KB) | Vendor log buffer hooks. Integrates vendor-specific logging with kernel log infrastructure. |

---

## Miscellaneous

| Module | Size | Description |
|---|---|---|
| `pinctrl-spmi-mpp.ko` | 34,112 (33.3 KB) | SPMI MPP (Multi-Purpose Pin) pin controller. Manages configurable pins on PMIC over SPMI bus. |
| `mfi_acp_30.ko` | 25,632 (25.0 KB) | MFi (Made for iPhone) Accessory Communication Protocol v3.0 driver. Apple MFi authentication chip interface. |

---

## Module Load Order

The `modules.load` file specifies 155 entries (with `qca_cld3_kiwi.ko` and `qca_cld3_kiwi_v2.ko` each listed twice). Modules are loaded in dependency order:

1. **System stats and debug** (`msm_sysstats`, `boot_stats`, etc.)
2. **MHI and PCIe** (`mhi`, `mhi_dev_*`, `pci-msm-drv`)
3. **Clock controllers** (`camcc-neo`, `debugcc-neo`, `gpucc-neo`, `videocc-neo`)
4. **DMA and bus** (`bam_dma`, `gpi`)
5. **IPC framework** (`qcom_glink`, `qcom_glink_smem`, `qcom_smd`, `smp2p`)
6. **Power management** (`qcom_lpm`, `qcom_aoss`, `power_state`)
7. **Connectivity** (`cnss2`, `qca_cld3_kiwi*`, `btpower`, `bt_fm_slim`)
8. **USB** (`phy-msm-*`, `usb_f_*`, `repeater*`)
9. **Sensors** (`psoc_ts_drv_right`, `aw2110x`, `qcom_tsens`)
10. **Battery** (`cw221x_battery`, `mp2724_charger`)
11. **Security** (`qseecom-mod`, `smcinvoke_mod`, `qce50`, `qcedev-mod`)
12. **Speech co-processor** (`rt600_spiboot` -- blocklisted, loaded on demand)
13. **Audio subsystem** (Q6 notifier -> SPF core -> GPR -> audio codecs -> machine driver)
14. **Display and GPU** (`panel-jbd-jbd4020-right`, `msm_kgsl`, `msm_drm`)
15. **Camera and video** (`camera`, `msm-eva`, `msm_video`)

---

## Blocklisted Modules

The following modules from `modules.blocklist` are explicitly prevented from loading:

- `llcc_perfmon` -- LLCC performance monitor (disabled in production)
- `mmrm_test_module` -- multimedia resource manager test
- `rt600_spiboot` -- RT600 SPI boot (loaded on demand, not at boot)
- Various TV tuner drivers (`tda18250`, `tda9887`, `tuner-simple`, `mt2266`, `tea5767`, `xc5000`, `mt2131`, `qt1010`, `tuner-types`, `tua9001`, `m88rs6000t`, `tda18218`, `mxl5007t`, `fc2580`, `r820t`, `mc44s803`, `fc0012`, `si2157`, `tda827x`, `tuner-xc2028`, `mt2060`, `qm1d1b0004`, `qm1d1c0042`, `tda18212`, `fc0013`, `msi001`, `fc0011`, `tda8290`, `max2165`, `xc4000`, `it913x`, `mt20xx`, `mxl301rf`, `mt2063`, `e4000`, `tea5761`, `tda18271`, `mxl5005s`)
- Test/debug modules (`8250_of`, `dummy_hcd`, `dummy-cpufreq`, `kheaders`, `atomic64_test`, `test_user_copy`, `lkdtm`, `rtc-test`, `torture`, `locktorture`, `rcutorture`, `q5drv_linux`, `limits_stat`)
- Network modules not needed (`net_failover`, `failover`, `vmw_vsock_*`, `vsock*`, `can-*`)
- ADC thermal monitor (`adc-tm`)
