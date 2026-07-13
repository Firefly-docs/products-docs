


# 人脸识别API V2.0

## [DEMO|github](https://github.com/T-Firefly/FaceApiDemoV2)

## 硬件接口API

### 1. ICCard/身份证/

* 连接设备
>启动监听服务，监听刷卡操作，建议在onResume()方法中执行；
```

/*
    打开后台监听服务
*/
 IDCardUtil.getInstance().bindIDCardService(Context context);

/*
	nfc模块是以键值形式上报
*/
@Override
public boolean dispatchKeyEvent(KeyEvent event) {
        if (IDCardUtil.getInstance().handleEvent(event)) {return true;}

        return super.dispatchKeyEvent(event);
}
```

* 检查功能支持
>由于监听服务为异步启动，所以在连接设备后立刻进行功能支持检测不一定准确。
>建议在监听回调onMachineConnect中再次进行功能支持检查；
```
/*
	检测设备是否支持身份证识别
    boolean:true 表示支持、false 表示不支持
*/
boolean result =IDCardUtil.getInstance().isSupportIDCard();

/*
	检测设备是否支持ICCard识别
    boolean:true 表示支持、false 表示不支持
*/
boolean result = IDCardUtil.getInstance().isSupportICCard();

```


* 设置刷卡模式
>身份证模块虽然支持身份证和ICCard，但是不支持两者同时读取，需根据需要设置读卡模式；
>NFC模块 只支持 ICCard；

```
/*
   根据指定刷卡时的读取方式;
   主要分为两种：READCARD_MODE_IDENTITY_CARD和READCARD_MODE_IC_CARD；

    IDCardConfig.READCARD_MODE_IDENTITY_CARD = 0;  //身份证模式
    IDCardConfig.READCARD_MODE_IC_CARD = 1; //ICCard
    IDCardConfig.READCARD_MODE_IDENTITY_CARD_UUID = 2; //身份证UUID模式
*/
IDCardUtil.getInstance().setModel(int readMode);
```



* 设置读取ICCard的字节序模式
>设置读取ICCard时的字节序模式，默认为大端；

```
/*
    boolean:true 大端、false 小端
*/
IDCardUtil.getInstance().setICCardEndianMode(boolean useBig);
```



* 设置监听和回调函数

```

/*
	绑定刷卡监听回调
    context 
    callback 监听回调
*/
IDCardUtil.getInstance().setIDCardCallBack(IDCardUtil.IDCardCallBack callBack);
    
//监听回调
IDCardUtil.IDCardCallBack callBack = new IDCardUtil.IDCardCallBack() {

       //监听服务为异步启动,启动后若发现存在读卡设备,则回调onMachineConnect
        @Override
        public void onMachineConnect() {
            Log.i("firefly", "onMachineConnect ");
        }

        //身份证刷卡时执行回调
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

        // IC卡刷卡时执行回调
        @Override
        public void onSwipeICCard(final ICCardBean info) {
             Log.i("firefly", "onSwipeICCard IC=" + info.getIcID());
        }

        // 当设置读卡模式为READCARD_MODE_IDENTITY_CARD_UUID时,刷身份证回调
        @Override
        public void onSwipeIDCardUUID(final String uuid) {
             Log.i("firefly", "onSwipeIDCardUUID uuid=" + uuid);
        }
    };

```



* 断开连接

> 移除监听回调并停止监听服务，建议在onStop()方法中执行；

```

IDCardUtil.getInstance().setIDCardCallBack(null);
IDCardUtil.getInstance().unBindIDCardService(Context context);

```



### 2. 补光灯控制

补光灯分为4种：红外补光灯，白色补光灯，红色补光灯和绿色补光灯；

* 红外补光灯:

>新版机器支持红外补光灯开关(默认为关)，旧版机器不支持开关(默认为开).
>故提供接口供用户使用。

注：闸机进行活体识别失败时可能是因为红外补光灯没有打开，导致IR活体检测失败
```

/*
    检测设备是否支持红外补光灯开关；
    Boolean:true 表示支持、false 表示不支持
*/
HardwareCtrl.isSupportInfraredFillLight();

/*
   红外线补光灯操作；
   open  true 表示打开、false 关闭
*/
HardwareCtrl.setInfraredFillLight(open);

```



* 白色补光灯:

新版机器支持白色补光灯亮度调节,旧版机器只支持开关；

```

/*
	检测设备是否支持白色补光灯亮度调节
    Boolean:true 表示支持亮度调节、false 表示不支持
*/
HardwareCtrl.isFillLightBrightnessSupport();

/*
    获取亮度调节的最大值；
*/
int maxValue = HardwareCtrl.getFillLightBrightnessMax();

/*
    获取亮度调节的最小值；
*/
int minValue = HardwareCtrl.getFillLightBrightnessMin();

/*
    打开/关闭，如果支持亮度调节，则开灯时可以传入对应的亮度值；
    enable  true  打开，并设置对应的亮度值；
            false 关闭
*/
HardwareCtrl.ctrlLedWhite(enable,brightness);

/*
    打开/关闭；
    enable  true  打开，若当前版本支持亮度调节则设为亮度最大值
            false 关闭
*/
HardwareCtrl.ctrlLedWhite(enable);



```



* 红色LED灯；


```

/*
    isChecked  true  打开
    		  false 关闭
*/
HardwareCtrl.ctrlLedRed(isChecked);

```



* 绿色LED灯；


```

/*
    isChecked  true 打开
               false 关闭
*/
HardwareCtrl.ctrlLedGreen(isChecked);

```



### 3 信号控制

* “rs485/232”信号操作；

> 485串口地址：/dev/ttyS4  ，波特率 :19200  
>
>  232串口地址：/dev/ttyS3  ，波特率 :19200
>
>可以通过cat /dev/ttyS4 或者 cat /dev/ttyS3 查看设置的值变化；

```

/*
    获取串号serialPort
    485串口：/dev/ttyS4;  232串口：/dev/ttyS3
*/
SerialPort serialPort = HardwareCtrl.openSerialPortSignal(new File("/dev/ttyS3"), 19200, new SerialPort.Callback() {

        //rs485/232发送信号后，接收到的返回值
         @Override
         public void onDataReceived(byte[] bytes, int i) {
            String result = StringUtils.bytesToHexString(bytes, size);
            Log.i("firefly", "result = "+result);
         }
     });

/*
    发送‘48562311’信号
*/
HardwareCtrl.sendSerialPortHexMsg(serialPort, "48562311")

/*
    页面退出时，关闭串口
*/
HardwareCtrl.closeSerialPortSignal(serialPort);

```


* 韦根信号操作，监听 韦根输入信号；



```

/*开始监听韦根输入*/
HardwareCtrl.openRecvMiegandSignal("/dev/wiegand");

/*监听回调*/
HardwareCtrl.recvWiegandSignal(new RecvWiegandCallBack() {
    @Override
    public void recvWiegandMsg(int i) {
            Log.i("firefly", "result = "+i);
    }
});

/*停止监听韦根输入*/
HardwareCtrl.closeRecvMiegandSignal();

```

-  韦根输出，分为 韦根26和 韦根34；

```
/*
    韦根26Output
    通过cat /sys/devices/platform/wiegand-gpio/wiegand26 查看设置的值变化。
*/
HardwareCtrl.sendWiegandSignal("123456789");

/*
    韦根34Output
    通过cat /sys/devices/platform/wiegand-gpio/wiegand34 查看设置的值变化。
*/
HardwareCtrl.sendWiegand34Signal("123456789");
```



* 电平信号/继电器信号；


```

/*
    D0电平信号 isChecked  true 表示打开、false 关闭
*/
LevelSignalUtil.sendSignalD0(isChecked);

/*
    D1电平信号 isChecked  true 表示打开、false 关闭
*/
LevelSignalUtil.sendSignalD1(isChecked);

/*
    继电器信号 isChecked  true 表示打开、false 关闭
*/
RelayUtil.sendRelaySignal(isChecked);

```



### 4 二维码


```

/*
    启动二维码扫描服务
*/
QrCodeUtil.getInstance().init();

/*
    检测设备是否支持二维码功能；
    ret:true 表示支持、false 表示不支持
*/
booean ret = QrCodeUtil.getInstance().isQrCodeSupport();

/*
    添加二维码监听回调
*/
QrCodeUtil.getInstance().setQRCodeCallback(new QrCodeUtil.QRCodeCallback() {
    //监听服务为异步启动,启动后若发现存在二维码设备,则回调onConnect
    @Override
    public void onConnect() {
        Log.i("firefly","QRCode onConnect:");
    }

     //二维码识别的内容回调方法
    @Override
    public void onData(final String s) {
        Log.i("firefly","QRCode onData:"+s);
    }
});

/*
    二维码扫描补光灯操作
    QrCodeUtil.LED_STATE_AUTO //扫码时自动打开
    QrCodeUtil.LED_STATE_ON //常亮
    QrCodeUtil.LED_STATE_OFF //常灭
*/
QrCodeUtil.getInstance().setLedState(int state);

/*
   设置对焦灯状态
    QrCodeUtil.LED_STATE_AUTO //扫码时自动打开
    QrCodeUtil.LED_STATE_ON //常亮
    QrCodeUtil.LED_STATE_OFF //常灭
*/
QrCodeUtil.getInstance().setFocusLedState(int state);

/*
    设置二维码工作状态
    active 为false时不会回调数据而且会自动关闭照明灯
*/
QrCodeUtil.getInstance().setActive(boolean active);

/*
    页面退出时，关闭二维码扫描
*/
QrCodeUtil.getInstance().release();

```

### 5. 体温检测

> 提供了温度校正接口，测温距离为60cm-1m ；
>


```

/*
    启动体温检测服务
*/
TempatureUtil.getInstance().openDevice();

/*
    检测设备是否支持体温；
    Boolean:true 表示支持、false 表示不支持
*/
TempatureUtil.getInstance().isSupport();

/*
    添加体温检测监听回调
*/
TempatureUtil.getInstance().setTempatureCallback(new TempatureUtil.TempatureCallback() {
     //监听服务为异步启动,启动后若发现存在测温设备,则回调onConnect
    @Override
    public void onConnect() {
        Log.i("firefly","TempatureCallback onConnect:");
    }

     //体温检测的内容回调方法
    @Override
    public void update(float ambientTempature/*环境温度*/, float objectTempature/*人体温度*/) {
        Log.i("firefly","TempatureCallback update:ambientTempature="+ambientTempature + "objectTempature="+objectTempature);
        
        Log.i("firefly","体温校正后 TempatureCallback update:ambientTempature="+ambientTempature + "objectTempature="+TempatureUtil.correctTemp(objectTempature));
    }
});

/*
   温度校正函数，对体温进行一个简单的校正
   float: objectTempature 人体温度
*/
TempatureUtil.correctTemp(float objectTempature) {
    Log.i("firefly","TempatureCallback onConnect:");
}

/*
    页面退出时，关闭测温功能
*/
TempatureUtil.getInstance().closeDevice();

```


### 6. 雷达

> 监听雷达信号，雷达信号以KeyEvent的形式触发,使用handleEvent(KeyEvent event)处理；
> 
>返回值ret说明:
> 
> RadarUtil.EVENT_HANDLE_NOTHING_UNHANDLE  //非雷达事件未处理
> RadarUtil.EVENT_HANDLE_NOTHING_HANDLED   //非雷达事件已处理
> RadarUtil.EVENT_HANDLE_RADAR_IN  //有物体进入
> RadarUtil.EVENT_HANDLE_RADAR_OUT; //有物体离开


```

@Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        int ret = RadarUtil.handleEvent(event);
        if (ret == RadarUtil.EVENT_HANDLE_RADAR_IN) {//物体进入
            Log.i("firefly","EVENT_HANDLE_RADAR_IN");
            return true;
        } else if (ret == RadarUtil.EVENT_HANDLE_RADAR_OUT) { //物体离开
            Log.i("firefly","EVENT_HANDLE_RADAR_OUT");
            return true;
        } else if (ret == RadarUtil.EVENT_HANDLE_NOTHING_HANDLED) { //无物体
            Log.i("firefly","EVENT_HANDLE_NOTHING_HANDLED");
            return true;
        }

        return super.dispatchKeyEvent(event);
    }

```



## Android 快速入门

<h4>1.添加依赖库</h4>
```
1.将 Demo工程 faceEngineYtlf/libs/ 目录下的 aar包拷贝到 AS 工程对应的 libs 文件夹下；
2.将 Demo工程 faceEngineYtlf/src/main/assets 目录下 ytlf_v2 文件夹，拷贝到 AS 工程对应的 src/main/assets文件夹下；

```
<h4>2.添加编译 aar 信息</h4>
```
implementation(name: 'arctern-release', ext: 'aar')
implementation(name: 'iface-release', ext: 'aar')
implementation(name: 'faceEngineYtlfExternal', ext: 'aar')
    
到 build.gradle 的 dependencies

详细请参照技术案例Demo工程配置和 doc/FaceEngine集成说明截图.png；

```
<h4>3.设置权限</h4>
```
在AndroidManifest.xml文件中添加如下权限

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-feature android:name="android.hardware.camera" android:required="true"/>
<uses-permission android:name="android.permission.CAMERA"/>
```
<h4>4.在程序中初始化YTLFFaceManager</h4>
>实例YTLFFaceManager时，需要指明本地SD卡文件存放目录rootPath，例如："/sdcard/firefly/"；
>首次启动SDK时，会检查本地SD卡是否存在models 和 license公钥等文件，如果没有，那么默认会从App assets 目录下拷贝必需文件到rootPath目录；
```
YTLFFaceManager.getInstance().initPath(String rootPath);
```
<h4>5.通过 YTLFFaceManager 调用 Face API 接口</h4>



## 人脸识别SDK

V2.0版采用线程池的异步处理方式，提高系统的利用率，减少人脸检测、人脸跟踪等服务的响应时间；

本SDK开发指南指导您如何安装和配置开发环境,如何通过调用 SDK 提供的接口函数(API)进行二次开发与系统集成。
用户按照要求调用SDK提供的API即可实现使用 人脸检测、人脸跟踪、活体判断、人脸识别等服务的目的。

   

   ### 1. SDK接入

   1.1 SDK初始化

实例YTLFFaceManager时，需要指明本地SD卡文件存放目录rootPath，例如："/sdcard/firefly/"；
首次启动SDK时，会检查本地SD卡是否存在models 和 license公钥等文件，如果没有，那么默认会从App assets 目录下拷贝必需文件到rootPath目录；

   ```
   
   // 指定 本地SD卡文件存放目录
   YTLFFaceManager.getInstance().initPath(String rootPath);
   
   ```

   1.2 激活 License

初始化时会检查本地是否存在密钥,若无则进行在线激活，激活结束后会回调激活状态，分为同步和异步两种激活方式 ；

   ```
   
   // 异步方式
   YTLFFaceManager.getInstance().initLicenseByAsync(String apiKey, new InitLicenseListener(){
                   @Override
                   public void onLicenseSuccess() {
                               toast("激活成功");
                   }
   
                   @Override
                   public void onLicenseFail(final int code,final String msg) {
                               toast("激活失败");
                   }
               });
   
   
   // 同步方式 true 表示激活成功
   boolean result = YTLFFaceManager.getInstance().initLicense(String apiKey);
   
   ```

   1.3 获取设备唯一编码

获取当前设备的 signature ，设备的唯一编码；

   ```
   
   /*
      String :返回当前设备的 signature 即设备当前的唯一编码
   */
   String signature = YTLFFaceManager.getInstance().getSignature();
   
   ```

   

   ### 2. SDK 启动

启动SDK时，会检测运行环境和初始化SDK，有同步和异步两种启动方式；


   2.1 同步方式启动SDK
   ```
   
   /*
       通过FACE_PATH 判断出 config_path，即Config.json 文件存放目录
       eg:  sdcard/firefly/ytlf_v2/config.json"
   
      Int:0 表示成功、1 表示失败
   */
   int result = YTLFFaceManager.getInstance().startFaceSDK();
   
   /*
        config_json 指明config.json 内容
   
      Int:0 表示成功、1 表示失败
   */
   int result = YTLFFaceManager.getInstance().startFaceSDKByConfigJson(String config_json);
   
   ```
   2.2 异步方式启动SDK
为防止初始化并启动SDK时占用过多时间，造成阻塞主线程，可以使用异步方式启动SDK；

   ```
   
    /*
        config_json 指明config.json 内容
        runSDKCallback 异步启动后的回调方法
   */
    YTLFFaceManager.getInstance().startFaceSDKByAsynchronous(String config_json, new RunSDKCallback(){
       @Override
       public void onRunSDKListener(int i) {   //Int:0 表示成功、1 表示失败
   
       }
   });
   
   ```

   

   ### 3. 人脸实时数据送检

根据传入检测器的数据进行人脸检测与跟踪,并实时返回人脸检测的结果以及人脸跟踪
的结果,注意:输入人脸检测器的数据人脸方向应为正向,即人脸角度应为 0 度而非其他的
角度；
目前通过输送 RGB 以及 IR 视频流来进行人脸检测跟踪以及活体的相关检测
当不需要打开 IR 摄像头或不需要活体检测时 IR 数据参数可传 RGB 参数,不可传
NULL；

   ```
   //从摄像头回调函数中获取视频流，实时输  RGB 视频流以及 IR 视频流
    YTLFFaceManager.getInstance().doDelivery(ArcternImage img_rgb,  ArcternImage img_ir)
       
   //设置RGB和IR监听回调函数
    YTLFFaceManager.getInstance().setOnDetectCallBack(new DetectCallBack() {
           //  RGB 回调:
           @Override
           public void onDetectListener(ArcternImage arcternImage, ArcternRect[] arcternRects, float[] confidences) {
           
           }
   
       //  IR 回调:
       @Override
       public void onLivingDetectListener(ArcternImage arcternImage, ArcternRect[] arcternRects, float[] confidences) {
   
       }
   });
   
   /*参数说明：
       arcternImage RGB/IR 检测的人脸图数据
       arcternRects  RGB/IR 检测到的人脸框集合
       confidences RGB/IR 检测到每个人脸的置信度
   */
   
   ```

   ### 4. 人脸实时跟踪回调

如果回调接口有数据返回，那么说明实时数据正在检测跟踪；


   ```
   //设置人脸实时检测跟踪回调
   YTLFFaceManager.getInstance().setOnTrackCallBack(new TrackCallBack() {
   
   /*人脸实时检测跟踪
       arcternImage 检测的人脸图数据
       trackIds  图像中人脸的跟踪 ID
       arcternRects 检测到的人脸位置
   */
       @Override
       public void onTrackListener(ArcternImage arcternImage, long[] trackIds, ArcternRect[] arcternRects) {
   
       }
   });
   ```
   ### 5. 实时人脸属性检测

进行实时人脸检测跟踪时，添加人脸属性检测回调方法，可以实时接收人脸属性，包括活体检测值、质量、人脸角度、口罩、图像颜色。


   ```
   //设置实时人脸属性检测回调
   YTLFFaceManager.getInstance().setOnAttributeCallBack(new AttributeCallBack() {
   
   /*人脸属监听回调：
       arcternImage 检测的人脸图数据
       arcternRects 检测到的人脸位置
       trackIds  图像中人脸的跟踪 ID
       arcternAttribute 图像中所有人脸的所有属性
   */
       @Override
       public void onAttributeListener(ArcternImage arcternImage, long[] trackIds, ArcternRect[] arcternRects, ArcternAttribute[][] arcternAttributes, int[] landmarks) {
           StringBuilder s = new StringBuilder();
           for (int i = 0; i < arcternRects.length; i++) {
               for (int j = 0; j < arcternAttributes[i].length; j++) {
                   ArcternAttribute attr = arcternAttributes[i][j];
                   switch (j) {
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.POSE_PITCH:
                           s.append("人脸角度:\n以x轴为中心,脸部上下俯仰角度:").append(attr.confidence);
                           break;
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.POSE_YAW:
                           s.append("\n以y轴为中心,脸部左右旋转角度:").append(attr.confidence);
                           break;
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.POSE_ROLL:
                           s.append("\n以中心点为中心，x-y平面旋转角度:").append(attr.confidence);
                           break;
                           
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.QUALITY:
                           s.append("\n人脸质量:").append(attr.confidence);
                           break;
                           
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.LIVENESS_IR:
                           if (attr.label != -1) {
                               if (attr.confidence >= 0.5) {
                                   s.append("\n活体检测:活体 ").append(attr.confidence);
                               } else {
                                   s.append("\n活体检测:非活体 ").append(attr.confidence);
                               }
                           }
                           break;
                           
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.IMAGE_COLOR:
                           if (attr.label == ArcternAttribute.LabelFaceImgColor.COLOR) {
                               s.append("\n彩图 ").append(attr.confidence);
                           } else {
                               s.append("\n黑白图 ").append(attr.confidence);
                           }
                           break;
                           
                       case ArcternAttribute.ArcternFaceAttrTypeEnum.FACE_MASK:
                           if (attr.label == ArcternAttribute.LabelFaceMask.MASK) {
                               s.append("\n口罩 ").append(attr.confidence);
                           } else {
                               s.append("\n未带口罩 ").append(attr.confidence);
                           }
                           break;
                   }
               }
           }
       }
   });
   
   ```

   ### 6. 人脸实时搜索

根据传入检测器的数据进行实时搜索，从数据库中搜索相似度大于设置相似度最高的一个 ID,通过 ID 可获取到识别的人脸以及其相关信息；


   ```
   //设置人脸实时搜索回调
   YTLFFaceManager.getInstance().setOnSearchCallBack(new SearchCallBack() {
   
   
   /*搜索回调：
       arcternImage 检测的人脸图数据
       trackIds  图像中人脸的跟踪 ID
       arcternRects 检测到的人脸位置
       searchId_list 图像中识别人脸的 id 集合
   */
       @Override
   public void onSearchListener(ArcternImage arcternImage, long[] trackId_list, ArcternRect[] arcternRects, long[] searchId_list, int[] landmarks, float[] socre) {
       if (searchId_list.length > 0 && searchId_list[0] != -1) {
           Person person = DBHelper.get(searchId_list[0]);
           if (person != null) {
               //通过 ID 可获取到识别的人脸以及其相关信息;
           }
       } else {
               // 人脸不存在;
       }
   }
   ```


   ### 7. 指定图片文件提取人脸特征值

根据检测的人脸提取人脸特征值，可以用于人脸比对；


   ```
   
   /*指定图片文件提取人脸特征值：
       imagePath 图片地址
       etractCallBack  人脸特征提取回调
   */
   YTLFFaceManager.getInstance().doFeature(imagePath, new ExtractCallBack() {
   
   /*
       acternImage 提取特征值的图片
       bytes  多个人脸的人脸特征值集合
       arcternRects  人脸检测结果集合
   */
       @Override
       public void onExtractFeatureListener(ArcternImage arcternImage, byte[][] bytes, ArcternRect[] arcternRects) {
       
           if (features.length > 0) {
               debugLog("bitmapFeature.length： " + features[0].length);
           } else {
               Tools.debugLog("特征值为空！！！");
           }
       });
   
   ```


   ### 8. 图片Bitmap 进行特征值提取

根据人脸图片Bitmap 提取人脸特征值，可以用于人脸比对；


   ```
   
   /*指定人脸图片Bitmap提取人脸特征值：
       bitmap 图片Bitmap数据
       etractCallBack  人脸特征提取回调
   */
   YTLFFaceManager.getInstance().doFeature(Bitmap bitmap, new ExtractCallBack())
   
   /*
       acternImage 提取特征值的图片
       bytes  多个人脸的人脸特征值集合
       arcternRects  人脸检测结果集合
   */
       @Override
       public void onExtractFeatureListener(ArcternImage arcternImage, byte[][] bytes, ArcternRect[] arcternRects) {
       
           if (features.length > 0) {
               debugLog("bitmapFeature.length： " + features[0].length);
           } else {
               Tools.debugLog("特征值为空！！！");
           }
       });
   
   ```


   ### 9. 获取人脸的眼睛、嘴巴、鼻子等等landmarks坐标

根据人脸ArcternImage提取人脸特征值，可以用于获取人脸的眼睛、嘴巴、鼻子等等landmarks坐标；


   ```
   
   a/*
       指定ArcternImage提取人脸特征值，包含landmarks
   */
   ArcternAttrResult ArcternAttrResult = YTLFFaceManager.getInstance().doFeature(ArcternImage arcternImage))
   
   ```


   ### 10. 人脸特征比对

通过两个特征值进行比对,得出两个人脸特征值的相似度；


   ```
   
   /*
       feature1 第一个人脸特征值
       Feature2  第二个人脸特征值
       
       return float :返回人脸比对相似度
   */
   float result = YTLFFaceManager.getInstance().doFeature(byte[] feature1,  byte[] feature2)
   
   ```

   

   ### 11. 人脸库管理


   11.1 人脸入库

将提取出来的人脸特征值和相关 Id，添加到SDK人脸数据库；


   ```
   
   /*
       id 人脸 ID
       feature  人脸的特征值
       
       Int:0 表示执行成功、1 表示执行失败
   */
   int result = YTLFFaceManager.getInstance().dataBaseAdd(long id, byte[] feature)
   
   ```

   11.2 人脸库删除

根据指定的 Id，删除SDK人脸数据库的人脸信息；


   ```
   
   /*
       id 人脸 ID
    
       Int:0 表示执行成功、1 表示执行失败
   */
   int result = YTLFFaceManager.getInstance().dataBaseDelete(long id)
   
   ```

   11.3 人脸库更新

根据指定的 Id，更新SDK人脸数据库的人脸信息；


   ```
   
   /*
       id 人脸 ID
       feature  人脸的特征值
    
       Int:0 表示执行成功、1 表示执行失败
   */
   int result = YTLFFaceManager.getInstance().dataBaseUpdate(long id, byte[] feature)
   
   ```

   11.4 人脸批量添加特征值

根据指定的多个 Id，批量更新SDK人脸数据库对应的多个人脸信息；


   ```
   
   /*
       id 人脸 ID 数组
       feature  多个人脸特征值数组
    
       Int:0 表示执行成功、1 表示执行失败
   */
   int result = YTLFFaceManager.getInstance().dataBaseAdd(long[] id, byte[][] feature)
   
   ```

   11.5 人脸数据清除

删除SDK人脸数据库的所有人脸信息；


   ```
   
   /*
       Int:0 表示执行成功、1 表示执行失败
   */
   int result = YTLFFaceManager.getInstance().dataBaseClear()
   
   ```