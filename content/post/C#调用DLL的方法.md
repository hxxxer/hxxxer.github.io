---
title: "C#调用DLL的方法"
date: 2024-11-11T00:29:13+08:00
description: "在C#中调用外部dll的函数的两种方法"
tags: ["C#", "dll"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
---

**最后编辑于2024年11月11日**

# 前言

之前需要调用dll里面的图标，而调用的函数用的是一个dll的内部的函数，所以这里记录一下调用的两种方法。

---

# VS的项目引用

在VS的方案资源管理器里面（`Ctrl+Alt+L`），右键依赖项 - 添加项目引用，然后选择dll文件，例如AddDll.dll（里面有一个ADD类，ADD里面有一个calculate公共方法）。

之后就可以在依赖项 - 程序集里面看到Add了。

调用也很简单：

```c#
using AddDll;
AddDll.ADD add = new();
int c = add.calculate(a,b);
```

---

# 使用DllImport

在需要引用的函数的上一行添加`[DllImport("user.dll", ......)]`就行，下面是我调用的获取dll图标函数：

```c#
[DllImport("shell32.dll", CharSet = CharSet.Auto)]
private static extern IntPtr ExtractIcon(IntPtr hInst, string lpszExeFileName, int nIconIndex);
```

后面那个`CharSet = CharSet.Auto`是调用字符集，Auto是自动选择合适的。

下面函数的定义记得加`static extern`，以表示这个是一个外部方法实现，它将在运行时解析到指定的非托管代码。

这个方法需要知道dll内部的函数及其形参。

## 补充其它的DllImportAttribute属性

- `EntryPoint="MessageBoxA"`，这个可以也指定函数，这样下面的函数引用就可以用别的名字，但是注意这个EntryPoint赋值和下面的函数引用不一定相同，比如上面的`ExtractIcon`，在EntryPoint里面应该是`ExtractIconW`。
- `ExactSpelling` 指示 `EntryPoint` 是否必须与指示的入口点的拼写完全匹配，如：`ExactSpelling=false`；
- `PreserveSig`指示方法的签名应当被保留还是被转换， 如：`PreserveSig=true`；
- `SetLastError` 指示方法是否保留 Win32"上一错误"，如：`SetLastError=true`；
- `CallingConvention`指示入口点的调用约定， 如：`CallingConvention=CallingConvention.Winapi`；

# 参考

---

> [C#调用DLL的几种方法](https://www.cnblogs.com/Im-Victor/p/14708695.html)
>
> [【C#】DllImport的使用](https://blog.csdn.net/wangnaisheng/article/details/142462074)
>
> [VS2022 C#编写DLL和调用外部DLL中的方法](https://blog.csdn.net/qq_43069920/article/details/123100601)
>

