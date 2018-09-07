---
layout: post
title:  "Linux danted代理服务器"
categories: Linux
tags: Linux danted 代理服务器
author: DJY
---

* content
{:toc}
# 说明

danted-server 作为 linux 下的 socks 代理服务器，支持 socks4 和 socks5，并且支持多 IP 输入输出。

# 安装

```
apt-get install -y dante-server
```

# 配置

备份 /etc/danted.conf，然后保存配置到文件 /etc/danted.conf

```
logoutput: /var/log/danted.loginternal: 0.0.0.0 port = 7788external: 192.168.2.21method: username noneuser.privileged: proxyuser.notprivileged: nobodyuser.libwrap: nobodyclient pass {from: 0.0.0.0/0 to: 0.0.0.0/0    log: connect disconnect}pass {from: 0.0.0.0/0 to: 0.0.0.0/0 port gt 1023    command: bind    log: connect disconnect}pass {from: 0.0.0.0/0 to: 0.0.0.0/0    command: connect udpassociate    log: connect disconnect}block {from: 0.0.0.0/0 to: 0.0.0.0/0    log: connect error}
```

# 启动

```
service danted start
```

# 使用

客户端配置 socks5 代理，比如 firefox 浏览器，可以直接打开 设置->选项->网络->设置 界面按照如下示例配置：

![title](https://woniuxiang.space/api/file/getImage?fileId=59a5977541a3d5000e6918fe)