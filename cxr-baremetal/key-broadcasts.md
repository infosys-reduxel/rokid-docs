# Keys, Wear Detection, and Fold Events (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html> — "Keys and Wear Detection and Fold Events" page (English, fetched 2026-08-30)
>
> **Doc version: 1.0.0** — this workspace superseded the earlier Chinese workspace (`57e35cd3ae294d16b1b8fc8dcbb1b7c7`, doc v0.0.1, 2026-03-01) that this page was previously translated from. The upstream rewrite **corrects and expands** the action-string reference below (see the discrepancy note under [Touch panel broadcasts](#touch-panel-broadcasts-system-broadcasts)); treat this revision as authoritative over the older, superseded reference implementation kept further down this page.

This page covers the system-function buttons, touchpad, and wear/fold sensors on Rokid Glasses, and how a bare-metal Android app intercepts them. See the [Bare-Metal Development Guide](./development-guide.md) for context and the list of system interactions you **cannot** override (long-press touchpad → AI module, double-click right button → back, top button → camera).

## Overview

Rokid Glasses exposes two kinds of input events to bare-metal apps:

- **System broadcasts** — temple function key, two-finger touchpad gestures (double-tap, swipe forward, swipe back, long press), and single-finger long press on the touchpad. Register a dynamic `BroadcastReceiver` to receive them.
- **Standard `KeyEvent`** — single-finger tap, single-finger double-tap, two-finger tap, single-finger long press, and single-finger swipe forward / swipe back (a sequence of two key-down events) on the touchpad. Handle these in the Activity's `dispatchKeyEvent` / `onKeyDown` / `onKeyUp`.

This page lists the **action / extra / key code** for each event and how to integrate them.

## Wear and fold

### Wear state change

| Field | Value |
|---|---|
| Action | `com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED` |
| Extra | `glasses_take_state` (`String`): `"1"` worn, `"0"` removed |

### Temple open / fold

| Field | Value |
|---|---|
| Action | `com.rokid.sprite.ACTION_LEG_STATUS_CHANGED` |
| Extra | `glasses_leg_state` (`String`): `"1"` open, `"0"` folded |

### Wear/fold example

```kotlin
ContextCompat.registerReceiver(
    context,
    receiver,
    IntentFilter().apply {
        addAction("com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED")
        addAction("com.rokid.sprite.ACTION_LEG_STATUS_CHANGED")
    },
    ContextCompat.RECEIVER_EXPORTED,
)
// onReceive: intent.getStringExtra("glasses_take_state"), etc.
```

## Function key and touch panel (system broadcasts)

Glasses run **Android 12 (API 31)**. Prefer `ContextCompat.registerReceiver(..., ContextCompat.RECEIVER_EXPORTED)`. When `targetSdk` ≥ 33, an explicit exported flag is required; `ContextCompat` runs correctly on API 31 glasses.

Broadcasts are delivered only to **dynamically registered** receivers.

### Temple function key

| Event | Action | Notes |
|---|---|---|
| Click | `com.android.action.ACTION_SPRITE_BUTTON_CLICK` | |
| Down | `com.android.action.ACTION_SPRITE_BUTTON_DOWN` | |
| Up | `com.android.action.ACTION_SPRITE_BUTTON_UP` | Not sent if long press already fired |
| Double-click | `com.android.action.ACTION_SPRITE_BUTTON_DOUBLE_CLICK` | |
| Triple-click | `com.android.action.ACTION_BOLON_PAIRING` | Triggers Bluetooth pairing |
| Long press | `com.android.action.ACTION_SPRITE_BUTTON_LONG_PRESS` | Enters shutdown by default; intercept with `abortBroadcast()` |
| Very long press | `com.android.action.ACTION_SPRITE_BUTTON_VERY_VERY_LONG_PRESS` | |

### Touch panel broadcasts (system broadcasts)

| Event | Action |
|---|---|
| Single-finger long press | `com.android.action.ACTION_AI_START` |
| Two-finger double-tap | `com.android.action.ACTION_TWO_FINGER_DOUBLE_TAP` |
| Two-finger swipe forward | `com.android.action.ACTION_TWO_FINGER_SWIPE_FORWARD` |
| Two-finger swipe back | `com.android.action.ACTION_TWO_FINGER_SWIPE_BACK` |
| Two-finger long press (settings) | `com.android.action.ACTION_SETTINGS_KEY` |

Two-finger tap is **not** delivered as a broadcast: on current glasses firmware it is reported as a `KeyEvent` (`KEYCODE_NOTIFICATION`, key code 83) — see [Registration and KeyEvent example](#registration-and-keyevent-example) below. Per the [Notes](#notes) section, two-finger tap itself is currently **not supported** as an app-facing gesture.

Single-finger tap and double-tap use `KeyEvent` (`KEYCODE_ENTER` / `KEYCODE_BACK`); single-finger long press also delivers `KEYCODE_PROG_BLUE` in addition to the `ACTION_AI_START` broadcast — both paths can be consumed; see the example below.

> **Change from the previous (Chinese) revision of this doc:** the earlier translation of this page listed a distinct `ACTION_TWO_FINGER_SINGLE_TAP` broadcast action. The current upstream English documentation does not include that action string among the touch-panel broadcasts above, and explicitly states that two-finger tap is delivered via `KeyEvent` rather than broadcast, and is not currently supported as a gesture. The reference `KeyReceiver` implementation further down this page — carried over from the older revision — still registers for `ACTION_TWO_FINGER_SINGLE_TAP`; treat that action string as unverified/legacy until confirmed against `GlassesBareDevSample`.

### Registration and KeyEvent example

```kotlin
ContextCompat.registerReceiver(
    context,
    keyReceiver,
    IntentFilter().apply {
        priority = IntentFilter.SYSTEM_HIGH_PRIORITY
        addAction("com.android.action.ACTION_SPRITE_BUTTON_CLICK")
        addAction("com.android.action.ACTION_SPRITE_BUTTON_LONG_PRESS")
        addAction("com.android.action.ACTION_AI_START")
        addAction("com.android.action.ACTION_TWO_FINGER_SWIPE_FORWARD")
        // … other actions from the tables above
    },
    ContextCompat.RECEIVER_EXPORTED,
)

override fun onKeyUp(keyCode: Int, event: KeyEvent?): Boolean {
    if (keyCode == KeyEvent.KEYCODE_ENTER && event?.repeatCount == 0) {
        // Single-finger tap
        return true
    }
    return super.onKeyUp(keyCode, event)
}

override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
    when (keyCode) {
        KeyEvent.KEYCODE_BACK -> {
            // Single-finger double-tap / back
            return true
        }
        KeyEvent.KEYCODE_PROG_BLUE -> {
            // Single-finger long press (also sent as ACTION_AI_START)
            return true
        }
        KeyEvent.KEYCODE_NOTIFICATION -> {
            // Two-finger tap (key code 83, scan code 204)
            return true
        }
        KeyEvent.KEYCODE_SETTINGS -> {
            // Two-finger long press (also sent as ACTION_SETTINGS_KEY)
            return true
        }
    }
    return super.onKeyDown(keyCode, event)
}
```

Call `abortBroadcast()` inside `onReceive`, before `goAsync()`, and only for **ordered** broadcasts, to block system defaults (AI, shutdown, etc.). Non-ordered broadcasts cannot be aborted; consume the `KeyEvent` path when available.

### Single-finger swipe forward / back (`KeyEvent` sequence)

Single-finger swipe forward and swipe back on the touchpad are reported as a **sequence of two key-down events**, with no more than 500 ms between the two keys:

| Gesture | Key sequence (key code) | Interval |
|---|---|---|
| Swipe forward | `KEYCODE_DPAD_RIGHT` (22) → `KEYCODE_DPAD_DOWN` (20) | ≤ 500 ms |
| Swipe back | `KEYCODE_DPAD_LEFT` (21) → `KEYCODE_DPAD_UP` (19) | ≤ 500 ms |

Non-consecutive keys or a timeout invalidate the sequence. Consume the first key of the sequence as well so it does not leak into focus navigation. Example:

```kotlin
class SwipeDetector(
    private val onSwipeForward: () -> Unit,
    private val onSwipeBack: () -> Unit,
) {
    private var lastKeyCode = -1
    private var lastEventTime = 0L

    /** Call for each ACTION_DOWN (repeatCount == 0); returns true when consumed. */
    fun onKeyDown(keyCode: Int, eventTime: Long): Boolean {
        val prev = if (eventTime - lastEventTime <= MAX_INTERVAL_MS) lastKeyCode else -1
        lastKeyCode = keyCode
        lastEventTime = eventTime
        return when {
            prev == FORWARD_FIRST && keyCode == FORWARD_SECOND -> {
                onSwipeForward()
                reset()
                true
            }
            prev == BACK_FIRST && keyCode == BACK_SECOND -> {
                onSwipeBack()
                reset()
                true
            }
            // Consume the first key too, so it does not leak into focus navigation
            keyCode == FORWARD_FIRST || keyCode == BACK_FIRST -> true
            else -> false
        }
    }

    fun reset() {
        lastKeyCode = -1
        lastEventTime = 0L
    }

    companion object {
        private const val FORWARD_FIRST = KeyEvent.KEYCODE_DPAD_RIGHT   // 22
        private const val FORWARD_SECOND = KeyEvent.KEYCODE_DPAD_DOWN   // 20
        private const val BACK_FIRST = KeyEvent.KEYCODE_DPAD_LEFT       // 21
        private const val BACK_SECOND = KeyEvent.KEYCODE_DPAD_UP        // 19
        private const val MAX_INTERVAL_MS = 500L
    }
}

// In the Activity:
override fun dispatchKeyEvent(event: KeyEvent): Boolean {
    if (event.action == KeyEvent.ACTION_DOWN && event.repeatCount == 0) {
        if (swipeDetector.onKeyDown(event.keyCode, event.eventTime)) {
            return true
        }
    }
    return super.dispatchKeyEvent(event)
}
```

## Notes

- Do not rely on actions not listed on this page.
- Two-finger tap is **not supported**.
- Single-finger swipe forward / back are key-sequence events; the app must track sequence state itself. If the app uses focus navigation, consume the first key of the sequence.

## Legacy reference implementation (superseded)

The sample below predates the current upstream English documentation and its `GlassesBareDevSample` reference project — it was reverse-engineered from an older Chinese-language revision of this page, sourced from a different sample package (`com.rokid.cxrssdksamples`). It is kept here because the underlying **ordered-broadcast + `abortBroadcast()`** pattern it demonstrates is still valid, but its action-string list (notably `ACTION_TWO_FINGER_SINGLE_TAP`) has **not** been re-verified against the current spec above and should be treated as unconfirmed. Prefer the [Registration and KeyEvent example](#registration-and-keyevent-example) for new code.

### 1. Button broadcasts — receiver (legacy)

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

### 2. Registering and consuming the broadcasts (legacy)

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

### Other key events (legacy)

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

- [Bare-Metal Development Guide](./development-guide.md) — overview, reserved system interactions, dev environment, sample project.
- [Audio Recording](./audio-recording.md) — uses the same legacy `KeyReceiver` pattern to start/stop recording on `CLICK`.
