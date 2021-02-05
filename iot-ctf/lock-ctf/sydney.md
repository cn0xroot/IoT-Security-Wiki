# Sydney

Lockitall LockIT Pro, rev a.02，作为前一个的更新版本，我们有必要浏览一下显示的手册：

> DETAILS
>
> ​    ...
>
> ​    This is  Software Revision 02.  We have received reports  that the
> ​            prior  version of  the  lock was  bypassable  without knowing  the
> ​            password. We have fixed this and removed the password from memory.

大概意思是从内存中删除了密码，密码不会在内存中以硬编码的形式存在了。

首先查看**main**函数，显然，并没有之前**create_password**函数。**main**中仍然有**put**函数打印字符串输出，**check_password**函数检查密码是否正确，以及**INT**函数。

```
4438 <main>
4438:  3150 9cff      add    #0xff9c, sp
443c:  3f40 b444      mov    #0x44b4 "Enter the password to continue.", r15
4440:  b012 6645      call    #0x4566 <puts>
4444:  0f41           mov    sp, r15
4446:  b012 8044      call    #0x4480 <get_password>
444a:  0f41           mov    sp, r15
444c:  b012 8a44      call    #0x448a <check_password>
4450:  0f93           tst    r15
4452:  0520           jnz    #0x445e <main+0x26>
4454:  3f40 d444      mov    #0x44d4 "Invalid password; try again.", r15
4458:  b012 6645      call    #0x4566 <puts>
445c:  093c           jmp    #0x4470 <main+0x38>
445e:  3f40 f144      mov    #0x44f1 "Access Granted!", r15
4462:  b012 6645      call    #0x4566 <puts>
4466:  3012 7f00      push    #0x7f
446a:  b012 0245      call    #0x4502 <INT>
446e:  2153           incd    sp
4470:  0f43           clr    r15
4472:  3150 6400      add    #0x64, sp
```

根据静态分析，在**check_password**函数调用后，根据之前的经验，函数返回值存放在`r15`寄存器。返回后下一条指令”tst    r15“，检查`r15`寄存器也就是的值是否为零。我们查看**check_password**函数进一步分析。

```
448a <check_password>
448a:  bf90 2c41 0000 cmp    #0x412c, 0x0(r15)
4490:  0d20           jnz    $+0x1c
4492:  bf90 3c67 0200 cmp    #0x673c, 0x2(r15)
4498:  0920           jnz    $+0x14
449a:  bf90 3c65 0400 cmp    #0x653c, 0x4(r15)
44a0:  0520           jne    #0x44ac <check_password+0x22>
44a2:  1e43           mov    #0x1, r14
44a4:  bf90 6b63 0600 cmp    #0x636b, 0x6(r15)
44aa:  0124           jeq    #0x44ae <check_password+0x24>
44ac:  0e43           clr    r14
44ae:  0f4e           mov    r14, r15
44b0:  3041           ret
```

可以看到，**check_password**中一共有4个`cmp`指令，将源操作数与`r15`寻址的内存中的内容比较，且目的操作数之后都是以两个字节偏移递增。若是经过4次`cmp`比较，`r15`将会被赋值为0x1，也就是能通过密码检查。

需要注意的是这里使用的是`cmp`，与上一等级的`cmp.b`相比，少了`.b`扩展名也叫助记符，所以操作数不再是一字节（byte）；`cmp`虽然省略了`.w`扩展名，但其相当于`cmp[.w]`，操作数是一个字（word）。在msp430用户指南中解释，**如果不使用扩展名，指令是一个字指令**。

![20210204200318](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204200318.jpg)

此时，我们需要知道`r15`所寻址的内存地址，我们回到**main**函数中可以发现，调用**check_password**函数前，将`sp` 当前栈指针移动到`r15`，`sp`的值我们还不知道，那我们开始调试吧。

```
4438 <main>
...
444a:  0f41           mov    sp, r15
444c:  b012 8a44      call    #0x448a <check_password>
...
```

使用`break 444a`命令，在地址0x444a处设置断点，查看`sp`的值以及`sp`指向的栈空间的内容。首先会调用请求输入**get_password**函数，我们输入”test“，进行测试。

输入完毕后，在此`c`命令运行，执行道地址0x444a后中断，我们可以查看`sp`的值，栈空间（sp）的内容正是我们输入密码。

![20210204200337](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204200337.png)

继续设置断点`break check_password`，然后`c`运行，进入**check_password**分析，我们已经知道`r15`寻址的内存中内容正是我们输入的密码，所有将`cmp`的源操作数提取出来，依次是0x412c、0x673c、0x653c、0x636b，这应该就是正确密码了。

```
448a:  bf90 2c41 0000 cmp    #0x412c, 0x0(r15)
4492:  bf90 3c67 0200 cmp    #0x673c, 0x2(r15)
449a:  bf90 3c65 0400 cmp    #0x653c, 0x4(r15)
44a4:  bf90 6b63 0600 cmp    #0x636b, 0x6(r15)
```

这里有一个问题，将以上16进制数组合起来：412c673c653c636b，若是直接作为输入肯定是会出错的，因为，我们还忽略了字节序的问题，MSP430的是小端存储（little-endian），所以我们需要将其高字节与低字节进行交换。

关于字节序，大家都不陌生，维基百科中关于[字节序](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F)中介绍：

>字节的排列方式有两个通用规则。例如，一个多位的整数，按照存储地址从低到高排序的字节中，如果该整数的最低有效字节（类似于[最低有效位](https://zh.wikipedia.org/wiki/最低有效位)）在最高有效字节的前面，则称**小端序**；反之则称**大端序**。

## 解锁

勾选16进制编码输入复选框，以16进制编码输入：2c413c673c656b63。

![20210204200416](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/微信图片_20210204200416.jpg)

解锁成功！

![20210204200425](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/微信图片_20210204200425.png)