---
layout: post
title:  "Linux 磁盘管理"
categories: Linux
tags: Linux 磁盘管理
author: DJY
---

* content
{:toc}
## 文件权限

### 基本权限

#### 查看文件权限信息

```
root@vm1:/fdsk# ls -ltotal 16drwx------ 2 root root 16384 Aug  8 06:27 lost+found
```

     左边一列表示权限信息，type(1)>root(3)>owner(3)>group(3)，第一个 d 表示是目录，第一个 rwx 表示 root 账号有可读可写可执行权限，后面留个 “-” 表示这个目录的所有者和与所有者为同一组的其它账号都没有可读可写可执行权限。

     rwx 分别对应可读可写可执行，读取权限对应 4，写权限对应 2，执行权限对应 1

```
3=1+2 -- 可写可执行不可读5=4+1 -- 可读可执行不可写6=4+2 -- 可写可读不可执行7=4+2+1 -- 可读可写可执行
```

#### 修改文件权限

```
chmod u+x file    -- 给当前用户加上对 file 文件的可执行权限chmod 755 file    -- 将文件 file 权限设置为 755，也就是 root 账号可读可写可执行，其它账号可读可执行chmod -R 777 dir    -- 将目录 dir 及该目录下所有文件的权限设置为 777
```

### attr 权限

#### lsattr 查看文件属性

```
root@vm1:/fdsk# lsattr -------------e-- ./ha_log.bak-------------e-- ./ha_log-------------e-- ./logfile-------------e-- ./docsislogfile-------------e-- ./casa-nfv-4.2.0-122.tar.gz-------------e-- ./startup-config-------------e-- ./snmpd.cnf-------------e-- ./casa-nfv-4.2.0-122----i--------e-- ./mirror-2016-07-25.13-58-03.pcap-------------e-- ./cdb.log-------------e-- ./snmpd.cnf~-------------e-- ./snmpd_short.cnf-------------e-- ./snmpd_short.cnf~
```

#### chattr 修改文件属性

```
chattr +i file    添加该属性后，文件无法被删除，即使是 root 账户chattr -i file
```

## 查看磁盘信息

### 查看分区信息

```
fdisk -l
```

### 查看挂载信息

```
df -h    -- 显示所有挂载节点信息df -i    -- 显示节点信息df -aT    -- 显示特殊挂载点
```

注：删除文件后，如果产生文件的进程没有退出，那么就不会释放文件节点，这个文件所占用的空间是不会被释放的。

### 查看目录占用磁盘大小

```
du -h    -- 列出所有文件，以及以便于阅读的方式列出当前目录占用的磁盘空间du -sh    -- 安静模式，近列出当前目录占用的磁盘空间du -c    -- 列出当前目录及所有子目录所占用的磁盘大小ls | xargs du -sh    -- 列出当前目录所有文件大小，目录则显示当前目录所占磁盘空间
```

### 查看分区 UUID 等分区信息

```
blkid
```

### 查看分区和挂载对应关系

```
lsblk
```

### 查看挂载信息

```
mount
```

## 磁盘分区

### fdisk

     直接输入 fdisk /dev/sdN 进入交互模式进行磁盘分区管理操作，输入 fdisk 加参数则可以直接操作而不进入交互模式。

查看分区信息

```
fdisk -l
```

- -b<分区大小>：指定每个分区的大小；
- -l：列出指定的外围设备的分区表状况；
- -s<分区编号>：将指定的分区大小输出到标准输出上，单位为区块；
- -u：搭配”-l”参数列表，会用分区数目取代柱面数目，来表示每个分区的起始地址； -v：显示版本信息。

分区

```
fdisk /dev/sdanwquit
```

交互模式下支持的操作

```
root@vm1:~# fdisk /dev/sdaCommand (m for help): mCommand action   a   toggle a bootable flag   b   edit bsd disklabel   c   toggle the dos compatibility flag   d   delete a partition   l   list known partition types   m   print this menu   n   add a new partition   o   create a new empty DOS partition table   p   print the partition table   q   quit without saving changes   s   create a new empty Sun disklabel   t   change a partition\'s system id   u   change display/entry units   v   verify the partition table   w   write table to disk and exit   x   extra functionality (experts only)
```

### parted

```
parted -lparted -l /dev/sdbparted /dev/sdb     help     mklabel gpt -- 创建 gpt 格式的分区表（mbr 格式的分区表分区容量上限是 2T）     mkpart primary -- 创建主分区（交互模式）
```

## 格式化

将磁盘分区 /dev/sda1 格式化为 ex4 分区

```
mkfs.ext4 /dev/sda1
```

## 挂载

### mount

```
mount    -- 列出当前所有的挂载点mount -a    -- 挂载 /etc/fstab 文件中定义的所有磁盘mount /dev/sda1 /mnt    -- 将分区 /dev/sda1 挂载到 mnt 分区mount -o compress=lzo /dev/sdb1 /media/Nas-1    -- 挂载时指定启用 btrfs 文件格式的压缩功能并指定使用 lzo 压缩算法mount -t ntfs-3g -o uid=1002,gid=1002 -U uuid /media/Alice    -- 设置挂载的所有文件的所有者和用户组为 1002，通过 uuid 识别盘符，可通过 blkid 查看所有磁盘的 uuid    mount 172.0.5.5:/vob /vob    -- 将 172.0.5.5 上通过 nfs 共享出来的 /vob 目录挂载到本地 /vob/sbin/mount.cifs //192.168.0.6/nas-0 /media/nas-0 -o credentials=/root/account    -- 挂载 //192.168.0.6/nas-0 的 samba 共享目录到 /media/nas-0，账号内容配置在 /root/accourt    -- account 内容：        cat /root/account        username=root        password=9012345678mount -t tmpfs -o size=1G tmpfs /mnt/tmp/    -- 挂载 1G 内存到 /mnt/tmp 目录下作为硬盘使用mount -t aufs -o dirs=./mem:/etc none /etc    -- 联合挂载 mem 和 /etc 到 /etc，这样写入到 /etc 的所有文件都会保存到 mem 目录下了mount --bind dir1 dir2    -- 将目录 dir1 挂载到 dir2，这样可以直接从 dir2 访问 dir1，可以直接读写mount --rbind dir1 dir2    -- 递归挂载 dir1 到 dir2，dir1 下的其它所有挂载点也会挂载到 dir2mount -o remount,ro,bind dir1 dir2    -- 在将 dir1 挂载到 dir2 之后再执行这一条命令，则可以限制从 dir2 访问 dir1 上的文件的权限为只读。
```

#### LVM

LVM 分区的挂载方式和普通分区的挂载不同，不能直接用 mount /dev/xxx 的方式挂载。

##### 查看是否 LVM

```
fdisk -l    -- 如果显示的 SYSTEM 项为 Linux LVM，则说明是 LVM 分区
```

##### 挂载

ubuntu 14.04 下需要安装 lvm2，装完后需要重启

```
root@ubuntu:~# pvs  PV         VG         Fmt  Attr PSize   PFree  /dev/sdb2  VolGroup00 lvm2 a--  199.88g    0 
```

```
root@ubuntu:~# lvdisplay VolGroup00  --- Logical volume ---  LV Path                /dev/VolGroup00/LogVol00  LV Name                LogVol00  VG Name                VolGroup00  LV UUID                5RA9OH-b3wi-5xPy-YZiM-GZZd-WEVw-QBCGGx  LV Write Access        read/write  LV Creation host, time ,   LV Status              available  # open                 0  LV Size                182.25 GiB  Current LE             5832  Segments               1  Allocation             inherit  Read ahead sectors     auto  - currently set to     256  Block device           252:0  --- Logical volume ---  LV Path                /dev/VolGroup00/LogVol01  LV Name                LogVol01  VG Name                VolGroup00  LV UUID                nYHMtE-v60K-Bdle-14YM-XaHD-wu4K-9TrhiE  LV Write Access        read/write  LV Creation host, time ,   LV Status              available  # open                 0  LV Size                17.62 GiB  Current LE             564  Segments               1  Allocation             inherit  Read ahead sectors     auto  - currently set to     256  Block device           252:1
```

```
mount /dev/VolGroup00/LogVol00 /mnt
```

### 开机自动挂载

将需要挂载的规则写入到 /etc/fstab 文件中，即可实现开机挂载

配置示例

```
# <file system>                             <mount point>   <type>  <options>       <dump>  <pass>#挂载指定 UUID 设备为根目录，若有错误则重新挂载为只读模式UUID=873d80e1-df64-4aec-ac1a-af22eae93434 /               ext4    errors=remount-ro 0       1#挂载指定 UUID 设备为 swap 分区UUID=b5bf4548-fc73-4690-a6b4-30186a1d2792 none            swap    sw              0       0# 挂载 20G 内存到 /mnt/ram 作为硬盘使用tmpfs   /mnt/ram    tmpfs   defaults,size=20G   0   0# 挂载 nfs 共享目录192.168.2.6:/media/nas-0 /media/nas-0 nfs intr 0 0
```

## 交换分区

### 挂载交换分区

```
#挂载指定 UUID 设备为 swap 分区UUID=b5bf4548-fc73-4690-a6b4-30186a1d2792 none            swap    sw              0       0
```

### 交换分区控制

```
swapon -s    -- 显示正在使用的交换分区设备信息swapon -p 1 /dev/sda5    -- 设置交换分区 /dev/sda5 的优先级为 1swapon -a    -- 开启 /etc/fstab 下配置的所有交换分区swapoff -a    -- 关闭所有的交换分区
```

## 磁盘控制

     hdparm 可以用来控制磁盘进行休眠等操作，或设置磁盘的参数。

```
hdparm -Y /dev/sdb     -- 立即让 sdb1 进入睡眠模式hdparm -y /dev/sdb     -- 立即让 sdb1 进入省电模式hdparm -S [num] /dev/sdb     sudo hdparm -S 180 /dev/sdb     -- 设置硬盘 sdb1 在多久没有操作之后进入睡眠模式          0-240 单位是 5s，这里的 180 也即 15 分钟，240 表示 20 分钟，240 以后的单位是 0.5 小时，也即 241 表示 0.5 小时，242 表示 1 小时。hdparm -g /dev/sdb     -- 显示硬盘的磁轨，磁头，磁区等参数hdparm -t /dev/sdb     -- 评估硬盘的读取速度hdparm -T /dev/sdb     -- 评估硬盘快取的读取速度hdparm -v /dev/sdb     -- 显示硬盘的相关设定hdparm -C /dev/sdb     -- 检测硬盘的电源管理模式
```

如果 hdparm 无法支持，则可以试试 hd-idle 或 sdparm。

## 磁盘状态监控

```
使用 S.M.A.R.T 查看磁盘状态信息，并开启自动监控服务，以下以 rhel 7.1 的配置作为示例。
```

### 安装

```
yum install -y smartmontools
```

### 控制命令

#### 1）查看帮助信息

```
man smartdman smartctlman smartd.conf
```

#### 2）查看信息

```
smartctl --scan     -- 扫描设备smartctl -i /dev/sda     -- 显示设备标识信息smartctl -a /dev/sda     -- 显示设备所有 SMART 信息smartctl -x /dev/sda     -- 显示设备所有信息smartctl -H /dev/sda     -- 显示设备 SMART 健康状态smartctl -c /dev/sda     -- 确认设备支持自我检测，执行后需要看到以下信息：          Short self-test routinerecommended polling time:      (   2) minutes.Extended self-test routinerecommended polling time:      ( 125) minutes.eg：smartctl -H /dev/sdasmartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.19.0-54-generic] (local build)Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org=== START OF READ SMART DATA SECTION ===SMART overall-health self-assessment test result: PASSEDPASSED 表示测试通过，硬盘状态良好。
```

#### 3）对设备启用 SMART

```
smartctl --smart=on --offlineauto=on --saveauto=on /dev/sda     -- 在 /dev/sda 上启用 SMARTsudo smartctl -s on -o on -S on /dev/sdaeg：[root@toy ~]# smartctl --smart=on --offlineauto=on --saveauto=on /dev/sdasmartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-229.el7.x86_64] (local build)Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org=== START OF ENABLE/DISABLE COMMANDS SECTION ===SMART Enabled.SMART Attribute Autosave Enabled.SMART Automatic Offline Testing Enabled every four hours.     -- 每四小时测试一次
```

#### 4）磁盘测试

```
smartctl -t short /dev/sda     -- 简单测试（2分钟）smartctl --test=long /dev/sda     -- 详细测试（84分钟）root@ltepc7:~# smartctl --test=long /dev/sdasmartctl 6.2 2013-07-26 r3841 [x86_64-linux-3.19.0-54-generic] (local build)Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.org=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===Sending command: "Execute SMART Extended self-test routine immediately in off-line mode".Drive command "Execute SMART Extended self-test routine immediately in off-line mode" successful.Testing has begun.Please wait 84 minutes for test to complete.Test will complete after Wed Apr  6 18:15:43 2016Use smartctl -X to abort test.root@ltepc7:~#
```

#### 5）查看测试结果

```
smartctl
```

### 后台监控

#### 1）配置

```
vi /etc/smartmontools/smartd.conf/dev/sda -a -I 194 -W 4,45,55 -R 5 -m lte@casachina.com.cn     -- 监控标准温度以外的所有参数，当 SMART 状态为 failures 或 温度高于 55 摄氏度的时候，向外发送邮件通知/dev/sda -H -C 0 -U 0 -m lte@casachina.com.cn     -- 静默检测，只在检测到 SMART 健康状态为 failes 的时候发送邮件/dev/sda -a -d sat -o on -S on -s (S/../.././02|L/../../6/03) -m lte@casachina.com.cn     -- /dev/sda: 指定设备     -- -a: 启用一些选项     -- -d sat: 如果在使用 smartctl 命令的时候需要通过 -d TYPE 选项指定设备连接方式，则这里必须加入这个设置，否则无法正常工作。如果使用 smartctl 命令不需要通过 -d 选项指定连接方式，则这里也不需要     -- -o On， -S on: 和 smartctl 命令中的相同     -- -s (S/../.././02|L/../../6/03): 指定短时自我检测时间为每天早上 2 点，长时自我检测时间为每周六早上 3 点     -- -m 如果检测到错误，则向指定邮箱发送邮件。该选项需要一个可用的 email 设置，大多数的 linux 发行版自带了可用的发送邮件功能
```

#### 2）启用后台监控进程

```
redhat：     service smartd start     service smartd.service startubuntu：     /usr/sbin/smartd3）查看后台进程状态[root@toy nfv2_1]# service smartd statusRedirecting to /bin/systemctl status  smartd.service● smartd.service - Self Monitoring and Reporting Technology (SMART) Daemon   Loaded: loaded (/usr/lib/systemd/system/smartd.service; enabled; vendor preset: enabled)   Active: active (running) since Wed 2016-04-06 16:38:17 CST; 1min 59s agoMain PID: 6550 (smartd)   CGroup: /system.slice/smartd.service           └─6550 /usr/sbin/smartd -n -q neverApr 06 16:38:17 toy smartd[6550]: smartd 6.2 2013-07-26 r3841 [x86_64-linux-3.10.0-229.el7.x86_64] (local build)Apr 06 16:38:17 toy smartd[6550]: Copyright (C) 2002-13, Bruce Allen, Christian Franke, www.smartmontools.orgApr 06 16:38:17 toy smartd[6550]: Opened configuration file /etc/smartmontools/smartd.confApr 06 16:38:17 toy smartd[6550]: Configuration file /etc/smartmontools/smartd.conf was parsed, found DEVICESCAN, scanning devicesApr 06 16:38:17 toy smartd[6550]: Device: /dev/sda, type changed from 'scsi' to 'sat'Apr 06 16:38:17 toy smartd[6550]: Device: /dev/sda [SAT], openedApr 06 16:38:17 toy smartd[6550]: Device: /dev/sda [SAT], WDC WD1002F9YZ-09H1JL1, S/N:WD-WMC5K0D83J8U, WWN:5-0014ee-0aeb2...1.00 TBApr 06 16:38:17 toy smartd[6550]: Device: /dev/sda [SAT], not found in smartd database.Apr 06 16:38:17 toy smartd[6550]: Device: /dev/sda [SAT], is SMART capable. Adding to "monitor" list.Apr 06 16:38:17 toy smartd[6550]: Monitoring 1 ATA and 0 SCSI devicesHint: Some lines were ellipsized, use -l to show in full.
```

## 磁盘测试

### hdparm

```
hdparm -Tt /dev/sda/dev/sda:Timing cached reads: 6676 MB in 2.00 seconds = 3340.18 MB/secTiming buffered disk reads: 218 MB in 3.11 seconds = 70.11 MB/sec
```

可以看到,2 秒钟读取了 6676MB 的缓存,约合 3340.18 MB/sec; 
在 3.11 秒中读取了 218MB 磁盘(物理读),读取速度约合 70.11 MB/sec

### dd

/dev/null 伪设备，回收站.写该文件不会产生 IO 
/dev/zero 伪设备，会产生空字符流，对它不会产生 IO

测试方法

```
time dd if=/dev/zero of=test.dbf bs=8k count=300000    -- 测试磁盘的 IO 写速度，如果要测试实际速度 还要在末尾加上 oflag=direct 测到的才是真实的 IO 速度dd if=test.dbf bs=8k count=300000 of=/dev/null    -- 测试磁盘的 IO 读速度，
```

## 文件恢复

### testdisk

testdisk 是分区表恢复、raid 恢复、分区恢复的开源免费工具（testdisk 支持如下文件系统: FAT12/FAT16/FAT32/NTFS/ext2/ext3/ext4）。

testdisk 支持的功能:

- 修复分区表
- 恢复已删除分区
- 用 FAT32 备份表恢复启动扇区
- 重建 FAT12/FAT16/FAT32 启动扇区
- 修复 FAT 表
- 重建 NTFS 启动扇区
- 用备份表恢复 NTFS 启动扇区
- 用 mft 镜像表(mft mirror)修复 mft 表
- 查找 ext2/ext3 备份的 superblock
- 从 FAT,NTFS 及 ext2 文件系统恢复删除文件
- 从已删除的 FAT,NTFS 及 ext2/ext3 分区复制文件。

```
安装apt-get install testdisk启动testdisk
```

启动软件之后会进入一个交互界面进行操作

### photorec

photorec 是一款用于恢复硬盘、光盘中丢失的视频、文档、压缩包等文件，或从数码相机存储卡中恢复丢失图片的数据恢复软件（因此，该软件命名为 photo recovery 这个名字)。

photorec 忽略文件系统，能直接从介质底层恢复数据，因此，在介质的文件系统严重破坏或被重新格式化后，它也能进行数据恢复。出于安全考虑, photorec 以只读方式来访问您要恢复数据所在的磁盘或存储卡介质。

```
安装apt-get install testdisk启动photorec
```

启动软件之后会进入一个交互界面进行操作

### extundelete

ext 可用于恢复 ext 系列格式文件系统中的被删除文件。

实用示例：

```
apt-get install extundeleteextundelete /dev/sda1 --restore-file file1extundelete /dev/sda1 --restore-directory /public/SQAextundelete /dev/sda1 --restore -all
```

extundelete 的源码可以从 <http://extundelete.sourceforce.net/> 下载，载安装 extundelete 之前要安装两个软件包 e2fsprogs 和 e2fslibs。

### formost

formost 是一个基于文件头和尾部信息以及文件的内件数据结构恢复文件的命令行工具。可以分析由 dd、Safeback和 Encase 等生成的镜像文件或驱动器。 
formost 支持以下文件格式的恢复：

```
avi, bmp, dll, doc, exe, gif, htm, jar, jpg, mbd, mov, mpg, pdf, png, ppt, rar, rif, sdw, sx, sxc, sxi, sxw, vis, wav, wmv, xls, zip
```

使用示例：

```
apt-get install formostformost -t png -i /dev/sda1    -- 恢复 /dev/sda1 中的 png 文件到当前目录的 output 文件夹（默认）下foremost -v -T -t doc,pdf,jpg,gif -i /dev/sda6 -o /media/disk/Recover    -- 恢复多种类型的文件到 /media/disk/Recover
```

## 磁盘修复

### 磁盘错误修复

#### fcsk

```
fcsk -p /dev/sda1    -- 尝试自动修复磁盘 /dev/sda1
```

### 分区恢复

#### testdisk

     TestDisk 是一款自由开源的数据恢复工具，主要设计用来帮助恢复丢失的磁盘分区，修复无法引导的磁盘中的软件问题，以及特定种类的病毒或人类过失（例如不慎抹除分区表）。 TestDisk也可用来收集关于某个损坏磁盘的详细信息，可以用来送给技师进一步分析。

## RAID

| RAID档次 | 最少硬盘 | 最大容错 | 可用容量 | 读取性能 | 写入性能 | 安全性                                           | 目的                           | 应用产业             |
| -------- | -------- | -------- | -------- | -------- | -------- | ------------------------------------------------ | ------------------------------ | -------------------- |
| 单一硬盘 | (引用)   | 0        | 1        | 1        | 1        | 无                                               |                                |                      |
| JBOD     | 1        | 0        | n        | 1        | 1        | 无（同RAID 0）                                   | 增加容量                       | 个人（暂时）存储备份 |
| 0        | 2        | 0        | n        | n        | n        | 一个硬盘异常，全部硬盘都会异常                   | 追求最大容量、速度             | 视频剪接缓存用途     |
| 1        | 2        | n-1      | 1        | n        | 1        | 最高，一个正常即可                               | 追求最大安全性                 | 个人、企业备份       |
| 5        | 3        | 1        | n-1      | n-1      | n-1      | 高                                               | 追求最大容量、最小预算         | 个人、企业备份       |
| 6        | 4        | 2        | n-2      | n-2      | n-2      | 安全性较RAID 5高                                 | 同RAID 5，但较安全             | 个人、企业备份       |
| 10       | 4        | n/2      | n/2      | n/2      | n/2      | 安全性高，但在同一个子组群中不能出现两颗毁损硬盘 | 综合RAID 0/1优点，理论速度较快 | 大型数据库、服务器   |

### Setup RAID 5

使用mdadm创建在/dev/md0上创建一个由sdb、sdc、sdd3块盘组成(另外1块盘sde为热备)的RAID5：

```
mdadm --create --verbose /dev/md0 --level=raid5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd --spare-devices=1 /dev/sde 
```

### Reference

- [wiki -zh - RAID](https://zh.wikipedia.org/wiki/RAID)

## 文件系统格式

### ext2/3/4

    是 linux 传统的常见磁盘格式，支持文件系统日志等特性。

#### 查看分区属性信息

```
tune2fs -l /dev/sda1
```

### btrfs

    是一种支持较多新特性的磁盘格式，支持透明压缩，写实复制，trim 等高级特性。 
要在系统中格式化为 btrfs 格式，需要先安装对应的工具。在 ubuntu 14.04 下需要安装：

```
apt-get install btrfs-tools
```

#### 格式化

格式化 /dev/sdb1 的分区标签为 myLabel，并设定 blockSize

```
mkfs.btrfs -L myLabel -n blockSzie /dev/sdb1
```

#### btrfs 工具

由于 btrfs 支持透明压缩等新特性，使用传统的 df 查看到的磁盘使用信息可能有误，需要使用 btrfs 的工具来查看。

查看磁盘使用情况

```
btrfs filesystem df /dev/sda1btrfs filesystem usage /dev/sda1    -- usage 是较新版本中新加的命令
```

碎片整理 
btrfs 支持在线碎片整理,要整理根文件夹:

```
btrfs filesystem defragment /
```

这并不会’整理整个文件系统参阅 Btrfs Wiki上的这个页面(<https://btrfs.wiki.kernel.org/index.php/Problem_FAQ#Defragmenting_a_directory_doesn.27t_work>)获得更多信息. 
要整理整个文件系统并观看冗长的输出:

```
btrfs filesystem defragment -r -v /
```

#### tips

强制使用写实复制

```
cp --reflink=always file1 file2
```

#### 相关资料

- [btrfs on archlinux wiki](https://wiki.archlinux.org/index.php/Btrfs_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [manage btrfs ](https://btrfs.wiki.kernel.org/index.php/Manpage/btrfs(5))
- [FAQ](https://btrfs.wiki.kernel.org/index.php/FAQ)
- [Problem FAQ](https://btrfs.wiki.kernel.org/index.php/Problem_FAQ)
- [Btrfs-zero-log ](https://btrfs.wiki.kernel.org/index.php/Btrfs-zero-log)

### NTFS

    linux kernel 中对 NTFS 格式的支持仅支持读取 NTFS 格式磁盘的文件，如果需要写入，需要安装 ntfs-3g，然后挂载为 ntfs-3g 格式，才能进行读写 
     通过这种方式提供对 NTFS 格式磁盘的支持，在写入数据的时候，需要耗费相当庞大的 CPU 资源。

### ramdisk

     传统意义上的，可以格式化，然后加载的内存磁盘文件。这在Linux内核2.0/2.2就已经支持，其不足之处是大小固定，之后不能改变。 
    装载后写入速度在 700M/S-800M/S 之间，如果umount再加载，只要不重启linux，那文件依然会保存在/dev/ramX中。

```
ls /dev/ram*    -- 查看可用的 ramdiskmkdir /mnt/testmke2fs /dev/ram0mount /dev/ram /mnt/test
```

### ramfs

    Ramfs顾名思义是内存文件系统，它处于虚拟文件系统(VFS)层，而不像ramdisk那样基于虚拟在内存中的其他文件系统(ex2fs)。因而，它无需格式化，可以创建多个，只要内存足够，在创建时可以指定其最大能使用的内存大小。缺省情况下，Ramfs被限制最多可使用内存大小的一半。可以通过maxsize(以kbyte为单位)选项来改变。 
    写入速度在900M/S-1100M/S之间，umount后再加载数据消失。

```
mkdir /testRammount -t ramfs none /testRAMmount -t ramfs none /testRAM -o maxsize=2000    -- 创建一个限定最大使用内存为2M的ramdisk
```

### tmpfs

     Tmpfs是一个虚拟内存文件系统，它不同于传统的用块设备形式来实现的Ramdisk，也不同于针对物理内存的Ramfs。Tmpfs可以使用物理内存，也可以使用交换分区。在Linux内核中，虚拟内存资源由物理内存(RAM)和交换分区组成，这些资源是由内核中的虚拟内存子系统来负责分配和管理。Tmpfs向虚拟内存子系统请求页来存储文件，它同Linux的其它请求页的部分一样，不知道分配给自己的页是在内存中还是在交换分区中。同Ramfs一样，其大小也不是固定的，而是随着所需要的空间而动态的增减。 
     系统所默认加载的 /dev/shm，也是 tmpfs。 
     写入速度在1.2G/S-1.3G/S，umount后再加载数据消失。

```
mount tmpfs /mnt/tmpfs -t tmpfs -o size=32m
```

## 参考

- [fdisk命令](http://man.linuxde.net/fdisk)
- [linux磁盘分区fdisk命令详解](http://linux008.blog.51cto.com/2837805/548711)
- [ubuntu下设置USB硬盘自动休眠 ](http://blog.wangfan.org:8088/?p=22)
- [linux下测试磁盘的读写IO速度(IO物理测速)](http://blog.csdn.net/zqtsx/article/details/25487185)
- [Linux硬盘性能检测 ](http://www.cnblogs.com/lurenjiashuo/p/hard-disk-performance-detection.html)
- [Linux 硬盘性能测试DD命令详解 ](http://iblog.daobidao.com/linux-hard-drive-performance-test-dd-command-detailed.DaoBiDao)
- [monitoring-hard-drive-health-on-linux-with-smartmontools ](https://blog.shadypixel.com/monitoring-hard-drive-health-on-linux-with-smartmontools/)
- [Linux中ramdisk,tmpfs,ramfs比较与说明 ](http://www.embeddedlinux.org.cn/html/filesys/201308/26-2612.html?utm_source=tuicool&utm_medium=referral)
- [TestDisk - 中文维基 ](https://zh.wikipedia.org/zh/TestDisk)
- [使用 Linux 文件恢复工具 ](https://www.ibm.com/developerworks/cn/linux/1312_caoyq_linuxrestore/index.html)