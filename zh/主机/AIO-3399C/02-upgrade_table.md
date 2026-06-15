# 使用USB线烧写须知(重要)


## 前言

AIO-3399C 出厂时默认安装Android7.1(industry)系统。

AIO-3399C 官方可能支持多种系统，因此会有烧写不同系统的固件的场景出现。

每种系统的固件都有对应的烧写工具版本，使用的烧写工具版本不对可能会导致烧写固件失败，尤其需要注意**跨系统烧写固件**的场景。

本章节主要介绍**系统固件与烧写工具版本的对应关系**以及**跨系统烧写固件注意事项**。

## 系统固件与烧写工具版本的对应关系

| 系统固件 | 烧写工具 |
| :----:| :----: |
| Android7.1(tvbox) | Windows: **AndroidTool v2.38** <br/> Linux: **upgradetool v1.24** |
| Android7.1(industry) | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |
| Android8.1 | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |
| Android10.0 | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.49** |
| Ubuntu | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |

根据上表下载对应的烧写工具版本:

* [Linux_Upgrade_Tool]
* [Androidtool_xxx(版本号)]

## 跨系统烧写固件注意事项

### 名词解释
`跨系统烧写固件`: 即将要烧写的系统固件和当前已烧写的系统固件不属于同一种系统的固件。

### 注意事项
跨系统烧写固件一般需要**先擦除**，**后烧写**，且擦除和烧写都需要注意**使用对应的烧写工具版本**，步骤如下：

* 使用当前已烧写的系统固件对应的烧写工具版本擦除
* 使用即将要烧写的系统固件对应的烧写工具版本烧写

#### 举例
以AIO-3399C 出厂时默认烧写Android7.1(industry)固件，客户需要烧写Android10.0固件举例：

根据[系统固件与烧写工具版本的对应关系](02-upgrade_table.html#xi-tong-gu-jian-yu-shao-xie-gong-ju-ban-ben-de-dui-ying-guan-xi)可以知道：

| 系统固件 | 烧写工具 |
| :----:| :----: |
| Android7.1(industry) | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.34** |
| Android10.0 | Windows: **AndroidTool v2.65** <br/> Linux: **upgradetool v1.49** |

步骤如下：

* 使用 **AndroidTool v2.65** 或 **upgradetool v1.34** 擦除
* 使用 **AndroidTool v2.65** 或 **upgradetool v1.49** 烧写



[Linux_Upgrade_Tool]: http://www.t-firefly.com/doc/download/page/id/54.html#windows_375
[Androidtool_xxx(版本号)]: http://www.t-firefly.com/doc/download/page/id/54.html#other_248
[RK3399_Android7.1_EraseIDB.7z]: http://www.t-firefly.com/doc/download/page/id/54.html#other_397