---
layout: post
title:  "Linux 内存管理"
categories: Linux
tags: Linux 内存管理
author: DJY
---

* content
{:toc}
# Linux 内存管理

   操作系统的内存管理是以分段和分页的方式进行管理，分段是粗粒度的管理方式，分页是细粒度的管理方式。

   linux的内存管理采取的是分页存取机制，系统默认把内存划分为一个个 4K 的页，系统按照 4K 为单位进行管理。

   为了保证物理内存能得到充分的利用，内核会按照LRU算法在适当的时候将物理内存中不经常使用的内存页自动交换到虚拟内存中，

   当系统内存较大时（有说法是超过 12G），由于分页过多，系统在换页时候进行的查询操作会导致较大的开销继而导致性能下降，使用 hugepages 可以提高系统性能。

# 查看

------

## 查看系统内存

```
free -mfree -h
```

eg:

```
[root@djy ~]# free -h              total        used        free      shared  buff/cache   availableMem:            31G        843M         27G         89M        2.4G         29GSwap:            0B          0B          0B
```

- total：总物理内存
- used：已使用内存
- free：完全未被使用的内存
- shared：应用程序共享内存
- buffers：缓冲区
- cached：系统缓存
- -buffers/cache：应用程序使用的内存大小，used减去缓存值
- +buffers/cache：所有可供应用程序使用的内存大小，free加上缓存值
- total = used + free
- -buffers/cache=used-buffers-cached，这个是应用程序真实使用的内存大小
- +buffers/cache=free+buffers+cached，这个是服务器真实还可利用的内存大小

## 查看系统内存详情

```
root@VM-MOBILE:~/src# cat /proc/meminfo MemTotal:       64865632 kBMemFree:        27893204 kBMemAvailable:   30150044 kBBuffers:          168792 kBCached:          2364764 kBSwapCached:            0 kBActive:          2520256 kBInactive:         488172 kBActive(anon):     608088 kBInactive(anon):    30860 kBActive(file):    1912168 kBInactive(file):   457312 kBUnevictable:           0 kBMlocked:               0 kBSwapTotal:      65982460 kBSwapFree:       65982460 kBDirty:               204 kBWriteback:             0 kBAnonPages:        543464 kBMapped:            69828 kBShmem:             97912 kBSlab:             201820 kBSReclaimable:     112608 kBSUnreclaim:        89212 kBKernelStack:        9472 kBPageTables:         3660 kBNFS_Unstable:          0 kBBounce:                0 kBWritebackTmp:          0 kBCommitLimit:    81638060 kBCommitted_AS:     807652 kBVmallocTotal:   34359738367 kBVmallocUsed:      141152 kBVmallocChunk:   34359585916 kBHardwareCorrupted:     0 kBAnonHugePages:    370688 kBCmaTotal:              0 kBCmaFree:               0 kBHugePages_Total:   16384HugePages_Free:        0HugePages_Rsvd:        0HugePages_Surp:        0Hugepagesize:       2048 kBDirectMap4k:      167916 kBDirectMap2M:    65818624 kB     -- hugepages
```

## pidstat -r 查看内存使用情况

```
apt-get install sysstat
```

```
pidstat -r -p 13298    -- 输出进程 13298 的内存状态pidstat -r -p 13298 1    -- 输出进程 13298 的内存状态，每个一秒输出一次，持续输出pidstat -r -p 13298 1 3    -- 输出进程 13298 的内存状态，每个一秒输出一次，一共输出三次pidstat -r -C virtVM-hsw    -- 查看 virtVM-hsw 命令的所有进程的内存状态pidstat -r -U 0    -- 输出用户 ID 为 0 的用户（root）的所有进程的内存使用情况
```

输出说明

| 项目     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| minflt/s | 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的 page fault 次数 |
| majflt/s | 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的 page 在 swap 中，这样的 page fault 为 major page fault，一般在内存使用紧张时产生 |
| VSZ      | 进程虚拟内存，也就是进程一共标记使用了这么多内存，但这些内存并不一定都分配了物理内存 |
| RSS      | 进程所分配到的实际的物理内存                                 |
| %MEM     | 进程使用的内存占总内存的百分比                               |

参考

- Linux 运行进程实时监控pidstat命令详解 
  <http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858874.html>

## pmap - 查看进程内存详细情况

------

pmap 可以查看一个进程的详细内存使用情况，包括该进程所使用的文件所占用的内存以及统计信息。

```
pmap pidpmap -x pid    -- x 显示详细信息pmap -X pid    -- 显示更多详细信息pmap -d pid    -- 显示设备格式，最后一行的 mapped 表示虚拟内存，是进程可能会分配的总大小，writeable/private 是进程实际使用的物理内存，shared 是共享内存
```

| header     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Address    | 内存开始地址                                                 |
| Perm       | 内存访问权限，r=read, w=write, x=execute, s=shared, p=private(copy on write) |
| Offset     | 偏移量                                                       |
| Device     | 设备                                                         |
| Inode      | 节点数量                                                     |
| Size       | 进程占用的地址空间                                           |
| Rss        | 保留内存的字节数（KB），也就是实际被分配的内存的大小，包含共享库 |
| Pss        | 实际分配的内存大小，共享库占用的内存根据使用的共享库进程数量按比例分配 |
| Referenced |                                                              |
| Anonymous  |                                                              |
| Swap       |                                                              |
| Locked     | 是否允许swapped                                              |
| Mapping    | bash 对应的映像文件名                                        |

最后一行 
mapped：进程映射的虚拟地址空间大小，即进程预分配的虚拟内存，同 ps 中的 vsz 
writeable/private：进程的私有地址空间，即进程实际使用的内存 
shared：该进程和其他进程共享的内存

# 说明

## 系统内存

### VSS

进程可访问的全部地址空间。 
这里的大小包含了没有分配到实际的物理内存的空间，比如 malloc 分配了内存但没有写入，这时候分配的内存显示在 VSS 中，但并没有实际占用物理内存。如果要确定进程实际使用了多少内存，VSS 用处不大。

### RSS

RSS(Resident size) 内存指的是真正具有数据页的物理内存（包含进程用到的所有共享库占用的全部内存）。

RSS 可能会造成误导，因为这里报告的内存包含了进程使用到的共享库所占用的全部内存，即使一个共享库仅仅是被进程加载到内存，而无论有多少个进程使用了这个共享库。RSS 并并不能精确指出一个单独进程所占用的实际内存。

Linux 使用的是虚拟内存，进程的代码、库、堆栈使用的内存都会消耗内存，但是申请出来的内存（也就是虚拟内存），在 touch 之前，由于没有真正为之分配物理内存页，所以是不占用物理内存的。

这里的 RSS 内存也是 free 中的 used 内存，ps 中的 res 内存，pmap 的 RSS 内存。

### PSS

PSS 指的是进程占用的实际物理内存大小，对于该进程用到的共享库，会根据使用该库的进程数量，按比例显示该进程占用的内存。

比如说有一个共享库，同时被三个进程使用，而该共享库有 30 个页，那么在每个进程的 PSS 项指出的内存中，这个库都是占用了 10 个页。当有一个进程被 kill 掉之后，那么剩下的两个进程的 PSS 项指出的内存中，这个库都是占用了 15 个页。

由于一个进程被 kill 掉之后，其余使用了相同搞得库的 PSS 值会增加，而该进程 kill 掉之后，PSS 值并不会全部返回给整个系统，所以这里可能会造成一些误解。

### USS

USS 指的是一个进程独占的实际分配的私有内存。

USS 极其有用，因为这个值指出了一个正在运行的进程实际的内存增长情况。当一个进程被 kill 
掉之后，其 USS 指出的内存大小是真正返回到系统的内存。在最初怀疑一个进程内存泄漏时，USS 是最好的检查数字。

### SHR

SHR 内存表示一个进程调用的共享库占用的内存。 
RSS - SHR = USS

### slab 内存

Linux 为了提高性能，会对重复使用的对象进行缓冲，这个 slab 内存就是用来缓冲这些常用对象的。由于 Linux 会尽可能多的将重复使用的对象进行缓冲，所以往往会有大量的 slab 内存。

slab 内存可以使用 slabtop 来查看。

### page tables 内存

page tables 内存是管理物理内存所必须的开销 

## 进程内存

虚拟内存技术使得每个进程都可以独占整个内存空间，地址从零开始，直到内存上限。 每个进程都将这部分空间（从低地址到高地址）分为六个部分： 
![title](https://woniuxiang.space/api/file/getImage?fileId=5a153d2741a3d5000ca6ad6e) 
![title](https://woniuxiang.space/api/file/getImage?fileId=5a3383e941a3d5000ca6ae33) 
TEXT段：整个程序的代码，以及所有的常量。这部分内存是是固定大小的，只读的。 
DATA段，又称GVAR：初始化为非零值的全局变量。 
BSS段：初始化为0或未初始化的全局变量和静态变量。 
HEAP（堆空间）：动态内存区域，使用malloc或new申请的内存。 
未使用的内存。 
STACK（栈空间）：局部变量、参数、返回值都存在这里，函数调用开始会参数入栈、局部变量入栈；调用结束依次出栈。

其中堆空间和栈空间的大小是可变的，堆空间从下往上生长，栈空间从上往下生长。 
由于常量存储在TEXT段中，所有对常量的赋值都将产生segment fault异常。 
可以认为BSS段中的所有字节都是0。因为未初始化的全局变量、静态变量都在BSS段中， 所以它们都会被初始化为0，同时类的成员变量也会被初始化为0，但编译器不保证局部变量的初始化。

资料来自：<http://harttle.com/2015/07/22/memory-segment.html>

一段代码示例：

```
//main.cpp    int a = 0;                  //全局初始化区    char *p1;                   //全局未初始化区    main() {        int b;                  //栈        char s[] = "abc";       //栈        char *p2;               //栈        char *p3 = "123456";    //123456/0在常量区，p3在栈上。        static int c =0;        //全局（静态）初始化区        p1 = (char *)malloc(10);        p2 = (char *)malloc(20);//分配得来得10和20字节的区域就在堆区。        strcpy(p1, "123456");   //123456/0放在常量区，编译器可能会将它与p3所指向的"123456"优化成一个地方。    }    
```

# 管理

------

## 管理虚拟内存（swap）

```
swapoff    -- 禁用虚拟内存swapon     -- 启用虚拟内存
```

## sync 强制同步到磁盘

     Linux 系统中欲写入硬盘的资料有的时候会了效率起见，会写到 filesystem buffer 中，这个 buffer 是一块记忆体空间，如果欲写入硬盘的资料存于此 buffer 中，而系统又突然断电的话，那么资料就会流失了，sync 指令会将存于 buffer 中的资料强制写入硬盘中。

命令：

```
sync
```

## 释放内存

------

### 操作

```
[root@djy ~]# free -h              total        used        free      shared  buff/cache   availableMem:            31G        839M        322M         89M         30G         29GSwap:            0B          0B          0B[root@djy ~]# echo 1 > /proc/sys/vm/drop_caches [root@djy ~]# free -h              total        used        free      shared  buff/cache   availableMem:            31G        842M         27G         89M        2.4G         29GSwap:            0B          0B          0B
```

### 说明

#### 通过调整/proc/sys/vm/drop_caches来释放内存

- 0 – 不释放
- 1 – 释放页缓存
- 2 – 释放dentries和inodes
- 3 – 释放所有缓存 
  数字1是用来清空最近放问过的文件页面缓存 
  数字2是用来清空文件节点缓存和目录项缓存 
  数字3是用来清空1和2所有内容的缓存。

#### 关于drop_caches的官方说明如下：

Writing to this file causes the kernel to drop clean caches,dentries and inodes from memory, causing that memory to becomefree. 
To free pagecache, use echo 1 > /proc/sys/vm/drop_caches; 
to free dentries and inodes, use echo 2 > /proc/sys/vm/drop_caches; 
to free pagecache, dentries and inodes, use echo 3 >/proc/sys/vm/drop_caches. 
Because this is a non-destructive operation and dirty objects are not freeable, the user should run sync first.

# hugepages

------

## 相关术语

### page table

PT页面转换表。Linux 系统管理的是虚拟地址，page table 用于物理地址到虚拟之间的映射，因此对于内存的访问，先是访问Page Table，然后根据Page Table 中的映射关系，隐式的转移到物理地址来存取数据。

### TLB

Translation Lookaside Buffer，CPU中的一块固定大小的cache，包含了部分page table的映射关系，用于实现快速的虚拟地址到物理地址的转换。如果 CPU 要读取的数据已经在 TLB 中（TLB hit）则可以直接读取，否则（TLB miss）就要去查询内存。

### hugetlb

hugetlb 是 TLB 中指向 HugePage 的一个 entry (通常大于4k或预定义页面大小)。 HugePage 的读取通过 hugetlb entries 来实现，也可以理解为 HugePage 是 hugetlb page entry的一个句柄。

### hugetlbfs

一个类似于 tmpfs 的新的 in-memory filesystem，分配在 hugetblfs 这个类型的文件系统上的分页实际上都是被分配在 hugepages 中的，这个特性在 2.6 内核被提出。

## hugepage 的作用

### 减少分页

当系统内存过大的时候，如果按照默认的 4K 的页大小进行管理，将会有大量的页，系统要管理这些页需要较大的性能开销（50G 的内存，在 4K 分页的情况下，page table 的大小越为 800M - (50*1024*1024)kb/4kb*64bytes/1024/1024=800mb；CPU 的 cache 的命中率严重下降，查询开销增大）。

总的来说，如果只用 4K 的分页，那么系统内存越大，越容易造成 TLB miss，影响 CPU 性能，也会浪费更多的内存去管理这些页。

hugepage 将内存的分页划分得更大之后，可以减少分页数量，减少分页数量之后可以带来下面两个好处：

- 提高 CPU cache 命中率
- 减小 page table，避免使用过多的内存来管理分页

### Not swappable

hugepage 是无法被交换到硬盘上的，所有就没有page table lookups，另外还可以避免被交换到硬盘上之后重新加载进内存导致的性能下降。

## 4. 注意事项

### 4.1 系统启动时分配

HugePage使用的是共享内存，在操作系统启动期间被动态分配并被保留，因为他们不会被置换。

### 4.2 独占性

由于不会被置换的特点，在使用hugepage的内存不能被其他的进程使用。所以要合理设置该值，避免造成内存浪费。

### 4.3 Oracle server

对于只使用Oracle的服务器来说，把Hugepage设置成SGA(所有instance SGA之和)大小即可。

### 4.4 环境改变需要重设

如果增加HugePage或添加物理内存或者是当前服务器增加了新的instance以及SGA发生变化，应该重新设置所需的HugePage。

## 设置 hugepages 大小的方法

------

设置 hugepages 的方式有多种，以下是常用的几种：

方法 1. sysctl

```
sysctl -w vm.nr_hugepages=51000    -- 修改立即生效，重启失效
```

方法 2. 直接修改 /proc 下的文件

```
echo 51000 > /proc/sys/vm/nr_hugepages    -- 修改立即生效，重启失效
```

方法 3. 通过 grub 修改 
在 /etc/default/grub 下的 GRUB_CMDLINE_LINUX_DEFAULT 选项中添加 hugepages 的配置：

```
GRUB_CMDLINE_LINUX_DEFAULT="default_hugepagesz=1G hugepagesz=1G hugepages=32
```

```
update-grubreboot
```

## 查看 hugepages

------

    cat /proc/meminfo 下的下列几行：

```
HugePages_Total:   16384        -- 分配的 hugepages 总数量HugePages_Free:        0        -- 空闲的 hugepages 数量HugePages_Rsvd:        0        -- 保留的 hugepaegs 数量HugePages_Surp:        0Hugepagesize:       2048 kB     -- 单个 hugepages 的大小DirectMap4k:      167916 kB     DirectMap2M:    65818624 kB  
```

## kvm 虚拟机使用 hugepages

------

    kvm 虚拟机要使用 hugepages 的话，需要在物理机中配置 hugepages，并且物理机中配置的 hugepages 大小要大于虚拟机的总内存大小，而不是虚拟机内所使用的 hugepages 的大小。

### 1. 物理机中配置 hugepages

     参考上面的设置方法

### 2. 指定虚拟机使用 hugepages

virsh edit vm，加入下列配置项：

```
  <memoryBacking>    <hugePages/>  </memoryBacking>
```

### 3. 修改 /etc/default/qemu-kvm

     将 /etc/default/qemu-kvm 中的 KVM_HUGEPAGES=0 配置项为 KVM_HUGEPAGES=1，修改后需要重启物理机

# KSM

KSM(Kernel Samepage Merging) 是 Linux 内核中的一个内存管理特性，它可以合并内存中相同的页，以起到节省内存，其基于 CoW 技术。

KSM 的管理和监控通过 sysfs（位于根 /sys/kernel/mm/ksm）执行

## 启用/禁用

```
echo 0 > /sys/kernel/mm/ksm/run    -- 关闭 ksmd 守护进程，禁用 KSM 特性echo 1 > /sys/kernel/mm/ksm/run    -- 启动 ksmd 守护进程，启用 KSM 特性echo 2 > /sys/kernel/mm/ksm/run    -- 从运行状态停止 KSM 并请求取消合并所有合并页面
```

## 配置

- pages_to_scan: 一次扫描的页数。将此数目设置为较高的值会影响性能
- sleep_millisecs: 扫描之间的时间间隔
- merge_across_nodes: 允许在 NUMA 节点中执行合并

## 查看

- full_scans: 为重复内容扫描 KSM 的次数
- pages_shared: 合并的页面数
- pages_sharing: 正在共享单个页面的虚拟页面数
- pages_unshared: 作为共享候选者但当前未共享的页数
- pages_volatile: 作为共享候选者但频繁更改的页数。不会合并这些页面

较高的 pages_sharing/pages_shared 比率表明高效的页面共享（反之则表明资源浪费）。

# 参考

- [hugepages： Linux HugePage 特性 ](http://blog.csdn.net/leshami/article/details/8777639)
- [hugepages：在Linux 64位系统下使用hugepage](https://blogs.oracle.com/Database4CN/entry/%E5%9C%A8linux_64%E4%BD%8D%E7%B3%BB%E7%BB%9F%E4%B8%8B%E4%BD%BF%E7%94%A8hugepage)
- [KSM(Kernel Samepage Merging) 剖析：Linux 内核中的内存去耦合 ](http://blog.csdn.net/summer_liuwei/article/details/6013255)
- <https://www.ibm.com/support/knowledgecenter/zh/SSZJY4_3.1.0/liabp/liabpksm.htm> 
  <https://www.ibm.com/support/knowledgecenter/zh/SSZJY4_3.1.0/liabp/liabpksm.htm>