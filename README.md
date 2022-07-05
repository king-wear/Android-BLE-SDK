# Android-BLE-SDK

## Installation
Copy ```BluetoothSDK-release.aar``` to libs folder of your project.
Add a path to the dependency in your app's build.gradle file. For example:
```
dependencies {
   implementation files('libs/BluetoothSDK-release.aar')
}
```
### Requirements
* minSdkVersion: 21+
* targetSdkVersion: 31+

## Usage
Each API is a static method, which is called through BluetoothSDK#method. Try to call SDK methods in the same thread, such as the main thread. All time-consuming operations of the SDK will be performed in the sub thread, so don't worry about the problem of performance caused by calling methods in the main thread.
### Setup SDK
1. Init the SDK: 
```
// Init SDK, it is usually called in onCreate of class application.
//
// @param application Application
// @param maxMTU each product has a max mtu, please contact us.
public static void init(Application application, int maxMTU);
 ```

2. When you no longer need to use the SDK, call ```BluetoothSDK.destroy ```<br>
**It should be called when you will never use SDK to do anything, for example, when APP will be terminated.**
3. Get SDK version by call ```BluetoothSDK.getVersion```.

### Connection
#### Permission requirements:
``` BLUETOOTH ``` ```BLUETOOTH_ADMIN ``` ```BLUETOOTH_SCAN ``` ```BLUETOOTH_CONNECT ``` ```ACCESS_FINE_LOCATION ``` ```ACCESS_COARSE_LOCATION ```
#### Summary
1. You can make a BLE connection by calling ```BluetoothSDK.connect(@NonNull String macAddress, @NonNull final ConnectCallback callback)``` to pass in the Bluetooth mac address of the watch.
2. Or you can call ```BluetoothSDK.connect(@NonNull Device device, @NonNull final ConnectCallback callback)``` with the device returned by ```BluetoothSDK.scan``` API.
3. You can monitor Bluetooth connection status by ```addConnectionStateListener(ConnectionStateCallback listener)```.
4. ```BluetoothSDK.isConnected``` Get bluetooth connection status.
5. ```BluetoothSDK.getConnectedDevice``` Get the connected device.

After connected, you can call any APIs to read/write data.

#### Reconnection
The SDK itself will not actively reconnect the watch, but by monitoring the connection status and calling back to the app. The app will decide whether to reconnect after receiving the callback of the connection state changed.

For example, when users unbind their watches, there is no need to reconnect. When the normal chain is broken, the app calls the SDK method to initiate the connection to achieve the effect of reconnection.

If you need to reconnect after disconnection, you need to call ```BluetoothSDK.reconnect```.The processing mechanismthis between ```BluetoothSDK.connect``` and ```BluetoothSDK.reconnect``` is different;
<br>
```BluetoothSDK.connect``` is to connect the watch directly;
<br>
```BluetoothSDK.reconnect```:
* If the app is in the foreground:
Connect directly. If the connection fails, the next time it is called,
it will scan first, and then connect it after the watch is scanned. If the watch cannot be scanned, it will directly return to the error that cannot be scanned;

* If the app is in the background:
After calling reconnect, the connection operation will be delayed for 6 seconds.

**Tips:**
<br>
```Reconnect the result of each connection will be called back through addconnectionstatelistener.```

**Question: Why are there two different methods, connect and reconnect?**
Because on some mobile phones, such as Huawei P20, P30 and other mobile phones, very fast connection / disconnection will lead to the wrong connection state between the mobile phone and the watch, which can be restored only after Bluetooth is switched on and off.

### Bind the Watch
Each customer has its own binding process, and our SDK also allows customers to define their own binding process, as follows:
* ```BluetoothSDK.startBind(BoolCallback callback)``` start to bind watch, you can do anything before calling this API.
* ```BluetoothSDK.endBind(BoolCallback callback)``` end binding watch. you can do anything before calling this API.

We provide the reference binding process as follows:
1. Get watch basic info, such as ID/Firmware Version/Type.
2. Start binding.
3. Set watch basic info, such as user's height/weight/age, set watch's time.
4. End binding.

**For example:**
1. ```getDeviceID```;
2. ```getFirmwareVersion```;
3. ```getDeviceType```;
4. ```startBind```;
5. ```setDeviceTime```;
6. ```setUserInfo```;
7. ```endBind```;

### Health Data
There are many pieces of health data, so when obtaining health data, you need to obtain the number first. Of course, this number only needs to be called once each time.
#### You can call ```getActivityData``` to get all the health data, such as activity, heart rate, sleep, pressure, spO2. It will callback when one kind of data finish fetching and it will keep fetching data(can not be cancelled) regardless of any failure, until all done.
#### Or you can get any single kind of health data by the methods as follow:
* ```getHealthDataCountWithCallback```, it will callback all health data count.
* ```getActivities```, get step/calorie/distance/duration health data.
* ```getSleeps```, get sleep data.
* ```getHeartrates```, get heart rate data.
* ```getHeartrateFatigues```, get stress & spo2 data.
* ```deleteXXXX```, after getting each kind of health data, the data in the watch should be deleted in time (this will not affect the display on the watch)
  
If you need to display health data by hour, you only need to group by hour through the time attribute of health data.

#### Data model
```Sport```: Please check ```com.huawo.sdk.bluetoothsdk.interfaces.ops.models.Sport```;<br>
```Sleep```: Please check ```com.huawo.sdk.bluetoothsdk.interfaces.ops.models.Sleep```;<br>
```Heartrate```: Please check ```com.huawo.sdk.bluetoothsdk.interfaces.ops.models.Heartrate```;<br>
```Hrv```: Please check ```com.huawo.sdk.bluetoothsdk.interfaces.ops.models.Hrv```.<br>

### Settings
The APIs of all setting methods start with set: setXXXX(Boolean Callback)

### Workout
#### Get workout data
1. ```getWorkouts(WorkoutsCallback callback)```
2. ```delWorkouts(BoolCallback callback)```
3. APIs of other features are coming soon...

#### Data model
```Workout```: Please check ```com.huawo.sdk.bluetoothsdk.interfaces.ops.models.Workout```

### Incoming call
* If BT is connected, App do not need to do anything.
* If BT is disconnected and App want to inform the watch, call 
```
/**
 * Inform the watch there is an incomming call.
 * @param name who
 * @param phoneNumber phoneNumber
 * @param callback BoolCallback
*/
public static void incommingCall(String name, String phoneNumber, BoolCallback callback);
``` 
**Hand Up Call**
```
/**
 * Inform the watch to hang up the call
 * @param name who
 * @param phoneNumber phoneNumber
 * @param callback BoolCallback
*/
public static void hangUpCall(String name, String phoneNumber, BoolCallback callback);
```
### Push Message
```
/**
 * Call it when you want the device to pop up a message
 * [社交消息推送]
 * @param message SocialMessage Try to fill in all the information
 * @param callback BoolCallback
*/
public static void pushMessage(SocialMessage message, BoolCallback callback);
```
**Example:**
Push a WeChat message sent by Mr.Li and content is 'Hello'
```
SocialMessage sm = new SocialMessage();
sm.setType(SocialType.Wechat.getValue());
sm.setTime(new Date().getTime());
sm.setId(1);
sm.setTitle("Mr.Li");
sm.setContent("Hello");
BluetoothSDK.pushMessage(sm, new BoolCallback()
```

### Watchface
* Get the currently displayed watchface: ``` getCurrentWatchface(IntValueCallback callback) ```;
* Set the currently displayed watchface: ``` switchWatchfaceBy(int id, BoolCallback callback) ```.

#### Custom Watchface
```
setCustomWatchface(CustomWatchface watchface, SetCustomWatchfaceCallback callback);
```

#### Online Watchface
```
setOnlineWatchface(byte[] data, OtaCallback callback);
```

### OTA
```
/**
 Each kind of Ota data needs to be assembled into OtaData, and each kind of data can have multiple.
 1. The API will first ask whether the watch can OTA, and this step will call back ```onReady```;
 2. This API will transfer data to the watch (verify while transmitting), call back ```onFail```when any error occurs;
 3. This API will call back ```onSuccess/onFail``` to inform app of Ota results.
 Please note: When ```onFail``` is called, ota will stop.
 */
ota(List<OtaData> otaDataList, OtaCallback callback);
```
**OtaData:**
```
public OtaData(OtaDataType type, byte[] data);
```
