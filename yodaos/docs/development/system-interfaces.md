# System Interfaces

## Sysfs Interfaces

The Rokid hardware control MCU is accessible at I2C address `1-0008` on bus `a90000.i2c`:

Base path: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/`

| Sysfs Node | Type | Description |
|---|---|---|
| uart_en | write | Enable (1) / disable (0) UART debug console |
| enforce_hall | write | Force hall sensor state |
| enforce_psensor | write | Force proximity sensor state |
| thres_pct | write | Proximity sensor threshold percentage (0=standard, 3=sensitive) |
| proximity_tuning | write | Enable proximity sensor tuning mode |
| slider_tuning | write | Enable slider tuning mode |
| slider_nu | write | Set slider number (0-5) |

Camera LED indicator: `/sys/class/leds/white/brightness` (set to 255 when camera opens, 0 when it closes).

Brightness sync: `/mnt/vendor/logfs/panel_bright` and `/mnt/vendor/logfs/system_bright`.

## Services and Daemons

### Rokid-Specific Services

| Service | Binary | Description |
|---|---|---|
| bootconfigs | /system/xbin/bootconfigs | Boot-time configuration setup, runs on post-fs |
| backup_keys | /system/xbin/backupkeys | Key backup service, runs on boot_completed |

### Key Vendor Services

| Service | Description |
|---|---|
| vendor.sensors | Sensor HAL multihal service |
| vendor.face-default | Face biometric HAL (android.hardware.biometrics.face@1.0-service.face) |
| vendor.pd_mapper | Peripheral daemon mapper |
| vendor.cnss_diag | WLAN diagnostics (HELIUM chipset) |
| vendor.power_off_alarm | Power-off alarm handler |

## Persist Partition Files

The `/mnt/vendor/persist/` partition contains device-specific identity files:

| File | Description |
|---|---|
| sn.bin | Device serial number |
| pcbasn.bin | PCB assembly serial number |
| quec_bt_mac | Bluetooth MAC address |
| serialno.txt | Human-readable serial number |
| typeid.bin | Device type identifier |
| kiwi_v2/wlan_mac.bin | WLAN MAC address |

All files are set to mode `0644` during post-fs.
