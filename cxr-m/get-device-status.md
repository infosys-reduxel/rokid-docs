# Get device status

## Glasses status

Note: For information related to the hardware of the glasses, it is necessary to ensure that the Bluetooth channel between the mobile phone and the glasses is connected.

## 1. Obtain eyeglass information

It can be obtained through public ValueUtil.CxrStatus getGlassInfo(GlassInfoResultCallback var1)certain methods.

Example:

```kotlin
private val glassesInfoCallback = object :GlassInfoResultCallback{
    override fun onGlassInfoResult(
        p0: ValueUtil.CxrStatus?,
        p1: GlassInfo?
    ) {
        Log.i("GlassesInfo", "onGlassInfoResult: ${p1?.toString()}")
    }
}

/**
 * Get Glasses Info
 */
fun getGlassesInfo(){
    CxrApi.getInstance().getGlassInfo(glassesInfoCallback)
}
```

## 2. Synchronize glasses time and time zone

There are pbulic CxrStatus setGlassTime()ways to synchronize the time and time zone of your glasses.

Example:

```kotlin
/**
 * Sync Time
 * @return CxrStatus: REQUEST_FAILED, REQUEST_SUCCEED, REQUEST_WAITING
 * REQUEST_SUCCEED: Set time success
 * REQUEST_WAITING: Waiting for response from glasses, Do Not Need Call Again
 * REQUEST_FAILED: Set time failed
 * @see CxrStatus
 */
private fun setTime(): CxrStatus{
    return CxrApi.getInstance().setGlassTime()
}
```

## 3. Set the glasses brightness

The brightness of the glasses can public CxrStatus setGlassBrightness(int value)be set. The brightness value ranges from [0, 15]. After successful setting, the setting result can be obtained through the callback BrightnessUpdateListenerof the set interface fun onBrightnessUpdate(p0: String?). Additionally, if the brightness information is manually modified on the glasses side, it can also be listened for in the callback.

Example:

```kotlin
fun setGlassesBrightness(@IntRange(0,15) brightness: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setGlassBrightness(brightness)
}

private val glassesBrightnessUpdateListener = object : BrightnessUpdateListener{
    override fun onBrightnessUpdated(p0: String?) {
        Log.i("GlassesInfo", "onBrightnessUpdated: $p0")
    }
}

fun setGlassesListener(set: Boolean){
    if (set)
        CxrApi.getInstance().setBrightnessUpdateListener(glassesBrightnessUpdateListener)
    else
        CxrApi.getInstance().setBrightnessUpdateListener(null)
}
```

## 4. Adjust the glasses volume

The volume of the glasses can public CxrStatus setGlassVolume(int value)be set. The volume value ranges from [0, 15]. After successful setting, the setting result can be obtained through the callback VolumeUpdateListenerof the set interface fun onVolumeUpdated(p0: String?). Additionally, if the volume is manually changed on the glasses, it can also be listened to in the callback.

```kotlin
 fun setGlassesVolume(@IntRange(0,15) volume: Int): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().setGlassVolume(volume)
}

private val glassesVolumeUpdateListener = object : VolumeUpdateListener{
    override fun onVolumeUpdated(p0: String?) {
        Log.i("GlassesInfo", "onVolumeUpdated: $p0")
    }
}

fun setGlassesVolumeListener(set: Boolean){
    if (set)
        CxrApi.getInstance().setVolumeUpdateListener(glassesVolumeUpdateListener)
    else
        CxrApi.getInstance().setVolumeUpdateListener(null)
}
```

## 5. Monitoring glasses battery level changes

The battery level change can be obtained by setting the callback function BatteryLevelUpdateListenerof the interface .fun onBatteryLevelUpdated(p0: String?)

```kotlin
private val glassesBatteryLevelUpdateListener = object : BatteryLevelUpdateListener{
    override fun onBatteryLevelUpdated(p0: String?) {
        Log.i("GlassesInfo", "onBatteryLevelUpdated: $p0")
    }
}

fun setGlassesBatteryLevelListener(set: Boolean){
    if (set)
        CxrApi.getInstance().setBatteryLevelUpdateListener(glassesBatteryLevelUpdateListener)
    else
        CxrApi.getInstance().setBatteryLevelUpdateListener(null)
}
```
