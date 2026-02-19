# Audio

Source: `vendor/etc/audio_policy_configuration.xml`, `vendor/etc/audio_policy_volumes.xml`, `vendor/etc/mixer_paths_neo_idp.xml`, `vendor/etc/audio_effects.xml`, `vendor/etc/microphone_characteristics.xml`.

### 1.1 Audio HAL Modules

| Module | HAL Version | Purpose |
|--------|-------------|---------|
| `primary` | 2.0 | Main audio HAL (Qualcomm PAL-based) |
| `usb` | 2.0 | USB accessory audio output |
| `r_submix` | (included) | Remote submix for audio routing |
| `bluetooth_qti_hearing_aid` | (included) | QTI hearing aid BT audio |

Platform-specific HAL implementation: `audio.primary.neo.so`.

### 1.2 Attached Devices (Always Present)

| Device | Type | Format | Rate | Channels |
|--------|------|--------|------|----------|
| Earpiece | `OUT_EARPIECE` | PCM 16-bit | 48000 | Mono |
| Speaker | `OUT_SPEAKER` | PCM 16-bit | 48000 | Stereo |
| Telephony Tx | `OUT_TELEPHONY_TX` | PCM 16-bit | 8000/16000 | Mono/Stereo |
| Built-In Mic | `IN_BUILTIN_MIC` | PCM 16-bit | 16000 | 8-channel |
| Built-In Back Mic | `IN_BACK_MIC` | PCM 16-bit | 8k-48k | Mono/Stereo/Front-Back |
| FM Tuner | `IN_FM_TUNER` | PCM 16-bit | 48000 | Mono/Stereo |
| Telephony Rx | `IN_TELEPHONY_RX` | PCM 16-bit | 8k/16k/48k | Mono |

Default output device: **Speaker**.

### 1.3 Removable/Optional Devices

**Output:**
- Wired Headset / Wired Headphones / Line Out -- PCM 16-bit, 48 kHz, stereo
- BT SCO / BT SCO Headset / BT SCO Car Kit -- PCM 16-bit, 8/16 kHz, mono
- BT A2DP Out / A2DP Headphones / A2DP Speaker -- PCM 16-bit, 48 kHz, stereo. Encoded formats: SBC, AAC, aptX, aptX HD, LDAC, CELT, aptX Adaptive, aptX TWS+, LC3
- HDMI (AUX Digital) -- PCM 16-bit, 8k-192k
- USB Device Out / USB Headset Out -- PCM 16-bit, 44.1k-192k
- USB Host Out (accessory) -- PCM 16-bit, 44.1 kHz, stereo

**Input:**
- Wired Headset Mic -- PCM 16-bit, 8k-48k, up to front-back
- BT SCO Headset Mic -- PCM 16-bit, 8k/16k, mono
- USB Device In / USB Headset In -- unspecified (dynamic)
- A2DP In -- PCM 16-bit, 44.1k/48k, mono/stereo; encoded: LC3

### 1.4 Output Streams (Mix Ports)

| Stream | Flags | Format | Rate | Channels |
|--------|-------|--------|------|----------|
| `primary output` | FAST, PRIMARY | PCM 16-bit | 48000 | Stereo |
| `raw` | FAST, RAW | PCM 16-bit | 48000 | Stereo |
| `deep_buffer` | DEEP_BUFFER | PCM 16-bit | 48000 | Stereo |
| `haptics output` | -- | PCM 16-bit | 48000 | Stereo + Haptic A |
| `direct_pcm` | DIRECT | PCM 16/24/32-bit | 8k-384k | Mono through 7.1 |
| `compressed_offload` | DIRECT, COMPRESS, NON_BLOCKING, GAPLESS | Multiple (see below) | varies | Mono-7.1 |
| `compress_passthrough` | DIRECT, COMPRESS, NON_BLOCKING | -- | -- | -- |
| `mmap_no_irq_out` | DIRECT, MMAP_NOIRQ | PCM 16-bit | 48000 | Stereo |
| `hifi_playback` | -- | unspecified | -- | -- |
| `voice_tx` | -- | PCM 16-bit | 8k/16k/48k | Mono/Stereo |
| `voip_rx` | DIRECT, VOIP_RX | PCM 16-bit | 8k-48k | Mono |
| `incall_music_uplink` | INCALL_MUSIC | PCM 16-bit | 8k/16k/48k | Stereo |

### 1.5 Input Streams (Mix Ports)

| Stream | Flags | Max Open | Format | Rate | Channels |
|--------|-------|----------|--------|------|----------|
| `primary input` | -- | 2 | PCM 16-bit | 8k-48k | Mono/Stereo/Front-Back/4ch/8ch |
| `fast input` | FAST | -- | PCM 16-bit | 8k-48k | Mono/Stereo/Front-Back |
| `quad mic` | -- | -- | PCM 16-bit | 48000 | 4-channel index mask |
| `compress-input` | DIRECT | -- | AAC-LC | 8k-48k | Mono/Stereo/Front-Back |
| `voip_tx` | VOIP_TX | -- | PCM 16-bit | 8k-48k | Mono |
| `usb_surround_sound` | -- | -- | PCM 16/32-bit, float | 8k-192k | Up to 8-channel |
| `record_24` | -- | 2 | PCM 24-bit, 8_24, float | 8k-192k | Mono/Stereo/Front-Back/3ch/4ch |
| `voice_rx` | -- | -- | PCM 16-bit | 8k/16k/48k | Mono/Stereo |
| `mmap_no_irq_in` | MMAP_NOIRQ | -- | PCM 16-bit | 8k-48k | Mono/Stereo/Front-Back/3ch |
| `hifi_input` | -- | -- | unspecified | -- | -- |

### 1.6 Compressed Offload Formats

Supported formats for hardware-accelerated decode:
- MP3 (8k-48k, mono/stereo)
- FLAC (8k-192k, mono/stereo)
- ALAC (8k-192k, mono through 7.1)
- APE (8k-192k, mono/stereo)
- AAC-LC / AAC-HE-v1 / AAC-HE-v2 (8k-96k, mono/stereo)
- AAC-ADTS-LC / AAC-ADTS-HE-v1 / AAC-ADTS-HE-v2 (8k-96k, mono/stereo)
- DTS (32k/44.1k/48k, mono through 5.1)
- DTS-HD (32k-192k, mono through 7.1)
- WMA / WMA Pro (8k-96k, mono through 7.1)
- Vorbis (8k-192k, mono/stereo)

### 1.7 Volume Curves

Volume curves defined per stream type and device category. Key custom curves (millibel attenuation at volume index 0/33/66/100):

| Stream | Category | Curve |
|--------|----------|-------|
| Voice Call | Headset | -4200 / -2800 / -1400 / 0 |
| Voice Call | Speaker | -2400 / -1600 / -800 / 0 |
| Voice Call | Earpiece | -2400 / -1600 / -800 / 0 |
| Ring | Speaker | -2970 / -2010 / -1020 / 0 |
| Alarm | Speaker | -2970 / -2010 / -1020 / 0 |
| Notification | Speaker | -2970 / -2010 / -1020 / 0 |
| BT SCO | Headset | -4200 / -2800 / -1400 / 0 |
| BT SCO | Speaker | -2400 / -1600 / -800 / 0 |
| System | Headset/Speaker | FULL_SCALE (no attenuation) |
| TTS | Speaker | FULL_SCALE |
| TTS | Headset/Earpiece/Ext | SILENT |

Global config: `speaker_drc_enabled="true"`, `call_screen_mode_supported="true"`.

### 1.8 Microphones

4 built-in microphones, all omnidirectional, sensitivity -37.0 dBFS, max SPL 132.5 dB, min SPL 28.5 dB:

| Mic ID | Type | Location | Frequency Response Points |
|--------|------|----------|--------------------------|
| `builtin_mic_1` | `IN_BUILTIN_MIC` | bottom | 93 points (100 Hz - 20 kHz) |
| `builtin_mic_2` | `IN_BACK_MIC` | back | 92 points (106 Hz - 20 kHz) |
| `builtin_mic_3` | `IN_BUILTIN_MIC` | mainbody | 92 points (100 Hz - 19 kHz) |
| `builtin_mic_4` | `IN_BACK_MIC` | mainbody | 92 points (106 Hz - 20 kHz) |

Mic routing by sound device:
- **Handset mic** (PAL_DEVICE_IN_HANDSET_MIC): mic 1
- **Speaker mic** (PAL_DEVICE_IN_SPEAKER_MIC): mic 1 + mic 2
- **Voice assistant mic** (PAL_DEVICE_IN_HANDSET_VA_MIC): mic 1 + mic 2 + mic 3
- **Ultrasound mic** (PAL_DEVICE_IN_ULTRASOUND_MIC): all 4 mics

### 1.9 Mixer Paths

Three mixer path configurations for different board variants:
- `mixer_paths_neo_idp.xml` -- standard IDP board
- `mixer_paths_neo_idp_sg.xml` -- IDP board (SG variant)
- `mixer_paths_neo_qxr.xml` -- QXR (XR-specific) board

WSA (Wall Smart Amplifier) speaker configuration: WSA_SPKR_VI_2 and WSA2_SPKR_VI_1 mixer controls present.

### 1.10 Audio Effects

| Effect | Library | UUID |
|--------|---------|------|
| Bass Boost | bundle / offload_bundle (proxied) | 8631f300 / 2c4a8c24 |
| Virtualizer | bundle / offload_bundle (proxied) | 1d4033c0 / 509a4498 |
| Equalizer | bundle / offload_bundle (proxied) | ce772f20 / a0dac280 |
| Volume | bundle | 119341a0 |
| Reverb (4 variants) | reverb / offload_bundle (proxied) | multiple |
| Visualizer | visualizer_sw / visualizer_hw (proxied) | d069d9e0 / 7a8044a0 |
| Downmix | downmix | 93f04452 |
| Loudness Enhancer | loudness_enhancer | fa415329 |
| Dynamics Processing | dynamics_processing | e0e6539b |
| AEC | audio_pre_processing | 0f8d0d2a |
| Noise Suppression | audio_pre_processing | 1d97bb0b |
| Audiosphere | audiosphere | 184e62ab |

Post-processing applied automatically: volume listeners on music, ring, alarm, voice call, and notification streams.
Pre-processing: AEC + NS on voice_communication stream.

Rokid-specific: `libRokidAudioEffects.so` (vendor/lib64 and vendor/lib).

### 1.11 Audio Resource Manager

File: `vendor/etc/audio/sku_neo/resourcemanager_neo_idp.xml`.

- Native audio mode: `multiple_mix_dsp`
- Max sessions: 128
- Max non-tunnel sessions: 4
- HiFi filter: disabled
- Dual mono: disabled
- BT sink HFP channel config: 2 channels
- UPD dedicated backend: enabled

### 1.12 Sound Trigger

Hardware implementation: `sound_trigger.primary.neo.so`.
ACD (Audio Context Detection) models in `vendor/etc/models/acd/`:
- `event.eai` -- event detection
- `music.eai` -- music detection
- `speech.eai` -- speech detection

### 1.13 Audio Codec (Kernel Modules)

From `vendor_dlkm/lib/modules/`:
- `wcd937x_dlkm.ko` / `wcd937x_slave_dlkm.ko` -- WCD9370/WCD9375 codec
- `wcd938x_dlkm.ko` / `wcd938x_slave_dlkm.ko` -- WCD9380/WCD9385 codec
- `wcd9xxx_dlkm.ko` / `wcd_core_dlkm.ko` -- WCD codec core
- `wsa883x_dlkm.ko` -- WSA883x smart amplifier
- `lpass_cdc_dlkm.ko` -- LPASS internal codec
- `lpass_cdc_rx_macro_dlkm.ko` -- RX macro (playback)
- `lpass_cdc_tx_macro_dlkm.ko` -- TX macro (capture)
- `lpass_cdc_va_macro_dlkm.ko` -- VA macro (voice assistant)
- `lpass_cdc_wsa_macro_dlkm.ko` / `lpass_cdc_wsa2_macro_dlkm.ko` -- WSA macros
- `mbhc_dlkm.ko` -- Multi-Button Headset Control
- `swr_dlkm.ko` / `swr_ctrl_dlkm.ko` -- SoundWire controller
- `swr_dmic_dlkm.ko` -- SoundWire digital mic
- `swr_haptics_dlkm.ko` -- SoundWire haptics
