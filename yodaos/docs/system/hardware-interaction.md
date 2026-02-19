# Hardware Interaction via Init Properties

### Camera LED Indicator
When the camera is opened, init writes to the white LED sysfs node:
- Camera open: `write /sys/class/leds/white/brightness 255`
- Camera close: `write /sys/class/leds/white/brightness 0`

Triggered by `vendor.rkd.camera.session_open` property.

### UART Debug Console
Controlled via an I2C device at `1-0008`:
- Path: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/uart_en`
- Enabled automatically on userdebug builds, disabled on user builds
- Can be manually toggled via `rokid.debug.uart` property

### Proximity Sensor
- Threshold control: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/thres_pct`
- Standard mode: threshold 0
- Sensitive mode: threshold 3
- Force override: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/enforce_psensor`
- Tuning mode: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/proximity_tuning`

### Hall Sensor
- Force override: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/enforce_hall`

### Slider Input
- Position control: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/slider_nu` (values 0-5)
- Tuning mode: `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/slider_tuning`

### RT600 Co-Processor (SPI)
- Device node: `/dev/rt600_spidev`
- Kernel module: `rt600_spiboot.ko` (loaded on boot_completed)
- AEC control: `/sys/bus/spi/devices/spi0.0/rt600_aec/value`
- Codec control: `/sys/bus/spi/devices/spi0.0/rt600_codec/value`
- Model selection: `/sys/bus/spi/devices/spi0.0/model`
- Language: `/sys/bus/spi/devices/spi0.0/language`
- Firmware update: `/sys/bus/spi/devices/spi0.0/change_firmware`
- Suspend control: `/sys/bus/spi/devices/spi0.0/suspend`
- HFP mode: `/sys/bus/spi/devices/spi0.0/hfp_on`

### Display Brightness Sync to XBL
Brightness values are written to vendor logfs for boot-stage display control:
- Panel brightness: `/mnt/vendor/logfs/panel_bright`
- System brightness: `/mnt/vendor/logfs/system_bright`

### Persistent Storage Files
The following files in `/mnt/vendor/persist/` are set readable during post-fs:
- `sn.bin` - Serial number
- `pcbasn.bin` - PCB assembly serial number
- `quec_bt_mac` - Bluetooth MAC address (Quectel module)
- `serialno.txt` - Human-readable serial number
- `typeid.bin` - Device type identifier (maps to OEM variant)
- `kiwi_v2/wlan_mac.bin` - WiFi MAC address (Kiwi v2 WLAN chip)
