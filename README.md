# react-native-nordic-dfu [![npm version](https://badge.fury.io/js/react-native-nordic-dfu.svg)](https://badge.fury.io/js/react-native-nordic-dfu) [![CircleCI](https://circleci.com/gh/Pilloxa/react-native-nordic-dfu.svg?style=svg)](https://circleci.com/gh/Pilloxa/react-native-nordic-dfu) [![Known Vulnerabilities](https://snyk.io/test/github/pilloxa/react-native-nordic-dfu/badge.svg)](https://snyk.io/test/github/pilloxa/react-native-nordic-dfu)

This library allows you to do a Device Firmware Update (DFU) of your nrf51 or
nrf52 chip from Nordic Semiconductor. It works for both iOS and Android, with full Expo support.

For more info about the DFU process, see: [Resources](#resources)

### This is a fork from the main library!

- Please keep in mind the our availability to maintain this fork is limited and is based on our project needs.
- If need the main documentation you can find it [here](https://github.com/Pilloxa/react-native-nordic-dfu).
- This fork contains the latest verisons of `iOSDFULibrary` & `Android-BLE-Library`.

## Expo Support

This library has been optimized to work with Expo SDK 52+ using the Expo plugin system.

### Installation for Expo Projects

```bash
# Using npm
npm install --save https://github.com/Beima-Tech/react-native-nordic-dfu

# Using yarn
yarn add https://github.com/Beima-Tech/react-native-nordic-dfu
```

### Configuration for Expo

In your `app.json` or `app.config.js`, add the plugin:

```json
{
  "expo": {
    "plugins": [
      "react-native-nordic-dfu"
    ]
  }
}
```

The plugin will automatically:
- Configure necessary Bluetooth permissions for iOS
- Add required Bluetooth permissions for Android
- Set up other required configurations for the Nordic DFU process

### Usage with Expo

```javascript
import { NordicDFU, DFUEmitter } from "react-native-nordic-dfu";

// Listen for progress updates
DFUEmitter.addListener("DFUProgress", ({ percent, currentPart, partsTotal, avgSpeed, speed }) => {
  console.log("DFU progress: " + percent + "%");
});

// Listen for state changes
DFUEmitter.addListener("DFUStateChanged", ({ state }) => {
  console.log("DFU State:", state);
});

// Start the DFU process
NordicDFU.startDFU({
  deviceAddress: "C3:53:C0:39:2F:99",
  deviceName: "Your Device Name",
  filePath: "path/to/firmware.zip",
  // Optional parameters
  alternativeAdvertisingNameEnabled: true, // iOS only
  packetReceiptNotificationParameter: 12
})
  .then(res => console.log("Transfer done:", res))
  .catch(error => console.error("DFU error:", error));
```

## React Native New Architecture Support

This library supports React Native's New Architecture (Fabric & TurboModules). The implementation automatically detects whether your app is using the new architecture or the legacy one.

### Requirements

- React Native 0.68 or higher with New Architecture enabled

### Configuration for New Architecture

For iOS:
- Make sure you have enabled the new architecture in your Podfile

For Android:
- Enable the new architecture by setting `newArchEnabled=true` in your `gradle.properties` file

## Regular React Native Installation

If you're not using Expo, install the package:

```bash
npm install --save https://github.com/Beima-Tech/react-native-nordic-dfu
```

or

```bash
yarn add https://github.com/Beima-Tech/react-native-nordic-dfu
```

Then follow the setup instructions in the [Project Setup](#project-setup) section below.

## Project Setup

For non-Expo React Native projects:

`Podfile`:

```ruby
target "YourApp" do
  pod "react-native-nordic-dfu", path: "../node_modules/react-native-nordic-dfu"
  use_frameworks! :linkage => :static
end
```

`AppDelegate.mm`:

```objective-c
#import "RNNordicDfu.h"

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [RNNordicDfu setCentralManagerGetter:^() {
    return [[CBCentralManager alloc] initWithDelegate:nil queue:dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)];
  }];

  // Reset manager delegate since the Nordic DFU lib "steals" control over it
  [RNNordicDfu setOnDFUComplete:^() {
    NSLog(@"onDFUComplete");
  }];
  [RNNordicDfu setOnDFUError:^() {
    NSLog(@"onDFUError");
  }];
  
  // Rest of your AppDelegate code
}
```

## API

### startDFU(options)

Starts the DFU process

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| options.deviceAddress | String | The MAC address for the device that should be updated |
| options.deviceName | String | Optional name of the device in the update notification |
| options.filePath | String | The file system path to the zip-file used for updating |
| options.alternativeAdvertisingNameEnabled | Boolean | Send unique name to device before it is switched into bootloader mode (iOS only) |
| options.packetReceiptNotificationParameter | Number | Number of packets of firmware data to be received by the DFU target before sending a new Packet Receipt Notification |
| options.retries | Number | Android only - number of retries for retrying the operation |
| options.maxMtu | Number | Android only - Maximum MTU size used during the DFU process |

Returns: Promise - resolves with the device address when completed

## Resources

- [DFU Introduction](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v11.0.0/examples_ble_dfu.html?cp=6_0_0_4_3_1 "BLE Bootloader/DFU")
- [Secure DFU Introduction](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.0.0/ble_sdk_app_dfu_bootloader.html?cp=4_0_0_4_3_1 "BLE Secure DFU Bootloader")
- [Nordic Semiconductor DFU Documentation](https://www.nordicsemi.com/Software-and-tools/Software/nRF-Util)

### Issues

- For configuration issues please also check this (https://github.com/Pilloxa/react-native-nordic-dfu/issues/171)

### Resources

- [DFU Introduction](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v11.0.0/examples_ble_dfu.html?cp=6_0_0_4_3_1 "BLE Bootloader/DFU")
- [Secure DFU Introduction](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.0.0/ble_sdk_app_dfu_bootloader.html?cp=4_0_0_4_3_1 "BLE Secure DFU Bootloader")
- [How to create init packet](https://github.com/NordicSemiconductor/Android-nRF-Connect/tree/master/init%20packet%20handling "Init packet handling")
- [nRF51 Development Kit (DK)](http://www.nordicsemi.com/eng/Products/nRF51-DK "nRF51 DK") (compatible with Arduino Uno Revision 3)
- [nRF52 Development Kit (DK)](http://www.nordicsemi.com/eng/Products/Bluetooth-Smart-Bluetooth-low-energy/nRF52-DK "nRF52 DK") (compatible with Arduino Uno Revision 3)

## Expo Usage

This library now includes Expo plugin support, making it easy to use with Expo projects.

### Installation for Expo (SDK 52+)

```bash
npx expo install react-native-nordic-dfu
```

### Configuration

In your `app.json` or `app.config.js`, add the plugin:

```json
{
  "expo": {
    "plugins": [
      "react-native-nordic-dfu"
    ]
  }
}
```

The plugin will automatically configure the necessary permissions for both iOS and Android platforms.

### Usage in Expo

The API usage is the same as for regular React Native applications:

```javascript
import { NordicDFU, DFUEmitter } from "react-native-nordic-dfu";

// Listen for progress updates
DFUEmitter.addListener("DFUProgress", ({ percent, currentPart, partsTotal, avgSpeed, speed }) => {
  console.log("DFU progress: " + percent + "%");
});

// Listen for state changes
DFUEmitter.addListener("DFUStateChanged", ({ state }) => {
  console.log("DFU State:", state);
});

// Start the DFU process
NordicDFU.startDFU({
  deviceAddress: "C3:53:C0:39:2F:99",
  deviceName: "Device Name",
  filePath: "path/to/firmware.zip"
})
  .then(res => console.log("Transfer done:", res))
  .catch(error => console.error("DFU error:", error));
```

Note: On Android, you may need to request runtime permissions for Bluetooth scanning on newer Android versions when using Expo.

## React Native New Architecture Support

This library now supports React Native's New Architecture (Fabric & TurboModules). The implementation automatically detects whether your app is using the new architecture or the legacy one.

### Requirements

- React Native 0.73 or higher with New Architecture enabled

### Usage with New Architecture

The API remains the same whether you're using the new architecture or not. The library will automatically use TurboModules when running in an app with the new architecture enabled.

For iOS:
- Make sure you have enabled the new architecture in your Podfile by setting `use_frameworks! :linkage => :static` and enabling the new architecture flags.

For Android:
- Enable the new architecture by setting `newArchEnabled=true` in your `gradle.properties` file.
