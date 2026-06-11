# SDK Integration
This chapter uses Kotlin DSL (build.gradle.kts) as an example.

## Configure Maven repository
The CXR-M SDK uses Maven for online SDK package management.

Maven repository address: (https://maven.rokid.com/repository/maven-public/)

Locate settings.gradle.kts and add a Maven repository in the `<maven_repository_name>` section of dependencyResolutionManagementthe `<maven_repository_name>` node .repositories

image-20250311102401647

```kotlin
pluginManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven { url = uri("https://maven.rokid.com/repository/maven-public/") }
        google()
        mavenCentral()
    }
}

rootProject.name = "CXR_MDocSample"
include(":app")
```
 
## Dependency import
CXR-M SDK Package ("com.rokid.cxr:client-m:1.2.2").

Add the dependency to the `<dependency>` node in `build.gradle.kts` dependencies. Note: The SDK needs to have `minSdk` set to ≥ 28 .

```kotlin
//...Other Settings
android {
	//Other settings
    defaultConfig {
        //other settings
        minSdk = 28
    }
    //Other settings
}
dependencies {
	//....Others
    
    implementation("com.rokid.cxr:client-m:1.2.2")
}
```
Other dependencies (if there is a version conflict with existing versions in the project, please use the corresponding version from the SDK first):

```kotlin
implementation ("com.squareup.retrofit2:retrofit:2.9.0")
implementation ("com.squareup.retrofit2:converter-gson:2.9.0")
implementation ("com.squareup.okhttp3:okhttp:4.9.3")
implementation ("org.jetbrains.kotlin:kotlin-stdlib:2.1.0")
implementation ("com.squareup.okio:okio:2.8.0")
implementation ("com.google.code.gson:gson:2.10.1")
implementation ("com.squareup.okhttp3:logging-interceptor:4.9.1")
```
image-20250311105512744

## Permission Request
### 1. Declare permissions
The CXR-M SDK requires permissions for network, Wi-Fi, and Bluetooth (Bluetooth permission requires simultaneous application of the FINE_LOCATION permission). The following is the minimum set of permissions to request in AndroidManifest.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />

    <application>
        <!--Other Settings-->
    </application>

</manifest>
```
### 2. Dynamically apply for permissions
Before using the CXR-M SDK, please dynamically request the necessary permissions. Note that the SDK will be unavailable if permissions are insufficient. Here is a simple example:


```kotlin
class MainActivity : AppCompatActivity() {
    companion object {
        const val TAG = "MainActivity"
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
    }
    // Permission
    private val permissionGrantedResult = MutableLiveData<Boolean?>()

    override fun onCreate(savedInstanceState: Bundle?) {
       	// Other Code
        // Request Permissions
        permissionGrantedResult.postValue(null)
        requestPermissions(REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)
        
        // Observe Permission Result
        permissionGrantedResult.observe(this) {
            if (it == true) {
                // Permission All Granted
            }else{
                // Some Permission Denied or Not Started
            }
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == REQUEST_CODE_PERMISSIONS.hashCode()){
            val allGranted = grantResults.all { it == PackageManager.PERMISSION_GRANTED}
            permissionGrantedResult.postValue(allGranted)
        }
    }
}
```
