# Reykjavik

首先我们浏览一下手册，看该版本更新了哪些内容。

>OVERVIEW
>
>- Lockitall developers  have implemented  military-grade on-device
>  encryption to keep the password secure.
>- This lock is not attached to any hardware security module.
>
>DETAILS
>
>​        ...
>
>​        This is Software Revision 02. This release contains military-grade
>​                encryption so users can be confident that the passwords they enter
>​                can not be read from memory.   We apologize for making it too easy
>​                for the password to be recovered on prior versions.  The engineers
>​                responsible have been sacked.

这里说未使用 HSM-1 安全模块，但引用军工级别加密。

和之前一样，首先查看反汇编窗口的 **main** 函数。

```
4438 <main>
4438:  3e40 2045      mov    #0x4520, r14
443c:  0f4e           mov    r14, r15
443e:  3e40 f800      mov    #0xf8, r14
4442:  3f40 0024      mov    #0x2400, r15
4446:  b012 8644      call    #0x4486 <enc>
444a:  b012 0024      call    #0x2400
444e:  0f43           clr    r15
```

在 **main** 中我们看到一个陌生的函数 **enc** 。首先它将 `0xf8` 和 `0x2400` 放入 `r14` 和 `r15`中。除此之外，**main** 最后一个调用的函数  `call #0x2400` 有些奇怪，它没有显示函数名，我们查看内存窗口 0x2400 处，并没有显示该段内存，说明都是 0 。其次 **0x2400** 也作为 **enc** 的参数，其中必然有一定联系。

![image-20200524225423170](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20200524225423170.png)

动态单步运行，经过 **__do_copy_data** 拷贝初始化后，地址 0x2400 终于有了一些数据，现在我们还不清楚那些数据是什么。

![image-20200621212058278](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20200621212058278.png)

先在 **main** 函数下断点，然后单步步过 **enc**  函数，看看地址 0x2400 的数据是否发生什么变化。

![image-20200621213640387](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20200621213640387.png)

经过对比很明显，数据经过 **enc** 函数发生了改变。现在分析逐步分析一下 enc 函数，首先以地址 0x247c 起始，循环填充 0x100 字节，每字节以自加 1 递增。

```
4486 <enc>
4486:  0b12           push    r11
4488:  0a12           push    r10
448a:  0912           push    r9
448c:  0812           push    r8
448e:  0d43           clr    r13
4490:  cd4d 7c24      mov.b    r13, 0x247c(r13)
4494:  1d53           inc    r13
4496:  3d90 0001      cmp    #0x100, r13
449a:  fa23           jne    #0x4490 <enc+0xa>
449c:  3c40 7c24      mov    #0x247c, r12
```

这一段代码有一些算法，从 0x247c 提取前面写入的 0x00~0xff 字节，并且从地址 0x447c （该地址数据ASCII 为 ThisIsSecureRight?）获取一些字节。依次类推，一共循环 0x100 次，似乎在做加密或混淆。

```
44a0:  0d43           clr    r13
44a2:  0b4d           mov    r13, r11
44a4:  684c           mov.b    @r12, r8
44a6:  4a48           mov.b    r8, r10
44a8:  0d5a           add    r10, r13
44aa:  0a4b           mov    r11, r10
44ac:  3af0 0f00      and    #0xf, r10
44b0:  5a4a 7244      mov.b    0x4472(r10), r10
44b4:  8a11           sxt    r10
44b6:  0d5a           add    r10, r13
44b8:  3df0 ff00      and    #0xff, r13
44bc:  0a4d           mov    r13, r10
44be:  3a50 7c24      add    #0x247c, r10
44c2:  694a           mov.b    @r10, r9
44c4:  ca48 0000      mov.b    r8, 0x0(r10)
44c8:  cc49 0000      mov.b    r9, 0x0(r12)
44cc:  1b53           inc    r11
44ce:  1c53           inc    r12
44d0:  3b90 0001      cmp    #0x100, r11
44d4:  e723           jne    #0x44a4 <enc+0x1e>
```

最后一段代码，首先 jmp 到 0x450c 判断 **enc** 的参数之一 r14 的值（0xf8），不为 0 则跳转。然后继续一些简单的算法，将结果还是依次存放在 0x247c 起始之后，最后与 0x2400 地址起始的值进行异或，每次自增 1，最终一共循环 0xf8 次，最后函数结束返回。

```
44d6:  0b43           clr    r11
44d8:  0c4b           mov    r11, r12
44da:  183c           jmp    #0x450c <enc+0x86>
44dc:  1c53           inc    r12
44de:  3cf0 ff00      and    #0xff, r12
44e2:  0a4c           mov    r12, r10
44e4:  3a50 7c24      add    #0x247c, r10
44e8:  684a           mov.b    @r10, r8
44ea:  4b58           add.b    r8, r11
44ec:  4b4b           mov.b    r11, r11
44ee:  0d4b           mov    r11, r13
44f0:  3d50 7c24      add    #0x247c, r13
44f4:  694d           mov.b    @r13, r9
44f6:  cd48 0000      mov.b    r8, 0x0(r13)
44fa:  ca49 0000      mov.b    r9, 0x0(r10)
44fe:  695d           add.b    @r13, r9
4500:  4d49           mov.b    r9, r13
4502:  dfed 7c24 0000 xor.b    0x247c(r13), 0x0(r15)
4508:  1f53           inc    r15
450a:  3e53           add    #-0x1, r14
450c:  0e93           tst    r14
450e:  e623           jnz    #0x44dc <enc+0x56>
4510:  3841           pop    r8
4512:  3941           pop    r9
4514:  3a41           pop    r10
4516:  3b41           pop    r11
4518:  3041           ret
```

**enc** 返回后，立即调用 `call    #0x2400` ，所以说以地址 0x2400 的这段数据是作为代码来执行，但是在反汇编窗口看不到代码。结合之前的 **enc** 函数，猜测它的作用是将 0x2400 起始，且大小 0x7f 字节的数据解密出来，并执行。现在点击右上角的提供的 ASSEMBLER 页面，并将数据手动进行反汇编。若以 `ret` 作为函数结束，则可以提取以下代码。

```
<0x2400>
0b12           push    r11
0412           push    r4
0441           mov    sp, r4
2452           add    #0x4, r4
3150 e0ff      add    #0xffe0, sp
3b40 2045      mov    #0x4520, r11
073c           jmp    $+0x10
1b53           inc    r11
8f11           sxt    r15
0f12           push    r15
0312           push    #0x0
b012 6424      call    #0x2464
2152           add    #0x4, sp
6f4b           mov.b    @r11, r15
4f93           tst.b    r15
f623           jnz    $-0x12
3012 0a00      push    #0xa
0312           push    #0x0
b012 6424      call    #0x2464
2152           add    #0x4, sp
3012 1f00      push    #0x1f
3f40 dcff      mov    #0xffdc, r15
0f54           add    r4, r15
0f12           push    r15
2312           push    #0x2
b012 6424      call    #0x2464
3150 0600      add    #0x6, sp
b490 d3c0 dcff cmp    #0xc0d3, -0x24(r4)
0520           jnz    $+0xc
3012 7f00      push    #0x7f
b012 6424      call    #0x2464
2153           incd    sp
3150 2000      add    #0x20, sp
3441           pop    r4
3b41           pop    r11
3041           ret
<0x2464>
1e41 0200      mov    0x2(sp), r14
0212           push    sr
0f4e           mov    r14, r15
8f10           swpb    r15
024f           mov    r15, sr
32d0 0080      bis    #0x8000, sr
b012 1000      call    #0x10
3241           pop    sr
3041           ret
```

现在这些代码不太好动态分析，我们可以结合右上角的当前指令窗口调试。当分析后发现，这些代码的逻辑跟之前的关卡一样，首先打印提示字符串字符串，然后请求用户输入，最后判断密码是否正确。根据 0x2464 处的代码可以知道，其中 `call    #0x2464` 其实就是之前关卡的 `call INT`，参数为中断调用号。

在输入完密码后，发现在 0x2448 有一个关键判断，当判断成立则传入开锁的调用号 **0x7f**，所以只要我们 16 进制输入 d3c0（别忘了字节序），就能完成开锁。

![image-20200622111526380](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20200622111526380.png)

## 解锁

使用十六进制输入：d3c0

开锁成功！

![image-20200622113326591](https://yaseng-1251294608.cos.ap-guangzhou.myqcloud.com/image/image-20200622113326591.png)