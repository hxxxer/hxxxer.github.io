---
title: Git彻底删除历史中的大文件
date: 2024-11-12 17:00:34+08:00
description: git删除那些已经没用或者不小心提交的大文件，包括本地和远程
tags: []
featured_image: ''
images: []
categories: null
comment: false
---

**最后编辑于2024年11月12日**

# 前言

在搞go+fyne移植一个net项目时，不小心把一个检测fyne是否已经配置好的`Fyne Setup.exe`给提交了，过了几次提交才发现，所以就要删除。

---

为了通用性考虑，记录就从寻找大文件开始。

下面的命令都是在git bash里面执行。

# 找出大文件

运行下面的命令列出文件大小的top10：

```sh
git rev-list --all | xargs -rL1 git ls-tree -r --long | sort -uk3 | sort -rnk4 | head -10
```

下面的命令也可以，这个是列出全部文件的大小，和上面相反，是升序：

```sh
git rev-list --objects --all \
| git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
| sed -n 's/^blob //p' \
| sort --numeric-sort --key=2 \
| cut -c 1-12,41- \
| $(command -v gnumfmt || echo numfmt) --field=2 --to=iec-i --suffix=B --padding=7 --round=nearest
```

# 移除commit中文件的引用

该命令仅仅只是移除commit中对某文件的引用,如果某次commit同时提交了要删除的文件和要保留的文件，该命令只会移除要删除文件的引用，并不会移除要保留的文件和当次commit记录，所以无需担心会把不该删除的文件删掉了。

```sh
git filter-branch --force --prune-empty --index-filter 'git rm -rf --cached --ignore-unmatch 文件路径/文件名' --tag-name-filter cat -- --all
```

我在这一步怎么都删除不了，应该是因为我的文件名带空格，不过考虑到exe文件就这一个，我就用`*.exe`代替了。
# 删除指向旧提交的指针

```sh
rm -rf .git/refs/original/
```

# 让历史记录全部过期

设置所有未关联对象过期时间为现在，默认为90天。

目的是放弃所有未关联对象恢复的可能性，因为reflog 是找寻它们踪迹的最后途径了。

```sh
git reflog expire --expire=now --all
```

# 重新打包并删除垃圾文件

```sh
git repack -A -d
git gc --aggressive --prune=now
```

# 强制推送到远程

```sh
git push --force --all
```

# 最后

如果这个项目有其他人，那大家都要在强制推送之后clone一下，防止又把垃圾文件push上去。

# 参考

---
> [git项目大小优化笔记,删除历史提交中的大文件 ](https://www.cnblogs.com/fuhua/p/15527023.html)
>
> [彻底删除git中的较大文件（包括历史提交记录）](https://blog.csdn.net/HappyRocking/article/details/89313501)
>

