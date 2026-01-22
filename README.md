# Overview

SEON's Device Fingerprinting SDK collects insights about the device associated to a user and allows developers to integrate device identification into native Android apps.

Account takeovers, multiple account signups and payments can easily be avoided by using SEON's Device Fingerprinting SDK.


## Requirements
- Android 5.0 or higher (API level 21)
- **INTERNET** permission
- _(optional)_ **READ_PHONE_STATE** permission for `device_cellular_id` (under API 28) and the full functionality of `is_on_call`
- _(optional)_ **ACCESS_WIFI_STATE** permission for `wifi_ssid` (under API 27) and more precise unique IDs
- _(optional)_ **ACCESS_NETWORK_STATE** permission for `network_config` for WiFi configurations and **READ_PHONE_STATE** for cellular data configurations
- _(optional)_ **ACCESS_FINE_LOCATION** (starting from API 29) and **ACCESS_COARSE_LOCATION** (starting from API 27) permission for `wifi_mac_address`, `wifi_ssid`, `device_location`*
- _(optional)_ **com.google.android.providers.gsf.permission.READ_GSERVICES** for `gsf_id`

> __Note:__ If the optional permissions listed are not available the application, the values collected using those permissions will be ignored. We recommend using as much permission as possible based on your use-case to provide reliable device fingerprint.

> __*device_location:__ Please see the Geolocation Integration section for more info

## Installation

#### Using Gradle with Groovy

```
dependencies {
  implementation 'io.seon.androidsdk:androidsdk:6.8.2'
}
```

#### Using Gradle with Kotlin DSL

```
dependencies {
    implementation("io.seon.androidsdk:androidsdk:6.8.2")
}
```

### Known issues in previous versions
If there's a duplicate class error due to conflicting versions of the Bouncy Castle dependency in `6.3.0` and `6.4.0`, which could look like the following error message:
```
Duplicate class org.bouncycastle.LICENSE found in modules jetified-bcprov-jdk14-1.77 (org.bouncycastle:bcprov-jdk14:1.77) and jetified-bcprov-jdk18on-1.77 (org.bouncycastle:bcprov-jdk18on:1.77)
```
In this case, please add the following line to the SDK's implementation block to remove SEON's version of the module:
```
exclude group: 'org.bouncycastle', module: 'bcprov-jdk14'
```
---

On earlier versions you might to need to include the following ProGuard rule to avoid JNI runtime errors :
```
-keepclasseswithmembers,includedescriptorclasses class io.seon.androidsdk.service.JNIHandler {
   native <methods>;
}
```


## Integration

>### __Note:__ Starting from v6 there is a change in SEON’s API Policy. From now on SEON might introduce new fields in the SDK with minor versions. We advise you to integrate in a way that addition of new fields is handled gracefully.

### Kotlin Integration
```
val sessionID = "CUSTOM_SESSION_ID"

// Build the Seon object with the required parameters
// an application context and a session_id
//
// Optional - You can optionally set a custom timeout for the SDK's network call
// with passing the following method to SeonBuilder: withDnsTimeout(int timeoutInMillisec)
// Note: Passing 0 here effectively skips the network logic which in turn won't
// populate the Fraud API fields with device_ip_* and dns_ip_* prefixes

val sfp = SeonBuilder().withContext(applicationContext).withSessionId(sessionID).build()

// Optional - Enable logging
sfp.setLoggingEnabled(true)

try {
    sfp.getFingerprintBase64 { seonFingerprint: String? ->
        //set seonFingerprint as the value for the session
        //property of your Fraud API request.
    }
} catch (e : SeonException){
    e.printStackTrace()
}

```
### Java Integration

```
final String SESSION_ID = "CUSTOM_SESSION_ID";

// Build the Seon object with the required parameters
// an application context and a session_id
//
// Optional - You can optionally set a custom timeout for the SDK's network call
// with passing the following method to SeonBuilder: withDnsTimeout(int timeoutInMillisec)
// Note: Passing 0 here effectively skips the network logic which in turn won't
// populate the Fraud API fields with device_ip_* and dns_ip_* prefixes

Seon seonFingerprint = new SeonBuilder()
    .withContext(getApplicationContext())
    .withSessionId(SESSION_ID)
    .build();

// Optional - Enable logging
seonFingerprint.setLoggingEnabled(true);

try {
    seonFingerprint.getFingerprintBase64(fp->{
        //set fp as the value for the session property of the fraud API request.
    });
} catch (SeonException e) {
    e.printStackTrace();
}
```

# Behaviour Monitoring (Optional)
Behaviour Monitoring allows the SEON SDK to be able to detect potentially suspicious user behaviour on the device. The SDK collects data during the session, which is then analyzed to identify potentially fraudulent environments and actions. This feature enhances the SDK’s ability to prevent fraud by detecting various forms of automated or suspicious activity, such as bot usage or device farms.

>__Note:__ For the result of the behaviour evaluation, we are introducing a new response field in the Fraud API response named `suspicious_flags`. It's available in sessions generated by Android SDK version 6.5.0 or later when the session had been generated by the new `startBehaviourMonitoring` and `stopBehaviourMonitoring` interfaces.

The monitoring must be started with calling `startBehaviourMonitoring` wherever you would like to detect suspicious activity in your application and should be stopped with `stopBehaviourMonitoring` whenever it's reasonable. The returned session string should be then used in a Fraud API request as usual. Note: If you call `stopBehaviourMonitoring` without `startBehaviourMonitoring` called previously, the method will exit with `BehaviouralMonitoringException` thrown.

### Possible `suspicious_flags` values:
-	`"possible_automation"`: Suggests that automation tools or scripts may be controlling the device.
-	`"possible_device_farm"`: Suggests that the device might be part of a device farm used for fraudulent activities.
-	`"possible_vishing"`: Flags possible vishing (voice phishing) activity, where the user might be coerced into providing sensitive information.
-	`"possible_ongoing_call"`: Flags possible ongoing phone call, which could be useful information in case the **READ_PHONE_STATE** permission wasn't granted for `is_on_call` field to work. This behaviour based flag does not need any permissions to work, but it's only a best-effort metric.
- **To be continously improved and extended with new signals**

### Java Integration

```
final String SESSION_ID = "CUSTOM_SESSION_ID";

// Build the Seon object with the required parameters
// an application context and a session_id
//
// Optional - You can optionally set a custom timeout for the SDK's network call
// with passing the following method to SeonBuilder: withDnsTimeout(int timeoutInMillisec)
// Note: Passing 0 here effectively skips the network logic which in turn won't
// populate the Fraud API fields with device_ip_* and dns_ip_* prefixes

Seon seonFingerprint = new SeonBuilder()
    .withContext(getApplicationContext())
    .withSessionId(SESSION_ID)
    .build();

// Optional - Enable logging
seonFingerprint.setLoggingEnabled(true);

//To get behaviour based signals, you have to start monitoring before the relevant user journey:
seonFinderprint.startBehaviourMonitoring();

/* ---- Relevant user journey happens here ----
* Note: The behaviour analysis needs time to collect signals on user behaviour,
* so it's advisable to run it at least for a few seconds.
*/

try {
    //When you want to collect the results, call stopBehaviourMonitoring() as you would getFingerprintBase64()
    seonFingerprint.stopBehaviourMonitoring(fp->{
        //set fp as the value for the session property of the fraud API request.
        //fp contains everything you would get from getFingerprintBase64(), plus the occasional behavioural results
    });
} catch (BehaviouralMonitoringException e) {
    //Handle behavioural monitoring failure
    e.printStackTrace();
}
catch (SeonException e) {
    e.printStackTrace();
}
```
### Kotlin Integration
```
val sessionID = "CUSTOM_SESSION_ID"

// Build the Seon object with the required parameters
// an application context and a session_id
//
// Optional - You can optionally set a custom timeout for the SDK's network call
// with passing the following method to SeonBuilder: withDnsTimeout(int timeoutInMillisec)
// Note: Passing 0 here effectively skips the network logic which in turn won't
// populate the Fraud API fields with device_ip_* and dns_ip_* prefixes

val seonFingerprint = SeonBuilder()
    .withContext(applicationContext)
    .withSessionId(sessionID)
    .build()

// Optional - Enable logging
seonFingerprint.isLoggingEnabled = true

// To get behaviour based signals, you have to start monitoring before the relevant user journey:
seonFingerprint.startBehaviourMonitoring()

/* ---- Relevant user journey happens here ----
* Note: The behaviour analysis needs time to collect signals on user behaviour,
* so it's advisable to run it at least for a few seconds.
*/

try {
    // When you want to collect the results, call stopBehaviourMonitoring() as you would getFingerprintBase64()
    seonFingerprint.stopBehaviourMonitoring { fp ->
        // Set fp as the value for the session property of the fraud API request.
        // fp contains everything you would get from getFingerprintBase64(), plus the occasional behavioural results
    }
} catch (e: BehaviouralMonitoringException) {
    // Handle behavioural monitoring failure
    e.printStackTrace()
} catch (e: SeonException) {
    e.printStackTrace()
}

```

## Geolocation Integration (Opt-in)
**To enable SEON’s geolocation feature on your account please reach out the customer success team to enable the functionality on your Admin page and your Scoring Engine!**

> __Note:__ Currently even if the integration has been done correctly there won't be a **device_location** field in the Fraud API response until the feature flag has been set by our customer success team.

> Note: Android SDK Version 6.6.0 or higher and the following Geolocation integration setup is required to use the Geofence API. For further information please visit https://docs.seon.io/api-reference/geofence-api

### Important: Collecting consent and necessary permissions from the end user for location tracking is required.

Use the `SeonGeolocationConfig` object created with `SeonGeolocationConfigBuilder` to customise how geolocation is collected or just use the instance as-is for default values. The following properties are available on object:
- `setGeolocationEnabled` - Setter to enable or disable geolocation collection. Defaults to false.
- `setPrefetchEnabled` - By passing true the geolocation service is going to pre-fetch a valid location as soon as a `Seon` object is created. Defaults to true.
- `setMaxGeoLocationCacheAgeSec` - Sets the maximum allowed age of a location object in seconds. Default value is 600.
- `setGeolocationServiceTimeoutMs` - Sets the maximum time in milliseconds `getFingerprintBase64` or `stopBehaviourMonitoring` can wait for a valid location. Default value is 3000.

Make sure you call `setGeolocationConfig` on `Seon` before calling `getFingerprintBase64` or `stopBehaviourMonitoring` for the SDK to enrich the collected device information with location data.
For the most accurate results when using the Geofence API, prefer Behaviour Monitoring over simply calling `getFingerprintBase64`. Refer to the relevant section in the documentation about how to set up and use Behaviour Monitoring.

To receive status codes about the geolocation collection, pass the `SeonCallbackWithGeo` interface to `getFingerprintBase64` or `stopBehaviourMonitoring`. Example:

#### Java:

```
// You should initialise the Seon SDK, enable geolocation collection and prompt the user for appropriate location permission(s) before trying to retrieve a fingerprint with valid location data.
// ...
seon.getFingerprintBase64(new SeonCallbackWithGeo() {
    @Override
    public void onComplete(String response) {
        // Successfully received fingerprint response with device location data.
    }
    @Override
    public void onCompleteWithGeoFailure(String response, int geoStatusCode) {
        // Successfully received fingerprint response, without valid location data and geolocation service status code.
    }
});
```

#### Kotlin:
```
// You should initialize the Seon SDK, enable geolocation collection, 
// and prompt the user for appropriate location permission(s) before 
// trying to retrieve a fingerprint with valid location data.
// ...

seon.getFingerprintBase64(object : SeonCallbackWithGeo {
    override fun onComplete(response: String) {
        // Successfully received fingerprint response with device location data.
    }

    override fun onCompleteWithGeoFailure(response: String, geoStatusCode: Int) {
        // Successfully received fingerprint response, without valid location data and geolocation service status code.
    }
})
```

The SDK can return the following Geolocation specific status codes in the `onCompleteWithGeoFailure`  callback as the value of `geoStatusCode`:
- `-1` : `Unknown` : An unknown error has occured during geolocation collection.
- `1` : `Fail` : Failed to return location data.
- `2` : `Timeout` : The location service has timed out.
- `3` : `No Permission` : The user has denied the use of location services.
- `4` : `No Provider` : There are currently no available location providers to determine the device’s position.
- `5` : `Disabled` : Location services are disabled on the device.
- `6` : `No Support` : There's no location service support.

### Kotlin Integration
```
// Create a custom Geolocation Config object
val seonGeolocationConfig = SeonGeolocationConfigBuilder()
    .withPrefetchEnabled(true) // When enabled, prefetch location from the API on init for better performance
    .withGeolocationServiceTimeoutMs(3000) // The timeout in milliseconds for the location service
    .withMaxGeoLocationCacheAgeSec(600) // Maximum Location data age permitted in seconds
    .build()

val seon = SeonBuilder()
    .withContext(applicationContext)
    .withSessionId(UUID.randomUUID().toString())
    .withGeolocationEnabled() // must be explicitly enabled to turn on the Geolocation feature
    .withGeoLocationConfig(seonGeolocationConfig) // optional config
    .build()

// Geolocation and SeonGeolocationConfig can also be set later on the Seon object
seon.setGeolocationEnabled(true)
seon.setGeoLocationConfig(seonGeolocationConfig)

```

### Java Integration
```
// Create a custom Geolocation Config object
        SeonGeolocationConfig seonGeolocationConfig = new SeonGeolocationConfigBuilder()
                .withPrefetchEnabled(true) // When enabled, prefetch location from the API on init for better performance
                .withGeolocationServiceTimeoutMs(3000) // The timeout in milliseconds for the location service
                .withMaxGeoLocationCacheAgeSec(600) // Maximum Location data age permitted in seconds
                .build();

        seon = new SeonBuilder()
                .withContext(getApplicationContext())
                .withSessionId(UUID.randomUUID().toString())
                .withGeolocationEnabled() // must be explicitly enabled to turn on the Geolocation feature
                .withGeoLocationConfig(seonGeolocationConfig) // optional config
                .build();

        // Geolocation and SeonGeolocationConfig can also be set later on the Seon object
        seon.setGeolocationEnabled(true);
        seon.setGeoLocationConfig(seonGeolocationConfig);
```


### 16 KB page size compatibility on Google Play
- Starting November 1st, 2025, all new apps and updates to existing apps submitted to Google Play and targeting Android 15+ devices must support 16 KB page sizes on 64-bit devices.
- SEON Android SDK is compatible with the 16 KB page size from version `6.8.0`
- Your application might need to meet further requirements to pass the compatibility check:
    - Based on Google documentation, `Android Gradle Plugin version 8.5 or lower` are only 16 KB page size compatible if you ship with legacy(compressed) native libraries. :exclamation:
    - You can enable legacy(compressed) packaging for native libraries in your `build.gradle` :
    ```
    android {
        ....
        packagingOptions {
            jniLibs {
                useLegacyPackaging true
            }
        }
    }
    ```
    - Warning: When you use compressed shared libraries, your app takes up more space when installed, as libraries are extracted from the APK and copied onto disk. Your app might more frequently fail to install because this increase in disk usage means there is less space on device. To avoid this, upgrade to AGP version 8.5.1 or higher. :exclamation:
    - If you are using `Android Gradle Plugin version 8.5.1 or higher`, you are not required to use legacy packaging. :heavy_check_mark:
- To stay Google Play compliant, please always refer to the latest [Google documentation on this matter](https://developer.android.com/guide/practices/page-sizes).


# Changelog
## 6.8.2
- Fixed return value of `cpu_hash` when the field is empty. Now it correctly returns null instead of the hashed empty string ( `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`)
- Fixed return value of `pasteboard_hash` when the pasteboard is empty. Now it correctly returns null instead of the hashed empty string ( `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`)
 > ⚠️ __Note:__ Please make sure to expect null values instead of the empty hash (`e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`) from this version and adjust any rules you might have in the Scoring Engine including either `cpu_hash` and `pasteboard_hash` as needed!
- Various internal detection improvements
- Internal changes and improvements for upcoming features.

## 6.8.1
- Removed the following permissions from our `AndroidManifest.xml`:
    > __Note:__ If you depend any of the optional features connected to these permissions, from this version onwards you need to add them to your `AndroidManifest.xml` manually
    > Please, read carefully what features are connected to these permissions in our [Requirements section](#requirements).
    - `android.permission.ACCESS_FINE_LOCATION`
    - `android.permission.ACCESS_COARSE_LOCATION`
    - `android.permission.READ_PHONE_STATE`
    - `android.permission.READ_PHONE_NUMBERS`
- Internal changes and improvements.

## 6.8.0
- Added 16 KB page size support to ensure Google Play compatibility beyond November 1st, 2025.
    - SEON Android SDK is fully compatible with 16 KB page size from this version (`6.8.0`).
    - There are more requirements to be 16 KB page size compatible. Please carefully read the [Google documentation](https://developer.android.com/guide/practices/page-sizes).
    - You can also check out our short section on this topic [here](#16-kb-page-size-compatibility-on-google-play).
- Improved app cloning detection to cover advanced tools.
- Internal changes and improvements.

## 6.7.1
- Optimised payload size in some rare, edge cases.
- Internal changes and improvements.

## 6.7.0
- Added payload compression which considerably reduces output size.
- Improved behaviour monitoring exception messages.
- Introduced fallback mechanism for `is_on_call` when `READ_PHONE_STATE` permission is not granted.
- Improved emulator detection.
- Internal changes and improvements for upcoming features.

## 6.6.1
- Introducing Supremo remote control detection:
    - `remote_control_provider` possible return values has been extended with `Supremo`.
- Improved SDK stability.
- Internal changes and improvements for upcoming features.

## 6.6.0
- Introducing the following new response fields to help determine the security and integrity of the device:
  - `is_app_cloned` : Indicates whether the current app instance integrating the SDK is a cloned version, helping identify potential fraud and validate the application integrity.
  - `system_integrity` : Indicates the integrity of the device, helping to assess its security state. Possible values are:
    - `ORIGINAL` : Indicates a high-confidence secure device state.
    - `POSSIBLY_COMPROMISED` : Suggests a suspicious device state with a modified bootloader but no other direct evidence of compromise.
    - `COMPROMISED` : Indicates a highly compromised state. This is a strong indicator that the OS has been tampered with.
    - `UNKNOWN` : The integrity status cannot be determined.
- Introducing new response field: `true_device_id`.
- Added new callback signature type `SeonCallbackWithGeo` to handle geolocation status responses.
- Raised the SDK's targetSdkVersion to API level 34.
- Optimized resource handling.
- Fix `physical_memory` inconsistencies on some devices.
- Minor fixes and improvements.
- Internal changes.

## 6.5.1
-	Fixed linter errors for invalid package reference (`javax.naming..`)
-	Internal improvements and changes for upcoming features.

## 6.5.0
-	Added Behaviour Monitoring feature, which allows detection of suspicious device behaviour.
-	New `suspicious_flags` field in the Fraud API response. Possible values include:
    -	`"possible_automation"`: Indicates potential automation tool or script usage.
    -	`"possible_device_farm"`: Suggests the device might be part of a device farm.
    -	`"possible_vishing"`: Flags possible vishing activity.
    -	`"possible_ongoing_call"`: Flags possible ongoing phone call.
-	Added public interfaces for [Behaviour Monitoring](#behaviour-monitoring-optional):
      -	startBehaviourMonitoring()
      -	stopBehaviourMonitoring(fp->{...})
-	Internal improvements and changes for upcoming features.

## 6.4.2
- Fixed a rare issue with payload generation
- Improved emulator detection
- Internal changes for upcoming features.
## 6.4.1
- Fixed strict mode policy warning
- Removed transitive dependency on Bouncy Castle
## 6.4.0
- Improved emulator detection
- Added VPN detection logic and the related response field:
  - `vpn_state` Returns the vpn connection state of the device. The possible return values are:
    - `"UNKNOWN"`
    - `"CONNECTED"`
    - `"NOT_CONNECTED"`
- Added proxy detection logic and the following related response fields:
  - `proxy_state` Returns the proxy connection state of the device. The possible return values are:
    - `"UNKNOWN"`
    - `"CONNECTED"`
    - `"NOT_CONNECTED"`
  - `proxy_address` Returns a String value of the connected proxy's host address followed by the port. The value can be null if no proxy connection has been found. Example value: `"111.11.11.11:8008"`
- Added the following new device information response fields:
  - `first_api_level` Returns an integer value of the stock API level the device has shipped with. The value can be -1 if the api level can't be determined.
  - `power_source` Returns a predefined String value of the type of the currently plugged in power source. Possible values are:
    - `"NO DATA"`
    - `"AC"`
    - `"USB"`
    - `"WIRELESS"`
    - `"DOCK"`
- Improved clarity of client side exceptions regarding incorrect SDK integrations.
- Internal changes for upcoming features.

## 6.3.0
- Added GeoLocation feature, the SDK now optionally can retrieve the device's location. See the documentation about how to use it.
- Internal improvements and changes for upcoming features
- Added **Bouncy Castle - jdk14 v1.77** as a new dependency. If it causes any dependency conflicts, please follow the documentation about resolving the conflict.

## 6.2.0
- Improved Fingerprint execution time and general SDK performance
- Introduced optional DNS timeout confing on SeonBuilder
- Internal changes

## 6.1.2
- Added consumer ProGuard rules to prevent stripping of SDK resources

### Other
- Internal changes to prepare for upcoming feature improvements

## 6.1.1
- Fixed an encoding issue related to `region_timezone` field

## 6.1.0
- Improved emulator detection based on high volume data analysis
- Fine-tuned root detection's sensitivity

### Other
- Internal changes to prepare for upcoming features

## 6.0.3
- Improved auto clicker detection
- Improved emulator detection for:
  - Windows Subsystem for Android
  - Redfinger Cloud
  - Bliss OS
  - Virtual Box
  - VMWare
  - Andy
- Fix occasional StaleDataException on Samsung devices

## 6.0.2

### New features and improvements
  - Optimised the performance of emulator detection
  - Improved and extended root detection
  - Improved `device_hash` uniqueness

### Bugfixes
- Fixed a bug where the SDK's emulator detection returned false positive results in some rare cases.
- Fixed `device_hash`'s occasional non-uniqueness, which resulted in different physical devices generating the same `device_hash` in rare scenarios.
> __Note:__ This is NOT a breaking change to the `device_hash` property, the hash only changes for approximately less than `0.5%` of all devices, where the `device_hash` was not unique previously.

### Other
- Internal changes to prepare for upcoming features

## 6.0.1
### Important Integration changes

- #### Starting from v6 there is a change in SEON’s API Policy. From now on SEON might introduce new fields in the SDK with minor versions. We advise you to integrate in a way that addition of new fields is handled gracefully.

- #### `device_hash` field is calculated differently, resulting in different values for a given device. This means these values are going to break between versions.
- #### Raised minimum API level to 21
- #### Removed deprecated interface members:
  - `SeonBuilder.seon`
  - `SeonBuilder.getInstance()`

### New features and improvements
  - Improved, extended and optimised the performance of emulator detection
  - Improved and extended root detection with included detection of popular rooting tools
  - Introducing new auto clicker detection
  - Introducing list of potentially interfering applications
  - Introducing remote control and screen mirroring detection
  - Introducing call status detection
  - Introducing various new response fields
  - General performance improvements
  - Improved stability of device hash
  - Improved error handling


### New response fields
- `biometric_status` Indicates the status of biometric authentication on the device.
- `bootloader_state` Bootloader lock state, calculation based on system properties.
- `build_model` A human-readable name that represents the marketing or consumer-facing name of the device.
- `developer_options_state` Returns the state of the developer options setting.
- `device_orientation` Returns the current orientation of the device
- `gsf_id` Returns a unique identifier which only changes after a factory reset is performed on the device.

  ***Requires permission: com.google.android.providers.gsf.permission.READ_GSERVICES***

- `interfering_apps` Contains the list of installed applications that were given permissions to potentially interfere with other applications on the device by our metric. *Accurate results to the given metric. Applications might interfere with the host application through different methods/permissions, which are not detected here.*
- `is_click_automator_installed` Returns true if the SDK detects enabled click automator applications known to us.
- `is_keyguard_secure` Indicates whether the keyguard is secured by a PIN, pattern or password or a SIM card is currently locked.
- `is_nfc_available` Returns whether the device has NFC hardware available.
- `is_nfc_enabled` Returns whether the device has NFC functionalities enabled.
- `is_on_call` Returns true or false depending on whether the device was on a call while collecting the fingerprint. Detects both cellular and VOIP calls.

  ***Requires permission: android.permission.READ_PHONE_STATE***
- `is_remote_control_connected` Returns whether the device is being remotely controlled by a known remote control application at the time of the fingerprint.
- `is_screen_being_mirrored` Returns whether the screen of the device is being mirrored to an additional display. This is triggered by screen sharing, screen mirroring to a television, using a display through HDMI connection, etc. This might be the indicator of the user’s screen content is being visible to someone else in some way.
- `remote_control_provider` If is_remote_control_connected is true, this field returns the readable name of detected remote control application, otherwise null.
- `timezone_identifier` Returns the current system timezone’s geopolitical region ID.
- `usb_cable_state` Shows if the device is currently being connected to a PC/other device by a USB-cable.
- `usb_debugging_state` State of the USB debugging toggle in developer options.

### Bugfixes

- Fixed a bug where the SDK returned current audio level incorrectly
- Fixed incorrect display metrics for screen_width for some devices
- Fixed incorrect display metrics for screen_height for some devices
- Fixed battery_voltage issue where sometimes the returned value returned the voltage in Volts instead of mV
- Fixed an error on some devices on API level 27 or lower
- Fixed an error where a payload couldn't be generated when the Seon object had not been initialized on the main thread
- Fixed an uncaught exception on older devices
- Fixed `screen_brightness` return values on some devices
- An exception is now correctly thrown by the SDK if the session_id is either null or empty

### Other

- Internal changes to prepare for upcoming features and improvements

## 5.1.3
- Fixed a few Context type related issues
- Fixed a possible Cursor object leak
- Fixed network related strict mode warnings
- Minor performance improvements

## 5.1.2
- Fixed a concurrency issue if the Seon object is not instantiated on the main thread

## 5.1.1
- Fixed some rare occasions where fingerprint generation can get stuck on some environments with a 'Fingerprint could not be generated' message

## 5.1.0
- Rootbeer dependency updated to 0.1.0
- Minor fixes and improvements

## 5.0.3
- Fixed an error on some older devices
- Fixed missing device name warning logs in the console even when logging was set to false
- Improved device name and manufacturer identification

## 5.0.2
- Fixed an issue where some older devices couldn't generate a fingerprint
- Fixed an issue with Bluestacks emulator

## 5.0.1
- Improved error handling
- Bugfixes

## 5.0.0
### _**Breaking changes:** The newly computed device hash won't be compatible with previous versions_
- Improved device hash persistence and accuracy
- Minor fixes and improvements

## 4.1.2
- Fixed an issue where some older devices couldn't generate a fingerprint

## 4.1.1
- Fixed an issue with the latest emulator detection changes

## 4.1.0
- Improved emulator detection accuracy and troubleshooting
- Device information dependency updates
- Socket timeout exception handling

## 4.0.0
- Changed fingerprint method to be async, improving speed and reliability
- device_ip fields are now available
- Raised SDK build target API level to 33
- Performance improvements
- Bugfixes

## 3.2.1
- Improved emulator detection accuracy
- Bugfixes

## 3.2.0
- Improved, more persistent device identification
- Bugfixes

## 3.1.0
- Emulator detection improvements
- Bugfixes and compatibility improvements

## 3.0.6
- Bugfixes and compatibility improvements

## 3.0.5
- Bugfixes and compatibility improvements

## 3.0.2
- Improved data collection methods
- Bugfixes and security improvements

## 3.0.0
- Removed background HTTP request for data transmission, the SDK returns an encrypted, base64 encoded string to use with SEON's REST API
- Removed public key support
- Bugfixes and security improvements

## 2.1.4
- Improved data collection methods

## 2.1.3
- Bugfixes and stability improvements

## 2.1.2
- Improved data collection methods

## 2.1.1
- Bugfixes and security improvements

## 2.0.24
- Bugfixes and performance improvements

## 2.0.23
- Performance improvements

## 2.0.20
- Removed `start` method
- Added `scanFingerprint` method
- Added public key support
- Enhanced logging settings
- Bugfixes and performance improvements

## 1.1.6
- Bugfixes and performance improvements

## 1.1.5
- First stable build
