## 软件编程过程

软件编程绝大部分基于[官方示例](https://www.infineon.com/cms/en/product/promopages/aim-mc/dave_downloads.html)，下载ETHCAT_SSC_XMC43，解压后找到Getting Started - XMC4300_Relax_EtherCat_APP_Slave_SSC Example_V3.3.pdf

使用立创EDA设计硬件，DAVE设计软件，不牵涉版权问题，这也是设计之初主要考虑的一点。调试器建议采购一个KIT_XMC45_RELAX_LITE_V1，约100元，类似STM32的NUCLEO系列，掰断即为XMCLINK，仅可对英飞凌单片机进行编程，不牵涉盗版jlink版权问题。

除了[DAVE软件](https://infineoncommunity.com/dave-download_ID645)外，您需要额外准备SSC_V5i12.zip，或[EtherCAT Slave Stack Code Tool.exe](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/EtherCAT%20Slave%20Stack%20Code%20Tool.exe)。

由于我没有系统性学习过C语言，对指针、函数等基础知识可能理解有误，恳请各位指正。

## 开始

DAVE的APP需要对照[官方视频](https://www.youtube.com/watch?v=zBh1E93ktUo)熟悉一下，我觉得还是比较好用的。加完APP每次修改完需要Generate Code不要忘了。

## 需要注意的内容

1、视频教程比较老，实际需要参考官方示例（ETHCAT_SSC_XMC43）里的PDF教程。

2、目前版本的官方示例（V3.3）中提到SSC 5.12中有一个BUG需要在生成SSC代码后手动修改（coeappl.c，410行）。

3、目前版本的APPL已改为memcpy方式，并在main.c中#include "SSC/Src/XMC_ESCObjects.h"，虽然不太理解，但我觉得还是蛮好用的。对这两点我都做了patch，可以在我的ssc/patch.zip中解压覆盖。

4、如果是自己新建DAVE项目，则需严格按照官方PDF匹配的版本做。

5、总线断开时IO设备也需要关闭输出，可以在main中添加：
```
//Output Enable
  uint8_t outenable;
  void process_stopoutput(){outenable=false;}
  void process_startoutput(){outenable=true;}
```
并在/SSC/src/XMC_ESC.c的153、170行左右添加
```
void process_startoutput();
UINT16 APPL_StartOutputHandler(void)
{
  process_startoutput();
  return ALSTATUSCODE_NOERROR;
}

void process_stopoutput();
UINT16 APPL_StopOutputHandler(void)
{
  process_stopoutput();
  return ALSTATUSCODE_NOERROR;
}
```

之后输出的变量后加上`& outenable`即可。

## 外围设备选择

核心板上一共有25个GPIO，可以配置为2路SPI+1路I2C或2路I2C+1路SPI。

单片机自带的模拟引脚我都没有引出，一方面是2层板layout困难，另一方面是工业上需要做好隔离。可以考虑用ADS1015/MCP4728+LM258之类的器件通讯。

数字IO扩展我比较推荐I2C方式+PCF8574，IO模块对速率要求并不严格。前后挂光耦或MOS管，或是VN808之类的智能高边芯片即可。

官方的I2C APP中error回调不太好用，一旦发生通讯异常I2C很可能会卡死。建议将I2C的interrupt中tx rx callback关联到i2c_txrx_callback上，具体代码如下：

```
void i2c_txrx_callback(void){i2c_completion = 1;}

void i2c_wait(){
  uint32_t waitcnt = 0;
    while(i2c_completion == 0){
    waitcnt++;
    if (waitcnt>3000){
      waitcnt=0;
      I2C_MASTER_AbortReceive(&I2C_MASTER_0);
      I2C_MASTER_AbortTransmit(&I2C_MASTER_0);
      I2C_MASTER_Init(&I2C_MASTER_0);
      i2c_completion = 1;
      }
    };
    waitcnt=0;
    i2c_completion = 0;
}
```

实际通讯时使用如下代码：

```
    I2C_MASTER_Receive(&I2C_MASTER_0,true,0x4F,received_data,2,true,true);
    i2c_wait();
```

