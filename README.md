# Overview

SEON's Device Fingerprinting SDK collects insights about the device associated to a user and allows developers to integrate device identification into native Android apps.

Account takeovers, multiple account signups and payments can easily be avoided by using SEON's Device Fingerprinting SDK.



## Requirements
- Android 5.0 or higher (API level 21)
- **INTERNET** permission
- _(optional)_ **READ_PHONE_STATE** permission for `is_on_call` and `device_id` (under API 28)
- _(optional)_ **ACCESS_WIFI_STATE** permission for `wifi_ssid` (under API 27)
- _(optional)_ **ACCESS_NETWORK_STATE** permission for `network_config`
- _(optional)_ **ACCESS_FINE_LOCATION** (starting from API 29) and ACCESS_COARSE_LOCATION (starting from API 27) permission for `wifi_mac_address` and `wifi_ssid`
- _(optional)_ **com.google.android.providers.gsf.permission.READ_GSERVICES** for `gsf_id`

> __Note:__ If the optional permissions listed are not available the application, the values collected using those permissions will be ignored. We recommend using as much permission as possible based on your use-case to provide reliable device fingerprint.

## Installation

#### Using Gradle

```
dependencies {
  implementation('io.seon.androidsdk:androidsdk:6.0.1') {
    transitive = true
  }
}
```
## Integration

>### __Note:__ Starting from v6 there is a change in SEON’s API Policy. From now on SEON might introduce new fields in the SDK with minor versions. We advise you to integrate in a way that addition of new fields is handled gracefully.

### Kotlin Integration
```
val sessionID = "CUSTOM_SESSION_ID"

// Build the Seon object with the required parameters
// an application context and a session_id
val sfp = SeonBuilder().withContext(applicationContext).withSessionId(sessionID).build()

// Enable logging
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
Seon seonFingerprint = new SeonBuilder()
    .withContext(getApplicationContext())
    .withSessionId(SESSION_ID)
    .build();

// Enable logging
seonFingerprint.setLoggingEnabled(true);

try {
    seonFingerprint.getFingerprintBase64(fp->{
        //set fp as the value for the session property of the fraud API request.
    });
} catch (SeonException e) {
    e.printStackTrace();
}
```
> __Note:__ 3.0.6 and earlier versions need to include the following ProGuard rule to avoid JNI runtime errors :

```
-keepclasseswithmembers,includedescriptorclasses class io.seon.androidsdk.service.JNIHandler {
   native <methods>;
}
```

# Changelog
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
