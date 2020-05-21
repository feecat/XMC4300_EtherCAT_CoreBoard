## 硬件设计过程
EtherCAT从站芯片参考[https://www.ethercat.org/download/documents/ESC_Overview.pdf](https://www.ethercat.org/download/documents/ESC_Overview.pdf)。
主要分为三大类：
- 纯从站，需配上位机，如lan9252
- 嵌入内核，如xmc4800
- 软核+实时硬核，如amic110

设计之初需要考虑好不好购买和技术支持，毕竟不是大厂批量采购，技术支持全靠谷歌。这样筛选下来就剩：
- lan9252
- amic110
- xmc4800

9252+STM32的方案很多了，国内至少十几家在做，也有热心网友如[丁丁的个人网站](https://www.hexcode.cn/article/5e3ee9a835616641b2daef97)将整个流程清晰完整地展现出来。但我觉得门槛仍有些高。

amic110的资料实在太少太复杂，我连官方教程（基于TI CCS IDE）配置都没有通过。

XMC4800资料多，EVM板价格也比较适中，买了个测试了挺久。缺点也是有的，外挂phy太占用引脚了。

我的硬件设计是工作中自学的，断断续续做一些小电路板测试。xmc4300最纠结的就是PHY选择。硬件设计可能会有不少问题，恳请各位指正。

## 设计目标

xmc4800 relax板的phy是bcm5241，但它需要的外围器件较多，相关资料也少。参考了倍福的phy selection后，结合购买渠道决定用microchip的ksz8081mnx，在立创、淘宝、贸泽、云汉上都能买到，价格不贵，外围电路也更简单。

由于4层板打样价格较高，xmc4300又是lqfp100封装，所以初期目标就是2层PCB，打样了一套后跑起来了，但有两个问题：

1、 [phy手册](https://download.beckhoff.com/download/Document/io/ethercat-development-products/an_phy_selection_guidev2.6.pdf)说明ksz8081需要对phy的LED模式做更改，否则可能在link up时丢帧。我不明白如何修改，打样出来也没有问题，后期就把两个0r电阻去掉了，但仍然保留了phy的串口。

2、 官方evm板对phy reset做了buffer，我没有做，只是加了个电容。

由于工作较忙搁置了一段时间，之后再用的话发现做出来的核心板没有意义，24v供电处理占用了1/3的PCB空间。偶然在一个设备上看到自制的fb1111核心板，便想尝试做个尺寸兼容的板子试试。

综上，目标是以下几点：
- xmc4300＋ksz8081（已验证）
- 双层PCB，单面贴片，元器件尽可能少
- 尺寸和fb1111一致，引脚尽量兼容（spi从模式，这样原先使用fb1111的只需要修改程序即可兼容）
- 尽量使用通用、易采购的元器件，尽量使用0603封装。

## 设计过程

pinout参考了官方的xmc4300 ethercat模板，我觉得官方选择的引脚应该是可靠的，所以直接照搬。

ksz8081是在Google搜到的原理图，结合evm板组合一下，剩下就是简单连线了。

网口电容用了4个2kv耐压电容，并与PE相连，这部分参考[原始原理图](https://www.infineon.com/dgdl/Infineon-Board_User_Manual_XMC4700_XMC4800_Relax_Kit_Series-UM-v01_02-EN.pdf?fileId=5546d46250cc1fdf01513f8e052d07fc)。

由于板子空间有限，放不下tvs、电感、电解电容了，所以供电部分尽量简化，防护主要做在底板上。

网口做了个4pin ph插座，不焊rj45可以改m12插座，或焊排针把核心板隐藏到内部（焊排针可以保留网口LED指示）。

第一版dc模式且双网口都在工作时，1117会略有发热，估算约50度，第二版加了一个小焊盘辅助散热，目前这一版还没有打样，但改动不大。

此外，实际测试时发现外围设备对电源的影响还是蛮大的，建议单独给核心板一个5v供电。dcdc水也很深，我用的是tps5430并严格按照官方layout，使用良好。外围设备有用5v的可以再来一个，3.3v的单独挂一个1117也ok。我理解的电气隔离并不需要隔离dcdc，让它们互不干扰即可。

## 温度问题

XMC4300F100F256标称最高运行温度是85度，K256是125度。实际使用时相对于LAN9252/ET1100，可以明显感觉XMC系列发热更高。

通过[XMC_SCU_GetTemperatureMeasurement()](https://www.infineonforums.com/threads/5861-XMC4300-Onchip-Temperature)获取片上核心温度传感器，无散热片自然散热，室温25摄氏度，空运行半小时后，在XMC48RELAX板上读取到**42**摄氏度，在本项目核心板上读取到**52**摄氏度，考虑到芯片尺寸和LAYOUT差异，应在正常范围。下一版需加大底层铜皮连接。

XMC48RELAX的5V电流约为350ma（含板载jlink），本项目核心板5V电流约为300ma，接近1117的自然散热上限（400ma左右），不建议对核心板上的3.3v做其它用处。此外定制底板时，如空间足够，建议将1117整合在底板内并加大散热焊盘以改善散热表现。




