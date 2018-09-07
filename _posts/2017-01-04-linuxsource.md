---
layout: post
title:  "Linux软件源"
categories: Linux
tags: Linux Software Source
author: DJY
---

* content
{:toc}
# 软件源

## 基本

### UBUNTU

#### apt

apt 和 apt-xxx（包括 apt-get、apt-cache、apt-file 等）以及 aptitude 都是 APT 软件包管理的前端软件，使用它们都可以管理 APT 软件包，但使用方式不尽相同。另外 apt 有更好地依赖处理能力，以及进度条~
```
apt install xxx
    -- 安装指定软件包
apt remove xxx
    -- 移除指定软件包
apt autoremove
    -- 自动移除不需要的依赖
apt update
    -- 更新可用软件包列表
apt edit-sources
    -- 编辑软件源
apt search xxx
    -- 根据软件包的描述信息查询软件包
apt list xxx
    -- 根据软件包的名字列出软件包
apt-cache search xxx
    -- 根据正则表达式查询软件包列表
apt-file upadte
    -- 从 apt 软件源拉取内容文件
apt-file search xxx
    -- 查询指定文件在哪个软件包内
apt-get install xxx
    -- 安装软件包 xxx
apt-get -y install xxx
    -- 当需要下载较多数据时不进行询问，直接下载安装
apt-get -f install xxx
    -- 强制安装软件包
apt-get remove xxx
    -- 移除软件包
apt-get download xxxx
    -- 下载软件包
apt-get autoremove
    -- 自动移除不需要的依赖包（慎用，可能会破坏软件依赖列表）
```
dpkg
dpkg 是用来管理 debian 系的操作系统中的本地软件包的。

```
dpkg -l | grep xxx
dpkg -i xxx.deb
dpkg -r xxx
dpkg -l
    列出所有软件包，可以搭配grep搜索
dpkg -i
    安装指定软件包，可以添加--force-depends选项忽略依赖关系强制安装，如：dpkg -i --force-depends xxxx.deb
dpkg -r
    删除指定软件包，可以添加--force-depends选项忽略依赖关系强制安装
dpkg --purge
    删除系统内部的软件，在dpkg -r --force-depends无效的情况下使用
```
多架构支持
dpkg 的多架构支持功能允许定义 “foreign architectures” 以允许在当前系统中安装其它架构的软件。添加或删除架构之后需要 apt-get update 之后才能用 apt 正确按照更改后的配置安装软件。

```
dpkg --print-architecture
    -- 打印当前架构
dpkg --print-foreign-architecture
    -- 打印 foreign 架构
dpkg --add-architecture <armhf/armel/i386....>
    -- 添加一个 foreign 架构
dpkg --remove-architecture <armhf/armel/i386....>
    -- 删除一个 foreign 架构
apt-get install package:architecture
    -- 指定让 apt 来安装指定架构的软件包
```

### RHEL

#### yum

```
yum search xxxyum install xxxyum -y install xxxyum reinstall xxxyum remove xxxyum provides xxx    -- 查询哪个软件包提供指定的内容yum clean all    -- 清空本地缓存yum makecache    -- 生成本地yum repolist all    -- 列出所有的已配置的源yum repolist -v    -- 列出源的详细信息，包括源的版本，更新时间，链接，大小，配置文件等内容yum --enablerepo=[repo]    -- 启用指定软件源yum --disablerepo=[repo]    -- 禁用指定软件源
```

#### rpm

```
rpm qa | grep xxxrpm -iv xxx.rpm
```

#### yum-config-manager

yum-config-manager 需要安装 yum-utils。

```
yum-config-manager --enable updates    -- 启用 updates 仓库yum-config-manager --disable updates    -- 禁用 updates 仓库
```

### OpenSuse

对于找不到的命令，可以使用 cnf 来查找：

```
cnf ifconfig
```

#### zypper

##### package

```
zypper search xxxzypper search -s xxx    -- 显示所有仓库中的可用版本zypper search -f tcpdump    -- 根据文件列表查找zypper install xxxzypper -y install xxxzypper remove xxx
```

##### repository

```
zypper lr    -- 列出所有的已配置的仓库zypper ref    -- 刷新所有软件仓库的本地缓存zypper modifyrepo --disable 5 6 7 8 9    -- 禁用 5,6,7,8,9 这五个软件仓库zypper -fc ar http://xxx repo_namezypper ar -fc http://mirrors.aliyun.com/opensuse/tumbleweed/repo/oss/ aliyun-oss    -- 添加软件仓库zypper mr -da    -- 禁用所有软件仓库zypper removerepo 1    -- 删除第一个软件仓库
```

------

### 

## 修改软件源

### ubuntu

#### 修改配置文件

可以直接修改默认的软件源配置文件 /etc/apt/sources.list，也可以将新的软件源以新的 xxx.list 文件的形式保存到 /etc/apt/sources.d 目录下 
**ubuntu 14.04(trusty)**

```
deb http://172.0.5.75/ubuntu/ trusty main restricted universe multiversedeb http://172.0.5.75/ubuntu/ trusty-security main restricted universe multiversedeb http://172.0.5.75/ubuntu/ trusty-updates main restricted universe multiversedeb http://172.0.5.75/ubuntu/ trusty-proposed main restricted universe multiversedeb http://172.0.5.75/ubuntu/ trusty-backports main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ trusty main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ trusty-security main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ trusty-updates main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ trusty-proposed main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ trusty-backports main restricted universe multiverse
```

**ubuntu 16.04(xenial)**

```
deb http://172.0.5.75/ubuntu/ xenial main restricted universe multiversedeb http://172.0.5.75/ubuntu/ xenial-security main restricted universe multiversedeb http://172.0.5.75/ubuntu/ xenial-updates main restricted universe multiversedeb http://172.0.5.75/ubuntu/ xenial-proposed main restricted universe multiversedeb http://172.0.5.75/ubuntu/ xenial-backports main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ xenial main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ xenial-security main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ xenial-updates main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ xenial-proposed main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ xenial-backports main restricted universe multiverse
```

**ubuntu 18.04(bionic)**

```
deb http://172.0.5.75/ubuntu/ bionic main restricted universe multiversedeb http://172.0.5.75/ubuntu/ bionic-security main restricted universe multiversedeb http://172.0.5.75/ubuntu/ bionic-updates main restricted universe multiversedeb http://172.0.5.75/ubuntu/ bionic-proposed main restricted universe multiversedeb http://172.0.5.75/ubuntu/ bionic-backports main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ bionic main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ bionic-security main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ bionic-updates main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ bionic-proposed main restricted universe multiversedeb-src http://172.0.5.75/ubuntu/ bionic-backports main restricted universe multiverse
```

**ubuntu cloud xenial pike**

```
deb http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-updates/pike maindeb http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-proposed/pike maindeb-src http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-updates/newton maindeb-src http://ubuntu-cloud.archive.canonical.com/ubuntu xenial-proposed/newton main
```

[参考](https://wiki.ubuntu.com/OpenStack/CloudArchive)

#### 更新

```
apt-get update
```

------

### rhel

#### 修改配置文件

```
[root@toy ~]# cat /etc/yum.repos.d/CentOS6-Base-163.repo [base]name=CentOS-7 - Base - 163.combaseurl=http://mirrors.163.com/centos/7/os/$basearch/gpgcheck=1gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7#released updates [updates]name=CentOS-7 - Updates - 163.combaseurl=http://mirrors.163.com/centos/7/updates/$basearch/gpgcheck=1gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7#additional packages that may be useful[extras]name=CentOS-7 - Extras - 163.combaseurl=http://mirrors.163.com/centos/7/extras/$basearch/gpgcheck=1gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7#additional packages that extend functionality of existing packages[centosplus]name=CentOS-7 - Plus - 163.combaseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/gpgcheck=1enabled=0gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7#contrib - packages by Centos Users[contrib]name=CentOS-7 - Contrib - 163.combaseurl=http://mirrors.163.com/centos/7/contrib/$basearch/gpgcheck=1enabled=0gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
```

#### 更新

```
yum clean allyum makecache
```

#### 软件源说明

| 源      | 说明                                                         | 备注                               |
| ------- | ------------------------------------------------------------ | ---------------------------------- |
| base    | base                                                         |                                    |
| updates | released updates                                             |                                    |
| extras  | additional packages that may be useful                       |                                    |
| plus    | additional packages that extend functionality of existing packages |                                    |
| contrib | packages by Centos Users                                     |                                    |
| epel    | EPEL（Extra Packages for Enterprise Linux） 是一个第三方源，由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS等提供高质量软件包的项目。 | 通过 epel-release 安装包安装       |
| scl     | scl 是方便 RedHat Software Collections 软件包使用的工具。Software Collections 是 redhat 推出的项目，旨在提供一些编程语言，数据库及相关软件包的新版本，并让同个工具的多个版本在系统共存 | 通过 centos-release-scl 安装包安装 |

------

## 使用光盘镜像作为软件源

### ubuntu

```
cat /etc/fstab/public/ubuntu-14.04.3-server-amd64.iso   /mnt/sys_iso    iso9660 defaults    0       0root@vm1:/etc/apt# cat sources.list.local deb file:///mnt/sys_iso/ubuntu trusty maindeb file:///mnt/sys_iso/ubuntu trusty restrictedapt-get update
```

命令

```
mkdir -p /mnt/sys_isoecho "/public/ubuntu-14.04.3-server-amd64.iso     /mnt/sys_iso    iso9660 defaults    0       0" >> /etc/fstabmv /etc/apt/sources.list /etc/apt/sources.list.orgecho "deb file:///mnt/sys_iso/ubuntu trusty maindeb file:///mnt/sys_iso/ubuntu trusty restricted" > /etc/sources.listapt-get update
```

------

### rhel

#### 1. 挂载

```
mount /public/rhel-server-7.1-x86_64-dvd.iso /mnt/cdrom/
```

#### 2. 添加一个 repo 配置文件（以 repo 作为后缀才能被 yum 识别）

```
cat /etc/yum.repos.d/redhat_local.repo[RHEL]name=RHEL7.1baseurl=file:///mnt/cdromgpgcheck=0gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-redhat-releaseenabled=1
```

### centos

```
[root@centos yum.repos.d]# cat /etc/yum.repos.d/local.repo [CentOS]name=CentOS-5.4baseurl=file:///mnt/cdromgpgcheck=0gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-CentOS-5enabled=1
```

#### 3. 检查源

查看当前所有可用的源，刚刚添加的是否在里边

```
yum repolist all
```

------

### openSUSE

挂载光盘： 
/etc/fstab

```
/media/hd2/mirrors/iso-mnt/iso/openSUSE-Leap-15.0-DVD-x86_64.iso    /media/hd2/mirrors/iso-mnt/mntp/opensuse/leap/15    iso9660 defaults    0       0
```

配置： 
通过 http 服务器提供服务：

```
zypper ar -fc http://172.0.5.75/iso-mnt/opensuse/leap/15/ private
```

------

## 离线安装软件

### ubuntu

备份当前缓存的 deb 包

```
cd /var/cache/apt/archivesmkdir ../archives.oldmv ./* ../archives.old
```

下载软件包

```
apt-get -d install kvm qemu-kvm libvirt-bin virtinst bridge-utils virt-viewercp -r /var/cache/apt/archives /offlinePackage
```

生成 package.gz

```
dpkg-scanpackages /offlinePackage/ 2>/dev/null | gzip > /offlinePackage/Packages.gzmkdir /offlinePackage/archivesmv /offlinePackage/Packages.gz /offlinePackage/archives/Packages.gz
```

打包

```
tar -Jcvf offlinePackage.tar.xz /offlinePackage/
```

在目的系统解压

```
cd /tar -Jxvf offlinePackage.tar.xz
```

修改软件源

```
vi /etc/apt/sources.listdeb file:///offlinePackage archives/
```

更新测试

```
apt-get updateapt-get install kvm qemu-kvm libvirt-bin virtinst bridge-utils virt-viewer
```

------

## 搭建本地软件源

### censo-5.4

使用 centos-5.4 的安装 iso 镜像作为软件源搭建一个本地软件源服务器

#### 准备

1. 安装并启动 apache2
2. mkdir -p /var/www/html/centos/5/iso
3. 将 CentOS-5.4-x86_64-bin-DVD.iso 复制到 /var/www/html/centos/5

#### 使用 iso 镜像作为软件源

```
echo "/var/www/html/centos/5/CentOS-5.4-x86_64-bin-DVD.iso  /var/www/html/centos/5/iso  iso9660 defaults    0       0" >> /etc/fstabmount -a
```

#### 测试

使用浏览器打开软件源服务器所在的 IP，默认使用 80 端口，如在浏览器地址栏输入：

```
172.0.5.75/centos/5/iso
```

看到有文件对应的 iso 中的文件列表即可

#### 使用

在 centos-5.4 的系统中，设置对应的软件源，并更新本地缓存即可使用。

设置软件源

```
echo "[CentOS-Private]name=CentOS-5.4-Privatebaseurl=http://172.0.5.75/centos/5/isogpgcheck=0gpgkey=http://172.0.5.75/centos/5/iso/CentOS/RPM-GPG-KEY-CentOS-5enabled=1" > /etc/yum.repos.d/private.repo
```

清空本地缓存并更新，然后列出可用的仓库，如果有列出 CentOS-Private 则说明可用

```
yum clean allyum makecacheyum repolist
```

#### 参考

- [CentOS搭建本地yum源（http方式）](https://my.oschina.net/u/1461927/blog/372147)

### centos-6.5

搭建方式和 centos-5.4 的方法相同。

一个可用的 repo 配置文件如下：

```
echo "[CentOS-Private]name=CentOS-6.5-Privatebaseurl=http://172.0.5.75/centos/6/isogpgcheck=0gpgkey=http://172.0.5.75/centos/6/iso/RPM-GPG-KEY-CentOS-6enabled=1" > /etc/yum.repos.d/private.repo
```

### rhel-7.1

搭建方式和 centos-5.4 的方法相同。

一个可用的 repo 配置文件如下：

```
root@CASA-MOBILE:~/mme_builder# cat private-rhel.repo [RHEL-Private]name=RHEL-7.1-Privatebaseurl=http://172.0.5.75/rhel/7/isogpgcheck=0gpgkey=http://172.0.5.75/rhel/7/iso/RPM-GPG-KEY-redhat-releaseenabled=1
```

### ubuntu iso

使用 ubuntu iso 在一台服务器上搭建本地软件源，以让其它服务器使用

#### 挂载 iso

iso 文件放在 /media/nas-0/softwares/sys_ios/linux/ubuntu/ubuntu-16.04.3-server-amd64.iso，挂在到 /var/spool/apt-mirror/mirror/iso/ubuntu/16.04

```
/media/nas-0/softwares/sys_ios/linux/ubuntu/ubuntu-16.04.3-server-amd64.iso /var/spool/apt-mirror/mirror/iso/ubuntu/16.04   iso9660 defaults    0   0
```

#### 配置 http 服务

假设已经安装了 apache2 server，使用默认路径 /var/www/html 作为 http 服务的根目录，则只需要设置一个软连接，将 /var/spool/apt-mirror/mirror/iso/ubuntu/16.04 链接到 http server 目录下即可：

```
ln -s /var/spool/apt-mirror/mirror /var/www/html/mirrors
```

#### 使用

/etc/apt/sources.list 文件配置如下：

```
deb http://192.168.2.6/mirrors/iso/ubuntu/16.04/ubuntu/ubuntu/ xenial main restricted
```

配置好之后，使用 “apt-get update” 更新下缓存，接下来就可以正确使用了。

### ubuntu

#### 说明

以运行在 amd64 架构上的 ubuntu 14.04 为例，在 192.168.2.6 的 PC 上面架设本地源，当前的软件源按照下列配置大致需要 250G 的磁盘空间。

apt-mirror 
架设本地仓库镜像源的主要工具，可以实现镜像源的获取，同步，管理等功能

apache2 
apt 软件源本质是 http 服务，使用 apache2 架设本地 http 服务器。

#### 软件安装

```
apt-get install -y apt-morror apache2
```

#### 配置

apt-mirror 的软件源配置语法和 sources.list 的完全一样，配置文件为 /etc/apt/mirror.list。需要注意的是，如果当前设备的架构和软件源的架构不一致，需要指定软件源的架构，如在 armhf 架构的设备上假设的 apt-mirror，如果要在这上面同步一个 amd64 的源，则 amd64 源的文件格式应该定义为 deb-amd64，此时配置里的 deb 默认为 deb-armhf。

修改 /etc/apt/mirror.list 文件为：

```
root@ltepc7# cat /etc/apt/mirror.listset nthreads     20set _tilde 0# Here is configuration for ubuntu 14.04, It may cost about 210GB disk space.deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiversedeb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiversedeb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiversedeb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiversedeb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiversedeb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiversedeb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiversedeb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiversedeb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiversedeb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse# Here is configuration for ubuntu 16.04, It may cost about 150GB disk space.#deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse#deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse#deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse#deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse#deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse#deb-i386 http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse#deb-i386 http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse#deb-i386 http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse#deb-i386 http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse#deb-i386 http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverseclean http://mirrors.aliyun.com/ubuntu
```

说明 
deb http://* 不带架构说明的 deb 默认为当前运行 apt-mirror 设备的架构，这里是 amd64 
deb-i386 http://* 对应的 32 位软件 
deb-src http://* 软件的源代码

apt 软件源格式：

```
ArchiveType RepositoryURL Distribution Component...
```

- ArchiveType 有 deb，deb-src

- RepositoryURL 为软件源网址

- Distribution 为发布版本

   

  ​

  - Debian 和 Ubuntu 的 Distribution ：

| 系统   | Distribution     | 说明                 |
| ------ | ---------------- | -------------------- |
| Debian | oldstable        | 旧版本的稳定安装     |
| Debian | stable           | 稳定的安装包         |
| Debian | unstable         | 可能是不稳定的安装包 |
| Debian | testing          | 正在测试中的安装包   |
| Ubuntu | xenial           |                      |
| Ubuntu | xenial-updates   |                      |
| Ubuntu | xenial-security  |                      |
| Ubuntu | xenial-backports |                      |

- Component 为组件

   

  ​

  - Debian 和 Ubuntu component 如下：

| 系统   | component  | 说明                                                         |
| ------ | ---------- | ------------------------------------------------------------ |
| Debian | main       | 包含符合DFSG（Debian Free Software Guidelines）的自由软件包，而且这些软件包不依赖不符合该指导原则的软件包。 这些软件包被视为Debian发型版的一部分。 |
| Debian | contrib    | 包含符合DFSG（Debian Free Software Guidelines）的自由软件包，不过这些软件包依赖不在main分类中的软件包。 |
| Debian | non-free   | 包含不符合DFSG（Debian Free Software Guidelines）指导原则的非自由软件包 |
| Ubuntu | main       | 官方支持的自由软件                                           |
| Ubuntu | restricted | 官方支持的非完全自由的软件                                   |
| Ubuntu | universe   | 社区维护的自由软件                                           |
| Ubuntu | multiverse | 非自由软件                                                   |

##### armhf 架构下的配置

下面的配置是运行在 armhf 架构下的 apt-mirror 的一个配置，同步的软件源包括：armhf（默认），armel，arm64，i386，amd64

```
############# config #################### set base_path    /var/spool/apt-mirror## set mirror_path  $base_path/mirror# set skel_path    $base_path/skel# set var_path     $base_path/var# set cleanscript $var_path/clean.sh# set defaultarch  <running host architecture># set postmirror_script $var_path/postmirror.sh# set run_postmirror 0set nthreads     20set _tilde 0############## end config ###############deb http://ftp.us.debian.org/debian unstable main contrib non-free#deb-src http://ftp.us.debian.org/debian unstable main contrib non-free# mirror additional architectures#deb-alpha http://ftp.us.debian.org/debian unstable main contrib non-free#deb-amd64 http://ftp.us.debian.org/debian unstable main contrib non-free#deb-armel http://ftp.us.debian.org/debian unstable main contrib non-free#deb-hppa http://ftp.us.debian.org/debian unstable main contrib non-free#deb-i386 http://ftp.us.debian.org/debian unstable main contrib non-free#deb-ia64 http://ftp.us.debian.org/debian unstable main contrib non-free#deb-m68k http://ftp.us.debian.org/debian unstable main contrib non-free#deb-mips http://ftp.us.debian.org/debian unstable main contrib non-free#deb-mipsel http://ftp.us.debian.org/debian unstable main contrib non-free#deb-powerpc http://ftp.us.debian.org/debian unstable main contrib non-free#deb-s390 http://ftp.us.debian.org/debian unstable main contrib non-free#deb-sparc http://ftp.us.debian.org/debian unstable main contrib non-free# add for x86deb-amd64 http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiversedeb-amd64 http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiversedeb-amd64 http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiversedeb-amd64 http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiversedeb-amd64 http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiversedeb-i386 http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse# add for orangepideb http://ftp.hr.debian.org/debian/ jessie main contrib non-freedeb http://ftp.hr.debian.org/debian/ jessie-updates main contrib non-freedeb http://ftp.hr.debian.org/debian/ jessie-backports main contrib non-free#deb http://security.debian.org/ jessie/updates main contrib non-free#deb http://oph.mdrjr.net/meveric/ jessie main# add for arm64# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to# newer versions of the distribution.deb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted## Major bug fix updates produced after the final release of the## distribution.deb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted## Uncomment the following two lines to add software from the 'universe'## repository.## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu## team. Also, please note that software in universe WILL NOT receive any## review or updates from the Ubuntu security team.deb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial universedeb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe## N.B. software from this repository may not have been tested as## extensively as that contained in the main release, although it includes## newer versions of some applications which may provide useful features.## Also, please note that software in backports WILL NOT receive any review## or updates from the Ubuntu security team.deb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricteddeb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricteddeb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial-security universedeb-arm64 http://ports.ubuntu.com/ubuntu-ports/ xenial-security multiversedeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial main restricted universedeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricted universedeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricted universedeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted universe multiverseclean http://ftp.us.debian.org/debian
```

#### 开启软件源自动同步

取消掉 /etc/cron.d/apt-mirror 中以下一行的注释：

```
0 4    * * *    apt-mirror    /usr/bin/apt-mirror > /var/spool/apt-mirror/var/cron.log0 8 * * * pkill apt-mirror1 8 * * * pkill wget
```

每天早上四点开始同步 
每天假如在早上八点前没有同步完成，则直接杀掉同步进程 
先杀掉 apt-mirror 进程，再杀掉 wget 进程，apt-mirror 实际上是用 wget 去下载文件的

由于权限等原因，如果需要以 apt-mirror 账号进行同步操作比较复杂的话，可以考虑使用 root 账号进行同步。通过 crontab -e 编辑 root 的 cron 任务，添加下列 6 行，表示周一到周五的凌晨 4 点到第二天下午 6 点进行同步，周末的话，则只在凌晨 4 点到第二天早上 9 点进行同步：

```
0 4   * * 1-5   /usr/bin/apt-mirror > /var/spool/apt-mirror/var/cron.log0 18  * * 1-5   pkill apt-mirror0 18  * * 1-5   pkill wget0 4  * * 0,6   /usr/bin/apt-mirror > /var/spool/apt-mirror/var/cron.log0 9  * * 0,6   pkill apt-mirror0 9  * * 0,6   pkill wget
```

#### 查看同步日志

/var/spool/apt-mirror/var

#### 手动同步软件仓库到本地

执行 apt-mirror 同步配置的软件仓库到本地 
同步的地址默认在 /var/spool/apt-mirror/mirror

#### 启动 http 服务器

service apache2 start

#### 设置本地软件源路径到 http 服务器下

```
ln -s /var/spool/apt-mirror/mirror/mirrors.aliyun.com/ubuntu/ /var/www/html/ubuntunot sure if neccessaryln -s /var/spool/apt-mirror/skel/mirrors.aliyun.com/ubuntu/dists/ /var/www/html/ubuntu/dists
```

#### 测试

在浏览器中输入：192.168.2.6/ubuntu 测试，看打开的页面是否和 mirrors.aliyun.com/ubuntu 这个地址打开的一样，一样的话就可以了

#### 使用本地镜像源

##### 配置

将 apt 的软件仓库地址设置为本地： 
amd64(ubuntu/trusty)

```
root@ltepc7# cat /etc/apt/sources.listdeb http://192.168.2.6/ubuntu/ trusty main restricted universe multiversedeb http://192.168.2.6/ubuntu/ trusty-security main restricted universe multiversedeb http://192.168.2.6/ubuntu/ trusty-updates main restricted universe multiversedeb http://192.168.2.6/ubuntu/ trusty-proposed main restricted universe multiversedeb http://192.168.2.6/ubuntu/ trusty-backports main restricted universe multiversedeb-src http://192.168.2.6/ubuntu/ trusty main restricted universe multiversedeb-src http://192.168.2.6/ubuntu/ trusty-security main restricted universe multiversedeb-src http://192.168.2.6/ubuntu/ trusty-updates main restricted universe multiversedeb-src http://192.168.2.6/ubuntu/ trusty-proposed main restricted universe multiversedeb-src http://192.168.2.6/ubuntu/ trusty-backports main restricted universe multiverse
```

armhf(debian/jessie)

```
deb http://192.168.2.6/debian/ jessie main contrib non-free#deb-src http://192.168.2.6/debian/ jessie main contrib non-freedeb http://192.168.2.6/debian/ jessie-updates main contrib non-free#deb-src http://192.168.2.6/debian/ jessie-updates main contrib non-freedeb http://192.168.2.6/debian/ jessie-backports main contrib non-free#deb-src http://192.168.2.6/debian/ jessie-backports main contrib non-free#deb http://security.debian.org/ jessie/updates main contrib non-free#deb http://oph.mdrjr.net/meveric/ jessie main
```

arm64(ubuntu/xenial)

```
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to# newer versions of the distribution.deb http://192.168.2.6/ubuntu-ports/ xenial main restricted#deb-src http://192.168.2.6/ubuntu-ports/ xenial main restricted## Major bug fix updates produced after the final release of the## distribution.deb http://192.168.2.6/ubuntu-ports/ xenial-updates main restricted#deb-src http://192.168.2.6/ubuntu-ports/ xenial-updates main restricted## Uncomment the following two lines to add software from the 'universe'## repository.## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu## team. Also, please note that software in universe WILL NOT receive any## review or updates from the Ubuntu security team.deb http://192.168.2.6/ubuntu-ports/ xenial universe#deb-src http://192.168.2.6/ubuntu-ports/ xenial universedeb http://192.168.2.6/ubuntu-ports/ xenial-updates universe#deb-src http://192.168.2.6/ubuntu-ports/ xenial-updates universe## N.B. software from this repository may not have been tested as## extensively as that contained in the main release, although it includes## newer versions of some applications which may provide useful features.## Also, please note that software in backports WILL NOT receive any review## or updates from the Ubuntu security team.deb http://192.168.2.6/ubuntu-ports/ xenial-backports main restricted#deb-src http://192.168.2.6/ubuntu-ports/ xenial-backports main restricteddeb http://192.168.2.6/ubuntu-ports/ xenial-security main restricted#deb-src http://192.168.2.6/ubuntu-ports/ xenial-security main restricteddeb http://192.168.2.6/ubuntu-ports/ xenial-security universe#deb-src http://192.168.2.6/ubuntu-ports/ xenial-security universedeb http://192.168.2.6/ubuntu-ports/ xenial-security multiverse#deb-src http://192.168.2.6/ubuntu-ports/ xenial-security multiverse
```

##### 测试

```
apt-get updateapt-get install sl
```

#### 清除本地软件源下不需要的文件

    每次执行 apt-mirror 进行同步之后，在结束的时候都有提示可以运行脚本进行清理不需要的文件。如果不再需要某个版本的源，修改 /etc/apt/mirror.list 下的配置，然后执行 apt-mirror 进行一次同步，再运行同步最后提示的脚本即可删掉那个版本的源遗留的文件。

默认安装的清楚脚本路径如下：

```
/var/spool/apt-mirror/var/clean.sh
```

#### 问题处理

##### Hash Sum mismatch

问题： 
apt-get update 提示 W: Failed to fetch <http://172.0.5.75/ubuntu/dists/trusty-updates/main/binary-i386/Packages>Hash Sum mismatch

解决 
这个问题出现的原因可能是客户端的问题也可能是软件源的问题，如果多个客户端同时出现这个问题则可能是软件源的原因。

如果是客户端的问题，尝试删除 /var/lib/apt/lists/ 目录下的所有文件，然后重新执行 apt-get update

假如确定是软件源的问题，可以删除软件源中 ubuntu/dists/trusty-updates/main/binary-i386 目录下的所有文件，然后执行 apt-mirror 更新重新同步数据；如果问题仍然没有解决，可以往上层继续删，比如说删掉 ubuntu/dists/trusty-updates 整个目录。

##### Packages 404 Not Found

apt-get update 时的错误输出：

```
E: Failed to fetch http://192.168.2.6/ubuntu-ports/dists/xenial/universe/binary-armhf/Packages  404  Not FoundE: Failed to fetch http://192.168.2.6/ubuntu-ports/dists/xenial-updates/main/binary-armhf/Packages  404  Not FoundE: Failed to fetch http://192.168.2.6/ubuntu-ports/dists/xenial-backports/main/binary-armhf/Packages  404  Not FoundE: Failed to fetch http://192.168.2.6/ubuntu-ports/dists/xenial-security/main/binary-armhf/Packages  404  Not Found
```

软件源中缺少配置中的 Packages 文件，在 apt-mirror 的配置中添加：

```
deb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial main restricteddeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial-updates main restricteddeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial-backports main restricteddeb-armhf http://ports.ubuntu.com/ubuntu-ports/ xenial-security main restricted
```

## 

## 软件源说明

### ubuntu

| repository | description                                              |
| ---------- | -------------------------------------------------------- |
| main       | Canonical 支持的自由开源软件                             |
| universe   | 社区维护的开源软件                                       |
| restricted | 设备的专有驱动                                           |
| multiverse | 有版权与合法性问题限制的软件                             |
| proposed   | 更新的测试源，建议有兴趣帮助测试更新和提供反馈的人员使用 |

参考：<https://help.ubuntu.com/community/Repositories/Ubuntu>

------

### opensuse

[openSUSE 软件源的相关说明]( https://cn.opensuse.org/%E8%BD%AF%E4%BB%B6%E6%BA%90)

#### [主软件源](https://zh.opensuse.org/index.php?title=%E4%B8%BB%E8%BD%AF%E4%BB%B6%E6%BA%90&variant=zh)

#### [第三方软件源](https://zh.opensuse.org/index.php?title=%E7%AC%AC%E4%B8%89%E6%96%B9%E8%BD%AF%E4%BB%B6%E6%BA%90&variant=zh#Google_Software_Repositories_for_Linux)

#### packman

Packman 是 openSUSE 最大的第三方软件源，其中包含各种各样的软件包，这些软件包或因版权或因其他原因而无法包含于 openSUSE 之中。通常可以在此软件源里找到 mp3 等多媒体解码包、众多的多媒体播放器（如 mplayer等），还有各种下载工具和游戏。packman源包含四个部分

- Essentials: 提供满足必要需求的多媒体解码包、视频及音频播放器。
- Multimedia: 提供其他与多媒体相关的程序或者库文件。
- Extra: 与多媒体无关的一些程序，大多与网络有关（如aircrack-ng等）。
- Games: 游戏

请注意！ 上述四个repo中，后面三个是在第一个 Essentials 的基础上打包的，如果您需要使用后面三个中某一个源的时候，为了保证功能不缺失，请同时将 Essentials 也添加进去。

地址： 
<http://packman.links2linux.org/mirrors>

#### 国内软件源

- 清华： <https://mirrors.ustc.edu.cn/opensuse/tumbleweed/repo/>
- 阿里云：<http://mirrors.aliyun.com/opensuse/tumbleweed/repo/>
- 中科大：<http://mirrors.aliyun.com/opensuse/tumbleweed/repo/>
- 搜狐：<http://mirrors.sohu.com/opensuse/tumbleweed/repo/>
- 首都在线：<http://mirrors.yun-idc.com/opensuse/tumbleweed/repo/>
- 北理：<http://mirror.bit.edu.cn/opensuse/tumbleweed/repo/>
- 浙大：<http://mirrors.zju.edu.cn/opensuse/tumbleweed/repo/>
- 厦大：<http://mirrors.xmu.edu.cn/opensuse/tumbleweed/repo/>（好像只有oss）
- 重庆大学：<https://c.mirrors.lanunion.org/opensuse/tumbleweed/repo/>

添加源：

```
zypper ar -fc http://mirrors.aliyun.com/opensuse/tumbleweed/repo/oss/ aliyun-osszypper ar -fc http://mirrors.aliyun.com/opensuse/tumbleweed/repo/non-oss/ aliyun-non-osszypper ar -fc http://mirrors.aliyun.com/opensuse/tumbleweed/repo/src-non-oss aliyun-src-non-osszypper ar -fc http://mirrors.aliyun.com/opensuse/tumbleweed/repo/src-oss aliyun-src-osszypper ar -fc http://mirrors.aliyun.com/opensuse/tumbleweed/repo/debug aliyun-debugzypper ar -fc http://mirrors.hust.edu.cn/packman/suse/openSUSE_Tumbleweed/ packmanzypper ar -fc http://download.videolan.org/pub/vlc/SuSE/Tumbleweed/ vlczypper ar -fc http://download.opensuse.org/repositories/home:/opensuse_zh/openSUSE_Tumbleweed/ opensuse_zhzypper ar -fc http://download.opensuse.org/repositories/GNOME:/Apps/openSUSE_Factory/ gnome_GFzypper ar -fc http://download.opensuse.org/repositories/GNOME:/Apps/openSUSE_Factory/ gnome_appszypper ar -fc http://download.opensuse.org/repositories/KDE:/Extra/openSUSE_Tumbleweed/ kde_extrazypper ar -fc http://download.opensuse.org/repositories/KDE:/Applications/openSUSE_Factory_standard/ kde_application
```

------

## 问题处理

### apt

------

#### libc6相关

E: Internal Error, No file name for 
解决： 
sudo rm -f /etc/apt/sources.list.d/* 
sudo dpkg –configure -a 
（sudo dpkg –configure -a 可能执行到一部分失败，可以回去重新apt-get安装尝试一下）

------

#### this may be caused by held packages.

apt-get -f install提示： 
E: Error, pkgProblemResolver::Resolve generated breaks, this may be caused by held packages. 
解决： 
查看/var/log/apt/term.log文件，最末位的一次Log started: $date开始，查看所有操作过的包，一般是replace xxxx （using xxxx）， 
此时把replace的所有可能影响的软件包都用dpkg一个个删除掉，之后再apt-get install 尝试安装，应该会提示要apt-get -f install 
这次执行apt-get -f install应该可以成功，如果提示libc6的问题，参照上一个问题解决

------

#### group ‘uml-net’ in statoverride file

apt get install xxx 的时候提示： 
syntax error: unknown group ‘uml-net’ in statoverride file 
E: Sub-process /usr/bin/dpkg returned an error code (2) 
解决： 
添加一个 uml-net 帐号： 
useradd uml-net

------

#### E: Cannot get debconf version. Is debconf installed?

```
...Use of uninitialized value $value in substitution (s///) at /usr/share/perl5/Debconf/Format/822.pm line 65, <__ANONIO__> line 93....E: Cannot get debconf version. Is debconf installed?debconf: apt-extracttemplates failed: No such file or directoryExtracting templates from packages: 68%E: Cannot get debconf version. Is debconf installed?debconf: apt-extracttemplates failed: No such file or directoryExtracting templates from packages: 100%dpkg: error: parsing file '/var/lib/dpkg/available' near line 0: field name ELF' must be followed by colonE: Sub-process /usr/bin/dpkg returned an error code (2)
```

处理 
首先确认是否有依赖冲突或 PPA 相关的冲突，如果没有，执行下列命令尝试修复

```
apt-get updateapt-get cleanapt-get install -fydpkg -i /var/cache/apt/archives/*.debdpkg --configure -aapt-get install -fy
```

在执行上述步骤时如果出现下面问题

```
dpkg --configure -adpkg: error: too-long line or missing newline in `/var/lib/dpkg/triggers/File'
```

查看相关文件发现全是乱码

```
cat /var/lib/dpkg/triggers/File 
```

```
rm -v  /var/lib/dpkg/triggers/File 
```

另外也看一下 available 文件是否损坏

```
cat /var/lib/dpkg/available
```

如果 available 文件已损坏，则：

```
dpkg --clear-availapt-get update
```

这个问题主要还是依赖问题，apt-get -fy install 的时候会尝试安装所缺的依赖，但可能会因为依赖循坏导致安装出错，这个时候需要用 dpkg 去安装 /var/cache/apt/archives 下的 deb 包。 

------

## 参考

- [opensuse 主软件源](https://zh.opensuse.org/index.php?title=%E4%B8%BB%E8%BD%AF%E4%BB%B6%E6%BA%90&variant=zh)
- [openuse 第三方软件源](https://zh.opensuse.org/index.php?title=%E7%AC%AC%E4%B8%89%E6%96%B9%E8%BD%AF%E4%BB%B6%E6%BA%90&variant=zh#Google_Software_Repositories_for_Linuxhttps://zh.opensuse.org/index.php?title=%E7%AC%AC%E4%B8%89%E6%96%B9%E8%BD%AF%E4%BB%B6%E6%BA%90&variant=zh#Google_Software_Repositories_for_Linux)
- [OpenSUSE Leap在线更新到Tumbleweed小记](http://jiangzhuti.me/blog/OpenSUSE-Leap%E5%9C%A8%E7%BA%BF%E6%9B%B4%E6%96%B0%E5%88%B0Tumbleweed%E5%B0%8F%E8%AE%B0)