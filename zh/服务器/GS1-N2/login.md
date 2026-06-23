# 登录

|模组|用户名|密码|
|----|----|----|
|RK3588(Core-3588JD4)<br>RK3576(Core-3576JD4)|firefly|firefly|
|NVIDIA(Jetson Orin Nano)<br>NVIDIA(Jetson Orin NX)|nvidia|nvidia|
|SOPHGO(Core-1688JD4)|linaro|linaro|

## 主模组 RK3588(Core-3588JD4)
### HDMI 或者 VGA 登录
接入显示器。
### 登录到副模组
主模组通过串口（节点 `/dev/ttyS1`，波特率 `115200` ）连接到副模组的 DEBUG 口。
```
sudo minicom -b 115200 -D /dev/ttyS1
```

## 主模组 RK3576(Core-3576JD4)
### HDMI 或者 VGA 登录
接入显示器。
### 登录到副模组
主模组通过串口（节点 `/dev/ttyS8`，波特率 `115200` ）连接到副模组的 DEBUG 口。
```
sudo minicom -b 115200 -D /dev/ttyS8
```

## 主模组 NVIDIA(Jetson Orin Nano/Jetson Orin NX)
### HDMI 或者 VGA 登录
接入显示器。
### 登录到副模组
主模组通过串口（节点 `/dev/ttyTHS1`，波特率 `115200` ）连接到副模组的 DEBUG 口。
```
sudo minicom -b 115200 -D /dev/ttyTHS1
```