# RKMedia

## 模块介绍
`RKMedia` 是对 `RV1126/RV1109` 内所有媒体资源调用进行了整合封装的一套 `API` 接口，它可以大大降低刚接触 `RV1126/RV1109` 芯片平台用户的开发成本，以少量代码就可以很快速、简单地调用 Soc 上所有媒体资源。同时 `RKMedia` 还提供了媒体资源以外的硬件资源联合调用 DEMO，如：`RKAIQ、RKNN、RTSP` 等等。`RKMedia` 的核心思想是把各个硬件资源独立成模块，模块开放出输入和输出端通过绑定的方式控制流从某个模块流出并且流入另外一个模块。这里会对每个模块进行相关介绍。

![](../../../rv1126_img/CAM-C1126S2U/rkmedia.png)

### 视频
#### VI
* `VI` 为视频输入模块，可以理解为摄像头采集模块，需要注意的是 `VI` 需要 `RKAIQ` 的支持，在使用 `VI` 前需要提前初始化好 `RKAIQ`，详细初始化步骤可以在 Example 中查看。
* `VI` 通道属性设置
```
MPP_CHN_S stViChn;			#定义模块设备通道结构体
stViChn.enModId = RK_ID_VI;	#模块号为 RK_ID_VI
stViChn.s32ChnId				#设置 VI 通道号

VI_CHN_ATTR_S vi_chn_attr;	#定义 VI 通道属性结构体指针
vi_chn_attr.u32BufCnt			# VI 捕获视频缓冲区计数
vi_chn_attr.u32Width			# video 宽度
vi_chn_attr.u32Height			# video 高度
vi_chn_attr.enWorkMode		# VI 通道工作模式
vi_chn_attr.pcVideoNode		# video 节点路径
vi_chn_attr.enPixFmt			# video 格式
```

* 设置 `VI` 通道属性
```
#函数定义：
RK_MPI_VI_SetChnAttr(VI_PIPE ViPipe, VI_CHN ViChn, const VI_CHN_ATTR_S *pstChnAttr);
# ViPipe 为 VI 管道号；ViChn 为 VI 通道号； pstChnAttr 为 VI 通道属性结构体指针。
#使用示例：
RK_MPI_VI_SetChnAttr(vi_pipe, stViChn.s32ChnId, &vi_chn_attr);
```

* 启用 `VI` 通道
```
#函数定义：
RK_S32 RK_MPI_VI_EnableChn(VI_PIPE ViPipe, VI_CHN ViChn);
# ViPipe 为 VI 管道号； ViChn 为 VI 通道号。
#使用示例：
RK_MPI_VI_EnableChn(vi_pipe, stViChn.s32ChnId);
```
#### VP
* 视频一入四出 `VP` 使用 `ISPP` 模块实现一个视频从 rkispp_input_image 节点对应 video 输入，四个视频从 rkispp_m_bypass 、 rkispp_scale0 、 rkispp_scale1 、 rkispp_scale2 节点对应 video 输出，节点信息可以通过以下命令查看：
```
media-ctl -p -d /dev/media*
```
* 使用 VP 完成视频从 rkispp_input_image 节点对应 video 输入，使用 VI 完成视频从 rkispp_m_bypass 、 rkispp_scale0 、 rkispp_scale1 、 rkispp_scale2 节点对应 video 输出，即可实现视频一入四出。 rkispp_scale0 、 rkispp_scale1 、 rkispp_scale2 可以实现`缩放`，可以替换 RGA 的缩放功能，对 DVR 等相关产品可以缓解 RGA 硬件处理压力。
* 使用 VP 功能需要在 kernel 先配置 rkispp_vir2 节点（默认已配置），例如：
```
&rkispp_vir2 {
       status = "okay";
};
```
* 使用 VP 功能需要先配置对应的 media 节点，例如：
```
media-ctl -d /dev/media3 -l '"rkispp_input_image":0->"rkispp-subdev":0[1]'
media-ctl -d /dev/media3 --set-v4l2 '"rkisppsubdev":0[fmt:YUYV8_2X8/1920x1080]'
media-ctl -d /dev/media3 --set-v4l2 '"rkisppsubdev":2[fmt:YUYV8_2X8/1920x1080]'
```
说明：配置完成后需要先打开 4 个输出节点（可以打开 1 个到 4 个），最后打开输入节点。输入节点的数据可以来自其他 VI 、 RGA 等。具体示例可参照 SDK 中的 `sdk/external/rkmedia/examples/rkmedia_vi_vp_vo_test.c`。

#### VDEC
* `VDEC` 模块，即视频解码模块。本模块支持`多路实时解码`，且`每路解码独立`，支持 `H264/H265/MJPEG/JPEG` 解码。
* 解码通道属性设置
```
MPP_CHN_S stVdecChn;					#定义模块设备通道结构体
stVdecChn.enModId = RK_ID_VDEC;		#模块号为 RK_ID_VDEC 
stVdecChn.s32ChnId						#设置解码通道号

VDEC_CHN_ATTR_S stVdecAttr;			#定义解码通道属性结构体
stVdecAttr.enCodecType					#解码格式
stVdecAttr.enMode							#解码输入模式，支持帧/流
stVdecAttr.enDecodecMode				#解码模式，支持硬件或软件解码
```

* 创建 `VDEC` 解码通道
```
#函数定义：
RK_S32 RK_MPI_VDEC_CreateChn(VDEC_CHN VdChn, const VDEC_CHN_ATTR_S *pstAttr);
# VdChn 为解码通道号； pstAttr 为解码通道属性指针。
#使用示例：
RK_MPI_VDEC_CreateChn(stVdecChn.s32ChnId, &stVdecAttr);
```


#### RGA
* `RGA` 模块用于 `2D` 图像的`裁剪`、`格式转换`、`缩放`、`旋转`、`图片叠加`等。
* 使用 `RGA` 处理可以大大降低 CPU 的负担，同时加快图片的处理速度。（参考：1080p 图片格式转换。RGA 耗时：7ms，Opencv 耗时：50ms）
* `rkmedia` 中 `RGA` 通道仅支持`裁剪`、`格式转换`、`缩放`、`旋转`功能，`图片叠加`则需要单独调用librga.so库，参见`《Rockchip_Developer_Guide_Linux_RGA_CN.pdf》`
* `RGA` 通道属性设置
```
MPP_CHN_S stRgaChn;				#定义模块设备通道结构体
stRgaChn.enModId = RK_ID_RGA;		#模块号为 RK_ID_RGA
stRgaChn.s32ChnId					#设置解码通道号

RGA_ATTR_S stRgaAttr;				#定义 RGA 属性结构体
stRgaAttr.bEnBufPool					#使能缓冲池
stRgaAttr.u16BufPoolCnt				#缓冲池计数
stRgaAttr.u16Rotaion					#旋转   取值范围 0，90，180，270
stRgaAttr.stImgIn.u32X					# RGA 通道输入图片的X轴坐标
stRgaAttr.stImgIn.u32Y 				# RGA 通道输入图片的Y轴坐标
stRgaAttr.stImgIn.imgType				# RGA 通道输入图片格式
stRgaAttr.stImgIn.u32Width 			#输入图片宽度
stRgaAttr.stImgIn.u32Height 			#输入图片高度
stRgaAttr.stImgIn.u32HorStride 		#输入图片虚宽
stRgaAttr.stImgIn.u32VirStride 		#输入图片虚高
stRgaAttr.stImgOut.u32X 				# RGA 通道输出图片的 X 轴坐标
stRgaAttr.stImgOut.u32Y				# RGA 通道输出图片的 Y 轴坐标
stRgaAttr.stImgOut.imgType			# RGA 通道输出图片格式
stRgaAttr.stImgOut.u32Width			#输出图片宽度
stRgaAttr.stImgOut.u32Height			#输出图片高度
stRgaAttr.stImgOut.u32HorStride		#输出图片虚宽
stRgaAttr.stImgOut.u32VirStride		#输出图片虚高
```

* 创建 `RGA` 通道
```
#函数定义：
RK_S32 RK_MPI_RGA_CreateChn(RGA_CHN RgaChn, RGA_ATTR_S *pstRgaAttr);
# RgaChn 为 RGA 通道号； pstAttr 为 RGA 通道属性指针。
#使用示例：
RK_MPI_RGA_CreateChn(stRgaChn.s32ChnId, &stRgaAttr);
```

`stride` 宽度，通常与 `buffer_width` 相同。若 `u32VirWidth` 大于 `buffer` 宽度，则必须满足 `16` 对齐。
可使用 `CELING_2_POWER` 函数对变量进行 `16` 对齐设置。例如：
```
# 定义 CELING_2_POWER 宏
#define CELING_2_POWER(x,a)  (((x) + ((a)-1)) & (~((a) - 1)))
# 使用 CELING_2_POWER 宏
stRgaAttr.stImgOut.u32VirStride = CELING_2_POWER(u32Height,16);
```
#### VO
* `VO` 模块用于视频输出管理。`VO` 模块是对 `DRM/KMS` 的封装，支持`多 VOP` 以及`多图层显示`。
```
MPP_CHN_S VoChn;						#定义模块设备通道结构体
VO_CHN_ATTR_S stVoAttr = {0};			#定义视频输出属性结构体并初始化为 0
memset(&stVoAttr, 0, sizeof(stVoAttr));		

stVoAttr.pcDevNode = "/dev/dri/card0";	#视频 VO 输出设备节点设置为"/dev/dri/card0"
stVoAttr.emPlaneType						#视频输出图层类型
stVoAttr.enImgType						#输入图片格式
stVoAttr.u16Zpos							#输出图层 Z 轴高度
stVoAttr.u32Width							#视频输出宽度
stVoAttr.u32Height							#视频输出高度
stVoAttr.stImgRect.s32X					#视频输入图像区域的 X 轴坐标
stVoAttr.stImgRect.s32Y					#视频输入图像区域的 Y 轴坐标
stVoAttr.stImgRect.u32Width				#输入图像区域的宽度
stVoAttr.stImgRect.u32Height				#输入图像区域的高度
stVoAttr.stDispRect.s32X					#显示区域的起始 X 轴坐标
stVoAttr.stDispRect.s32Y					#显示区域的起始 Y 轴坐标
stVoAttr.stDispRect.u32Width				#显示区域的宽度
stVoAttr.stDispRect.u32Height				#显示区域的高度

VoChn.enModId = RK_ID_VO;				#模块号为 RK_ID_VO
VoChn.s32DevId							#模块设备号
VoChn.s32ChnId							#模块通道号
```

* 创建 `VO` 通道
```
#函数定义：
RK_S32 RK_MPI_VO_CreateChn(VO_CHN VoChn, const VO_CHN_ATTR_S *pstAttr);
# VoChn 为 VO 通道号；pstAttr 为 VO 通道属性指针。
#使用示例：
RK_MPI_VO_CreateChn(VoChn.s32ChnId, &stVoAttr);
```


#### VENC
* `VENC` 模块，即视频编码模块。本模块支持`多路实时编码`，且`每路编码独立`，编码协议和编码 profile 可以不同。支持视频编码同时，调度 Region 模块对编码图像内容进行叠加和遮挡。支持 `H264/H265/MJPEG/JPEG` 编码。
```
MPP_CHN_S stVenChn;							#定义模块设备通道结构体
stVenChn.enModId = RK_ID_VENC;					#模块号设置为 RK_ID_VENC
stVenChn.s32ChnId								#编码通道号
memset(&venc_chn_attr, 0, sizeof(venc_chn_attr));

VENC_CHN_ATTR_S venc_chn_attr;				#定义 VENC 通道属性结构体
venc_chn_attr.stVencAttr.enType					#编码格式
venc_chn_attr.stVencAttr.imageType				#编码图片格式
venc_chn_attr.stVencAttr.u32PicWidth				#图片宽度
venc_chn_attr.stVencAttr.u32PicHeight			#图片高度
venc_chn_attr.stVencAttr.u32VirWidth				#图片虚宽
venc_chn_attr.stVencAttr.u32VirHeight			#图片虚高
venc_chn_attr.stVencAttr.u32Profile				#用于H264编码器的 Profile IDC 值
```

`H264` 或 `H265` 编码的参数参照以下配置
```
venc_chn_attr.stRcAttr.enRcMode									#编码协议类型 根据具体使用编码格式配置
venc_chn_attr.stRcAttr.stH264Cbr.u32Gop = fps;					# I 帧间隔，取值范围：[1, 65536]
venc_chn_attr.stRcAttr.stH264Cbr.u32BitRate =               
    u32Width * u32Height * fps / 14;									#平均比特率，取值范围：[2000, 98000000]，单位：bps
venc_chn_attr.stRcAttr.stH264Cbr.fr32DstFrameRateDen = 1;		#目标帧率分子
venc_chn_attr.stRcAttr.stH264Cbr.fr32DstFrameRateNum = fps;	#目标帧率分母
venc_chn_attr.stRcAttr.stH264Cbr.u32SrcFrameRateDen = 1;		#数据源帧率分子
venc_chn_attr.stRcAttr.stH264Cbr.u32SrcFrameRateNum = fps;	#数据源帧率分母
```
* 创建 `VENC` 通道
```
#函数定义：
RK_S32 RK_MPI_VENC_CreateChn(VENC_CHN VeChn, VENC_CHN_ATTR_S *stVencChnAttr);
# VeChn 为编码通道号； stVencChnAttr 编码通道属性指针。
#使用示例：
RK_MPI_VENC_CreateChn(stVenChn.s32ChnId, &venc_chn_attr);
```



#### 通道绑定
* 设置好通道属性之后就可以对通道进行绑定了。
* 数据源和数据接收者绑定接口如下：
```
RK_S32 RK_MPI_SYS_Bind(const MPP_CHN_S *pstSrcChn,const MPP_CHN_S *pstDestChn);
#pstSrcChn 为源通道指针； pstDestChn 目的通道指针。
```
* 注：通道绑定需要注意源通道的输出与目的通道的输入的数据格式要对应。

#### 通道数据发送与获取
* 常用向指定通道输入数据函数如下：
```
RK_S32 RK_MPI_SYS_SendMediaBuffer(MOD_ID_E enModID, RK_S32 s32ChnID, MEDIA_BUFFER buffer);
# enModID 为模块号； s32ChnID 为通道号； buffer 为缓冲区。
```
* 常用向指定通道获取数据函数如下：
```
MEDIA_BUFFER RK_MPI_SYS_GetMediaBuffer(MOD_ID_E enModID, RK_S32 s32ChnID, RK_S32 s32MilliSec);
# enModID 为模块号； s32ChnID 为通道号； s32MilliSec 为阻塞等待时间，当该值为 -1 的时候表示阻塞等待。
```

#### RTSP 拉流与推流
为了配合网络摄像头视频分析场景，我们提供了简单的 RTSP 拉流和推流接口。

* `RTSP` 拉流
使用 `firefly` 编写的库函数接口 `ffrtspGet` 可轻松实现拉取 `RTSP` 数据流。
需要配置结构体 `ffrtsp_get` ，其结构体配置介绍如下：
```
struct FFRTSPGet ffrtsp_get;					#定义 FFRTSPGet 类型结构体
ffrtsp_get.count = 0;
ffrtsp_get.callback = NULL;
ffrtsp_get.callback = FFRKMedia_Vdec_Send;	#回调函数
ffrtsp_get.count								#拉取的 RTSP 数据流的个数
ffrtsp_get.ffrtsp_get_info[i].url					#拉取的 RTSP 数据流的链接
```
注：配置好结构体之后只需要创建一个线程任务即可实现拉取单个或多个 `RTSP` 数据流。


* `RTSP` 推流
使用 `firefly` 编写的库函数接口 `ffrtspPush` 可轻松实现推送 `RTSP` 数据流。
需要配置结构体 `ffrtsp_push` ，其结构体配置介绍如下：
```
ffrtsp_push[i].idex										#推送 RTSP 数据流的编号
ffrtsp_push[i].port										#推送 RTSP 数据流的端口
ffrtsp_push[i].type = cfg.session_cfg[i].video_type;	#推送 RTSP 数据流的视频类型
```
注：配置好结构体之后需要创建的线程个数要匹配推送的 `RTSP` 数据流的个数。

#### 通道析构顺序

需要特别注意的是 rkmedia 对模块的析构顺序有特殊的要求：数据流管道中后级模块要先于前级模块销毁。比如：
```
VI --> RGA --> VENC
```
则建议析构顺序如下：
```
destroy VENC --> destroy RGA --> destroy VI
```
以 `VI` 为例， `VI` 是数据产生端。其生产的 `buffer` 在数据管道销毁时可能被后级占用，从而导致 `VI` 管理的资源也被占用。再次打开就会遇到 `Device Busy` 的错误。这个问题在频繁创建销毁数据通道时有概率发生。
* 详细 `RKMEDIA` 使用请参考手册`《Rockchip_Developer_Guide_Linux_RKMedia_CN.pdf》`。

### 音频

音频输入 `AI` 输出 `AO` ，用于和 Audio Codec 对接，完成声音的录制和播放。RKMedia AI/AO 依赖于 Linux ALSA 设备，不同的声卡，只要支持 ALSA 驱动，就可以使用 AI/AO 接口。 AI 中集成了音频算法，可通过配置开启。开启算法后， AI 输出经过算法处理后的 PCM 数据。

* 音频编解码是通过对 rkaudio 的封装实现，目前支持 `G711A/G711U/G726/MP2`。
* 算法支持：目前支持对讲场景 `AEC` 算法，录音场景 `ANR` 算法。

#### AI
* 音频输入
```
AI_CHN_ATTR_S ai_attr; 				#音频输入属性结构体
ai_attr.pcAudioNode					#音频设备节点路径
ai_attr.enSampleFormat				#采样格式
ai_attr.u32NbSamples					#每帧的采样点数
ai_attr.u32SampleRate				#采样率
ai_attr.u32Channels					#通道数
ai_attr.enAiLayout						#输入布局类型

RK_S32 s32CurrentVolmue = 0;
RK_S32 s32Volmue = 50;
RK_MPI_AI_SetChnAttr(0, &ai_attr); 				#设置 AI_CHN_ATTR_S 参数
RK_MPI_AI_EnableChn(0);							#使能通道 0 
RK_MPI_AI_GetVolume(0, &s32CurrentVolmue); 	#获取通道 0 设备的输入音量
RK_MPI_AI_SetVolume(0, s32Volume); 			#设置通道 0 设备输入音量
RK_MPI_AI_StartStream(0); 						#启动通道 0 的音频流
```

#### AO
* 音频输出
```
AO_CHN_ATTR_S ao_attr; 			#音频输出属性结构体
ao_attr.pcAudioNode 				#音频设备节点路径
ao_attr.enSampleFormat 			#采样格式
ao_attr.u32NbSamples			#每帧的采样点数
ao_attr.u32SampleRate 			#采样率
ao_attr.u32Channels 				#通道数

RK_S32 s32CurrentVolmue = 0;
RK_S32 s32Volmue = 50;
RK_MPI_AO_SetChnAttr(0, &ao_attr); 				#设置 AO_CHN_ATTR_S 参数
RK_MPI_AO_EnableChn(0);						#使能通道 0 
RK_MPI_AO_GetVolume(0, &s32CurrentVolmue); 	#获取通道 0 设备的输出音量
RK_MPI_AO_SetVolume(0, s32Volume); 			#设置通道 0 设备输出音量
```
#### AENC
* 音频编码
* 相关编码协议属性结构体： `stAencMP3/stAencMP2/stAencG711A/stAencG711U/stAencG726`
```
#音频编码属性结构体：
typedef struct rkAENC_CHN_ATTR_S {
	CODEC_TYPE_E enCodecType;
	RK_U32 u32Bitrate;
	RK_U32 u32Quality;
	union {
		AENC_ATTR_MP3_S stAencMP3;
		AENC_ATTR_MP2_S stAencMP2;
		AENC_ATTR_G711A_S stAencG711A;
		AENC_ATTR_G711U_S stAencG711U;
		AENC_ATTR_G726_S stAencG726;
	};
} AENC_CHN_ATTR_S;

MPP_CHN_S mpp_chn_aenc; 									#定义模块设备通道结构体
mpp_chn_aenc.enModId = RK_ID_AENC; 						#模块号为 RK_ID_AENC
mpp_chn_aenc.s32ChnId = 0; 									#通道号为 0 

AENC_CHN_ATTR_S aenc_attr; 										#定义音频编码属性结构体
aenc_attr.enCodecType 												#编码协议类型
aenc_attr.u32Bitrate  													#比特率
aenc_attr.u32Quality  													#编码质量
aenc_attr.stAencMP3.u32Channels 									#通道数
aenc_attr.stAencMP3.u32SampleRate 									#采样率
RK_MPI_AENC_CreateChn(mpp_chn_aenc.s32ChnId, &aenc_attr); 	#创建音频编码通道
```

#### ADEC
* 音频解码
* 相关解码协议属性结构体： `stAdecMP3/stAdecMP2/stAdecG711A/stAdecG711U/stAdecG726`
```
#音频解码属性结构体：
typedef struct rkADEC_CHN_ATTR_S {
	CODEC_TYPE_E enCodecType;
	union {
		ADEC_ATTR_MP3_S stAdecMP3;
		ADEC_ATTR_MP2_S stAdecMP2;
		ADEC_ATTR_G711A_S stAdecG711A;
		ADEC_ATTR_G711U_S stAdecG711U;
		ADEC_ATTR_G726_S stAdecG726;
	};
} ADEC_CHN_ATTR_S;

ADEC_CHN_ATTR_S stAdecAttr; 	#定义音频解码属性结构体
stAdecAttr.enCodecType			#编码协议类型
RK_MPI_ADEC_CreateChn(0, &stAdecAttr); #创建音频解码通道
```
#### VQE 声音质量增强
```
AI_TALKVQE_CONFIG_S										#音频输入声音质量增强（Talk）配置信息结构体
AI_RECORDVQE_CONFIG_S									#音频输入声音质量增强（Record）配置信息结构体

AI_TALKVQE_CONFIG_S stAiVqeTalkAttr;
memset(&stAiVqeTalkAttr, 0, sizeof(AI_TALKVQE_CONFIG_S));
stAiVqeTalkAttr.s32WorkSampleRate							#工作采样频率
stAiVqeTalkAttr.s32FrameSample								#采样点数目
stAiVqeTalkAttr.aParamFilePath								#参数文件路径
stAiVqeTalkAttr.u32OpenMask =
    AI_TALKVQE_MASK_AEC | AI_TALKVQE_MASK_ANR | AI_TALKVQE_MASK_AGC;	# Talk Vqe 的各功能使能的 Mask 值。目前支持 AI_TALKVQE_MASK_AEC、AI_TALKVQE_MASK_ANR、AI_TALKVQE_MASK_AGC

AI_RECORDVQE_CONFIG_S stAiVqeRecordAttr;
memset(&stAiVqeTalkAttr, 0, sizeof(AI_TALKVQE_CONFIG_S));
stAiVqeRecordAttr.s32WorkSampleRate							#工作采样频率
stAiVqeRecordAttr.s32FrameSample								#采样点数目
stAiVqeRecordAttr.stAnrConfig.fPostAddGain						# ANR 的 post-gain
stAiVqeRecordAttr.stAnrConfig.fGmin								# ANR 频谱增益底限，单位为 dB，默认值为 -30dB
stAiVqeRecordAttr.stAnrConfig.fNoiseFactor						# ANR 噪音抑制系数，默认值为 0.98
stAiVqeRecordAttr.u32OpenMask = AI_RECORDVQE_MASK_ANR;	# Record Vqe 的各功能使能的 Mask 值。目前支持 AI_RECORDVQE_MASK_ANR
```

## 具体示例

### 视频
为了方便用户快速掌握 RKMedia 的开发流程，Firefly 提供了大量的测试 DEMO。代码路径 `SDK/external/rkmedia/example/`、`SDK/app/firefly_rkmedia_demo`、 `SDK/app/firefly_rkmedia_uvc`。

#### VI->GetFrame
* 该开发流程对应的 DEMO 示例为 `rkmedia_vi_get_frame_test.c`
* 说明：设备输入保存至文件。演示 `VI` 没有 `Bind` 时如何取视频流。
* 快速使用：
```
#从摄像头节点 rkispp_scale0 抓取 10 帧图片并保存为 1080p.nv12 文件
./rkmedia_vi_get_frame_test  -a /oem/etc/iqfiles/ -w 1920 -h 1080 -d rkispp_scale0 -o /tmp/1080p.nv12 -c 10

# 命令录取 10 帧图像数据，截取最后一帧来预览
# 使用 dd 跳过前 9 帧数据，得到最后一帧。3110400 = 1920 x 1080 x 3 / 2 一帧 NV12 数据大小
dd if=1080p.nv12 of=1080pl.nv12 bs=3110400 skip=9
# 使用 ffmpeg 把 NV12 图像转换为 PNG 格式。
fmpeg -y -f rawvideo -pix_fmt nv12 -ss 00:01 -r 1 -s 1920x1080 -i 1080pl.nv12 -frames:v 1 output.png
# 打开 output.png 预览。
```
* DEMO 效果截图如下图所示：

![](../../../rv1126_img/CAM-C1126S2U/vi_get_frame.png)

#### VI->UVC
* 该开发流程对应的 DEMO 示例为 `firefly_rkmedia_vi_uvc_test.c`
* 说明：将设备虚拟成网卡与 UVC `复合设备`。设备摄像头输入并使用 `rknn` 模型处理图像数据。图像数据通过 UVC 传到 host 上位机。rknn 输出数据结果通过虚拟网卡传输到 host 上位机。host 上位机使用 `OpenCV` 开发，可应用于 `Windows/Linux/macOS` 等平台。该 demo 只适用于 sdk-ai 。sdk-ai [源码获取链接](https://wiki.t-firefly.com/zh_CN/CAM-C11092U/Source_code.html)。
```
# 由于配置项众多，目前已将 demo 固化成 mk 文件，使用 mk 文件一键编译生成 demo 固件
# 编译命令
./build.sh device/rockchip/rv1126_rv1109/aio-rv1126-rkmedia-uvcc.mk
./build.sh

# 将生成的固件烧录进设备，开机脚本 /etc/init.d/S58_lunch_init 默认启动 demo。板子不需要执行任何操作。只需要执行 Linux host 端的命令即可。
# 由于 demo 未完善。Linux host 端执行一次 sudo ./client 之后需要重启设备才能再次打开 demo 。
```
* 快速使用：
```
# Linux host
sudo ./client
```

* DEMO 效果截图如下图所示：

![](../../../rv1126_img/CAM-C1126S2U/vi_uvc.png)

#### VI->RKNN->VENC->RTSP
* 该开发流程对应的 DEMO 示例为 `rkmedia_vi_rknn_venc_rtsp_test.c`
* 说明：该示例解析 rtsp-nn.cfg 配置文件来配置 `VI` 接口和 `VENC` 接口的输入端与输出端信息，将 `VI` 接口获取摄像头节点图片数据送入 `NPU` 进行 `AI` 模型推理，然后解析推理结果。根据推理结果进行像素点偏移画框，然后将处理好的数据使用接口 `RK_MPI_SYS_SendMediaBuffer` 发送到 `VENC` 编码接口进行编码，最后使用 `rtsp_tx_video` 接口将 `VENC` 编码好的数据流推流到 `RTSP` 网络。
* 快速使用：
```
# 路径需要有相关文件
./rkmedia_vi_rknn_venc_rtsp_test -a /oem/etc/iqfiles/ -c /oem/usr/share/rtsp-nn.cfg -b /oem/usr/share/rknn_model/box_priors.txt -l /oem/usr/share/rknn_model/coco_labels_list.txt -p /oem/usr/share/rknn_model/ssd_inception_v2_rv1109_rv1126.rknn

# PC 端使用 VLC 播放器预览 RTSP 推流命令为
vlc rtsp://168.168.101.208:554/live/main_stream
#注： 168.168.101.208 为开发板的 IP 地址，根据实际情况进行调整
```
* DEMO 效果截图如下图所示：

![](../../../rv1126_img/CAM-C1126S2U/vi_rknn_venc_rtsp.png)

#### RTSPGet->VDEC(Multi)->VO
* 该开发流程对应的 DEMO 示例为 `rkmedia_rtspget_multi_test.cc`
* 说明：该示例使用 `RTSP` 拉流接口 `ffrtspGet` 从多个网络摄像头获取数据，然后使用 `VDEC` 接口解码，接着使用 `RGA` 加速将原始数据构建成目标数据，最后使用 `RK_MPI_SYS_SendMediaBuffer` 接口将目标数据发送到 `VO` 接口以分屏输出显示。
* 快速使用：
```
# RTSP 取流 2 个网络摄像头，每个摄像头取流 2 次，并输出显示到显示屏
./rkmedia_rtspget_multi_test rtsp://admin:firefly123@168.168.100.94:554/av_stream rtsp://admin:firefly123@168.168.100.94:554/av_stream rtsp://admin:firefly123@168.168.100.97:554/av_stream rtsp://admin:firefly123@168.168.100.97:554/av_stream
```
* DEMO 效果截图如下图所示：

![](../../../rv1126_img/CAM-C1126S2U/rtsp_multi_vo.jpg)

#### RTSPGet->VDEC->RKNN->VENC->RTSPPush
* 该开发流程对应的 DEMO 示例为 `rkmedia_rtspget_vdec_rknn_venc_rtsp_test.cc`
* 说明：该示例可实现多个 `RTSP` 拉流解码与多个 `RTSP` 编码推流的效果。其流程使用 `RTSP` 拉流接口 `ffrtspGet` 从多个网络摄像头获取数据，然后使用 `VDEC` 接口解码。将解码的图片数据送入 `NPU` 进行 `AI` 模型推理，然后解析推理结果，使用 OpenCV 将推理结果应用到图片上。`VENC` 则对 OpenCV 处理后的图片进行编码，最后使用接口 `ffrtspPush` 将 `VENC` 编码好的数据流推流到 `RTSP` 网络。
* 快速使用：
```
#运行该示例需要有库文件 libffrtsp.so
./rkmedia_rtspget_vdec_rknn_venc_rtsp_test -c /usr/share/ffrtsp-nn.cfg -p /usr/share/rknn_model/ssd_inception_v2_rv1109_rv1126.rknn -l /usr/share/rknn_model/coco_labels_list.txt -b /usr/share/rknn_model/box_priors.txt

# ffrtsp-nn.cfg 两个网络摄像头的配置分别为
video_type=6 video_fps=25 width=1920 height=1080 image_type=4 port=8554 video_url=rtsp://admin:firefly123@168.168.100.94:554/av_stream
video_type=6 video_fps=25 width=1920 height=1080 image_type=4 port=8555 video_url=rtsp://admin:firefly123@168.168.100.97:554/av_stream

#则 PC 端使用 VLC 预览 RTSP 推流画面命令为
vlc rtsp://168.168.101.208:8554/H264_stream_0
vlc rtsp://168.168.101.208:8555/H264_stream_1
#注：168.168.101.208 为开发板的 IP ，根据实际情况进行调整
```
* DEMO 效果截图如下图所示：

![](../../../rv1126_img/CAM-C1126S2U/rtspget_vdec_rknn_venc_rtsppush.png)

### 音频
* 音频功能演示示例： `rkmedia_audio_test` 。支持 `AI->AENC->AO` 循环， `AI->AENC->File` ， `File->ADEC->AO` ， `AI 输入 Vqe 增强后 AO 输出`四种模式。
* 快速使用：
```
#采样率。可选项：0、16000、22050、24000、32000、44100、48000
# AI->AENC->AO
./rkmedia_audio_test 0 16000

# AI->AENC->File
./rkmedia_audio_test 1 16000 /userdata/output.aac

# File->ADEC->AO
./rkmedia_audio_test 2 16000 /userdata/input.aac

# AI 输入 Vqe 增强后 AO 输出
./rkmedia_audio_test 3 16000
```