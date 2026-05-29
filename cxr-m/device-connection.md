# Device connection

Before reading this chapter, please note that you have already understood the content of the "SDK Import" chapter.

## Bluetooth connection

## 1. Find Bluetooth devices

Device discovery is performed via the Android standard Bluetooth interface. Here is a simple example:

```kotlin
package com.rokid.cxrandroiddocsample.helpers

//imports

/**
 * Bluetooth Helper
 * @author rokid
 * @date 2025/04/27
 * @param context Activity Register Context
 * @param initStatus Init Status
 * @param deviceFound Device Found
 */
class BluetoothHelper(
    val context: AppCompatActivity,
    val initStatus: (INIT_STATUS) -> Unit,
    val deviceFound: () -> Unit
) {
    companion object {
        const val TAG = "Rokid Glasses CXR-M"

        // Request Code
        const val REQUEST_CODE_PERMISSIONS = 100

        // Required Permissions
        private val REQUIRED_PERMISSIONS = mutableListOf(
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.BLUETOOTH,
            Manifest.permission.BLUETOOTH_ADMIN,
        ).apply {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
                add(Manifest.permission.BLUETOOTH_SCAN)
                add(Manifest.permission.BLUETOOTH_CONNECT)
            }
        }.toTypedArray()

        // Init Status
        enum class INIT_STATUS {
            NotStart,
            INITING,
            INIT_END
        }
    }

    // Scan Results
    val scanResultMap: ConcurrentHashMap<String, BluetoothDevice> = ConcurrentHashMap()

    // Bonded Devices
    val bondedDeviceMap: ConcurrentHashMap<String, BluetoothDevice> = ConcurrentHashMap()

    // Scanner
    private val scanner by lazy {
        adapter?.bluetoothLeScanner ?: run {
            Toast.makeText(context, "Bluetooth is not supported", Toast.LENGTH_SHORT).show()
            showRequestPermissionDialog()
            throw Exception("Bluetooth is not supported!!")
        }
    }

    // Bluetooth Enabled
    @SuppressLint("MissingPermission")
    private val bluetoothEnabled: MutableLiveData<Boolean> = MutableLiveData<Boolean>().apply {
        this.observe(context) {
            if (this.value == true) {
                initStatus.invoke(INIT_STATUS.INIT_END)
                startScan()
            } else {
                showRequestBluetoothEnableDialog()
            }
        }
    }

    //  Bluetooth State Listener
    private val requestBluetoothEnable = context.registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            adapter = manager?.adapter
        } else {
            showRequestBluetoothEnableDialog()
        }
    }

    // Bluetooth Adapter
    private var adapter: BluetoothAdapter? = null
        set(value) {
            field = value
            value?.let {
                if (!it.isEnabled) {
                    //to Enable it
                    requestBluetoothEnable.launch(Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE))
                } else {
                    bluetoothEnabled.postValue(true)
                }
            }
        }

    // Bluetooth Manager
    private var manager: BluetoothManager? = null
        set(value) {
            field = value
            initStatus.invoke(INIT_STATUS.INITING)
            value?.let {
                adapter = it.adapter
            } ?: run {
                Toast.makeText(context, "Bluetooth is not supported", Toast.LENGTH_SHORT).show()
                showRequestPermissionDialog()
            }
        }

    // Permission Result
    val permissionResult: MutableLiveData<Boolean> = MutableLiveData<Boolean>().apply {
        this.observe(context) {
            if (it == true) {
                manager =
                    context.getSystemService(AppCompatActivity.BLUETOOTH_SERVICE) as BluetoothManager
            } else {
                showRequestPermissionDialog()
            }
        }
    }

    // Scan Listener
    val scanListener = object : ScanCallback() {
        @SuppressLint("MissingPermission")
        override fun onScanResult(callbackType: Int, result: ScanResult?) {
            super.onScanResult(callbackType, result)
            result?.let { r ->
                r.device.name?.let {
                    scanResultMap[it] = r.device
                    deviceFound.invoke()
                }
            }
        }

        override fun onScanFailed(errorCode: Int) {
            super.onScanFailed(errorCode)
            Toast.makeText(
                context,
                "Scan Failed $errorCode",
                Toast.LENGTH_SHORT
            ).show()
        }
    }
    // check permissions
    fun checkPermissions() {
        initStatus.invoke(INIT_STATUS.NotStart)
        context.requestPermissions(REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)
        context.registerReceiver(
            bluetoothStateListener,
            IntentFilter(BluetoothAdapter.ACTION_STATE_CHANGED)
        )
    }

    // Release
    @SuppressLint("MissingPermission")
    fun release() {
        context.unregisterReceiver(bluetoothStateListener)
        stopScan()
        permissionResult.postValue(false)
        bluetoothEnabled.postValue(false)
    }


    // Show Request Permission Dialog
    private fun showRequestPermissionDialog() {
        AlertDialog.Builder(context)
            .setTitle("Permission")
            .setMessage("Please grant the permission")
            .setPositiveButton("OK") { _, _ ->
                context.requestPermissions(
                    REQUIRED_PERMISSIONS,
                    REQUEST_CODE_PERMISSIONS
                )
            }
            .setNegativeButton("Cancel") { _, _ ->
                Toast.makeText(
                    context,
                    "Permission does not granted, FINISH",
                    Toast.LENGTH_SHORT
                ).show()
                context.finish()
            }
            .show()
    }

    // Show Request Bluetooth Enable Dialog
    private fun showRequestBluetoothEnableDialog() {
        AlertDialog.Builder(context)
            .setTitle("Bluetooth")
            .setMessage("Please enable the bluetooth")
            .setPositiveButton("OK") { _, _ ->
                requestBluetoothEnable.launch(Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE))
            }
            .setNegativeButton("Cancel") { _, _ ->
                Toast.makeText(
                    context,
                    "Bluetooth does not enabled, FINISH",
                    Toast.LENGTH_SHORT
                ).show()
                context.finish()
            }
            .show()
    }

    // Start Scan
    @SuppressLint("MissingPermission")
    @RequiresPermission(Manifest.permission.BLUETOOTH_SCAN)
    fun startScan() {
        scanResultMap.clear()
        val connectedList = getConnectedDevices()
        for (device in connectedList) {
            device.name?.let {
                if (it.contains("Glasses", false)) {
                    bondedDeviceMap[it] = device
                    deviceFound.invoke()
                }
            }
        }

        adapter?.bondedDevices?.forEach { d ->
            d.name?.let {
                if (it.contains("Glasses", false)) {
                    if (bondedDeviceMap[it] == null) {
                        bondedDeviceMap[it] = d
                    }
                }
                deviceFound.invoke()
            }
        }

        try {
            scanner.startScan(
                listOf<ScanFilter>(
                    ScanFilter.Builder()
                        .setServiceUuid(ParcelUuid.fromString("00009100-0000-1000-8000-00805f9b34fb"))//Rokid Glasses Service
                        .build()
                ), ScanSettings.Builder().build(),
                scanListener
            )
        } catch (e: Exception) {
            Toast.makeText(context, "Scan Failed ${e.message}", Toast.LENGTH_SHORT).show()
        }
    }

    // Stop Scan
    @RequiresPermission(Manifest.permission.BLUETOOTH_SCAN)
    fun stopScan() {
        scanner.stopScan(scanListener)
    }

    //  Get Connected Devices
    @SuppressLint("MissingPermission")
    private fun getConnectedDevices(): List<BluetoothDevice> {
        return adapter?.bondedDevices?.filter { device ->
            try {
                val isConnected =
                    device::class.java.getMethod("isConnected").invoke(device) as Boolean
                isConnected
            } catch (_: Exception) {
                Toast.makeText(context, "Get Connected Devices Failed", Toast.LENGTH_SHORT).show()
                false
            }
        } ?: emptyList()
    }

    // Bluetooth State Listener
    val bluetoothStateListener = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            val action = intent?.action
            if (action == BluetoothAdapter.ACTION_STATE_CHANGED) {
                val state = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, BluetoothAdapter.ERROR)
                when (state) {
                    BluetoothAdapter.STATE_OFF -> {
                        initStatus.invoke(INIT_STATUS.NotStart)
                        bluetoothEnabled.postValue(false)
                    }
                }
            }
        }
    }

}
```

## 2. Initialize and connect Bluetooth

Device initialization and connection are controlled through the CxrApi class in the CXR_M SDK.

Method for initializing the Bluetooth module public void initBluetooth(Context context, BluetoothDevice devcie, CxrBluetoothCallback callback):

Here is a simple usage example:

```kotlin
// bluetooth status callback
private val bluetoothStatusCallback = object : BluetoothStatusCallback {
    override fun onConnectionInfo(p0: String?, p1: String?) {
        Log.e("onConnectionInfo", "onConnectionInfo: socket uuid = ${p0}, macAddress = ${p1}")
    }
    override fun onConnected() {
        Log.e("onConnected", "onConnected")
    }
    override fun onDisconnected() {
        Log.e("onDisconnected", "onDisconnected")
    }
    override fun onFailed(p0: ValueUtil.CxrBluetoothErrorCode?) {
        Log.e("onFailed", "onFailed: ${p0?.name ?: ""}")
    }
}

/**
 * connect to glasses
 * @param context Context
 * @param device BluetoothDevice
 */
@SuppressLint("MissingPermission")
fun connectToRokidGlasses(context: Context, device: BluetoothDevice) {
    CxrApi.getInstance().initBluetooth(context, device, bluetoothStatusCallback)
}
```

This BluetoothStatusCallbackincludes a Bluetooth information monitoring interface.

in:

fun onConnected()Callback when Bluetooth connection is successful
fun onDisconnected()Callback when Bluetooth connection is lost
fun onConnectionInfo(p0: String?, p1: String?)Device information update interface
p0Device UUID
p1Device MAC address

## 3. Obtain the Bluetooth communication module connection status

The current connection status of the Bluetooth communication module can be obtained through public boolean isBluetoothConnected()certain methods.

Return value:

trueBluetooth communication module is connected.
falseBluetooth module not connected
A simple example is as follows:

```kotlin
/**
 *  Get Bluetooth Connection Status
 */
fun getConnectionStatus(): Boolean {
    return CxrApi.getInstance().isBluetoothConnected
}
```

## 4. Deinitialize Bluetooth

Yes, it's possible public void deinitBluetooth().

A simple example is as follows:

```kotlin
private fun deInit() {
    CxrApi.getInstance().deinitBluetooth()
}
```

## 5. Bluetooth Reconnection

Bluetooth reconnection is achieved via `connectBluetooth`. The signature changed in v1.2.1:

**v1.1.0 and earlier:**
```
public void connectBluetooth(Context context, String uuid, String macAddress, BluetoothStatusCallback callback)
```

**v1.2.1+ (breaking change):** A fourth `String customInfo` parameter was added:
```
public void connectBluetooth(Context context, String uuid, String macAddress, String customInfo, BluetoothStatusCallback callback)
```

The `customInfo` parameter is a per-client identifier used for multi-client Bluetooth support. Pass an empty string `""` if not required.

Parameters:
- `uuid` — Device UUID
- `macAddress` — Device MAC address
- `customInfo` — (v1.2.1+) per-client custom identifier string

Example (v1.1.0):

```kotlin
@SuppressLint("MissingPermission")
fun reconnectRokidGlasses(context: Context, uuid: String, macAddress: String) {
    CxrApi.getInstance().connectBluetooth(context, uuid, macAddress, bluetoothStatusCallback)
}
```

Example (v1.2.1+):

```kotlin
@SuppressLint("MissingPermission")
fun reconnectRokidGlasses(context: Context, uuid: String, macAddress: String) {
    CxrApi.getInstance().connectBluetooth(context, uuid, macAddress, "", bluetoothStatusCallback)
}
```

## Wi-Fi connection

Note: Please complete the Bluetooth connection before using the Wi-Fi module. Also, the Wi-Fi module is a high-power module; please only turn it on when necessary.

## 1. Initialize the Wi-Fi communication module

Use public ValueUtil.CxrStatus initWifi(WifiStatusCallback var1)the initial Wi-Fi communication module.

inWifiStatusCallback

onConnectedCallback when connection is successful
onDisconnectedCallback when disconnection
onFailedCallback when connection fails
Code example:

```kotlin
/**
 * Wifi Callback
 */
private val wifiCallback: WifiStatusCallback = object : WifiStatusCallback {
    override fun onConnected() {
        Log.i("WifiModel", "onConnected")
    }
    override fun onDisconnected() {
        Log.i("WifiModel", "onDisconnected")
    }
    override fun onFailed(p0: ValueUtil.CxrWifiErrorCode?) {
        Log.e("WifiModel", "onFailed")
    }
}

/**
 * Init Wifi
 */
fun connectWifi(): ValueUtil.CxrStatus? {
    return CxrApi.getInstance().initWifi(wifiCallback)
}
```

## 2. Obtain the Wi-Fi communication module connection status

public boolean isWifiConnected()The connection status of the current Wi-Fi communication module can be obtained through certain methods.

Return value:

trueWi-Fi communication module is connected.
falseWi-Fi communication module not connected
A simple example is as follows:

```kotlin
/**
 * Get Wifi Connection Status
 * @return true: Connected, false: Disconnected
 */
private fun getWifiConnection(): Boolean{
    return CxrApi.getInstance().isWifiConnected
}
```

## 3. Deinitialize the Wi-Fi communication module

Yes, it's possible public void deinitWifi().

A simple example is as follows:

```kotlin
/**
 * Deinit Wifi
 */
private fun deinitWifi(){
    CxrApi.getInstance().deinitWifi()
}
```
