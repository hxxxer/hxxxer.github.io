---
title: C# ToString的使用
date: 2024-11-17 01:04:14+08:00
description: C#中数值转字符串的函数的使用方法
tags:
- C#
- 字符串
- 数值
- 四舍五入
featured_image: ''
images: []
categories: 经验
comment: false
---

**最后编辑于2024年11月17日**

# 前言

在看一篇有关获取图标的博客的时候，看到了这个将数值转字符串的函数，就记录一下。

---

# 常见使用方法

可以直接使用一个字母字符代表转换模式：

```c#
ToString("C{x}") // 货币模式，x代表保留几位小数，默认两位
ToString("D{x}") // 十进制，x代表有几位数，比如x为3，15就转换为015
ToString("E{x}") // 科学计数法，x代表底数保留几位小数
ToString("F{x}") // 小数，x代表底数保留几位小数
ToString("G") // 直接转换
ToString("N{x}") // 使用 , 作为分隔符，x代表保留几位小数，默认两位
ToString("X{x}") // 十六进制，x代表有几位数
```

也可以使用多个**0**代表转换模式，例子直接说明（Assert.AreEqual()是断言函数，就是判断）：

```c#
var value = 67.89;
Assert.AreEqual("68", value.ToString("0"));
Assert.AreEqual("68", value.ToString("00"));
Assert.AreEqual("068", value.ToString("000"));
Assert.AreEqual("067.9", value.ToString("000.0"));
Assert.AreEqual("067.89", value.ToString("000.00"));
Assert.AreEqual("067.890", value.ToString("000.000"));
```

这个模式和上面的模式中的部分都可以四舍五入。

# 参考

---

> [c# - C# ToString("00") 歧义](https://stackoverflow.org.cn/questions/17937487)
>
> [C# 之 ToString() 格式化全说明](https://blog.csdn.net/dmlk31/article/details/111206738)
>
