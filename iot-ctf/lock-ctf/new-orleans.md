# New Orleans

它的每一关都是世界各地的某个城市名，随着level数量的增加，难度也在增加。现在我们开始第一个Level的挑战——New Orleans。

![20210204195632](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204195632.jpg)

> OVERVIEW
>
> - This is the first LockIT Pro Lock.
> - This lock is not attached to any hardware security module.

查看反汇编窗口，开头是一些初始化或设置工作的函数。我们选择在**main**处设置断点，在控制台窗口输入`b main`。

   ```
4438 <main>
4438:  3150 9cff      add    #0xff9c, sp
443c:  b012 7e44      call    #0x447e <create_password>
4440:  3f40 e444      mov    #0x44e4 "Enter the password to continue", r15
4444:  b012 9445      call    #0x4594 <puts>
4448:  0f41           mov    sp, r15
444a:  b012 b244      call    #0x44b2 <get_password>
444e:  0f41           mov    sp, r15
4450:  b012 bc44      call    #0x44bc <check_password>
4454:  0f93           tst    r15
4456:  0520           jnz    #0x4462 <main+0x2a>
4458:  3f40 0345      mov    #0x4503 "Invalid password; try again.", r15
445c:  b012 9445      call    #0x4594 <puts>
4460:  063c           jmp    #0x446e <main+0x36>
4462:  3f40 2045      mov    #0x4520 "Access Granted!", r15
4466:  b012 9445      call    #0x4594 <puts>
446a:  b012 d644      call    #0x44d6 <unlock_door>
446e:  0f43           clr    r15
4470:  3150 6400      add    #0x64, sp
   ```

我们浏览**main**中`call`的函数，**create_password**、**puts**、**get_password**、**check_password**和**unlock_door**函数，其中**create_password**似乎创建生成了密码，**pust**是打印提示字符串，**get_password**是请求输入密码，而**check_password**是对密码进行检查，和前一关一样**unlock_door**是解锁函数。

首先我们进入**create_password**看看。该函数正在mov.b一字节一字节的的数据到r15（0x2400）寻址的内存中，这些数据似乎是ascii码，最后一个字节以\x0结尾，合并在一起组成一个字符串。

```
447e <create_password>
447e:  3f40 0024      mov    #0x2400, r15
4482:  ff40 4600 0000 mov.b    #0x46, 0x0(r15)
4488:  ff40 6c00 0100 mov.b    #0x6c, 0x1(r15)
448e:  ff40 6c00 0200 mov.b    #0x6c, 0x2(r15)
4494:  ff40 4900 0300 mov.b    #0x49, 0x3(r15)
449a:  ff40 6600 0400 mov.b    #0x66, 0x4(r15)
44a0:  ff40 2800 0500 mov.b    #0x28, 0x5(r15)
44a6:  ff40 3900 0600 mov.b    #0x39, 0x6(r15)
44ac:  cf43 0700      mov.b    #0x0, 0x7(r15)
44b0:  3041           ret
```

我们`n`命令步过这个函数，然后查看内存0x2400的内容。`"FllIf(9"`莫非是密码？

![20210204195538](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204195538.png)

接下来在**check_password**下断点，然后`c`命令运行。在请求输入时，选择以`"FllIf(9"`作为输入。

果然，在地址0x44c2中”cmp.b @r13, 0x2400(r14)“比较1个字节，r13寄存器寻址的正是我们输入的密码，而0x2400(r14)是**create_password**函数生成的字符串`"FllIf(9"`。经过循环比较每一个字节，判断我们的输入和`"FFllIf(9"`是否相等，最后设置返回值r15。

```
44bc <check_password>
44bc:  0e43           clr    r14
44be:  0d4f           mov    r15, r13
44c0:  0d5e           add    r14, r13
44c2:  ee9d 0024      cmp.b    @r13, 0x2400(r14)
44c6:  0520           jne    #0x44d2 <check_password+0x16>
44c8:  1e53           inc    r14
44ca:  3e92           cmp    #0x8, r14
44cc:  f823           jne    #0x44be <check_password+0x2>
44ce:  1f43           mov    #0x1, r15
44d0:  3041           ret
44d2:  0f43           clr    r15
44d4:  3041           ret
```

至此，我们已经猜中密码正是**create_password**函数生成的字符串，使用`c`命令运行，测试正确。

## 解锁

在控制台窗口输入”solve“，然后在请求输入窗口以字符串输入：FllIf(9，或者16进制输入：466c6c49662839。

![20210204195833](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/20210204195833.png)

 