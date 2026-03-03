---
title: "CSS实现文本渐变消失"
date: 2024-11-14T01:45:42+08:00
description: "通过CSS的蒙版和渐变，实现一个元素的渐变消失。当然也可以应用到其它的领域"
tags: ["css"]
featured_image: ""
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
---

**最后编辑于2024年11月14日**

# 前言

在搞博客搜索的时候，由于摘要只截了120个字符，导致我想在末尾加个省略号，加完之后又想加一个渐变消失的效果。

虽然最后也没有加上😂。

---

# 代码展示

下面是文本html：

```html
<p class="wraps">aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa</p>
```

下面是从左到右，渐渐消失的效果：

```css
.wraps{
        -webkit-mask-image: -webkit-linear-gradient(right, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 1) 70%); 
<!--	background: #fff;-->
        width: 500px;
        height: 30px;
<!--	border: 1px solid red;-->
```

改成从右到左只需要将上面css代码第二行的right改成left即可。

# 代码解释

像`background`、`width`、`height`就纯纯一看便知，而border是指一个框的效果；真正要解释的就第二行。

- `-webkit-mask-image`是类似于ps的蒙版效果，蒙版上越黑越透明。

- `-webkit-linear-gradient`是生成一种或多种颜色进行线性过渡的效果的图像，第一个参数是方向，后面的是颜色。以从左到右渐渐消失的为例：
    1. 方向为右，意味着是从右边开始，由第一种颜色向第二种颜色过渡。
    2. rgba的第四个参数是**不透明度**，默认为1，即不透明。
    3. 颜色参数后面那个百分数，意思是从方向上的路径的多少。比如right+30%就是以最左边为起点（0%），向右30%。
    4. 根据上面3点，可以得出效果如下图所示：
        ![过渡黑白图](/images/黑白过渡.png)
        
最后就能得出从左到右逐渐消失的效果。

其实和PS的蒙版可以说是一样的。

注意：上面的的70%等表示位置的是以元素的框的大小为标准的（这个框可以在F12里面看），不是以文字的长度为标准的。

# 补充

MDN上面的方法是没有`-webkit`前缀的，暂时不清楚为什么。

# 参考

---

> [css div怎么渐变到透明](https://segmentfault.com/q/1010000022493842)
>
> [MDN linear-gradient()](https://developer.mozilla.org/zh-CN/docs/Web/CSS/gradient/linear-gradient)
>
