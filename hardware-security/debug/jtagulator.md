# JTAGulator调试

## 前言

在线调试（OCD,On-Chip Debugging）接口可以提供对目标设备的芯片级控制，是工程师、研究人员、黑客用来提取程序固件代码或数据、修改存储器内容或改变设备操作的主要途径。如果你熟悉硬件电路或嵌入式系统，那么你肯定知道JTAG（Joint Test Action Group）和UART（Universal Asynchronous Receiver/Transmitter）可以说是使用最多的串行通信接口。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/1.png)

JTAG规范没有标准的接口定义，所以你可以在各种PCB硬件上中见到4-20pin的JTAG Header，而且各个引脚的功能定义也无法确定，这给调试工作造成了很大的麻烦，下图列举了4种接口定义，有ARM公司的定义，有ST公司的定义等等。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/2.png)



![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/3.png)

那如何知道引脚定义呢？传统的做法是通过逻辑分析仪做信号分析来解决，但是这样既费时费力又容易出错。于是自动化识别JTAG接口的设备便诞生了，例如有JTAGulator、JTAGenum、JTAG Finder、JTAG Pinout Tool等等，目前来说最好用的还是JTAGulator。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/4.png)

JTAGulator是一个开源硬件工具，可用于识别目标设备上测试点、过孔或元件焊盘的OCD连接，进而可以使用Attify Badge接线并读取程序固件。



## JTAG规范

JTAG 在上一节 JTAG 调试中做过简单的介绍，JTAG 是联合测试工作组（Joint Test Action Group）的简称，是在名为标准测试访问端口和边界扫描结构的IEEE的标准1149.1的常用名称。此标准用于验证设计与测试生产出的印刷电路板功能。

1990年JTAG正式由IEEE的1149.1-1990号文档标准化，在1994年，加入了补充文档对边界扫描描述语言（BSDL）进行了说明。从那时开始，这个标准被全球的电子企业广泛采用。边界扫描几乎成为了JTAG的同义词。

JTAG的主要三大功能：

1. 下载器，即下载程序固件到设备FLASH芯片。
2. DEBUG，类似于医生的听诊器，可以探听芯片内部的错误。
3. 边界扫描，可以访问芯片内部的型号逻辑状态，还有芯片引脚的状态等等。

在JTAG接口中，最常用的信号有五个，分别是TCK / TMS / TDO / TDI / TRST，其中4个是输入信号接口，另外1个是输出信号接口。

JTAG最初是用来对芯片进行测试的，其基本原理是在器件内部定义一个TAP（Test Access Port）并规定TAP状态机的行为，通过专用的测试工具进行内部节点进行测试。JTAG测试允许多个器件通过JTAG接口串联在一起，形成一个JTAG链，能实现对各个器件分别测试。现在，JTAG接口还常用于实现ISP（In-System Programmable），对Flash等器件进行编程。下面，我们介绍一下这5个接口：

- Test Clock Input (TCK)
  TCK在IEEE1149.1标准里是强制要求的。TCK为TAP的操作提供了一个独立的、基本的时钟信号，TAP的所有操作都是通过这个时钟信号来驱动的。
- Test Mode SelectionInput (TMS)
  TMS信号在TCK的上升沿有效。TMS在IEEE1149.1标准里是强制要求的。TMS信号用来控制TAP状态机的转换。通过TMS信号，可以控制TAP在不同的状态间相互转换。
- Test Data Input (TDI)
  TDI在IEEE1149.1标准里是强制要求的。TDI是数据输入的接口。所有要输入到特定寄存器的数据都是通过TDI接口一位一位串行输入的（由TCK驱动）。
- Test Data Output (TDO)
  TDO在IEEE1149.1标准里是强制要求的。TDO是数据输出的接口。所有要从特定的寄存器中输出的数据都是通过TDO接口一位一位串行输出的（由TCK驱动）。
- Test Reset Input (TRST)
  这个信号接口在IEEE 1149.1标准里是可选的，并不是强制要求的。TRST可以用来对TAPController进行复位（初始化）。因为通过TMS也可以对TAP Controll进行复位（初始化）。所以有四线JTAG与五线JTAG之分。

JTAG接口可以一对一的使用，也可以组成菊花链的一对多拓扑结构，两种拓扑结构如下图所示。多核的芯片，其芯片内部已经接成了菊花链的形式。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/5.png)



## 制作JTAGulator

JTAGulator官方售价$169，某宝代购价人民币1500元左右，自己DIY成本可以控制在每块板子人民币500元以内。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/6.png)

在[JTAGulator的官网](http://www.grandideastudio.com/jtagulator/)可以下载制作JTAGulator所需的所有资料，GERBER光绘文件，BOM元件列表等等。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/7.png)

把GERBER光绘文件发往工厂打样PCB，等待2-3天即可到货，然后购买BOM表中的电子元器件。PCB我是在嘉立创打样，电子元件在立创商城购买。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/8.png)

按照官网提供的PCB装配图将电子元器件用烙铁和焊锡焊接在PCB电路板上面。焊接完成后用洗板水清理松香和助焊剂的残留，在上电之前仔细检查是否存在虚焊和短路现象。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/9.png)

通过Mini USB数据线连接电脑，打开“设备管理器”会看到一个串行通信端口。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/10.png)

硬件测试没问题后在Github下载[程序固件源码](https://github.com/grandideastudio/jtagulator/releases)，给设备烧录程序固件需要安装芯片的烧录工具[Propeller Tool](https://www.parallax.com/downloads/propeller-tool-software-windows-spin-assembly)。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/11.png)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/12.png)

将下载的程序固件源码解压出来，打开里面的"JTAGulator.eeprom"文件，点击"Load ARM"就可以烧录固件进板子。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/13.png)

烧录完毕用串口调试工具打开JTAGulator对应的COM端口，波特路115200，按一下回车键，返回数据即可，"H"查看帮助，"I"查看版本。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/14.png)



## JTAG识别

用一块STM32开发板做测试，假装不知道它的引脚定义，板子单独供电，先用万用表测出GND引脚，然后把需要识别的引脚和GND引脚用杜邦线接到JTAGulator上。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/15.png)

输入"J"进入JTAG模式，"V"设置电压，"I"是IDCODE扫描，然后设置通道范围，成功识别出引定义。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/16.png)



## UART识别

拿一台网件的路由器做测试，一样的步骤，先用万用表测出GND引脚，然后把剩下的脚接到JTAGulator上。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/17.png)

成功识别出TX和RX引脚。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/18.png)

还可以用JTAGulator通过UART串口调试路由器。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/19.png)



## 总结

JTAGulator能够快速识别哪些引脚可能是JTAG，并找出这些引脚的顺序，在这里感谢作者Joe Grand开源这款硬件工具。

没有条件DIY的朋友可以尝试另一个比较便宜的方案JTAGenum，使用Arduino Nano开发板或树莓派烧录程序即可 。

JTAGulator和JTAGenum两者之间的差别是，JTAGulator硬件内置了电平转换和输入保护，就软件方面而言JTAGulator提供UART扫描，JTAGenum目前并不支持，JTAGulator还提供扫描未记录的JTAG接口功能，这也是JTAGenum目前不支持的功能。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/2020/4/4/20.png)



