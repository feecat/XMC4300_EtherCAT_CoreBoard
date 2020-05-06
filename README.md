# XMC4300_EtherCAT_CoreBoard
基于xmc4300的ethercat从站核心板，与fb1111尺寸一致

![](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/IMAGE/IMG01.jpg)

## 起因
ethercat是一种工业实时以太网，广泛用于自动化设备和数控机床上。它开放协议栈、实时性好、价格低，广泛用于驱动器、控制面板和IO设备，普及程度正一步步提高。

在很多设备上，研发人员疲于应对现场问题，对IO模块往往分为两个极端：

- 基于Modbus，只需要一颗stm32f103即可，速率限制较大，稳定性也较差（普遍为光耦或三极管输出），价格低廉。
- 基于ethercat的模块，例如倍福el系列io模块，价格高昂，稳定性极佳。

但即使是采用了价格较高的倍福模块，当使用需求较高时，在电气柜内仍需要错综复杂的接线。对于标准化的设备来说，工人接线越多故障率也越高。部分厂家开始定制ethercat模块。它们主要基于以下两大平台：

* beckhoff et1100
* microchip lan9252

实际应用中这两个平台都需要外挂单片机，并且编程门槛较高，开发环境不好搭建。

使用XMC4300核心板，可以快速开发电磁阀组、面板按钮、IO模组等。

本项目仍在测试阶段，不对成品负责。

## 选型
我参考了倍福官方的选型手册，fb1111原理图等，最终敲定xmc4300+ksz8081做核心板，有以下几个原因：
1. xmc4300自带cortex m4内核，不需要外挂单片机
2. infineon的Dave软件非常完善，解压即用
3. 尺寸兼容倍福fb1111，双层PCB，单面贴片，除了网口和排针外无插件原件。100批量下单件成本约130元
4. 稳定性高，核心板外接线简单
5. 两路i2c，一路spi（或两路spi，一路i2c，可配置）

参考了部分开源的：[diebieslave](https://github.com/DieBieEngineering/DieBieSlave) 、 [FreeECAT](https://github.com/suda-morris/FreeECAT) 、 [arducat](https://github.com/ethercat-diy/arducat)
，商业的：[Esmacat](https://www.esmacat.com/ease) 、 [easycat](https://www.bausano.net/en/hardware/ethercat-e-arduino/easycat.html) 。

综合比较下，xmc4300价格更合适、更适合工业环境。

## 成本

| 名称 | 数量 | 价格 | 链接 |
| :-----: | :-----: | :------: | :------ |
| xmc4300| 1 | 55 | https://www.ickey.cn/detail/1003022093547/XMC4300F100K256AAXUMA1.html |
| ksz8081mnxia | 2 | 14 | https://www.ickey.cn/detail/1003001221749055/KSZ8081MNXIA-TR.html |
| rj45 | 2 | 2 | https://item.szlcsc.com/149710.html |
| HY601680| 2 | 5 | https://item.szlcsc.com/14735.html |
| 其它元器件 | 1 | 10 | - |
| PCB | 1 | 3 | - |
| SMT | 1 | 16 | - |
| 合计 | - | 105 | - |

- 成本按照50台套单价计算
- XMC4300价格波动较大，最低54，最高95。
- SMT可以在立创做，单片折合约10元。第三方做一般开机费800
- 故核心板总成本在99~150之间

## 目标

只需要Arduino基础编程和电子基础知识，即可在数周时间内创造定制的IO模块。

## 引脚

![](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/IMAGE/PINOUT.png)

## 教程

1、[硬件设计](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/Tutorial_1_Hardware.md)

2、[软件编程](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/Tutorial_2_Software.md)

## 致谢

核心板硬件使用立创eda创建
