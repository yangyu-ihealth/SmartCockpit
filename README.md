### 1.添加SDK library和SO文件
将您的 SDK jar 文件和 SO 文件添加到您的项目中。

![](https://chenxuewei-ihealth.github.io/ihealthlabs-sdk-docs/assets/images/guide_android_2-0f4717d053bb80d111337fc80777a006.png)

在build.gradle中添加以下内容：
```
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

### 2.添加蓝牙所需权限
可参考Android官方文档，https://developer.android.com/guide/topics/connectivity/bluetooth/permissions

Android API 23~30 需要ACCESS_COARSE_LOCATION 或者 ACCESS_FINE_LOCATION权限
```
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```
Android API 31~34 需要蓝牙和附近设备权限
```
<uses-permission
    android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation"
    tools:targetApi="s" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```
### 3.初始化SDK
```java
public class BaseApplication extends Application {
    
    @Override
    public void onCreate() {
        super.onCreate();
        iHealthDevicesManager.getInstance().init(this,  Log.VERBOSE, Log.VERBOSE);
    }
}
```

### 4.注册扫连监听回调
```java
int callbackId = iHealthDevicesManager.getInstance().registerClientCallback(new iHealthDevicesCallback() {
    
    @Override
    public void onScanDevice(String mac, String deviceType, int rssi, Map manufactorData) {
        
    }

    @Override
    public void onDeviceConnectionStateChange(String mac, String deviceType, int status, int errorID, Map manufactorData) {
        if(status == iHealthDevicesManager.DEVICE_STATE_CONNECTING){
            Log.i("", "Device is connecting!");
        }else if(status == iHealthDevicesManager.DEVICE_STATE_CONNECTED){
            Log.i("", "Device is connected!");
        }else if(status == iHealthDevicesManager.DEVICE_STATE_CONNECTIONFAIL){
            Log.i("", "Device connect failed!");
        }
     }

    @Override
    public void onScanError(String reason, long latency) { }

    @Override
    public void onScanFinish() { }


    @Override
    public void onSDKStatus(int statusId, String statusMessage) { 
        if (iHealthDevicesManager.SDK_STATUS_BLUETOOTH_DISABLE == statusId) {
            Log.i("", "Bluetooth service is disable!");

        } else if (iHealthDevicesManager.SDK_STATUS_LOCATION_DISABLE == statusId) {
            Log.i("", "Location service is disable!");

        } else if (iHealthDevicesManager.SDK_STATUS_BLUETOOTH_PERMISSION == statusId) {
            Log.i("", "Miss android permission: " + statusMessage);

        } else if (iHealthDevicesManager.SDK_STATUS_LICENSE_EXPIRED == statusId) {
            Log.i("", "License is not match with application id or is expired!");

        } else if (iHealthDevicesManager.SDK_STATUS_LICENSE_DEVICE_PERMISSION == statusId) {
            Log.i("", "Need this device permission!");

        } 
    }
});

iHealthDevicesManager.getInstance().addCallbackFilterForDeviceType(mClientCallbackId, iHealthDevicesManager.TYPE_SMART_COCKPIT);
iHealthDevicesManager.getInstance().addCallbackFilterForAddress(int clientCallbackId, String... macs)
```

### 5.蓝牙扫描设备
```java
iHealthDevicesManager.getInstance().startDiscovery(DiscoveryTypeEnum.SMART_COCKPIT);
```

### 6.蓝牙连接设备
```java
iHealthDevicesManager.getInstance().connectDevice(mDeviceMac, iHealthDevicesManager.TYPE_SMART_COCKPIT);
```

### 7.获取设备对象
```java
SmartCockpitControl mSmartCockpitControl = iHealthDevicesManager.getInstance().getSmartCockpitControl(mDeviceMac);
```

### 8.设备状态上报
```java
private iHealthDevicesCallback miHealthDevicesCallback = new iHealthDevicesCallback() {
    @Override
    public void onDeviceNotify(String mac, String deviceType, String action, String message) {
        if (SmartCockpitProfile.ACTION_GET_DEVICE_STATUS.equals(action)) {
            try {
                JSONObject obj = new JSONObject(message);
                //佩戴状态(0:未佩戴，1佩戴)
                int wearingStatus = obj.getInt(SmartCockpitProfile.WEARING_STATUS);
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    } 
}
```

### 9.设备主动上报实时测量数据
```java
private iHealthDevicesCallback miHealthDevicesCallback = new iHealthDevicesCallback() {
    @Override
    public void onDeviceNotify(String mac, String deviceType, String action, String message) {
        if (SmartCockpitProfile.ACTION_REAL_TIME_DATA.equals(action)) {
            try {
                JSONObject obj = new JSONObject(message);
                //心率值(bpm), SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int heartRate = obj.getInt(SmartCockpitProfile.HEART_RATE);
                //血氧值(%), SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int bloodOxygen = obj.getInt(SmartCockpitProfile.BLOOD_OXYGEN);
                //HRV(ms), SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int hrv = obj.getInt(SmartCockpitProfile.HRV);
                //血压高压值(mmHg), SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int sys = obj.getInt(SmartCockpitProfile.SYS);
                //血压低压值(mmHg), SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int dia = obj.getInt(SmartCockpitProfile.DIA);
                //光电容积脉搏波(m/s), SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int ppg = obj.getInt(SmartCockpitProfile.PPG);
                //心电 0：正常 1:心律不齐, SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int ecg = obj.getInt(SmartCockpitProfile.ECG);
                //心电平均心率, SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int ecgHeartRateAvg = obj.getInt(SmartCockpitProfile.ECG_HEART_RATE_AVG);
                //呼吸率 0:正常, SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int breathingRate = obj.getInt(SmartCockpitProfile.BREATHING_RATE);
                //压力指数 0:正常, SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int pressureIndex = obj.getInt(SmartCockpitProfile.PRESSURE_INDEX);
                //疲劳指数 0:正常, SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int fatigueIndex = obj.getInt(SmartCockpitProfile.FATIGUE_INDEX);
                //血管弹性程度 0：正常 1：偏硬, SmartCockpitProfile.DEFAULT_NOT_DETECTED(-1)表示未检测到
                int bloodVesselElasticity = obj.getInt(SmartCockpitProfile.BLOOD_VESSEL_ELASTICITY);

            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    } 
};
```
