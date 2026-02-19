# Product Variants

From `vendor/etc/init/hw/init.rokid_oem_define.rc`:

| OEM ID | Model | Panel | Description |
|--------|-------|-------|-------------|
| 101 | RV101 | Yes | Rokid Glasses (domestic) |
| 102 | RV102 | Yes | Rokid Glasses (carrier edition) |
| 103 | RV203 | No | Rokid AI Glasses (no display) |
| 104 | RV101 | Yes | Rokid Glasses (state gift edition) |
| 105 | RV101 | Yes | Rokid Glasses (overseas) |
| 106 | RV101 | Yes | Rokid Glasses (Leqi Smart brand) |
| 201 | RV201 | No | Bolon AI Glasses |
| 202 | RV202 | No | Bolon AI Glasses (carrier edition) |
| 203 | RV201 | No | Bolon AI Glasses (celebrity custom, tinted gray) |

Each variant is identified by a `devicetypeid` UUID set during boot, which configures:
- `ro.boot.key` / `ro.boot.secret` -- device authentication keys
- `ro.boot.glassesWithPanel` / `ro.boot.Panel` -- display presence (1 = with display, 0 = without)
- `persist.sys.customanim.boot` -- boot animation path (display models only)
