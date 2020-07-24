## Hardware design
EtherCAT slave chip reference[https://www.ethercat.org/download/documents/ESC_Overview.pdf](https://www.ethercat.org/download/documents/ESC_Overview.pdf)。
Mainly divided into three categories:
- Pure slave, need to be equipped with host computer, such as lan9252
- Embedded core, such as xmc4800
- Soft core + real-time hard core, such as amic110

At the beginning of the design, you need to consider whether to purchase and technical support. After all, it is not a large-scale purchase from a large factory. The technical support depends on Google. This filter is left:
- lan9252
- amic110
- xmc4800

There are many 9252+STM32 solutions, and there are also enthusiastic netizens such as[丁丁的个人网站](https://www.hexcode.cn/article/5e3ee9a835616641b2daef97)Show the entire process clearly and completely. But I think the programming threshold is still a bit high.

The information of amic110 is too little and too complicated. I didn't even pass the official tutorial (based on TI CCS IDE) configuration.

XMC4800 has a lot of information, and the price of EVM board is relatively moderate. I bought it and tested it for a long time. There are also disadvantages. The external phy takes up too much pins.

I learned my hardware design by myself at work, doing some small circuit board tests on and off. There may be a lot of problems in the hardware design, so please correct me.

## Design Target

The phy of the xmc4800 relax board is bcm5241, but it requires more peripheral devices and less relevant information. After referring to Beckhoff's phy selection, we decided to use TI DP83848 in combination with purchase channels, which can be bought on Lichuang, Taobao, Mouser, and Yunhan. The price is not expensive and the peripheral circuit is simpler.

Due to the high price of 4-layer board proofing, the xmc4300 is packaged in lqfp100, so the initial goal is 2-layer PCB. After proofing a set, it works (KSZ8081 was used at the time), but there was serious frame loss when testing long-distance communication , Re-layout has no effect.

I happened to see a beckhoff fb1111 core board on a device, and I wanted to try to make a size-compatible board.

In summary, the goals are the following:
- xmc4300＋DP83848
- Double-layer PCB, single-sided SMT, as few components as possible
- The size is the same as fb1111, and the pins are as compatible as possible (spi slave mode, so that the original fb1111 only needs to modify the program to be compatible)
- Try to use common and easy-to-purchase components, and try to use 0603 package.

## Design

The pinout refers to the official xmc4300 ethercat template. I think the official pin should be reliable, so I just copied it.

DP83848 refers to the official design, but some resistors are removed due to space constraints.

The network port capacitor uses two 2kv withstand voltage capacitors and is connected to PE. Refer to this part[original schematic](https://www.infineon.com/dgdl/Infineon-Board_User_Manual_XMC4700_XMC4800_Relax_Kit_Series-UM-v01_02-EN.pdf?fileId=5546d46250cc1fdf01513f8e052d07fc)。

Due to the limited board space, tvs, inductors, and electrolytic capacitors cannot be placed, so the power supply part is simplified as much as possible, and the protection is mainly done on the bottom plate.

HR913550A and HR911105A are Pin2Pin compatible, both can be used, and the pitch is 2.54, which can be directly soldered to the expansion board.

The 1117 has a large heat, and the heat dissipation pad is added but it is still hot, but it is used normally.

In addition, It is recommended to separately supply a 5v power supply to the core board. I use tps5430 and strictly follow the official layout, and it works well. I understand that electrical isolation does not require isolation of dcdc, so that they do not interfere with each other.

## Temperature Problem

XMC4300F100F256 maximum operating temperature is 85 degrees, and K256 is 125 degrees. Compared with LAN9252/ET1100 in actual use, it is obvious that the XMC series heats up higher.

By [XMC_SCU_GetTemperatureMeasurement()](https://www.infineonforums.com/threads/5861-XMC4300-Onchip-Temperature)Get on-chip core temperature sensor, natural heat dissipation, room temperature 25 degrees Celsius, after half an hour of dry running, it reads **48** degrees on the XMC48RELAX board, and **54** degrees on the core board of this project, considering the chip The difference between size and LAYOUT should be in the normal range (about 700mW, ~Tamp+25℃). The next edition needs to increase the bottom copper connection. (The power is about twice higher than the same main frequency STM32 or XMC4500, which should be due to the EtherCAT core)

The 5V current of XMC48RELAX is about 290ma (including onboard jlink), the 5V current of the coreboard of this project is about 210ma, and the natural heat dissipation limit of the 5 to 3.3 1117 is about 400ma. It is not recommended to use the 3.3v on the core board for other purposes. In addition, when customizing the bottom plate, if there is enough space, it is recommended to integrate the 1117 into the bottom plate and increase the thermal pad to improve the heat dissipation performance.




