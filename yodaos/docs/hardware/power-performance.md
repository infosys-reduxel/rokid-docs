# Power and Performance

Source: `vendor/etc/powerhint.xml`, `vendor/etc/perf/`.

### 5.1 CPU Configuration

From `vendor/etc/perf/targetconfig.xml`:

**Target "neo_la"** (SoC IDs 554, 579):
- Clusters: 1
- Total cores: 4
- Cluster type: "big" (all cores same class)
- Sync core: yes
- Min cores online: 1
- CPU frequency governor: schedutil
- Governor instance type: single (one governor for all cores)

CPU frequency tiers (from powerhint comments):
- Little/L cluster: 614 MHz - 1.651 GHz (max boost to 1.094 GHz sustained)
- Big/B cluster: 652 MHz - 2.208 GHz
- Prime cluster: 768 MHz - 2.4 GHz

Note: the single-cluster 4-core design means the "Little/Big/Prime" labels in powerhint.xml are logical frequency tiers applied to the same physical cores.

### 5.2 GPU Configuration

GPU frequency tiers (from powerhint comments):
- Min power level 4: 285 MHz
- Level 3: ~443 MHz
- Max power level 0: 540 MHz

### 5.3 Power Hints

Target: `neo_la`. All from `vendor/etc/powerhint.xml`:

**Camera Hints:**

| Hint ID | Scenario | Key Settings |
|---------|----------|-------------|
| 0x1330 | ZSL Preview | BWMON 33ms sample, io_percent 100, LLC max 933 MHz, schedutil PL disabled, hispeed load 99 |
| 0x1331 | 30fps Recording | Same as ZSL + DDR min 1555 MHz |
| 0x1332 | 60fps Recording | BWMON 16ms sample, L CPU max uncapped / min 1497 MHz, big/prime schedutil PL disabled, core control disabled, upmigrate 35/85, downmigrate 30/85 |
| 0x1333 | HFR Recording | BWMON 16ms, L CPU max 1094 MHz, same tuning as 60fps |
| 0x1334 | HFR 480fps Encode | All CPUs power collapse disabled, sched boost, all clusters max freq |
| 0x1337 | Camera Open | Power collapse disabled, sched boost, all cores max freq, DDR min 1555 MHz |
| 0x1338 | Camera Close | Same as camera open |
| 0x1339 | Snapshot | Power collapse disabled, min freq 1.113 GHz |

**QVR (Qualcomm VR) Hints:**

All QVR levels (CPU1-3 x GPU1-3, IDs 0x130A through 0x1312) use identical settings:
- Prime: 768 MHz - 2.4 GHz
- Big: 652 MHz - 2.208 GHz
- Little: 614 MHz - 1.094 GHz
- GPU: 285 MHz - 540 MHz
- Min big CPUs online: 2
- Indefinite duration

**Performance Mode Hints:**

| Hint ID | Mode | CPU Settings | GPU Settings |
|---------|------|-------------|-------------|
| 0x1206 | Sustained Performance | Prime/L max 1.094 GHz, B max 1.382 GHz | 285-443 MHz |
| 0x1207 | VR Mode | Prime 1.094-2.169 GHz, B 1.132-1.804 GHz, L 0.902-1.651 GHz | 285-540 MHz |
| 0x1301 | VR Sustained | Prime 1.094 GHz locked, B 1.132 GHz locked, L 1.094 GHz locked | 443 MHz locked |

### 5.4 Performance Boost Configs

From `vendor/etc/perf/perfboostsconfig.xml`:

| Boost | Target | Timeout | Enabled |
|-------|--------|---------|---------|
| App Launch (main) | neo_la | 2000 ms | yes |
| App Launch (disable packing) | neo_la | 1500 ms | no |
| Display On | neo_la | indefinite | yes |
| Display Off | neo_la | indefinite | yes |
| Package Install | neo | -- | no |
| Rotation Latency | neo | 1500 ms | no |

### 5.5 Performance Framework

Perf config files in `vendor/etc/perf/`:
- `targetconfig.xml` -- target CPU topology
- `perfboostsconfig.xml` -- boost trigger configs
- `targetresourceconfigs.xml` -- resource opcode definitions
- `targetsysnodesconfigs.xml` -- sysfs node mappings
- `targetavcsysnodesconfigs.xml` -- AVC sysfs nodes
- `perfconfigstore.xml` -- perf config store
- `powerhint.xml` -- power hint definitions (same as vendor/etc)
- `commonresourceconfigs.xml` / `commonsysnodesconfigs.xml` -- shared configs

Libraries:
- `libqti-perfd.so` / `libqti-perfd-client.so` -- performance daemon
- `libperfconfig.so` / `libperfgluelayer.so` / `libperfioctl.so` -- perf framework
- `vendor.qti.hardware.perf@2.0-2.3.so` -- perf HAL interfaces
- `libpowercore.so` / `libpowercallback.so` -- power core
