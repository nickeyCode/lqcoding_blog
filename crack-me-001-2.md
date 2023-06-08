![image-20221127161420217](D:\coding\blog\crack-me-001-2.assets\image-20221127161420217.png)

随便输入一个验证意料之中的失败,然后找突破口

![image-20221127161608369](D:\coding\blog\crack-me-001-2.assets\image-20221127161608369.png)

在提示文字处添加断点

找到关键跳转

![image-20221127161736436](D:\coding\blog\crack-me-001-2.assets\image-20221127161736436.png)

在jne前的函数call acid burn.4039FC 有讲个变量分别是[ebp--0x10],[ebp--0xc]

![image-20221127181649339](D:\coding\blog\crack-me-001-2.assets\image-20221127181649339.png)

[ebp--0x10] 是我们的输入

[ebp--0xc] 是字符串

猜测如果两个字符串相等就能进入正确循环

# 验证

![image-20221127181847948](D:\coding\blog\crack-me-001-2.assets\image-20221127181847948.png)