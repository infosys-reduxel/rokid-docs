# Boot Chain

The boot sequence follows the standard Android init process with Qualcomm BSP and Rokid customizations layered on top. The import chain determines the order in which RC files are parsed.

### Init RC Import Chain

```
system/etc/init/hw/init.rc (main system init)
  |-- init.environ.rc (environment variables)
  |-- system/etc/init/hw/init.usb.rc (USB configuration)
  |-- init.${ro.hardware}.rc (hardware-specific, resolves to init.qcom.rc)
  |-- vendor/etc/init/hw/init.${ro.hardware}.rc (vendor hardware init)
  |   |-- vendor/etc/init/hw/init.qcom.rc
  |   |   |-- vendor/etc/init/hw/init.qti.ufs.rc (UFS storage)
  |   |   |-- vendor/etc/init/hw/init.qcom.usb.rc (USB)
  |   |   |-- vendor/etc/init/hw/init.qcom.test.rc (test hooks)
  |   |   |-- vendor/etc/init/hw/init.target.rc (target-specific)
  |   |   |   |-- vendor/etc/init/hw/init.qti.kernel.rc (kernel modules)
  |   |   |   |   |-- vendor/etc/init/hw/init.qti.kernel.test.rc
  |   |   |   |-- vendor/etc/init/hw/init.rokid.rc (ROKID MAIN)
  |   |   |       |-- vendor/etc/init/hw/init.rokid_oem_define.rc (OEM variant setup)
  |   |   |-- vendor/etc/init/hw/init.qcom.factory.rc (factory test mode)
  |-- system/etc/init/hw/init.usb.configfs.rc (USB ConfigFS)
  |-- system/etc/init/hw/init.zygote64_32.rc (Zygote launcher)
```

Additionally, all `.rc` files found in `vendor/etc/init/`, `system/etc/init/`, `system_ext/etc/init/`, and `product/etc/init/` are automatically imported by the init system.

### Boot Phase Sequence

1. **early-init**: Mount tracefs, create device symlinks, disable UFS power saving, load kernel modules via `vendor.modprobe`, run `hibernate.resume`
2. **init**: Create block device symlink, start `logd`, create cgroup memory groups, start `servicemanager`, `hwservicemanager`, `vndservicemanager`, `lmkd`
3. **early-fs**: Start `vold` (volume daemon)
4. **fs**: Start `hwservicemanager`, mount early fstab, run `set_serialno` and `set_mfi` (Rokid device identity setup)
5. **post-fs**: Set RLIMIT_MEMLOCK, remount rootfs read-only, set up persist partition permissions (sn.bin, pcbasn.bin, quec_bt_mac, serialno.txt, typeid.bin, wlan_mac.bin)
6. **late-fs**: Wait for hwservicemanager, mount late fstab, start `early_hal` class
7. **post-fs-data**: Create data directories, enable WLAN cold boot calibration, start `tombstoned`, `apexd`, setup keystore
8. **zygote-start**: Start `statsd`, `netd`, `zygote`, `zygote_secondary`
9. **early-boot**: Run `init.qcom.early_boot.sh`, start `vendor.sensors`, boot ADSP/CDSP/SLPI/CVP subsystems, run `verity_update_state`
10. **boot**: Configure Bluetooth, network, sensors, start HAL class, start `core` class. OEM variant resolution happens here via `init.rokid_oem_define.rc` property triggers.
11. **boot_completed** (property trigger): Load `rt600_spiboot.ko` kernel module, re-enable UFS power management, start `kernel-boot`, `kernel-post-boot`, `qcom-post-boot`, `GateServiced`, `backup_keys`, enable WiFi recovery, reinit lmkd
