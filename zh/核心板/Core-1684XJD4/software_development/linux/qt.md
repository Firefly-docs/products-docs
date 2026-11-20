# Qt 开发

## 如何开发 Qt 程序，并通过 HDMI 接口进行显示

整体分为两个步骤：
1. 交叉编译 Qt 程序
2. 设置 Qt 运行环境并运行程序

### 1. 交叉编译

以系统为 `Ubuntu18.04` 的交叉编译主机为例，步骤如下：

#### 1.1 获取资源

通过 `git clone` 获取该 [git](https://github.com/sophon-ai-algo/sophon-qt) 包  

系统内新建并命名文件 `se-build-server.yml`，并填写以下内容：  

```
version: "3.3"
services:
    se-build-server:
        image: huangzhenming/se-build-server:V1.0
        container_name: sophon_build 
        privileged: true 
        network_mode: host 
        restart: always 
        volumes:
            - /data/your_working_dir:/workspace 
            - /data/toolchains/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu:/toolchains 
            - /etc/timezone:/etc/timezone:ro 
            - /etc/localtime:/etc/localtime:ro 
        command: ["/bin/ping", "localhost"] 
        user: linaro 
        working_dir: /workspace 
        environment: 
        - "PATH=$PATH:/toolchains/bin"
```    

 下载交叉编译所需要的工具链：[gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu](https://pan.baidu.com/s/1bvaenEKbHFNHgGTXKMpJxg)。  

 在PC机系统上安装docker运行环境，安装docker 和 docker compose工具。各个平台有所不同，请在网上搜索相关公开资料安装，此处不再缀
述。    

运行 docker（运行时会自动从docker hub 上下载镜像文件，所需时间同网速相关，请确保外网连接畅通）：    

            
```
docker-compose -f se-build-server.yml  up -d
```  
  

运行之后，会自动在系统内创建 `/data/your_working_dir` 以及 `/data/toolchains/` 文件夹 ，你也可以通过修改 `yml` 文件去生成其他路径的文件夹。


拷贝并解压 `git` 包以及交叉编译工具链到指定路径：    
```
cp -r path/to/sophon-qt/qt-lib/qtbase-5.14.tgz /data/your_working_dir
cp -r path/to/sophon-qt/qt-example/SophUI/* /data/your_working_dir
cd /data/your_working_dir/ && tar xvf qtbase-5.14.tgz
cp -r path/to/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.7z /data/toolchains/
cd /data/toolchains/ && 7z x gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu.7z
```  

* 进行交叉编译（以 `SophUI` 源码为例）：    


    * 进入 docker：     


    ```
    docker ps -a  # 获取 docker 镜像以及容器信息，镜像名应该带有 `huangzhenming` 字样
    docker exec -it  container_id  /bin/bash # 进入 docker 
    ```  


    * 进行编译（在 docker 内）：    


    ```
    cd  /workspace
    ./install/bin/qmake SophUI.pro -spec linux-aarch64-gnu-g++ CONFIG+=qtquickcompiler
    make
    ```  


    编译之后便会生成 `SophUI` 的可执行文件，可以直接放到设备上面去执行。  
            

### 2. 运行 Qt 程序

通过以上的步骤，可以获得可执行文件 `SophUI` ,可以通过 `scp` 的方式拷贝到 1684 设备中。

运行之前需要先编写脚本去设置环境变量，内容如下：  

```
#!/bin/sh
export QTDIR=/usr/lib/aarch64-linux-gnu 
export QT_QPA_FONTDIR=$QTDIR/fonts 
export QT_QPA_PLATFORM_PLUGIN_PATH=$QTDIR/qt5/plugins/
export LD_LIBRARY_PATH=/system/lib:$LD_LIBRARY_PATH
export QT_QPA_PLATFORM=linuxfb:fb=/dev/fl2000-0 
export QWS_MOUSE_PROTO=/dev/input/event3
./SophUI 
```  
之后给予脚本运行权限，在接入 HDMI 显示器的前提下执行即可。


**需要注意的是，如果1684 设备的系统是 `Debian` 系统，则需要另外导入 Qt 的运行库以及一些其他文件，否则运行会报错，而 Ubuntu 系统则不会有这些问题**

通过 `scp` 的方式，从 PC 端导入，在 1684 设备上面执行：  

```
scp PC@IP:/path/to/qtbase-5.14/lib/* /usr/lib/aarch64-linux-gnu/ 
mkdir -p /usr/lib/aarch64-linux-gnu/qt5
scp  PC@IP:/path/to/qtbase-5.14/plugins/ /usr/lib/aarch64-linux-gnu/qt5/
```
如果 `Qt` 程序中还包含了字体，还需要在此处[下载](https://pan.baidu.com/s/1bvaenEKbHFNHgGTXKMpJxg) 字体文件，下载后，拷贝到 1684 设备的 `/usr/lib/aarch64-linux-gnu/font` 目录，如：`/usr/lib/aarch64-linux-gnu/fonts/wqy-microhei.ttc`