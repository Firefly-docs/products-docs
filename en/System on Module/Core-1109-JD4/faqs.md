# FAQS

## SDK version
* Check the current SDK version
```
cd path-to-sdk/
realpath .repo/manifest.xml
```

* Rollback SDK version
```
cd path-to-sdk/
# Link the SDK version file name to be rolled back
ln -sf .repo/manifests/rv1126_rv1109/xxx.xml .repo/manifest.xml
# Start rollback
repo sync -c
```

## NPU Version
* The SDK version is rv1126_rv1109_linux_release_20220324_v2.2.5b.xml and the date is 2022.03.24 or above. The default NPU version is NPU 1.7.0.
* Switch NPU version: modify the configuration file corresponding to buildroot according to the RK_CFG_BUILDROOT variable described in the mk file, and add the following configuration
```
BR2_PACKAGE_NPU_ARC=y
```
After compiling and burning the system, execute the /npu_arc/usr/bin/npu_arc_env.sh script on RV1126_RV1109 to configure the NPU 1.6.0 environment. The principle of this operation is to uninstall the 1.7.0 kernel NPU ko module driver, and then install the 1.6.0 kernel NPU ko module driver.

## AI Algorithm

### ROCKX Algorithm

Rock-X SDK is a set of AI component libraries based on the RK3399Pro/RK180X platform. Developers can quickly build AI applications through the API interface provided by Rock-X SDK. It is currently applicable to the RV1126_RV1109 platform. ROCKX algorithm is free, but its disadvantage is that it cannot optimize the interface functions. For more details about ROCKX, please refer to the `sdk/external/rockx/doc/Rockchip_Developer_Guide_RockX_SDK_CN.pdf` file in the SDK.

### ArcSoft Face Recognition Algorithm

RV1126_RV1109 has been adapted to the ArcSoft Face Recognition Algorithm. It is only applicable to NPU version 1.6.0, not applicable to higher NPU versions (including NPU version 1.7.0). `Currently, NPU versions above 1.6.0 (excluding NPU 1.6.0) will have serious memory leaks when using the ArcSoft Face Recognition Algorithm`, please be informed. Please contact the business department to obtain the ArcSoft Face Algorithm activation code.

### Xinche Car License Plate Recognition Algorithm

RV1126_RV1109 has been adapted to the Xinche Car License Plate Recognition Algorithm. The Xinche Car License Plate Recognition firmware can be downloaded from the official resource download interface. The license plate recognition application source code is located in the folder `sdk/external/xhlpr_app` in the SDK. Please contact the business department to obtain the license plate recognition algorithm activation code.