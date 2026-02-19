# TrustZoneAccessService (QTI TrustZone / Mink IPC)

**Package:** `com.qualcomm.qti.qms.service.trustzoneaccess`
**APK:** `TrustZoneAccessService.apk`
**Min SDK:** 31 | **Target SDK:** 33
**Native Library:** `libnative-api.so`

### Purpose

Provides a binder-based service that allows authorized apps to obtain file descriptors for communicating with TrustZone (Qualcomm Secure Execution Environment). This is the Android-side bridge for the Mink IPC protocol, which enables apps to invoke TrustZone trusted applications (TAs).

The service verifies caller credentials (UID, package name, signing certificates, permissions) against a whitelist, then returns a `ParcelFileDescriptor` the caller can use to communicate with TrustZone.

### Permissions

- `android.permission.RECEIVE_BOOT_COMPLETED`
- `android.permission.INTERNET`

### Exported Components

**This service is exported and bindable by other apps.**

| Type | Name | Exported | Notes |
|------|------|----------|-------|
| Service | `TZAccessService` | **Yes** | Bindable. Returns IMinkSocketFd binder. |

### Intent Filters

- `com.qualcomm.qti.qms.service.trustzoneaccess.TZAccessService` (bindable service action)

### Attributes

- `forceQueryable="true"` -- visible to all apps regardless of package visibility filtering (Android 11+)

### Behavior Summary

1. App binds to TZAccessService via intent action
2. Service returns `IMinkSocketFdImpl` binder (implements `IMinkSocketFd`)
3. App calls `request(String appName, int[] handleOut)`:
   - Service collects caller credentials (UID, package name, signing certs, granted permissions) via `CredentialHelper`
   - Credentials are CBOR-encoded
   - Service checks caller permissions against `/vendor/etc/ssg/tz_whitelist.json`
   - For `ssgtzd` and `qwes_ipc` requests: calls `nativeGetEnvHandle(appName, packageName, credentials, whitelist)`
   - For other requests: calls `nativeGetOpenerHandle(appName)`
4. Returns a `ParcelFileDescriptor` (file descriptor for Mink IPC socket) and a handle ID

### Binder Interface: `IMinkSocketFd`

Descriptor: `com.qualcomm.qti.qms.api.minksocket.IMinkSocketFd`

```java
ParcelFileDescriptor request(String appName, int[] handleOut) throws RemoteException;
```

- `appName` -- Name of the TrustZone app to connect to
- `handleOut[0]` -- Set to the handle ID on return
- Returns: ParcelFileDescriptor for the Mink IPC channel, or null on failure

### Mink Object API

The underlying Mink IPC protocol uses `IMinkObject` for invoking TrustZone operations:

```java
interface IMinkObject {
    void invoke(int method, byte[][] inBufs, int[] inBufSizes, byte[][] outBufs,
                IMinkObject[] inObjects, IMinkObject[] outObjects) throws InvokeException;
    void retain();
    void release();
    boolean isNull();
}
```

Two implementations:
- `CMinkObject` -- C-backed (JNI). Wraps a native pointer. Used for objects returned from TrustZone.
- `JMinkObject` -- Java-backed (abstract). For implementing Java-side Mink objects that TrustZone can call back into.

Error codes:
| Constant | Value | Meaning |
|----------|-------|---------|
| OK | 0 | Success |
| ERROR | 1 | Generic error |
| ERROR_INVALID | 2 | Invalid argument |
| ERROR_USERBASE | 10 | Base for TA-specific errors |
| ERROR_DEFUNCT | -90 | Object defunct |
| ERROR_ABORT | -91 | Aborted |
| ERROR_BADOBJ | -92 | Bad object reference |
| ERROR_NOSLOTS | -93 | No available slots |
| ERROR_MAXARGS | -94 | Too many arguments |
| ERROR_MAXDATA | -95 | Data too large |
| ERROR_UNAVAIL | -96 | Service unavailable |
| ERROR_KMEM | -97 | Kernel memory error |
| ERROR_REMOTE | -98 | Remote error |

### Credential Verification

`CredentialHelper.getCredentials()` builds a CBOR-encoded map containing:
| Key | Type | Content |
|-----|------|---------|
| 1 | int | Calling UID |
| 2 | int | Application flags (from PackageManager) |
| 3 | string | Package name |
| 4 | array(bytes) | X.509 signing certificate(s) DER-encoded |
| 5 | array(string) | Granted permissions |
| 6 | long | Current timestamp (millis) |

`CredentialHelper.getWhitelist()` reads `/vendor/etc/ssg/tz_whitelist.json` which contains entries like:
```json
{
  "whitelist": [
    { "classId": <int>, "permissions": ["perm1", "perm2"] }
  ]
}
```
Each entry maps a set of required permissions to a TrustZone class ID. Only callers with all listed permissions get access to that class.

### Smali/Java Structure

```
com.qualcomm.qti.qms.service.trustzoneaccess/
  TZAccessService        -- Exported bound service, loads libnative-api.so
  IMinkSocketFdImpl      -- Binder impl, credential checking, native calls

com.qualcomm.qti.qms.api.mink/
  IMinkObject            -- Interface for TrustZone object invocation
  CMinkObject            -- JNI-backed Mink object (native pointer)
  JMinkObject            -- Java-backed Mink object (abstract, for callbacks)

com.qualcomm.qti.qms.api.minksocket/
  IMinkSocketFd          -- AIDL-style binder interface (request method)

com.qualcomm.qti.qms.credential.utils/
  CredentialHelper       -- CBOR-encodes caller credentials, reads tz_whitelist.json

co.nstant.in.cbor/       -- Bundled CBOR library (encoder, builder, model classes)
```

### Development Relevance

**Moderate.** This is the only vendor service that is exported and bindable. In theory, an app could bind to it to access TrustZone trusted applications -- but access is gated by the whitelist at `/vendor/etc/ssg/tz_whitelist.json` and caller signing certificate verification. Without a matching entry in the whitelist, `nativeGetEnvHandle` will reject the request.

Potentially useful for:
- Understanding how DRM or secure storage works on the device
- Knowing that TrustZone access goes through this binder service (not direct `/dev/qseecom` access)
- The Mink IPC protocol details if you ever need to interact with a trusted application
