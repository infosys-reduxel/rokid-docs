# Product Variants

> For the full Rokid Glasses hardware specification table (SoC, RAM/ROM, battery, camera,
> optics, dimensions, connectivity, etc.), see [Device Specifications — Rokid Glasses](../sprite-overview.md#device-specifications--rokid-glasses)
> in `sprite-overview.md`, sourced from `https://developerdoc.rokid.com/sprite` (live-verified
> 2026-07-10; matches the upstream spec table exactly — no drift). This file covers only the
> OEM ID / model variant mapping, not the full spec sheet.

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
