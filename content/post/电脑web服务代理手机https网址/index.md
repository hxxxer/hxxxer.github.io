---
title: 电脑web服务代理手机https网址访问
date: 2025-12-12 01:02:27+08:00
description: 在手机连接电脑热点的情况下，使手机访问一个无法访问的https网址能连接到电脑的web服务
tags:
- https
- web服务器
- 子网
featured_image: ''
images: []
categories: 经验
comment: false
---

**最后编辑于2025年12月12日**

# 前言

之前用手机的一个网页记录打胶，把数据保存在别人的演示服务器上(https://dick.juwo.my/)，结果过了几个月，别人的服务器下线了……我的数据全都没了，但是我知道它应该是保存在浏览器的localStorage上（因为后面自己做了一个仿版，保存的数据也是保存在localStorage。同时它的项目readme也说了它的数据是保存在本机，这种小数据大概率是保存在localStorage）。

然后我的浏览器是可以打开开发工具台的，**但是**，只能在可以正常加载的页面才可以打开……

所以我就想~~劫持~~这个网址，让手机访问这个网址能连接到电脑的web服务。

---

我开始之前就大概感觉到这个会比较复杂，因为涉及到了https，不是平时常用的web服务器对应的http。最后做完之后总结一下难点

# 难点

- 手机访问电脑热点的ip地址怎么连接到web服务器。
- 手机怎么用host，使访问dick.juwo.my时能连接电脑热点的ip地址。
- 手机访问https网址怎么才能正常访问，而不是报SSL相关的错。

上面这些难点基本就是我一路循序渐进的思路。

## 手机如何连接电脑web服务器

电脑开热点给手机，然后开启一个flask服务器（端口为8000），然后需要监听所有ip地址，同时需要配置入站规则：在防火墙的高级设置里面添加两个新规则，使用端口规则类型，具体端口为80和443。

> 80是http下的默认端口，443是https下的默认端口。添加了80端口的规则是为了测试能否正常连接。不选择8000是因为到时候我们需要的是默认端口。

然后需要使用netsh命令设定端口映射：设定80和443端口的全部流量直接映射到8000端口的localhost地址，为了手机访问80/443端口的请求能被8000端口的服务器接受，具体命令如下：

```shell
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=8000 connectaddress=127.0.0.1
netsh interface portproxy add v4tov4 listenport=443 listenaddress=0.0.0.0 connectport=8000 connectaddress=127.0.0.1
```

然后下面是查看netsh配置规则和清除规则的命令：

```shell
netsh interface portproxy show all
netsh interface portproxy delete v4tov4 listenport=80 listenaddress=0.0.0.0
```

做完之后，手机访问192.168.137.1就能接受到web服务器的回应了。

## 手机如何配置host

配置host就能使dick.juwo.my的请求直接重定向到192.168.137.1，但是手机没有root的情况下，不能直接修改/etc/host文件，只能使用代理软件，我使用的是Clash。其中要注意它的配置方法，和电脑的配置方法相当不同，是分别填键和值，我填了两次才填对。

当配置好代理的host后，手机访问dick.juwo.my就能正常得到意向的网页了。

## 如何正常配置https下的访问

flask的默认启动配置下是没有https的支持的（具体是没有证书的支持），需要额外的操作。这一步有多个方法，我问ai得到了三个方法：

1. 直接netsh配置端口转发+flask配置ssl自签名证书
2. 不使用netsh的端口转发，使用Caddy进行证书配置和端口转发
3. 也不使用netsh，使用nginx进行更细致的配置

我一开始用的是Caddy，但是不知为何，不能正常工作，于是就使用了方案一。

### 生成证书

netsh的配置前面有提。这里需要先在命令行里面使用openssl生成自签名证书，命令如下：

```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout dick.juwo.my.key -out dick.juwo.my.crt -subj "/C=CN/ST=Beijing/L=Beijing/O=Org/CN=dick.juwo.my"
```

生成路径就是当前文件夹，建议在flask服务器的路径下生成。

### flask服务器的ssl证书配置

flask其实是支持解析https的流量，但是如果没有ssl证书，浏览器会报SSL相关的错，只需要简单地在启动函数里加ssl_context参数即可：

```python
context = ('dick.juwo.my.crt', 'dick.juwo.my.key')  # 之前生成的证书
app.run(host='0.0.0.0', port=8000, ssl_context=context)
```

## 成果

完成以上的配置之后，手机访问(https://dick.juwo.my)就是flask服务器的返回页面了，我的浏览器也能正常打开开发工具台了，然后数据果然在localStorage里面！最后也是正常导出了。

# 参考

---

> 全是问的Qwen
