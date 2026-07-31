# 一、产品介绍
## 产品简介
### 智能语音开发套件

一款专为高效开发设计的智能语音开发套件，集高性能、低功耗、跨平台特性于一体。套件集成 USB 声卡与 4/6 麦阵列，搭载 CAE 降噪与 AEC 回声消除技术。内置功放直驱双喇叭。配备 AI SDK 与 AIUI 云服务，低代码即可快速集成，缩短项目周期。麦克风灵敏度 -32dBA、信噪比 65dB，3-5 米稳定唤醒识别，宽温低功耗设计适配多场景，广泛适用于机器人、行业开发板、智能服务终端、魔镜、智能家居面板、商显设备等产品和领域。

![](../../../modules_img/Intelligent-Voice-Control-Kit/intelligent-voice-control-kit.png)

## 详细参数
### 麦克风阵列板

| 麦克风类型 | 线形4麦克风阵列 | 环形6麦克风阵列 |
| ---- | ---- | ---- |
| 模块阵型 | 长条形 | 环形 |
| 阵元数量 | 支持4个ECM驻极体MIC（推荐6027规格 | 支持板载6个MEMS模拟硅MIC |
| MIC信噪比 | SNR > 70dB | SNR > 65dB |
| MIC灵敏度 | -32dBA，一致性误差≤2dB | -32dBA，一致性误差≤1dB |
| 唤醒距离 | 3 ~ 5m | 3 ~ 5m |
| 识别距离 | 3 ~ 5m | 3 ~ 5m |
| 声源定位 | 水平180度 | 环360度 |
| 定位精度 | ±15° | ±15° |
| 其它特性 | 支持AEC功能、60°拾音波束（支持3波束） | 支持AEC功能、60°拾音波束（支持6波束） |
| 通讯接口 | 14Pin-0.5mm FPC座 | 14Pin-0.5mm FPC座 |
| 尺寸 | 111.5mm × 12mm，限高8mm | 直径79.5mm，限高8mm，安装孔孔径3mm |
| 工作温度 | -10°C ~ 70°C | -10°C ~ 70°C |
| 存储温度 | -40°C ~ 85°C | -40°C ~ 85°C |

<br>

### USB声卡

| 模组 | 参数 |
| ---- | ---- |
| 接口 | 1 × 12V功放供电&UAC（6Pin-2mm）、1 × Type-C（5V供电&音频传输）、1 × 4&6麦板接口（14Pin-0.5mm FPC座）、1 × 3.5mm音频输入接口、1 × AEC接口（5Pin-1.25mm）、1 × 左声道喇叭（2Pin-2mm）、1 × 右声道喇叭（2Pin-2mm） |
| 尺寸 | 88mm × 40mm，限高8mm |

# 二、硬件使用

<!--
## 使用说明
-->

## 硬件连接

![](img/connection.png)

### 拨码开关

![](img/toggle_switch.png)

# 三、安卓开发
量产授权SN码和安卓平台SDK请联系商务（sales@t-firefly.com）获取
## 1. 降噪算法授权
每台设备需要使用唯一的SN码进行鉴权，demo中自带的SN号，配置在app模块的 assets/auth.ini 。目前在用的降噪算法引擎授权api是：
```java
/**
** 授权api
** sn：授权码，由我司提供，每个设备必须使用唯一的sn号。
**/
public static native int CAEAuth(String sn);
```
本地授权：将授权码烧录到设备指定的目录下，然后通过代码读取该授权码再授权。用户需要管理好"设备唯一码"和"sn授权码"，以防设备刷机导致授权码丢失。

正式量产时，需要批量采购正式版的SN授权码，然后再使用。算法库每次初始化的时候，都会对SN号的有效性进行验证，因此初始化的时候，设备是需要联网的，同时要求系统时间是正确的。初始化成功后，就可以一直使用了，不管中途是否断网。


## 2. HLW降噪算法开发

### 2.1 降噪算法的处理框架
![](img/cae_algorithm_framework.png)

<br>

音频格式转换：不同mic类型的引擎，对音频数据的格式要求不一样，具体的格式如下：

| mic类型 | 通道 |
| ---- | ---- |
| 4麦 | 4mic + 2ref，共6个通道，每个通道32bit |
| 6麦 | 6mic + 2ref，共8个通道，每个通道32bit |

<br>

### 2.2 引擎相关的文件说明

| 文件 | 说明 |
| ---- | ---- |
| cae.jar | 引擎的jar包 |
| libhlw.so | 引擎so库，不同mic类型需使用各自的引擎 |
| libcae-jni.so | 引擎jni层封装库 |
| hlw.ini | 外部配置文件 |
| res_cae_model.bin | 引擎模型文件，不同mic类型需使用各自的文件 |
| hlw.param | 客户标识文件，包括appid，鉴权配置信息 |
| res_ivw_model.bin | 唤醒词文件 |

<br>

### 2.3 相关api说明
```java
public class CAE {
    public static final int CAE_ERROR_UNINIT = 4;
    public static final int CAE_FAIL = -1;
    public static final int CAE_SUCCESS = 0;
    private static boolean LIB_LOADED;

    public CAE() {
    }

    /**
    **  写入数据到引擎
    **  data：音频数据【转换后的音频数据】
    **  length: 音频的数据长度
    */
    public static native int CAEAudioWrite(byte[] data, int length);

    /**
    ** 销毁引擎
    **/
    public static native void CAEDestory();

    /**
    ** 获取引擎的版本号
    */
    public static native String CAEGetVersion();

    /**
    ** 引擎初始化
    ** iniPath ： 传入hlw.ini文件的实际路径
    ** paramPath ：传入hlw.param文件的实际路径
    ** listener: 引擎的监听器
    ** 
    ** return: 返回初始化code值： 0：初始化成功
                            其他值：初始化失败
    */
    public static native int CAENew(String iniPath, String paramPath, ICAEListener listener);
    
    
    /**
    ** 鉴权
    ** sn： 授权码
    **/
    public static native int CAEAuth(String sn);
    
    /**
    ** 设置波束
    ** 
    */
    public static native int CAESetRealBeam(int beam);
	
    /**
    ** 设置调试log
    ** level:取值【1-5】，数值越大，等级越高，显示的调试信息越少
    */
    public static native int CAESetShowLog(int level);

    /**
    ** 加载so库，
    */
    public static void loadLib() {
        if (!LIB_LOADED) {
            try {
                System.loadLibrary("cae-jni");
                LIB_LOADED = true;
            } catch (Exception var1) {
                LIB_LOADED = false;
            }
        }

    }
}
```
```java
/**
** 引擎回调接口
**/
public interface ICAEListener {
    /**
    ** 引擎处理后的音频【图一中的“音频抽取”】，格式为：16K，16bit，单通道
    ** data: 音频数据
    ** lenght:数据长度
    **
    ** 注意：引擎首次运行时，这个接口是不会被回调的，有两种方式可以让它回调：
    ** 1.通过唤醒词唤醒后，这个接口就会持续回调 
    ** 2.通过调用 CAESetRealBeam() 也可以让它回调 
    **/
    void onAudioCallback(byte[] data, int lenght);

    // 此接口可以忽略
    void onIvwAudioCallback(byte[] var1, int var2);
	
    /**
    **  语音唤醒回调
    **  power: 唤醒的能量值
    **	angle：唤醒角度
    **	beam： 波束值
    **  jsonParam：唤醒相关的信息，json格式，里面包含角度，波束相关信息
    **/
    void onWakeup(float power, int angle, int beam, String jsonParam);
}
```

<br>

### 2.4 算法api调用流程

![](img/api_process.png)

### 2.5 接口封装示例
```java
package com.voice.bothlent.caeLib.engine;

import android.content.Context;
import android.util.Log;

import com.iflytek.iflyos.cae.CAE;
import com.voice.bothlent.caeLib.bean.CheckResultBean;
import com.voice.bothlent.caeLib.bean.WakeInfo;
import com.voice.bothlent.caeLib.utils.FileUtils;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Locale;
import java.util.Map;

/**
 * 近距离hlw引擎【1.5m左右】
 */
public class HlwNearEngine extends BaseEngine {
    private static final String configCAE_RES = "CAE_RES";
    private static final String configIVW_RES = "IVW_RES";
    private static final String configINI_RES = "INI_RES";
    private static final String configPARAM_RES = "PARAM_RES";
    private static final String configCAE_DIST = "CAE_DIST";
    private static final String iniHlwName = "hlw.ini";
    private static final String configMicNum = "nMicNum";
    private static final String assetParentPath = "hlw_near/";
    private static final String TAG = "HlwNearEngine";

    static {
        CAE.loadLib("hlw");
    }


    @Override
    protected WakeInfo parseWakeInfo(String s) {
        // {"cur_ms":288768, "start_ms":287290, "end_ms":288310, "beam":1, "physical":1, "similar":1498.000000, "similar_thresh":900.000000, "power":1210436681728.000000, "angle":75.000000, "keyword":"ni3 hao3 hua1 sheng1"}
        JSONObject jsonObject = null;
        WakeInfo wakeInfo = new WakeInfo();
        try {
            jsonObject = new JSONObject(s);
            int beam = jsonObject.optInt("physical");
            int angle = jsonObject.optInt("angle");
            int similar = jsonObject.optInt("similar");
            wakeInfo.setBeam(beam);
            wakeInfo.setAngle(angle);
            wakeInfo.setScore(similar);
        } catch (JSONException e) {
            e.printStackTrace();
            wakeInfo.setAngle(0);
            wakeInfo.setBeam(0);
        }
        return wakeInfo;
    }

    @Override
    public CheckResultBean init(Context context, String sn) {
        //res check
        CheckResultBean checkResult = new CheckResultBean();
        List<String> keyList = Arrays.asList(configCAE_RES, configIVW_RES,configINI_RES,configPARAM_RES,configCAE_DIST);
        Map<String, String> stringStringMap = FileUtils.readAssetValueByKey(context,
                assetParentPath + iniHlwName, keyList);
        if (stringStringMap != null && stringStringMap.size() == keyList.size()) {
            try {
                configParentPath = context.getFilesDir().getAbsolutePath();

                String cae_res_path = stringStringMap.get(configCAE_RES);
                checkResource(context, configParentPath+cae_res_path, assetParentPath);
                String ivw_res_path = stringStringMap.get(configIVW_RES);
                checkResource(context, configParentPath+ivw_res_path, assetParentPath);

                String ini_res_path = stringStringMap.get(configINI_RES);
                checkResource(context, configParentPath+ini_res_path, assetParentPath);
                String param_res_path = stringStringMap.get(configPARAM_RES);
                checkResource(context, configParentPath+param_res_path, assetParentPath);
                String cae_dist = stringStringMap.get(configCAE_DIST);

                File fileIni = null ;
                FileOutputStream fileOutputStream =null ;
                try {
                    fileIni = new File(configParentPath + new File(ini_res_path).getParent(), "hlw_real.ini");
                    fileIni.createNewFile();
                    fileOutputStream = new FileOutputStream(fileIni);
                    fileOutputStream.write(String.format("CAE_RES = %s\n",configParentPath+cae_res_path).getBytes());
                    fileOutputStream.write(String.format(Locale.getDefault(),"CAE_DIST = %d\n",Integer.parseInt(cae_dist)).getBytes());
                    fileOutputStream.write(String.format("IVW_RES = %s\n",configParentPath+ivw_res_path).getBytes());
                    fileOutputStream.close();
                }finally {
                    if (fileOutputStream != null){
                        fileOutputStream.close();
                    }
                }

                // step1 : 根据配置资源进行初始化
                int isInit = CAE.CAENew(fileIni.getAbsolutePath(), configParentPath+param_res_path, mCAEListener);
                if (isInit == 0) {
                    // step2 : 根据授权码进行鉴权，鉴权正确后，就可以使用
                    String version = CAE.CAEGetVersion();
                    Log.e(TAG, "init: " + sn + "   " + version);
                    int auth = CAE.CAEAuth(sn);
                    if (auth >= 0) {
                        checkResult.setState(1);
                        List<String> keyListMicNum = Collections.singletonList(configMicNum);
                        Map<String, String> MicNumMap = FileUtils.readAssetValueByKey(context,
                                assetParentPath + new File(cae_res_path).getName(), keyListMicNum);
                        checkResult.setCheckResult(String.format(Locale.getDefault(), "%s,%s,micNum=%s",  version,cae_res_path, MicNumMap.get(configMicNum)));
                    } else {
                        checkResult.setState(0);
                        checkResult.setCheckResult("cae auth error, code  = " + auth);
                    }
                } else {
                    checkResult.setState(0);
                    checkResult.setCheckResult("cae init error,please check res file " + (configParentPath+ini_res_path) + ";" + (configParentPath+param_res_path));
                }

            } catch (IOException ignored) {
                checkResult.setState(0);
                checkResult.setCheckResult("copy res file error " + ignored);
            }
        } else {
            checkResult.setState(0);
            checkResult.setCheckResult("hlw.ini config error");
        }
        return checkResult;
    }


    @Override
    public void setBeam(int beam) {
        int i = CAE.CAESetRealBeam(beam);
        Log.e(TAG, "setBeam: " + i + "   " + beam);
    }


    @Override
    public void writeAudio(byte[] audio, int len) {
        CAE.CAEAudioWrite(audio, len);
    }


    @Override
    public void setShowLog(boolean show) {
        CAE.CAESetShowLog(show ? 0 : 1);
    }

    @Override
    public void onDestroy() {
        CAE.CAEDestory();
    }
}
```