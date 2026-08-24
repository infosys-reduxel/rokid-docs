# Keys, Wear Detection, and Fold Events (Bare-Metal)

> Source: <https://custom.rokid.com/prod/rokid_web/ff28c865a9634876be98cbc293588460/pc/us/index.html?documentId=ca8fedf26d534e1fabb8a34d1fa24e98> (official English documentation, fetched 2026-08-24)
>
> **Doc version: v1.0.0** (supersedes the v0.0.1 revision fetched 2026-05-29 from the now-migrated `57e35cd3ae294d16b1b8fc8dcbb1b7c7` workspace; the reference `KeyReceiver` sample below is retained from that earlier decompiled-sample capture and cross-checked against the current action list).

This page covers the input events Rokid Glasses exposes to bare-metal apps: the temple function key, the touch panel (single- and two-finger gestures), and wear / fold state changes. See the [Introduction to Bare-Metal Development on Rokid Glasses](./development-guide.md) for context.

## Overview

Rokid Glasses exposes two kinds of input events to bare-metal apps:

- **System broadcasts**: temple function key, two-finger touch-panel gestures (double-tap, swipe forward, swipe back, long press), and single-finger long press on the touch panel. Register a dynamic `BroadcastReceiver` to receive them.
- **Standard `KeyEvent`**: single-finger tap, single-finger double-tap, two-finger tap, single-finger long press, and single-finger swipe forward / swipe back (a sequence of two key-down events) on the touch panel. Handle them in Activity `dispatchKeyEvent` / `onKeyDown` / `onKeyUp`.

This chapter lists the **action / extra / key code** for each event and how to integrate them.

## Wear and fold

### Wear state change

| Field | Value |
| --- | --- |
| Action | `com.rokid.sprite.ACTION_TAKE_STATUS_CHANGED` |
| Extra | `glasses_take_state` (String): `"1"` worn, `"0"` removed |

### Temple open / fold

| Field | Value |
| --- | --- |
| Action | `com.rokid.sprite.ACTION_LEG_STATUS_CHANGED` |
| Extra | `glasses_leg_state` (String): `"1"` open, `"0"` folded |

### Example

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

Glasses run **Android 12 (API 31)**. Prefer `ContextCompat.registerReceiver(..., ContextCompat.RECEIVER_EXPORTED)`. When **targetSdk ≥ 33**, an explicit exported flag is required; `ContextCompat` runs correctly on API 31 glasses.

Broadcasts are delivered only to **dynamically registered** receivers.

### Temple function key

| Event | Action | Notes |
| --- | --- | --- |
| Click | `com.android.action.ACTION_SPRITE_BUTTON_CLICK` | |
| Down | `com.android.action.ACTION_SPRITE_BUTTON_DOWN` | |
| Up | `com.android.action.ACTION_SPRITE_BUTTON_UP` | Not sent if long press already fired |
| Double-click | `com.android.action.ACTION_SPRITE_BUTTON_DOUBLE_CLICK` | |
| Triple-click | `com.android.action.ACTION_BOLON_PAIRING` | Triggers Bluetooth pairing |
| Long press | `com.android.action.ACTION_SPRITE_BUTTON_LONG_PRESS` | Enters shutdown by default; intercept with `abortBroadcast` |
| Very long press | `com.android.action.ACTION_SPRITE_BUTTON_VERY_VERY_LONG_PRESS` | |

### Touch panel broadcasts (single-finger long press and two-finger gestures)

| Event | Action |
| --- | --- |
| Single-finger long press | `com.android.action.ACTION_AI_START` |
| Two-finger double-tap | `com.android.action.ACTION_TWO_FINGER_DOUBLE_TAP` |
| Two-finger swipe forward | `com.android.action.ACTION_TWO_FINGER_SWIPE_FORWARD` |
| Two-finger swipe back | `com.android.action.ACTION_TWO_FINGER_SWIPE_BACK` |
| Two-finger long press (settings) | `com.android.action.ACTION_SETTINGS_KEY` |

Two-finger **single tap** is **not** delivered as a broadcast: on the glasses firmware it is reported as a **`KeyEvent`** (`KEYCODE_NOTIFICATION`, key code 83) — see below. This differs from the v0.0.1-era sample, which listened for a (now removed) `ACTION_TWO_FINGER_SINGLE_TAP` broadcast.

Single-finger tap and double-tap use **`KeyEvent`** (`KEYCODE_ENTER` / `KEYCODE_BACK`); single-finger long press also delivers **`KEYCODE_PROG_BLUE`** and broadcast **`ACTION_AI_START`** — both paths can be consumed; see the example below.

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

Call `abortBroadcast()` inside `onReceive`, before `goAsync`, and only for **ordered** broadcasts, to block system defaults (AI, shutdown, etc.). Non-ordered broadcasts cannot be aborted; consume the `KeyEvent` path when available.

### Single-finger swipe forward / back (KeyEvent sequence)

Single-finger swipe forward and swipe back on the touch panel are reported as a **sequence of two key-down events**, with no more than 500 ms between the two keys:

| Gesture | Key sequence (key code) | Interval |
| --- | --- | --- |
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

- Do not rely on actions not listed in this chapter.
- Two-finger tap is reported as a `KeyEvent` (key code 83); do not rely on a broadcast.
- Single-finger swipe forward / back are key-sequence events; the app must track sequence state itself. If the app uses focus navigation, consume the first key of the sequence.

## Legacy reference sample (v0.0.1-era, temple-key broadcasts only)

The following `KeyReceiver` was captured from the v0.0.1-era decompiled sample. It only covers the **temple function key** and a superseded two-finger single-tap broadcast (`ACTION_TWO_FINGER_SINGLE_TAP`, no longer emitted — see the [Function key and touch panel](#function-key-and-touch-panel-system-broadcasts) tables above for the current action set). Kept for reference on the ordered-broadcast + `priority = 100` + `abortBroadcast()` pattern; prefer the current tables above for the authoritative action-string list.

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
                    // NOTE: the temple double-click is also reserved by the platform for the global back/exit
                    // action; abortBroadcast() stops downstream receivers in this process but the platform
                    // back semantic still applies.
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

Sample registration (`priority = 100`):

```kotlin
private val keyReceiver = KeyReceiver().apply {
    listener = object : KeyReceiverListener {
        override fun onReceive(keyType: KeyType) {
            latestKeyType = keyType
        }
    }
}

@SuppressLint("UnspecifiedRegisterReceiverFlag")
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    registerReceiver(keyReceiver, IntentFilter().apply {
        addAction(KeyType.CLICK.action)
        addAction(KeyType.BUTTON_DOWN.action)
        addAction(KeyType.BUTTON_UP.action)
        addAction(KeyType.DOUBLE_CLICK.action)
        addAction(KeyType.AI_START.action)
        addAction(KeyType.LONG_PRESS.action)
        addAction(KeyType.ACTION_TWO_FINGER_DOUBLE_TAP.action)
        addAction(KeyType.ACTION_TWO_FINGER_SWIPE_FORWARD.action)
        addAction(KeyType.ACTION_TWO_FINGER_SWIPE_BACK.action)
        addAction(KeyType.ACTION_SETTINGS_KEY.action)
        priority = 100
    })
}
```

## Related docs

- [Introduction to Bare-Metal Development on Rokid Glasses](./development-guide.md) — overview, runtime environment, dev environment.
- [Raw Audio on Glasses](./audio-recording.md) — can reuse the click event to start/stop recording.
- [GlassesBareDevSample Project and Pages](./sample-project.md) — how the current sample's `BareGlassesInputDispatcher` wires these events.
