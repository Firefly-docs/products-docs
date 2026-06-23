# 快速部署 Llama3

为了便于用户快速体验 Llama3，Firefly 开源团队提供了开箱即用的快速部署包，请从[下载中心]获取快速部署包。

## 部署流程

将部署包传输至 AIBOX-1684 机器，推荐使用 `scp` 命令传输：

1. 在 AIBOX-1684 上执行 `ifconfig` 命令查看其 IP 地址。
2. 输入 `scp` 命令传输部署包，例：`scp llama3.tar linaro@<aibox-ip-addr>:~/`，其中 `<aibox-ip-addr>` 为 AIBOX-1684 的 IP 地址。

解压部署包：`tar -xvf ./llama3.tar`。

一行命令部署 Llama3：`./talk_to_llama3.sh`。

注意：部署过程中无需联网，首次部署 Llama3 需要花费较长时间进行相应的软件安装（3-5分钟），请耐心等待。

[下载中心]: https://www.t-firefly.com/doc/download/280.html