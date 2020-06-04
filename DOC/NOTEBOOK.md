在INIT之后使用
```
XMC_ECAT_WritePhy(1,0x1F,0x8110);
```
可以按照phy selection所描述的切换到LED0，硬件连接也要切换到LED0。需要改PCB。会同时更改两个phy的寄存器。默认为0x8100。只需要更改一次可以永久保存。

可以用
```
XMC_ECAT_ReadPhy(1,0x1F,&Data);
```
读取phy寄存器以验证

---之前的---

官方PDF和示例文件都有问题：

ED_SYNC_0的Source必须为B，Rising Edge，否则DC模式下通讯成功后会报sync manager watchdog/synchronization error

同样，ED_SYNC_1的Source为A，Rising Edge


---之前的---

coeappl.c 410行
```
TOBJ1C00 sSyncmanagertype = {0x04, {0x0201, 0x0403}};
```
XMC_ESC.c 266行到303行
```
void APPL_InputMapping(UINT16* pData)
{
	memcpy(pData,&(((UINT16 *)&IN_GENERIC0x6000)[1]),SIZEOF(IN_GENERIC0x6000)-2);
}

/////////////////////////////////////////////////////////////////////////////////////////
/**
\param      pData  pointer to output process data

\brief    This function will copies the outputs from the ESC memory to the local memory
            to the hardware
*////////////////////////////////////////////////////////////////////////////////////////
void APPL_OutputMapping(UINT16* pData)
{
	memcpy(&(((UINT16 *)&OUT_GENERIC0x7000)[1]),pData,SIZEOF(OUT_GENERIC0x7000)-2);
}

/////////////////////////////////////////////////////////////////////////////////////////
/**
\brief    This function will called from the synchronisation ISR 
            or from the mainloop if no synchronisation is supported
*////////////////////////////////////////////////////////////////////////////////////////
void process_app(TOBJ7000 *OUT_GENERIC, TOBJ6000 *IN_GENERIC);
void APPL_Application(void)
{
	process_app(&OUT_GENERIC0x7000, &IN_GENERIC0x6000);
}
```
