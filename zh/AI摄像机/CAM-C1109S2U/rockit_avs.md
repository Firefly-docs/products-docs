## Rockit AVS

<font color=red>**平台限制：目前只在 3588 平台 Buildroot 系统下验证支持。如需使用请把 external/rockit 仓库切换到远程的 rk3588/avs 分支。**</font>

### AVS 在 Buildroot 系统适配

源码路径在`sdk/external/rockit/mpi/example/mod`，参考 Rockit AVS 添加了 firefly_mpi_avs_test 程序样例，采用6路输入等距柱面投影非压缩融合拼接模式，添加拼接图片素材、标定文件及程序运行文件输入输出路径，已整合到 buildroot 中。

```bash
diff --git a/mpi/example/mod/CMakeLists.txt b/mpi/example/mod/CMakeLists.txt
index 679c0b0..b6891df 100644
--- a/mpi/example/mod/CMakeLists.txt
+++ b/mpi/example/mod/CMakeLists.txt
@@ -76,6 +76,10 @@ set(RK_MPI_TEST_AVIO_SRC
     sys/test_sys_avio.cpp
 )

+set(FIREFLY_MPI_TEST_AVS_SRC
+    firefly_test_mpi_avs.cpp
+)
+
 #--------------------------
 # rk_mpi_ao_test
 #--------------------------
@@ -199,3 +203,15 @@ install(TARGETS rk_mpi_gdc_test RUNTIME DESTINATION "bin")
 add_executable(rk_mpi_avio_test ${RK_MPI_TEST_AVIO_SRC} ${RK_MPI_TEST_COMMON_SRC})
 target_link_libraries(rk_mpi_avio_test ${ROCKIT_DEP_COMMON_LIBS})
 install(TARGETS rk_mpi_avio_test RUNTIME DESTINATION "bin")
+
+#--------------------------
+# firefly_mpi_avs_test
+#--------------------------
+add_executable(firefly_mpi_avs_test ${FIREFLY_MPI_TEST_AVS_SRC} ${RK_MPI_TEST_COMMON_SRC})
+target_link_libraries(firefly_mpi_avs_test ${ROCKIT_DEP_COMMON_LIBS})
+install(TARGETS firefly_mpi_avs_test RUNTIME DESTINATION "bin")
+
+#--------------------------
+# add 6x_rectlinear data
+#--------------------------
+install(DIRECTORY "6x_rectlinear" DESTINATION "data/avs")
diff --git a/mpi/example/mod/firefly_test_mpi_avs.cpp b/mpi/example/mod/firefly_test_mpi_avs.cpp
new file mode 100644
index 0000000..9cfb68c
--- /dev/null
+++ b/mpi/example/mod/firefly_test_mpi_avs.cpp
```

### AVS 使用说明

1.文件路径说明

```bash
#拼接素材路径
/usr/data/avs/6x_rectlinear/input_image/image_data/
#输出路径
/usr/data/avs/6x_rectlinear/output_res/
#标定文件
/usr/data/avs/6x_rectlinear/avs_calib/calib_file.pto
```

2.运行说明

执行如下命令：

```bash
root@RK3588:/# chmod 777 /usr/data/avs/6x_rectlinear/firefly_prepare_avs_env.sh
root@RK3588:/# /usr/data/avs/6x_rectlinear/firefly_prepare_avs_env.sh
root@RK3588:/# firefly_mpi_avs_test
```

程序运行时重复输出拼接图片的 bin 格式文件到指定路径，默认执行 100 次。

```bash
output: /usr/data/avs/6x_rectlinear/output_res/chn_out_8192x2700_0_0_nv12.bin
```

若程序出现库缺失或者异常卡死情况，请将指定库文件拷贝到`/usr/lib/`下：

```bash
#将指定库文件拷贝到/usr/lib/下
cp /usr/data/avs/6x_rectlinear/lib/libgraphic_lsf /usr/lib/libgraphic_lsf.so
cp /usr/data/avs/6x_rectlinear/lib/libpanoStitchApp /usr/lib/libpanoStitchApp.so
```

使用 YUView 工具（Windows 工具）可查看素材路径下的 YUV 图片，demo 运行后在输出路径下生成 bin 格式文件，将 bin 文件拖进 YUView-Item 栏，设置 W、H、YUVformat 等选项查看 YUV 图片（8192x2700-YUV(IL) 4:2:0 8-bit），预览情况如下：

![](../../../rv1126_img/avs_yuviewtool_preview.png)

3.素材制作

demo 采用图片输入格式为 yuv-nv12，可将 jpg 图片素材转为 yuv-nv12 格式进行测试，需要自行标定，标定文件为`calib_file.pto`，包含了图片大小、偏航角、俯仰角、翻滚角等拼接参数信息。

```bash
#jpg转yuv-nv12
ffmpeg -i camera_0.jpg -s 1520x2560 -pix_fmt nv12 camera_0.yuv
```

标定文件如下：
```bash
root@RK3588:/usr/data/avs/6x_rectlinear/avs_calib# cat calib_file.pto

p w8378 h4189 f2 v360 u0 n"JPEG g0 q95"
m g0 i0

i n"camera_0.jpg" w2560 h1520 f0 y-113.603 r-90.9529 p30.5159 v90.12 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_1.jpg" w2560 h1520 f0 y-68.4270 r89.24700 p32.1190 v90.24 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_2.jpg" w2560 h1520 f0 y-26.2000 r-90.0960 p32.8350 v90.16 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_3.jpg" w2560 h1520 f0 y22.40400 r90.50000 p32.8000 v90.29 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_4.jpg" w2560 h1520 f0 y63.53500 r-89.1020 p31.5140 v90.26 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0

i n"camera_5.jpg" w2560 h1520 f0 y110.5400 r93.83400 p31.6180 v90.53 a0.03 b-0.12737 c-0.0363 d0 e0 g0 t0
```

### 拼接性能测试

GPU 定频到 1G，在程序运行时使用 dumpsys avs 命令统计图片数据的吞吐量以分析拼接性能。

```bash
#设置userspace策略定频gpu，然后设置不同频率，比较设置电压和实测电压
echo userspace > /sys/class/devfreq/ff400000.gpu/governor
#设置定频的频率
echo 1000000000 > /sys/class/devfreq/ff400000.gpu/userspace/set_freq
```

拼接性能方面，输出分辨率可接近 22M@30fps（8192x2700），测试 log 如下：

```bash
-------------------------------------------------------------------------------
END DUMP OF SERVICE avs:
# dumpsys avs
-------------------------------------------------------------------------------
DUMP OF SERVICE avs:
---------------------- avs group attr ----------------------------
grp_id    mode           enable    pipe_num  is_sync   src_rate  dst_rate
0         BLEND          Y         6         N         -1        -1
---------------------- avs lut attr ----------------------------
grp_id    lut_data_acc   lut_data_path
0         HIGH           NONE
---------------------- avs output attr ----------------------------
grp_id    proj_mode                center_x  center_y  fov_x     fov_y
0         EQUIRECTANGULAR          4220      2124      28000     9500

          ori_yaw   ori_pitch ori_roll  yaw       pitch     roll
0         0         0         0         0         0         0

          middel_lut_path     calib_path          mask_path           mesh_alpha_path
0         NONE                /data/avs/6x_rectlinNONE                /data/avs/6x_rectlin
                              ear/avs_calib/calib_                    ear/avs_mesh/
                              file.pto
---------------------- avs channel attr ----------------------------
grp_id    chn_id    enable    width     height    is_compress    dym_range depth     src_rate  dst_rate  
0         0         Y         8192      2700      N              SDR8      3         -1        -1
---------------------- avs group work status ----------------------------
grp_id    cost_time max_cost_time  false_count
0         34        40             0
---------------------- avs pipe work status ----------------------------
grp_id    pipe_id_0 pipe_id_1 pipe_id_2 pipe_id_3 pipe_id_4 pipe_id_5 pipe_id_6 pipe_id_7
0         100       100       100       100       100       100       0         0
          0         0         0         0         0         0         0         0
---------------------- avs pipe queue ----------------------------
grp_id    pipe_id_0 pipe_id_1 pipe_id_2 pipe_id_3 pipe_id_4 pipe_id_5 pipe_id_6 pipe_id_7
```

- 相关参数说明：
  - pipe_num：Pipe 数目
  - proj_mode：EQUIRECTANGULAR
  - cost_time：当前拼接任务耗时(ms)，拼接输出帧率 fps = 1000(ms) / cost_time(ms)，当 cost_time 接近 33ms 时，fps 为 30fps
  - max_cost_time：拼接历史最大耗时(ms)
  - pipe_id_0-pipe_id_7：对应输入 Pipe 上的接收的图像数/对应输入 Pipe 上的丢弃的图像数