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
  implementation('io.seon.androidsdk:androidsdk:3.0.0') {
    transitive = true
  }
}
```

## Integration

```
Seon seonFingerprint = SeonBuilder.getInstance();

final String SESSION_ID = "[CUSTOM_SESSION_ID]";

// Enable logging
seonFingerprint.setLoggingEnabled(true);

// Set parameters
seonFingerprint.withContext(getApplicationContext()).
        withSessionId(SESSION_ID);

try {
    seonFingerprint.getFingerprintBase64();
} catch (SeonException e) {
    e.printStackTrace();
} catch (Exception e) {
    e.printStackTrace();
}
```

> Context parameter is not required, but highly recommended because some important information depends on the application context.

## Changelog

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
