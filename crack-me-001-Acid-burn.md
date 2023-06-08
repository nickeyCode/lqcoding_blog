crackme 001-1 正式开始~

# 确认程序类型

用PE工具检查一下是 Delphi 的32bit程序

![image-20221115200731557](D:\coding\blog\image-20221115200731557.png)



# 寻找突破口

尝试运行一下

![image-20221119153600350](D:\coding\blog\image-20221119153600350.png)

这个程序有两个地方需要破解,我们先看第一个 Serial / Name

![image-20221119153525406](D:\coding\blog\image-20221119153525406.png)

随便输入Name和Serial之后出现"Sorry , The serial is incorect " 的提示,所以我决定用x64dbg搜索一下字符串寻找切入点

![image-20221119154012339](D:\coding\blog\image-20221119154012339.png)

结果找到两个位置 , 分别添加断点再运行一下

![image-20221119154113767](D:\coding\blog\image-20221119154113767.png)

运行结果果然停在了一个断点,简单看了一下代码感觉应该是切入点

![image-20221119154341865](D:\coding\blog\image-20221119154341865.png)

# 尝试分析

关键的跳转是

```
0042FAF8| 8B55 F0    | mov edx,dword ptr ss:[ebp-0x10]                    |
0042FAFB| 8B45 F4    | mov eax,dword ptr ss:[ebp-0xC]   
0042FAFE| E8 F93EFDFF| call acid burn.4039FC |
0042FB03| 75 1A      | jne acid burn.42FB1F  |
```

```
jne: 判断ZF标志位,当ZF==0的时候跳转到目标行
```

所以 上面代码的意思应该是 函数burn.4039FC 传入两个参数[ebp-0x10]和[ebp-0xC] 输出结果在ZF标志位.

而我们查看下传入参数的值是什么:

![image-20221119162605488](D:\coding\blog\image-20221119162605488.png)

可以看到[ebp-0x10]是我们输入的Serial , [ebp-0xC] = CW-8856-CRACKED

```
$-10     0012F998                 009D4E50                      "12345678"
$-C      0012F99C                 009D4E0C                      "CW-8856-CRACKED"
```

这里我们可以验证一下如果输入是"CW-8856-CRACKED"的话会不会正确

![image-20221119163053526](D:\coding\blog\image-20221119163053526.png)

看到提示成功了,如果是一次性的破解应该是已经可以完结了,但是通过crackme来学习逆向和汇编的话,我们可以再深究一下验证码是如何生成的,在通过分析结果写一个注册机

# 还原过程

从这里开始往上翻找 push ebp来找函数的开头,再走一遍代码还原过程:

## Step 1

```
0042F9A9| push ebp                        |
0042F9AA| push acid burn.42FB67           |
0042F9AF| push dword ptr fs:[eax]         |
0042F9B2| mov dword ptr fs:[eax],esp      |
0042F9B5| mov dword ptr ds:[0x431750],0x29| 29:')'
0042F9BF| lea edx,dword ptr ss:[ebp-0x10] |
0042F9C2| mov eax,dword ptr ds:[ebx+0x1DC]|
0042F9C8| call acid burn.41AA58           |
0042F9CD| mov eax,dword ptr ss:[ebp-0x10] | 第一次获取name
```

当运行完 call acid burn.41AA58之后

![image-20221119170034224](D:\coding\blog\image-20221119170034224.png)

寄存器的EXA和栈[ebp-0x10]发生了变化,而[ebp-0x10]在call函数之前被存在了edx中

![image-20221119170223221](D:\coding\blog\image-20221119170223221.png)

![image-20221119170238874](D:\coding\blog\image-20221119170238874.png)

所以我们可以猜测函数burn.41AA58是获取name的输入值,把值放到edx的地址中,把输入的长度放在eax中.

在x64dbg中可以通过给地址设置标签增加汇编的可读性:
![image-20221119170641923](D:\coding\blog\image-20221119170641923.png)

------

## Step 2

把name当到[0x43176C]中

```
0042F9CD| mov eax,dword ptr ss:[ebp-0x10] | 第一次获取name
0042F9D0| call acid burn.403AB0           |
0042F9D5| mov dword ptr ds:[0x43176C],eax | 把eax（nameinput）放到0x0043176C中
```

------

## Step 3

获取name并取第一个字符进行运算,结果存在esi中

```
0042F9DA| lea edx,dword ptr ss:[ebp-0x10] |
0042F9DD| mov eax,dword ptr ds:[ebx+0x1DC]|
0042F9E3| call <acid burn.getNameInput>   |
0042F9E8| mov eax,dword ptr ss:[ebp-0x10] |
0042F9EB| movzx eax,byte ptr ds:[eax]     | 获取name[0]
0042F9EE| mov esi,eax                     | eax放入esi中开始运算
0042F9F0| shl esi,0x3                     |
0042F9F3| sub esi,eax                     | 运算完成esi = 0x2F4
```

再取多一次name,在用第二个字符进行运算,结果与上一次结果相加存到esi中,并保存到地址0x431754中:

```
0042F9F5| lea edx,dword ptr ss:[ebp-0x14] |
0042F9F8| mov eax,dword ptr ds:[ebx+0x1DC]|
0042F9FE| call <acid burn.getNameInput>   |
0042FA03| mov eax,dword ptr ss:[ebp-0x14] | 第二次获取name
0042FA06| movzx eax,byte ptr ds:[eax+0x1] | 获取name[1]
0042FA0A| shl eax,0x4                     |
0042FA0D| add esi,eax                     | 运算完成esi = 0xA04
0042FA0F| mov dword ptr ds:[0x431754],esi | esi复制到地址0x431754中
```

## Step 4

取name的第四个字符进行运算,斌存在esi中:

```
0042FA15| lea edx,dword ptr ss:[ebp-0x10] |
0042FA18| mov eax,dword ptr ds:[ebx+0x1DC]|
0042FA1E| call <acid burn.getNameInput>   |
0042FA23| mov eax,dword ptr ss:[ebp-0x10] | 第三次获取name
0042FA26| movzx eax,byte ptr ds:[eax+0x3] | 获取name[3]
0042FA2A| imul esi,eax,0xB                | 运算完成esi = 0x4C5
```

再次取name的第三个字符进行运算,与上一次运算结果再运算,结果存在0x431758中:

```
0042FA2D| lea edx,dword ptr ss:[ebp-0x14] |
0042FA30| mov eax,dword ptr ds:[ebx+0x1DC]|
0042FA36| call <acid burn.getNameInput>   |
0042FA3B| mov eax,dword ptr ss:[ebp-0x14] | 第四次获取name
0042FA3E| movzx eax,byte ptr ds:[eax+0x2] | 获取name[2]
0042FA42| imul eax,eax,0xE                |
0042FA45| add esi,eax                     | 运行结果esi=0xA2F
0042FA47| mov dword ptr ds:[0x431758],esi | esi复制到地址0x431758中
```

## Step 5

获取0x0043176C的值,在Step 2中我们知道这个值就是name,判断如果小于4就显示错误信息

```
0042FA4D| mov eax,dword ptr ds:[0x43176C] | 把0x0043176C中的值放到eax中
0042FA52| call <acid burn.406930>         | 获取eax长度
0042FA57| cmp eax,0x4                     | 判断第一个输入框的字符长度是否大于4
0042FA5A| jge acid burn.42FA79            |
0042FA5C| push 0x0                        |
0042FA5E| mov ecx,acid burn.42FB74        | 42FB74:"Try Again!"
0042FA63| mov edx,acid burn.42FB80        | 42FB80:"Sorry , The serial is incorect !"
0042FA68| mov eax,dword ptr ds:[0x430A48] |
0042FA6D| mov eax,dword ptr ds:[eax]      |
0042FA6F| call acid burn.42A170           |
0042FA74| jmp acid burn.42FB37            |
0042FA79| lea edx,dword ptr ss:[ebp-0x10] |
```

在运行完call <acid burn.406930>之后我们可以注意到eax变成5

![image-20221123010156460](D:\coding\blog\image-20221123010156460.png)

而后面的判断是用0x4与eax比较,所以猜测这个函数应该是类似获取字符串长度的函数.

## Step 6

```
0042FA79| lea edx,dword ptr ss:[ebp-0x10] |
0042FA7C| mov eax,dword ptr ds:[ebx+0x1DC]|
0042FA82| call <acid burn.getNameInput>   |
0042FA87| mov eax,dword ptr ss:[ebp-0x10] |
0042FA8A| movzx eax,byte ptr ds:[eax]     | 获取name[0]
0042FA8D| imul dword ptr ds:[0x431750]    | eax与0x431750的值相乘
0042FA93| mov dword ptr ds:[0x431750],eax |
0042FA98| mov eax,dword ptr ds:[0x431750] |
0042FA9D| add dword ptr ds:[0x431750],eax |
```

在Step1的时候我们知道[0x431750]是0x029  , eax存的是name[0].

上面经过一番计算之后最终[0x431750]的值变成0x02298了

## Step 7

```
0042FAA3| lea eax,dword ptr ss:[ebp-0x4]|
0042FAA6| mov edx,acid burn.42FBAC      | 42FBAC:"CW"
0042FAAB| call acid burn.403708         |
```

把字符串"CW"放入[ebp-0x4]中

## Step 8

```
0042FAB0| lea eax,dword ptr ss:[ebp-0x8]|
0042FAB3| mov edx,acid burn.42FBB8      | 42FBB8:"CRACKED"
0042FAB8| call acid burn.403708         |
```

把字符串"CRACKED"放入[ebp-0x8]中

## Step 9

这里就是关键步骤了

```

0042FABD | push dword ptr ss:[ebp-0x4]    |
0042FAC0 | push acid burn.42FBC8          |42FBC8:"-"
0042FAC5 | lea edx,dword ptr ss:[ebp-0x18]|
0042FAC8 | mov eax,dword ptr ds:[0x431750]|
0042FACD | call acid burn.406718          | fun1
0042FAD2 | push dword ptr ss:[ebp-0x18]   |
0042FAD5 | push acid burn.42FBC8          |42FBC8:"-"
0042FADA | push dword ptr ss:[ebp-0x8]    |
```

这里有5个push,分别是push了

[ebp-0x4]  = "CW"

burn.42FBC8 = "-"

[ebp-0x18] = ??

burn.42FBC8 = "-"

[ebp-0x8] = "CRACKED"

现在问题就是[ebp-0x18] 是什么,而它是函数call acid burn.406718的返回结果.

![image-20221124003144132](D:\coding\blog\image-20221124003144132.png)

它的值是"8856",究竟是怎么来的呢?

从上面的mov eax, dword ptr ds:[0x00431750]得知0x00431750的值0x2298作为参数传入函数中,而0x2298的十进制数值就是8856

到这里就破案了,我们只需要把[0x431750]的相关步骤记录下来,就可以生成验证码了

------

## 总结验证码生成过程:

value1 = 0x29;

value2 = name[0];

value3 = value1*value2;

code = value3  + value3 ;  



## 转换为注册机代码:

```javascript
console.log("start");
const part1 = "CW-";
let part2 = "";
const part3 = "-CRACKED";
const num1 = new Number(0x29);
const inputName = "lqcode";//name input
const num2 = new Number(inputName.charCodeAt(0));
const code = num1 * num2 * 2;
part2 = (code).toString(10);
const result = part1 + part2 + part3
console.log(result);

```

验证一下:

这次我输入的name是"woyoulaile",注册机的输出:

![image-20221127160609900](D:\coding\blog\crack-me-001-Acid-burn.assets\image-20221127160609900.png)

![image-20221127160639194](D:\coding\blog\crack-me-001-Acid-burn.assets\image-20221127160639194.png)

成功!!
