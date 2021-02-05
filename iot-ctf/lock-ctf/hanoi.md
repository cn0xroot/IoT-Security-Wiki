# Hanoi

首先我们浏览一下手册，看该版本更新了哪些内容。

>OVERVIEW
>
>- This lock is attached the the LockIT Pro HSM-1.
>- We have updated  the lock firmware  to connect with the hardware
>  security module.
>
>DETAILS
>
>​    ...
>
>​    LockIT Pro Hardware  Security Module 1 stores  the login password,
>​            ensuring users  can not access  the password through  other means.
>​            The LockIT Pro  can send the LockIT Pro HSM-1  a password, and the
>​            HSM will  return if the password  is correct by setting  a flag in
>​            memory.

这里说该锁连接了 HSM-1（硬件安全模块），也就是说密码在的HSM中存储，LockIT Pro可以向HSM-1发送密码，再由HSM返回结果，而我们无法直接访问它。

接下来，我们查看反汇编窗口，看看它到底做了什么，首先是**main**函数，**main**中除了调用**login**函数外，并无其他功能。

```
4438 <main>
4438:  b012 2045      call    #0x4520 <login>
443c:  0f43           clr    r15
```

查看**login**函数，其中调用了一些函数，**put**函数、**getsn**函数请求用户输入、**test_password_valid**函数根据函数名知道其作用是测试密码有效性。

```
4520 <login>
4520:  c243 1024      mov.b    #0x0, &0x2410
4524:  3f40 7e44      mov    #0x447e "Enter the password to continue.", r15
4528:  b012 de45      call    #0x45de <puts>
452c:  3f40 9e44      mov    #0x449e "Remember: passwords are between 8 and 16 characters.", r15
4530:  b012 de45      call    #0x45de <puts>
4534:  3e40 1c00      mov    #0x1c, r14
4538:  3f40 0024      mov    #0x2400, r15
453c:  b012 ce45      call    #0x45ce <getsn>
4540:  3f40 0024      mov    #0x2400, r15
4544:  b012 5444      call    #0x4454 <test_password_valid>
4548:  0f93           tst    r15
454a:  0324           jz    $+0x8
454c:  f240 f100 1024 mov.b    #0xf1, &0x2410
4552:  3f40 d344      mov    #0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call    #0x45de <puts>
455a:  f290 c600 1024 cmp.b    #0xc6, &0x2410
4560:  0720           jne    #0x4570 <login+0x50>
4562:  3f40 f144      mov    #0x44f1 "Access granted.", r15
4566:  b012 de45      call    #0x45de <puts>
456a:  b012 4844      call    #0x4448 <unlock_door>
456e:  3041           ret
4570:  3f40 0145      mov    #0x4501 "That password is not correct.", r15
4574:  b012 de45      call    #0x45de <puts>
4578:  3041           ret
```

通过开头的手册提示，我们知道密码在HSM中，我们看看**test_password_valid**函数做了哪些操作。

```
4454 <test_password_valid>
4454:  0412           push    r4
4456:  0441           mov    sp, r4
4458:  2453           incd    r4
445a:  2183           decd    sp
445c:  c443 fcff      mov.b    #0x0, -0x4(r4)
4460:  3e40 fcff      mov    #0xfffc, r14
4464:  0e54           add    r4, r14
4466:  0e12           push    r14
4468:  0f12           push    r15
446a:  3012 7d00      push    #0x7d
446e:  b012 7a45      call    #0x457a <INT>
4472:  5f44 fcff      mov.b    -0x4(r4), r15
4476:  8f11           sxt    r15
4478:  3152           add    #0x8, sp
447a:  3441           pop    r4
447c:  3041           ret
```

通过简单分析可以看到，地址0x446e调用了**INT**函数，根据LockIT Pro[用户手册](https://microcorruption.com/manual.pdf)第7页，可以看到**INT**函数的作用是将中断号压入栈中，然后调用系统中断，在`call    #0x457a <INT>`上方正是将“0x7d”压入栈中，所以这是调用**INT 0x7d**中断。

![20210204201256](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204201451.jpg)

在LockIT Pro[用户手册](https://microcorruption.com/manual.pdf)第9页中说明，调用INT 0x7d中断后，若是密码正确，将会在某一位置上覆盖标志。

![20210204201340](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204201340.jpg)

我们回到**login**函数中，使用`break 4548`命令，将断点设置在调用**test_password_valid**后下一条指令位置，然后`c`运行，运行过程中调用请求输入函数**getns**，根据调用前参数，我们不知道正确的密码，依旧填“test”。在IO交互界面提示，密码在8~16字符之间。

![image-20210204201857126](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204201857126.png)

**getns**有两个参数，请求输入密码的内存缓冲区地址（0x2400），以及接收的最大字节数（0x1c），所以尽管上面提示密码在8~16字符之间，我们还是可以输入28（0x1c）个字符。

```
4534:  3e40 1c00      mov    #0x1c, r14
4538:  3f40 0024      mov    #0x2400, r15
453c:  b012 ce45      call    #0x45ce <getsn>
```

待在0x4548断下以后，我们单步分析。在不知道密码的情况下，寄存器**r15**（r15存放test_password_valid函数的返回值）的值为0是必然的，所以执行`jz $+0x8`，跳过`mov.b #0xf1, &0x2410`，随机打印提示测试密码是否有效字符串。

```
4544:  b012 5444      call    #0x4454 <test_password_valid>
4548:  0f93           tst    r15
454a:  0324           jz    $+0x8
454c:  f240 f100 1024 mov.b    #0xf1, &0x2410
4552:  3f40 d344      mov    #0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call    #0x45de <puts>
```

随即`cmp.b`比较源操作数**0xc6**与地址0x2410的内容，其中目的操作数是绝对寻址模式。下一条指令`jne `，若是不相等则跳过**unlock_door**函数，ret返回。所以这是解锁的关键代码，猜测之前调用INT 0x7d时，若密码正确覆盖的正是这一地址。

```
455a:  f290 c600 1024 cmp.b    #0xc6, &0x2410
4560:  0720           jne    #0x4570 <login+0x50>
4562:  3f40 f144      mov    #0x44f1 "Access granted.", r15
4566:  b012 de45      call    #0x45de <puts>
456a:  b012 4844      call    #0x4448 <unlock_door>
456e:  3041           ret
4570:  3f40 0145      mov    #0x4501 "That password is not correct.", r15
4574:  b012 de45      call    #0x45de <puts>
4578:  3041           ret
```

关于寻址模式，在msp430手册中介绍，针对源操作数的七个寻址模式和针对目的操作数的四个寻址模可在完整地址空间寻址。

![image-20210204201911618](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204201911618.png)

还记得之我们输入的密码在内存的位置吗？密码在地址0x2400起始的缓冲区中，和0x2410只相差0x10字节，而我们可以输入0x1c个字符，此时我们可以通过“溢出”0x10字符的范围，覆盖到地址0x2410中。

![image-20210204201945234](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204201945234.png)

## 解锁

我们已经确定了解决方案，下面开始解锁。

填充0x10字节数据，在其末尾加上0xc6即可。

以16进制编码输入：41414141414141414141414141414141c6。

![image-20210204201956417](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204201956417.png)

成功解锁！

![image-20210204202003384](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204202003384.png)