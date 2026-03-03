---
title: C#获取文件的图标
date: 2024-11-09 16:24:42+08:00
description: 在NET里面用C#来获取shell32.dll里面或者文件的图标
tags:
- C#
- dll
- 图标
featured_image: NET.png
images: []
categories: 经验
comment: false
---

"C#获取文件的图标"
date: 2024-11-09T16:24:42+08:00
description: "在NET里面用C#来获取shell32.dll里面或者文件的图标"
tags: ["C#", "dll", "图标"]
featured_image: "/images/NET.png"
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
---

**最后编辑于2024年11月24日**

# 前言

在搞那个快捷方式可视化的net程序时，因为图标列表第一个是百度网盘的图标，而文件夹没有指定图标，就默认第一个。结果就是一堆百度网盘😅。

就算指定文件夹节点的图标索引为-1也还是这样，不知道为什么，所以就要搞一个文件夹图标。

下面的方法都可以提取可执行文件、DLL 或图标文件里面的图标，但是能提取任意文件的~~只有方法三~~，现在有方法三和方法五了。

---

# 方法一

主要是通过shell32 API的ExtractIcon函数来提取，然后文件夹图标的索引在shell32.dll里面是3。代码如下

```c#
public class ShellIcon
{
    [DllImport("shell32.dll", CharSet = CharSet.Auto)]
    private static extern IntPtr ExtractIcon(IntPtr hInst, string lpszExeFileName, int nIconIndex);

    public static Icon? GetFolderIcon(int index)
    {
        // 从shell32.dll提取文件夹图标 (索引3是标准的关闭文件夹图标，从0开始)
        IntPtr hIcon = ExtractIcon(IntPtr.Zero, "shell32.dll", index);

        if (hIcon != IntPtr.Zero)
        {
            return Icon.FromHandle(hIcon);
        }

        return null;
    }
}

.......

// 添加文件夹图标
Icon folderIcon = ShellIcon.GetFolderIcon(3);
if (folderIcon != null)
{
    imageList1.Images.Add("folder", folderIcon);
}
``` 

根据微软的官网中shell32里面的函数介绍，ExtractIconEx也可以，而且还可以同时获取大小图标，但是使用起来略显麻烦，需要传入图标数组的句柄（说是数组是因为这个函数可以提取多个图标），而且获取完之后要先从句柄创建图标- 复制图标，然后消除句柄。下面是函数的使用例子：

```c#
[DllImport("shell32.dll", CharSet = CharSet.Unicode)]
public static extern uint ExtractIconEx(string lpszFile, 
    int nIconIndex, 
    out IntPtr phiconLarge, 
    out IntPtr phiconSmall, 
    uint nIcons);

[DllImport("user32.dll")]
private static extern bool DestroyIcon(IntPtr hIcon);

uint iconsExtracted = ExtractIconEx(dllPath, // 文件路径
    index, // 图标起始索引，注意是从0开始的
    out largeIconHandle, // 大图标数组句柄0
    out smallIconHandle, // 
    1 // 提取图标数);

// 从句柄创建Icon
Icon icon = Icon.FromHandle(largeIconHandle);
// 创建icon的克隆，这样我们就可以安全地销毁原始句柄
Icon clonedIcon = (Icon)icon.Clone();

// 清理资源
DestroyIcon(largeIconHandle);
```

## 参考

> [ExtractIcon](https://learn.microsoft.com/zh-cn/windows/win32/api/shellapi/nf-shellapi-extracticona)
>
> [ExtractIconEx](https://learn.microsoft.com/zh-cn/windows/win32/api/shellapi/nf-shellapi-extracticonexa)

---

# 方法二

还可以使用shell32里面的SHDefExtractIcon，这个也需要清除句柄:
    
```c#
[DllImport("shell32.dll", CharSet = CharSet.Auto)]
private static extern int SHDefExtractIcon(
    string pszIconFile,
    int iIndex,
    uint flags,
    out IntPtr phiconLarge,
    out IntPtr phiconSmall,
    uint nIconSize);

[DllImport("user32.dll")]
private static extern bool DestroyIcon(IntPtr hIcon);

// 从shell32.dll提取文件夹图标 (索引3是标准文件夹)
int result = SHDefExtractIcon(
    "shell32.dll",      // 图标文件
    3,                  // 图标索引
    0,                  // 标志
    out hIconLarge,     // 大图标句柄
    out hIconSmall,     // 小图标句柄
    0                   // 使用默认尺寸);
```

不过那个**nIconSize**我没有搞懂。

## 参考

> [SHDefExtractIcon](https://learn.microsoft.com/zh-cn/windows/win32/api/shlobj_core/nf-shlobj_core-shdefextracticonw)

---

# 方法三

这个是使用shell32的**SHGetFileInfo**函数，加上**SHFILEINFO**结构的引用变量（引用就是函数内部对变量本身修改，有点类似传指针进去）和标志组合，具体的标志内容可以去微软的官网看。

这个方法就可以获取任意文件的图标。但是我不知道怎么读取shell32.dll里面的图标。

```c#
[StructLayout(LayoutKind.Sequential)]
public struct SHFILEINFO
{
    public IntPtr hIcon;
    public int iIcon;
    public uint dwAttributes;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
    public string szDisplayName;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 80)]
    public string szTypeName;
}

[DllImport("shell32.dll", CharSet = CharSet.Auto)]
public static extern IntPtr SHGetFileInfo(string pszPath, 
    uint dwFileAttributes,
    ref SHFILEINFO psfi, 
    uint cbSizeFileInfo, 
    uint uFlags);

[DllImport("user32.dll")]
public static extern bool DestroyIcon(IntPtr hIcon);

public const uint SHGFI_ICON = 0x100;
public const uint SHGFI_LARGEICON = 0x0;    // 大图标
public const uint SHGFI_SMALLICON = 0x1;    // 小图标

SHFILEINFO shfi = new SHFILEINFO();
uint flags = SHGFI_ICON | SHGFI_LARGEICON;

// 获取图标
IntPtr result = SHGetFileInfo(
    @"D:\软件\tModLoader.lnk",
    FILE_ATTRIBUTE_DIRECTORY,
    ref shfi,
    (uint)Marshal.SizeOf(shfi),
    flags);
```

## 参考

> [SHGetFileInfo](https://learn.microsoft.com/zh-cn/windows/win32/api/shellapi/nf-shellapi-shgetfileinfoa)
>
> [SHFILEINFO](https://learn.microsoft.com/zh-cn/windows/win32/api/shellapi/ns-shellapi-shfileinfoa)

---

# 方法四

这个方法就可以获取更大尺寸的图标，比如256*256，使用的是user32.dll里面的PrivateExtractIcons函数，不过这个函数可能会在后续Windows版本移除。

```c#
[DllImport("User32.dll", CharSet = CharSet.Auto)]
public static extern int PrivateExtractIcons(
    string lpszFile, //文件名可以是exe,dll,ico,cur,ani,bmp
    int nIconIndex,  //从第几个图标开始获取
    int cxIcon,      //获取图标的尺寸x
    int cyIcon,      //获取图标的尺寸y
    IntPtr[] phicon, //获取到的图标指针数组
    int[] piconid,   //图标对应的资源编号
    int nIcons,      //指定获取的图标数量，仅当文件类型为.exe 和 .dll时候可用
    int flags        //标志，默认0就可以，具体可以看LoadImage函数
);

[DllImport("user32.dll")]
public static extern bool DestroyIcon(IntPtr hIcon);

// 从shell32.dll提取文件夹图标 (索引3是标准文件夹)
int result = PrivateExtractIcons(
    "shell32.dll",      // 图标文件
    3,                  // 图标索引
    32, 
    32, 
    hIcons,
    ids, 
    1,
    0);
```

## 参考

> [C# 获取exe、dll中的图标，支持获取256x256分辨率](https://www.cnblogs.com/JmlSaul/p/7515885.html)
>
> [PrivateExtractIcons](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-privateextracticonsw)
>

# 方法五

这个方法是搜索c#的ExtractAssociatedIcon函数的底层时知道的，就是shell32的ExtractAssociatedIcon。（虽然不知道c#的底层是不是这个），但是可以提取文件图标：

```c#
// 导入shell32.dll的ExtractAssociatedIcon函数
[DllImport("shell32.dll", CharSet = CharSet.Unicode)]
private static extern IntPtr ExtractAssociatedIcon(
    IntPtr hInst,
    string lpIconPath, 
    ref ushort lpiIcon);
    
ushort iconIndex = 0;
IntPtr hInst = IntPtr.Zero;
IntPtr hIcon = ExtractAssociatedIcon(hInst, filePath, ref iconIndex);
```

## 参考
> [ExtractAssociatedIconW 函数](https://learn.microsoft.com/zh-cn/windows/win32/api/shellapi/nf-shellapi-extractassociatediconw)
>

# 参考

---

> Claude~~又是AI😋~~