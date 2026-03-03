---
title: "Ubuntu新建开机服务"
date: 2024-11-06T22:36:41+08:00
description: "通过服务单元建立开机服务"
tags: ["Ubuntu", "开机服务"]
featured_image: "/images/ubuntu-logo.avif"
# images is optional, but needed for showing Twitter Card
images: []
categories: "经验"
comment: false
---

**最后编辑于2024年11月06日**

# 前言

在搞VBox的共享文件夹的时候，怎么搞都不能在开机的时候自动挂载，包括安装增强功能，指定目录开启自动挂载，这些都不行。最后直接祭出杀器，新建一个开机服务。

---

# 法一：使用`/etc/rc.local`

因为这个方法是很传统的方法，在新版本的Ubuntu已经不推荐了，但是可能用得上，记录一下。

1. 打开或新建`/etc/rc.local`文件。
2. 在文件中添加挂载命令：
	
	```sh
	#!/bin/bash
	/bin/mount -t vboxsf shared /home/hx/Desktop/shared
	exit 0
	```
3. 给予`+x`权限。

---

# 法二：使用服务单元文件（推荐）

这个方法在新版Ubuntu中更为推荐，因为可操纵性更强。

1. 新建`/etc/systemd/system/vbox-mount.service`。
2. 写入以下内容：
	
	```
	[Unit]
	Description=vbox mount shared
	After=network.target
	
	[Service]
	Type=oneshot
	RemainAfterExit=yes
	ExecStart=/bin/mount -t vboxsf shared /home/hx/Desktop/shared
	User=root

	[Install]
	WantedBy=multi-user.target
	```
3. 然后重新加载配置文件：
	
	```sh
	sudo systemctl daemon-reload
	```
4. 然后启动服务单元：
	
	```sh
	sudo systemctl enable vbox-mount.service
	```
5. 可以查看一下启动状态：
	
	```sh
	sudo systemctl status vbox-mount.service
	```

## 内容解释

- `After=network.target`，就是在网路服务之后再执行，注意不是强制性的。
- `Type=oneshot`，只执行一次。
- `RemainAfterExit=yes`，执行退出之后仍然认为服务活跃，这是为了依赖这项服务的服务能正常运行（虽然这里没有）。
- `WantedBy=multi-user.target`，在多用户模式下启动？猜测是在开机选用户那里启动。

---

# 法三：使用`init.d`脚本

这个和法一一样不常用了，但我是松鼠😋！

1. 创建启动脚本`/etc/init.d/myscript`。
2. 在文件中添加挂载命令：
	
	```sh
	#!/bin/bash
	/bin/mount -t vboxsf shared /home/hx/Desktop/shared
	exit 0
	```
3. 给予`+x`权限。
4. 注册脚本到启动序列：
	```sh
	sudo update-rc.d myscript defaults
	```

---

# 法四：使用`profile`文件

就像将`arm-linux-gcc`及其库添加到环境变量一样，直接添加命令到`~/.profile`或者`~/.bashrc`即可

# 参考

---

>全是AI写的😋







