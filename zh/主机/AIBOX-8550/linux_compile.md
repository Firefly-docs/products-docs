# Linux 编译

这篇教程介绍如何给 AIBOX-8550 编译应用程序。

## 本地编译

AIBOX-8550 运行 Ubuntu 系统，因此可以直接在设备上通过 apt 安装工具进行软件的编译:
```bash
sudo apt update
sudo apt install build-essential

# 查看安装好的编译器
ls /usr/bin/aarch64-linux-gnu-*
```

将编译器路径填入程序的构建规则中，比如 Makefile:
```makefile
# 指定编译工具链
CC := aarch64-linux-gnu-gcc
LD := aarch64-linux-gnu-ld

CFLAGS := -Wall -O2

TARGET := hello
OBJS := main.o utils.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)
```

## 交叉编译

如果需要交叉编译，请使用相同版本的 Ubuntu 上位机，安装交叉编译工具：
```bash
sudo apt update
sudo apt install crossbuild-essential-arm64

# 查看安装好的编译器
ls /usr/bin/aarch64-linux-gnu-*
```

并且需要将设备上的以下目录复制到上位机的一个单独目录中作为 sysroot:
```bash
# 库文件
/opt/qti/lib
/opt/qti/usr/lib
/lib
/usr/local/lib
/usr/aarch64-linux-gnu/lib
/usr/lib

# 头文件
/opt/qti/usr/include
/usr/include
/usr/local/include

# 其他
/usr/share/pkgconfig
/etc/alternatives
```

然后将编译器路径和 sysroot 路径填入构建规则中，比如 Makefile:
```makefile
# 交叉编译工具链前缀
CROSS_COMPILE := aarch64-linux-gnu
CC := $(CROSS_COMPILE)gcc
LD := $(CROSS_COMPILE)ld

# sysroot 路径
SYSROOT ?= /path/to/sysroot

# 编译和链接标志
CFLAGS := -Wall -O2
CFLAGS += --sysroot=$(SYSROOT)
LDFLAGS := --sysroot=$(SYSROOT)

TARGET := hello
OBJS := main.o utils.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)
```

## 定制

如果需要定制内核或者自制底板，请联系 sales@t-firefly.com