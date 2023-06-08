# 查壳脱壳

![image-20230219173025189](D:\coding\blog\crack-me-005.assets\image-20230219173025189.png)



![image-20230219173429026](D:\coding\blog\crack-me-005.assets\image-20230219173429026.png)

脱壳后的大小是448kb

![image-20230219173453475](D:\coding\blog\crack-me-005.assets\image-20230219173453475.png)

# 反编译

拉入`Darkde4`

![image-20230219173545051](D:\coding\blog\crack-me-005.assets\image-20230219173545051.png)

![image-20230219173555777](D:\coding\blog\crack-me-005.assets\image-20230219173555777.png)

能看到控件的地址和分别有什么事件

# 导出MAP文件

把脱壳后的文件用IDA打开

![image-20230219173730991](D:\coding\blog\crack-me-005.assets\image-20230219173730991.png)

添加所有`Delphi`的库 (Shift + F5):

![image-20230219173833107](D:\coding\blog\crack-me-005.assets\image-20230219173833107.png)

导出MAP文件:

![image-20230219173931841](D:\coding\blog\crack-me-005.assets\image-20230219173931841.png)

# 分析汇编

## 注册成功的条件

通过字符串搜索可以看到:

```
004473E4 | 53                       | push ebx                                       |
004473E5 | 8BD8                     | mov ebx,eax                                    |
004473E7 | 81BB 04030000 340C0000   | cmp dword ptr ds:[ebx+304],C34                 |
004473F1 | 0F84 88000000            | je ckme002-fix.44747F                          |
004473F7 | 81BB 08030000 0D230000   | cmp dword ptr ds:[ebx+308],230D                |
00447401 | 74 7C                    | je ckme002-fix.44747F                          |
00447403 | 81BB 10030000 940F0000   | cmp dword ptr ds:[ebx+310],F94                 |
0044740D | 75 70                    | jne ckme002-fix.44747F                         |
0044740F | 8B83 18030000            | mov eax,dword ptr ds:[ebx+318]                 |
00447415 | 3B83 14030000            | cmp eax,dword ptr ds:[ebx+314]                 |
0044741B | 75 62                    | jne ckme002-fix.44747F                         |
0044741D | 81BB 1C030000 E7030000   | cmp dword ptr ds:[ebx+31C],3E7                 |
00447427 | 74 56                    | je ckme002-fix.44747F                          |
00447429 | 33D2                     | xor edx,edx                                    |
...
00447465 | BA 8C744400              | mov edx,ckme002-fix.44748C                     | 44748C:"厉害厉害真厉害！佩服佩服真佩服！！"
0044746A | E8 EDC4FBFF              | call <ckme002-fix.System.@LStrAsg>             |
0044746F | BA B8744400              | mov edx,ckme002-fix.4474B8                     | 4474B8:"注册了"
00447474 | 8B83 EC020000            | mov eax,dword ptr ds:[ebx+2EC]                 |
0044747A | E8 3DCCFDFF              | call <ckme002-fix.Controls.TControl.SetText>   |
0044747F | 5B                       | pop ebx                                        |
```

可以看到注册成功的条件主要是

```
004473E7 | 81BB 04030000 340C0000   | cmp dword ptr ds:[ebx+304],C34                 |
004473F1 | 0F84 88000000            | je ckme002-fix.44747F                          |
004473F7 | 81BB 08030000 0D230000   | cmp dword ptr ds:[ebx+308],230D                |
00447401 | 74 7C                    | je ckme002-fix.44747F                          |
00447403 | 81BB 10030000 940F0000   | cmp dword ptr ds:[ebx+310],F94                 |
0044740D | 75 70                    | jne ckme002-fix.44747F                         |
0044740F | 8B83 18030000            | mov eax,dword ptr ds:[ebx+318]                 |
00447415 | 3B83 14030000            | cmp eax,dword ptr ds:[ebx+314]                 |
0044741B | 75 62                    | jne ckme002-fix.44747F                         |
0044741D | 81BB 1C030000 E7030000   | cmp dword ptr ds:[ebx+31C],3E7                 |
00447427 | 74 56                    | je ckme002-fix.44747F                          |
```

这五个判断分别是:

[ebx+304] != 0xC34

[ebx+308] != 230D

[ebx+310] == 0xF94

[ebx+318] == [ebx+314]

00961AE0 != 0x3E7

### 条件一 [ebx+304] != 0xC34

```
004473E7 | 81BB 04030000 340C0000   | cmp dword ptr ds:[ebx+304],C34                 |
004473F1 | 0F84 88000000            | je ckme002-fix.44747F                          |
```

想不让判断跳转就要让`[ebx+304] != C34 `,搜索引用`[ebx+304]`:

![image-20230219134308677](D:\coding\blog\crack-me-005.assets\image-20230219134308677.png)

分别是两个判断:

```
00446D8D | 74 0A                    | je <ckme002-fix.loc_446D99>                    |
00446D8F | C783 04030000 340C0000   | mov dword ptr ds:[ebx+304],C34                 |
00446D99 | 8D85 30FEFFFF            | lea eax,dword ptr ss:[ebp-1D0]                 |
```

```
00446DB1 | E8 EED1FDFF              | call <ckme002-fix.Controls.TControl.SetVisible |
00446DB6 | EB 0A                    | jmp <ckme002-fix.loc_446DC2>                   |
00446DB8 | C783 04030000 340C0000   | mov dword ptr ds:[ebx+304],C34                 |
00446DC2 | 33C0                     | xor eax,eax                                    | eax:&L"\\system32\\cmd.exe"
```

判断文件(C:\\ajj.126.c0m\\j\\o\\j\\o\\ok.txt)是否存在

```
00446D43 | 8983 18030000            | mov dword ptr ds:[ebx+318],eax                 | eax:&"<珺"
00446D49 | BA EC6D4400              | mov edx,<ckme002-fix.aCAjj_126_c0mJO>          | 446DEC:"C:\\ajj.126.c0m\\j\\o\\j\\o\\ok.txt"
00446D4E | 8D85 30FEFFFF            | lea eax,dword ptr ss:[ebp-1D0]                 |
00446D54 | E8 EDE7FBFF              | call <ckme002-fix.System.@Assign>              |
00446D59 | 8D85 30FEFFFF            | lea eax,dword ptr ss:[ebp-1D0]                 |
00446D5F | E8 07EAFBFF              | call <ckme002-fix.System.@ResetText>           |
00446D64 | E8 8BBAFBFF              | call <ckme002-fix.System.IOResult>             |
00446D69 | 85C0                     | test eax,eax                                   | eax:&"<珺"
00446D6B | 75 4B                    | jne <ckme002-fix.loc_446DB8>                   |
```

并读取里面的内容与内存中的值比较:

```
00446D6D | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]                   |
00446D70 | 8D85 30FEFFFF            | lea eax,dword ptr ss:[ebp-1D0]                 |
00446D76 | E8 5DD1FBFF              | call <ckme002-fix.System.@ReadLString>         |
00446D7B | E8 44BAFBFF              | call <ckme002-fix.System.@_IOTest>             |
00446D80 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]                   |
00446D83 | BA 146E4400              | mov edx,<ckme002-fix.dword_446E14>             |
00446D88 | E8 0BCFFBFF              | call <ckme002-fix.System.@LStrCmp>             |
00446D8D | 74 0A                    | je <ckme002-fix.loc_446D99>                    |
00446D8F | C783 04030000 340C0000   | mov dword ptr ds:[ebx+304],C34                 |
00446D99 | 8D85 30FEFFFF            | lea eax,dword ptr ss:[ebp-1D0]                 |
```

查看内存`ckme002-fix.dword_446E14`的值是:

```
20 61 6A 6A D0 B4 B5 C4 43 4B 6D 65 D5 E6 C0 C3
21 FF FF 00
```



![image-20230219135149192](D:\coding\blog\crack-me-005.assets\image-20230219135149192.png)

转换成字符串: ( ajj写的CKme真烂!) 注意前后是有空白字符的.

当判断通过之后

```
00446D88 | E8 0BCFFBFF              | call <ckme002-fix.System.@LStrCmp>             |
00446D8D | 74 0A                    | je <ckme002-fix.loc_446D99>                    |
00446D8F | C783 04030000 340C0000   | mov dword ptr ds:[ebx+304],C34                 |
00446D99 | 8D85 30FEFFFF            | lea eax,dword ptr ss:[ebp-1D0]                 |
```

就会跳过` mov dword ptr ds:[ebx+304],C34`赋值

这样第一个条件就满足了

### 条件二  [ebx+308] != 230D

```
004473F7 | 81BB 08030000 0D230000   | cmp dword ptr ds:[ebx+308],230D                |
00447401 | 74 7C                    | je ckme002-fix.44747F                          |
```

同样地搜索`0x308`引用:

![image-20230219135758833](D:\coding\blog\crack-me-005.assets\image-20230219135758833.png)

由于我们需要`[ebx+308]`不等于`230D` ,所以我们会看`mov`指令和`add`指令

这是FromCreate的初始化:

```
00446D23 | C783 08030000 8E020000   | mov dword ptr ds:[ebx+308],28E                 |
```

而其他三个赋值指令是在Button1的MouseDown事件下:

```
00446FA4 | 55                       | push ebp                                       |
00446FA5 | 8BEC                     | mov ebp,esp                                    |
00446FA7 | 8B90 08030000            | mov edx,dword ptr ds:[eax+308]                 |
00446FAD | 81FA 0D230000            | cmp edx,230D                                   |
00446FB3 | 74 20                    | je <ckme002-fix.loc_446FD5>                    |
00446FB5 | 80F9 01                  | cmp cl,1                                       |
00446FB8 | 75 09                    | jne <ckme002-fix.loc_446FC3>                   |
00446FBA | 8380 08030000 03         | add dword ptr ds:[eax+308],3                   |
00446FC1 | EB 12                    | jmp <ckme002-fix.loc_446FD5>                   |
00446FC3 | 81FA 94020000            | cmp edx,294                                    |
00446FC9 | 7D 0A                    | jge <ckme002-fix.loc_446FD5>                   |
00446FCB | C780 08030000 0D230000   | mov dword ptr ds:[eax+308],230D                |
00446FD5 | 5D                       | pop ebp                                        |
```

所以第二个条件是只要不点击Button1的MouseDown,就会暂时成立.

### 条件三  [ebx+310] == 0xF94

```
00447403 | 81BB 10030000 940F0000   | cmp dword ptr ds:[ebx+310],F94                 |
0044740D | 75 70                    | jne ckme002-fix.44747F                         |
```

我们要让[ebx+310] == F94, 搜索引用`0x310`:

![image-20230219141717650](D:\coding\blog\crack-me-005.assets\image-20230219141717650.png)

两个mov指令都是在函数FromMouseMove下:

![image-20230219142020338](D:\coding\blog\crack-me-005.assets\image-20230219142020338.png)

第一段核心代码:

```
004470F6 | 8B55 08                  | mov edx,dword ptr ss:[ebp+8]                    |
004470F9 | 8B45 0C                  | mov eax,dword ptr ss:[ebp+C]                    |
004470FC | 33C9                     | xor ecx,ecx                                     | ecx:&"藊B"
004470FE | 55                       | push ebp                                        |
004470FF | 68 17724400              | push <ckme002-fix.loc_447217>                   |
00447104 | 64:FF31                  | push dword ptr fs:[ecx]                         |
00447107 | 64:8921                  | mov dword ptr fs:[ecx],esp                      |
0044710A | 8B8B E0020000            | mov ecx,dword ptr ds:[ebx+2E0]                  | ecx:&"藊B"
00447110 | 8079 47 01               | cmp byte ptr ds:[ecx+47],1                      |
00447114 | 75 19                    | jne <ckme002-fix.loc_44712F>                    |
00447116 | 3D E2000000              | cmp eax,E2                                      |
0044711B | 7E 12                    | jle <ckme002-fix.loc_44712F>                    |
0044711D | 81FA 2C010000            | cmp edx,12C                                     |
00447123 | 7E 0A                    | jle <ckme002-fix.loc_44712F>                    |
00447125 | C783 10030000 10000000   | mov dword ptr ds:[ebx+310],10                   |
0044712F | 8B8B DC020000            | mov ecx,dword ptr ds:[ebx+2DC]                  | ecx:&"藊B"
```

首先 从`Darkde`可知Image3的地址是`2E0`,这样是判断中间的图片是否Image3在显示:

```
0044710A | 8B8B E0020000            | mov ecx,dword ptr ds:[ebx+2E0]                  | ecx:&"藊B"
00447110 | 8079 47 01               | cmp byte ptr ds:[ecx+47],1                      |
00447114 | 75 19                    | jne <ckme002-fix.loc_44712F>                    |
```

接下来是判断eax和edx的值范围, 由于这个是鼠标移动事件,所以这个应该是鼠标指针的坐标:

```
00447116 | 3D E2000000              | cmp eax,E2                                      |
0044711B | 7E 12                    | jle <ckme002-fix.loc_44712F>                    |
0044711D | 81FA 2C010000            | cmp edx,12C                                     |
00447123 | 7E 0A                    | jle <ckme002-fix.loc_44712F>                    |
00447125 | C783 10030000 10000000   | mov dword ptr ds:[ebx+310],10                   |
```

当全部条件满足了之后就会`[ebx+310]=10  ` 赋值,这个赋值在后面的判断中有用

第二层判断,同样地`2DC`是Image2的地址,所以首先图片需要是Image2:

```
0044712F | 8B8B DC020000            | mov ecx,dword ptr ds:[ebx+2DC]                  | ecx:"<珺"
00447135 | 8079 47 01               | cmp byte ptr ds:[ecx+47],1                      |
00447139 | 75 6C                    | jne <ckme002-fix.loc_4471A7>                    |
```

再判断指针位置:

```
0044713B | 83F8 17                  | cmp eax,17                                      | eax:&"<珺"
0044713E | 7D 67                    | jge <ckme002-fix.loc_4471A7>                    |
00447140 | 81FA 2C010000            | cmp edx,12C                                     |
00447146 | 7E 5F                    | jle <ckme002-fix.loc_4471A7>                    |
```

再判断[ebx+310]是否等于10,这里就是确保第一层判断通过  :

```
00447148 | 83BB 10030000 10         | cmp dword ptr ds:[ebx+310],10                   |
0044714F | 75 56                    | jne <ckme002-fix.loc_4471A7>                    |
00447151 | 83BB 0C030000 09         | cmp dword ptr ds:[ebx+30C],9                    | 9:'\t'
00447158 | 74 4D                    | je <ckme002-fix.loc_4471A7>                     |
0044715A | C783 10030000 940F0000   | mov dword ptr ds:[ebx+310],F94                  |
```

假如[ebx+30C] != 9  , 就会运行`mov dword ptr ds:[ebx+310],F94 ` , 条件三就会完成了.

但是后面还有两个逻辑:

1. 对ebx +314进行赋值:

   ```
   00447164 | 8B83 0C030000            | mov eax,dword ptr ds:[ebx+30C]                  | eax:&"<珺"
   0044716A | 83E8 01                  | sub eax,1                                       | eax:&"<珺"
   0044716D | 72 0A                    | jb <ckme002-fix.loc_447179>                     |
   0044716F | 74 14                    | je <ckme002-fix.loc_447185>                     |
   00447171 | 48                       | dec eax                                         | eax:&"<珺"
   00447172 | 74 1D                    | je <ckme002-fix.loc_447191>                     |
   00447174 | 48                       | dec eax                                         | eax:&"<珺"
   00447175 | 74 26                    | je <ckme002-fix.loc_44719D>                     |
   00447177 | EB 2E                    | jmp <ckme002-fix.loc_4471A7>                    |
   00447179 | C783 14030000 41000000   | mov dword ptr ds:[ebx+314],41                   | 41:'A'
   00447183 | EB 22                    | jmp <ckme002-fix.loc_4471A7>                    |
   00447185 | C783 14030000 3D000000   | mov dword ptr ds:[ebx+314],3D                   | 3D:'='
   0044718F | EB 16                    | jmp <ckme002-fix.loc_4471A7>                    |
   00447191 | C783 14030000 34000000   | mov dword ptr ds:[ebx+314],34                   | 34:'4'
   0044719B | EB 0A                    | jmp <ckme002-fix.loc_4471A7>                    |
   0044719D | C783 14030000 DF000000   | mov dword ptr ds:[ebx+314],DF                   |
   ```

2. 判断Edit1输入是否等于`ajj`,是的话会显示数字在UI上:

   ```
   004471A7 | 81BB 10030000 940F0000   | cmp dword ptr ds:[ebx+310],F94                  |
   004471B1 | 75 46                    | jne <ckme002-fix.loc_4471F9>                    |
   004471B3 | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]                    | [ebp-4]:"l怌"
   004471B6 | 8B83 E8020000            | mov eax,dword ptr ds:[ebx+2E8]                  | eax:&"<珺"
   004471BC | E8 CBCEFDFF              | call <ckme002-fix.Controls.TControl.GetText>    |
   004471C1 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]                    | [ebp-4]:"l怌"
   004471C4 | BA 30724400              | mov edx,<ckme002-fix.dword_447230>              | 447230:"ajj"
   004471C9 | E8 CACAFBFF              | call <ckme002-fix.System.@LStrCmp>              |
   004471CE | 75 29                    | jne <ckme002-fix.loc_4471F9>                    |
   004471D0 | B2 01                    | mov dl,1                                        |
   004471D2 | 8B83 FC020000            | mov eax,dword ptr ds:[ebx+2FC]                  | eax:&"<珺"
   004471D8 | E8 C7CDFDFF              | call <ckme002-fix.Controls.TControl.SetVisible> |
   004471DD | 8D55 F8                  | lea edx,dword ptr ss:[ebp-8]                    |
   004471E0 | 8B83 0C030000            | mov eax,dword ptr ds:[ebx+30C]                  | eax:&"<珺"
   004471E6 | E8 AD0AFCFF              | call <ckme002-fix.sysutils.IntToStr>            |
   004471EB | 8B55 F8                  | mov edx,dword ptr ss:[ebp-8]                    |
   004471EE | 8B83 FC020000            | mov eax,dword ptr ds:[ebx+2FC]                  | eax:&"<珺"
   004471F4 | E8 C3CEFDFF              | call <ckme002-fix.Controls.TControl.SetText>    |
   ```

![image-20230219165450845](D:\coding\blog\crack-me-005.assets\image-20230219165450845.png)

这个数字决定了条件四的操作



### 条件2.1 :[ebx+30C] != 9

接下来是需要让`[ebx+30C]'不等于9,这样就要再搜索`30C`引用:

![image-20230219143342252](D:\coding\blog\crack-me-005.assets\image-20230219143342252.png)

同样地第一个`mov`是在初始化,所以我们需要让第二个`mov`运行,而第二个`mov`是在`Edit2DbClick`事件上完成的:

```
00446FF8 | 55                       | push ebp                                        | CKme.TForm1.Edit2DblClick
00446FF9 | 8BEC                     | mov ebp,esp                                     |
00446FFB | 33C9                     | xor ecx,ecx                                     |
... ...
004470B9 | E8 47ECFBFF              | call <ckme002-fix.System.@_llmod>               |
004470BE | 8983 0C030000            | mov dword ptr ds:[ebx+30C],eax                  | eax:"upported"
004470C4 | 33C0                     | xor eax,eax                                     | eax:"upported"
```

而目目前这个函数是无法断点的,因为现在Edit的状态是不可用,我们需要先让Edit2变成可用状态

从Darkde上知道Edit2的地址是`0x2F0`,所以常量:

![image-20230219144736350](D:\coding\blog\crack-me-005.assets\image-20230219144736350.png)

当中前面三个`mov`指令都是在初始化运行,而第四个mov指令是在`Panel1DblClick`中,非常可疑:

![image-20230219144844664](D:\coding\blog\crack-me-005.assets\image-20230219144844664.png)

点击事件中需要让[eax+308] == 29D , 才能解锁Edit2的点击和输入事件:

```
00446FDC | 81B8 08030000 9D020000   | cmp dword ptr ds:[eax+308],29D                  | CKme.TForm1.Panel1DblClick
```

还记得[eax+308]的值在条件二分析过,初始化时`28E`,通过Button1的MouseDown事件去修改:

```
00446FA4 | 55                       | push ebp                                       |
00446FA5 | 8BEC                     | mov ebp,esp                                    |
00446FA7 | 8B90 08030000            | mov edx,dword ptr ds:[eax+308]                 |
00446FAD | 81FA 0D230000            | cmp edx,230D                                   |
00446FB3 | 74 20                    | je <ckme002-fix.loc_446FD5>                    |
00446FB5 | 80F9 01                  | cmp cl,1                                       |
00446FB8 | 75 09                    | jne <ckme002-fix.loc_446FC3>                   |
00446FBA | 8380 08030000 03         | add dword ptr ds:[eax+308],3                   |
00446FC1 | EB 12                    | jmp <ckme002-fix.loc_446FD5>                   |
00446FC3 | 81FA 94020000            | cmp edx,294                                    |
00446FC9 | 7D 0A                    | jge <ckme002-fix.loc_446FD5>                   |
00446FCB | C780 08030000 0D230000   | mov dword ptr ds:[eax+308],230D                |
00446FD5 | 5D                       | pop ebp                                        |
```

分析可知道我们需要这个逻辑通过:

```
00446FB5 | 80F9 01                  | cmp cl,1                                       |
00446FB8 | 75 09                    | jne <ckme002-fix.loc_446FC3>                   |
00446FBA | 8380 08030000 03         | add dword ptr ds:[eax+308],3                   |
```

让`[eax+308]`不断加3从`28E`变成`29D `

而经过测试`cmp cl,1   `是当点击鼠标左键cl就会是0,点击右键就会是1,而从28E变成29D相差15

所以Edit2的解锁操作是:

1. 右键点击`注册`按钮五次 ----> 让`[eax+308]`不断加3从`28E`变成`29
2. 双击![image-20230219145023137](D:\coding\blog\crack-me-005.assets\image-20230219145023137.png)

### Edit2 双击判断逻辑

现在就可以输入了:

![image-20230219151525871](D:\coding\blog\crack-me-005.assets\image-20230219151525871.png)

由于Edit1 控件的地址是`0x2E8`

Edit2控件的地址是`0x2F0`

![image-20230219114317745](D:\coding\blog\crack-me-005.assets\image-20230219114317745.png)

获取Edit2输入长度判断是否等于8

```
00447013 | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]                   | [ebp-4]:"lqcode"
00447016 | 8B83 F0020000            | mov eax,dword ptr ds:[ebx+2F0]                 | [ebx+2F0]:&"<珺"
0044701C | E8 6BD0FDFF              | call <ckme002-fix.Controls.TControl.GetText>   |
00447021 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]                   | [ebp-4]:"lqcode"
00447024 | E8 5FCBFBFF              | call <ckme002-fix.System.@LStrLen>             |
00447029 | 83F8 08                  | cmp eax,8                                      |
```

`cmp byte ptr ds:[eax+1],5F `,由于是`byte` 所以是获取输入的第二个字符是否0x5F (_) ,

```
00447032 | 8D55 F8                  | lea edx,dword ptr ss:[ebp-8]                   | [ebp-8]:"lqcode"
00447035 | 8B83 F0020000            | mov eax,dword ptr ds:[ebx+2F0]                 | eax:"lqcode", [ebx+2F0]:&"<珺"
0044703B | E8 4CD0FDFF              | call <ckme002-fix.Controls.TControl.GetText>   |
00447040 | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]                   | [ebp-8]:"lqcode"
00447043 | 8078 01 5F               | cmp byte ptr ds:[eax+1],5F                     | eax+1:"qcode", 5F:'_'
```

继续 判断第6个字符是否0x2C (,)

```
00447049 | 8D55 F4                  | lea edx,dword ptr ss:[ebp-C]                   | [ebp-C]:"lqcode"
0044704C | 8B83 F0020000            | mov eax,dword ptr ds:[ebx+2F0]                 | eax:"lqcode", [ebx+2F0]:&"<珺"
00447052 | E8 35D0FDFF              | call <ckme002-fix.Controls.TControl.GetText>   |
00447057 | 8B45 F4                  | mov eax,dword ptr ss:[ebp-C]                   | [ebp-C]:"lqcode"
0044705A | 8078 05 2C               | cmp byte ptr ds:[eax+5],2C                     | 2C:','
```

由于`mov eax,dword ptr ds:[ebx+2E8]  ` 是`[ebx+2E8]`, 所以这里是获取Edit1的输入字符串长度

```
00447060 | 8D55 F0                  | lea edx,dword ptr ss:[ebp-10]                  |
00447063 | 8B83 E8020000            | mov eax,dword ptr ds:[ebx+2E8]                 | [ebx+2E8]:&"<珺"
00447069 | E8 1ED0FDFF              | call <ckme002-fix.Controls.TControl.GetText>   |
0044706E | 8B45 F0                  | mov eax,dword ptr ss:[ebp-10]                  |
00447071 | E8 12CBFBFF              | call <ckme002-fix.System.@LStrLen>             |
00447076 | 83C0 03                  | add eax,3                                      |
00447079 | B9 03000000              | mov ecx,3                                      |
0044707E | 99                       | cdq                                            |
0044707F | F7F9                     | idiv ecx                                       |
00447081 | 85D2                     | test edx,edx                                   |
00447083 | 75 3F                    | jne <ckme002-fix.loc_4470C4>                   |
```

`cdq`指令把 EAX 的第 31 bit 复制到 EDX 的每一个 bit 上。 它大多出现在除法运算之前。它实际的作用只是把EDX的所有位都设成EAX最高位的值。也就是说，当EAX <80000000, EDX 为00000000；当EAX >= 80000000， EDX 则为FFFFFFFF。

`idiv ecx` 将有符号 DX:AX（其中 DX 必须包含 AX  的符号扩展）除以有符号 ecx字。（结果：AX = 商，DX =  余数）

`test edx,edx  ` 就是判断上面的除法运算是否有余数

所以Edit1输入的字符串长度一定要是3的倍数

这样就能运行:

```
004470BE | 8983 0C030000            | mov dword ptr ds:[ebx+30C],eax                  |
```

让**条件2.1 :[ebx+30C] != 9**成立



### 条件四  [ebx+318] == [ebx+314]

在条件三的判断中我们曾经对`[ebx+314]`有过赋值:

```
00447164 | 8B83 0C030000            | mov eax,dword ptr ds:[ebx+30C]                  | eax:&"<珺"
0044716A | 83E8 01                  | sub eax,1                                       | eax:&"<珺"
0044716D | 72 0A                    | jb <ckme002-fix.loc_447179>                     |
0044716F | 74 14                    | je <ckme002-fix.loc_447185>                     |
00447171 | 48                       | dec eax                                         | eax:&"<珺"
00447172 | 74 1D                    | je <ckme002-fix.loc_447191>                     |
00447174 | 48                       | dec eax                                         | eax:&"<珺"
00447175 | 74 26                    | je <ckme002-fix.loc_44719D>                     |
00447177 | EB 2E                    | jmp <ckme002-fix.loc_4471A7>                    |
00447179 | C783 14030000 41000000   | mov dword ptr ds:[ebx+314],41                   | 41:'A'
00447183 | EB 22                    | jmp <ckme002-fix.loc_4471A7>                    |
00447185 | C783 14030000 3D000000   | mov dword ptr ds:[ebx+314],3D                   | 3D:'='
0044718F | EB 16                    | jmp <ckme002-fix.loc_4471A7>                    |
00447191 | C783 14030000 34000000   | mov dword ptr ds:[ebx+314],34                   | 34:'4'
0044719B | EB 0A                    | jmp <ckme002-fix.loc_4471A7>                    |
0044719D | C783 14030000 DF000000   | mov dword ptr ds:[ebx+314],DF                   |
```

现在暂时还不知道赋值的逻辑是怎样,但是断点可以查看到这次`[ebx+314]`的值是`3D`  (十进制61)

搜索[ebx+318]  引用:

![image-20230219161742747](D:\coding\blog\crack-me-005.assets\image-20230219161742747.png)

猜测应该是一个累加的逻辑,所有的`add`指令都在Image1-4的点击事件中:

![image-20230219161911068](D:\coding\blog\crack-me-005.assets\image-20230219161911068.png)

而点击Image事件分别会让[ebx+318] 变化:

Image1: 左键:0x2 (十进制2) ,右键0x11 (十进制17)

Image2: 左键:0x3 (十进制3) ,右键0x13 (十进制19)

Image3: 左键:0x5 (十进制5) ,右键0x17(十进制23) 

Image4: 左键:0x7 (十进制7) ,右键0x1B(十进制27) 

### 条件五 [ebx+31C] != 3E7 

这个条件就是最容易了,搜索引用:

![image-20230219170004780](D:\coding\blog\crack-me-005.assets\image-20230219170004780.png)

只有在button1Click事件上赋值

![image-20230219170026024](D:\coding\blog\crack-me-005.assets\image-20230219170026024.png)

所以这个条件成立就是不要同左键点击按钮就可以

### 注册成功

按钮文字会变成 **注册了**

![image-20230219171456189](D:\coding\blog\crack-me-005.assets\image-20230219171456189.png)



## 注册步骤

1. 在**C:\\ajj.126.c0m\\j\\o\\j\\o\\ok.txt** 下新建文件,并写入数据(要16进制转换成字符串) , 验证通过的话会出现第二个输入框,但是不可输入状态

   ```
   20 61 6A 6A D0 B4 B5 C4 43 4B 6D 65 D5 E6 C0 C3 21 FF FF 00
   ```

2. 右键点击注册按钮五次
3. 双击中间Panel空白区域(没图片的地方), 这样第二个输入框就能输入了
4. 在第一个输入框输入`ajj`
5. 在第二个输入框输入符合条件的字符串
   1. 长度是8
   2. 第二个字符是`_`
   3. 第六个字符是`,`

 	5. 双击第二个输入框
 	6. 当中间图片显示第三个图片(性相近)时,鼠标在右下角移动
 	7. 当中间图片显示第二个图片(性本善)的是时候,鼠标在左下角移动, 这时候按钮左边就会出现数字(0~5)
 	8. 根据数字对图片进行不同的点击操作:
     - 数字为0——在“习相远”图片时左键点击图片2次，在“人之初”图片时右键点击图片3次
     - 数字为1——在“习相远”图片时左键点击图片1次，在“习相远”图片时右键点击图片2次
     - 数字为2——在“性本善”图片时左键点击图片2次，在“性相近”图片时右键点击图片2次
     - 数字为3——在“习相远”图片时左键点击图片1次，在“习相远”图片时右键点击图片8次
     - 数字为4——在“习相远”图片时左键点击图片2次，在“人之初”图片时右键点击图片3次

注册完成 --- 按钮文字变成"注册了"



# 总结

1. `Delphi` 脱壳有可能会导致汇编出错
2. 可以用`DarkDe`对Delphi程序进行反汇编,查看控件和事件的地址
3. 在`IDA`中添加`Delphi`的函数库后导出的MAP文件会比原来多注释,对理解汇编逻辑有帮助
4. 事件回调函数的变量可以根据事件类型进行猜测,例如鼠标点击事件的变量有可能是点击的按键,或者指针的坐标
5. `cdq`指令把 EAX 的第 31 bit 复制到 EDX 的每一个 bit 上。 它大多出现在除法运算之前。它实际的作用只是把EDX的所有位都设成EAX最高位的值。也就是说，当EAX <80000000, EDX 为00000000；当EAX >= 80000000， EDX 则为FFFFFFFF。
6. `idiv ecx` 将有符号 DX:AX（其中 DX 必须包含 AX  的符号扩展）除以有符号 ecx字。（结果：AX = 商，DX =  余数）