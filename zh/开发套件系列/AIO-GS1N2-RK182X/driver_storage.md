# STORAGE 使用

RK182X开发套件 开发板上有 1 个 PCIE3.0 x 1  接口，1 个 SATA 3.0 接口以及一个 TF-Card 接口。

[接口位置如图所示](interface_definition.html#zheng-ji-jie-kou-ding-yi)  

## PCIE 的使用

PCIE3.0  接口(该接口与 SATA 3.0 接口复用，默认系统使用 PCIE 接口)默认支持接 NVME SSD 2242/2260/2280 ,在系统默认的名字为 `nvme0n1`。

## SATA 的使用
如果想切换使用 SATA3.0 接口，需要做软件上的配置（bm1688_1688jd4.dtsi）:

```
#if 0
&pcie0{
	reset-gpio = <&porte 3 GPIO_ACTIVE_HIGH>; 
	status = "okay";
};
&sata{
	status = "disabled";
};
#else
&pcie0{
	reset-gpio = <&porte 3 GPIO_ACTIVE_HIGH>; 
	status = "disabled";
};
&sata{
	status = "okay";
};
#endif
```
