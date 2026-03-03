---
title: WPF中使用ctrl多选的bug
date: 2025-01-19 17:32:47+08:00
description: 开发WPF程序时，错误地使用PreviewMouseLeftButtonDown和ctrl多选，由此造成了一个bug。
tags:
- WPF
- bug
featured_image: ''
images: []
categories: 经验
comment: false
draft: false
---

WPF中使用ctrl多选的bug"
date: 2025-01-19T17:32:47+08:00
description: "开发WPF程序时，错误地使用PreviewMouseLeftButtonDown和ctrl多选，由此造成了一个bug。"
tags: [WPF, bug]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
draft: false
---

**最后编辑于2025年01月19日**

# 前言

好久没写blog了，近期挺忙的。

这次是在尝试在C#下的另一个桌面开发框架WPF中开发一个小程序时，遇到的一个小bug，纯纯是我自己没理解Preview事件导致的。

---

# 背景知识记录

Drag-and-drop操作过程中，会由一系列事件Event组成，这些Event有两种模型，一个是冒泡Bubbling，一个是隧道Tunnelling（带Preview前缀的）。

**Bubbling的event是动作发生后触发；Tunnelling的event是动作发生前触发。**

## 冒泡事件

从最具体的子元素开始，逐层向上冒泡到父元素。如果子元素处理了事件并设置了 e.Handled = true，则父元素不会接收到该事件。

## 隧道事件

从最不具体的父元素开始，逐层向下传递到子元素。如果父元素处理了事件并设置了 e.Handled = true，则子元素不会接收到该事件。

# bug场景复现

我在一个ListBox里面设置了Extended多选，即可以通过按住ctrl来实现多选，这个多选事件是发生在我自己的PreviewMouseLeftButtonDown事件之后的（但是不清除它是冒泡还是隧道事件）。而我的PreviewMouseLeftButtonDown事件里面设定了如下代码：

```c#
// 选中当前项
listBoxItem.IsSelected = true;

e.Handled = true
```

当时拖动文件的代码是写在PreviewMouseLeftButtonDown事件里面的，而拖动文件的操作是阻塞的。我这样写是为了在拖动子项时，子项能变成选中状态。

然后我就发现按住ctrl无法使已选中的项变成未选中。想了一下，应该是因为`e.Handled = true`，使得接下来按住ctrl选择没有接收到点击事件，但是当我删除`e.Handled = true`之后，依然存在bug：

- 按住ctrl可以使已选中的项变成未选中
- 但是按住ctrl无法使未选中的项变成已选中

# bug原因

后来经过我的分析之后，得出了原因：`listBoxItem.IsSelected = true`和ctrl多选依次发生，导致出现bug。这个过程可以如下描述：

1. 点击之后，`listBoxItem.IsSelected = true`正常发生，这时子项被选中。
2. 当PreviewMouseLeftButtonDown事件结束之后，ctrl多选发生，它会改变子项的选中状态，可以理解为进行了非操作。
3. 最后子项的选中状态是`false`。

所以ctrl多选就只能使子项变成非选中状态。

# 解决方法

解决也很简单，只需要将拖动文件的代码转移到MouseMove事件里面，并删除上面的两行代码即可。

因为拖动文件的代码在MouseMove事件里面，所以PreviewMouseLeftButtonDown事件不再会阻塞ctrl多选的事件，这样ctrl多选的各种行为也能正常发生了。

# 最后总结

Preview事件里面不要放一些会阻塞的代码，最好只放一些检查、记录的代码。我这里就改成记录选中项和鼠标位置了。

# 参考

---

> [【WPF】鼠标拖拽功能DragOver和Drop](https://www.cnblogs.com/mqxs/p/7534730.html)
>
> [（原创）[C#] 一步一步自定义拖拽（Drag&Drop）时的鼠标效果：（一）基本原理及基本实现 
](https://www.cnblogs.com/lesliexin/p/16073879.html)
>
> [Kimi](https://kimi.moonshot.cn/)