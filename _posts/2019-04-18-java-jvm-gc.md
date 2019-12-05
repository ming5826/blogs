---
layout:     post
title:      "使用FRP工具远程控制Windows"
subtitle:   "使用内网穿透工具——FRP实习远程控制处于内网环境的Windows计算机。"
date:       2019-12-05
author:     "mingyue"
header-img: "img/in-post/2019.12/05/post-frp-windows.png"
tags:
    - FRP
---


# 一款内网穿透工具--FRP
# 前言

想对Windows的机器进行远程控制，有这几种方式

1. 具有公网IP的直接使用公网IP
2. 使用Teamview工具进行远程控制
3. 使用向日葵工具进行远程控制（不充钱操作有点卡）

这里推荐使用内网穿透工具——FRP，以及一台云服务器当做服务端，进行简单配置后即可远程访问

> FRP软件下载地址：https://github.com/fatedier/frp/releases

# 服务端配置

配置 FRP 服务端的前提条件是需要一台具有**公网 IP **的设备，得益于 **FRP** 是 Go 语言开发的，具有良好的跨平台特性。你可以在 Windows、Linux、MacOS、ARM等几乎任何可联网设备上部署。

- 一台云服务器，比如 CentOS7
- 一个公网IP，带宽视情况而定，公网IP地址以 `117.73.3.210`为例

## 服务端下载FRP

服务端下载Linux版的FRP工具，比如`frp_0.30.0_linux_amd64.tar.gz`，为了方便管理，这里进行了重命名为frp

```
wget https://github.com/fatedier/frp/releases/download/v0.30.0/frp_0.30.0_linux_amd64.tar.gz
tar -zxvf frp_0.30.0_linux_amd64.tar.gz
mv frp_0.30.0_linux_amd64 frp
```

进入frp目录： `cd frp`,如图

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/frp-linux-install.png)

## 配置文件

文件介绍：

|文件名|功能|
|-----|----|
|frpc|客户端应用程序|
|frps|服务端应用程序|
|frpc.ini|客户端配置文件-精简版|
|frps.ini|服务端配置文件-精简版|
|frpc_full.ini|客户端配置文件-完整版|
|frps_full.ini|服务配置文件-完整版|

这里进行配置时使用精简版的配置文件 `frps.ini`

查看编辑配置文件，`vim frps.ini` ,如图，这里的port可以自己指定，比如我使用7000（默认） 

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/frpc-ini.png)

## 启动服务端FRP

进入目录，指定配置文件`frps.ini`启动服务端程序：

```
./frps -c ./frps.ini
```

也可以使用后台不挂断的方式启动，并且指定日志文件

```
nohup ./frps -c ./frps.ini &> /var/log/frps.log &
```

# 客户端配置

客户端即要被远程访问的机器，比如处于公司内网的办公电脑，为Windows操作系统

## 开启远程控制功能

> 首先要开启远程控制功能，进入控制面板 `控制面板\所有控制面板项\系统`(或者直接右键`此电脑`，点击`属性`)
> 点击`高级系统设置`，点击`远程`，选择`允许远程访问`，点击确定

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/win-open3389.png)

## 客户端下载FRP

客户端下载Windows版本的FRP工具，比如`frp_0.30.0_windows_amd64.zip`

[下载地址](https://github.com/fatedier/frp/releases/download/v0.30.0/frp_0.30.0_windows_amd64.zip)

下载后解压，可以重命名一下，进入文件目录，如图

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/frp-windows-install.png)
## 配置文件

文件介绍如服务端：

|文件名|功能|
|-----|----|
|frpc.exe|客户端应用程序|
|frps.exe|服务端应用程序|

这里的配置文件使用精简版的

编辑并保存 `frpc.ini` ，如下

```
[common]
server_addr = 117.73.3.210  #服务端公网IP地址
server_port = 7000  #服务端开启的端口

[3389]
type = tcp
local_ip = 192.168.1.9   #客户端的ip地址，可以通过打开cmd执行`ipconfig`查看
local_port = 3389   #Windows远程控制端口，无需修改
remote_port = 33211  #远程端口，这里自己设置一个不常用的端口
```
## 启动客户端

进入目录，打开CMD命令行，通过指定配置文件`frpc.ini`启动客户端，如下图：

```
.\frpc.exe -c .\frpc.ini
```

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/frp-win-start.png)

# 访问

> 此时，远程控制已经配置完成，接下来就可以访问了
> 访问地址为服务端的公网IP地址加上客户端配置文件指定的remote_port，例如 `117.73.3.210:33211`

## Windows操作系统远程控制

使用Windows操作系统进行远程控制，例如家里的电脑

打开Windows的远程桌面连接工具，按快捷键 `Win + R`，输入 `mstsc`

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/win_r-mstsc.png)

输入远程主机地址和用户名，例如 `117.73.3.210:33211`和我办公电脑的用户名

![](https://mingyue-1300243549.cos.ap-chengdu.myqcloud.com/picgo/20191205/win-new3389.png)

点击连接

## 手机远程控制

下载微软的一个远程控制软件 `RD Client` 手机版
[下载地址](https://apks-1300243549.cos.ap-chengdu.myqcloud.com/CD%20Client.apk)

1. 点击软件右上角 `+` 号
2. 点击 `Desktop`
3. PC name输入公网IP地址和remote_port，例如，117.73.3.210:33211
4. User name输入被控制的电脑的登录用户名
5. 点击右上角save
6. 点击主页的远程桌面即可进行远程控制

# FRP介绍

FRP 全名：Fast Reverse Proxy。FRP 是一个使用 Go 语言开发的高性能的反向代理应用，可以帮助您轻松地进行内网穿透，对外网提供服务。FRP 支持 TCP、UDP、HTTP、HTTPS等协议类型，并且支持 Web 服务根据域名进行路由转发。

> FRP 项目地址：https://github.com/fatedier/frp

## FRP 的作用

利用处于内网或防火墙后的机器，对外网环境提供 HTTP 或 HTTPS 服务。

对于 HTTP, HTTPS 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个 80 端口。

利用处于内网或防火墙后的机器，对外网环境提供 TCP 和 UDP 服务，例如在家里通过 SSH 访问处于公司内网环境内的主机。

[参考文档](https://www.jianshu.com/p/00c79df1aaf0)