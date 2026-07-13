# 人脸识别API  V1.0


## 硬件接口API


### 1. 补光灯控制开关

控制补光灯实例代码 ：
```
//红灯
HardwareCtrl.ctrlLedSwitch( HardwareCtrl.LED_RED, true);
or
HardwareCtrl.ctrlLedRed(true);

//绿灯
HardwareCtrl.ctrlLedSwitch( HardwareCtrl.LED_GREEN, true);
or
HardwareCtrl.ctrlLedGreen(true);

//白灯
HardwareCtrl.ctrlLedSwitch( HardwareCtrl.LED_WHITE, true);
or
HardwareCtrl.ctrlLedWhite(true);

//只开白灯,并把其他补光灯关闭。
HardwareCtrl.ctrlLedOnlyOpenWhite();//比如只开绿灯：ctrlLedOnlyOpenGreen(), 只开红灯：ctrlLedOnlyOpenRed();

//硬件版本2.3以上支持调节白色补光灯亮度，绿灯和红灯不支持调节亮度。
HardwareCtrl.ctrlWhiteLightness(0);//控制白色补光灯亮度：取值范围：0～8
```
<br>

### 2. 控制屏幕亮度

> public static void setBrightness(int value)
>
> 功能&emsp;&emsp;&emsp;调节屏幕亮度
>
> 参数&emsp;&emsp;&emsp;value : 有效值 0～255

<br>

实例代码 ：
```
HardwareCtrl.setBrightness(255);

```
### 3. 背光控制开关
> public static void ctrlBlPower(boolean open)
>
> 功能&emsp;&emsp;&emsp;背光控制开关
>
> 参数&emsp;&emsp;&emsp;open : true为打开，false为关闭

<br>

实例代码 ：
```
HardwareCtrl.ctrlBlPower(true);

```
### 4. 屏幕触摸开关
> public static void ctrlTp(boolean open)
>
> 功能&emsp;&emsp;&emsp;屏幕触摸开关
>
> 参数&emsp;&emsp;&emsp;open : true为打开，false为关上

<br>

实例代码 ：
```
HardwareCtrl.ctrlTp(true);

```
### 5. RS485/RS232信号控制
打开RS485/RS232
> public static SerialPort openSerialPortSignal(File device, int baudrate, SerialPort.Callback callback)
>
> 功能&emsp;&emsp;&emsp;打卡RS485/RS232
>
> 参数&emsp;&emsp;&emsp;device : 串口文件<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;baudrate : 波特率<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;callback : 信息回调接口

<br>
发送RS485/RS232

> public static void sendSerialPortHexMsg(SerialPort mSerialPort, String msg)
>
> 功能&emsp;&emsp;&emsp;发送RS485/RS232信号
>
> 参数&emsp;&emsp;&emsp;mSerialPort : 串口对象<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;msg : 信号 (十六进制信号，比如"1E60010000002F")

<br>
关闭RS485/RS232

> public statis void closeSerialPortSignal(SerialPort mSerialPort)
>
> 功能&emsp;&emsp;&emsp;关闭RS485/RS232
>
> 参数&emsp;&emsp;&emsp;无

<br>

实例代码 ：
```
//输入相关内容
/**
比如：
1.A向刷卡开闸,上位机需发送十六进制数据:
发送:0x1E 0x60 0x01 0x00 0x00 0x00 0x2F
闸机返回码分以下几种:
a),人已经通过闸机
返回: 0x1E 0x61 0x01 0x00 0x00 0x00 0x2F
b)刷卡后超时未通过闸机, 闸机自动关门,取消此次通行
返回: 0x1E 0x44 0x01 0x00 0x00 0x00 0x2F
c)刷卡后有人反向过闸机, 闸机自动关门,取消此次通行
返回: 0x1E 0x44 0x01 0x00 0x00 0x00 0x2F
*/
//RS485/RS232
SerialPort mSerialPort = HardwareCtrl.openSerialPortSignal(new File("dev/ttyS4"), 9600, new SerialPort.Callback() {
     @Override
     public void onDataReceived(byte[] buffer, int size) {
         String result = StringUtils.bytesToHexString(buffer, size);
         Log.e("lkdong","result = "+result);
     }
});

发送信号
HardwareCtrl.sendSerialPortHexMsg(mSerialPort, "1E60010000002F");
//关闭RS485/RS232
HardwareCtrl.closeSerialPortSignal(mSerialPort);

```
<font color='red'> 
注意：<br>
1. 485信号控制旧接口可以继续使用, 比如：HardwareCtrl.openRs485Signal(File device, int baudrate, SerialPort.Callback callback)、
HardwareCtrl.sendRs485Signal(SerialPort mSerialPort, String msg)和HardwareCtrl.closeRs485Signal(SerialPort mSerialPort)
<br>
<br>
2. 485串口：/dev/ttyS4  ;  232串口：/dev/ttyS3<br>
<br>
<br>
</font>

### 6. 韦根26/34信号控制
#### 韦根26
> public static void sendWiegandSignal(String msg)
>
> 功能&emsp;&emsp;&emsp;韦根信号控制
>
> 参数&emsp;&emsp;&emsp;msg : 比如卡号等等
<br>

#### 韦根34
> public static void sendWiegand34Signal(String msg)
>
> 功能&emsp;&emsp;&emsp;韦根信号控制
>
> 参数&emsp;&emsp;&emsp;msg : 比如卡号等等
<br>

实例代码 ：
```
//输入相关内容, 比如卡号等等
HardwareCtrl.sendWiegand34Signal("1233456789");

```
<font color='red'> 注意：韦根26和韦根34共用一条线路，所以在同一时间内，只能使用其中一个</font>

### 7.韦根输入
> public static void recvWiegandSignal(RecvWiegandCallBack callBack)
> 
> 功能&emsp;&emsp;&emsp;韦根信号输入
> 
> 参数&emsp;&emsp;&emsp;callBack : 韦根输入信号返回值接口
<br>

实例代码 ：
```
//韦根输入
HardwareCtrl.openRecvMiegandSignal("/dev/wiegand");//打开串口
HardwareCtrl.recvWiegandSignal(new RecvWiegandCallBack() {
    @Override
    public void recvWiegandMsg(int i) {
       Log.e("lkdong","recvWiegandMsg = "+i);
    }
});
HardwareCtrl.closeRecvMiegandSignal();
```
### 8. 继电器信号
> public static void sendRelaySignal(boolean up)
> 
> 功能&emsp;&emsp;&emsp;继电器控制
> 
> 参数&emsp;&emsp;&emsp;up : 信号拉高为true，拉低为false。
<br>

实例代码 ：
```
HardwareCtrl.sendRelaySignal(!HardwareCtrl.getRelayValue());
```
### 9. 雷达
雷达功能工作的原理：雷达辐射范围内，有物体活动和没有物体活动都会上报键值到Android应用层，开发者可以根据自己的需要进行开发。
<br>

实例代码 ：
```
public class MainActivity extends AppCompatActivity {
    //雷达键值对
    public static final int KEYCODE_RADAR_IN = 305;//有物体活动
    public static final int KEYCODE_RADAR_OUT = 306;//没有物体活动

    @Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        Log.v("lkdong", "dispatchKeyEvent keycode=" + event.getKeyCode());
        if (event.getKeyCode() == KEYCODE_RADAR_IN) {//雷达上报有物体活动
            if (event.getAction() == KeyEvent.ACTION_UP) {
		//TODO
            }
        } else if (event.getKeyCode() == KEYCODE_RADAR_OUT) {//雷达上报没有物体活动
            if (event.getAction() == KeyEvent.ACTION_UP) {
		//TODO
            }
        } 
        return super.dispatchKeyEvent(event);
    }

}
```
### 10.NFC
nfc功能工作的原理：刷卡时，系统会上报键值到Android应用层，同时把卡的信息保存在/dev/dl1825节点上。在开发nfc读卡时，需要在接收到系统会上报键值后，才可以去读/dev/dl1825节点值。

实例代码 ：
```
public class MainActivity extends AppCompatActivity {
    //NFC键值对
    public static final int KEYCODE_NFC=307;

     @Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        Log.v("lkdong", "dispatchKeyEvent keycode=" + event.getKeyCode());
        if (event.getKeyCode() == KEYCODE_NFC) {
            if (event.getAction() == KeyEvent.ACTION_UP) {
                String mNfcCode = NfcUtil.getCode();//nfc读卡，详细的实现流程，请参考最下面gpioDemo

            }
        }
        return super.dispatchKeyEvent(event);
    }

}
```

### 11.二维码（qrcode）
二维码读取主要是通过串口操作，具体操作流程如下：

实例代码 ：
```
public class QrCodeUtil {
    public static final String TAG = QrCodeUtil.class.getSimpleName();
    //当前命令基于模块E3000H
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
     * 启用二维码识别并打开串口
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
     * 关闭二维码识别串口
     */
    public static void close() {
        if (isQrCodeSupport() && sQrCodeUtil != null) {
            sQrCodeUtil.closeQrCodeSerialPort();
        }
    }

    /**
     * 关闭串口并释放
     */
    public static void destory() {
        if (isQrCodeSupport() && sQrCodeUtil != null) {
            sQrCodeUtil.closeQrCodeSerialPort();
            sQrCodeUtil = null;
        }
    }


    /**
     * 打开串口
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
     * 关闭串口
     */
    private void closeQrCodeSerialPort() {
        Log.v(TAG, "closeQrCodeSerialPort()");
        if (mQrCodeSerialPort != null) {
            HardwareCtrl.closeQrCodeSerialPort(mQrCodeSerialPort);
            mQrCodeSerialPort = null;
        }
    }

    /**
     * 打开 识别配置二维码 功能
     */
    private void enableQRCodeConfig() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(ENABLE_QR_CONFIG);
        }
    }

    /**
     * 关闭 识别配置二维码 功能
     */
    private void disableQRCodeConfig() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(DISABLE_QR_CONFIG);
        }
    }

    /**
     * 补光灯在扫描时自动打开
     */
    private void setLEDAuto() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(LED_AUTO);
        }
    }

    /**
     * 补光灯一直关闭
     */
    private void setLEDOFF() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(LED_OFF);
        }
    }

    /**
     * 补光灯一直打开
     */
    private void setLEDON() {
        if (mQrCodeSerialPort != null) {
            mQrCodeSerialPort.sendHexMsg(LED_ON);
        }
    }

    /**
     * 配置二维码扫描补光灯状态
     * 目前仅分：扫描时自动打开、常关
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
     * 支持二维码扫描成功的监听
     *
     * @param callback
     */
    public void setQRCodeCallback(QRCodeCallback callback) {
        mQRCodeCallback = callback;
    }
}
```

### 12. 普通GPIO控制
D0 信号
> public static void sendSignalD0(boolean up)
>
> 功能&emsp;&emsp;&emsp;普通GPIO D0信号控制
>
> 参数&emsp;&emsp;&emsp;up : false为拉低，true为拉高

<br>

实例代码 ：
```
HardwareCtrl.sendSignalD0(true);

```
D1 信号
> public static void sendSignalD1(boolean up)
>
> 功能&emsp;&emsp;&emsp;普通GPIO D1控制
>
> 参数&emsp;&emsp;&emsp;up : false为拉低，true为拉高

<br>

实例代码 ：
```
HardwareCtrl.sendSignalD1(true);

```
### 13. 关机
> public static void shutdown()
>
> 功能&emsp;&emsp;&emsp;关机
>
> 参数&emsp;&emsp;&emsp;无

<br>

实例代码 ：
```
HardwareCtrl.shutdown();

```
### 14. 重启设备
> public static void reboot()
>
> 功能&emsp;&emsp;&emsp;重启设备
>
> 参数&emsp;&emsp;&emsp;无

<br>

实例代码 ：
```
HardwareCtrl.reboot();

```
### 15. 看门狗
> public static void setWdt(int value)
>
> 功能&emsp;&emsp;&emsp;系统死机或者长时间没有响应，重启设备
>
> 参数&emsp;&emsp;&emsp;value ： 有效值：0～3<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0对应是0.46s<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1对应是2.56s<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2对应是10.24s<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;3对应是40.96s

<br>

实例代码 ：
```
HardwareCtrl.ctrlWdt(1);

```
### 16. 获取设备唯一ID
> public static String getFireflyCid()
>
> 功能&emsp;&emsp;&emsp;设备唯一id
>
> 参数&emsp;&emsp;&emsp;无

<br>

实例代码 ：
```
String cid = HardwareCtrl.getFireflyCid();
```

### 17. 其他命令使用
> public static void execSuCmd(String command) 
>
> 功能&emsp;&emsp;&emsp;其他shell命令的使用
>
> 参数&emsp;&emsp;&emsp;command：需要执行的命令

<br>

实例代码 ：
```
//比如同步文件等等
HardwareCtrl.execSuCmd("sync");

```
### 18. 其他GPIO使用
> public static int gpioParse(String gpioStr)
>
> 功能&emsp;&emsp;&emsp;将gpio名字转换成对应的gpio编码
>
> 参数&emsp;&emsp;&emsp;gpioStr：gpio名字,比如GPIO2_A2

<br>
控制GPIO

> public static void ctrlGpio(int gpio, String direction, int value)
>
> 功能&emsp;&emsp;&emsp;控制GPIO
>
> 参数&emsp;&emsp;&emsp;gpio：gpio编码,比如152<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;direction : <br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;value : 设置GPIO值

<br>

实例代码 ：
```
HardwareCtrl.ctrlGpio(HardwareCtrl.gpioParse("GPIO2_A2"), "out", 1);

```

### 19.TestGpioDemo下载链接 

* [https://pan.baidu.com/s/1EeCIUoHA4i_RQFHV7fgsrg](https://pan.baidu.com/s/1EeCIUoHA4i_RQFHV7fgsrg) 提取码：d4yq



## Android 快速入门


<h4>1. 添加依赖库</h4>
```
复制 facelibrary-release.aar和openCVLibrary331-release.aar 到工程下 app/libs 目录
```
<h4>2. 添加编译 aar 信息</h4>
```
compile(name: 'facelibrary-release', ext: 'aar')
compile(name: 'openCVLibrary331-release', ext: 'aar')
    
到 build.gradle 的 dependencies
```
<h4>3. 设置权限</h4>
```
在AndroidManifest.xml文件中添加如下权限

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-feature android:name="android.hardware.camera" android:required="true"/>
<uses-permission android:name="android.permission.CAMERA"/>
```
<h4>4. 在程序中定义 FaceAPP 变量</h4>
```
private FaceAPP face = FaceAPP.GetInstance(); //face 作为成员变量
```
<h4>5. 通过 face 调用 Android API 接口</h4>



## 人脸识别SDK
本SDK开发指南指导您如何安装和配置开发环境,如何通过调用 SDK 提供的接口函数(API)进行二次开发与系统集成。
用户按照要求调用SDK提供的API即可实现使用 人脸检测/跟踪、活体识别、人脸识别等服务的目的。

### 1. 主要返回参数
> public static final int SUCCESS = 0; //执行接口返回成功<br>
> public static final int ERROR_INVALID_PARAM = -1; //非法参数<br>
> public static final int ERROR_TOO_MANY_REQUESTS = -2; //太多请求<br>
> public static final int ERROR_NOT_EXIST = -3; //不存在<br>
> public static final int ERROR_FAILURE = -4; // 执行接口返回失败

<br>

### 2. FaceInfo 人脸信息
```
public class FaceInfo {
     public Rect mRect;//人脸方框
     public FaceAttribute mAttr;//人脸属性
     public FaceQuality mQuality;//人脸质量
     public Landmark mLandmark;//用于存储5个关键点坐标值，依次是左眼、右眼、鼻子、左侧嘴唇、右侧嘴唇。
}
```

### 3. FaceAttribute 人脸属性
```
public class FaceAttribute {
     public int mGender;//性别 0：男性；1：女性
     public int mEmotion;//表情 0：平静；1：高兴
     public int mAge;//年龄
}
```

### 4. FaceQuality 人脸质量

```
public class FaceQuality {
     public float mScore;//人脸质量的置信度
     public float mLeftRight;//左右角度
     public float mUpDown;//上下角度
     public float mHorizontal;//水平角度
     public float mClarity;//图片清晰度
     public float mBright;//亮度
}
```

### 5. 构造函数
> static FaceAPP GetInstance()
> 
> 功能&emsp;&emsp;&emsp;获取单例的对象,人脸识别类采用单例模式,一个类Class只有一个实例存在
> 
> 参数&emsp;&emsp;&emsp;无
> 
> 返回值&emsp;&emsp;FaceAPP类型的对象

<br>

实例代码 ：
```
private FaceAPP face = FaceAPP.GetInstance();

```
### 6. 识别人脸特征
> int Recognize( Image image, float featureArray [][512], int size, List<FaceInfo> faceinfos, int[] res ) 
> 
> 功能&emsp;&emsp;&emsp;识别提交的Image中的人脸特征,然后和featureArray里这些特征数组进行比较,找出其中相似度<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;最高的返回特征数组的二维数组的索引值
> 
> 参数&emsp;&emsp;&emsp;image : 人脸图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;featureArray : 特征数组的二维数组,特征值数组是存储人脸特征信息的数组,由512个float组成。<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;size : 特征数组的二维数组大小<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息><br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;image中人脸不在feature数组中返回 ERROR_NOT_EXIST, >=0特征数组的索引
>
> 返回值&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
float[][] featurelist = new float[][]; //存储特征值的数组
int size = featurelist.lenth;
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];
FaceAPP.Image image = FaceAPP.GetInstance().new Image();
image.matAddrframe = mRgbaFrame.getNativeObjAddr();
face.Recognize( image, featurelist, size, tmpPos, res );
```

### 7. 识别人脸特征(根据特征值)
> int Recognize( float[] feature, float featureArray [][512], int size, float[] high, int[] res )
>    
> 功能&emsp;&emsp;&emsp;根据已经存在的人脸特征,然后和featureArray里这些特征数组进行比较,找出其中相似度<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;最高的返回特征数组的二维数组的索引值,返回相似度得分值。
> 
> 参数&emsp;&emsp;&emsp;feature : 特征数组<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;featureArray : 特征数组的二维数组,特征值数组是存储人脸特征信息的数组,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;由512个float组成。<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;size : 特征数组的二维数组大小<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;high : float[]型,返回最大得分值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;image中人脸不在feature数组中返回 ERROR_NOT_EXIST,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;>=0特征数组的索引号
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
float[][] featurelist = new float[][]; //存储特征值的数组
int size = featurelist.lenth;
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];
float[] feature;
float[] high = float[1];
FaceAPP.Image image = FaceAPP.GetInstance().new Image();
image.matAddrframe = mRgbaFrame.getNativeObjAddr();
face.Recognize( feature, featurelist, size, high, tmpPos, res );
```

### 8. 检测人脸
> int Detect( Image image, List<FaceInfo> faceinfos, int[] res )
>   
> 功能&emsp;&emsp;&emsp;检测提交的图片中的是否有人脸
>
> 参数&emsp;&emsp;&emsp;image : 人脸图片,用于检测的图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息><br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];//byte数组用于存位置信息
FaceAPP.Image image = FaceAPP.GetInstance().new Image(); //初始化
image.matAddrframe = mRgbaFrame.getNativeObjAddr(); //image赋值
if( success = face.Detect( image, tmpPos, res ) ){
//to do
};
```
### 9. 比较特征数据
> int Compare( float[] origin, float[] chose, float score )
>   
> 功能&emsp;&emsp;&emsp;用于比较两个feature值相似度
> 
> 参数&emsp;&emsp;&emsp;origin : 待比较feature数组<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;chose : 用于比较的feature数组<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;score : origin和chose比较的相似度
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
float score;
float[] origin = new float[512];
Float[] chose = new float[512];
face.Compare( origin, chose, score );
```
### 10. 双目带有活体的提取人脸特征
> int GetFeature( Image image, Image grayImage, float[] feature, List<FaceInfo> faceinfos, int[] res )
>   
> 功能&emsp;&emsp;&emsp;获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;由512个float型数字组成,只获取图片中一个人的特征,多于一人会返回错误信息。
> 
> 参数&emsp;&emsp;&emsp;image : 人脸图片,用于检测的图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;grayImage : 红外摄像头获取的图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;feature : 存储image中检测到的人脸特征信息,无人脸返回空数组<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息><br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 正确获取人脸信息<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 未获取到人脸信息

<br>

实例代码 ：
```
float[] feature = new float[512];
int[] ret = new int[1];
byte[] tmpPos = new byte[1024]; //byte数组用于存位置信息
ret = face.GetFeature( image, grayImage, feature, tmpPos, res);
if( ret == SUCCESS ){
//to do 成功获取到活体人脸特征值
}
```
### 11. 活体检测
> int DetectLiveness(Image image, List<FaceInfo> faceinfos, int[] res)
> 
> 功能&emsp;&emsp;&emsp;检测识别活体和非活体。
> 
> 参数&emsp;&emsp;&emsp;image : 彩色人脸图片,用于检测和识别的图片。<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息><br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 是活体<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 非活体

<br>

实例代码 ：
```
int[] ret=new int[1];
ret= face.DetectLiveness(image,grayImage ,tmpPos,res);
if(ret== SUCCESS){
//to do 活体检测成功
}
```
### 12. 双目活体检测
> int GetDetectLiveness(Image image, Image grayImage, List<FaceInfo> faceinfos, int[] res)
> 
> 功能&emsp;&emsp;&emsp;检测识别活体和非活体。
> 
> 参数&emsp;&emsp;&emsp;image : 彩色人脸图片,用于检测和识别的图片。<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;grayImage :红外摄像头获取的图片。<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息><br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 是活体<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 非活体

<br>

实例代码 ：
```
int[] ret=new int[1];
ret= face.GetDetectLiveness(image,grayImage ,tmpPos,res);
if(ret== SUCCESS){
//to do 活体检测成功
}
```
### 13. 双目校准
> int Calibration( Image image, Image grayImage, float[] scale, int[] Rect, int[] res );
>    
> 功能&emsp;&emsp;&emsp;对红外和普通光组成的双摄像头识别进行校准,要求1个人在最佳位置(0.8-1米)站定,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;大约需要校验20次,得到人脸框修正参数用于人脸画框,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;返回红外摄像头相对普通光的显示区域坐标,该区域是有效识别和活体检测区域
> 
> 参数&emsp;&emsp;&emsp;image : 输入红外图像<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;grayImage : 输入彩色图像<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;scale : 输出 人脸框修正参数<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;rect : 输出红外图像与彩色图像重叠区域(红外图像在彩色图像的对应区域/推荐的检测区域)<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 正确校准<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 校准失败

<br>

实例代码 ：
```
int[] ret = new int[1];
float[] scale = new float[1];
int[] rect = new int[4];
ret = face.Calibration( image, grayimage, scale, rect, res );
if( ret == SUCCESS ){
//to do 校准成功
}
```
### 14. 设备激活一
> int AuthorizedDevice( String uidStr, String password, Context activity )
>    
> 功能&emsp;&emsp;&emsp;设备激活
> 
> 参数&emsp;&emsp;&emsp;uidStr : OEMID号+合同号<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;password : 用户密码
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : 鉴权成功

<br>

实例代码 ：
```
String oem_id = "1000000000000001";//OEMID
String contract_id = "0001";//合同号
String password = "0123456789abcdef0123456789abcdef"; //初始授权密码
String uidStr = oem_id + contract_id;
int res = face.AuthorizedDevice( uidStr, password, LoginActivity.this ); 
```
### 15. 设备激活二
> int fireflyInit(Context context, String uidStr, String password) 
>    
> 功能&emsp;&emsp;&emsp;设备激活，这个接口是firefly一个临时接口，可以永久激活设备，该接口到后面可能会被移除，<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;如果后续开发文档中没有该接口说明，则已被移除。开发文档以开源Demo中的文档为准。
> 
> 参数&emsp;&emsp;&emsp;uidStr : OEMID号+合同号<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;password : 用户密码
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : 鉴权成功

<br>

实例代码 ：
```
String oem_id = "1000000000000001";//OEMID
String contract_id = "0001";//合同号
String password = "0123456789abcdef0123456789abcdef"; //初始授权密码
String uidStr = oem_id + contract_id;
int res =face.fireflyInit(LoginActivity.this, uidStr, password);
```
### 16. 设备激活三
> int AuthorizedDeviceUserPassword(String uidStr, String password, Context context, String userPassword)
>    
> 功能&emsp;&emsp;&emsp;设备激活，这个接口主要用于带有用户密码的激活方式。
> 
> 参数&emsp;&emsp;&emsp;uidStr : OEMID号+合同号<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;password : 授权密码<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;userPassword:用户密码
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : 鉴权成功

<br>

实例代码 ：
```
String oem_id ="1000000000000001";//OEMID
String contract_id ="0001";//合同号
String password = "0123456789abcdef0123456789abcdef";//初始授权密码
String userPassword ="012345678912";//授权密码;
String uidStr = oem_id+contract_id;
int res =face. AuthorizedDeviceUserPassword(uidStr,password,
LoginActivity.this,userPassword);
```
### 17. 获取鉴权激活状态值
> int getAuthStatus()
>   
> 功能&emsp;&emsp;&emsp;获取鉴权激活状态值
> 
> 参数&emsp;&emsp;&emsp;无
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;0 : 鉴权成功

<br>

实例代码 ：
```
int res = face.getAuthStatus(); 
```
### 18. 提取人脸特征
> int GetFeature( Image image, float[] feature, List<FaceInfo> faceinfos, int[] res )
>    
> 功能&emsp;&emsp;&emsp;获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;由512个float型数字组成,只获取图片中一个人的特征。
>
> 参数&emsp;&emsp;&emsp;image : 人脸图片,用于检测的图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;feature : 存储image中检测到的人脸特征信息,无人脸返回空数组<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息><br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 正确获取人脸信息<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 未获取到人脸信息

<br>

实例代码 ：
```
float[] feature = new float[512];
int[] ret = new int[1];
byte[] tmpPos = new byte[1024]; //byte数组用于存位置信息
ret = face.GetFeature( image, feature, tmpPos, res );
if( ret == SUCCESS ){
//to do 成功获取到人脸特征值
}
```
### 19. 提取人脸特征(根据人脸坐标信息)
> int GetFeature( Image image, FaceInfo detectInfo, float[] feature, int[] res )
>   
> 功能&emsp;&emsp;&emsp;根据传入的人脸和关键点坐标信息,获取image中的人脸特征值数组,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;特征值数组是存储人脸特征信息的数组,由512个float型数字组成,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;只获取图片中一个人的特征。
> 
> 参数&emsp;&emsp;&emsp;image : 人脸图片,用于检测的图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;detectInfo : 检测到的人脸信息,用于人脸特征提取<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;feature : 存储image中检测到的人脸特征信息,无人脸返回空数组<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 正确获取人脸信息<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 未获取到人脸信息

<br>

实例代码 ：
```
float[] feature = new float[512];
int[] ret = new int[1];
float[] detectinfo = new float[]{ x0, y0, x1, y1, landmarkx0, landmarky0,
				 landmarkx1, landmarky1, landmarkx2, landmarky2,
				 landmarkx3, landmarky3, landmarkx4, landmarky4 }
byte[] tmpPos = new byte[1024]; //byte数组用于存位置信息
ret = face.GetFeature( image, detectinfo, feature, res );
if( ret == SUCCESS ){
//to do 成功获取到人脸特征值
}
```
### 20. 提取人脸特征(根据图片文件)
> float[] GetFeature(String path, List<FaceInfo> faceinfos)
>   
> 功能&emsp;&emsp;&emsp;根据传入的图片文件路径,获取image中的人脸特征值数据。<br>
> 
> 参数&emsp;&emsp;&emsp;path :人脸图片文件的绝对路径<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的信息>
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Float[] : 人脸特征值数组

<br>

实例代码 ：
```
int ret = this.FaceGetFeatureFromAddr(addr, feature, mFaceInfos, policy);

if (ret == -1) {
     return null;
} else {
     faceInfos.addAll(Arrays.asList(this.mFaceInfos).subList(0,ret));
     return this.feature;
}
```
### 21. 提取人脸关键点
> int GetLandmark ( Image image, float[] landmark, int[] res )
>    
> 功能&emsp;&emsp;&emsp;获取人脸关键点坐标信息
>
> 参数&emsp;&emsp;&emsp;Image : 人脸图片,用于提取人脸关键点信息的图片,只提取一个人的关键点信息<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Landmark : 用于存储5个关键点坐标值,依次是左眼,右眼,鼻子,左侧嘴唇,右侧嘴唇<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 正确获取人脸关键点坐标信息<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 未获取到人脸关键点信息

<br>

实例代码 ：
```
float[] landmark = new float[10];
int[] ret = new int[1];
ret = face.GetLandmark( image, landmark, res);
if( ret == SUCCESS ){

}
```
### 22. 人脸质量提取
> FaceInfo getQuality(long matAddrframe)
>    
> 功能&emsp;&emsp;&emsp;传入图像,返回图像中人脸位置和最大一张人脸的质量信息。
> 
> 参数&emsp;&emsp;&emsp;matAddrframe : 保存在Mat中图像地址<br>
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;FaceInfo : 用于保存返回人脸在图片中的信息

<br>

实例代码 ：
```
Faceinfo faceinfo=new Faceinfo();
faceinfo=face.getQuality(matAddrframe);
if(faceinfo!=null){
}
```
### 23. 参数设置
> bool SetParameter( const String[] name, float value[] )
>    
> 功能&emsp;&emsp;&emsp;设置输入的参数名和对应数值
> 
> 参数&emsp;&emsp;&emsp;char[] name : 参数的名字<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;a : a参数值 内部参数,按示例设置,请不要随意修改<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;b : b参数值 内部参数,按示例设置,请不要随意修改<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;c : c参数值 内部参数,按示例设置,请不要随意修改<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;d : d参数值 内部参数,按示例设置,请不要随意修改<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;factor : 检测人脸放大比例 内部参数,按示例设置,请不要随意修改<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;min_size : 最小人脸框大小 范围 32-80<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;faceclarity : 照片清晰度阈值 范围 建议200-400<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;perfoptimize : 是否优化效果 范围 0或1<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;livenessdetect : 是否活体检测 范围0-1<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;gray2colorscale : 双目活体检测比值 范围 0.1-0.5<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;frame_num : 优化的帧数,范围20-40<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;quality_thresh : 图片质量阀值,建议范围0.7-0.8<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;mode : 工作模式 0 闸机 1 门禁<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;facenum : 检测最大人脸数,最多支持检测3张人脸识别1张脸,范围1-3<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;value[] : 参数的数值(可能多个)
>
> 返回值&emsp;&emsp;参数设置是否成功

<br>

实例代码 ：
```
String[] name = { "a", "b","c", "d", "factor", "min_size", "clarity", "perfoptimize", 
		  "livenessdetect", "gray2colorscale", "frame_num", "qualit_thresh",
		  "mode", "facenum" };
double[] value = {0.9, 0.9, 0.9, 0.715, 0.6, 64, 400, 1, 0, 0.5, 20, 0.8, 1, 1 };
face.SetParameter( name, value );
```
### 24. 参数设置(参数可变长度)
> bool SetParameters( String[] name, float value[] )
>    
> 功能&emsp;&emsp;&emsp;可变长度的设置输入的参数名和对应数值
>
> 参数&emsp;&emsp;&emsp;char[] name : 参数的名字(>=1),详情见14,<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;value[] : 参数的数值(>=1个)
>
> 返回值&emsp;&emsp;参数设置是否成功

<br>

实例代码 ：
```
String[] name = { "perfoptimize", "livenessdetect", "frame_num", "quality_thresh", 
		  "mode", "facenum" };
double[] value = { 1, 0, 20, 0.8, 1, 1 };
face.SetParameter( name, value );
```
### 25. 获取版本信息
> public String GetVersion()
>    
> 功能&emsp;&emsp;&emsp;获取当前SDK版本的信息
>
> 参数&emsp;&emsp;&emsp;无

> 返回值&emsp;&emsp;返回当前版本信息的字符串

<br>

实例代码 ：
```
Face.GetVersion();
```
### 26. 获取底层库信息
> public String GetFacelibVersion()
>  
> 功能&emsp;&emsp;&emsp;获取当前SDK底层库的信息
>
> 参数&emsp;&emsp;&emsp;无
>
> 返回值&emsp;&emsp;返回当前版本底层库的字符串

<br>

实例代码 ：
```
Face.GetFacelibVersion();
```
### 27. 打开人脸数据库
> int OpenDB()
>  
> 功能&emsp;&emsp;&emsp;使用人脸数据库,内部人脸数据库可实现1:N高效快速查询
>
> 参数&emsp;&emsp;&emsp;无
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
if(face.OpenDB() == SUCCESS){
// TODO
}
```
### 28. 注册人脸
> int AddDB( float[] feature, string name )
>   
> 功能&emsp;&emsp;&emsp;注册人脸
> 
> 参数&emsp;&emsp;&emsp;feature : 人脸特征<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;name : 登记名字
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
String name= "test";
int[] res=new int[];
if( Face.GetFeature( image, feature, tmpPos, res ) == SUCCESS ){
	Face.AddDB ( feature, name );
}
```
### 29. 保存人脸
> int SaveDB()
>   
> 功能&emsp;&emsp;&emsp;将AddDB()写入的数据保存到文件；如果在reboot或者断电重启设备之前，没有调用SaveDB()，<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;则AddDB()添加的人脸将丢失。
> 
> 参数&emsp;&emsp;&emsp;无
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
Face.AddDB(feature,name);
...
//在添加人脸之后，要确保重启设备之前执行Face.SaveDB()；
Face.SaveDB();
```
### 30. 删除人脸
> int DelDB(string name)
>   
> 功能&emsp;&emsp;&emsp;删除人脸
> 
> 参数&emsp;&emsp;&emsp;name : 要删除的名字
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
String name= "test";
Face.DelDB (name);
```
### 31. 删除所有已注册人脸
> int DelAllDB()
>   
> 功能&emsp;&emsp;&emsp;删除所有已注册人脸
> 
> 参数&emsp;&emsp;&emsp;无
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
Face.DelAllDB ();
```
### 32. 查询人脸特征对应名字
> String QueryDB( float[] feature, float [] score )
>
> 功能&emsp;&emsp;&emsp;给定人脸特征最接近的数据库所登记的人脸,并给出相似度
>
> 参数&emsp;&emsp;&emsp;feature : 人脸特征<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;score : 相似度分值
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 返回登记名字<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 返回为unknown

<br>

实例代码 ：
```
float[] score = new float[1];
String name = Face.QueryDB( feature, score );
if( score > thresh_hold ){
// TODO
}
```
### 33. 关闭人脸数据库
> int CloseDB()
>    
> 功能&emsp;&emsp;&emsp;关闭人脸数据库
>
> 参数&emsp;&emsp;&emsp;无
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
if( Face.CloseDB() == SUCCESS ){
// TODO
}
```
### 34. 加载快速比对功能
> int FastQueryInit()
>    
> 功能&emsp;&emsp;&emsp;加载快速比对功能
>
> 参数&emsp;&emsp;&emsp;无
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
if(Face.FastQueryInit () == SUCCESS){
// TODO
}
```
### 35. 刷新快速比对缓存区
> int FastQueryFlush(float [] data, int num)
>    
> 功能&emsp;&emsp;&emsp;刷新快速比对缓存区
> 
> 参数&emsp;&emsp;&emsp;Data : 外部特征缓存区<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Num : 特征数量
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
if(Face.FastQueryFlush(data,num) == SUCCESS){
// TODO
}
```
### 36. 快速查询
> int FastQuery(float[] data, float[] feature, float[] scores, int num)
>    
> 功能&emsp;&emsp;&emsp;快速查询
> 
> 参数&emsp;&emsp;&emsp;Data : 外部特征缓存区<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Feature : 要查询的特征<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Scores:获得的相似度<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Num : 特征数量
> 
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行成功 SUCCESS<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;执行失败 ERROR_FAILURE

<br>

实例代码 ：
```
if(Face.FastQuery(data,feature,num) == SUCCESS){
// TODO
}
```
### 37. 获取人脸属性
> int GetFaceAttr( Image image, FaceInfo data, FaceAttribute face_attr, int *res )
>    
> 功能&emsp;&emsp;&emsp;Detect后执行,通过检测获取的人脸位置和关键点信息,获取人脸属性包括年龄、性别和表情 。
>
> 参数&emsp;&emsp;&emsp;image : 人脸图片,用于检测的图片<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;data : 检测得到的人脸位置和人脸关键点信息<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;face_attr : 存储计算得到的人脸属性<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
>
> 返回值<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;SUCCESS : 正确获取人脸属性<br>
> &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ERROR_FAILURE : 未获取到人脸属性

<br>

实例代码 ：
```
if( face.GetFaceAttr( Image, data, attr, res ) == SUCCESS ){
// TODO
}
```
### 38. 释放人脸识别资源
> public void Destroy()
>    
> 功能&emsp;&emsp;&emsp;释放初始化和设置参数时分配的资源
> 
> 参数&emsp;&emsp;&emsp;无
> 
> 返回值&emsp;&emsp;无

<br>

实例代码 ：
```
Face.Destroy(); 
```
### 39. 示例代码
初始化双目摄像头人脸识别Demo。
```
public class MainActivity extends Activity implements CvCameraViewListener2 {

    private FaceAPP face= FaceAPP.GetInstance(); //face 作为成员变量
    .........
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      .........
        String[] name={"a","b","c","d","factor","min_size","clarity","perf-optimize","liveness-detect","gray2color-scale"};
	double[] value={0.9,0.9,0.9,0.715,0.6,64,400,1,0,0.5};
	face.SetParameter(name,value);
	mainLoop = new Thread() { //人脸检测不要放在 Android 主线程

	public void run() {
	    .........
	    float[] feature=new float[512];
	    byte[] tmpPos = new byte[1024];// byte 数组用于存位置信息
            switch (mixController.curState){
                 ......
                 case mixController. STATE_IDLE :
                      FaceAPP.Image image= FaceAPP. GetInstance ().new Image();
                      image.matAddrframe=mRgbaFrame.getNativeObjAddr();
                      int[] res=new int[1];
                      int ret;
                      ret= face.GetFeature(image,feature,tmpPos,res);
                      if(ret== SUCCESS){
                            //to do 成功获取到人脸特征值
                       }
            }
        }
    }
}

```