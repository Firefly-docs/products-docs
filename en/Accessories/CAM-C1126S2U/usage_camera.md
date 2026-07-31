
# Camera

Regardless of Facial_Gate or AI UVC firmware, the startup application will occupy the camera node. Before testing the camera, try to execute the following command to close the startup application.

```bash
[root@RV1126_RV1109:/]# RkLunch-stop.sh
```

For the specific use of the camera, please refer to the `/path/to/sdk/external/rkmedia/examples/rkmedia_vi_double_cameras_test.c` sample code for development.

If you want to connect a V2 MIPI screen or a V3 MIPI screen to test the binocular camera display, you need to set the display resolution to 1280x800.


