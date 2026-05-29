# Button Broadcasts (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/57e35cd3ae294d16b1b8fc8dcbb1b7c7/pc/cn/13083daf77dd40bf84cf5c59711e987a.html> (Chinese, fetched 2026-05-29)
>
> **Doc version: v0.0.1 (2026-03-01)**

This page covers the system-function buttons on Rokid Glasses and how a bare-metal Android app intercepts them. See the [Bare-Metal Development Guide](./development-guide.md) for context and the list of system interactions you **cannot** override (long-press touchpad → AI module, double-click right button → back, top button → camera).

## System-function buttons on Rokid Glasses

On Rokid Glasses, system button events are delivered as **ordered broadcasts**. Register a `BroadcastReceiver` with `priority = 100` to intercept and consume them; call `abortBroadcast()` so the default system handler does not also fire.

### Action string reference

The 11 action strings emitted by the Sprite key driver:

| `KeyType` constant | Intent action | Trigger |
|--------------------|---------------|---------|
| `CLICK` | `com.android.action.ACTION_SPRITE_BUTTON_CLICK` | Single click on the right-temple side button |
| `BUTTON_DOWN` | `com.android.action.ACTION_SPRITE_BUTTON_DOWN` | Side button pressed (down edge) |
| `BUTTON_UP` | `com.android.action.ACTION_SPRITE_BUTTON_UP` | Side button released (up edge) |
| `DOUBLE_CLICK` | `com.android.action.ACTION_SPRITE_BUTTON_DOUBLE_CLICK` | Double-click — reserved for **back/exit** (see note below) |
| `AI_START` | `com.android.action.ACTION_AI_START` | Long-press of the right-temple touchpad — launches the on-device Rokid AI module |
| `LONG_PRESS` | `com.android.action.ACTION_SPRITE_BUTTON_LONG_PRESS` | Long press on the side button |
| `ACTION_TWO_FINGER_SINGLE_TAP` | `com.android.action.ACTION_TWO_FINGER_SINGLE_TAP` | Two-finger single tap on the touchpad |
| `ACTION_TWO_FINGER_DOUBLE_TAP` | `com.android.action.ACTION_TWO_FINGER_DOUBLE_TAP` | Two-finger double tap on the touchpad |
| `ACTION_TWO_FINGER_SWIPE_FORWARD` | `com.android.action.ACTION_TWO_FINGER_SWIPE_FORWARD` | Two-finger forward swipe on the touchpad |
| `ACTION_TWO_FINGER_SWIPE_BACK` | `com.android.action.ACTION_TWO_FINGER_SWIPE_BACK` | Two-finger back swipe on the touchpad |
| `ACTION_SETTINGS_KEY` | `com.android.action.ACTION_SETTINGS_KEY` | Settings key (two-finger long-press, per the reference code's log message) |

> **Note on `DOUBLE_CLICK`:** Per the source comment, this event cannot be fully intercepted — the system reserves the double-click on the right-temple button for the global back/exit action. `abortBroadcast()` will stop downstream receivers in your process, but the back semantic is still applied by the platform.

### 1. Button broadcasts — receiver

The button broadcast definitions are as follows:

```kotlin
package com.rokid.cxrssdksamples.activities.keys

import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent

interface KeyReceiverListener {
    fun onReceive(keyType: KeyType)
}

enum class KeyType(val action: String) {
    CLICK("com.android.action.ACTION_SPRITE_BUTTON_CLICK"),
    BUTTON_DOWN("com.android.action.ACTION_SPRITE_BUTTON_DOWN"),
    BUTTON_UP("com.android.action.ACTION_SPRITE_BUTTON_UP"),
    DOUBLE_CLICK("com.android.action.ACTION_SPRITE_BUTTON_DOUBLE_CLICK"),
    AI_START("com.android.action.ACTION_AI_START"),
    LONG_PRESS("com.android.action.ACTION_SPRITE_BUTTON_LONG_PRESS"),
    ACTION_TWO_FINGER_SINGLE_TAP("com.android.action.ACTION_TWO_FINGER_SINGLE_TAP"),
    ACTION_TWO_FINGER_DOUBLE_TAP("com.android.action.ACTION_TWO_FINGER_DOUBLE_TAP"),
    ACTION_TWO_FINGER_SWIPE_FORWARD("com.android.action.ACTION_TWO_FINGER_SWIPE_FORWARD"),
    ACTION_TWO_FINGER_SWIPE_BACK("com.android.action.ACTION_TWO_FINGER_SWIPE_BACK"),
    ACTION_SETTINGS_KEY("com.android.action.ACTION_SETTINGS_KEY")
}

class KeyReceiver : BroadcastReceiver() {

    var listener: KeyReceiverListener? = null

    override fun onReceive(context: Context?, intent: Intent?) {
        intent?.action?.let {
            when (it) {
                KeyType.CLICK.action -> {
                    // Button click received — abort the broadcast
                    listener?.onReceive(KeyType.CLICK)
                    abortBroadcast()
                }
                KeyType.BUTTON_DOWN.action -> {
                    listener?.onReceive(KeyType.BUTTON_DOWN)
                    abortBroadcast()
                }
                KeyType.BUTTON_UP.action -> {
                    listener?.onReceive(KeyType.BUTTON_UP)
                    abortBroadcast()
                }
                KeyType.DOUBLE_CLICK.action -> {
                    // Button double-click — abort the broadcast — NOTE: this event cannot truly be intercepted; the system reserves it for the back/exit action
                    listener?.onReceive(KeyType.DOUBLE_CLICK)
                    abortBroadcast()
                }
                KeyType.AI_START.action -> {
                    listener?.onReceive(KeyType.AI_START)
                    abortBroadcast()
                }
                KeyType.LONG_PRESS.action -> {
                    listener?.onReceive(KeyType.LONG_PRESS)
                    abortBroadcast()
                }
                KeyType.ACTION_TWO_FINGER_SINGLE_TAP.action -> {
                    listener?.onReceive(KeyType.ACTION_TWO_FINGER_SINGLE_TAP)
                    abortBroadcast()
                }
                KeyType.ACTION_TWO_FINGER_DOUBLE_TAP.action -> {
                    listener?.onReceive(KeyType.ACTION_TWO_FINGER_DOUBLE_TAP)
                    abortBroadcast()
                }
                KeyType.ACTION_TWO_FINGER_SWIPE_FORWARD.action -> {
                    listener?.onReceive(KeyType.ACTION_TWO_FINGER_SWIPE_FORWARD)
                    abortBroadcast()
                }
                KeyType.ACTION_TWO_FINGER_SWIPE_BACK.action -> {
                    listener?.onReceive(KeyType.ACTION_TWO_FINGER_SWIPE_BACK)
                    abortBroadcast()
                }
                KeyType.ACTION_SETTINGS_KEY.action -> {
                    listener?.onReceive(KeyType.ACTION_SETTINGS_KEY)
                    abortBroadcast()
                }
            }
        }
    }
}
```

### 2. Registering and consuming the broadcasts

> **Note on the source's "left leg" log strings:** the log messages below say "button on left leg". This is a typo in the upstream sample — physically, the activated button is on the **right** temple. The string values are preserved verbatim from the source for fidelity; treat them as if they said "right temple".

Sample registration:

```kotlin
private val keyReceiver = KeyReceiver().apply {
    listener = object : KeyReceiverListener {
        override fun onReceive(keyType: KeyType) {
            latestKeyType = keyType
            when (keyType) {
                KeyType.CLICK -> Log.d("KeysActivity", "system event: button on left leg")
                KeyType.BUTTON_DOWN -> Log.d("KeysActivity", "system event: button on left leg down")
                KeyType.BUTTON_UP -> Log.d("KeysActivity", "system event: button on left leg up")
                KeyType.DOUBLE_CLICK -> Log.d("KeysActivity", "system event: button on left leg double click")
                KeyType.AI_START -> Log.d("KeysActivity", "system event: touchpad long pressed")
                KeyType.LONG_PRESS -> Log.d("KeysActivity", "system event: long pressed the button on left leg")
                KeyType.ACTION_TWO_FINGER_SINGLE_TAP -> Log.d("KeysActivity", "system event: two finger single tap")
                KeyType.ACTION_TWO_FINGER_DOUBLE_TAP -> Log.d("KeysActivity", "system event: two finger double tap")
                KeyType.ACTION_TWO_FINGER_SWIPE_FORWARD -> Log.d("KeysActivity", "system event: two finger swipe forward")
                KeyType.ACTION_TWO_FINGER_SWIPE_BACK -> Log.d("KeysActivity", "system event: two finger swipe back")
                KeyType.ACTION_SETTINGS_KEY -> Log.d("KeysActivity", "system event: two finger long pressed")
            }
        }
    }
}

@SuppressLint("UnspecifiedRegisterReceiverFlag")
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()
    window.addFlags(android.view.WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
    setContent {
        CXRSSDKSamplesTheme {
            KeysScreen(latestKeyType = latestKeyType?.name ?: "")
        }
    }
    registerReceiver(keyReceiver, IntentFilter().apply {
        addAction(KeyType.CLICK.action)
        addAction(KeyType.BUTTON_DOWN.action)
        addAction(KeyType.BUTTON_UP.action)
        addAction(KeyType.DOUBLE_CLICK.action)
        addAction(KeyType.AI_START.action)
        addAction(KeyType.LONG_PRESS.action)
        addAction(KeyType.ACTION_TWO_FINGER_SINGLE_TAP.action)
        addAction(KeyType.ACTION_TWO_FINGER_DOUBLE_TAP.action)
        addAction(KeyType.ACTION_TWO_FINGER_SWIPE_FORWARD.action)
        addAction(KeyType.ACTION_TWO_FINGER_SWIPE_BACK.action)
        addAction(KeyType.ACTION_SETTINGS_KEY.action)
        priority = 100
    })
}
```

### Other key events

Any remaining keys not covered by the broadcasts above can be observed through the standard system `KeyEvent` callbacks:

```kotlin
@SuppressLint("GestureBackNavigation")
override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
    Log.d("KeysActivity", "onKeyDown: $keyCode")
    when (keyCode) {
        KeyEvent.KEYCODE_BACK -> {
            Log.d("KeysActivity", "onKeyDown: back pressed")
            return true
        }
        KeyEvent.KEYCODE_ENTER -> {
            Log.d("KeysActivity", "onKeyUp: touchpad single down")
            return true
        }
        else -> Log.d("KeysActivity", "onKeyUp: $keyCode")
    }
    return super.onKeyDown(keyCode, event)
}

@SuppressLint("GestureBackNavigation")
override fun onKeyUp(keyCode: Int, event: KeyEvent?): Boolean {
    Log.d("KeysActivity", "onKeyUp: $keyCode")
    when (keyCode) {
        KeyEvent.KEYCODE_BACK -> {
            Log.d("KeysActivity", "onKeyUp: back pressed")
            return true
        }
        KeyEvent.KEYCODE_ENTER -> {
            Log.d("KeysActivity", "onKeyUp: touchpad single up")
            return true
        }
        else -> Log.d("KeysActivity", "onKeyUp: $keyCode")
    }
    return super.onKeyUp(keyCode, event)
}
```

## Related docs

- [Bare-Metal Development Guide](./development-guide.md) — overview, reserved system interactions, dev environment.
- [Audio Recording](./audio-recording.md) — uses the same `KeyReceiver` to start/stop recording on `CLICK`.
