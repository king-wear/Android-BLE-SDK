# Android-BLE-SDK

## Installation
1. Drag **HwBluetoothSDK.framework** into your project.
2. Swift: ```import HwBluetoothSDK```<br>
   Objc: ``` #import <HwBluetoothSDK/HwBluetoothSDK.h> ```

### Requirements
iOS 9.0+

## Usage
Each API is a static method, which is called through BluetoothSDK#method. Try to call SDK methods in the same thread, such as the main thread. All time-consuming operations of the SDK will be performed in the sub thread, so don't worry about the problem of performance caused by calling methods in the main thread.
### Setup SDK
1. Init the SDK: ```
   /**
     * Init SDK, it is usually called in onCreate of class application.
     *
     * @param application Application
     * @param maxMTU      each product has a max mtu, please contact us.
     */
    public static void init(Application application, int maxMTU)
 ```<br>
**It should be called when APP did finish launching.**
2. When you no longer need to use the SDK, call ```[[HwBluetoothSDK sharedInstance] destroySDK] ```<br>
**It should be called when you will never use SDK to do anything, for example, when APP will be terminated.**
3. Get SDK version by call ```[[HwBluetoothSDK sharedInstance] version]```.

### Connection

It is very important to note that scanning and connection can only be performed when HwBluetoothState is HwBluetoothStateAvailable.
You can monitor the changes of HwBluetoothState through ```addBluetoothStateChangedCallback```

1. You can make a ble connection by calling ```connectWithBleName``` to pass in the Bluetooth name of the watch.
2. Or you can call 'connectWithDevice' with the device returned by ```scanWithCallback``` API.
3. You can monitor Bluetooth connection status by ```addBluetoothConnectionStateChangedCallback```.
4. ```connected``` Get bluetooth connection status.
5. ```getPhoneConnectedDevices``` Get the connected devices in the Bluetooth settings of the mobile phone.

After connected, you can call any APIs to read/write data.

### Bind the Watch
Each customer has its own binding process, and our SDK also allows customers to define their own binding process, as follows:
* ```startBindDeviceWithCallback```/```startQRBindDeviceWithCallback``` start to bind watch, you can do anything before calling this API.
```startQRBindDeviceWithCallback```: Bind watch by QR code.<br>
```startBindDeviceWithCallback```: Bind watch by scanning results.
* ```endBindDeviceWithCallback``` end binding watch. you can do anything before calling this API.

We provide the reference binding process as follows:
1. Get watch basic info, such as ID/Firmware Version/Type.
2. Start binding.
3. Set watch basic info, such as user's height/weight/age, set watch's time.
4. End binding.

**For example:**
1. ```getDeviceIdWithCallback```;
2. ```getFirmwareVersionWithCallback```;
3. ```getDeviceTypeWithCallback```;
4. ```startBindDeviceWithCallback```;
5. ```setDeviceTime```;
6. ```setUserInfo```;
7. ```endBindDeviceWithCallback```;

### Health Data
There are many pieces of health data, so when obtaining health data, you need to obtain the number first. Of course, this number only needs to be called once each time.

* ```getHealthDataCountWithCallback```, it will callback all health data count.
* ```getActivities```, get step/calorie/distance/duration health data.
* ```getSleeps```, get sleep data.
* ```getHeartrates```, get heart rate data.
* ```getHeartrateFatigues```, get stress & spo2 data.
* ```deleteXXXX```, after getting each kind of health data, the data in the watch should be deleted in time (this will not affect the display on the watch)

#### Data model
```HwActivity```: Please check HwActivity.h header file;<br>
```HwSleep```: Please check HwSleep.h header file;<br>
```HwHeartRate```: Please check HwHeartRate.h header file;<br>
```HwHeartrateFatigue```: Please check HwHeartrateFatigue.h header file.<br>

### Settings
The APIs of all setting methods start with set. You can find them through the header file or consult us directly.

### Workout
#### Get workout data
1. ```getWorkoutsWithCallback```
2. ```deleteWorkoutsWithCallback```
3. APIs of other features are coming soon...

#### Data model
```HwWorkout```: Please check HwWorkout.h header file.

### Watchface
* Get the currently displayed watchface: ``` getCurrentWatchfaceIndexWithCallback ```;
* Set the currently displayed watchface: ``` setCurrentWatchfaceByIndex ```.

#### Custom Watchface
```
- (void) otaCustomWatchface:(HwCustomWatchface *_Nonnull)customWatchface
           progressCallback:(void(^_Nullable)(float f))progressCallback
             finishCallback:(void(^_Nullable)(BOOL b, NSError * _Nullable error))finishCallback;
```

#### Online Watchface
```
- (void) otaOnlineWatchface:(NSData *_Nonnull)binData
           progressCallback:(void(^_Nullable)(float f))progressCallback
             finishCallback:(void(^_Nullable)(BOOL b, NSError * _Nullable error))finishCallback;
```

### OTA
```
/**
 Each kind of Ota data needs to be assembled into HwOtaDataModel, and each kind of data can have multiple.
 1. The API will first ask whether the watch can OTA, and this step will call back readyCallback;
 2. This API will transfer data to the watch (verify while transmitting);
 3. This API will call back finishcallback to inform app of Ota results.
 Please note: if the readyCallback callback result is NO (the device does not allow OTA, or other exceptions occur), finishCallback will not be called again.
 */
- (void) otaWithDataModels:(NSArray<HwOtaDataModel *> *_Nonnull)dataModels
             otaDeviceName:(NSString *_Nonnull)otaDeviceName
             readyCallback:(HwBoolCallback _Nonnull)readyCallback
          progressCallback:(HwBCFloatCallback _Nullable)progressCallback
            finishCallback:(HwBoolCallback _Nonnull)finishCallback;
```
**HwOtaDataModel:**
```
+ (HwOtaDataModel *)dataModelWithType:(HwOtaType)type
                                 data:(NSData *)data;
```
