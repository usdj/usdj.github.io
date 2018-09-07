---
layout: post
title:  "Linux share 文件共享服务器"
categories: Linux
tags: Linux 文件共享服务器
author: DJY
---

* content
{:toc}
# samba

## 服务端

### 安装

#### ubuntu

```
apt-get install -y samba
```

安装 samba 的时候需要非常注意，尽可能只装 samba 这一个安装包，不要安装更多的其它包，否则可能导致数据库不一致引起的创建账号无法生效等问题。

嗯，samba 的东西很多时候就像是混沌的艺术~

### 配置

配置文件为 /etc/samba/smb.conf 
每个 [ xxx ] 表示一个共享域，下面是两个示例的共享域，将它们添加到 /etc/samba/smb.conf 文件的结尾（如果不需要其它的共享域可以删掉）即可共享 /vob 和 / 两个目录。

修改完配置之后需要重启服务才能生效。

```
[vob]   comment = vob PATH   path = /vob   public = no   writable = yes   printable = no[toptree]   comment = vob PATH   path = /   public = no   writable = yes   printable = no
```

#### 完整配置示例

```
[global]    workgroup = MYGROUP    server string = Samba Server Version %v    log file = /var/log/samba/log.%m    max log size = 50    security = user    passdb backend = tdbsam    load printers = yes    cups options = raw[homes]    comment = Home Directories    browseable = no    writable = yes;   valid users = %S;   valid users = MYDOMAIN\%S[vob]    comment = vob PATH    path = /vob    public = no    writable = yes    printable = no
```

### samba 账号

samba 共享使用 samba 自由的账号管理，也就是说要登录 samba 共享需要使用 samba 账号，但是每个 samba 账号对应一个同名的系统账号，密码可以不通。

#### 添加账号

下面命令以 root 为例

```
smbpasswd -a root
```

#### 禁用账号

下面命令以 root 为例

```
smbpasswd -d root
```

### 管理

#### 重启服务

```
service smbd restartservice nmbd restart
```

#### 停止服务

```
service smbd stopservice nmbd stop
```

#### 启动服务

```
service smbd startservice nmbd start
```

## 客户端

### 安装软件

```
apt-get install cifs-utils
```

### 挂载

```
mount //172.0.5.90/public /public或mount -t cifs //172.0.5.90/public /public直接指定账号密码/sbin/mount.cifs //192.168.0.6/nas-0 /media/nas-0 -o credentials=/root/account
```

account 文件中包含用户名和密码，内容示例：

```
cat /root/accountusername=rootpassword=9012345678
```

# nfs

## 服务端

### 安装

#### ubuntu

```
apt-get install nfs-common nfs-kernel-server
```

#### rhel

```
yum install -y rpcbind nfs-utils
```

### 配置

配置文件为 /etc/exports，格式示例如下：

```
/vob       *(rw,no_root_squash,no_all_squash,no_subtree_check)
```

说明：

```
# /etc/exports: the access control list for filesystems which may be exported#               to NFS clients.  See exports(5).## Example for NFSv2 and NFSv3:# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)## Example for NFSv4:# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

#### 检查 selinux

如果系统中安装了 selinux，selinux 的默认安全策略会禁用 nfs，要启用 nfs，需要关闭 selinux 中关于 nfs 的限制。 
判断是否安装 selinux

```
sestatus
```

如果提示没有这个命令则表示没有安装，直接跳过这一节。

如果有输出并且输出不是：SELinux status: disabled，则需要关闭 selinux 中关于 nfs 的限制，脚本如下：

```
IFS=' '; sestatus -b | grep nfs| while read ln; do arr=($ln); if [ ${arr[1]} = "off" ]; then echo $ln; echo toggle ${arr[0]} to on; setsebool -P ${arr[0]} on; fi;done
```

### 服务管理

#### 启动

##### ubuntu

```
service nfs-kernel-server status
```

##### rhel

```
service rpcbind startservice nfs start
```

##### systemd

基于 systemd 的启动管理系统（如 ubuntu 16.04）中，管理命令如下：

```
systemctl start nfs-serversystemctl restart nfs-serversystemctl stop nfs-serversystemctl status nfs-server
```

## 客户端

### 挂载

#### 手动

```
mount 172.0.5.5:/vob_pub /vob    -- 将 172.0.5.5 上共享出来的 vob_pub 目录挂在到本地的 /vob 目录下
```

#### 自动

将挂载点添加到 /etc/fstab 目录即可实现开机自动挂载，格式如下

```
192.168.2.6:/media/nas-0 /media/nas-0 nfs intr 0 0
```

### 扫描

```
showmout -e IP
```

## 问题处理

```
# showmount -e 172.0.11.55clnt_create: RPC: Port mapper failure - Authentication error# rpcinfo -p 172.0.11.55rpcinfo: can't contact portmapper: RPC: Authentication error; why = Client credential too weak
```

可能是 rpcbind 权限设置问题，检查 /etc/hosts.deny 和 /etc/hosts.allow 两个文件是否有对 rpcbind 设置的相关的限制。