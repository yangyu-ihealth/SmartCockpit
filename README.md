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
    public void onDeviceConnectionStateChange(String mac, String deviceType, int status, int errorID, Map manufactorData) { }

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
                //包序号
                int packageIndex = obj.getInt(SmartCockpitProfile.PACKAGE_INDEX);
                //心率值(bpm)
                int heartRate = obj.getInt(SmartCockpitProfile.HEART_RATE);
                //血氧值(%)
                int bloodOxygen = obj.getInt(SmartCockpitProfile.BLOOD_OXYGEN);
                //体温(℃)
                int bodyTemperature = obj.getInt(SmartCockpitProfile.BODY_TEMPERATURE);
                //HRV
                int hrv = obj.getInt(SmartCockpitProfile.HRV);
                //血压高压值
                int sys = obj.getInt(SmartCockpitProfile.SYS);
                //血压低压值
                int dia = obj.getInt(SmartCockpitProfile.DIA);
                //ECG
                int ecg = obj.getInt(SmartCockpitProfile.ECG);
                //压力指数
                int pressureLevel = obj.getInt(SmartCockpitProfile.PRESSURE_LEVEL);
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    } 
};
```
