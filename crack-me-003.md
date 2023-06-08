![image-20230122170142848](D:\coding\blog\crack-me-003.assets\image-20230122170142848.png)![image-20230122170153037](D:\coding\blog\crack-me-003.assets\image-20230122170153037.png)

这个程序有讲个破解点,第一是去除Nag,第二是找到验证算法.

去除Nag这个完全没有经验,但是网上搜了一下发现有一个C4去除法:
[160个CrackMe003详细分析 - 『脱壳破解区』 - 吾爱破解 - LCG - LSG |安卓破解|病毒分析|www.52pojie.cn](https://www.52pojie.cn/thread-882601-1-1.html)

# 第一步

找到VB程序入口处:

![image-20230122171746796](D:\coding\blog\crack-me-003.assets\image-20230122171746796.png)

转到内存地址查看是VBHeader的结构体:

![image-20230122171838599](D:\coding\blog\crack-me-003.assets\image-20230122171838599.png)

然后转到0x04067D4 + 4C . 就是0x0406820 . 转到Dword
![image-20230122173141245](D:\coding\blog\crack-me-003.assets\image-20230122173141245.png)

可以看到两个相同的数据块 , 这就是两个对话框的数据结构,里面标记了对话框的序号和启动顺序:

![image-20230122173236483](D:\coding\blog\crack-me-003.assets\image-20230122173236483.png)

我们只需要修改两个对话框的启动循序就可以去除Nag:

![image-20230122174330477](D:\coding\blog\crack-me-003.assets\image-20230122174330477.png)

保存修改.



# 破解注册算法

打算随便出入点东西然后观察尝试搜索字符串,没想到发现一个坑

![image-20230122174611143](D:\coding\blog\crack-me-003.assets\image-20230122174611143.png)

就是这两个输入框只能输入数字! 输入英文字母就会出现这个错误提示,而这个提示找不到字符串.

正确的错误提示是这样:

![image-20230127171115197](D:\coding\blog\crack-me-003.assets\image-20230127171115197.png)



这时候我们再搜索字符串就能找到对应的断点:

![image-20230127171634582](D:\coding\blog\crack-me-003.assets\image-20230127171634582.png)



# 开始分析算法

## 先说结果

这是一个浮点型数值验证的算法,会用到FPU寄存器和运算符

1. 获取第一个输入框输入的长度(value1)

2. value2 = value1 * 0x15B38
3. 获取第一个输入框输入的第一个字符并转换为16进制(value3)
4. value4 = value2 + value3
5. value5 = 由于前面所有运算都是16进制数值,所以value4转换为十进制浮点数
6.  value6 = value5+ (10/5)
7. value7 = value6 * 3
8. value8 = value7 -2
9. value9 = value8 - (-15)
10. value10 = 获取第二个输入框的输入并转换成浮点数
11. value11 = value9 / value10
12. 判断value11 是否等于1

以上就是算法的整个计算过程和判断过程,接下来看对应的汇编

## 汇编分析

从新输入数字name和code:

![image-20230211174152247](D:\coding\blog\crack-me-003.assets\image-20230211174152247.png)



### 第一部分汇编:

![image-20230211174824738](D:\coding\blog\crack-me-003.assets\image-20230211174824738.png

![image-20230211194640543](D:\coding\blog\crack-me-003.assets\image-20230211194640543.png)

最终获得第一个阶段的值`800041`

### 第二部分汇编

![image-20230211200127155](D:\coding\blog\crack-me-003.assets\image-20230211200127155.png)

```
004082E9   .  FF15 74B14000 call dword ptr ds:[<&MSVBVM50.__vbaR8Str>]
```

__vbaR8Str是vba函数,把str转换成浮点数 ---->(800041)

``` 
004082EF   .  D905 08104000 fld dword ptr ds:[0x401008]
```

`fld `指令是将 **[0x401008]** (=10)的值压入 FPU 寄存器堆栈 ---->(10)

> FPU 寄存器 ??
>
> FPU 不使用通用寄存器 (EAX、EBX 等等)。反之，它有自己的一组寄存器，称为寄存器栈 (register stack)。数值从内存加载到寄存器栈，然后执行计算，再将堆栈数值保存到内存。
>
> FPU 有 8 个独立的、可寻址的 80 位数据寄存器 R0〜R7，如下图所示，这些寄存器合称为寄存器栈。FPU 状态字中名为 TOP 的一个 3 位字段给出了当前处于栈顶的寄存器编号。例如，在下图中，TOP 等于二进制数 011，这表示栈顶为 R3。在编写浮点指令时，这个位置也称为 ST(0)（或简写为 ST）。最后一个寄存器为 ST(7)。
>
> ![浮点数据寄存器栈](D:\coding\blog\crack-me-003.assets\4-1Z52GK304P6-1676126680983-3.gif)
>
> 
> 如同所想的一样，入栈（push）操作（也称为加载）将 TOP 减 1，并把操作数复制到标识为 ST(0) 的寄存器中。如果在入栈之前，TOP 等于 0，那么 TOP 就回绕到寄存器 R7。
>
> 出栈（pop）操作（也称为保存）把 ST(0) 的数据复制到操作数，再将TOP加1。如果在出栈之前，TOP 等于 7，则 TOP 就回绕到寄存器 R0。

```
004082FE   .  D835 0C104000 fdiv dword ptr ds:[0x40100C]
```

`fdiv`指令是将 ST(0) 除以 **[0x40100C]**(=5)，结果存储到 ST(0)  ---->(2)

```
0040831E   .  DEC1          faddp st(1),st
```

`faddp`指令将 ST(0) 与 ST(i) 相加，结果存储到  ST(i)，并弹出寄存器堆栈 ---->(800043)

```
0040832A   .  DD1C24        fstp qword ptr ss:[esp]
```

`fstp`指令是将 ST(0) 复制到 **[esp]**，并弹出寄存器堆栈  ---->(800043)

```
0040832D   .  FF15 48B14000 call dword ptr ds:[<&MSVBVM50.__vbaStrR8>]     ;  msvbvm50.__vbaStrR8
```

`__vbaStrR8` 把浮点数转换成字符串 ---->(String:800043)

### 第三部分汇编

![image-20230211203740310](D:\coding\blog\crack-me-003.assets\image-20230211203740310.png)

```
004083F5   .  FF15 74B14000 call dword ptr ds:[<&MSVBVM50.__vbaR8Str>]     ;  msvbvm50.__vbaR8Str
```

把上面的String字符串再转换成浮点数并压入堆栈 --->(800043)

```
004083FB   .  DC0D 10104000 fmul qword ptr ds:[0x401010]
```

`fmul`指令是将 ST(0) 乘以 **[0x401010]**(=3)，结果存储到 ST(0) --->(2400129)

```
00408404   .  DC25 18104000 fsub qword ptr ds:[0x401018]
00408414   .  DD1C24        fstp qword ptr ss:[esp]
00408417   .  FF15 48B14000 call dword ptr ds:[<&MSVBVM50.__vbaStrR8>]     ;  msvbvm50.__vbaStrR8
```

`fsub`指令是 ST(0) 减去 **[0x401018]**(=2)，结果存储到 ST(0) --->(2400127)

并且转换成字符串

### 第四部分汇编

![image-20230211205422283](D:\coding\blog\crack-me-003.assets\image-20230211205422283.png)

```
004084DF   .  FF15 74B14000 call dword ptr ds:[<&MSVBVM50.__vbaR8Str>]     ;  msvbvm50.__vbaR8Str
004084E5   .  DC25 20104000 fsub qword ptr ds:[0x401020]
004084FB   .  FF15 48B14000 call dword ptr ds:[<&MSVBVM50.__vbaStrR8>]     ;  msvbvm50.__vbaStrR8
```

`__vbaR8Str` 把字符串转换回浮点数(2400127)压入FPU堆栈

`fsub` 指令是ST(0) 减去 **[0x401018]**(=-15)，结果存储到 ST(0) ---> (2400142)

`__vbaStrR8`把浮点数转换成字符串---> (2400142)

### 第五部分汇编

![image-20230211204342092](D:\coding\blog\crack-me-003.assets\image-20230211204342092.png)

```
004085C8   .  FF15 18B14000 call dword ptr ds:[<&MSVBVM50.__vbaHresultChec>;  msvbvm50.__vbaHresultCheckObj
004085CE   >  8B45 E8       mov eax,dword ptr ss:[ebp-0x18]                ;  获取验证码
004085D1   .  50            push eax
004085D2   .  FF15 74B14000 call dword ptr ds:[<&MSVBVM50.__vbaR8Str>]     ;  msvbvm50.__vbaR8Str
004085D8   .  8B4D E4       mov ecx,dword ptr ss:[ebp-0x1C]
```

`__vbaR8Str` 把验证码输入框String转换成浮点型并压入FPU堆栈--->(234567890)

```
004085DB   .  DD9D 1CFFFFFF fstp qword ptr ss:[ebp-0xE4]
```

`fstp` 将 ST(0) 复制到 **[ebp-0xE4]**，并弹出寄存器堆栈--->(234567890)

```
004085E2   .  FF15 74B14000 call dword ptr ds:[<&MSVBVM50.__vbaR8Str>]     ;  msvbvm50.__vbaR8Str
```

把上一部分的值压入堆栈 --->(2400142)

```
004085F1   .  DCBD 1CFFFFFF fdivr qword ptr ss:[ebp-0xE4]                  ;  相除
```

`fdivr`将 **[ebp-0xE4] **(=234567890) 除以 ST(0) (=2400142)，结果存储到 ST(0) 

就是浮点数除运算 234567890/2400142 = 97.73.....

```
0040861A   .  DC1D 28104000 fcomp qword ptr ds:[0x401028]                  ;  对比结果是否等于1
```

`fcomp` 指令比较 ST(0) 与 **[0x401028]   **(=97.73.....)，并弹出寄存器堆栈

其实这两步就是判断两个浮点数是否相等. 所以我们正确的验证码应该是 `2400142 `



## 验证

![image-20230211213824313](D:\coding\blog\crack-me-003.assets\image-20230211213824313.png)





# 总结

这题的知识点有两到三个:

1. 去除Neg , 用到C4去除法
2. vba转换函数
   1. __vbaR8Str
   2. __vbaStrR8
3. 浮点数运算
   1. `fld`: 将 **m32real** 压入 FPU 寄存器堆栈
   2. `fdiv`: 将 ST(0) 除以 **m32real**，结果存储到 ST(0)
   3. `faddp` :  将 ST(0) 与 ST(i) 相加，结果存储到  ST(i)，并弹出寄存器堆栈
   4. `fstp`: 将 ST(0) 复制到 **m32real**，并弹出寄存器堆栈
   5. `fsub`:  从 ST(0) 减去 **m32real**，结果存储到 ST(0)
   6. `fdivr`: 将 **m32real** 除以 ST(0)，结果存储到 ST(0)
   7. `fcomp`:  比较 ST(0) 与 **m32real**。

