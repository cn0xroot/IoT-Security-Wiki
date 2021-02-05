# 智能锁具安全风险

## 物理层

锁具在安装之后，存在结构薄弱点，直接暴露锁簧或锁体结构的薄弱点，甚至可以通过外部拨动锁体；

![image23](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image24.jpg)

![图片1](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/sdgdsry87908.jpg)

![媒体1_clip](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/1_clip.gif)

如图为一种专门针对市面上大部分智能锁以及指纹锁一类的插片开启工具，工具用法简单，直接在外门面板缝中插入该工具，碰到带动杆就能直接转开了，既简单实用又廉价。

锁具的整体材料强度不达标准，无法抵抗一般的暴力开锁手段，关键结构点没有械加固，锁芯不具备空转结构或分体设计等功能。

![image23](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image23.png)





![TB2XPjucLiSBuNkSnhJXXbDcpXa_!!25616172](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/TB2XPjucLiSBuNkSnhJXXbDcpXa_!!25616172.jpg)





![暴力开锁_clip](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/clip.gif)

如上图尽管官方宣传该款锁的U形环可以抵抗10T液压钳，但在锁体关键连接部位采用一般铝合金材质，使用常见工具短时间内即可暴力开锁。

## 电路层

智能锁内部集成电路在PCB设计、布线、物料选型、抗干扰设计上存在缺陷，以传统电路设计为主导，未充分考虑基于物联网的安全性设计，导致存在安全隐患。

没有做好充分的电磁防护设计，使用外部磁场或强电压可以进行信号干扰。

![image25](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image25.jpg)

通过电磁注入，产生重置信号，实现开锁。（来源 DPLSLAB）

某款共享单车电路设计和机械构造存在问题，可以从外部截断输入锁体的电源线，在用一个高电压的脉冲电压作为输入电源，即可开锁。其原因在于电路设计和电机控制芯片选型存在缺陷，没有做充分的过载保护和断路保护，使控制锁柱运动的电机异常工作。

## 硬件通信层

锁具内部电路的各接口和引脚信号存在未经加密或验证的输入输出信号，存在可能暴露内部逻辑的调试信号，可以通过I2C、SPI、UART、JTAG等协议进行数据窃取和劫持，获取锁具芯片访问权和固件程序；

![5b8feecd02a39](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/5b8feecd02a39.jpg)
spi通信

![5b8fef1f084b2](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/5b8fef1f084b2.jpg)
I2C通信

![5b8ff2ea3e6d5](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/5b8ff2ea3e6d5.jpg)
UART通信

![微信截图_20181019165050](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/20181019165050.png)
逻辑分析仪抓取spi通信数据分析



![微信图片_20180930114820](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/20180930114820.jpg)
通过 SBW 和 GDB 调试锁具芯片。

芯片未做加密保护，容易被拆解提取固件。

![5b352394427b3](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/5b352394427b3.jpg)

拆下芯片通过调试器直接读取锁具固件。


## 固件层

芯片固件存在逻辑算法漏洞，没有进行严格的代码审计，存在代码层的安全漏洞。

下面是对某款声波锁的固件逆向过程。

![IMG_3505](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/IMG_3505.jpg)



![IMG_3506](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/IMG_3506.jpg)

![IMG_3507](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/IMG_3507.jpg)

进行固件逆向分析，可以得到密码验证算法，通过数学计算可以算出相应的密钥。

![1538279364702](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/1538279364702.png)




![image46](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image46.jpg)

声音通过傅里叶算法，实现AD相互转换，利用逆向的算法生成开锁声波。

对于固件更新，没有进行签名和完整性检验，可以对固件进行修改和重打包。

空中升级固件，加密算法强度不够，抵御不了高阶的侧信道攻击。

## 移动APP层

存在被逆向、反编译源码的可能；
可能存在底层代码漏洞和敏感信息泄露；
APP自身业务存在逻辑问题，如异常开锁、异常登录等。

## 身份识别层

### 指纹模块

指纹模块的算法安全性不达标，对错误指纹、非活体指纹、假指纹等的识别率偏低；

![媒体3[00_00_02][20180930-120632-0]](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/[00_00_02][20180930-120632-0].jpg)

一般光学或电容指纹模块，没有进行活体检测，容易被假指纹欺骗。



指纹数据能被提取，通过指纹复制，制作假指纹；



![图片2](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/20180226135.jpg)

通过简单材料可以伪造出指纹模型。

指纹模板和数据存在被盗取或替换的隐患；
指纹模块本身存在被替换的问题。

### RFID

RFID 卡片可以被远程嗅探、复制；
RFID 卡片可以被近程伪造，锁具读卡器被欺骗

### 密码键盘

密码键盘是否存在被偷窥风险
可能存在不同按键声音可能，存在泄露密码的风险
不具备主动探测的防拆机制，可能存在安装物理攻击设备窃取密码的风险

### 其他认证方式

虹膜、人脸识别存在被高清图像，3D打印等非活体手段绕过的风险
声音包括超声波存在被录制重放的风险

## 通信层

### 无线电通信

WiFi通信可能存在中间人攻击；

![image80](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image80.jpg)

![image84](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image84.png)

![image83](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image83.png)



![序列 01_clip](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/01_clip.gif)

某款保险柜存在被 wifi 中间人劫持攻击的漏洞。



蓝牙与APP通信连接过程的加密和验证强度不够，容易进行蓝牙抓包、劫持、重放等攻击；

![5b9fe0fd21e76](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/5b9fe0fd21e76.jpg)



![vlcsnap-2018-09-30-12h03m29s219](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/vlcsnap-2018-09-30-12h03m29s219.png)

![媒体2_clip](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/2_clip.gif)

某款蓝牙采用AES加密，数据不改变，无签名校验，抓包重放即可开锁 

无线遥控开锁在信号广播发送过程中存在被抓包、解码、重放破解的风险；

无线通信加密算法未考虑侧信道防护，存在被侧信道分析破解的可能；

芯片层没有做有效的数模信号隔离，可能导致关键密钥信息通过无线载波发射出来，并通过侧信道破解算法。

### 云端通信

APP与云端的通信没有使用安全通信，容易被抓包、劫持；

客户端与服务端没有对数据进行有效的合法性校验，容易通过中间人攻击；

采用 GMS 通信存在被伪基站劫持的风险；

## 云服务层

云端服务器存在安全漏洞，可能有SQL注入、XSS跨站脚本攻击、CSRF跨站请求伪造、越权访问等等安全问题，导致用户数据被泄露；

![image71](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image71.jpg)

![image72](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/suo3vcf/image72.jpg)

某款锁可以遍历云端存储的所有开锁密码和用户信息。