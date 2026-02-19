# HAL Interfaces

## VINTF HAL Interfaces

The Vendor Interface (VINTF) manifest declares all HAL services available to the system.
The device manifest uses `target-level="6"` and SEPolicy version `32.0`.

### Vendor Manifest (Main)

Source: `vendor/etc/vintf/manifest.xml`

| HAL Name | Format | Transport | Version | Interface | Instance |
|---|---|---|---|---|---|
| android.hardware.audio | HIDL | hwbinder | 7.0 | IDevicesFactory | default |
| android.hardware.audio.effect | HIDL | hwbinder | 7.0 | IEffectsFactory | default |
| android.hardware.bluetooth | HIDL | hwbinder | 1.0 | IBluetoothHci | default |
| android.hardware.bluetooth.audio | HIDL | hwbinder | 2.0 | IBluetoothAudioProvidersFactory | default |
| android.hardware.camera.provider | HIDL | hwbinder | 2.7 | ICameraProvider | legacy/1 |
| android.hardware.gatekeeper | HIDL | hwbinder | 1.0 | IGatekeeper | default |
| android.hardware.media.omx | HIDL | hwbinder | 1.0 | IOmx, IOmxStore | default |
| android.hardware.secure_element | HIDL | hwbinder | 1.2 | ISecureElement | SIM1, SIM2 |
| com.dsi.ant | HIDL | hwbinder | 1.0 | IAnt | default |
| vendor.qti.hardware.AGMIPC | HIDL | hwbinder | 1.0 | IAGM | default |
| vendor.qti.hardware.alarm | HIDL | hwbinder | 1.0 | IAlarm | default |
| vendor.qti.hardware.bluetooth_audio | HIDL | hwbinder | 2.1 | IBluetoothAudioProvidersFactory | default |
| vendor.qti.hardware.bluetooth_sar | HIDL | hwbinder | 1.1 | IBluetoothSar | default |
| vendor.qti.hardware.btconfigstore | HIDL | hwbinder | 2.0 | IBTConfigStore | default |
| vendor.qti.hardware.camera.aon | HIDL | hwbinder | 1.0 | IAONService | aoncameraservice |
| vendor.qti.hardware.camera.postproc | HIDL | hwbinder | 1.0 | IPostProcService | camerapostprocservice |
| vendor.qti.hardware.dsp | HIDL | hwbinder | 1.0 | IDspService | dspservice |
| vendor.qti.hardware.eid | HIDL | hwbinder | 1.0 | IEid | default |
| vendor.qti.hardware.embmssl | HIDL | hwbinder | 1.1 | IEmbms | embmsslServer0 |
| vendor.qti.hardware.factory | HIDL | hwbinder | 1.1 | IFactory | default |
| vendor.qti.hardware.fm | HIDL | hwbinder | 1.0 | IFmHci | default |
| vendor.qti.hardware.pal | HIDL | hwbinder | 1.0 | IPAL | default |
| vendor.qti.hardware.power.powerstateservice | HIDL | hwbinder | 1.0 | IPowerStateService, IPowerStateUtility | default |
| vendor.qti.hardware.qseecom | HIDL | hwbinder | 1.0 | IQSEECom | default |
| vendor.qti.hardware.radio.internal.deviceinfo | HIDL | hwbinder | 1.0 | IDeviceInfo | deviceinfo |
| vendor.qti.hardware.secureprocessor.device | HIDL | hwbinder | 1.0 | ISecureProcessor | qti-tee |
| vendor.qti.hardware.sensorscalibrate | HIDL | hwbinder | 1.0 | ISensorsCalibrate | default |
| vendor.qti.hardware.soter | HIDL | hwbinder | 1.0 | ISoter | default |
| vendor.qti.hardware.trustedui | HIDL | hwbinder | 1.1 | ITrustedInput | default, qtee-vm |
| vendor.qti.hardware.trustedui | HIDL | hwbinder | 1.2 | ITrustedUI | default, qtee-vm |
| vendor.qti.hardware.tui_comm | HIDL | hwbinder | 1.0 | ITuiComm | default |
| vendor.qti.hardware.wifi.wifilearner | HIDL | hwbinder | 1.0 | IWifiStats | wifiStats |
| vendor.qti.qesdhal | HIDL | hwbinder | 1.1 | IQesdhal | default |
| vendor.qti.qspmhal | HIDL | hwbinder | 1.0 | IQspmhal | default |
| vendor.qti.spu | HIDL | hwbinder | 1.1, 2.0 | ISPUManager | default |

### Vendor Manifest Fragments

Source: `vendor/etc/vintf/manifest/`

| Fragment File | HAL Name | Format | Version | Interface | Instance |
|---|---|---|---|---|---|
| android.hardware.wifi@1.0-service.xml | android.hardware.wifi | HIDL | 1.5 | IWifi | default |
| android.hardware.atrace@1.0-service.xml | android.hardware.atrace | HIDL | 1.0 | IAtraceDevice | default |
| android.hardware.boot@1.2.xml | android.hardware.boot | HIDL | 1.2 | IBootControl | default |
| android.hardware.cas@1.2-service-lazy.xml | android.hardware.cas | HIDL | 1.2 | IMediaCasService | default |
| android.hardware.graphics.mapper-impl-qti-display.xml | android.hardware.graphics.mapper | HIDL (passthrough 32+64) | 4.0 | IMapper | default |
| (same file) | vendor.qti.hardware.display.mapper | HIDL (passthrough 32+64) | 4.0 | IQtiMapper | default |
| android.hardware.health@2.1.xml | android.hardware.health | HIDL | 2.1 | IHealth | default |
| android.hardware.security.keymint-service-qti.xml | android.hardware.security.keymint | AIDL | -- | IKeyMintDevice | default |
| (same file) | android.hardware.security.sharedsecret | AIDL | -- | ISharedSecret | default |
| (same file) | android.hardware.security.secureclock | AIDL | -- | ISecureClock | default |
| (same file) | android.hardware.security.keymint | AIDL | -- | IRemotelyProvisionedComponent | default |
| android.hardware.sensors@2.1-multihal.xml | android.hardware.sensors | HIDL | 2.1 | ISensors | default |
| android.hardware.thermal@2.0-service.qti.xml | android.hardware.thermal | HIDL | 1.0, 2.0 | IThermal | default |
| android.hardware.usb.gadget@1.2-service.xml | android.hardware.usb.gadget | HIDL | 1.2 | IUsbGadget | default |
| android.hardware.usb@1.2-service.xml | android.hardware.usb | HIDL | 1.2 | IUsb | default |
| android.hardware.wifi.hostapd.xml | android.hardware.wifi.hostapd | HIDL | 1.3 | IHostapd | default |
| (same file) | vendor.qti.hardware.wifi.hostapd | HIDL | 1.2 | IHostapdVendor | default |
| android.hardware.wifi.supplicant.xml | android.hardware.wifi.supplicant | HIDL | 1.4 | ISupplicant | default |
| (same file) | vendor.qti.hardware.wifi.supplicant | HIDL | 2.2 | ISupplicantVendor | default |
| c2_manifest_vendor.xml | android.hardware.media.c2 | HIDL | 1.0 | IComponentStore | default, software |
| manifest_android.hardware.drm@1.4-service.clearkey.xml | android.hardware.drm | HIDL | 1.4 | ICryptoFactory, IDrmFactory | clearkey |
| manifest_non_qmaa.xml | android.hardware.soundtrigger | HIDL | 2.3 | ISoundTriggerHw | default |
| manifest_non_qmaa_extn.xml | vendor.qti.hardware.ListenSoundModel | HIDL | 1.0 | IListenSoundModel | default |
| power.xml | android.hardware.power | AIDL | -- | IPower | default |
| vendor.qti.hardware.display.allocator-service.xml | android.hardware.graphics.allocator | HIDL | 4.0 | IAllocator | default |
| (same file) | vendor.qti.hardware.display.allocator | HIDL | 4.0 | IQtiAllocator | default |
| vendor.qti.hardware.display.composer-service.xml | android.hardware.graphics.composer | HIDL | 2.4 | IComposer | default |
| (same file) | vendor.qti.hardware.display.composer | HIDL | 3.1 | IQtiComposer | default |
| (same file) | vendor.display.config | HIDL | 2.0 | IDisplayConfig | default |
| (same file) | vendor.display.color | HIDL | 1.7 | IDisplayColor | default |
| (same file) | vendor.display.postproc | HIDL | 1.0 | IDisplayPostproc | default |
| (same file) | vendor.qti.hardware.display.config | AIDL | 5 | IDisplayConfig | default |
| vendor.qti.hardware.display.demura-service.xml | vendor.qti.hardware.display.demura | HIDL | 2.0 | IDemuraFileFinder | default |
| vendor.qti.hardware.lights.service.xml | android.hardware.light | AIDL | -- | ILights | default |
| vendor.qti.hardware.limits-service.xml | vendor.qti.hardware.limits | HIDL | 1.1 | ILimits | default |
| vendor.qti.hardware.perf.xml | vendor.qti.hardware.perf | HIDL | 2.3 | IPerf | default |
| vendor.qti.hardware.power.powermodule.xml | vendor.qti.hardware.power.powermodule | HIDL | 1.0 | IPowerModule | default |
| vendor.qti.hardware.vibrator.service.xml | android.hardware.vibrator | AIDL | 2 | IVibrator | default |
| vendor.qti.memory.pasrmanager@1.0-service.xml | vendor.qti.memory.pasrmanager | HIDL | 1.1 | IPasrManager | pasrhal |

### System (Framework) Manifest

Source: `system/system/etc/vintf/manifest.xml`

| HAL Name | Format | Transport | Version | Interface | Instance |
|---|---|---|---|---|---|
| android.frameworks.displayservice | HIDL | hwbinder | 1.0 | IDisplayService | default |
| android.frameworks.schedulerservice | HIDL | hwbinder | 1.0 | ISchedulingPolicyService | default |
| android.frameworks.sensorservice | HIDL | hwbinder | 1.0 | ISensorManager | default |
| android.hidl.manager | HIDL | hwbinder | 1.2 | IServiceManager | default |
| android.hidl.memory | HIDL | passthrough (32+64) | 1.0 | IMapper | ashmem |
| android.hidl.token | HIDL | hwbinder | 1.0 | ITokenManager | default |
| android.system.net.netd | HIDL | hwbinder | 1.1 | INetd | default |
| android.system.wifi.keystore | HIDL | hwbinder | 1.0 | IKeystore | default |
| netutils-wrapper | native | -- | 1.0 | -- | -- |
| vendor.qti.hardware.qccsyshal | HIDL | hwbinder | 1.1 | IQccsyshal | qccsyshal |
| vendor.qti.hardware.radio.atcmdfwd | HIDL | hwbinder | 1.0 | IAtCmdFwd | AtCmdFwdService |
| vendor.qti.hardware.sigma_miracast | HIDL | hwbinder | 1.0 | Isigma_miracast | sigmahal, sigmahal64 |

### System Manifest Fragments

Source: `system/system/etc/vintf/manifest/`

| Fragment File | HAL Name | Format | Version | Interface | Instance |
|---|---|---|---|---|---|
| android.frameworks.stats@1.0-service.xml | android.frameworks.stats | HIDL | 1.0 | IStats | default |
| (same file) | android.frameworks.stats | AIDL | 1 | IStats | default |
| android.hidl.allocator@1.0-service.xml | android.hidl.allocator | HIDL | 1.0 | IAllocator | ashmem |
| android.system.keystore2-service.xml | android.system.keystore2 | AIDL | -- | IKeystoreService | default |
| android.system.suspend@1.0-service.xml | android.system.suspend | HIDL | 1.0 | ISystemSuspend | default |
| manifest_android.frameworks.cameraservice.service@2.2.xml | android.frameworks.cameraservice.service | HIDL | 2.2 | ICameraService | default |
| manifest_media_c2_software.xml | android.hardware.media.c2 | HIDL | 1.2 | IComponentStore | software |

## Compatibility Matrix

Source: `vendor/etc/vintf/compatibility_matrix.xml`

Vendor NDK version: 32. System SDK version: 31.

### Required Framework HALs

| HAL Name | Version | Interface | Instance | Optional |
|---|---|---|---|---|
| android.frameworks.sensorservice | 1.0 | ISensorManager | default | No |
| android.hidl.allocator | 1.0 | IAllocator | ashmem | No |
| android.hidl.manager | 1.0 | IServiceManager | default | No |
| android.hidl.memory | 1.0 | IMapper | ashmem | No |
| android.hidl.token | 1.0 | ITokenManager | default | No |
| android.system.wifi.keystore | 1.0 | IKeystore | default | No |
| vendor.qti.hardware.qccsyshal | 1.0-1 | IQccsyshal | qccsyshal | Yes |
| vendor.qti.hardware.sigma_miracast | 1.0 | Isigma_miracast | sigmahal, sigmahal64 | Yes |
