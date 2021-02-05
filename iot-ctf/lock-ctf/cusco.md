# Cusco

首先我们浏览一下手册，看该版本更新了哪些内容。

>OVERVIEW
>
>- We have fixed issues with passwords which may be too long.
>- This lock is attached the the LockIT Pro HSM-1.
>
>DETAILS
>
>​        ...
>
>​        This is Software Revision 02. We have improved the security of the
>​                lock by  removing a conditional  flag that could  accidentally get
>​                set by passwords that were too long.

也就是说这一关卡还是使用了HSM-1，还删除了条件标志来提高安全性，从而避免密码太长而覆盖。

查看反汇编窗口，首先**main**函数，跟之前Hanoi一样，这里只调用了**login**函数。

```
4438 <main>
4438:  b012 0045      call    #0x4500 <login>
```

**login**函数代码如下，调用了**puts**、**getsn**、**test_password_valid**以及**unlock_door**，看起来没有什么特别之处。

```
4500 <login>
4500:  3150 f0ff      add    #0xfff0, sp
4504:  3f40 7c44      mov    #0x447c "Enter the password to continue.", r15
4508:  b012 a645      call    #0x45a6 <puts>
450c:  3f40 9c44      mov    #0x449c "Remember: passwords are between 8 and 16 characters.", r15
4510:  b012 a645      call    #0x45a6 <puts>
4514:  3e40 3000      mov    #0x30, r14
4518:  0f41           mov    sp, r15
451a:  b012 9645      call    #0x4596 <getsn>
451e:  0f41           mov    sp, r15
4520:  b012 5244      call    #0x4452 <test_password_valid>
4524:  0f93           tst    r15
4526:  0524           jz    #0x4532 <login+0x32>
4528:  b012 4644      call    #0x4446 <unlock_door>
452c:  3f40 d144      mov    #0x44d1 "Access granted.", r15
4530:  023c           jmp    #0x4536 <login+0x36>
4532:  3f40 e144      mov    #0x44e1 "That password is not correct.", r15
4536:  b012 a645      call    #0x45a6 <puts>
453a:  3150 1000      add    #0x10, sp
453e:  3041           ret
```

我们从何下手呢？还记得上一关卡Hanoi时，我们知道当我们输入内容过长时，可能覆盖它后面的内存数据，所以，首先看看**getsn**函数接收的输入长度以及缓冲区的位置。

```
4514:  3e40 3000      mov    #0x30, r14
4518:  0f41           mov    sp, r15
451a:  b012 9645      call    #0x4596 <getsn>
```

也就是说最大可接收0x30字节的输入，以及它存放在当前栈空间中，尝试输入若干个“A”，测试一下看看它是否会覆盖什么。

![图片](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/640)

从内存中可以看到确实是0x30字节，但也没有什么重要的信息，继续运行。

![123324234534](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/1233242345345.png)

密码错误，但是我们发现控制台窗口有一个报错，地址未对齐。原因是，我们查看上面的寄存器窗口，`pc`寄存器是0x4141，这不是“A”的ascii码，正是我们输入的密码，地址0x4141并未16为对齐。

当我们在反汇编窗口向上拉，overvriteen表示原本的代码被覆盖了。基于以上说明，我们输入的内容更改了`pc`寄存器，并且已经溢出了当前的栈帧外，覆盖了我们的代码。

![123123123123](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/123123123123.png)

很明显，这是一个栈溢出漏洞，我们在**login**函数的`ret`指令处下断点，并运行到`ret`指令的地址0x453e。

![123123123](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/123123123.png)

`sp`寄存器指向的正是**login**的返回地址，关于`ret`指令的作用，msp430手册中说明，将当前`sp`指向的栈中的返回地址移动到`pc`寄存器，也就是相当于`pop`和 `jmp `的操作，所以可改变程序的执行流程。

![1231233123](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/1231233123.png)

至于为什么会覆盖返回地址呢？首先我们查看login的栈的空间结构，如右图，栈是从高地址向低地址递进。当**main**函数`call login`时，首先将当前`pc`的下一条指令地址放入堆栈栈中，接着**login**函数第第一条指令`add #0xfff0, sp`用来开辟**0x10**字节大小的栈空间。但是由于我们输入了**0x30**字节的输入，当前栈空间不足以存放这么多数据，就会向高地址溢出，覆盖返回地址以及代码。

![123123123](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/123123123.webp)

既然返回地址被我们输入的数据覆盖，那么我们就利用这一点，来达到劫持程序流程的目的。首先我们确定返回地址的偏移，返回地址在**0x43fe**的位置，而我们的密码在**0x43ee**，所以它的偏移在+**0x10**的位置。确定偏移后，需要填充返回地址，返回时执行我们希望执行的代码，既然我们的目的是解锁，那么不如将**unlock_door**解锁函数的地址0x4446作为填充。

```
4446 <unlock_door>
4446:  3012 7f00      push    #0x7f
444a:  b012 4245      call    #0x4542 <INT>
444e:  2153           incd    sp
4450:  3041           ret
```

## 解锁

一切准备之后，开始进行栈溢出漏洞利用，别忘了返回地址的字节序。

使用十六进制输入：414141414141414141414141414141414644。

![image-20210204205140657](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204205140657.png)

解锁成功！

![image-20210204205146444](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20210204205146444.png)

