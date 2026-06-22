## Rockit AVS

<font color=red>**Platform limitations: Currently only verify support under platform 3588 Buildroot system. If you need to use it, please switch the external/rockit repository to the remote rk3588/avs branch.**</font>

### AVS ADAPTS in Buildroot systems

source code path in ` SDK/external/rockit/mpi/example/mod `, reference Rockit AVS added firefly_mpi_avs_test AVS program, using 6 road input isometric cylindrical projection of compression fusion splicing patterns, Add Mosaic image material, calibration file and program run file input and output path, has been integrated into buildroot.

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

### AVS instructions

1.File path instructions

```bash
#Splice material path
/usr/data/avs/6x_rectlinear/input_image/image_data/
#Output path
/usr/data/avs/6x_rectlinear/output_res/
#Calibration file
/usr/data/avs/6x_rectlinear/avs_calib/calib_file.pto
```

2.Operating instruction

Execute the following command：

```bash
root@RK3588:/# chmod 777 /usr/data/avs/6x_rectlinear/firefly_prepare_avs_env.sh
root@RK3588:/# /usr/data/avs/6x_rectlinear/firefly_prepare_avs_env.sh
root@RK3588:/# firefly_mpi_avs_test
```

When the program runs, the bin format file of the splicing picture is repeatedly output to the specified path, and the default execution is 100 times

```bash
output: /usr/data/avs/6x_rectlinear/output_res/chn_out_8192x2700_0_0_nv12.bin
```

If the program library is missing or abnormally stuck, please copy the specified library file to `/usr/lib/`：

```bash
#Copy the specified library file to /usr/lib/
cp /usr/data/avs/6x_rectlinear/lib/libgraphic_lsf /usr/lib/libgraphic_lsf.so
cp /usr/data/avs/6x_rectlinear/lib/libpanoStitchApp /usr/lib/libpanoStitchApp.so
```
Use YUView tool (Windows tool) to view YUV images in the material path, generate bin format file in the output path after demo runs, and drag the bin file into the YuView-item bar. Set W, H, YUVformat and other options to view the YUV picture (8192x2700-YUV(IL) 4:2:0 8-bit), the preview is as follows:

![](../../../rv1126_img/avs_yuviewtool_preview.png)

3.Material production
The demo uses the image input format of yuv-nv12, jpg image material can be converted to yuv-nv12 format for testing, which needs to be calibrated by itself. The calibration file is `calib_file.pto`, which contains the image size, yaw Angle, pitch Angle, roll Angle and other splicing parameter information.

```bash
#jpg converted to yuv-nv12
ffmpeg -i camera_0.jpg -s 1520x2560 -pix_fmt nv12 camera_0.yuv
```

The calibration file is as follows：
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

### Splicing performance test

The GPU is set to 1G and the `dumpsys avs` command is used to calculate the throughput of picture data to analyze the splicing performance.

```bash
#Set the userspace policy fixed frequency gpu, then set different frequencies, and compare the set voltage with the measured voltage
echo userspace > /sys/class/devfreq/ff400000.gpu/governor
#Set the frequency of the fixed frequency
echo 1000000000 > /sys/class/devfreq/ff400000.gpu/userspace/set_freq
```

In terms of splicing performance, the output resolution can be close to 22M@30fps (8192x2700). The test log is as follows:
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

- Related parameter description：
  - pipe_num: Pipe Number
  - proj_mode: EQUIRECTANGULAR
  - cost_time: The current splicing task takes ms. The output frame rate of the splicing task is 1000(ms)/cost_time(ms). When cost_time approaches 33ms, the fps is 30fps
  - max_cost_time: Maximum time spent stitching history(ms)
  - pipe_id_0-pipe_id_7: Corresponds to the number of images received on the input Pipe/The number of discarded images corresponding to the input Pipe