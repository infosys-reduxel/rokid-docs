# AOSP and Non-Rokid Components

Inventory of standard Android (AOSP), Qualcomm, and third-party components found on the YodaOS device.
This excludes Rokid-specific apps (see [rokid-apps-overview.md](apps/rokid-apps-overview.md)) and Chinese payment apps (see [chinese-apps.md](apps/chinese-apps.md)).

## Partition Layout

The decompiled apps are organized by partition and type:

| Partition    | Subdirectories                    |
|--------------|-----------------------------------|
| system       | app, priv-app, framework          |
| product      | app, priv-app, framework, overlay |
| system_ext   | app, priv-app, framework          |
| vendor       | app, overlay                      |
| apex         | (APEX modules, see below)         |

---

## AOSP Apps

### system/app

| App Name                  | Package                              | Type |
|---------------------------|--------------------------------------|------|
| BasicDreams               | com.android.dreams.basic             | app  |
| Bluetooth                 | com.android.bluetooth                | app  |
| BluetoothMidiService      | com.android.bluetoothmidiservice     | app  |
| BookmarkProvider          | com.android.bookmarkprovider         | app  |
| CameraExtensionsProxy     | com.android.cameraextensions         | app  |
| CertInstaller             | com.android.certinstaller            | app  |
| CompanionDeviceManager    | com.android.companiondevicemanager   | app  |
| CtsShimPrebuilt           | com.android.cts.ctsshim             | app  |
| ExtShared                 | android.ext.shared                   | app  |
| KeyChain                  | com.android.keychain                 | app  |
| PacProcessor              | com.android.pacprocessor             | app  |
| PlatformCaptivePortalLogin| com.android.captiveportallogin       | app  |
| Protips                   | com.android.protips                  | app  |

### system/priv-app

| App Name                         | Package                                    | Type     |
|----------------------------------|--------------------------------------------|----------|
| BackupRestoreConfirmation        | com.android.backupconfirm                  | priv-app |
| BlockedNumberProvider            | com.android.providers.blockednumber        | priv-app |
| CalendarProvider                 | com.android.providers.calendar             | priv-app |
| ContactsProvider                 | com.android.providers.contacts             | priv-app |
| CtsShimPrivPrebuilt              | com.android.cts.priv.ctsshim              | priv-app |
| DownloadProvider                 | com.android.providers.downloads            | priv-app |
| DownloadProviderUi               | com.android.providers.downloads.ui         | priv-app |
| DynamicSystemInstallationService | com.android.dynsystem                      | priv-app |
| ExternalStorageProvider          | com.android.externalstorage                | priv-app |
| FusedLocation                    | com.android.location.fused                 | priv-app |
| InProcessNetworkStack            | com.android.networkstack.inprocess         | priv-app |
| InputDevices                     | com.android.inputdevices                   | priv-app |
| LocalTransport                   | com.android.localtransport                 | priv-app |
| ManagedProvisioning              | com.android.managedprovisioning            | priv-app |
| MediaProviderLegacy              | com.android.providers.media                | priv-app |
| MtpService                       | com.android.mtp                            | priv-app |
| MusicFX                          | com.android.musicfx                        | priv-app |
| PackageInstaller                 | com.android.packageinstaller               | priv-app |
| PlatformNetworkPermissionConfig  | com.android.networkstack.permissionconfig  | priv-app |
| ProxyHandler                     | com.android.proxyhandler                   | priv-app |
| SettingsProvider                 | com.android.providers.settings             | priv-app |
| SharedStorageBackup              | com.android.sharedstoragebackup            | priv-app |
| Shell                            | com.android.shell                          | priv-app |
| StatementService                 | com.android.statementservice               | priv-app |
| TelephonyProvider                | com.android.providers.telephony            | priv-app |
| Traceur                          | com.android.traceur                        | priv-app |
| UserDictionaryProvider           | com.android.providers.userdictionary       | priv-app |
| VpnDialogs                       | com.android.vpndialogs                     | priv-app |

### product/app

| App Name       | Package                    | Type |
|----------------|----------------------------|------|
| Camera2        | com.android.camera2        | app  |
| ModuleMetadata | com.android.modulemetadata | app  |
| webview        | (no manifest extracted)    | app  |

### product/priv-app

| App Name              | Package                            | Type     |
|-----------------------|------------------------------------|----------|
| OneTimeInitializer    | com.android.onetimeinitializer     | priv-app |
| SettingsIntelligence  | com.android.settings.intelligence  | priv-app |

### system_ext/priv-app

| App Name       | Package                    | Type     |
|----------------|----------------------------|----------|
| CarrierConfig  | com.android.carrierconfig  | priv-app |
| Provision      | com.android.provision      | priv-app |
| Settings       | com.android.settings       | priv-app |
| StorageManager | com.android.storagemanager | priv-app |

---

## Qualcomm (QTI/QC) Components

Qualcomm-specific apps and services bundled with the Snapdragon BSP.

### system_ext/app

| App Name            | Package                                              | Type |
|---------------------|------------------------------------------------------|------|
| BluetoothDsDaService| com.qualcomm.qtil.btdsda                             | app  |
| colorservice        | com.qti.service.colorservice                         | app  |
| HeadlessLauncher    | com.qualcomm.qti.headlesslauncher                    | app  |
| QdcmFF              | com.qti.snapdragon.qdcm_ff                           | app  |
| QTIDiagServices     | com.qti.diagservices                                 | app  |
| WigigTetheringRRO   | com.qualcomm.qti.server.wigig.tethering.rro          | app  |

### system_ext/priv-app

| App Name       | Package                                          | Type     |
|----------------|--------------------------------------------------|----------|
| QAS_DVC_MSP    | com.qualcomm.ltebc                               | priv-app |
| seccamservice  | com.qualcomm.qti.seccamservice                   | priv-app |

### product/priv-app

| App Name          | Package                    | Type     |
|-------------------|----------------------------|----------|
| QtiSoundRecorder  | com.android.soundrecorder  | priv-app |

### vendor/app

| App Name               | Package                                          | Type |
|------------------------|--------------------------------------------------|------|
| pasrservice            | com.qti.pasrservice                              | app  |
| TimeService            | com.qualcomm.timeservice                         | app  |
| TrustZoneAccessService | com.qualcomm.qti.qms.service.trustzoneaccess     | app  |

---

## APEX Modules

APEX (Android Pony EXpress) modules provide updatable system components.

| APEX Module                            | Bundled APKs / Contents                                      |
|----------------------------------------|--------------------------------------------------------------|
| com.android.appsearch                  | framework-jadx                                               |
| com.android.art                        | framework-jadx                                               |
| com.android.conscrypt                  | framework-jadx                                               |
| com.android.extservices                | ExtServices                                                  |
| com.android.i18n                       | framework-jadx                                               |
| com.android.ipsec                      | framework-jadx                                               |
| com.android.media                      | framework-jadx                                               |
| com.android.mediaprovider              | framework-jadx, MediaProvider                                |
| com.android.os.statsd                  | framework-jadx                                               |
| com.android.permission                 | framework-jadx, PermissionController                         |
| com.android.scheduling                 | framework-jadx                                               |
| com.android.sdkext                     | framework-jadx                                               |
| com.android.tethering.inprocess        | framework-jadx, InProcessTethering, ServiceConnectivityResources |
| com.android.wifi                       | framework-jadx, OsuLogin, ServiceWifiResources               |

---

## Resource Overlay APKs

Runtime Resource Overlays (RROs) used to customize AOSP component behavior and appearance.

### product/overlay

| Overlay Name                    | Package                                              |
|---------------------------------|------------------------------------------------------|
| CarrierConfigResCommon_Sys      | com.android.carrierconfig.overlay.common             |
| CellBroadcastReceiverResCommon_Sys | com.android.cellbroadcastreceiver.overlay.common  |
| FrameworksResCommonQva_Sys      | android.qvaoverlay.common                            |
| FrameworksResCommon_Rkd         | android.overlay.common.rkd                           |
| FrameworksResCommon_Sys         | android.overlay.common                               |
| SettingsResCommon_Sys           | com.android.settings.overlay.common                  |
| SystemUIResCommon_Sys           | com.android.systemui.overlay.common                  |
| TelecommResCommon_Sys           | com.android.server.telecom.overlay.common            |
| TelephonyResCommon_Sys          | com.android.phone.overlay.common                     |
| WifiResCommon_Rkd               | com.android.wifi.resources.overlay.rkd               |
| WifiResCommon_Sys               | com.android.wifi.resources.overlay.common            |
| WifiStackResCommon_Rkd          | com.android.networkstack.overlay.rkd                 |

Note: Overlays with `_Rkd` suffix are Rokid-specific customizations of AOSP resources (WiFi config, framework defaults).

### vendor/overlay

| Overlay Name                | Package                                         |
|-----------------------------|-------------------------------------------------|
| FrameworksResTarget_Vendor  | android.overlay.target                          |
| WifiResTarget_Vendor        | com.android.wifi.resources.overlay.target       |

---

## System Framework

The `framework/` subdirectories contain framework JARs (decompiled via jadx) and the core `framework-res` resource package (`android`, system/framework).

### system/framework (notable JARs)

android.hidl.base-V1.0-java, android.hidl.manager-V1.0-java, android.test.base, android.test.mock, android.test.runner, com.android.future.usb.accessory, com.android.location.provider, com.android.mediadrm.signer, com.android.media.remotedisplay, ethernet-service, ext, framework, framework-graphics, framework-res (package: `android`)

### product/framework

com.qti.snapdragon.sdk.display, extphonelib-product, ims-ext-common

### system_ext/framework

audiosphere, com.qualcomm.qti.imscmservice (V2.0-V2.2), com.qualcomm.qti.uceservice (V2.0-V2.3), com.quicinc.cne.api (V1.0-V1.1), com.quicinc.cne.constants (V1.0-V2.1), extphonelib, vendor.qti.data.factory (V1.0-V2.4)
