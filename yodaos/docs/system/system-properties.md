# Rokid-Specific System Properties

### Build / Identity Properties

| Property | Source | Description |
|---|---|---|
| `ro.rokid.product.model` | vendor/build.prop | Product model string ("RG-glasses") |
| `ro.rokid.ota.check_url` | vendor/build.prop | OTA server URL (https://ota.rokid.com) |
| `ro.rokid.ota.check_api` | vendor/build.prop | OTA check endpoint (/v1/extended/ota/check) |
| `ro.product.rokid.oem.model` | init.rokid_oem_define.rc | Hardware model variant (RV101/RV102/RV201/RV202/RV203) |
| `ro.product.rokid.oem.id` | init.rokid_oem_define.rc | Numeric OEM ID (101-106, 201-203) |

### Boot Properties (set by init.rokid_oem_define.rc)

| Property | Description |
|---|---|
| `ro.boot.devicetypeid` | UUID identifying the hardware variant (read from bootloader) |
| `ro.boot.key` | Device authentication key (per-variant) |
| `ro.boot.secret` | Device authentication secret (per-variant) |
| `ro.boot.glassesWithPanel` | Whether device has a display panel (1=yes, 0=no) |
| `ro.boot.Panel` | Same as above, duplicate property |

### Vendor Runtime Properties

| Property | Source | Description |
|---|---|---|
| `vendor.rkd.camera.session_open` | init.rokid.rc | Camera session state (1=open, 0=closed); triggers LED indicator |
| `vendor.rkd.display.brightness` | init.rokid.rc | System display brightness; synced to XBL via logfs |
| `vendor.rkd.display.brightness_xbl` | init.rokid.rc | XBL-side brightness value written to panel_bright |

### Debug Properties

| Property | Source | Description |
|---|---|---|
| `rokid.debug.uart` | init.rokid.rc | Enable/disable UART console (1/0) |
| `rokid.debug.enforce_hall` | init.rokid.rc | Force hall sensor state (1/0) |
| `rokid.debug.enforce_psensor` | init.rokid.rc | Force proximity sensor state (1/0) |
| `rokid.debug.proximity_tuning` | init.rokid.rc | Enable proximity sensor tuning mode (1/0) |
| `rokid.debug.slider_tuning` | init.rokid.rc | Enable slider input tuning mode (1/0) |
| `rokid.debug.slider_nu` | init.rokid.rc | Set slider position number (0-5) |
| `rokid.psensor.mode` | init.rokid.rc | Proximity sensor mode ("standard" = threshold 0, "sensitive" = threshold 3) |

### SELinux Property Contexts

The following Rokid-specific property prefixes are defined in `system_ext/etc/selinux/system_ext_property_contexts`:

| Property prefix | SELinux context |
|---|---|
| `ro.vendor.rkd.` | `u:object_r:exported_system_prop:s0` |
| `persist.vendor.rkd.` | `u:object_r:exported_system_prop:s0` |
| `ro.rokid.` | `u:object_r:exported_system_prop:s0` |
| `vendor.rkd.` | `u:object_r:exported_system_prop:s0` |
| `ro.boot.glassesWithPanel` | `u:object_r:exported_system_prop:s0` |
