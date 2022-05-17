# Overview

The Device Fingerprint tool collects thorough insight about the devices associated to a user. Account takeovers and multiple account signups, payments can easily be avoided by applying the Device Fingerprinting module. To implement SEON SDK for Android, follow the steps below.

## Requirements
- Android 4.4 or higher (API level 19)
- INTERNET permission
- _(optional)_ READ_PHONE_STATE permission for device_id (under API 28)
- _(optional)_ ACCESS_WIFI_STATE permission for wifi_ssid (under API 27)
- _(optional)_ ACCESS_NETWORK_STATE permission for network_config
- _(optional)_ ACCESS_FINE_LOCATION (starting from API 29) or ACCESS_COARSE_LOCATION (starting from API 27) permission for wifi_mac_address and wifi_ssid

> __NOTE:__ If the permissions listed are not available the application, the values collected using those permissions will be ignored. We recommend using as much permission as possible based on your use-case to provide reliable device fingerprint.

## Installation

#### Using Gradle

```
dependencies {
  implementation('io.seon.androidsdk:androidsdk:3.2.1') {
    transitive = true
  }
}
```
## Integration

### Kotlin Integration
```
val SESSION_ID = "CUSTOM_SESSION_ID"

// Build with parameters
val seonFingerprint = SeonBuilder().withContext(applicationContext).withSessionId(sessionID).build()

// Enable logging
seonFingerprint.setLoggingEnabled(true)

try {
    seonFingerprint.fingerprintBase64 
}
catch (e : SeonException){
    e.printStackTrace()
}
catch (e : Exception){
    e.printStackTrace()
}

```
### Java Integration

```
final String SESSION_ID = "CUSTOM_SESSION_ID";

// Build with parameters
Seon seonFingerprint = new SeonBuilder()
    .withContext(getApplicationContext())
    .withSessionId(SESSION_ID)
    .build();

// Enable logging
seonFingerprint.setLoggingEnabled(true);

try {
    seonFingerprint.getFingerprintBase64();
} catch (SeonException e) {
    e.printStackTrace();
} catch (Exception e) {
    e.printStackTrace();
}
```
> __NOTE:__ 3.0.6 and earlier versions need to include the following ProGuard rule to avoid JNI runtime errors :

```
-keepclasseswithmembers,includedescriptorclasses class io.seon.androidsdk.service.JNIHandler {
   native <methods>;
}
```

## Changelog

#### 3.2.1
- Improved emulator detection accuracy
- Bugfixes
#### 3.2.0
- Improved, more persistent device identification
- Bugfixes

#### 3.1.0
- Emulator detection improvements
- Bugfixes and compatibility improvements

#### 3.0.6
- Bugfixes and compatibility improvements

#### 3.0.5
- Bugfixes and compatibility improvements

#### 3.0.2
- Improved data collection methods
- Bugfixes and security improvements

#### 3.0.0
- Removed background HTTP request for data transmission, the SDK returns an encrypted, base64 encoded string to use with SEON's REST API
- Removed public key support
- Bugfixes and security improvements

#### 2.1.4
- Improved data collection methods

#### 2.1.3
- Bugfixes and stability improvements

#### 2.1.2
- Improved data collection methods

#### 2.1.1
- Bugfixes and security improvements

#### 2.0.24
- Bugfixes and performance improvements

#### 2.0.23
- Performance improvements

#### 2.0.20
- Removed `start` method
- Added `scanFingerprint` method
- Added public key support
- Enhanced logging settings
- Bugfixes and performance improvements

#### 1.1.6
- Bugfixes and performance improvements

#### 1.1.5
- First stable build