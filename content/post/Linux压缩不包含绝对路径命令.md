---
title: "Linux压缩不包含绝对路径命令"
date: 2024-11-11T00:16:42+08:00
description: "在Linux压缩文件不包含绝对路径，只有指定目录下的文件"
tags: ["压缩", "Linux"]
featured_image: 
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
---

**最后编辑于2024年11月11日**

# 前言

在GitHub actions自动发布的时候，需要使用`zip`命令来压缩发布文件，但是会连带很多上级目录一起压缩，需要去除这些目录。

---

# 操作

原来的命令是这样的：

```sh
zip -r ShortcutVisualizer.zip ./ShortcutVisualizer/bin/Release/net6.0-windows/win-x64/publish
```

这样在压缩出来的时候就会带上`ShortcutVisualizer/bin/Release/net6.0-windows/win-x64/publish/`这些无关的文件夹。

想要去除只需要将`-r`更改为`-rj`。

---

# 补充

使用tar命令压缩的话，只需要在末尾使用-C指定文件所在目录就行，其本质是切换工作目录，来达到工作目录之前的文件夹不压缩的目的。

# 参考

---
> [Linux下压缩不包含路径信息的压缩包](https://www.cnblogs.com/iamfy/archive/2012/11/10/2764307.html)
>
> [Shell创建zip文件不包含完整路径方法](https://www.cnblogs.com/azureology/p/13217461.html)
>

