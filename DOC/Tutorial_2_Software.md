## Software Program

Software programming is mostly based on[Official example](https://www.infineon.com/cms/en/product/promopages/aim-mc/dave_downloads.html), Download ETHCAT_SSC_XMC43, after unzip, found: Getting Started - XMC4300_Relax_EtherCat_APP_Slave_SSC Example_V3.3.pdf

Using LCEDA to design hardware and DAVE to design software does not involve copyright issues, which is also the main consideration at the beginning of the design. But the debugger needs jlink, and you can choose the following three methods:

- chinese Jlink V9 Mini copy. ~50
- KIT_XMC45_RELAX_LITE_V1, Break it is XMCLINK, only Infineon MCU can be programmed, but there are still education version authorization issues. ~100
- KITXMCLINKSEGGERV1TOBO1, Only Infineon MCU can be programmed. ~700

Except[DAVE4](https://infineoncommunity.com/dave-download_ID645)In addition, you need to prepare additional SSC_V5i12.zip, or[EtherCAT Slave Stack Code Tool.exe](https://github.com/feecat/XMC4300_EtherCAT_CoreBoard/blob/master/DOC/EtherCAT%20Slave%20Stack%20Code%20Tool.exe)

Since I have not studied the C language systematically, I may have misunderstood the basic knowledge of pointers, functions, etc., please correct me.

## Start

DAVE's APP needs to be compared[Official video](https://www.youtube.com/watch?v=zBh1E93ktUo), I think it's easier to use. Don’t forget to generate Code after adding APP and every time you modify it.

## Things to note

1、The video tutorial is relatively old, you actually need to refer to the PDF tutorial in the official example (ETHCAT_SSC_XMC43).

2、The current version of the official example (V3.3) mentioned that there is a BUG in SSC 5.12 that needs to be manually modified after generating the SSC code (coeappl.c, line 410).

3、The current version of APPL has been changed to memcpy, and #include "SSC/Src/XMC_ESCObjects.h" in main.c. Although I don't understand it, I think it's pretty easy to use. I have patched both of these points, which can be decompressed and overwritten in my ssc/patch.zip. (It is better not to modify the project name XMC_ESC in SSC, and the XML file name can be specified separately)

4、If you are creating a new DAVE project yourself, you must strictly follow the official PDF matching version.

5、The IO device also needs to turn off the output when the ethercat bus is disconnected, which can be added in main:
```
//Output Enable
  uint8_t outenable;
  void process_stopoutput(){outenable=false;}
  void process_startoutput(){outenable=true;}
```
And add it around lines 153 and 170 of /SSC/src/XMC_ESC.c
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

Add `& outenable` or if judgment after the output variable.

6、IO is generally a 16-bit UINT variable that represents 16 IOs. If you take a bit for mapping, you can take it like this (shift and 0x0 or 0x1 for AND gate):

```
(OUT_GENERIC->OUT_0 >> 0) & outenable
(OUT_GENERIC->OUT_0 >> 1) & outenable
```



## Peripheral equipment selection

There are a total of 25 GPIOs on the core board, which can be configured as 2 SPI+1 I2C or 2 I2C+1 SPI.

I didn't lead out the analog pins of the single-chip microcomputer. On the one hand, it is difficult to lay out the 2-layer board, and on the other hand, the industry needs to be well isolated. Consider communicating with devices such as ADS1015/MCP4728+LM258.

For digital IO expansion, I recommend I2C +PCF8574/PCF8575. The IO module does not require strict speed. Mount the front and rear optocouplers or MOS tubes (TBD62783), or smart high-side chips such as VN808.

The error callback in the official I2C APP is not very useful. Once a communication abnormality occurs, I2C may be stuck. It is recommended to associate tx rx callback in I2C interrupt with i2c_txrx_callback, the specific code is as follows:

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

The following code is used in actual communication. I2C is recommended to be added under Mainloop, not in process app:

```
    I2C_MASTER_Receive(&I2C_MASTER_0,true,0x4F,received_data,2,true,true);
    i2c_wait();
```
And in Dave APP (Dave\Generated\I2C_MASTER\I2C_Master.C) the following lines:

```
Line 323:    while (!XMC_USIC_CH_TXFIFO_IsEmpty(handle->channel)){}
Line 445:    while (XMC_USIC_CH_GetTransmitBufferStatus(handle->channel) == XMC_USIC_CH_TBUF_STATUS_BUSY){}
Line 1052:    while (XMC_USIC_CH_GetTransmitBufferStatus(handle->channel) == XMC_USIC_CH_TBUF_STATUS_BUSY){}
```

Add the following patch: (count can be changed according to the actual situation), otherwise the EMC test will probably not pass.

```
uint32_t count = 0;
while (!XMC_USIC_CH_TXFIFO_IsEmpty(handle->channel)){
	count++;
	if (count > 1000){
		return;
	}
}
```
