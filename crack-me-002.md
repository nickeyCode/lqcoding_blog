# crack - me 002 笔记

## 尝试

![image-20221203175407244](D:\coding\blog\crack-me-002.assets\image-20221203175407244.png)



## 搜索字符串寻找突破口

![image-20221203175506305](D:\coding\blog\crack-me-002.assets\image-20221203175506305.png)

## 汇编分析

![image-20221203175701129](D:\coding\blog\crack-me-002.assets\image-20221203175701129.png)



上图可见 `0040258B| je afkayas.1.4025E5| zf=1 ?`是关键跳转

je: 当ZF标志位为1时候跳转 (ZF :zero Flag 零标志位)

### 字符串对比汇编

```
00402530| call ebx                               |
00402532| push eax                               |
00402533| call dword ptr ds:[<&__vbaStrCmp>]     | 对比字符串
00402539| mov esi,eax                            |
0040253B| lea edx,dword ptr ss:[ebp-0x20]        |
0040253E| neg esi                                |
00402540| lea eax,dword ptr ss:[ebp-0x18]        |
00402543| push edx                               |
00402544| sbb esi,esi                            | ？？
00402546| lea ecx,dword ptr ss:[ebp-0x1C]        |
00402549| push eax                               |
0040254A| inc esi                                |
0040254B| push ecx                               |
0040254C| push 0x3                               |
0040254E| neg esi                                |
00402550| call dword ptr ds:[<&__vbaFreeStrList>]|
00402556| add esp,0x10                           |
00402559| lea edx,dword ptr ss:[ebp-0x28]        |
0040255C| lea eax,dword ptr ss:[ebp-0x24]        |
0040255F| push edx                               |
00402560| push eax                               |
00402561| push 0x2                               |
00402563| call dword ptr ds:[<&__vbaFreeObjList>]|
00402569| add esp,0xC                            |
0040256C| mov ecx,0x80020004                     |
00402571| mov eax,0xA                            | A:'\n'
00402576| mov dword ptr ss:[ebp-0x64],ecx        |
00402579| test si,si                             |
0040257C| mov dword ptr ss:[ebp-0x6C],eax        |
0040257F| mov dword ptr ss:[ebp-0x54],ecx        |
00402582| mov dword ptr ss:[ebp-0x5C],eax        |
00402585| mov dword ptr ss:[ebp-0x44],ecx        |
00402588| mov dword ptr ss:[ebp-0x4C],eax        |
0040258B| je afkayas.1.4025E5                    | zf=1 ?
```

上面汇编是字符串对比的主要代码,在运行`00402533| call dword ptr ds:[<&__vbaStrCmp>]`的时候栈顶是:

![image-20221203191638047](D:\coding\blog\crack-me-002.assets\image-20221203191638047.png)



可以看到[ebp-4]是我们输入的验证码,[ebp-8]是最终验证码,至于[ebp-8]是怎么来的我们稍后再往上分析,现在先分析如果对比字符串.

上面的代码去掉中间的部分无效代码可以得出:

```
00402530| call ebx                               |
00402532| push eax                               |
00402533| call dword ptr ds:[<&__vbaStrCmp>]     | 对比字符串
00402539| mov esi,eax                            |
0040253E| neg esi                                |
00402544| sbb esi,esi                            |
0040254A| inc esi                                |
0040254E| neg esi                                |
00402579| test si,si                             |
0040258B| je afkayas.1.4025E5                    | zf=1 ?
```

eas存放着`vbaStrCmp`的结果,通过`mov esi,eax  `放到esi中.接下来这一段我本来以为也是混淆代码,但是查了一下应该是判断0的经典汇编:

```
0040253E| neg esi                                |
00402544| sbb esi,esi                            |
0040254A| inc esi                                |
```

#### neg

对第一操作数取反操作再加1(等于是取第一个操作数的相反数) 修改CF操作位,如果第一操作数(原操作数)是0,就CF = 0

```
IF DEST  0 
THEN CF  0 
ELSE CF  1; 
FI;
DEST  - (DEST)
```

所以

如果esi = -1 , neg esi 后就会是1 , CF会是1 (因为原操作数不等于0),

如果esi = 0 , neg esi 后就会是0 , CF会是0 (因为原操作数等于0),

如果esi = 1 , neg esi 后就会是-1 , CF会是1 (因为原操作数不等于0),

#### sbb

带借位整数减法 ,意思是 第一个操作数减去第二个操作再减CF标志位

```
DEST  DEST -SRC - CF
```

由于`sbb esi esi` 第一个操作数和第二个操作数相同,所以最终esi的结果会是 -CF,

#### inc

操作数自增1,相当于i++



所以当以下代码的运行后:

```
0040253E| neg esi
00402544| sbb esi,esi   
0040254A| inc esi  
```

esi结果只会是 0 或者 1, 在这时候,就能判断esi最初是否等于0了

而后面的

```
0040254E| neg esi                                |
00402579| test si,si                             |
```

其实也算多余的代码,就是通过test判断esi是否为0

#### test

两个操作数进行`与`运算,改变符合位(SF),零位(ZF),奇偶位(PF),不改变原来两个操作数

`test esi esi`当esi为0 时, ZF位为1 ,所以jz会跳转跳过了正确的弹出代码:

```
0040258B| je afkayas.1.4025E5            | zf=1 ?
0040258D| push afkayas.1.401B80          | 401B80:L"You Get It"
00402592| push afkayas.1.401B9C          | 401B9C:L"\r\n"
00402597| call edi                       |
00402599| mov edx,eax                    |
0040259B| lea ecx,dword ptr ss:[ebp-0x18]|
0040259E| call ebx                       |
004025A0| push eax                       |
004025A1| push afkayas.1.401BA8          | 401BA8:L"KeyGen It Now"
004025A6| call edi                       |
004025A8| lea ecx,dword ptr ss:[ebp-0x6C]|
```



### 验证码生成

从前面可知验证码样式是`AKA-xxxxxxxxx` 我们只需要知道`xxxxx`是怎么生成.

![image-20221203225002789](D:\coding\blog\crack-me-002.assets\image-20221203225002789.png)

这里开始是对输入name进行操作:

1. 获取输入长度之后与0x17CF8相乘,看有无溢出

   ```
   00402415| call dword ptr ds:[<&__vbaLenBstr>]        | get name len
   0040241B| mov edi,eax                                |
   0040241D| mov ecx,dword ptr ss:[ebp-0x18]            |
   00402420| imul edi,edi,0x17CFB                       |
   00402426| push ecx                                   |
   00402427| jo afkayas.1.4026BE                        | OF=1 跳转
   ```

   imul: 对两个有符号操作数执行乘法。

   edi的结果是`0x0D64D3`

2. 接下来是用name[0] 加上 edi,同样地判断一下溢出

   ```
   0040242D| call dword ptr ds:[<&rtcAnsiValueBstr>]    |
   00402433| movsx edx,ax                               |
   00402436| add edi,edx                                |
   00402438| jo afkayas.1.4026BE                        |
   ```

   我为什么会知道`rtcAnsiValueBstr`是获取name[0]呢?,是观察的结果

   ![image-20221203230241274](D:\coding\blog\crack-me-002.assets\image-20221203230241274.png)

   这时候edi的值变成了`0x0D653F`

3. 在把edi的十六进制变成十进制string:

   ```
   0040243E| push edi                                   |
   0040243F| call dword ptr ds:[<&__vbaStrI4>]          |
   00402445| mov edx,eax                                |
   00402447| lea ecx,dword ptr ss:[ebp-0x20]            |
   ```

   ![image-20221203230521884](D:\coding\blog\crack-me-002.assets\image-20221203230521884.png)

所以最终的验证码是`AKA-877887`

