# 软件兼容性

## 服务器安装
### 工具准备

相关工具准备如下：

- 防静电腕带或防静电手套
- M3 十字螺丝刀
- 劳保手套
- 防静电包装袋
- 一字螺丝刀


### 防静电

#### 操作准则

为降低静电对您和产品造成损伤的几率，请注意以下操作准则：

- 所有机房应该铺设防静电地板（或防静电地垫），使用防静电工作椅。机房的隔板、屏风、窗帘等应使用防静电材料。
- 机房的落地式用电设备、金属框架、机架的金属外壳必须直接与大地连接，工作台上的所有用电仪器工具应通过工作台的公共接地点接地。
- 请注意监控机房温度、湿度。暖气会降低室内湿度并增加静电。
- 在运输、保管服务器组件的过程中，必须使用专用的防静电袋与防静电盒，以确保服务器组件的防静电安全。
- 机房内的人员在进行服务器组件安装、插拔等接触操作时必须佩戴防静电腕带，并将接地端插入机架上的 ESD 插孔。
- 在接触设备前，应当穿上防静电工作服、佩戴防静电手套或防静电腕带、去除身体上携带的易导电物体（如首饰、手表等），以免被电击或灼伤，如图所示。

    ![ESD Prohibited Conductive Objects Diagram](../../../img/servers/CSA1-N8/esd_remove_conductive_items.png)

- 防静电腕带的两端必须接触良好，一端接触您的皮肤，另一端牢固地连接到机箱的 ESD 接口。佩戴防静电腕带的具体步骤请参见佩戴防静电腕带。
- 在更换的过程中，应将所有还没有安装的服务器组件保留在带有防静电屏蔽功能的包装袋中，将暂时拆下来的服务器组件放置在具有防静电功能的泡沫塑料垫上。
- 请勿触摸焊接点、引脚或裸露的电路。


#### 佩戴防静电腕带

请确认机柜已正确接地。

1. 如图所示，将手伸进防静电腕带。

    ![ESD Wrist Strap Wearing Diagram](../../../img/servers/CSA1-N8/esd_wrist_strap.png)

2. 拉紧锁扣，确认防静电腕带与皮肤接触良好。

3. 将防静电腕带的接地端插入机柜的防静电腕带插孔。


## 5.1.3 安装环境要求

### 5.1.3.1 空间要求和通风要求

为方便服务器维修和正常通风，请满足以下空间和通风要求：

- 服务器必须安装在出入受限区域。
- 保持设备所在区域整洁。
- 为了设备通风散热和便于设备维护，确保机柜前后都要空余 800 mm 的空间。
- 服务器入风口处应避免有障碍物阻挡，影响正常进风和散热。
- 服务器放置位置的空调送风量应足够提供服务器需要的风量，保证服务器内部各器件散热。

### 5.1.3.2 温度要求与湿度要求

为确保服务器能够持续安全可靠地运行，请将服务器安装或放置在通风良好、温度及湿度可控制的环境中。

- 不论气候条件，均应设置长年的温控装置。
- 对于干燥或湿度过大的地区可采用加湿机或抽湿机来保证环境湿度。

| 项目 | 说明 |
| :--- | :--- |
| 温度 | 5 ℃ ～ 40 ℃（41 ℉ ～ 104 ℉） |
| 湿度 | 8% RH ～ 90% RH（无冷凝） |


### 5.1.3.3 机柜要求

为确保服务器能够持续安全可靠地运行，请将服务器安装或放置在通风良好、温度及湿度可控制的环境中。

- 满足 IEC（International Electrotechnical Commission）297 标准的宽 19 英寸、深 800 mm 以上的通用机柜。
- 在机柜门上安装防尘网。
- 在机柜后面提供交流电源接入。


## 5.2 服务器上电

### 上电操作步骤

1. 开机前确认服务器整机完全下电。
2. 将品字电源线一端插接至服务器电源接口。
3. 电源线另一端接入 PDU 机柜配电单元或合格供电插排。
4. 闭合漏电保护开关，给 PDU / 供电插排送电。
5. 若服务器刚完成断电操作，需静置等待不少于 30 秒后方可重新上电。

### 安全警示

> **严禁带电插拔电源线缆至服务器电源接口**；带电插接易产生电弧，极易击穿、损伤电源内部 MOS 功率管，缩短电源使用寿命；极端情况下会直接造成服务器电源失效、无法启动。

### 上电方式

服务器有以下几种上电方式：

**方式一：通过 Power 按键开机（电源按钮/指示灯为绿色常亮）**

1. 通过短按前面板的电源按钮，将服务器上电。电源按钮位置请参见前面板指示灯和按钮。

**方式二：通过 aBMC WebUI 上电**

1. 通过 aBMC WebUI 首页右上角的电源按键开机进入“服务器上下电”界面。
2. 单击“上电”，出现上电提示时单击“确定”将服务器上电。

> 系统默认“通电开机策略”为“保持上电”，即服务器的电源模块通电后系统自动开机。用户可通过 aBMC 修改“通电开机策略”。

**方式三：通过远程虚拟控制台将服务器上电**

1. 通过浏览器远程登录 aBMC。
 ![aBMC dashboard View](../../../img/servers/CSA1-N8/abmc_dashboard_view.png)
2. 通过aBMC WebUI首页右上角的电源按钮设置为“ON”，服务器完成整机上电。
 ![aBMC Web Power Button View](../../../img/servers/CSA1-N8/abmc_power_button_view.png)


## 服务器下电
<div>
  <h3>Explanation</h3>
  <ul style="padding-left: 24px; margin: 10px 0;">
    <li>下电后，所有业务和程序将终止，因此下电前请务必确认服务器所有业务和程序已经停止或者转移到其他设备上。</li>
    <li>本章节的"下电"指将服务器下电至 Standby 状态（电源按钮指示灯为黄色常亮）。</li>
    <li>服务器强制下电后，需要等待 10 秒以上，以确保服务器完全下电，此时可进行再次上电操作。</li>
  </ul>
</div>

服务器有以下几种下电方式：
- 服务器处于上电状态，通过短按前面板的电源按钮，可将服务器正常下电。
- 通过远程虚拟控制台将服务器上电。

## 远程登录服务器

### 登录须知
BMC默认设置了缺省参数以方便用户操作，参数如下表所示。为了保证系统的安全性，建议在首次操作时修改缺省值，并定期更新。

<table border="1" cellPadding="8" cellSpacing="0" width="100%">
  <thead>
    <tr>
      <th>类别</th>
      <th>名称</th>
      <th>默认值</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowSpan="2">aBMC 管理系统数据</td>
      <td>登录用户名</td>
      <td>admin</td>
    </tr>
    <tr>
      <td>登录密码</td>
      <td>admin</td>
    </tr>
    <tr>
      <td rowSpan="2">aBMC 管理网口 IPv4 地址<br/>● MGNT 或 GM 网口</td>
      <td rowSpan="2">管理网口 IP 与子网掩码</td>
      <td>默认 IP 地址：192.168.1.2</td>
    </tr>
    <tr>
      <td>默认子网掩码：255.255.255.0</td>
    </tr>
    <tr>
      <td>BMC Console 串口</td>
      <td>波特率</td>
      <td>115200</td>
    </tr>
    <tr>
      <td rowSpan="2">BMC Linux 用户数据</td>
      <td>登录用户名</td>
      <td>bmc</td>
    </tr>
    <tr>
      <td>登录密码</td>
      <td>bmc</td>
    </tr>
  </tbody>
</table>

## 5.4.2 登录 aBMC Web 远程虚拟控制台

aBMC 提供 Web 界面，通过可视化、友好的界面来帮助用户完成服务器的管理。

### 5.4.2.1 环境准备

#### 1. 将服务器连接到网络

登录 aBMC 前，请先将 aBMC 管理接口连接到网络，确保本地 PC 和服务器路由可达，如下图所示。
 ![PC-Switch-Server Basic Network Connection Topology Diagram](../../../img/servers/CSA1-N8/pc_switch_server_basic_network_topology.png)

服务器支持以下两种 aBMC 管理接口，详情请参考网络设置，你可以根据业务需求，选择合适的 aBMC 管理接口。

- **aBMC 共享网口**：可以同时处理 aBMC 管理流量和服务器业务数据流量的网络接口。
- **aBMC 专用网口**：专门用于处理 aBMC 管理流量的网络接口，如下图所示。

 ![MGMT Management Port Wiring Diagram](../../../img/servers/CSA1-N8/mgmt_port_cable_connection.png)

#### 2. 获取 aBMC 管理 IP 地址

登录 aBMC 前，需要先获取 aBMC 管理 IP（aBMC 专用网口/共享网卡的 IP 地址）。可以通过 Linux 系统的网络命令（`ip` 或 `ifconfig`）来获取 aBMC 管理 IP 地址，如下图所示使用 `ip` 命令获取 MGMT 网卡 IP 地址：

> （此处为命令执行示例图）

 ![MGMT Port IP Query Command Output Screenshot](../../../img/servers/CSA1-N8/mgnt_ip_query_terminal_screenshot.png)

#### 3. 访问aBMC
通过Web浏览器即可访问aBMC。aBMC支持的浏览器版本及客户端分辨率如下表所示。
**WEB客户端配置需求**

<table border="1" cellPadding="8" cellSpacing="0" width="100%">
  <thead>
    <tr>
      <th>浏览器版本</th>
      <th>默认值分辨率</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Google Chrome 48.0 及以上</td>
      <td rowSpan="4">要求不低于 1366*768，推荐设置为 1600*900 或更高</td>
    </tr>
    <tr>
      <td>Mozilla Firefox 50.0 及以上</td>
    </tr>
    <tr>
      <td>Internet Explorer 11 及以上</td>
    </tr>
    <tr>
      <td>Microsoft Edge 97 及以上</td>
    </tr>
  </tbody>
</table>




#### 5.4.2.2 登录aBMC Web页面
本指南以Chrome浏览器为例介绍登录 aBMC 界面的操作步骤。
1.  打开Chrome浏览器，在地址栏使用HTTPS方式输入 aBMC 管理IP地址（如https://192.168.1.2），弹出告警窗口，如下图所示。
    ![aBMC Certificate Warning Operation Schematic Diagram](../../../img/servers/CSA1-N8/abmc_chrome_cert_warning_schematic.png)
2. 点击“Advanced”按钮：当你看到类似于 “Your connection is not private”（你的连接不是私人连接）或者 “Warning: Potential Security Risk Ahead”（警告：潜在的安全风险）时，点击页面底部的 “Advanced” 按钮。
3. 点击“Proceed to (site) (unsafe)”：通常在警告信息下方会有这个选项，点击它就能继续访问该网站。
下图为aBMC登录页面
    ![aBMC Login Page Schematic Diagram](../../../img/servers/CSA1-N8/abmc_login_page.png)
4. 成功登录后可以看到aBMC整机设备概览，用户对服务器运行状态进行监控分析和健康检查。
下图为设备列表页面，用户可以在可以通过此页面查看ARM计算单元运行信息和Shell命令行操作。
![aBMC dashboard View](../../../img/servers/CSA1-N8/abmc_device_list.png)
下图为aBMC的“固件升级”页面，方便用户对ARM计算单元进行固件升级操作。
![Add Firmware Upgrade Popup Schematic Diagram](../../../img/servers/CSA1-N8/abmc_fw_upgrade_popup.png)
![Firmware Upgrade Task Monitoring Page Schematic Diagram](../../../img/servers/CSA1-N8/abmc_fw_upgrade_monitor_page.png)
5. 更多aBMC的操作请查看《aBMC用户指南》



### 5.4.3 登录aBMC 命令行
<div>
  <h3>Explanation</h3>
  <ul style="padding-left: 24px; margin: 10px 0;">
    <li>为保证系统的安全性，初次登录时，请及时修改初始密码，并定期更新。</li>
  </ul>
</div>


#### 5.4.3.1 通过 Console 串口登录

1. 使用 RJ45 串口线连接 Console。
2. 通过超级终端登录串口命令行，需要设置的参数有：
   - 波特率：115200
   - 数据位：8
   - 奇偶校验：无
   - 停止位：1
   - 数据流控制：无
3. 呼叫成功后输入用户名和密码。
4. 登录成功。

![BMC OS Release Query Command Line Schematic Diagram](../../../img/servers/CSA1-N8/cmd_os_release_info.png)

#### 5.4.3.2 使用 SSH 登录

1. 可以使用系统 `ssh` 命令或者 MobaXterm 通过 SSH 登录 BMC。
2. 输入默认账号密码。