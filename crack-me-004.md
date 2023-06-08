# 初探

随程序 有个txt文件,里面提到是`Delphi`程序,而且特点是没有确定按钮,所以第一反应估计是监听了输入事件进行逻辑判断.

而由于我不会`Delphi`,所以搜索了一下逆向.发现可以用IDR - Delphi反编译.

![image-20230212174340121](D:\coding\blog\crack-me-004.assets\image-20230212174340121.png)

# 反编译

果然是用到`OnKeyUp`监听运行` chkcode`函数,然后还发现了有单击事件(OnClick)和双击事件(OnDbClick),找到对应的地址后面在OD中进行断点.

![image-20230212174905244](D:\coding\blog\crack-me-004.assets\image-20230212174905244.png)

在打开OD之前我们可以用IDR到处反编译的map,辅助debug

![image-20230212175230920](D:\coding\blog\crack-me-004.assets\image-20230212175230920.png)

# 先说结论

## 程序的整体过程

程序过程涉及三个事件

1. 通过监听OnKeyUp事件运行chkCode函数,这里是主要的注册码验证函数,如果验证通过就会设置一个内存地址的值为`0x3E`
2. 通过监听双击事件,判断特点内存地址的值是否等于`0x3E`,如果等于就设置内存地址的值为`0x85`
3. 通过监听单击事件,判断特点内存地址的值是否等于`0x85`,如果是的话就显示图片

## 注册码生成与检验

注册码的检验是通过字符串对比的方式.

样例输入:

![image-20230212233355321](D:\coding\blog\crack-me-004.assets\image-20230212233355321.png)

这个输入正确的注册码是`黑头Sun Bird11dseloffc-012-OKlqcode`

注册码由四部分组成:(结合汇编分析)

1. 黑头Sun Bird
2. 11
3. dseloffc-012-OK
4. lqcode

其中 1,3 部分是固定字符串

第二是用户名长度加五

第四部分是用户名



# 汇编分析

## OD导入MAP

首先使用插件导入Map文件

![image-20230212175356471](D:\coding\blog\crack-me-004.assets\image-20230212175356471.png)

在刚刚反编译的三个关键事件调用添加断点

![image-20230212175636796](D:\coding\blog\crack-me-004.assets\image-20230212175636796.png)

## 输入

<img src="D:\coding\blog\crack-me-004.assets\image-20230212175954889.png" alt="image-20230212175954889"  />

## 导入map的作用

注册码每输入一个数字断点都会停在`chkcode`函数的开始位置,后面的注释就是添加Map的显示

![image-20230212180147168](D:\coding\blog\crack-me-004.assets\image-20230212180147168.png)

## 注册码生成

进入`chkCode`函数时候`ecx`的值已经是这次OnKeyUp输入的字符

![image-20230212234354078](D:\coding\blog\crack-me-004.assets\image-20230212234354078.png)

这段看是对输入字符做处理,但是实际上是后面没有用到的

```
00457C44  |.  B9 05000000   mov ecx,0x5
00457C49  |>  6A 00         /push 0x0
00457C4B  |.  6A 00         |push 0x0
00457C4D  |.  49            |dec ecx
00457C4E  |.^ 75 F9         \jnz short CKme.00457C49
00457C50  |.  51            push ecx
00457C51  |.  874D FC       xchg dword ptr ss:[ebp-0x4],ecx          ;  获取注册码输入框输入的字符
00457C54  |.  53            push ebx
00457C55  |.  56            push esi
00457C56  |.  8BD8          mov ebx,eax
00457C58  |.  33C0          xor eax,eax
```

在`esi`寄存器中获取用户名的长度并加五

```
00457C66  |.  8BB3 F8020000 mov esi,dword ptr ds:[ebx+0x2F8]         ;  用户名输入框输入的长度
00457C6C  |.  83C6 05       add esi,0x5
```

把注册码第一段放入堆栈

```
00457C6F  |.  FFB3 10030000 push dword ptr ds:[ebx+0x310]            ;  验证码第一段： "黑头Sun Bird"
```

这里调用`IntToStr`函数,两个参数分别是`edx`和`eax`

`edx`存的是`[ebp-0x8]`的内存地址

`eax`存的是`esi`的值,就是之前计算的用户名长度加五

再把结果`[ebp-0x8]`的值放入堆栈

```
00457C75  |.  8D55 F8       lea edx,dword ptr ss:[ebp-0x8]
00457C78  |.  8BC6          mov eax,esi
00457C7A  |.  E8 85FEFAFF   call <CKme.sysutils.IntToStr>
00457C7F  |.  FF75 F8       push dword ptr ss:[ebp-0x8]              ;  验证码第二段： 用户名长度加5
```

把注册码第三段放入堆栈

```
00457C82  |.  FFB3 14030000 push dword ptr ds:[ebx+0x314]            ;  验证码第三段："dseloffc-012-OK"
```

获取输入的用户名作为第四段放入堆栈

```
00457C88  |.  8D55 F4       lea edx,dword ptr ss:[ebp-0xC]
00457C8B  |.  8B83 D4020000 mov eax,dword ptr ds:[ebx+0x2D4]
00457C91  |.  E8 B2B6FCFF   call <CKme.Controls.TControl.GetText>
00457C96  |.  FF75 F4       push dword ptr ss:[ebp-0xC]              ;  获取用户名输入
```

再把验证码组合成一个字符串与输入对比

![image-20230213000000595](D:\coding\blog\crack-me-004.assets\image-20230213000000595.png)

通过`jnz`指令 判断字符串对比结果,如果相等,就把`[ebx+0x30C]`的值设为0x3E

```
00457D3A  |. /75 0A         jnz short CKme.00457D46
00457D3C  |. |C783 0C030000>mov dword ptr ds:[ebx+0x30C],0x3E
00457D46  |> \8B83 0C030000 mov eax,dword ptr ds:[ebx+0x30C]
```

## 双击事件逻辑

![image-20230213000329032](D:\coding\blog\crack-me-004.assets\image-20230213000329032.png)

在双击事件逻辑中关键的判断语句:

```
00457EF5  |.  83BE 0C030000>cmp dword ptr ds:[esi+0x30C],0x3E
00457EFC  |.  75 0A         jnz short CKme.00457F08
00457EFE  |.  C786 0C030000>mov dword ptr ds:[esi+0x30C],0x85
```

这里的`[esi+0x30C]`就是上一步设置`0x3E`的地址,使用`cmp`判断相等的话就`[esi+0x30C]`改成`0x85`

这里双击事件就完成了,其他都是无用代码

## 单击事件逻辑

![image-20230213000727451](D:\coding\blog\crack-me-004.assets\image-20230213000727451.png)

同样地关键的逻辑代码很少:

```
cmp dword ptr ds:[esi+0x30C],0x85
```

判断`[esi+0x30C]`值为`0x85`的话就会运行注册成功的逻辑

```
00458096  |.  8B86 F0020000 mov eax,dword ptr ds:[esi+0x2F0]
0045809C  |.  E8 BFB1FCFF   call <CKme.Controls.TControl.SetVisible>
004580A1  |.  A1 20B84500   mov eax,dword ptr ds:[0x45B820]
004580A6  |.  83C0 70       add eax,0x70
004580A9  |.  BA 14814500   mov edx,CKme.00458114                              ;  恭喜恭喜！注册成功
004580AE  |.  E8 9DB8FAFF   call <CKme.System.@LStrAsg>
```

包括`<CKme.Controls.TControl.SetVisible>`显示图片

![image-20230213000933427](D:\coding\blog\crack-me-004.assets\image-20230213000933427.png)

# 验证

![image-20230213001328773](D:\coding\blog\crack-me-004.assets\image-20230213001328773.png)

朱茵真好看!!



# 总结

1. `Delphi`程序可以用IDR反编译查看代码 并且到处MAP文件辅助debug
2. 好像没有了..







