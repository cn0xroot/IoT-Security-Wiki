## LoRa通信

准备LoRa模块两个（SX1262  686、915MHz  160mW  LoRa无线模块）

![image-20210108153314749](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210108153314749.png)



首先进入WOR模式进行配置，设置信道为65，频率为915.125MHz，发送方和接收发的频率信号必须一致

**发送方**

![image-20210113113901417](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210113113901417.png)

**接收发**

![image-20210113113817259](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210113113817259.png)

**相互通信**

设置为一般模式后，可以通过串口输入书，发射模块会启动无线发射，接收模块无线接收功能打开，收到无线数据后会通过 TXD 引脚输出。

![image-20210113115151002](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210113115151002.png)

