

# Face recognition API V2.0

## [DEMO|github](https://github.com/T-Firefly/FaceApiDemoV2/blob/master/README_EN.md)

## Hardware Interface API

### 1. IDCard/ICCard

* Connected devices
>Start the monitoring service and monitor the card swiping operation, it is recommended to execute in the on Resume() method.

```

/*
Open the background monitoring service
*/
IDCardUtil.getInstance().bindIDCardService(Context context);

/*
The NFC module is reported in the form of key value
*/
@Override
public boolean dispatchKeyEvent(KeyEvent event) {
if (IDCardUtil.getInstance().handleEvent(event)) {return true;}

return super.dispatchKeyEvent(event);
}
```



* Check function support
>Since the monitoring service is started asynchronously, it is not necessarily accurate to perform function support detection immediately after connecting the device.
>It is recommended to check the function support again in the monitoring callback of 6 on Machine Connect.

```
/*
Check if the device supports ID card recognition
boolean: true means support, false means not support
*/
boolean result =IDCardUtil.getInstance().isSupportIDCard();

```
```
/*
Check if the device supports ICCard recognition
boolean: true means support, false means not support
*/
boolean result = IDCardUtil.getInstance().isSupportICCard();

```


* Set swipe mode
> Although the ID card module supports both ID card and IC Card, it does not support simultaneous reading of both, and the card reading mode needs to be set as required
>NFC module only supports ICCard.

```
/*
According to the specified reading method when swiping the card;
There are two main types：READCARD_MODE_IDENTITY_CARD and READCARD_MODE_IC_CARD；

IDCardConfig.READCARD_MODE_IDENTITY_CARD = 0;  //IDCard
IDCardConfig.READCARD_MODE_IC_CARD = 1; //ICCard
IDCardConfig.READCARD_MODE_IDENTITY_CARD_UUID = 2; //IDCard UUID 
*/
IDCardUtil.getInstance().setModel(int readMode);
```


* Set the endian mode for reading IC Card
>Set the endian mode when reading the IC Card, the default is big endian.

```
/*
boolean:true Bigendian, false Little endian
*/
IDCardUtil.getInstance().setICCardEndianMode(boolean useBig);
```



* Set up listening and callback functions

```
/*
Binding credit card monitoring callback
context 
callback Listen callback
*/
IDCardUtil.getInstance().setIDCardCallBack(IDCardUtil.IDCardCallBack callBack);

//Listen callback
IDCardUtil.IDCardCallBack callBack = new IDCardUtil.IDCardCallBack() {

//The monitoring service is started asynchronously, if a card reading device is found after startup, it will call back on Machine Connect
@Override
public void onMachineConnect() {
    Log.i("firefly", "onMachineConnect ");
}

//Callback when ID card is swiped
@Override
public void onSwipeIDCard(final IDCardBean info) {
    Log.i("firefly",
        "picture:"+               + "\n" +
        "name：" + info.getName() + "\n" +
        "sex：" + info.getSex() + "\n" +
        "nation：" + info.getNation() + "\n" +
        "birthDate：" + info.getBirthDateStr() + "\n" +
        "address：" + info.getAddress() + "\n" +
        "number：" + info.getNum() + "\n" +
        "issue：" + info.getIssue() + "\n" +
        "expiration date：" + info.getCreateTimeStr() + "-" + info.getValidTimeStr() + "\n" +
        "picture:"+               + info.getPhoto() + "\n" );
}

// Execute callback when swiping IC card
@Override
public void onSwipeICCard(final ICCardBean info) {
    Log.i("firefly", "onSwipeICCard IC=" + info.getIcID());
}

// When setting the card reading mode to READCARD_MODE_IDENTITY_CARD_UUID Time, swipe ID card callback
@Override
public void onSwipeIDCardUUID(final String uuid) {
    Log.i("firefly", "onSwipeIDCardUUID uuid=" + uuid);
}
};

```


* Disconnect
>Remove the monitoring callback and stop the monitoring service, it is recommended to execute in the on Stop () method.

```
IDCardUtil.getInstance().setIDCardCallBack(null);
IDCardUtil.getInstance().unBindIDCardService(Context context);

```



### 2. Fill light control

There are 4 types of fill light: infrared fill light, white fill light, red fill light and green fill light.

* Check whether the device supports infrared fill light;

```
/*
Boolean: true means support, false means not support
*/
HardwareCtrl.isSupportInfraredFillLight();

```



* Operation of infrared fill light;


```
/*
isChecked true means open, false close
*/
HardwareCtrl.setInfraredFillLight(isChecked);

```



* Check whether the device supports the brightness adjustment of the white fill light, and obtain the range of brightness adjustment;


```
/*
Boolean:true Indicates that infrared fill light is supported, false indicates that it is not supported
*/
HardwareCtrl.isFillLightBrightnessSupport();

/*
Support the maximum value of brightness adjustment;
*/
HardwareCtrl.getFillLightBrightnessMax();

/*
Support the minimum value of brightness adjustment;
*/
HardwareCtrl.getFillLightBrightnessMin();

```



* White fill light operation;


```
/*
On/Off;
isChecked true means open and set the maximum value; false close
*/
HardwareCtrl.ctrlLedWhite(isChecked);

/*
On/Off, if brightness adjustment is supported, the corresponding brightness value can be passed in when the light is turned on;
isChecked  true means open and set the maximum value; false close
*/
HardwareCtrl.ctrlLedWhite(isChecked,brightness);

```



* Red fill light operation;


```
/*
isChecked true means open and set the maximum value; false close
*/
HardwareCtrl.ctrlLedRed(isChecked);

```



* Green fill light operation;


```
/*
isChecked true means open and set the maximum value; false close
*/
HardwareCtrl.ctrlLedGreen(isChecked);

```



### 3. Signal control

* "Rs485/232" 
>485 serial port: /dev/ttyS4; 
>232 serial port: /dev/ttyS3;
>you can view the set value changes through cat /dev/ttyS4 or cat /dev/ttyS3;

```
/*

Get serial number serialPort 
485 serial port: /dev/ttyS4; 232 serial port: /dev/ttyS3
*/
SerialPort serialPort = HardwareCtrl.openSerialPortSignal(new File("/dev/ttyS3"), 19200, new SerialPort.Callback() {

//rs485/232After sending the signal, the return value received
@Override
public void onDataReceived(byte[] bytes, int i) {
    String result = StringUtils.bytesToHexString(bytes, size);
    Log.i("firefly", "result = "+result);
}
});

/*
Send ‘48562311’ signal
*/
HardwareCtrl.sendSerialPortHexMsg(serialPort, "48562311")

/*
Close the serial port (it is recommended to close the serial port when the page exits)
*/
HardwareCtrl.closeSerialPortSignal(serialPort);

```



* Wiegand signal operation;


```
/*
Wiegand Input
*/
HardwareCtrl.openRecvMiegandSignal("/dev/wiegand");

/*
Add Wiegand Input listening callback
*/
HardwareCtrl.recvWiegandSignal(new RecvWiegandCallBack() {
@Override
public void recvWiegandMsg(int i) {
    Log.i("firefly", "result = "+i);
}
});

/*
When the page exits, close Wiegand
*/
HardwareCtrl.closeRecvMiegandSignal();

/*
Wiegand 26 Output
Check the set value changes through cat /sys/devices/platform/wiegand-gpio/wiegand26.
*/
HardwareCtrl.sendWiegandSignal("123456789");

/*
Wiegand 34 Output
Check the set value changes through cat /sys/devices/platform/wiegand-gpio/wiegand34.
*/
HardwareCtrl.sendWiegand34Signal("123456789");

```



* Level signal/relay signal;


```
/*
D0 level signal isChecked true means open, false close
*/
LevelSignalUtil.sendSignalD0(isChecked);

/*
D1 level signal isChecked true means open, false close
*/
LevelSignalUtil.sendSignalD1(isChecked);

/*
Relay signal isChecked true means open, false closed
*/
RelayUtil.sendRelaySignal(isChecked);

```



### 4. QRCode

* QRCode Operation


```
/*
Check whether the device supports the QR code function;
Boolean: true means support, false means not support
*/
QrCodeUtil.getInstance().isQrCodeSupport();

/*
Turn on QR code scanning
*/
QrCodeUtil.getInstance().init();

/*
Whether to turn on the QR code scanning fill light
QrCodeUtil.LED_STATE_AUTO // Automatic
QrCodeUtil.LED_STATE_ON // ON
QrCodeUtil.LED_STATE_OFF // OFF
*/
QrCodeUtil.getInstance().setLedState(state);

/*
Add QR code monitoring callback
*/
QrCodeUtil.getInstance().setQRCodeCallback(new QrCodeUtil.QRCodeCallback() {
// Callback method when QR code is connected
@Override
public void onConnect() {
Log.i("firefly","QRCode onConnect:");
}

// Content callback method for QR code recognition
@Override
public void onData(final String s) {
Log.i("firefly","QRCode onData:"+s);
}
});

/*
When the page exits, turn off QR code scanning
*/
QrCodeUtil.getInstance().release();

```



### 5. body temperature

* Body temperature operation;


```
/*
Check whether the device supports body temperature;
Boolean: true means support, false means not support
*/
TempatureUtil.getInstance().isSupport();

/*
Turn on body temperature detection
*/
TempatureUtil.getInstance().openDevice();

/*
Add temperature detection monitoring callback
*/
TempatureUtil.getInstance().setTempatureCallback(new TempatureUtil.TempatureCallback() {
// Callback method when body temperature detection is connected
@Override
public void onConnect() {
Log.i("firefly","TempatureCallback onConnect:");
}

// Content callback method for body temperature detection
@Override
public void update(float ambientTempature, float objectTempature) {
Log.i("firefly","TempatureCallback update:ambientTempature="+ambientTempature + "objectTempature="+objectTempature);
}
});

/*
When the page exits, turn off the body temperature function
*/
TempatureUtil.getInstance().closeDevice();

```



### 6. Radar

* Radar operation;


```
/*
Handling radar events, KeyEvent event
In the Android Activity page, public boolean dispatchKeyEvent(KeyEvent event), handle KeyEvent event
*/
RadarUtil.handleEvent(event);

/*
Dispatch Key Event on the Android Activity page
*/
@Override
public boolean dispatchKeyEvent(KeyEvent event) {
int ret = RadarUtil.handleEvent(event);
if (ret == RadarUtil.EVENT_HANDLE_RADAR_IN) {// Object entry
    Log.i("firefly","EVENT_HANDLE_RADAR_IN");
    return true;
} else if (ret == RadarUtil.EVENT_HANDLE_RADAR_OUT) { // Object leaving
    Log.i("firefly","EVENT_HANDLE_RADAR_OUT");
    return true;
} else if (ret == RadarUtil.EVENT_HANDLE_NOTHING_HANDLED) { // No object
    Log.i("firefly","EVENT_HANDLE_NOTHING_HANDLED");
    return true;
}

return super.dispatchKeyEvent(event);
}



// Monitor radar signals
// The radar signal is triggered in the form of KeyEvent and processed using handleEvent (KeyEvent event)
public static final int RadarUtil.EVENT_HANDLE_NOTHING_UNHANDLE = -1; // Non-radar events are not processed
public static final int RadarUtil.EVENT_HANDLE_NOTHING_HANDLED = 0; // Non-radar events processed

// Radar key-value pair
public static final int RadarUtil.KEYCODE_RADAR_IN = 305; // Something enters
public static final int RadarUtil.KEYCODE_RADAR_OUT = 306; // There is an object leaving

// Radar event
public static final int RadarUtil.EVENT_HANDLE_RADAR_IN = 1; // Something enters
public static final int RadarUtil.EVENT_HANDLE_RADAR_OUT = 2; // There is an object leaving
```


## Android Quick start


<h4>1. Add dependent libraries</h4>
```
1.Copy the aar package in the faceEngineYtlf/libs/ directory of the Demo project to the libs folder corresponding to the AS project；
2.Copy the ytlf_v2 folder in the faceEngineYtlf/src/main/assets directory of the Demo project to the src/main/assets folder corresponding to the AS project;

```
<h4>2. Add compile information for aar</h4>
```
implementation(name: 'arctern-release', ext: 'aar')
implementation(name: 'iface-release', ext: 'aar')
implementation(name: 'faceEngineYtlfExternal', ext: 'aar')

Add the following code to build.gradle's dependencies

For details, please refer to the technical case Demo project configuration or doc/FireflyApi_instructions.png ；
```
<h4>3. Setting permissions</h4>
```
Add the following permissions in the AndroidManifest.xml file

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-feature android:name="android.hardware.camera" android:required="true"/>
<uses-permission android:name="android.permission.CAMERA"/>
```
<h4>4. Defining FaceAPP variables in your program</h4>
> When using YTLF Face Manager, you need to indicate the root path of the local SD card file storage directory, for example: "/sdcard/firefly/"；
> When the SDK is started for the first time, it will check whether there are files such as models and license public keys in the local SD card. If not, then by default, the necessary files will be copied from the App assets directory to the rootPath directory；

```
// Specify the local SD card file storage directory
YTLFFaceManager.getInstance().initPath(String rootPath);
```
<h4>5.Calling Face API interface through YTLFFaceManager</h4>



## Face recognition

V2.0 adopts the asynchronous processing method of the thread pool to improve the utilization of the system and reduce the response time of services such as face detection and face tracking.

This SDK development guide will guide you how to install and configure the development environment, and how to perform secondary development and system integration by calling the interface (API) provided by the SDK.
Users can call the API provided by the SDK according to requirements to achieve the purpose of using services such as face detection / tracking, living body recognition, face recognition, and other services.


### 1. SDK Access

1.1 SDK Initialize

> When using YTLF Face Manager, you need to indicate the root path of the local SD card file storage directory, for example: "/sdcard/firefly/"；
> When the SDK is started for the first time, it will check whether there are files such as models and license public keys in the local SD card. If not, then by default, the necessary files will be copied from the App assets directory to the rootPath directory；

```
// Specify the local SD card file storage directory
YTLFFaceManager.getInstance().initPath(String rootPath);

```

1.2 Activating License

> During initialization, it will check whether there is a local key, if not, it will be activated online, and the activation status will be called back after activation. It is divided into synchronous and asynchronous activation methods. ；
>

```
// Asynchronous mode
YTLFFaceManager.getInstance().initLicenseByAsync(String apiKey, new InitLicenseListener(){
    @Override
    public void onLicenseSuccess() {
                toast("success");
    }

    @Override
    public void onLicenseFail(final int code,final String msg) {
                toast("failure");
    }
});


// Synchronization mode true means successful activation
boolean result = YTLFFaceManager.getInstance().initLicense(String apiKey);

```

1.3 Get unique device code

> Get the signature of the current device, the unique encoding of the device；
>

```
/*
String :Returns the current device's signature, which is the device's current unique code
*/
String signature = YTLFFaceManager.getInstance().getSignature();

```



### 2. SDK StartUp

> When starting the SDK, it will detect the operating environment and initialize the SDK. There are synchronous and asynchronous startup methods.；
>


2.1 Start the SDK synchronously
```
/*
Determine the config_path through FACE_PATH, which is the directory where the Config.json file is stored
eg:  sdcard/firefly/ytlf_v2/config.json"

Int: 0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().startFaceSDK();

/*
config_json Specify the contents of config.json

Int:0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().startFaceSDKByConfigJson(String config_json);

```

2.2 Start SDK asynchronously
> To prevent too much time when initializing and starting the SDK, causing the main thread to be blocked, you can use the asynchronous method to start the SDK；
>

```
/*
config_json Specify the contents of config.json
runSDKCallback Callback method after asynchronous start
*/
YTLFFaceManager.getInstance().startFaceSDKByAsynchronous(String config_json, new RunSDKCallback(){
@Override
public void onRunSDKListener(int i) {   //Int:0 Means success, 1 means failure

}
});
```



### 3. Face real-time data submission

> Perform face detection and tracking based on the data passed into the detector, and return the results of face detection and face tracking in real time.Note: The face direction of the data input to the face detector should be positive, that is, the face angle Should be 0 degrees and no other angle；
> Currently, RGB and IR video streams are used for face detection tracking and related detection of living bodies.When it is not necessary to turn on the IR camera or do not need living body detection, IR data parameters can pass RGB parameters, but not NULL；

```
//Get video stream from camera callback function, input RGB video stream and IR video stream in real time
YTLFFaceManager.getInstance().doDelivery(ArcternImage img_rgb,  ArcternImage img_ir)

//Set the RGB and IR monitor callback function
YTLFFaceManager.getInstance().setOnDetectCallBack(new DetectCallBack() {
//  RGB:
@Override
public void onDetectListener(ArcternImage arcternImage, ArcternRect[] arcternRects, float[] confidences) {

}

//  IR:
@Override
public void onLivingDetectListener(ArcternImage arcternImage, ArcternRect[] arcternRects, float[] confidences) {

}
});

/*Parameter Description：
arcternImage RGB/IR Face image data detected
arcternRects  RGB/IR Collection of detected face frames
confidences RGB/IR Detect the confidence of each face
*/
```

### 4. Face real-time tracking callback

If there is data returned from the callback interface, then the real-time data is being detected and tracked;

```
//Set real-time face detection tracking callback
YTLFFaceManager.getInstance().setOnTrackCallBack(new TrackCallBack() {

/*Face real-time detection and tracking
arcternImage Face image data detected
trackIds  Face tracking in images ID
arcternRects Detected face position
*/
@Override
public void onTrackListener(ArcternImage arcternImage, long[] trackIds, ArcternRect[] arcternRects) {

}
});
```
### 5.  Real-time face attribute detection

When performing real-time face detection tracking, add a face attribute detection callback method, which can receive face attributes in real time, including live detection value, quality, face angle, mask, and image color.

```
//Set real-time face attribute detection callback
YTLFFaceManager.getInstance().setOnAttributeCallBack(new AttributeCallBack() {

/*The face belongs to the monitoring callback:
arcternImage Face image data detected
arcternRects Detected face position
trackIds  Face tracking in images ID
arcternAttribute All attributes of all faces in the image
*/
@Override
public void onAttributeListener(ArcternImage arcternImage, long[] trackIds, ArcternRect[] arcternRects, ArcternAttribute[][] arcternAttributes, int[] landmarks) {
StringBuilder s = new StringBuilder();
for (int i = 0; i < arcternRects.length; i++) {
for (int j = 0; j < arcternAttributes[i].length; j++) {
    ArcternAttribute attr = arcternAttributes[i][j];
    switch (j) {
        case ArcternAttribute.ArcternFaceAttrTypeEnum.POSE_PITCH:
            s.append("Face angle:\n With the x axis as the center, the angle of the face up and down:").append(attr.confidence);
            break;
        case ArcternAttribute.ArcternFaceAttrTypeEnum.POSE_YAW:
            s.append("\n Rotate the face left and right around the y-axis:").append(attr.confidence);
            break;
        case ArcternAttribute.ArcternFaceAttrTypeEnum.POSE_ROLL:
            s.append("\n Taking the center point as the center, the x-y plane rotation angle:").append(attr.confidence);
            break;
            
        case ArcternAttribute.ArcternFaceAttrTypeEnum.QUALITY:
            s.append("\nFace quality:").append(attr.confidence);
            break;
            
        case ArcternAttribute.ArcternFaceAttrTypeEnum.LIVENESS_IR:
            if (attr.label != -1) {
                if (attr.confidence >= 0.5) {
                    s.append("\nBiopsy: live body ").append(attr.confidence);
                } else {
                    s.append("\nBiopsy: non-living ").append(attr.confidence);
                }
            }
            break;
            
        case ArcternAttribute.ArcternFaceAttrTypeEnum.IMAGE_COLOR:
            if (attr.label == ArcternAttribute.LabelFaceImgColor.COLOR) {
                s.append("\nColor picture ").append(attr.confidence);
            } else {
                s.append("\nBlack and white ").append(attr.confidence);
            }
            break;
            
        case ArcternAttribute.ArcternFaceAttrTypeEnum.FACE_MASK:
            if (attr.label == ArcternAttribute.LabelFaceMask.MASK) {
                s.append("\nMask ").append(attr.confidence);
            } else {
                s.append("\nWithout mask ").append(attr.confidence);
            }
            break;
    }
}
}
}
});
```

### 6. Face real-time search

Perform a real-time search based on the data passed into the detector, and search the database for an ID with a similarity greater than the highest set similarity. The ID can be used to obtain the recognized face and its related information;

```
//Set real-time face search callback
YTLFFaceManager.getInstance().setOnSearchCallBack(new SearchCallBack() {


/*Search callback:
arcternImage Face image data detected
trackIds  Face tracking in images ID
arcternRects Detected face position
searchId_list A collection of ids that identify faces in images
*/
@Override
public void onSearchListener(ArcternImage arcternImage, long[] trackId_list, ArcternRect[] arcternRects, long[] searchId_list, int[] landmarks,float[] socre) {
if (searchId_list.length > 0 && searchId_list[0] != -1) {
Person person = DBHelper.get(searchId_list[0]);
if (person != null) {
//The recognized face and related information can be obtained by ID;
}
} else {
// Face does not exist;
}
}
```


### 7. Get the landmarks coordinates of the eyes, mouth, nose, etc. of the face

Extract facial feature values ​​based on the Arctern Image of the face, which can be used to obtain the landmarks coordinates of the face, such as eyes, mouth, nose, etc.;

```
/*
Specify Arctern Image to extract facial feature values, including landmarks
*/
ArcternAttrResult ArcternAttrResult = YTLFFaceManager.getInstance().doFeature(ArcternImage arcternImage))

```


### 8. Extract face feature values ​​from specified image files

Extract face feature values ​​based on detected faces, which can be used for face comparison;

```
/*Specify image files to extract face feature values:
imagePath The map's address
etractCallBack  Face feature extraction callback
*/
YTLFFaceManager.getInstance().doFeature(imagePath, new ExtractCallBack() {

/*
acternImage Extract feature image
bytes  Set of face feature values ​​for multiple faces
arcternRects  Face detection result set
*/
@Override
public void onExtractFeatureListener(ArcternImage arcternImage, byte[][] bytes, ArcternRect[] arcternRects) {

if (features.length > 0) {
debugLog("bitmapFeature.length： " + features[0].length);
} else {
Tools.debugLog("Characteristic value is empty !");
}
});
```


### 9. Image Bitmap for feature value extraction

According to the face image Bitmap to extract the face feature value, it can be used for face comparison;

```
/*Specify face image Bitmap to extract face feature values:
bitmap Picture Bitmap data
etractCallBack  Face feature extraction callback
*/
YTLFFaceManager.getInstance().doFeature(Bitmap bitmap, new ExtractCallBack() {

/*
acternImage Extract feature image
bytes  Set of face feature values ​​for multiple faces
arcternRects  Face detection result set
*/
@Override
public void onExtractFeatureListener(ArcternImage arcternImage, byte[][] bytes, ArcternRect[] arcternRects) {

if (features.length > 0) {
debugLog("bitmapFeature.length： " + features[0].length);
} else {
Tools.debugLog("Characteristic value is empty !");
}
});

```


### 10. Face feature comparison

Compare the two feature values ​​to get the similarity between the two face feature values;

```
/*
feature1 First face feature value
Feature2  Second face feature value

return float :Return the similarity of face comparison
*/
float result = YTLFFaceManager.getInstance().doFeature(byte[] feature1,  byte[] feature2)

```



### 11. Face database management


11.1 Face storage

Add the extracted face feature values ​​and related IDs to the SDK face database;

```
/*
id Face ID
feature  Eigenvalues ​​of faces

0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().dataBaseAdd(long id, byte[] feature)

```

11.2 Face library delete

According to the specified ID, delete the face information of the SDK face database;

```
/*
id Face ID

0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().dataBaseDelete(long id)
```

11.3 Face library update

According to the specified ID, update the face information of the SDK face database;

```
/*
id Face ID
feature  Eigenvalues ​​of faces

0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().dataBaseUpdate(long id, byte[] feature)
```

11.4 Add feature values ​​to faces in batches

According to the specified multiple IDs, batch update multiple face information corresponding to the SDK face database;

```
/*
id Face ID Array
feature  Array of multiple face feature values

0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().dataBaseAdd(long[] id, byte[][] feature)

```

11.5 Face data removal

Delete all face information of SDK face database;

```
/*
0 Means success, 1 means failure
*/
int result = YTLFFaceManager.getInstance().dataBaseClear()
```





