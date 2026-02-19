# Sensors

Source: `vendor/etc/sensors/`, JSON configs in `vendor/etc/sensors/config/`.

### 2.1 Sensor HALs

File: `vendor/etc/sensors/hals.conf`:
```
sensors.ssc.so
sensors.dynamic_sensor_hal.so
```
- `sensors.ssc.so` -- Qualcomm Snapdragon Sensor Core (SSC), main sensor HAL
- `sensors.dynamic_sensor_hal.so` -- dynamic sensor discovery

### 2.2 IMU -- ICM-4x6xx (InvenSense)

Config: `vendor/etc/sensors/config/neo_icm4x6xx_0.json`.
Platform: IDP/QXR boards, SoC IDs 554/579.

**Provided sensor types:**
- **Accelerometer** -- DRI mode, resolution index 4
- **Gyroscope** -- DRI mode, resolution index 7
- **Motion detect** -- DRI mode
- **Freefall detect** -- DRI mode
- **Temperature** -- polling mode, resolution index 2

**Bus configuration:**
- Bus type: 3 (I3C)
- Bus instance: 1
- I2C slave address: 0x68 (104 decimal)
- I3C address: 10
- Max bus speed: 12,500 kHz (I3C high-speed)
- DRI IRQ: GPIO 31, pull-up, chip pin, rising edge trigger
- Power rail: `/pmic/client/sensor_vddio`
- Rigid body type: 0 (device body)

**Orientation mapping** (sensor axes to device axes):
- X = +Y
- Y = -X
- Z = +Z

**Motion detection threshold:** 0.6132 (m/s^2)

### 2.3 Proximity / Ambient Light -- UCS146E0

Config: `vendor/etc/sensors/config/neo_ucs146e0_0.json`.
Platform: IDP/QXR boards, SoC IDs 554/579.

**Provided sensor types:**
- **Ambient light sensor (ALS)** -- polling mode, resolution index 3
- **Proximity sensor** -- polling mode, resolution index 0

**Bus configuration:**
- Bus type: 0 (I2C)
- Bus instance: 2
- Slave address: 0x38 (56 decimal)
- Bus speed: 400 kHz (I2C fast mode)
- DRI IRQ: GPIO 11, no pull, chip pin, low-level trigger
- Power rails: `/pmic/client/sensor_vdd` + `/pmic/client/sensor_vddio`

**Proximity calibration defaults:**
- Near threshold: 1500
- Far threshold: 1000

### 2.4 Virtual / Fusion Sensors

Defined in `vendor/etc/sensors/config/neo_default_sensors.json`:

| Sensor | Description |
|--------|-------------|
| `accel` | Accelerometer (default priority selection) |
| `gyro` | Gyroscope |
| `mag` | Magnetometer |
| `motion_detect` | Motion detection |
| `sensor_temperature` | Sensor temperature (source: icm4x6xx) |
| `proximity` | Proximity sensor |
| `ambient_light` | Ambient light sensor |
| `sar` | SAR (Specific Absorption Rate) sensor |
| `accel_cal` | Accelerometer calibration |
| `gyro_cal` | Gyroscope calibration |
| `mag_cal` | Magnetometer calibration |
| `amd` | Absolute Motion Detector |
| `tilt` | Tilt detector |
| `gyro_rot_matrix` | Gyro rotation matrix |
| `gravity` | Gravity sensor |
| `game_rv` | Game rotation vector |
| `geomag_rv` | Geomagnetic rotation vector |
| `fmv` | Fast Motion Vector |
| `rotv` | Rotation vector |

### 2.5 Sensor Algorithm Configs

Individual JSON configs for each sensor algorithm (in `vendor/etc/sensors/config/`):

| File | Algorithm |
|------|-----------|
| `sns_amd.json` | Absolute Motion Detector |
| `sns_aont.json` | Always-On Timer |
| `sns_basic_gestures.json` | Basic gesture recognition |
| `sns_bring_to_ear.json` | Bring-to-ear detection |
| `sns_ccd_v*.json` | Concurrency Control Daemon (walk detection, v1-v4) |
| `sns_cm.json` | Connection Manager |
| `sns_dae.json` | Data Acquisition Engine |
| `sns_device_orient.json` | Device orientation |
| `sns_distance_bound.json` | Distance bound |
| `sns_dpc.json` | Device Position Classifier |
| `sns_facing.json` | Facing detection |
| `sns_fmv.json` / `sns_fmv_legacy.json` | Fast Motion Vector |
| `sns_geomag_rv.json` | Geomagnetic rotation vector |
| `sns_gyro_cal.json` | Gyroscope calibration |
| `sns_heart_rate.json` | Heart rate |
| `sns_mag_cal.json` / `sns_mag_cal_legacy.json` | Magnetometer calibration |
| `sns_multishake.json` | Multi-shake detection |
| `sns_pedometer.json` | Pedometer |
| `sns_rmd.json` | Relative Motion Detector |
| `sns_rotv.json` | Rotation vector |
| `sns_smd.json` | Significant Motion Detector |
| `sns_tilt.json` | Tilt detector |
| `sns_tilt_to_wake.json` | Tilt-to-wake |
| `sns_wrist_pedo.json` | Wrist pedometer |

### 2.6 3-DOF Head Tracking (RFM)

Config: `vendor/etc/rfm_3dof.conf`.

Parameters for 3-DOF (rotation-only) head tracking:
- EMA smoothing alpha: 0.333
- Motion variance threshold: 2.5
- Static variance threshold: 2.0
- Gravity offset tolerance: 0.4
- Motion decision frames: 10

Sensor rate reference (from config comments):
| Constant | Interval | Frequency |
|----------|----------|-----------|
| SENSOR_DELAY_NORMAL | 200 ms | 5 Hz |
| SENSOR_DELAY_UI | 60 ms | ~17 Hz |
| SENSOR_DELAY_GAME | 20 ms | 50 Hz |
| SENSOR_DELAY_FASTEST | 0 ms (ASAP) | 100-200+ Hz |

### 2.7 Rokid-Specific Sensor Controls

From `vendor/etc/init/hw/init.rokid.rc`:

- **Hall sensor** -- `enforce_hall` sysfs at `/sys/devices/platform/soc/a90000.i2c/i2c-1/1-0008/enforce_hall`
- **Proximity sensor override** -- `enforce_psensor` sysfs, tuning via `proximity_tuning`
- **Proximity modes**: standard (threshold 0%) and sensitive (threshold 3%)
- **Slider sensor** -- `slider_tuning` sysfs, `slider_nu` for slider number (0-5)
- All controls via I2C device at bus `a90000.i2c`, address `1-0008`
