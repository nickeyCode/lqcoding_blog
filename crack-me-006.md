# 初探

界面

![image-20230221064457000](D:\coding\blog\crack-me-006.assets\image-20230221064457000.png)

About - Help里面有说到crack成功的条件:

![image-20230221064538591](D:\coding\blog\crack-me-006.assets\image-20230221064538591.png)

1. 让**OK**按钮可点击并且消失
2. 让**Cancella**按钮消失看见Ringrer0 logo

# 查壳

![image-20230221064835098](D:\coding\blog\crack-me-006.assets\image-20230221064835098.png)

看起来是没有壳

在DarkDe上看下事件和控件

![image-20230221065547082](D:\coding\blog\crack-me-006.assets\image-20230221065547082.png)

![image-20230221065557192](D:\coding\blog\crack-me-006.assets\image-20230221065557192.png)

主要分为点击事件和监听事件。

在IDA中到处MAP文件。开始分析

# 分析汇编

输入

![image-20230221070820982](D:\coding\blog\crack-me-006.assets\image-20230221070820982.png)

由于Ok button不可点击，所以决定先看看Cancella按钮的逻辑





1. 判断长度是否小于5
2. 获取name[4] -----0x64
3. name[5]/7的余数  ----- 2
4. 余数加2 ------ 4
5. call 0x00442A20 ----- esi 18
6. esi * name[0]  ----- A20
7. 循环
8. 对比结果是否7A69



OK btn logic





## name输入变化逻辑

从DarkDe可知道Name输入变化callback的地址是`00442C78`

![image-20230223074852124](D:\coding\blog\crack-me-006.assets\image-20230223074852124.png)

核心逻辑:

1. 判读Cancella是否显示 ,因为`2D0`是Cancella控件的地址

   ```
   00442E1C | 8B83 D0020000            | mov eax,dword ptr ds:[ebx+2D0]          | eax:&"詻B"
   00442E22 | 8078 47 00               | cmp byte ptr ds:[eax+47],0              |
   00442E26 | 75 0F                    | jne <along3x.1.loc_442E37>              |
   ```

 2. 获取Codice输入框的输入,因为`2E0`是Codice控件的地址

    ```
    00442E37 | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]            |
    00442E3A | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          | eax:&"詻B", [ebx+2E0]:&"詻B"
    00442E40 | E8 7B04FEFF              | call <along3x.1.TControl::GetText>      |
    00442E45 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
    00442E48 | 50                       | push eax                                | eax:&"詻B"
    ```

3. 获取name输入框的输入,因为`2DC`是name控件的地址

   ```
   00442E49 | 8D55 F8                  | lea edx,dword ptr ss:[ebp-8]            |
   00442E4C | 8B83 DC020000            | mov eax,dword ptr ds:[ebx+2DC]          | eax:&"詻B", [ebx+2DC]:&"詻B"
   00442E52 | E8 6904FEFF              | call <along3x.1.TControl::GetText>      |
   00442E57 | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            |
   ```

4. 运行函数`sub_442A3C`

   ```
   00442E5A | 5A                       | pop edx                                 |
   00442E5B | E8 DCFBFFFF              | call <along3x.1.sub_442A3C>             |
   00442E60 | 84C0                     | test al,al                              |
   00442E62 | 74 0F                    | je <along3x.1.loc_442E73>               |
   00442E64 | B2 01                    | mov dl,1                                |
   00442E66 | 8B83 CC020000            | mov eax,dword ptr ds:[ebx+2CC]          | eax:&"詻B", [ebx+2CC]:&"詻B"
   00442E6C | 8B08                     | mov ecx,dword ptr ds:[eax]              | [eax]:"詻B"
   00442E6E | FF51 60                  | call dword ptr ds:[ecx+60]              |
   00442E71 | EB 0D                    | jmp <along3x.1.loc_442E80>              |
   00442E73 | 33D2                     | xor edx,edx                             |
   00442E75 | 8B83 CC020000            | mov eax,dword ptr ds:[ebx+2CC]          | eax:&"詻B", [ebx+2CC]:&"詻B"
   00442E7B | 8B08                     | mov ecx,dword ptr ds:[eax]              | [eax]:"詻B"
   00442E7D | FF51 60                  | call dword ptr ds:[ecx+60]              |
   ```

   判断函数运行结果`al`是否等于0

   `````
   00442E60 | 84C0                     | test al,al                              |
   00442E62 | 74 0F                    | je <along3x.1.loc_442E73>               |
   `````

   如果符合就更新Ok控件 , 因为`2CC`是Ok控件的地址

   ```
   00442E64 | B2 01                    | mov dl,1                                |
   00442E66 | 8B83 CC020000            | mov eax,dword ptr ds:[ebx+2CC]          | eax:&"詻B", [ebx+2CC]:&"詻B"
   00442E6C | 8B08                     | mov ecx,dword ptr ds:[eax]              | [eax]:"詻B"
   00442E6E | FF51 60                  | call dword ptr ds:[ecx+60]
   ```



### 函数`sub_442A3C` 逻辑

这个函数主要功能是判断两个输入是否通过验证,Name输入可以试任意字符,Codice只能是数字

从call函数前的堆栈操作可以知道`[ebp-4]`是Name的输入值,`[ebp-8]`是Codice的输入值

```
00442A44 | 8955 F8                  | mov dword ptr ss:[ebp-8],edx            |
00442A47 | 8945 FC                  | mov dword ptr ss:[ebp-4],eax            |
00442A4A | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
00442A4D | E8 9611FCFF              | call <along3x.1.__linkproc__ LStrAddRef |
00442A52 | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            |
00442A55 | E8 8E11FCFF              | call <along3x.1.__linkproc__ LStrAddRef |
00442A5A | 33C0                     | xor eax,eax                             |
```

首先判断Name长度是否大于5

```
00442A62 | 64:FF30                  | push dword ptr fs:[eax]                 |
00442A65 | 64:8920                  | mov dword ptr fs:[eax],esp              |
00442A68 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
00442A6B | E8 C40FFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442A70 | 83F8 05                  | cmp eax,5                               |
00442A73 | 7E 53                    | jle <along3x.1.loc_442AC8>              |
```

如果大于五就进入循环:

```
00442A75 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
00442A78 | E8 B70FFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442A7D | 8BD8                     | mov ebx,eax                             | ebx:&"詻B"
00442A7F | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
00442A82 | E8 AD0FFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442A87 | 8BD0                     | mov edx,eax                             |
00442A89 | 4A                       | dec edx                                 |
00442A8A | 85D2                     | test edx,edx                            |
00442A8C | 7E 20                    | jle <along3x.1.loc_442AAE>              |
00442A8E | B8 01000000              | mov eax,1                               |
00442A93 | 8B4D FC                  | mov ecx,dword ptr ss:[ebp-4]            |
00442A96 | 0FB64C01 FF              | movzx ecx,byte ptr ds:[ecx+eax-1]       |
00442A9B | 8B75 FC                  | mov esi,dword ptr ss:[ebp-4]            |
00442A9E | 0FB63406                 | movzx esi,byte ptr ds:[esi+eax]         |
00442AA2 | 0FAFCE                   | imul ecx,esi                            |
00442AA5 | 0FAFC8                   | imul ecx,eax                            |
00442AA8 | 03D9                     | add ebx,ecx                             | ebx:&"詻B"
00442AAA | 40                       | inc eax                                 |
00442AAB | 4A                       | dec edx                                 |
00442AAC | 75 E5                    | jne <along3x.1.loc_442A93>              |
```

获取Name长度存在ebx

```
00442A75 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
00442A78 | E8 B70FFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442A7D | 8BD8                     | mov ebx,eax                             | ebx:&"詻B"
```

再把Name长度存在edx,判断是否大于0,并且作为for循环的退出条件

```
00442A7F | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            |
00442A82 | E8 AD0FFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442A87 | 8BD0                     | mov edx,eax                             |
00442A89 | 4A                       | dec edx                                 |
00442A8A | 85D2                     | test edx,edx                            |
00442A8C | 7E 20                    | jle <along3x.1.loc_442AAE>              |
```

`eax` 赋值 1 , 等于for循环的i= 1

```
00442A8E | B8 01000000              | mov eax,1                               |
```

获取字符串

```
00442A93 | 8B4D FC                  | mov ecx,dword ptr ss:[ebp-4]            |
00442A96 | 0FB64C01 FF              | movzx ecx,byte ptr ds:[ecx+eax-1]       |
00442A9B | 8B75 FC                  | mov esi,dword ptr ss:[ebp-4]            |
00442A9E | 0FB63406                 | movzx esi,byte ptr ds:[esi+eax]         |
```

ecx : name[eax -1]

esi: name[eax]

```
00442AA2 | 0FAFCE                   | imul ecx,esi                            |
00442AA5 | 0FAFC8                   | imul ecx,eax                            |
00442AA8 | 03D9                     | add ebx,ecx                             | ebx:&"詻B"
00442AAA | 40                       | inc eax                                 |
00442AAB | 4A                       | dec edx                                 |
00442AAC | 75 E5                    | jne <along3x.1.loc_442A93>              |
```

`imul`是乘法指令

所以for循环的逻辑是:

ebx += name[eax -1] * name[eax] * eax

`inc`是累加指令 , eax会累加1

`dec`是累减指令, edx每次减1,直到为0

如果用变成语言写这段逻辑应该是:

```
let result2 = nname.length;
  for (let i = 1; i < nname.length; i++) {
    console.log("i:" + i);
    console.log("result2:" + result2);
    let a = new Number(nname.charCodeAt(i));
    let b = new Number(nname.charCodeAt(i - 1));
    result2 += a * b * i;
  }
```

------

获取的Codice值,转换成Int型

```
00442AAE | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            |
00442AB1 | E8 BA4BFCFF              | call <along3x.1.@StrToInt>              |
```

与For循环的结果相减,判断是否等于`0x29A`

```
00442AB6 | 2BD8                     | sub ebx,eax                             | ebx:&"詻B"
00442AB8 | 81FB 9A020000            | cmp ebx,29A                             | ebx:&"詻B"
00442ABE | 75 04                    | jne <along3x.1.loc_442AC4>              |
00442AC0 | B3 01                    | mov bl,1                                |
00442AC2 | EB 06                    | jmp <along3x.1.loc_442ACA>              |
```

到这里整个逻辑就完整了

1. For循环操作Name的输入得出结果result1
2. 获取Codice的输入为result2
3. result1 - result2必须等于`0x29A`

### 验证

当输入nama 是 `lqcode` , For循环的计算结果是:`162451`,只有当Codice是`161785`时才会让Ok控件可点击

![image-20230223082248337](D:\coding\blog\crack-me-006.assets\image-20230223082248337.png)

## Codice的输入变化逻辑

过程基本与Name的逻辑相同,调用的函数都是`sub_442A3C`

```
00442D0B | E8 2CFDFFFF              | call <along3x.1.sub_442A3C>             |
```

## Ok控件点击事件

```
00442D6F | 68 ED2D4400              | push <along3x.1.loc_442DED>             |
00442D74 | 64:FF30                  | push dword ptr fs:[eax]                 |
00442D77 | 64:8920                  | mov dword ptr fs:[eax],esp              |
00442D7A | 8B83 D0020000            | mov eax,dword ptr ds:[ebx+2D0]          | [ebx+2D0]:&"詻B"
00442D80 | 8078 47 01               | cmp byte ptr ds:[eax+47],1              |
00442D84 | 75 12                    | jne <along3x.1.loc_442D98>              |
00442D86 | BA 002E4400              | mov edx,<along3x.1.dword_442E00>        |
00442D8B | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          | [ebx+2E0]:&"詻B"
00442D91 | E8 5A05FEFF              | call <along3x.1.TControl::SetText>      |
00442D96 | EB 3F                    | jmp <along3x.1.loc_442DD7>              |
```

判断Cancella (地址为`2D0`)控件是否显示:

```
00442D7A | 8B83 D0020000            | mov eax,dword ptr ds:[ebx+2D0]          | [ebx+2D0]:&"詻B"
00442D80 | 8078 47 01               | cmp byte ptr ds:[eax+47],1              |
```

如果显示就重置Codice (地址`2E0`)输入值然后跳转结束地址(`loc_442DD7`):

```
00442D8B | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          | [ebx+2E0]:&"詻B"
00442D91 | E8 5A05FEFF              | call <along3x.1.TControl::SetText>      |
```

所以我们点击Ok前要先把Cancella 控件隐藏,才能确保Ok的逻辑完整运行.

## Cancella控件点击事件

获取Codice(`[ebx+2E0]`)的输入和name(`[ebx+2DC]`)的输入,并运行函数`sub_442AF4`

```
00442EBE | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]            | [ebp-4]:&"斢@"
00442EC1 | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          |
00442EC7 | E8 F403FEFF              | call <along3x.1.TControl::GetText>      |
00442ECC | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:&"斢@"
00442ECF | E8 9C47FCFF              | call <along3x.1.@StrToInt>              |
00442ED4 | 50                       | push eax                                |
00442ED5 | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]            | [ebp-4]:&"斢@"
00442ED8 | 8B83 DC020000            | mov eax,dword ptr ds:[ebx+2DC]          |
00442EDE | E8 DD03FEFF              | call <along3x.1.TControl::GetText>      |
00442EE3 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:&"斢@"
00442EE6 | 5A                       | pop edx                                 |
00442EE7 | E8 08FCFFFF              | call <along3x.1.sub_442AF4>             |
```

根据结果更新Cancella(`[ebx+2D0] `)按键是否显示

```
00442EEC | 84C0                     | test al,al                              |
00442EEE | 74 1C                    | je <along3x.1.loc_442F0C>               |
00442EF0 | 33D2                     | xor edx,edx                             |
00442EF2 | 8B83 D0020000            | mov eax,dword ptr ds:[ebx+2D0]          |
00442EF8 | E8 B302FEFF              | call <along3x.1.TControl::SetVisible>   | 更新Cancella按键是否显示
00442EFD | B2 01                    | mov dl,1                                |
00442EFF | 8B83 CC020000            | mov eax,dword ptr ds:[ebx+2CC]          |
00442F05 | 8B08                     | mov ecx,dword ptr ds:[eax]              |
00442F07 | FF51 60                  | call dword ptr ds:[ecx+60]              |
00442F0A | EB 10                    | jmp <along3x.1.loc_442F1C>              |
00442F0C | BA 482F4400              | mov edx,<along3x.1.dword_442F48>        |
00442F11 | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          |
00442F17 | E8 D403FEFF              | call <along3x.1.TControl::SetText>      |
```

### sub_442AF4函数逻辑

同样地首先判断Name的输入长度是否大于5

```
00442AFF | 8945 FC                  | mov dword ptr ss:[ebp-4],eax            | [ebp-4]:"lqcode"
00442B02 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442B05 | E8 DE10FCFF              | call <along3x.1.__linkproc__ LStrAddRef |
00442B0A | 33C0                     | xor eax,eax                             |
00442B0C | 55                       | push ebp                                |
00442B0D | 68 902B4400              | push <along3x.1.loc_442B90>             |
00442B12 | 64:FF30                  | push dword ptr fs:[eax]                 |
00442B15 | 64:8920                  | mov dword ptr fs:[eax],esp              |
00442B18 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442B1B | E8 140FFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442B20 | 83F8 05                  | cmp eax,5                               |
00442B23 | 7E 53                    | jle <along3x.1.loc_442B78>              |
```

获取Name[5]

```
00442B25 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442B28 | 0FB640 04                | movzx eax,byte ptr ds:[eax+4]           |
```

eax = Name[5] % 7 + 2

```
00442B2C | B9 07000000              | mov ecx,7                               |
00442B31 | 33D2                     | xor edx,edx                             |
00442B33 | F7F1                     | div ecx                                 |
00442B35 | 8BC2                     | mov eax,edx                             |
00442B37 | 83C0 02                  | add eax,2                               |
00442B3A | E8 E1FEFFFF              | call <along3x.1.sub_442A20>             |
```

`div ecx` 指令是用eax除以ecx,得到的余数存在edx,商存在eax

所以当输入`lqcode`时候,到运行函数`sub_442A20`时`eax`的值应该是 (d)    ->(0x64 % 7) + 2 = 4

#### sub_442A20函数

```
00442A20 | 53                       | push ebx                                | ebx:&"詻B"
00442A21 | 8BD8                     | mov ebx,eax                             | ebx:&"詻B"
00442A23 | 85DB                     | test ebx,ebx                            | ebx:&"詻B"
00442A25 | 75 07                    | jne <along3x.1.loc_442A2E>              |
00442A27 | B8 01000000              | mov eax,1                               |
00442A2C | 5B                       | pop ebx                                 | ebx:&"詻B"
00442A2D | C3                       | ret                                     |
00442A2E | 8BC3                     | mov eax,ebx                             | ebx:&"詻B"
00442A30 | 48                       | dec eax                                 |
00442A31 | E8 EAFFFFFF              | call <along3x.1.sub_442A20>             |
00442A36 | F7EB                     | imul ebx                                | ebx:&"詻B"
00442A38 | 5B                       | pop ebx                                 | ebx:&"詻B"
00442A39 | C3                       | ret                                     |
```

这个函数虽然短,但是他是一个递归函数,做的是递归阶层的动作

```
00442A21 | 8BD8                     | mov ebx,eax                             | ebx:&"詻B"
00442A23 | 85DB                     | test ebx,ebx                            | ebx:&"詻B"
00442A25 | 75 07                    | jne <along3x.1.loc_442A2E>              |
00442A27 | B8 01000000              | mov eax,1                               |
00442A2C | 5B                       | pop ebx                                 | ebx:&"詻B"
00442A2D | C3                       | ret                                     |
```

eax是传入参数4,判断eax如果小于等于0,就`mov eax , 1`赋值,并向上递归

若果eax大于0

```
00442A2E | 8BC3                     | mov eax,ebx                             | ebx:&"詻B"
00442A30 | 48                       | dec eax                                 |
00442A31 | E8 EAFFFFFF              | call <along3x.1.sub_442A20>             |
00442A36 | F7EB                     | imul ebx                                | ebx:&"詻B"
00442A38 | 5B                       | pop ebx                                 | ebx:&"詻B"
00442A39 | C3                       | ret                                     |
```

就会以eax - 1作为参数再次调用`sub_442A20`函数,然后进行阶乘`imul ebx`

这里如果用js代码表示如下:

```
//TODO
let ebx = 0;
function sub_442A20(eax){
	if(eax <= 0){
		eax = 1
		return eax
	}
	let eax = sub_442A20(eax -1)
	
	eax = eax * ebx
	
}

```

所以函数最后的结果是4 * 3 * 2 * 1 = 24 ---- (十六进制:0x18)

### 继续sub_442AF4

![image-20230224082033511](D:\coding\blog\crack-me-006.assets\image-20230224082033511.png)

把`sub_442A20`的结果0x18存在`esi` , 并初始化ebx等于0

```
00442B3F | 8BF0                     | mov esi,eax                             |
00442B41 | 33DB                     | xor ebx,ebx                             | ebx:&"詻B"
```

获取Name的长度到eax

```
00442B43 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442B46 | E8 E90EFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442B4B | 85C0                     | test eax,eax                            |
00442B4D | 7E 16                    | jle <along3x.1.loc_442B65>              |
```

进行for循环运算:

```
00442B4B | 85C0                     | test eax,eax                            |
00442B4D | 7E 16                    | jle <along3x.1.loc_442B65>              |
00442B4F | BA 01000000              | mov edx,1                               | for循环开始 i = 1
00442B54 | 8B4D FC                  | mov ecx,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442B57 | 0FB64C11 FF              | movzx ecx,byte ptr ds:[ecx+edx-1]       | 每次获取Name[i - 1]
00442B5C | 0FAFCE                   | imul ecx,esi                            | Name[i - 1] * esi
00442B5F | 03D9                     | add ebx,ecx                             | 累加到ebx
00442B61 | 42                       | inc edx                                 | i++
00442B62 | 48                       | dec eax                                 |
00442B63 | 75 EF                    | jne <along3x.1.loc_442B54>              | 循环
```

循环次数是eax = Name的长度，edx初始化为1，每次递增1，每次循环获取Name[edx - 1] * esi , 累加到ebx.

所以for的结果是：0x6c * 0x18 + 0x71 * 0x18 + 0x63 * 0x18 + 0x6f * 0x18 + 0x64 * 0x18 + 0x65 * 0x18 = 0x3B40

for循环得出的结果与`[ebp-8]`相减,判断是否等于`0x7A69`

```
00442B65 | 2B5D F8                  | sub ebx,dword ptr ss:[ebp-8]            |
00442B68 | 81FB 697A0000            | cmp ebx,7A69                            | ebx:&"詻B"
00442B6E | 75 04                    | jne <along3x.1.loc_442B74>              |
00442B70 | B3 01                    | mov bl,1                                |
00442B72 | EB 06                    | jmp <along3x.1.loc_442B7A>              |
```

所以我们总结一下sub_442AF4的逻辑（Name输入`lqcode`）：

1. 判断Name输入是否大于5`

2. 获取Name[4]%7 + 2 = 4

3. 阶乘（2）的结果 4 * 3 * 2 * 1 = 24 (0x18)

4. 循环运算：

   0x6c * 0x18 + 0x71 * 0x18 + 0x63 * 0x18 + 0x6f * 0x18 + 0x64 * 0x18 + 0x65 * 0x18 = 0x3B40

5. 与Codice输入相减看是否等于`0x7A69`

所以当我们Codice输入`0x3B40 - 0x7A69 = ` **-16169** , 验证通过

![image-20230228074055202](D:\coding\blog\crack-me-006.assets\image-20230228074055202.png)

## 继续回到Ok控件点击事件

```
00442D8B | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          | eax:"lqcode", [ebx+2E0]:&"詻B"
00442D91 | E8 5A05FEFF              | call <along3x.1.TControl::SetText>      |
00442D96 | EB 3F                    | jmp <along3x.1.loc_442DD7>              |
00442D98 | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442D9B | 8B83 E0020000            | mov eax,dword ptr ds:[ebx+2E0]          | eax:"lqcode", [ebx+2E0]:&"詻B"
00442DA1 | E8 1A05FEFF              | call <along3x.1.TControl::GetText>      |
00442DA6 | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442DA9 | E8 C248FCFF              | call <along3x.1.@StrToInt>              |
00442DAE | 50                       | push eax                                | eax:"lqcode"
00442DAF | 8D55 FC                  | lea edx,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442DB2 | 8B83 DC020000            | mov eax,dword ptr ds:[ebx+2DC]          | eax:"lqcode", [ebx+2DC]:&"詻B"
00442DB8 | E8 0305FEFF              | call <along3x.1.TControl::GetText>      |
00442DBD | 8B45 FC                  | mov eax,dword ptr ss:[ebp-4]            | [ebp-4]:"lqcode"
00442DC0 | 5A                       | pop edx                                 |
00442DC1 | E8 DAFDFFFF              | call <along3x.1.sub_442BA0>             |	
```

获取输入的之后运行函数`sub_442BA0`

### sub_442BA0

```
00442BC8 | 8D55 F8                  | lea edx,dword ptr ss:[ebp-8]            |
00442BCB | 8BC6                     | mov eax,esi                             |
00442BCD | E8 6E4AFCFF              | call <along3x.1.@IntToStr>              |
00442BD2 | 8D45 F4                  | lea eax,dword ptr ss:[ebp-C]            |
00442BD5 | 8B55 F8                  | mov edx,dword ptr ss:[ebp-8]            |
00442BD8 | E8 730CFCFF              | call <along3x.1.__linkproc__ LStrLAsg>  |
00442BDD | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            |
00442BE0 | E8 4F0EFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442BE5 | 83F8 05                  | cmp eax,5                               |
00442BE8 | 7E 60                    | jle <along3x.1.loc_442C4A>              |
00442BEA | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            |
00442BED | E8 420EFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442BF2 | 8BF0                     | mov esi,eax                             |
00442BF4 | 83FE 01                  | cmp esi,1                               |
00442BF7 | 7C 2F                    | jl <along3x.1.loc_442C28>               |
00442BF9 | 8D45 F4                  | lea eax,dword ptr ss:[ebp-C]            |
00442BFC | E8 0310FCFF              | call <along3x.1.@UniqueString>          |
00442C01 | 8D4430 FF                | lea eax,dword ptr ds:[eax+esi-1]        |
00442C05 | 50                       | push eax                                |
00442C06 | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            |
00442C09 | 0FB64430 FF              | movzx eax,byte ptr ds:[eax+esi-1]       |
00442C0E | F7E8                     | imul eax                                |
00442C10 | 0FBFC0                   | movsx eax,ax                            |
00442C13 | F7EE                     | imul esi                                |
00442C15 | B9 19000000              | mov ecx,19                              |
00442C1A | 99                       | cdq                                     |
00442C1B | F7F9                     | idiv ecx                                |
00442C1D | 83C2 41                  | add edx,41                              |
00442C20 | 58                       | pop eax                                 |
00442C21 | 8810                     | mov byte ptr ds:[eax],dl                |
00442C23 | 4E                       | dec esi                                 |
00442C24 | 85F6                     | test esi,esi                            |
00442C26 | 75 D1                    | jne <along3x.1.loc_442BF9>              |
00442C28 | 8B45 F4                  | mov eax,dword ptr ss:[ebp-C]            |
00442C2B | 8B55 FC                  | mov edx,dword ptr ss:[ebp-4]            | [ebp-4]:&"斢@"
00442C2E | E8 110FFCFF              | call <along3x.1.__linkproc__ LStrCmp>   |
00442C33 | 75 17                    | jne <along3x.1.loc_442C4C>              |
```

这段代码分为三部分

第一部分是获取Codice输入到`edx`

第二部分是经过算法把Codice的输入转换为特定字符串

第三部分是把算法的字符串与Name比较是否相同

---

我们经过上一轮的校验Codice的输入值是`-16169`

所以首先获取到`-16169`转为为字符串，存到edx，并判断是否大于5

```
00442BC3 | 64:8920                  | mov dword ptr fs:[eax],esp              |
00442BC6 | 33DB                     | xor ebx,ebx                             |
00442BC8 | 8D55 F8                  | lea edx,dword ptr ss:[ebp-8]            | [ebp-8]:"-16169"
00442BCB | 8BC6                     | mov eax,esi                             |
00442BCD | E8 6E4AFCFF              | call <along3x.1.@IntToStr>              |
00442BD2 | 8D45 F4                  | lea eax,dword ptr ss:[ebp-C]            | [ebp-C]:"-16169"
00442BD5 | 8B55 F8                  | mov edx,dword ptr ss:[ebp-8]            | [ebp-8]:"-16169"
00442BD8 | E8 730CFCFF              | call <along3x.1.__linkproc__ LStrLAsg>  |
00442BDD | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            | [ebp-8]:"-16169"
00442BE0 | E8 4F0EFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442BE5 | 83F8 05                  | cmp eax,5                               |
00442BE8 | 7E 60                    | jle <along3x.1.loc_442C4A>              |
```

开始循环处理字符串

```
00442BEA | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            | [ebp-8]:"-16169"
00442BED | E8 420EFCFF              | call <along3x.1.__linkproc__ LStrLen>   |
00442BF2 | 8BF0                     | mov esi,eax                             |
00442BF4 | 83FE 01                  | cmp esi,1                               |
00442BF7 | 7C 2F                    | jl <along3x.1.loc_442C28>               |
00442BF9 | 8D45 F4                  | lea eax,dword ptr ss:[ebp-C]            | [ebp-C]:"-16169"
00442BFC | E8 0310FCFF              | call <along3x.1.@UniqueString>          |
00442C01 | 8D4430 FF                | lea eax,dword ptr ds:[eax+esi-1]        |
00442C05 | 50                       | push eax                                |
00442C06 | 8B45 F8                  | mov eax,dword ptr ss:[ebp-8]            | [ebp-8]:"-16169"
00442C09 | 0FB64430 FF              | movzx eax,byte ptr ds:[eax+esi-1]       |
00442C0E | F7E8                     | imul eax                                |
00442C10 | 0FBFC0                   | movsx eax,ax                            |
00442C13 | F7EE                     | imul esi                                |
00442C15 | B9 19000000              | mov ecx,19                              |
00442C1A | 99                       | cdq                                     |
00442C1B | F7F9                     | idiv ecx                                |
00442C1D | 83C2 41                  | add edx,41                              |
00442C20 | 58                       | pop eax                                 |
00442C21 | 8810                     | mov byte ptr ds:[eax],dl                |
00442C23 | 4E                       | dec esi                                 |
00442C24 | 85F6                     | test esi,esi                            |
00442C26 | 75 D1                    | jne <along3x.1.loc_442BF9>              |
```

其中esi是字符串（-16169）的长度等于6 （包括符号），eax是字符串的值`-16169`

1. 循环获取`[eax+esi-1] `就是从尾到头获取字符. 

   ![image-20230228082409296](D:\coding\blog\crack-me-006.assets\image-20230228082409296.png)

2. 对字符进行自乘   ---> 0x39 * 0x39 = 0xCB1

3. 获取(2)结果的低4位乘以`esi` (字符串长度)   ------> 0xCB1 * 0x6 = 0x4c26

   ![image-20230228082649609](D:\coding\blog\crack-me-006.assets\image-20230228082649609.png)

4.  对(3)的结果除以19取余,余数加上41,转换成字符  0x4c26 % 0x19 + 0x41 = 0x54

   ![image-20230228082713795](D:\coding\blog\crack-me-006.assets\image-20230228082713795.png)

5. 所以得出字符`T`

---

循环过后,最终的出字符串是`ACXEFT`

![image-20230228082759540](D:\coding\blog\crack-me-006.assets\image-20230228082759540.png)

所以当我们修改Name的值为`ACXEFT`OK点击事件就会通过

![image-20230228083039754](D:\coding\blog\crack-me-006.assets\image-20230228083039754.png)



# 总结

程序一共有三个判断逻辑

1. `sub_442A3C`函数,是当Name和Codice输入变化时候调用,但是这个判断经过分析下来应该作者的迷惑烟雾
2. `sub_442AF4`函数,点击Cancella按键会触发
3. `sub_442BA0`函数,点击OK按键会触发,但是这个函数的触发条件是Cancella按键先消失

接下来就能总结出程序的注册的完整逻辑:

1. Name输入长度大于5的字符串
2. 根据`sub_442AF4`函数反推`Codice`的值点击Cancella让Cancella消失
3. 根据`sub_442BA0`函数反推修改`Name`的值,点击OK让OK按键消失
4. crack~~

# 知识点

1. 这个程序很多都是for循环的盘山汇编,可以用来锻炼在汇编中看for循环逻辑.



