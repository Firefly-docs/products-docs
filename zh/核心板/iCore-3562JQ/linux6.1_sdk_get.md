## 获取 SDK

请联系销售(sales@t-firefly.com)获取 SDK 下载链接。

<font color=red>

**注意：**

**1. SDK 采用交叉编译，所以要在 X86_64 电脑上使用 SDK，不要将 SDK 下载到板子上**

**2. 编译环境请使用 Ubuntu20.04（真机或 docker 容器），如果使用其他版本可能导致编译出错**

**3. 不要在虚拟机共享文件夹以及非英文目录存放、解压SDK**

**4. 获取、编译 SDK 请全程使用普通用户，不允许也不需要使用 root 权限（除非需要 apt 安装软件）**
</font>

准备一个空文件夹用于存放 SDK，建议在 home 目录下，本文以`~/proj`为例

### 校验与解压

下载完成后先验证一下 MD5 码：

```bash
$ md5sum rk3562_6.1.tgz.split*
```

将得到的结果和 md5sum.txt 中的内容进行对比，确认无误后，就可以解压：
```bash
# 解压
cd ~/proj/
cat rk3562_6.1.tgz.split0* | tar -zx

# 解压后得到
~/proj/rk3562_6.1

# 安装 python2
sudo apt update && sudo apt install python2

# 从 repo 中提取文件
cd ~/proj/rk3562_6.1
python2 .repo/repo/repo sync -l --no-tags
```
