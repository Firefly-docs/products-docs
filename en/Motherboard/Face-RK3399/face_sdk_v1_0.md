# Face recognition API  V1.0


## Hardware Interface API


### 1. Led control
Example code for controlling Led:
```
//red light
HardwareCtrl.ctrlLedSwitch( HardwareCtrl.LED_RED, true);
or
HardwareCtrl.ctrlLedRed(true);

//green light
HardwareCtrl.ctrlLedSwitch( HardwareCtrl.LED_GREEN, true);
or
HardwareCtrl.ctrlLedGreen(true);

//white light
HardwareCtrl.ctrlLedSwitch( HardwareCtrl.LED_WHITE, true);
or
HardwareCtrl.ctrlLedWhite(true);

//Only turn on the white light and turn off the other Led lights.
HardwareCtrl.ctrlLedOnlyOpenWhite();//green light only：ctrlLedOnlyOpenGreen(), red light only：ctrlLedOnlyOpenRed();

//Hardware version 2.3 and above supports adjusting the brightness of white led light, while green light and red light do not support adjusting the brightness.
HardwareCtrl.ctrlWhiteLightness(0);//Control the brightness of white led light: value range: 0-8
```

<br>

### 2. Control screen brightness

> public static void setBrightness(int value)
>
> Function&emsp;&emsp;&emsp;Adjust screen brightness
>
> Parameter&emsp;&emsp;&emsp;value : range: 0 to 255

<br>

Example ：
```
HardwareCtrl.setBrightness(255);

```
### 3. Backlight control switch
> public static void ctrlBlPower(boolean open)
>
> Function&emsp;&emsp;&emsp;Backlight control switch
>
> Parameter&emsp;&emsp;&emsp;open :true is on, false is off

<br>

Example ：
```
HardwareCtrl.ctrlBlPower(true);

```
### 4. Screen touch switch
> public static void ctrlTp(boolean open)
>
> Function&emsp;&emsp;&emsp;Screen touch switch
>
> Parameter&emsp;&emsp;&emsp;open : true is on, false is off

<br>

Example ：
```
HardwareCtrl.ctrlTp(true);

```
### 5.  RS485 / RS232 signal 
Open RS485/RS232 serial port
> public static SerialPort openSerialPortSignal(File device, int baudrate, SerialPort.Callback callback)
>
> Function&emsp;&emsp;&emsp;Open RS485/RS232 serial port
>
> Parameter&emsp;&emsp;&emsp;device : Serial port file<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;baudrate : Baud rate<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;callback : callback

<br>
Send signal

> public static void sendSerialPortHexMsg(SerialPort mSerialPort, String msg)
>
> Function&emsp;&emsp;&emsp;Send signal
>
> Parameter&emsp;&emsp;&emsp;mSerialPort :  Serial port object<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;msg :Signal (hex string, such as "1E60010000002F")<br>

<br>
Close the RS485/RS232 serial port

> public statis void closeSerialPortSignal(SerialPort mSerialPort)
>
> Function&emsp;&emsp;&emsp;Close the RS485/RS232 serial port
>
> Parameter&emsp;&emsp;&emsp;void

<br>

Example ：
```
// Enter related content
/ **
such as:
1.A opens the gate to the credit card, the upper computer needs to send hexadecimal data:
Send: 0x1E 0x60 0x01 0x00 0x00 0x00 0x2F
The return code of the gate is divided into the following types:
a), the person has passed the gate
Returns: 0x1E 0x61 0x01 0x00 0x00 0x00 0x2F
b) The card does not pass the gate after a timeout after the card is swiped, the gate automatically closes the door and cancels the passage
Returns: 0x1E 0x44 0x01 0x00 0x00 0x00 0x2F
c) After the card is swiped, someone passes the gate in the reverse direction, and the gate is automatically closed to cancel the passage.
Returns: 0x1E 0x44 0x01 0x00 0x00 0x00 0x2F
*/
//Open RS485/RS232 serial port
SerialPort mSerialPort = HardwareCtrl.openSerialPortSignal(new File("dev/ttyS4"), 9600, new SerialPort.Callback() {
     @Override
     public void onDataReceived(byte[] buffer, int size) {
         String result = StringUtils.bytesToHexString(buffer, size);
         Log.e("lkdong","result = "+result);
     }
});

//Send signal
HardwareCtrl.sendSerialPortHexMsg(mSerialPort, "1E60010000002F");
//Close the RS485/RS232 serial port
HardwareCtrl.closeSerialPortSignal(mSerialPort);

```
<font color='red'> 
Note:<br>
1. The old interface of 485 signal control can be used continuously, such as：HardwareCtrl.openRs485Signal(File device, int baudrate, SerialPort.Callback callback)、
HardwareCtrl.sendRs485Signal(SerialPort mSerialPort, String msg) and HardwareCtrl.closeRs485Signal(SerialPort mSerialPort)
<br>
<br>
2. 485 serial port：/dev/ttyS4  ;  232 serial port：/dev/ttyS3<br>
<br>
<br>
</font>

### 6. Wigan 26/34 signal 
#### Wigan 26 
> public static void sendWiegandSignal(String msg)
> 
> Function&emsp;&emsp;&emsp;Wiegand Signal Control
> 
> Parameter&emsp;&emsp;&emsp;msg : 
<br>

#### Wigan 34
> public static void sendWiegand34Signal(String msg)
> 
> Function&emsp;&emsp;&emsp;Wiegand Signal Control
> 
> Parameter&emsp;&emsp;&emsp;msg : 
<br>

Example ：
```
//Enter related content, such as card number, etc.
HardwareCtrl.sendWiegandSignal("1233456789");
```
<font color='red'> 
Note:<br>
1. Wiegand 26 and Wiegand 34 share the same line, so only one of them can be used at a time
<br>
</font>
<br>

### 7.Wigan input
> public static void recvWiegandSignal(RecvWiegandCallBack callBack)
> 
> Function&emsp;&emsp;&emsp;Wiegand signal input
> 
> Parameter&emsp;&emsp;&emsp;callBack : Wiegand input signal return value interface
<br>

Example ：
```
HardwareCtrl.openRecvMiegandSignal("/dev/wiegand");//Open serial port
HardwareCtrl.recvWiegandSignal(new RecvWiegandCallBack() {
    @Override
    public void recvWiegandMsg(int i) {
       Log.e("lkdong","recvWiegandMsg = "+i);
    }
});
HardwareCtrl.closeRecvMiegandSignal();

```
### 8. RelaySignal
> public static void sendRelaySignal(boolean up)
> 
> Function&emsp;&emsp;&emsp;Relay control
> 
> Parameter&emsp;&emsp;&emsp;up : The signal is pulled up to true and pulled down to false.
<br>

Example ：
```
HardwareCtrl.sendRelaySignal(!HardwareCtrl.getRelayValue());
```

### 9. Radar
The working principle of the radar function: within the radar radiation range, with or without object activity, the key value is reported to the Android application layer, and developers can develop according to their own needs.
<br>

Example ：
```
public class MainActivity extends AppCompatActivity {
    //Radar key value pair
    public static final int KEYCODE_RADAR_IN = 305;//There is object activity
    public static final int KEYCODE_RADAR_OUT = 306;//No object activity

    @Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        Log.v("lkdong", "dispatchKeyEvent keycode=" + event.getKeyCode());
        if (event.getKeyCode() == KEYCODE_RADAR_IN) {//Radar reports object activity
            if (event.getAction() == KeyEvent.ACTION_UP) {
		//TODO
            }
        } else if (event.getKeyCode() == KEYCODE_RADAR_OUT) {//Radar reports no object activity
            if (event.getAction() == KeyEvent.ACTION_UP) {
		//TODO
            }
        } 
        return super.dispatchKeyEvent(event);
    }

}
```
### 10.NFC
The principle of the nfc function: When the card is swiped, the system will report the key value to the Android application layer, and save the card information on the / dev / dl1825 node. When developing an NFC card reader, you need to read the "/dev/dl1825" node value after receiving the system report key value.
<br>

Example ：
```
public class MainActivity extends AppCompatActivity {
    //NFC key pair
    public static final int KEYCODE_NFC=307;

     @Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        Log.v("lkdong", "dispatchKeyEvent keycode=" + event.getKeyCode());
        if (event.getKeyCode() == KEYCODE_NFC) {
            if (event.getAction() == KeyEvent.ACTION_UP) {
                //NFC card reading, please refer to the bottom gpiodemo for detailed implementation process.
		String mNfcCode = NfcUtil.getCode();

            }
        }
        return super.dispatchKeyEvent(event);
    }

}
```
### 11.Qrcode
The two-dimensional code reading is mainly performed through the serial port. The specific operation process is as follows:
<br>

Example ：
```
public class QrCodeUtil {
    public static final String TAG = QrCodeUtil.class.getSimpleName();
    //The current command is based on module e3000h
    public static final byte[] ENABLE_QR_CONFIG = new byte[]{0x07, (byte) 0xC6, 0x04, 0x08, 0x00, (byte) 0xEC, 0x01, (byte) 0xFE, 0x3A};
    public static final byte[] DISABLE_QR_CONFIG = new byte[]{0x07, (byte) 0xC6, 0x04, 0x08, 0x00, (byte) 0xEC, 0x00, (byte) 0xFE, 0x3B};
    public static final byte[] LED_ON = new byte[]{0x08, (byte) 0xC6, 0x04, 0x08, 0x00, (byte) 0xF2, 0x02, 0x01, (byte) 0xFE, 0x31};
    public static final byte[] LED_OFF = new byte[]{0x08, (byte) 0xC6, 0x04, 0x08, 0x00, (byte) 0xF2, 0x02, 0x02, (byte) 0xFE, 0x30};
    public static final byte[] LED_AUTO = new byte[]{0x08, (byte) 0xC6, 0x04, 0x08, 0x00, (byte) 0xF2, 0x02, 0x00, (byte) 0xFE, 0x32};
    public static final byte[] COMMADN_RESULT = new byte[]{0x04, (byte) 0xd0, 0x00, 0x00, (byte) 0xff, 0x2c};
    private SerialPort mQrCodeSerialPort = null;
    private static QrCodeUtil sQrCodeUtil = null;
    private static boolean sQrCodeSupport = true;

    /**
     * Enable QR code recognition and open serial port
     */
    public static void open() {
        if (isQrCodeSupport() && sQrCodeUtil != null) {
            sQrCodeUtil.openQrCodeSerialPort(new SerialPort.Callback() {
                @Override
                public void onDataReceived(byte[] bytes, int i) {
                    if (QrCodeUtil.isCommandResult(bytes, i)) {
                    } else {
                        String s = new String(bytes, 0, i);
                        Log.v(TAG, "QR string " + s);
                        if (sQrCodeUtil.mQRCodeCallback != null) {
                            sQrCodeUtil.mQRCodeCallback.onQrCodeData(s);
                        }
                    }

                }
            });
        }
    }

    /**
     * Close the QR code identification serial port
     */
    public static void close() {
        if (isQrCodeSupport() && sQrCodeUtil != null) {
            sQrCodeUtil.closeQrCodeSerialPort();
        }
    }

    /**
     * Close the serial port and release
     */
    public static void destory() {
        if (isQrCodeSupport() && sQrCodeUtil != null) {
            sQrCodeUtil.closeQrCodeSerialPort();
            sQrCodeUtil = null;
        }
    }


    /**
     * Open serial port
     *
     * @param qrCodeDataCallBack
     */
    private void openQrCodeSerialPort(SerialPort.Callback qrCodeDataCallBack) {
        if (mQrCodeSerialPort == null) {
            Log.v(TAG, "openQrCodeSerialPort===");
            mQrCodeSerialPort = HardwareCtrl.openSerialPortSignal(new File("/dev/ttyS0"), 9600, qrCodeDataCallBack);
            disableQRCodeConfig();
        }
    }

    /**
     * Close serial port
     */
    private void closeQrCodeSerialPort() {
        Log.v(TAG, "closeQrCodeSerialPort()");
        if (mQrCodeSerialPort != null) {
            HardwareCtrl.closeQrCodeSerialPort(mQrCodeSerialPort);
            mQrCodeSerialPort = null;
        }
    }

    /**
     * Turn on identification configuration QR code function
     */
    private void enableQRCodeConfig() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(ENABLE_QR_CONFIG);
        }
    }

    /**
     * Turn off identification configuration QR code function
     */
    private void disableQRCodeConfig() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(DISABLE_QR_CONFIG);
        }
    }

    /**
     * Qrcode light automatically turns on during scanning
     */
    private void setLEDAuto() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(LED_AUTO);
        }
    }

    /**
     * Qrcode light is always off
     */
    private void setLEDOFF() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(LED_OFF);
        }
    }

    /**
     * Qrcode light on all the time
     */
    private void setLEDON() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(LED_ON);
        }
    }

    /**
     * Configure QR code scan qrcode light status.
     * At present, it is only opened and closed automatically during scanning
     *
     * @param isAuto
     */
    public static void setLedState(boolean isAuto) {
        if (isQrCodeSupport() && sQrCodeUtil != null) {
            if (isAuto)
                sQrCodeUtil.setLEDAuto();
            else
                sQrCodeUtil.setLEDOFF();
        }
    }

    public interface QRCodeCallback {
        public void onQrCodeData(String s);
    }

    private QRCodeCallback mQRCodeCallback;

    /**
     * Support successful monitoring of QR code scanning
     *
     * @param callback
     */
    public void setQRCodeCallback(QRCodeCallback callback) {
        mQRCodeCallback = callback;
    }
}
```

### 12. Normal GPIO control
D0 Signal
> public static void sendSignalD0(boolean up)
>
> Function&emsp;&emsp;&emsp; GPIO D0 signal control
>
> Parameter&emsp;&emsp;&emsp;up : false is pulled low, true is pulled high

<br>

Example ：
```
HardwareCtrl.sendSignalD0(true);

```
D1 Signal
> public static void sendSignalD1(boolean up)
>
> Function&emsp;&emsp;&emsp;GPIO D1 signal control
>
> Parameter&emsp;&emsp;&emsp;up :  false is pulled low, true is pulled high
<br>

Example ：
```
HardwareCtrl.sendSignalD1(true);

```
### 13. Shutdown
> public static void shutdown()
>
> Function&emsp;&emsp;&emsp;Shutdown
>
> Parameter&emsp;&emsp;&emsp;void

<br>

Example ：
```
HardwareCtrl.shutdown();

```
### 14. reboot
> public static void reboot()
>
> Function&emsp;&emsp;&emsp;reboot
>
> Parameter&emsp;&emsp;&emsp;void

<br>

Example ：
```
HardwareCtrl.reboot();

```
### 15. Watchdog
> public static void setWdt(int value)
>
> Function&emsp;&emsp;&emsp;System freezes or does not respond for a long time, reboot the device
>
> Parameter&emsp;&emsp;&emsp;value ： Valid values are integer values between 0 (inclusive) and 3 (inclusive).<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 means reboot after 0.46s of crash<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1 means reboot after 2.56s of crash<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2 means reboot after 10.24s of crash<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;3 means reboot after 40.96s of crash

<br>

Example ：
```
HardwareCtrl.ctrlWdt(1);

```
### 16. Get device unique ID
> public static String getFireflyCid()
>
> Function&emsp;&emsp;&emsp;device unique ID
>
> Parameter&emsp;&emsp;&emsp;void

<br>

Example ：
```
String cid = HardwareCtrl.getFireflyCid();
```

### 17. Other commands
> public static void execSuCmd(String command) 
>
> Function&emsp;&emsp;&emsp;Run command through shell
>
> Parameter&emsp;&emsp;&emsp;command：command

<br>

Example ：
```
//Such as syncing files, etc.
HardwareCtrl.execSuCmd("sync");

```
### 18. Other GPIO use
> public static int gpioParse(String gpioStr)
>
> Function&emsp;&emsp;&emsp;Convert gpio name to corresponding gpio encoding
>
> Parameter&emsp;&emsp;&emsp;gpioStr：gpio name, such as GPIO2_A2

<br>
Controlling GPIO

> public static void ctrlGpio(int gpio, String direction, int value)
>
> Function&emsp;&emsp;&emsp;Controlling GPIO
>
> Parameter&emsp;&emsp;&emsp;gpio：gpio encoding, such as 152<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;direction : <br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;value : Set GPIO value

<br>

Example ：
```
HardwareCtrl.ctrlGpio(HardwareCtrl.gpioParse("GPIO2_A2"), "out", 1);

```

### 19.TestGpioDemo download link

* [https://drive.google.com/open?id=14wzm7jRB9rdi-0MnwpOW6gWiPksjDlRC](https://drive.google.com/open?id=14wzm7jRB9rdi-0MnwpOW6gWiPksjDlRC)




## Android Quick start


<h4>1. Add dependent libraries</h4>
```
Copy facelibrary-release.aar and openCVLibrary331-release.aar to the app/libs directory
```
<h4>2. Add compile information for aar</h4>
```

Add the following code to build.gradle's dependencies:

compile(name: 'facelibrary-release', ext: 'aar')
compile(name: 'openCVLibrary331-release', ext: 'aar')
    
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
```
private FaceAPP face = FaceAPP.GetInstance(); //Get the instance of FaceApp
```
<h4>5.Calling Android API interface through face</h4>



## Face recognition SDK

  This SDK development guide will guide you how to install and configure the development environment, and how to perform secondary development and system integration by calling the interface (API) provided by the SDK.
  Users can call the API provided by the SDK according to requirements to achieve the purpose of using services such as face detection / tracking, living body recognition, face recognition, and other services.

### 1. Return values
> public static final int SUCCESS = 0; //execution succeed<br>
> public static final int ERROR_INVALID_PARAM = -1; //Illegal parameter<br>
> public static final int ERROR_TOO_MANY_REQUESTS = -2; //Too many requests<br>
> public static final int ERROR_NOT_EXIST = -3; //does not exist<br>
> public static final int ERROR_FAILURE = -4; // Execution failed

<br>

### 2. FaceInfo 
```
public class FaceInfo {
     public Rect mRect;//Face recognition frame
     public FaceAttribute mAttr;//Face attributes
     public FaceQuality mQuality;//Face recognition quality
     public Landmark mLandmark;//Used to store the coordinates of 5 key points, which are left eye, right eye, nose, left side of lips, right side of lips
}
```

### 3. FaceAttribute 
```
public class FaceAttribute {
     public int mGender;//Gender 0-> Male, 1->Female
     public int mEmotion;//expression 0->calm 1->glad
     public int mAge;//Age
}
```

### 4. FaceQuality 

```
public class FaceQuality {
     public float mScore;//Confidence in Face Quality
     public float mLeftRight;//Roll
     public float mUpDown;//Pitch
     public float mHorizontal;//Yaw
     public float mClarity;//Clarity
     public float mBright;//Bright
}
```

### 5. Constructors
> static FaceAPP GetInstance()
> 
> Function:<br>
> Returns the FaceApp object
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns:<br>
> &emsp;&emsp;FaceAPP

<br>

Example ：
```
private FaceAPP face = FaceAPP.GetInstance();

```
### 6. Recognize facial features
> int Recognize( Image image, float featureArray [][512], int size, List<FaceInfo> faceinfos, int[] res ) 
> 
> Function : <br>
> Identify the facial features in the submitted Image, then compare with the data in the featureArray to find the data with the highest similarity.Returns the index of the array.
> 
> Parameters :<br>
> &emsp;&emsp;image : Image object holding face image<br>
> 
> &emsp;&emsp;featureArray : An array of facial feature values. The length of the facial feature value array is 512.<br>
> 
> &emsp;&emsp;size : The length of featureArray<br>
> 
> &emsp;&emsp;faceinfos : FaceInfo checklist. FaceInfo is an object used to save the results of operations<br>
> 
> &emsp;&emsp;res: Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.If the face in image is not in featureArray,res[0] return ERROR_NOT_EXIST.If res [0]> = 0, the index of the featureArray is returned.
> 
> Returns&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
float[][] featurelist = new float[][]; //Array storing facial feature values
int size = featurelist.lenth;
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];
FaceAPP.Image image = FaceAPP.GetInstance().new Image();
image.matAddrframe = mRgbaFrame.getNativeObjAddr();
face.Recognize( image, featurelist, size, tmpPos, res );
```

### 7. Recognize facial features(According to facial feature values)
> int Recognize( float[] feature, float featureArray [][512], int size, float[] high, int[] res )
>    
> Function :<br>
> According to the input facial feature values, then compare with the data in the featureArray to find the data with the highest similarity.Returns the index of the array。
> 
> Parameters :<br>
> &emsp;&emsp;feature : Facial feature values
> 
> &emsp;&emsp;featureArray :  An array of facial feature values. The length of the facial feature value array is 512.
> 
> &emsp;&emsp;size : The length of featureArray
> 
> &emsp;&emsp;high : Similarity score<br>
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.If the face in image is not in featureArray, res[0] return ERROR_NOT_EXIST. If res [0]> = 0, the index of the featureArray is returned.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
float[][] featurelist = new float[][]; //Array storing facial feature values
int size = featurelist.lenth;
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];
float[] feature;
float[] high = float[1];
FaceAPP.Image image = FaceAPP.GetInstance().new Image();
image.matAddrframe = mRgbaFrame.getNativeObjAddr();
face.Recognize( feature, featurelist, size, high, tmpPos, res );
```

### 8. Detect faces
> int Detect( Image image, List<FaceInfo> faceinfos, int[] res )
>   
> Function :<br>
> Detect faces in submitted pictures
> 
> Parameters :<br>
> &emsp;&emsp;image : Image object containing a face image
> 
> &emsp;&emsp;faceinfos :  FaceInfo checklist. FaceInfo is an object used to save the results of operations
> 
> &emsp;&emsp;res :  Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.<br>
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];//Used to store location information
FaceAPP.Image image = FaceAPP.GetInstance().new Image(); //Init
image.matAddrframe = mRgbaFrame.getNativeObjAddr(); //Set the memory address of the picture to be identified
if( success = face.Detect( image, tmpPos, res ) ){
//to do
};
```
### 9. Compare feature data
> int Compare( float[] origin, float[] chose, float score )
>   
> Function : <br>
> Used to compare the similarity of two feature values
>  
> Parameters : <br>
> &emsp;&emsp;origin : Feature array to be compared
> 
> &emsp;&emsp;chose : Feature array for comparison
> 
> &emsp;&emsp;score : similarity between origin and chose
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
float score;
float[] origin = new float[512];
Float[] chose = new float[512];
face.Compare( origin, chose, score );
```
### 10. Face extraction with binocular and living body
> int GetFeature( Image image, Image grayImage, float[] feature, List<FaceInfo> faceinfos, int[] res )
>   
> Function :<br>
> Extracting facial features from data obtained by binocular cameras,and only one face can be extracted.
> 
> Parameters:<br>
> &emsp;&emsp;image : Image object containing a face image<br>
> 
> &emsp;&emsp;grayImage : Image object containing an infrared image<br>
> 
> &emsp;&emsp;feature : Stores the facial feature information of the detection results, and returns an empty array without a face
> 
> &emsp;&emsp;faceinfos : FaceInfo checklist. FaceInfo is an object used to save the results of operations<br>
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Successfully obtained face information<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : No face information was obtained

<br>

Example ：
```
float[] feature = new float[512];
int[] ret = new int[1];
byte[] tmpPos = new byte[1024]; //Used to store location information
ret = face.GetFeature( image, grayImage, feature, tmpPos, res);
if( ret == SUCCESS ){
//to do 
}
```
### 11. Liveness Detection
> int DetectLiveness(Image image, List<FaceInfo> faceinfos, int[] res)
> 
> Function:<br>
> Detect whether a face is alive.
> 
> Parameters:<br>
> &emsp;&emsp;image : Color face pictures, pictures for detection and recognition.
> 
> &emsp;&emsp;faceinfos : FaceInfo checklist. FaceInfo is an object used to save the results of operations
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Living<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : Non-living

<br>

Example ：
```
int[] ret=new int[1];
ret= face.DetectLiveness(image,grayImage ,tmpPos,res);
if(ret== SUCCESS){
//to do 
}
```
### 12. Binocular for Liveness detection
> int GetDetectLiveness(Image image, Image grayImage, List<FaceInfo> faceinfos, int[] res)
> 
> Function:<br>
> Detect whether a face is alive.
> 
> Parameters:<br>
> &emsp;&emsp;image : Color face pictures, pictures for detection and recognition.
> 
> &emsp;&emsp;grayImage :Image object containing an infrared image.
> 
> &emsp;&emsp;faceinfos : FaceInfo checklist. FaceInfo is an object used to save the results of operations
> 
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Living<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : Non-living

<br>

Example ：
```
int[] ret=new int[1];
ret= face.GetDetectLiveness(image,grayImage ,tmpPos,res);
if(ret== SUCCESS){
//to do 
}
```
### 13. Binocular Calibration
> int Calibration( Image image, Image grayImage, float[] scale, int[] Rect, int[] res );
>    
> Function:<br>
> The calibration of the dual camera recognition consisting of infrared light and ordinary light requires one person to stand at the optimal position (0.8-1 meters), and it takes about 20 checks to obtain the face correction parameters. And return the coordinates of the infrared camera display area relative to ordinary light, this area is an effective recognition and living body detection area.
> 
> Parameters :<br>
> &emsp;&emsp;image : Color face pictures, pictures for detection and recognition.
> 
> &emsp;&emsp;grayImage : Image object containing an infrared image
> 
> &emsp;&emsp;scale : Returns Face Frame Correction Parameters
> 
> &emsp;&emsp;rect : Returns the overlapping area of the infrared image and the color image (the corresponding area of the infrared image in the color image / recommended detection area)
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Calibration succeeded<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : Calibration failed

<br>

Example ：
```
int[] ret = new int[1];
float[] scale = new float[1];
int[] rect = new int[4];
ret = face.Calibration( image, grayimage, scale, rect, res );
if( ret == SUCCESS ){
//to do 
}
```
### 14. Device Activation 1
> int AuthorizedDevice( String uidStr, String password, Context activity )
>    
> Function: <br>
> Device activation
> 
> Parameters:<br>
> &emsp;&emsp;uidStr : OEMID + contract Id
> 
> &emsp;&emsp;password : password
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : Authentication succeeded

<br>

Example ：
```
String oem_id = "1000000000000001";//OEMID
String contract_id = "0001";// contract Id
String password = "0123456789abcdef0123456789abcdef"; //password
String uidStr = oem_id + contract_id;
int res = face.AuthorizedDevice( uidStr, password, LoginActivity.this ); 
```
### 15. Device Activation 2
> int fireflyInit(Context context, String uidStr, String password) 
>    
> Function :<br>
> Device activation;This interface is a temporary interface for firefly, which can permanently activate the device. This interface may be removed later.If the interface description is not included in subsequent development documents, it has been removed. The development documents are subject to the documents in the open source Demo.
> 
> Parameters :<br>
> &emsp;&emsp;uidStr : OEMID + contract Id
> 
> &emsp;&emsp;password : password
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : Authentication succeeded

<br>

Example ：
```
String oem_id = "1000000000000001";//OEMID
String contract_id = "0001";//contract Id
String password = "0123456789abcdef0123456789abcdef"; //password
String uidStr = oem_id + contract_id;
int res =face.fireflyInit(LoginActivity.this, uidStr, password);
```
### 16. Device Activation 3
> int AuthorizedDeviceUserPassword(String uidStr, String password, Context context, String userPassword)
>    
> Function :<br>
> Device activation，This interface is mainly used for the activation method with user password.
> 
> Parameters :<br>
> &emsp;&emsp;uidStr : OEMID + contract Id
>
> &emsp;&emsp;password : password
>
> &emsp;&emsp;userPassword:user password
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : Authentication succeeded

<br>

Example ：
```
String oem_id ="1000000000000001";//OEMID
String contract_id ="0001";//contract Id
String password = "0123456789abcdef0123456789abcdef";//password
String userPassword ="012345678912";//user password;
String uidStr = oem_id+contract_id;
int res =face. AuthorizedDeviceUserPassword(uidStr,password,
LoginActivity.this,userPassword);
```
### 17. Get authentication activation status
> int getAuthStatus()
>   
> Function:<br>
> Get authentication activation status
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : activated

<br>

Example ：
```
int res = face.getAuthStatus(); 
```
### 18. Extracting facial features
> int GetFeature( Image image, float[] feature, List<FaceInfo> faceinfos, int[] res )
>    
> Function:<br>
> Extract feature values of pictures in image,and get only the characteristics of one person in the picture。
> 
> Parameters:<br>
> &emsp;&emsp;image : Image object containing a face image
> 
> &emsp;&emsp;feature : Stores the facial feature information of the detection results, and returns an empty array without a face
> 
> &emsp;&emsp;faceinfos : FaceInfo checklist. FaceInfo is an object used to save the results of operations
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
>
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Feature extraction successful<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : No face information was obtained

<br>

Example ：
```
float[] feature = new float[512];
int[] ret = new int[1];
byte[] tmpPos = new byte[1024]; //Used to store location information
ret = face.GetFeature( image, feature, tmpPos, res );
if( ret == SUCCESS ){
//to do 
}
```
### 19. Extraction of face features (based on face coordinate information)
> int GetFeature( Image image, FaceInfo detectInfo, float[] feature, int[] res )
>   
> Function: <br>
> Extract feature values of pictures in image,and get only the characteristics of one person in the picture。
> 
> Parameters:<br>
> &emsp;&emsp;image : Image object containing a face image
> 
> &emsp;&emsp;detectInfo : Face information detected for facial feature extraction
> 
> &emsp;&emsp;feature : Stores the facial feature information of the detection results, and returns an empty array without a face
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Feature extraction successful<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE :  No face information was obtained

<br>

Example ：
```
float[] feature = new float[512];
int[] ret = new int[1];
float[] detectinfo = new float[]{ x0, y0, x1, y1, landmarkx0, landmarky0,
				 landmarkx1, landmarky1, landmarkx2, landmarky2,
				 landmarkx3, landmarky3, landmarkx4, landmarky4 }
byte[] tmpPos = new byte[1024]; //Used to store location information
ret = face.GetFeature( image, detectinfo, feature, res );
if( ret == SUCCESS ){
//to do 
}
```
### 20. Extracting facial features (based on picture files)
> float[] GetFeature(String path, List<FaceInfo> faceinfos)
>   
> Function:<br>
> Get the facial feature value data in the image according to the path of the incoming image file.
> 
> Parameters:<br>
> &emsp;&emsp;path :The absolute path of the face image file.
> 
> &emsp;&emsp;faceinfos : FaceInfo checklist. FaceInfo is an object used to save the results of operations
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Float[] : Face feature value array

<br>

Example ：
```
int ret = this.FaceGetFeatureFromAddr(addr, feature, mFaceInfos, policy);

if (ret == -1) {
     return null;
} else {
     faceInfos.addAll(Arrays.asList(this.mFaceInfos).subList(0,ret));
     return this.feature;
}
```
### 21. Extract key points of the face
> int GetLandmark ( Image image, float[] landmark, int[] res )
>    
> Function:<br>
> Get face key point coordinate information
> 
> Parameters:<br>
> &emsp;&emsp;Image : Face picture, a picture used to extract key point information of the face, only extract key point information of a person
> 
> &emsp;&emsp;Landmark : Used to store the coordinates of 5 key points, which are left eye, right eye, nose, left side of lips, right side of lips
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Successfully obtain coordinate information of face keypoints<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : No facial keypoint information was obtained

<br>

Example ：
```
float[] landmark = new float[10];
int[] ret = new int[1];
ret = face.GetLandmark( image, landmark, res);
if( ret == SUCCESS ){

}
```
### 22. Face quality
> FaceInfo getQuality(long matAddrframe)
>    
> Function:<br>
> Get the face information of the largest face in the image。
> 
> Parameters:<br>
> &emsp;&emsp;matAddrframe : Image address stored in Mat object<br>
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;FaceInfo : Used to store face information in pictures

<br>

Example ：
```
Faceinfo faceinfo=new Faceinfo();
faceinfo=face.getQuality(matAddrframe);
if(faceinfo!=null){
}
```
### 23. Parameters Setting
> bool SetParameter( const String[] name, float value[] )
>    
> Function:<br>
> Set the entered Parameter key and value
> 
> Parameters:<br>
> &emsp;&emsp;char[] name : Parameters key array
> 
> &emsp;&emsp;a : Internal Parameters, set according to the example, please do not modify.
> 
> &emsp;&emsp;b : Internal Parameters, set according to the example, please do not modify.
> 
> &emsp;&emsp;c : Internal Parameters, set according to the example, please do not modify.
> 
> &emsp;&emsp;d : Internal Parameters, set according to the example, please do not modify.
> 
> &emsp;&emsp;factor : Detect face magnification. Internal Parameters, set according to the example, please do not modify.
> 
> &emsp;&emsp;min_size : Minimum face frame size, recommended range:32-80
> 
> &emsp;&emsp;faceclarity : Photo sharpness threshold, recommended range:200-400
> 
> &emsp;&emsp;perfoptimize : Whether to optimize the effect , recommended range: 0 or 1
> 
> &emsp;&emsp;livenessdetect : Whether live detection , recommended range:0 or 1
> 
> &emsp;&emsp;gray2colorscale : Binocular live detection ratio , recommended range: 0.1-0.5
> 
> &emsp;&emsp;frame_num : Number of frames,recommended range:20-40
> 
> &emsp;&emsp;quality_thresh : Picture quality threshold,recommended range:0.7-0.8
> 
> &emsp;&emsp;mode : Operating mode :0-> Entrance 1-> Access control
> 
> &emsp;&emsp;facenum : Detects the maximum number of faces and supports detection of up to 3 faces and 1 face.recommended range:1-3
> 
> &emsp;&emsp;value[] : Parameters key array
> 
> Returns:<br>
> &emsp;&emsp;Whether the parameters are set successfully

<br>

Example ：
```
String[] name = { "a", "b","c", "d", "factor", "min_size", "clarity", "perfoptimize", 
		  "livenessdetect", "gray2colorscale", "frame_num", "qualit_thresh",
		  "mode", "facenum" };
double[] value = {0.9, 0.9, 0.9, 0.715, 0.6, 64, 400, 1, 0, 0.5, 20, 0.8, 1, 1 };
face.SetParameter( name, value );
```
### 24. Parameters Setting(Mode for modifying parameter length as needed)
> bool SetParameters( String[] name, float value[] )
>    
> Function:<br>
> Set the entered Parameter key and value
> 
> Parameters:<br>
> &emsp;&emsp;char[] name : Parameters key array(length>=1),See details in 23.
> 
> &emsp;&emsp;value[] : Parameters value array(length>=1)
>
> Returns:<br>
> &emsp;&emsp;Whether the parameters are set successfully

<br>

Example ：
```
String[] name = { "perfoptimize", "livenessdetect", "frame_num", "quality_thresh", 
		  "mode", "facenum" };
double[] value = { 1, 0, 20, 0.8, 1, 1 };
face.SetParameter( name, value );
```
### 25. Get version information
> public String GetVersion()
>    
> Function:<br>
> Get information about the current SDK version
> 
> Parameters&emsp;&emsp;&emsp;void
> 
> Returns:<br>
> &emsp;&emsp;Return current version information

<br>

Example ：
```
Face.GetVersion();
```
### 26. Get Facelib Version
> public String GetFacelibVersion()
>  
> Function:<br>
> Get Facelib Version
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns:<br>
> &emsp;&emsp;Return Facelib version information

<br>

Example ：
```
Face.GetFacelibVersion();
```
### 27. Open the face database
> int OpenDB()
>  
> Function:<br>
> Using face database, internal face database can achieve 1-to-N efficient and fast query
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
if(face.OpenDB() == SUCCESS){
// TODO
}
```
### 28. Registered face
> int AddDB( float[] feature, string name )
>   
> Function:<br>
> Registered face to database
> 
> Parameters:<br>
> &emsp;&emsp;feature : Facial features
> 
> &emsp;&emsp;name : Registered name
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
String name= "test";
int[] res=new int[];
if( Face.GetFeature( image, feature, tmpPos, res ) == SUCCESS ){
	Face.AddDB ( feature, name );
}
```
### 29. Save face
> int SaveDB()
>   
> Function:<br>
> Save the results of adding, modifying, and deleting faces to a database file；If it is not called，the modification is invalid after exiting the program.
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
Face.AddDB(feature,name);
...
//After adding faces, make sure to execute Face.SaveDB () before restarting the device;
Face.SaveDB();
```
### 30. Delete face
> int DelDB(string name)
>   
> Function:<br>
> &emsp;&emsp;Delete face
> 
> Parameters:<br>
> &emsp;&emsp;name : Registered name
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
String name= "test";
Face.DelDB (name);
```
### 31. Delete all registered faces
> int DelAllDB()
>   
> Function:<br>
> Delete all registered faces
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
Face.DelAllDB ();
```
### 32. Querying Face
> String QueryDB( float[] feature, float [] score )
> 
> Function:<br>
> Returns the data that is closest to the input facial features.
> 
> Parameters:<br>
> &emsp;&emsp;feature : Facial features
> 
> &emsp;&emsp;score : Store the similarity score of the queried face data and the input face data
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return registered name<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return unknown

<br>

Example ：
```
float[] score = new float[1];
String name = Face.QueryDB( feature, score );
if( score > thresh_hold ){
// TODO
}
```
### 33. Close the face database
> int CloseDB()
>    
> Function:<br>
> Close the face database
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
if( Face.CloseDB() == SUCCESS ){
// TODO
}
```
### 34. Load Quick Compare Function
> int FastQueryInit()
>    
> Function:<br>
> Load Quick Compare Function
> 
> Parameters: <br>
> &emsp;&emsp;void
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
if(Face.FastQueryInit () == SUCCESS){
// TODO
}
```
### 35. Refresh fast match buffer
> int FastQueryFlush(float [] data, int num)
>    
> Function:<br>
> Refresh fast match buffer
> 
> Parameters:<br>
> &emsp;&emsp;Data : External feature buffer
> 
> &emsp;&emsp;Num : Number of features
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
if(Face.FastQueryFlush(data,num) == SUCCESS){
// TODO
}
```
### 36. Quick Query
> int FastQuery(float[] data, float[] feature, float[] scores, int num)
>    
> Function:<br>
> &emsp;&emsp;Quick query
> 
> Parameters:<br>
> &emsp;&emsp;Data : External feature buffer
> 
> &emsp;&emsp;feature : The feature to query
> 
> &emsp;&emsp;scores:Similarity score
> 
> &emsp;&emsp;num : Number of features
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution succeed return SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Execution failed return ERROR_FAILURE

<br>

Example ：
```
if(Face.FastQuery(data,feature,num) == SUCCESS){
// TODO
}
```
### 37. Get face attributes
> int GetFaceAttr( Image image, FaceInfo data, FaceAttribute face_attr, int *res )
>    
> Function:<br>
> After Detect is executed, the face attributes including age, gender and expression are obtained by detecting the acquired face position and keypoint information.
> 
> Parameters:<br>
> &emsp;&emsp;image : Image object containing a face image
> 
> &emsp;&emsp;data : Detected face position and face keypoint information
> 
> &emsp;&emsp;face_attr : Store calculated face attributes
> 
> &emsp;&emsp;res : Face recognition results.The length of res is always 1.If no face is recognized in the picture,res[0] return ERROR_INVALID_PARAM.
> 
> Returns<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : Obtain face attributes succeed<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : Obtain face attributes failed

<br>

Example ：
```
if( face.GetFaceAttr( Image, data, attr, res ) == SUCCESS ){
// TODO
}
```
### 38. Release face recognition resources
> public void Destroy()
>    
> Function:<br>
> Free resources allocated when initializing and setting Parameters
> 
> Parameters:<br>
> &emsp;&emsp;void
> 
> Returns&emsp;&emsp;void

<br>

Example ：
```
Face.Destroy(); 
```
### 39. Sample code
Initialize the binocular camera face recognition demo.
```
public class MainActivity extends Activity implements CvCameraViewListener2 {

    private FaceAPP face= FaceAPP.GetInstance(); //face as member variable
    .........
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      .........
        String[] name={"a","b","c","d","factor","min_size","clarity","perf-optimize","liveness-detect","gray2color-scale"};
	double[] value={0.9,0.9,0.9,0.715,0.6,64,400,1,0,0.5};
	face.SetParameter(name,value);
	mainLoop = new Thread() { //Face detection needs to run on a child thread

	public void run() {
	    .........
	    float[] feature=new float[512];
	    byte[] tmpPos = new byte[1024];// used to store position information
            switch (mixController.curState){
                 ......
                 case mixController. STATE_IDLE :
                      FaceAPP.Image image= FaceAPP. GetInstance ().new Image();
                      image.matAddrframe=mRgbaFrame.getNativeObjAddr();
                      int[] res=new int[1];
                      int ret;
                      ret= face.GetFeature(image,feature,tmpPos,res);
                      if(ret== SUCCESS){
                            //to do Successfully obtained facial feature values
                       }
            }
        }
    }
}

```