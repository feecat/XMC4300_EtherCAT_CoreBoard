# XMC4300_EtherCAT_CoreBoard

[中文](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/README_CN.md)

Ethercat slave core board based on xmc4300, same size as beckhoff fb1111

![](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/IMAGE/IMG01.jpg)

I have manufactured 30 pcs and all works well.I think i have finish it. Next we will made more 100 pcs in china. I will update the process.

![](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/IMAGE/Manufactured.jpg)


## Origin

Ethercat is an industrial real-time Ethernet, Widely used in automation equipment and CNC machine tools. It has an open sources protocol stack, good real-time performance, and low price. It is widely used in drives, control panels and IO devices, and its popularity is increasing step by step.

On many devices, R&D personnel are tired of dealing with on-site problems, and IO modules are often divided into two extremes:

- Based on Modbus, only one stm32f103 is needed, the communication rate is limited, the stability is also poor (generally optocoupler or triode output), and the price is low.
- Ethercat-based modules, such as Beckhoff el series io modules, are expensive and have excellent stability.

But even if the higher-priced Beckhoff module is used, intricate wiring is still required in the electrical cabinet. For standardized equipment, the more worker wiring the higher the failure rate. Some manufacturers began to customize ethercat modules. They are mainly based on the following two platforms:

* beckhoff et1100
* microchip lan9252

In actual applications, both platforms require external MCUs, and programming is difficult, and the development environment is not easy to build.

Using the XMC4300 core board, you can quickly develop solenoid valve groups, panel buttons, IO modules, etc.

This project is still in the testing stage and is not responsible for the finished product.

## Selection
I refer to the Beckhoff official selection manual, fb1111 schematics, etc., and finally finalized xmc4300+DP83848 as the core board. There are several reasons:
1. xmc4300 comes with cortex-m4 core, no need for external MCU
2. Infineon's Dave4 software is very complete, ready to use after decompression
3. The size is compatible with Beckhoff fb1111, double-layer PCB, single-sided SMT, no plug-in components except the network port and pin header. The cost of ordering 100 batches is about RMB130
4. High stability, simple external wiring of the core board
5. Two i2c, one spi (or two spi, one i2c, configurable)

Refer to some open source:[diebieslave](https://github.com/DieBieEngineering/DieBieSlave) 、 [FreeECAT](https://github.com/suda-morris/FreeECAT) 、 [arducat](https://github.com/ethercat-diy/arducat)
,commercial:[Esmacat](https://www.esmacat.com/ease) 、 [easycat](https://www.bausano.net/en/hardware/ethercat-e-arduino/easycat.html) 。

Under comprehensive comparison, the price of xmc4300 is more suitable and suitable for industrial environment.

## Cost

| Name | Quantity | Price | Link |
| :-----: | :-----: | :------: | :------ |
| xmc4300| 1 | 55 | https://www.ickey.cn/detail/1003022093547/XMC4300F100K256AAXUMA1.html |
| DP83848IVV | 2 | 16 | https://www.ickey.cn/detail/100300320411267/DP83848IVVX__point__NOPB.html |
| HR913550A | 2 | 8 | https://item.szlcsc.com/174889.html |
| Others | 1 | 10 | - |
| PCB | 1 | 3 | - |
| SMT | 1 | 16 | - |
| Total | - | 108 | - |

- The cost is calculated according to the unit price of 50 sets
- The price of XMC4300 fluctuates greatly, the lowest is 54 and the highest is 95.
- SMT can be made in szjlc, and the contract is 10 yuan per piece. The third party does a general startup fee of 800
- Therefore, the total cost of the core board is between 99 and 150. Considering material loss, etc., generally 130 is the actual cost

## Target

You only need basic Arduino programming and basic electronic knowledge to create a customized IO module in a few weeks.

## Pinout

![](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/IMAGE/PINOUT.png)

## Tutorial

1、[Hardware Design](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/Tutorial_1_Hardware.md)

2、[Software Program](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/Tutorial_2_Software.md)

3、[Simple Output Module](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/Tutorial_3_SimpleOutputModule.md)

## Acknowledgement

The core board hardware is created using LCEDA
