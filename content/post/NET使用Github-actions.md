---
title: "NET使用Github Actions"
date: 2024-11-08T01:53:42+08:00
description: "NET项目使用actions自动构建发布"
tags: ["NET", "actions", "自动化"]
featured_image: "/images/GitHub-Symbol.png"
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
---

**最后编辑于2024年11月09日**

# 前言

早就知道GitHub actions很方便好用，所以我想搞一个。不过遇到了好多困难，现在基本搞定了，记录一下。

主要的难点是yml文件的编写，毕竟是第一次接触，脑子有点混。

---

# 新建工作流

就直接在actions里面新建就行，GitHub有net desktop模板。

---

# yml文件的理解

开头的on是指当发生什么的时候开始这条工作流，比如：

```yml
on:
  push:
    tags:
      - v*
```

就是在push且tag是以v开头时启动。注意push内的每个条件是或关系，只要满足一个就开始工作。

然后就是具体工作内容，是一种多级列表，每一级都可以有多个。

其实工作流程就是大概按顺序运行steps，这里大概是：
1. 启动系统
2. 拉取仓库
3. 安装net core
4. net项目初始化依赖
5. 构建release版本
6. 压缩成zip
7. 发布到release

感觉就像自动执行指令。

## steps属性理解

最末级就是steps，里面的属性有：

- **name**，就是名字。
- **id**，非必要，其它地方引用时候用。
- **uses**，可以选择GitHub上现有的操作，比如获取仓库内容、安装net vore。
- **with**，一般是使用uses时配置具体的参数，比如安装net core时指定版本。
- **run**，就是执行指令，不同平台（win，Ubuntu）使用的不同
- **env**，一些变量，通过$env:<名字>，来引用。（好像在一些字符串里面不能被正确解析），比如：
	```yml
	run: |
        $zipFilePath = "${{ github.workspace }}\$env:"
	```

## 发布到release

这边是重点，我是先搞了一个给这个action用的token，然后再在yml文件里面设定steps。

### TOKEN

#### ~~新建TOKEN~~

在头像 - **setting** - 侧边栏的**Developer Settings** - 侧边栏的**Personal accesskey tokens** - **Token (classic)**。然后**generate new token** - **Generate new token (classic)**。然后输密码之后，来到新建token的页面，名字随便起，过期时间随便，下面是**Select scopes**选择前三个。
**
新建完成之后记得复制token，因为只会出现一次，后面要用。

![选择token类型](/images/选择token类型.png)

![新建token](/images/新建token.png)

#### ~~新建secrets~~

刚刚的token要保存到项目的secrets里面。secrets是一种可以在yml里面调用的变量，但是无法得知内容。

打开项目的**setting** - 侧边栏的**Secrets and variables** - **Actions**。然后选择绿色的**New repository secret**。接下来就是新建页面，名字随便，记住就行，secret就填刚刚的token。然后保存就行。

但是还没完，还要配置项目的权限。打开项目的**setting** - 侧边栏的**Actions** - **General**，找到**Workflow permission**，选择第一个Read and write permissions。到这里就完成了。

#### 补充

后来发现不用新建，直接在发布的`with: token`里用`${{ secrets.GITHUB_TOKEN }}`就行😂。

### 在yml文件设定发布

这里我是使用别人的actions和网友的模板：

```yml
- name: Create and Upload Release
  uses: softprops/action-gh-release@v2
  if: startsWith(github.ref, 'refs/tags/')
  with:
    token: ${{ secrets.PB_TOKEN }}
    tag_name: ${{ github.ref_name }}
    name: Release ${{ github.ref_name }}
    body: 船新版本！
    makelatest: true
    draft: false
    prerelease: false
    files: |
      HugoHelper.zip
```

`github.ref_name`就是上传连接的名字，比如我是限制在tags前缀是v时启动工作流，那这里就是v...。

`name`和`tag_name`顾名思义，`name`就是这次发布的名字，`tag_name`就是发布的tag的名字。

`makelatest`就是让这次发布设置为项目的最新release。

# 参考

---

> [Github Actions自动发布release](https://blog.csdn.net/m0_74075298/article/details/141230617)
>
> [学会Github Actions自动发布版本](https://segmentfault.com/a/1190000045270503)
>
> [GitHub Action 自动构建 并release](https://cloud.tencent.com/developer/article/1970730)
>
> [GitHub Actions - 使用 tag 作为发布的版本号](https://juejin.cn/post/6908427616298991629)
>
> [.NET Core项目提示找不到project.assets.json文件](https://www.cjavapy.com/article/328/)