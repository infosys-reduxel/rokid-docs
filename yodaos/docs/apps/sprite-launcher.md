# RokidSpriteLauncher

The main launcher and home screen application for Rokid AR glasses. This is the system launcher -- it serves as the home screen, hosts all built-in feature pages (camera, translate, navigation, chat, music, etc.), manages companion phone connectivity via Bluetooth, and controls the app lifecycle on the glasses.

## Package Info

| Field | Value |
|---|---|
| Package | `com.rokid.os.sprite.launcher` |
| Version | `0.2.4` (versionCode 2411) |
| Min SDK | 29 (Android 10) |
| Target SDK | 34 (Android 14) |
| Shared UID | `android.uid.system` (runs as system app) |
| Application class | `com.rokid.os.sprite.launcher.MainApplication` |
| Language | Kotlin (compiled from `.kt` sources) |
| Architecture | arm64-v8a only |

## Permissions

The app requests a broad set of permissions reflecting its system-level role:

**Network**: `INTERNET`, `ACCESS_NETWORK_STATE`, `ACCESS_WIFI_STATE`

**Storage**: `READ_EXTERNAL_STORAGE`, `WRITE_EXTERNAL_STORAGE`, `MANAGE_EXTERNAL_STORAGE`, `READ_MEDIA_IMAGES`, `ACCESS_MEDIA_LOCATION`

**Camera/Audio**: `CAMERA`, `RECORD_AUDIO`

**Bluetooth**: `BLUETOOTH`, `BLUETOOTH_ADMIN`, `BLUETOOTH_CONNECT`

**System**: `INSTALL_PACKAGES`, `REQUEST_INSTALL_PACKAGES`, `DELETE_PACKAGES`, `REQUEST_DELETE_PACKAGES`, `INSTALL_PACKAGE_UPDATES`, `FORCE_STOP_PACKAGES`, `WRITE_SETTINGS`, `GET_TASKS`, `READ_PHONE_STATE`, `READ_PRIVILEGED_PHONE_STATE`, `WAKE_LOCK`, `DISABLE_KEYGUARD`, `MANAGE_SHORTCUTS`, `READ_SHORTCUTS`

Because it runs with `android.uid.system`, many of these are granted automatically at the system level.

## Activities (21 total)

All activities are `exported="true"` and use `launchMode="singleTask"`. None have explicit intent filters (except the main launcher), meaning they are launched programmatically via explicit intents from within the launcher or from the companion phone via Bluetooth commands.

### 1. SpriteMainActivity -- Home Screen / Launcher

**Class**: `com.rokid.os.sprite.launcher.main.SpriteMainActivity`

**Intent filter** (the only activity with one):
```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
    <category android:name="android.intent.category.LAUNCHER" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.HOME" />
</intent-filter>
```

This is the device's home screen. It registers as both a regular launcher and a leanback (TV-style) launcher. Contains the main app grid, center panel with journey/plan info, message section, and the app list.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.main.SpriteMainActivity
```

**Key behavior**: Registers `ExternalCmdReceiver`, `AccountReceiver`, and `AppInstallReceiver` as broadcast receivers at runtime. Manages the global status bar. Handles `configChanges` for keyboard, orientation, screen size, MCC/MNC changes.

### 2. CameraPageActivity -- Camera (Photo/Video)

**Class**: `com.rokid.os.sprite.launcher.page.camera.CameraPageActivity`

**Scene ID**: `video_record` (from `SceneStatusData.SCENE_VIDEO_RECORD`)

Extends `CameraXPageActivity`, which is an abstract base that handles CameraX permissions (camera, audio, media). Runs in a separate task affinity (`com.rokid.sprite.taskaffinity.camera`) so it can operate independently. Uses `turnScreenOn="true"` to wake the display.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.camera.CameraPageActivity
```

**Native libs**: `libimage_processing_util_jni.so` (CameraX image processing)

### 3. ChatPageActivity -- AI Chat

**Class**: `com.rokid.os.sprite.launcher.page.chat.ChatPageActivity`

**Scene ID**: `ai_chat` (from `SceneStatusData.SCENE_AI_CHAT`)

Conversational AI chat interface. Communicates with the companion phone's AI app service via Bluetooth. Has sub-packages for chat beans (data models) and managers (chat lifecycle).

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.chat.ChatPageActivity
```

### 4. TranslatePageActivity -- Real-time Translation

**Class**: `com.rokid.os.sprite.launcher.page.translate.TranslatePageActivity`

**Scene ID**: `translate` (from `SceneStatusData.SCENE_TRANSLATE`)

Real-time translation overlay. Displays translated text in a RecyclerView that auto-scrolls. Supports two audio modes: cardioid (directional) and omnidirectional. Has an anti-light-leakage mode that dims brightness to minimum (level 2) and hides the status bar to reduce light bleed. Uses EventBus for events: `TransLanguageEvent`, `TransReadyParamEvent`, `TransSetTvConfigParamEvent`.

Content positioning is configurable via `ContentSite` (startPointX, startPointY, width, height), persisted in SharedPreferences under key `translate_content_site`.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.translate.TranslatePageActivity
```

### 5. WordTipsPageActivity -- Teleprompter / Word Tips

**Class**: `com.rokid.os.sprite.launcher.page.wordtips.WordTipsPageActivity`

**Scene ID**: `word_tips` (from `SceneStatusData.SCENE_WORD_TIPS`)

Teleprompter-style text display with two modes: **Normal mode** (NormalModeFragment) for static scripts and **AI mode** (AiModeFragment) for AI-generated prompts. Features pinyin matching for Chinese text, Latin fuzzy matching, anti-light-leakage mode, configurable text size, and auto-sleep after 24 hours of AI mode inactivity.

Reads prompt files from assets (`prompt/en_demo.txt`, `prompt/zh_demo.txt`) or from SD card. Content positioning persisted under key `word_tips_content_site`.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.wordtips.WordTipsPageActivity
```

### 6. MusicPageActivity -- Music Player

**Class**: `com.rokid.os.sprite.launcher.page.music.MusicPageActivity`

**Scene ID**: `music_word` (from `SceneStatusData.SCENE_MUSIC_WORD`)

Music playback display with lyrics (word display). Has adapters, data models, and custom widgets for the music UI.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.music.MusicPageActivity
```

### 7. NavigationPageActivity -- Navigation (China)

**Class**: `com.rokid.os.sprite.launcher.page.navigation.NavigationPageActivity`

**Scene ID**: `navigation`

Turn-by-turn navigation for China region. Receives navigation data via Bluetooth from the companion phone's navigation app. Uses `GattNavigationManager` for Bluetooth data flow. Native library `libnavigation.so` for map rendering. Assets include lane info icons and navigation icons (car, ride, walk, drive).

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.navigation.NavigationPageActivity
```

### 8. NavigationOverseaPageActivity -- Navigation (Overseas)

**Class**: `com.rokid.os.sprite.launcher.page.navigation.NavigationOverseaPageActivity`

Same navigation feature for non-China regions. The `MainApplication` determines which navigation class to use via `getNaviActivityClass()`, likely based on locale or device configuration. Supports EventBus events: `NaviStart`, `NaviStop`, `NaviUpdateInfo`, `NaviShowMode`, `NaviMapData`, `NaviUpdateLocPermissionTip`. Displays estimated arrival time, distance, turn icons, and map data.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.navigation.NavigationOverseaPageActivity
```

### 9. PaymentPageActivity -- QR Code Payment

**Class**: `com.rokid.os.sprite.launcher.page.payment.PaymentPageActivity`

**Scene ID**: `payment`

Displays QR code for payment. Managed by `GattPaymentManager` which receives payment data via Bluetooth from the companion phone.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.payment.PaymentPageActivity
```

### 10. AudioPageActivity -- Audio Recording

**Class**: `com.rokid.os.sprite.launcher.page.audio.AudioPageActivity`

**Scene ID**: `audio_record` (from `SceneStatusData.SCENE_AUDIO_RECORD`)

Audio recording interface with visual waveform animation. Auto-finishes after 5 seconds of MAX_FINISH_TIME. Uses `turnScreenOn="true"`.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.audio.AudioPageActivity
```

### 11. CaexpoPageActivity -- CAEXPO Content Display

**Class**: `com.rokid.os.sprite.launcher.page.caexpo.CaexpoPageActivity`

Markdown content reader, likely built for the China-ASEAN Expo (CAEXPO) trade show. Renders markdown content from assets (`caexpo/en_caexpo.md`, `caexpo/zh_caexpo.md`) using the Markwon library. Supports paged scrolling with page indicators and left/right navigation.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.caexpo.CaexpoPageActivity
```

### 12. StorageImageShowActivity -- Image Gallery

**Class**: `com.rokid.os.sprite.launcher.page.gallery.StorageImageShowActivity`

Image viewer/gallery using ViewPager2 with zoom support (ZoomImageViewV3). Supports JPG, PNG, JPEG, WebP, BMP, and GIF. Has orientation detection for gyroscope-based display rotation. Not a PageActivity -- extends BaseActivity directly.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.gallery.StorageImageShowActivity
```

### 13. SettingPageActivity -- Settings Hub

**Class**: `com.rokid.os.sprite.launcher.setting.SettingPageActivity`

Main settings screen. Horizontal scrolling list of setting items (uses `GridLayoutManager` with horizontal orientation). Displays Bluetooth connection status. From here, users navigate to sub-setting screens. Ignores `SCREEN_ON` auto-finish events (stays open when screen turns on).

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.setting.SettingPageActivity
```

### 14. SettingBluetoothActivity -- Bluetooth Settings

**Class**: `com.rokid.os.sprite.launcher.setting.bluetooth.SettingBluetoothActivity`

Bluetooth pairing and connection management. Shows connected device name and status. Uses `PhoneBluetoothManager` for device discovery and connection. Includes a peripheral visibility toggle.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.setting.bluetooth.SettingBluetoothActivity
```

### 15. BluetoothRepairActivity -- Bluetooth Repair Dialog

**Class**: `com.rokid.os.sprite.launcher.setting.bluetooth.BluetoothRepairActivity`

Confirmation dialog for Bluetooth repair/reset. Two-button layout (OK/Cancel) with focus-based navigation. Calls `AssistServerUnity` to perform the repair action.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.setting.bluetooth.BluetoothRepairActivity
```

### 16. SettingSystemInfoActivity -- System Information

**Class**: `com.rokid.os.sprite.launcher.setting.info.SettingSystemInfoActivity`

**Page ID**: `settings` (from `SceneStatusData.PAGE_SETTINGS`)

Displays device information: device name, firmware version, connected peripheral info. Can trigger factory reset (`android.intent.action.FACTORY_RESET`). Shows device name from `RKSettingsManager`.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.setting.info.SettingSystemInfoActivity
```

### 17. SystemResetActivity -- Factory Reset Confirmation

**Class**: `com.rokid.os.sprite.launcher.setting.info.SystemResetActivity`

Factory reset confirmation dialog. Two-button OK/Cancel layout. Sends `android.intent.action.FACTORY_RESET` broadcast with extras for wiping eSIMs and external storage.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.setting.info.SystemResetActivity
```

### 18. SettingSoundEffectsActivity -- Sound Effects Toggle

**Class**: `com.rokid.os.sprite.launcher.setting.sound.SettingSoundEffectsActivity`

Toggle for system sound effects (navigation clicks, focus sounds). Writes to Android `Settings.System` provider. Uses FuncDebounce to prevent rapid toggling.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.setting.sound.SettingSoundEffectsActivity
```

### 19. SettingBrightnessActivity -- Brightness Control

**Class**: `com.rokid.os.sprite.launcher.page.brightness.SettingBrightnessActivity`

**Page ID**: `brightness` (from `SceneStatusData.PAGE_BRIGHTNESS`)

Brightness slider/control. Uses left/right keys to increment/decrement brightness. Listens for `BrightnessChangeEvent` via EventBus. Reports scene status to `AssistServerUnity`.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.brightness.SettingBrightnessActivity
```

### 20. SettingVolumeActivity -- Volume Control

**Class**: `com.rokid.os.sprite.launcher.page.volume.SettingVolumeActivity`

Volume slider/control. Uses Android `AudioManager` for stream volume. Registers a broadcast receiver for `android.media.VOLUME_CHANGED_ACTION`. Left/right keys adjust volume.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.page.volume.SettingVolumeActivity
```

### 21. TestUiActivity -- Developer Test Screen

**Class**: `com.rokid.os.sprite.launcher.test.TestUiActivity`

Minimal test activity for UI development. Page identifier: `test`. Empty shell with just a status bar.

**Launch from ADB**:
```bash
adb shell am start -n com.rokid.os.sprite.launcher/.test.TestUiActivity
```

## Navigation Model

### Activity Hierarchy

All feature activities (except `StorageImageShowActivity`) extend the `PageActivity` base class, which in turn extends `BaseActivity`. This provides:

- Global status bar management (`configGlobalStatusBar()`, `showStatusBar()`)
- Short status bar configuration (`configShortStatusBar()`)
- Auto-finish behavior (activities auto-close on certain system events)
- App connection checking (`checkAppConnect()` -- verifies companion phone Bluetooth connection)
- EventBus registration/unregistration lifecycle
- Focus audio management for button navigation sounds

### How Users Navigate

The glasses use **button-only navigation** (no touchscreen). Key input handling:

- **Left/Right**: Move between items in horizontal lists (settings, main app grid)
- **Up/Down**: Move between items in vertical lists (translate text, word tips)
- **Enter/OK**: Confirm selection, open feature
- **Back**: Return to previous screen / exit feature

### How Features Are Launched

Features are launched through three pathways:

**1. User selection on home screen** -- SpriteMainActivity displays an app grid and center panel. User focuses on an item and presses Enter.

**2. Companion phone commands via Bluetooth** -- The `GattNormalManager` receives commands from the companion phone over BLE. Data is parsed and routed to `ControlSceneManager.controlSceneAppData()`, which dispatches to the appropriate activity based on command type:

| Command type | Name value | Activity |
|---|---|---|
| `scene` | `ai_chat` | ChatPageActivity |
| `scene` | `translate` | TranslatePageActivity |
| `scene` | `word_tips` | WordTipsPageActivity |
| `scene` | `music_word` | MusicPageActivity |
| `scene` | `navigation` | NavigationPageActivity or NavigationOverseaPageActivity |
| `scene` | `ai_assist` | AiDialogManager (overlay dialog, not activity) |
| `page` | `settings` | SettingSystemInfoActivity |
| `page` | `caexpo` | CaexpoPageActivity |
| `app` | (package name) | External app launch |

**3. External broadcast commands** -- `ExternalCmdReceiver` listens for two broadcast actions:

| Broadcast Action | Purpose |
|---|---|
| `com.rokid.os.sprite.launcher.cmd` | General command dispatch. Extras: `cmd=open_app`, `uri=<uri>`, `pkg=<package>` |
| `com.rokid.visualaidemo.ACTION_START` | Opens Visual AI demo mode |

Example of launching via broadcast:
```bash
adb shell am broadcast -a com.rokid.os.sprite.launcher.cmd \
    --es cmd "open_app" --es pkg "com.example.app"
```

### Settings Navigation Flow

```
SettingPageActivity (hub)
  -> SettingBluetoothActivity -> BluetoothRepairActivity
  -> SettingSystemInfoActivity -> SystemResetActivity
  -> SettingSoundEffectsActivity
  -> SettingBrightnessActivity
  -> SettingVolumeActivity
```

## Broadcast Receivers

### Registered in Manifest

| Receiver | Actions | Purpose |
|---|---|---|
| `androidx.profileinstaller.ProfileInstallReceiver` | `*.INSTALL_PROFILE`, `*.SKIP_FILE`, `*.SAVE_PROFILE`, `*.BENCHMARK_OPERATION` | Baseline profile management (AndroidX) |

### Registered at Runtime (in MainApplication / SpriteMainActivity)

| Receiver | Actions | Purpose |
|---|---|---|
| `ExternalCmdReceiver` | `com.rokid.os.sprite.launcher.cmd`, `com.rokid.visualaidemo.ACTION_START` | External command dispatch for opening apps and Visual AI |
| `AccountReceiver` | `com.rokid.erbox.action.ACCOUNT` | Login/logout events (extra: `loginStatus` boolean) |
| `AppInstallReceiver` | `PACKAGE_ADDED`, `PACKAGE_REPLACED`, `PACKAGE_REMOVED` | Tracks app installations/removals to update app grid |

## Content Providers

| Provider | Authority | Purpose |
|---|---|---|
| `FileProvider` | `com.rokid.os.sprite.launcher.provider` | File sharing (camera images, etc.). Exposes external, root, files, cache, external-files, external-cache, external-media paths |
| `InitializationProvider` | `com.rokid.os.sprite.launcher.androidx-startup` | AndroidX Startup: EmojiCompat, ProcessLifecycle, ProfileInstaller |

## Services

| Service | Purpose |
|---|---|
| `MetadataHolderService` | CameraX Camera2 default config provider (disabled by default) |
| `MultiInstanceInvalidationService` | AndroidX Room database cache invalidation |

## Native Libraries

| Library | Purpose |
|---|---|
| `libnavigation.so` | Map rendering for navigation feature |
| `librfm_lang.so` / `librfm_lang_jni.so` | Language detection/processing (likely for translate feature) |
| `libimage_processing_util_jni.so` | CameraX image processing |
| `libc++_shared.so` | C++ standard library (shared) |

## Assets

| Path | Purpose |
|---|---|
| `caexpo/en_caexpo.md`, `caexpo/zh_caexpo.md` | CAEXPO exhibition content (Markdown) |
| `prompt/en_demo.txt`, `prompt/zh_demo.txt` | Default teleprompter/word tips scripts |
| `navi/` | Navigation icons: car, ride, walk, drive, route markers, lane info overlays |
| `navi/laneInfo/` | Lane guidance overlay images (front/back compositing) |
| `cncity.txt` | Chinese city database (for navigation) |
| `duoyinzi_dict.json` | Chinese polyphonic character dictionary (for pinyin matching in word tips) |
| `dexopt/baseline.prof` | Baseline profile for ART optimization |

## Key Manager Classes

| Manager | Package | Purpose |
|---|---|---|
| `ControlSceneManager` | `global.status` | Central dispatcher -- opens/closes feature activities based on scene commands |
| `GattNormalManager` | `global.normal` | BLE status and connection management, device type tracking |
| `ModuleAppManager` | `global.normal` | Third-party module app lifecycle (recording, video modules) |
| `AssistManager` | `global.assist` | Bridge to `AssistServerUnity` for scene status reporting |
| `TranslateManager` | `page.translate.manager` | Translation session lifecycle (start/stop translation) |
| `WortTipsGattManager` | `page.wordtips` | Word tips Bluetooth data flow |
| `GattNavigationManager` | `page.navigation` | Navigation Bluetooth data flow |
| `GattPaymentManager` | `page.payment` | Payment QR code Bluetooth data flow |
| `AiDialogManager` | `page.ai.manager` | AI assistant overlay dialog management |
| `AIModeManager` | `page.ai.manager` | AI assistant mode lifecycle |
| `NetRequestManager` | `global.cms` | CMS/network request management, periodic status checks |
| `PhoneBluetoothManager` | `setting.bluetooth` | Bluetooth device discovery and connection |
| `AppStoreConfig` | `global.appstore` | App store configuration and download engine |

## Integration Points with Other Rokid Apps

### Settings App (`com.rokid.erbox.settings`)
Referenced in `ErConfig.SETTING_APP_PACKAGE`. The launcher may delegate some system settings to this separate settings app.

### AI App (`com.rokid.sprite.global.aiapp`)
The `ExternalCmdReceiver.ACTION_APP_VISUAL_AI` (`com.rokid.visualaidemo.ACTION_START`) integrates with the AI app for visual AI features. The `TranslatePageActivity.onAutoFinish()` checks for this action to determine auto-close behavior.

### Account Service
`AccountReceiver` listens for `com.rokid.erbox.action.ACCOUNT` broadcasts to handle login/logout state changes from the Rokid account system.

### Companion Phone App (via CXR-M Bluetooth)
All feature pages that require data from the phone (navigation, translate, music, chat, payment, word tips) communicate via Bluetooth through their respective `Gatt*Manager` classes. The `GattNormalManager` tracks overall BLE connection status and device type. The `AppConnectStatus` LiveData notifies the UI of connection state changes.

### AssistServerUnity
Almost every feature activity reports its scene status to `AssistServerUnity` on open/close:
```
AssistServerUnity.updateSceneStatus(SceneStatusData.SCENE_TRANSLATE, true/false)
```
This is the bridge to the system's scene management framework.

## Event Bus Events

The app uses GreenRobot EventBus extensively for internal communication:

| Event | Source | Purpose |
|---|---|---|
| `TransLanguageEvent` | TranslateManager | Language pair update (from/to language names) |
| `TransReadyParamEvent` | TranslateManager | Translation readiness state |
| `TransSetTvConfigParamEvent` | TranslateManager | Text display configuration (text size, content area) |
| `ChangeTvConfigEvent` | WordTips | Word tips display configuration change |
| `WordTipsAiEvent` | WordTips | AI mode text events |
| `WordContentResetEvent` | WordTips | Content reset trigger |
| `WtTurnPageEvent` | WordTips | Page turn in word tips |
| `NaviStart` / `NaviStop` | Navigation | Navigation session lifecycle |
| `NaviUpdateInfo` | Navigation | Turn-by-turn update (distance, direction) |
| `NaviShowMode` | Navigation | Map view mode change (overview/follow) |
| `NaviMapData` | Navigation | Map tile/image data |
| `BrightnessChangeEvent` | System | External brightness change notification |
| `AutoFinishEvent` | BaseActivity | Activity auto-close trigger |
| `ToastEvent` | Various | Toast message display |
| `MainEvent` / `MainFocusEvent` | SpriteMainActivity | Home screen focus management |
| `FragSelectEvent` | Main fragments | Fragment selection events |
| `AppDefaultFocusEvent` | Main | Default focus restore |

## Utility and Configuration

### RKSettingsManager Keys
- `KEY_SETTINGS_TRANSLATE_AUDIO_MODE` -- cardioid vs omnidirectional mic mode
- `VALUE_TRANSLATE_AUDIO_MODE_ORIENTATION` -- directional (cardioid) mode value

### SharedPreferences Keys
- `translate_content_site` -- JSON-serialized ContentSite for translate overlay position
- `word_tips_content_site` -- JSON-serialized ContentSite for word tips overlay position
- `word_tips_ai_config` -- JSON-serialized WtAiConfigParam for AI mode configuration
- `word_tips_tv_size` -- Text size for word tips
- `word_tips_PINYIN_MATCH` -- Pinyin matching toggle
- `word_tips_SHOW_ASR` -- ASR display toggle
- `word_tips_anti_light_leakage` -- Anti-light-leakage toggle for word tips
- `translate_anti_light_leakage` -- Anti-light-leakage toggle for translate
- `global_qrtip_needshow` -- QR tip display flag
- `global_mouse_assist_need_show` -- Mouse assist indicator flag

### ContentSite Configuration
Defines where overlay content (translate, word tips) renders on the 480px-wide display:
- Translate default: x=40, y=226, w=400, h=224 (max space: 480x464)
- Word Tips default: x=40, y=60, w=400, h=210 (max space: 480x490)

### Display Constants
- Screen height reference: 460px (used in translate background selection logic)
- Anti-light-leakage brightness: level 2 (minimum usable)
