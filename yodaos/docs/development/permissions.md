# Permissions

### Qualcomm Custom Permissions

Source: `system_ext/etc/permissions/qti_permissions.xml`

| Permission | Group GID | Description |
|---|---|---|
| com.qti.permission.DIAG | oem_2901 | Access to QTI diagnostics |
| com.qti.permission.AUDIO | audio | Access to QTI audio subsystem |

### Platform Permission-to-GID Mappings

Source: `system/system/etc/permissions/platform.xml`

| Permission | GID(s) |
|---|---|
| android.permission.BLUETOOTH_ADMIN | net_bt_admin |
| android.permission.BLUETOOTH | net_bt |
| android.permission.BLUETOOTH_STACK | bluetooth, wakelock, uhid |
| android.permission.VIRTUAL_INPUT_DEVICE | uhid |
| android.permission.NET_TUNNELING | vpn |
| android.permission.INTERNET | inet |
| android.permission.READ_LOGS | log |
| android.permission.ACCESS_MTP | mtp |
| android.permission.NET_ADMIN | net_admin |
| android.permission.MAINLINE_NETWORK_STACK | net_admin, net_raw |
| android.permission.ACCESS_CACHE_FILESYSTEM | cache |
| android.permission.DIAGNOSTIC | input, diag |
| android.permission.READ_NETWORK_USAGE_HISTORY | net_bw_stats |
| android.permission.UPDATE_DEVICE_STATS | net_bw_acct |
| android.permission.LOOP_RADIO | loop_radio |
| android.permission.MANAGE_VOICE_KEYPHRASES | audio |
| android.permission.ACCESS_BROADCAST_RADIO | media |
| android.permission.USE_RESERVED_DISK | reserved_disk |

### Bluetooth Permission Splits (API 31+)

For apps targeting SDK 31 (Android 12) and above, legacy Bluetooth permissions are automatically split:

| Original Permission | Splits Into |
|---|---|
| android.permission.BLUETOOTH | BLUETOOTH_SCAN, BLUETOOTH_CONNECT, BLUETOOTH_ADVERTISE |
| android.permission.BLUETOOTH_ADMIN | BLUETOOTH_SCAN, BLUETOOTH_CONNECT, BLUETOOTH_ADVERTISE |

### Privileged App Permissions

Source: `system/system/etc/permissions/privapp-permissions-qti.xml`

Notable privileged apps and their elevated permissions:

| Package | Key Permissions |
|---|---|
| com.quicinc.cne.CNEService | INTERACT_ACROSS_USERS, PACKET_KEEPALIVE_OFFLOAD |
| org.codeaurora.dialer | MODIFY_PHONE_STATE, READ_PRIVILEGED_PHONE_STATE, CONTROL_INCALL_EXPERIENCE |
| org.codeaurora.ims | READ_PRECISE_PHONE_STATE, INTERACT_ACROSS_USERS |
| com.android.soundrecorder | WRITE_MEDIA_STORAGE, CAPTURE_AUDIO_OUTPUT |
| com.quicinc.voice.activation | CAPTURE_AUDIO_HOTWORD, MANAGE_SOUND_TRIGGER |
| org.codeaurora.snapcam | MOUNT_UNMOUNT_FILESYSTEMS, WRITE_MEDIA_STORAGE |
| com.qualcomm.qti.callenhancement | RECORD_AUDIO, CAPTURE_AUDIO_OUTPUT |

### Custom Rokid Permissions

No custom `com.rokid.permission.*` declarations were found in the decompiled firmware. Rokid hardware control is achieved entirely through system properties (`rokid.debug.*`, `vendor.rkd.*`) and direct sysfs writes rather than through Android permission framework extensions.
