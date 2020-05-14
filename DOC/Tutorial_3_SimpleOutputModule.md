## 简易输出模块

熟悉DEMO并跑起来之后，我们需要规划实际应用。这里我们以一个24V的16路NPN输出模块举例（光耦或ULN2803等输出）。

如果直接拷贝的DEMO项目，需要修改项目文件夹下的.project文件，name标签和整个文件夹名称修改一下再从dave软件导入。

## ESC部分

1、使用excel编辑/SSC/XMC_ESC.xlsx，修改DATA（去除INPUT，保留一个0x01 UINT的OUT_GENERIC作为输出。

2、使用SSC TOOLS编辑/SSC/XMC_ESC.esp，修改Vendor id、Product Code、Device Name等信息

3、TOOL->Application->Import选择刚刚修改好的xlsx文件

4、Project->Create New Slave Files，指定目录和ESI文件名称，单击Start生成Slave程序和设备描述文件

5、解压Patch.zip到当前目录，覆盖。

## ESC修改

由于这个从站只有输出，所以需要修改一下从站application。

1、编辑/SSC/src/XMC_ESC.c，屏蔽226行（因为没有IN_GENERIC，所以没有TxPDO，InputSize大小即为0。

2、屏蔽270行的APPL_InputMapping内容，原因同上。

3、（可选）如果是32位及以上的数据，则需要修改OutputMapping的UINT16 *改为UINT32 *，并同时修改ecatappl.c的调用内容等。

4、修改290行、293行的process_app，去除TOBJ6000 *IN_GENERIC，原因同上。

## Main程序修改

1、在Init_GPIO内对各相关引脚初始化，我还加了.output_Strength=XMC_GPIO_OUTPUT_STRENGTH_WEAK,驱动能力弱似乎对散热有帮助。

2、修改process_app的定义，去除IN_GENERIC。

3、在Process_app中对引脚关联，如：digitalWrite(	P2_5	,(OUT_GENERIC->OUT >> 2)	 & outenable);

（对OUT_GENERIC.OUT移位2，即获取bit2作为最后一位，与outenable(0x0或0x1)求与门。编辑Mapping映射可以在EXCEL中拉出来，比较方便）

## 调试及下载

如果之前烧写过其它Vendor id或product id的程序，请用jlink commander手动擦除一下，这些内容存储在EEPROM里，可能在下载时不会完全擦除


