---
layout: post
title:  "Linux procfs文件系统"
categories: Linux
tags: Linux process 文件系统
author: DJY
---

* content
{:toc}
# 概述

在许多类 Unix 计算机系统中， procfs 是 进程 文件系统 (file system) 的缩写，包含一个伪文件系统（启动时动态生成的文件系统），用于通过内核访问进程信息。这个文件系统通常被挂载到 /proc 目录。由于 /proc 不是一个真正的文件系统，它也就不占用存储空间，只是占用有限的内存。

以下操作系统支持 procfs :

- Solaris
- BSD
- Linux（将此概念扩展到了非进程相关数据）
- IBM AIX （其实现基于Linux以提高兼容性）
- QNX
- 贝尔实验室九号项目（此概念之源头）

# Tools

Linux下使用 /proc 的基本工具是 procps (/proc processes) 中的程序，这个程序只对 procfs 具有意义。procfs 对部分功能从核心态移到用户态的过程中产生重大的意义。像是 GNU 版本的 ps 只需在用户态底下运作通过 procfs 获取数据便可以完成所有的工作。

## 相关命令

- sysctl
- lsdev 收集相关设备的DMA, IRQ, I/O端口信息并汇总显示
- procinfo

# Files

## acpi/apm

电源管理系统（如果有的话）。

## buddyinfo

该文件主要用来发现内存碎片问题。通过 buddy 算法，把所有空闲的内存，以2的幂次方的形式，分成 11 个块链表，分别对应为 1、2、4、8、16、32、64、128、256、512、1024 个页块的链表（比如最后一个链表中记录的是所有可用的连续 1024 页的内存）。

在下面的输出中，Node 0 节点中，DMA 区域（direct memory access），内存中一共有 1 个 2^(0*PAGE_SIZE) chunks（1 个单页可用内存）。类似地，可用内存中有 6 个 2^(1*PAGE_SIZE) chunks（2 个可用的连续两页的内存），以及 2 个 2^(2*PAGE_SIZE) chunks（2 个连续 4 页可用的内存）

```
Node 0, zone      DMA      1      3      1      1      3      2      2      1      2      2      2 Node 0, zone    DMA32   1359   1897   1773   1122    841    400    152     26      9      0      0 Node 0, zone   Normal    724   1149    860    750    482    317    126     41      5      0      0 Node 1, zone   Normal   2407     77     31   2006   1644   1093    708    440    227      1      0 
```

- DMA Zone:
- DMA Zone: 给传统设备使用的低于 16M 的内存，因为这些设备无法使用 16M 以外的内存。
- DMA32 Zone(仅 x86): 一些设备无法访问 4G 以上的内存。在 x86 上，这个区域可能会被 Normal zone 覆盖。
- Normal Zone: 以上，不需要内核 tricks 就可以访问的地址，通常在 x86 上，这个范围是 16M 到 896M。很多内核操作要求使用的内存都是这个范围的内存。
- Highmem Zone (x86 only): 大于 896M 的内存范围。

[Making sense of /proc/buddyinfo](http://andorian.blogspot.com/2014/03/making-sense-of-procbuddyinfo.html)

[DocumentationRed Hat Enterprise Linux4Reference Guide 5.2.2. /proc/buddyinfo](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Reference_Guide/s2-proc-buddyinfo.html)

## bus

包含对应于计算机上各种总线的目录, 如input/PCI/USB. 在/sys/bus下包含更丰富的信息。

## crypto

可利用的加密模块列表

## cmdline

传递给内核的启动选项，一般可以通过 /etc/defaut/grub 文件中的 GRUB_CMDLINE_LINUX_DEFAULT 选项进行配置。

## devices

字符设备与块设备列表，按照设备ID排序，但给出了/dev名字的主要部分

```
root@lt:/proc# cat devices Character devices:  1 mem  4 /dev/vc/0  4 tty  4 ttyS  5 /dev/tty  5 /dev/console  5 /dev/ptmx  5 ttyprintk  6 lp  7 vcs 10 misc 13 input 21 sg 29 fb 89 i2c 99 ppdev108 ppp128 ptm136 pts180 usb189 usb_device216 rfcomm226 drm252 bsg253 watchdog254 rtcBlock devices:  1 ramdisk  2 fd259 blkext  7 loop  8 sd  9 md 11 sr 65 sd 66 sd 67 sd 68 sd 69 sd 70 sd 71 sd128 sd129 sd130 sd131 sd132 sd133 sd134 sd135 sd252 device-mapper253 virtblk254 mdp
```

## diskstat

该文件中记录了每一块逻辑磁盘设备的统计情况。

![title](https://woniuxiang.space/api/file/getImage?fileId=5af06a18a02688000e91bd33)

以截图中的 sda1 的数据为例：

| 域   | Value  | Quoted                              | 解释                       |
| ---- | ------ | ----------------------------------- | -------------------------- |
| F1   | 8      | major number                        | 此块设备的主设备号         |
| F2   | 1      | minor mumber                        | 此块设备的次设备号         |
| F3   | sda1   | device name                         | 此块设备名字               |
| F4   | 237    | reads completed successfully        | 成功完成的读请求次数       |
| F5   | 99     | reads merged                        | 读请求的次数               |
| F6   | 10914  | sectors read                        | 读请求的扇区数总和         |
| F7   | 528    | time spent reading (ms)             | 读请求花费的时间总和       |
| F8   | 152    | writes completed                    | 成功完成的写请求次数       |
| F9   | 1      | writes merged                       | 写请求合并的次数           |
| F10  | 847690 | sectors written                     | 写请求的扇区数总和         |
| F11  | 392    | time spent writing (ms)             | 写请求花费的时间总和       |
| F12  | 0      | I/Os currently in progress          | 次块设备队列中的IO请求数   |
| F13  | 704    | time spent doing I/Os (ms)          | 块设备队列非空时间总和     |
| F14  | 916    | weighted time spent doing I/Os (ms) | 块设备队列非空时间加权总和 |

## dma

正在使用中的 ISA 直接内存访问通道的列表

```
root@lt:/proc# cat dma  2: floppy 4: cascade
```

## filesystems

当前时刻内核支持的文件系统的列表。

```
root@lt:/proc# cat filesystems nodev   sysfsnodev   rootfsnodev   ramfsnodev   bdevnodev   procnodev   cgroupnodev   cpusetnodev   tmpfsnodev   devtmpfsnodev   debugfsnodev   securityfsnodev   sockfsnodev   pipefsnodev   devpts    ext3    ext2    ext4nodev   hugetlbfs    vfatnodev   ecryptfs    fuseblknodev   fusenodev   fusectlnodev   pstorenodev   mqueuenodev   rpc_pipefsnodev   nfsnodev   nfs4nodev   nfsdnodev   aufs
```

## fb

可利用的帧缓冲的列表

## fs

fs 是一个目录，记录了当前系统中的文件系统，根据文件系统类型进行分类。

```
root@lt:/proc# ls /proc/fs/aufs  ext4  fscache  jbd2  lockd  nfs  nfsd  nfsfs
```

## interrupts

中断记录

```
root@lt:/proc# cat interrupts            CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7         0:          6          0          0          0          0          0          0          0   IO-APIC-edge      timer  1:         10          0          0          0          0          0          0          0   IO-APIC-edge      i8042  4:        367          0          0          0          0          0          0          0   IO-APIC-edge      serial  6:          3          0          0          0          0          0          0          0   IO-APIC-edge      floppy  8:          1          0          0          0          0          0          0          0   IO-APIC-edge      rtc0  9:          0          0          0          0          0          0          0          0   IO-APIC-fasteoi   acpi 10:          0          0          0          0          0          0          0          0   IO-APIC-fasteoi   virtio2 11:          0          0          0          0          0          0          0          0   IO-APIC-fasteoi   uhci_hcd:usb1 12:        144          0          0          0          0          0          0          0   IO-APIC-edge      i8042 14:       6287          0          0          0    1141735          0          0          0   IO-APIC-edge      ata_piix 15:          0          0          0          0          0          0          0          0   IO-APIC-edge      ata_piix 24:          0          0          0          0          0          0          0          0   PCI-MSI-edge      virtio0-config 25:         91          0          0   84232702        140          0          0          0   PCI-MSI-edge      virtio0-input.0 26:          2          0          0          0       1158          0          0          0   PCI-MSI-edge      virtio0-output.0 27:          0          0          0          0          0          0          0          0   PCI-MSI-edge      virtio1-config 28:          1          0          0          0          0          0          0     256273   PCI-MSI-edge      virtio1-input.0 29:          9          0          0          0          0          0          0          0   PCI-MSI-edge      virtio1-output.0NMI:          0          0          0          0          0          0          0          0   Non-maskable interruptsLOC:   39343295   37660771   24147690   51279960   20351426   18186692   23984671   21409987   Local timer interruptsSPU:          0          0          0          0          0          0          0          0   Spurious interruptsPMI:          0          0          0          0          0          0          0          0   Performance monitoring interruptsIWI:          0          0          1          0          0          0          0          0   IRQ work interruptsRTR:          0          0          0          0          0          0          0          0   APIC ICR read retriesRES:   25388223   20888380   18803355   18111562   16836326   16518735   17135972   16903805   Rescheduling interruptsCAL:     108646      78862      77380     365930      36361     123345     106567      75250   Function call interruptsTLB:    3587213    3302240    3149845    3228497    3065378    3104638    2958032    2971308   TLB shootdownsTRM:          0          0          0          0          0          0          0          0   Thermal event interruptsTHR:          0          0          0          0          0          0          0          0   Threshold APIC interruptsMCE:          0          0          0          0          0          0          0          0   Machine check exceptionsMCP:       6372       6372       6372       6372       6372       6372       6372       6372   Machine check pollsHYP:          0          0          0          0          0          0          0          0   Hypervisor callback interruptsERR:          0MIS:          0root@lt:/proc# 
```

## ioports

当前正在使用的已注册输入输出端口区域的列表。

```
root@lt:/proc# cat ioports 0000-0cf7 : PCI Bus 0000:00  0000-001f : dma1  0020-0021 : pic1  0040-0043 : timer0  0050-0053 : timer1  0060-0060 : keyboard  0064-0064 : keyboard  0070-0071 : rtc0  0080-008f : dma page reg  00a0-00a1 : pic2  00c0-00df : dma2  00f0-00ff : fpu  0170-0177 : 0000:00:01.1    0170-0177 : ata_piix  01f0-01f7 : 0000:00:01.1    01f0-01f7 : ata_piix  0376-0376 : 0000:00:01.1    0376-0376 : ata_piix  03f2-03f2 : floppy  03f4-03f5 : floppy  03f6-03f6 : 0000:00:01.1    03f6-03f6 : ata_piix  03f7-03f7 : floppy  03f8-03ff : serial  0510-051b : QEMU0002:00  0600-063f : 0000:00:01.3    0600-0603 : ACPI PM1a_EVT_BLK    0604-0605 : ACPI PM1a_CNT_BLK    0608-060b : ACPI PM_TMR  0700-070f : 0000:00:01.3    0700-0707 : piix4_smbus0cf8-0cff : PCI conf10d00-ffff : PCI Bus 0000:00  afe0-afe3 : ACPI GPE0_BLK  c000-c01f : 0000:00:01.2    c000-c01f : uhci_hcd  c020-c03f : 0000:00:03.0    c020-c03f : virtio-pci  c040-c05f : 0000:00:04.0    c040-c05f : virtio-pci  c060-c07f : 0000:00:05.0    c060-c07f : virtio-pci  c080-c08f : 0000:00:01.1    c080-c08f : ata_piixroot@lt:/proc# 
```

## irq/N

/proc/irq 目录下是系统硬件中断相关的目录。里边的 N 表示各个数字，每个数字对应不同的中断类型。 
另外该目录下还有 irq 相关的 affinity 配置。

注：IRQ(interrupt request) 是发送给处理器的硬件信号，用来临时停止一个正在运行的程序并替换为一个指定的作为 interrupt hanlder 的程序运行。硬件中断用来处理类似于来自网卡，键盘和鼠标等硬件的数据。

### 中断类型

#### Master PIC

- IRQ 0 – system timer (cannot be changed)
- IRQ 1 – keyboard controller (cannot be changed)
- IRQ 2 – cascaded signals from IRQs 8–15 (any devices configured to use IRQ 2 will actually be using IRQ 9)
- IRQ 3 – serial port controller for serial port 2 (shared with serial port 4, if present)
- IRQ 4 – serial port controller for serial port 1 (shared with serial port 3, if present)
- IRQ 5 – parallel port 2 and 3 or sound card
- IRQ 6 – floppy disk controller
- IRQ 7 – parallel port 1. It is used for printers or for any parallel port if a printer is not present. It can also be potentially be shared with a secondary sound card with careful management of the port.

#### Slave PIC

- IRQ 8 – real-time clock (RTC)
- IRQ 9 – Advanced Configuration and Power Interface (ACPI) system control interrupt on Intel chipsets.[1] Other chipset manufacturers might use another interrupt for this purpose, or make it available for the use of peripherals (any devices configured to use IRQ 2 will actually be using IRQ 9)
- IRQ 10 – The Interrupt is left open for the use of peripherals (open interrupt/available, SCSI or NIC)
- IRQ 11 – The Interrupt is left open for the use of peripherals (open interrupt/available, SCSI or NIC)
- IRQ 12 – mouse on PS/2 connector
- IRQ 13 – CPU co-processor or integrated floating point unit or inter-processor interrupt (use depends on OS)
- IRQ 14 – primary ATA channel (ATA interface usually serves hard disk drives and CD drives)
- IRQ 15 – secondary ATA channel

## kcore

表示以核心格式存储的系统的物理内存。

## kmsg

内核输出的一些信息

## ksyms

保存由模块（X）工具使用的内核导出符号定义，用以动态链接和绑定可加载模块。

## loadavg

负载平均数

```
root@lt:/proc# cat loadavg 0.11 0.11 0.08 1/418 13504root@lt:/proc# 
```

## locks

```
root@lt:/proc# cat locks 1: POSIX  ADVISORY  READ  20998 08:01:396504 124 1242: POSIX  ADVISORY  READ  20998 08:01:396504 128 1283: POSIX  ADVISORY  READ  20998 08:01:396502 1073741826 10737423354: FLOCK  ADVISORY  WRITE 1023 08:01:1056295 0 EOF5: FLOCK  ADVISORY  WRITE 1173 00:11:10806 0 EOF6: POSIX  ADVISORY  READ  26822 00:26:276 4 47: POSIX  ADVISORY  READ  1824 00:26:196 4 48: POSIX  ADVISORY  READ  1824 00:26:192 4 49: POSIX  ADVISORY  WRITE 1824 00:26:194 0 010: FLOCK  ADVISORY  WRITE 1211 00:15:9663 0 EOF11: POSIX  ADVISORY  READ  1825 00:26:263 4 412: POSIX  ADVISORY  READ  1825 00:26:260 4 413: POSIX  ADVISORY  READ  1825 00:26:256 4 414: POSIX  ADVISORY  READ  1825 00:26:196 4 415: POSIX  ADVISORY  READ  1825 00:26:251 4 416: POSIX  ADVISORY  READ  1825 00:26:250 4 417: POSIX  ADVISORY  READ  1825 00:26:249 4 418: POSIX  ADVISORY  READ  1825 00:26:248 4 419: POSIX  ADVISORY  READ  1825 00:26:247 4 420: POSIX  ADVISORY  READ  1825 00:26:246 4 421: POSIX  ADVISORY  READ  1825 00:26:245 4 422: POSIX  ADVISORY  READ  1825 00:26:192 4 423: POSIX  ADVISORY  WRITE 1825 00:26:242 0 024: POSIX  ADVISORY  READ  21003 08:01:396504 128 12825: POSIX  ADVISORY  READ  21003 08:01:396502 1073741826 107374233526: FLOCK  ADVISORY  WRITE 802 00:11:12552 0 EOF27: OFDLCK ADVISORY  READ  1023 00:06:1029 0 EOFroot@lt:/proc# 
```

## malloc

只有在内核编译期间定义了 CONFIGDEBUGMALLOC 的时候才会存在这个文件。

## meminfo

meminfo 文件中记录了系统级别的内存使用统计信息。

| field          | description                                                  |
| -------------- | ------------------------------------------------------------ |
| MemTotal       | 所有内存(RAM)大小,减去一些预留空间和内核的大小。             |
| MemFree        | 完全没有用到的物理内存，lowFree+highFree                     |
| MemAvailable   | 在不使用交换空间的情况下，启动一个新的应用最大可用内存的大小，计算方式：MemFree+Active(file)+Inactive(file)-(watermark+min(watermark,Active(file)+Inactive(file)/2)) |
| Buffers        | 块设备所占用的缓存页，包括：直接读写块设备以及文件系统元数据(metadata)，比如superblock使用的缓存页。 |
| Cached         | 表示普通文件数据所占用的缓存页。                             |
| SwapCached     | swap cache中包含的是被确定要swapping换页，但是尚未写入物理交换区的匿名内存页。那些匿名内存页，比如用户进程malloc申请的内存页是没有关联任何文件的，如果发生swapping换页，这类内存会被写入到交换区。 |
| Active         | active包含active anon和active file                           |
| Inactive       | inactive包含inactive anon和inactive file                     |
| Active(anon)   | anonymous pages（匿名页），用户进程的内存页分为两种：与文件关联的内存页(比如程序文件,数据文件对应的内存页)和与内存无关的内存页（比如进程的堆栈，用malloc申请的内存），前者称为file pages或mapped pages,后者称为匿名页。 |
| Inactive(anon) | 见上                                                         |
| Active(file)   | 见上                                                         |
| Inactive(file) | 见上                                                         |
| SwapTotal      | 可用的swap空间的总的大小(swap分区在物理内存不够的情况下，把硬盘空间的一部分释放出来，以供当前程序使用) |
| SwapFree       | 当前剩余的swap的大小                                         |
| Dirty          | 需要写入磁盘的内存去的大小                                   |
| Writeback      | 正在被写回的内存区的大小                                     |
| AnonPages      | 未映射页的内存的大小                                         |
| Mapped         | 设备和文件等映射的大小                                       |
| Slab           | 内核数据结构slab的大小                                       |
| SReclaimable   | 可回收的slab的大小                                           |
| SUnreclaim     | 不可回收的slab的大小                                         |
| PageTables     | 管理内存页页面的大小                                         |
| NFS_Unstable   | 不稳定页表的大小                                             |
| VmallocTotal   | Vmalloc内存区的大小                                          |
| VmallocUsed    | 已用Vmalloc内存区的大小                                      |
| VmallocChunk   | vmalloc区可用的连续最大快的大小                              |

### 输出说明

- MemTotal 
  系统从加电开始到引导完成，firmware/BIOS要保留一些内存，kernel本身要占用一些内存，最后剩下可供kernel支配的内存就是MemTotal。这个值在系统运行期间一般是固定不变的。可参阅[解读DMESG中的内存初始化信息](http://linuxperf.com/?p=139)。
- MemFree 
  表示系统尚未使用的内存。[MemTotal-MemFree]就是已被用掉的内存。
- MemAvailable 
  有些应用程序会根据系统的可用内存大小自动调整内存申请的多少，所以需要一个记录当前可用内存数量的统计值，MemFree并不适用，因为MemFree不能代表全部可用的内存，系统中有些内存虽然已被使用但是可以回收的，比如cache/buffer、slab都有一部分可以回收，所以这部分可回收的内存加上MemFree才是系统可用的内存，即MemAvailable。/proc/meminfo中的MemAvailable是内核使用特定的算法估算出来的，要注意这是一个估计值，并不精确。

### 内存黑洞

追踪Linux系统的内存使用一直是个难题，很多人试着把能想到的各种内存消耗都加在一起，kernel text、kernel modules、buffer、cache、slab、page table、process RSS…等等，却总是与物理内存的大小对不上，这是为什么呢？因为Linux kernel并没有滴水不漏地统计所有的内存分配，kernel动态分配的内存中就有一部分没有计入/proc/meminfo中。

我们知道，Kernel的动态内存分配通过以下几种接口：

- alloc_pages/__get_free_page: 以页为单位分配
- vmalloc: 以字节为单位分配虚拟地址连续的内存块
- slab allocator
- kmalloc: 以字节为单位分配物理地址连续的内存块，它是以slab为基础的，使用slab层的general caches — 大小为2^n，名称是kmalloc-32、kmalloc-64等（在老kernel上的名称是size-32、size-64等）。

通过slab层分配的内存会被精确统计，可以参见/proc/meminfo中的slab/SReclaimable/SUnreclaim；

通过vmalloc分配的内存也有统计，参见/proc/meminfo中的VmallocUsed 和 /proc/vmallocinfo（下节中还有详述）；

而通过alloc_pages分配的内存不会自动统计，除非调用alloc_pages的内核模块或驱动程序主动进行统计，否则我们只能看到free memory减少了，但从/proc/meminfo中看不出它们具体用到哪里去了。比如在VMware guest上有一个常见问题，就是VMWare ESX宿主机会通过guest上的Balloon driver(vmware_balloon module)占用guest的内存，有时占用得太多会导致guest无内存可用，这时去检查guest的/proc/meminfo只看见MemFree很少、但看不出内存的去向，原因就是Balloon driver通过alloc_pages分配内存，没有在/proc/meminfo中留下统计值，所以很难追踪。 
而通过alloc_pages分配的内存不会自动统计，除非调用alloc_pages的内核模块或驱动程序主动进行统计，否则我们只能看到free memory减少了，但从/proc/meminfo中看不出它们具体用到哪里去了。比如在VMware guest上有一个常见问题，就是VMWare ESX宿主机会通过guest上的Balloon driver(vmware_balloon module)占用guest的内存，有时占用得太多会导致guest无内存可用，这时去检查guest的/proc/meminfo只看见MemFree很少、但看不出内存的去向，原因就是Balloon driver通过alloc_pages分配内存，没有在/proc/meminfo中留下统计值，所以很难追踪。

### 内存都到哪里去了

使用内存的，不是kernel就是用户进程，下面我们就分类讨论。

注：page cache比较特殊，很难区分是属于kernel还是属于进程，其中被进程mmap的页面自然是属于进程的了，而另一些页面没有被mapped到任何进程，那就只能算是属于kernel了。

#### 1. 内核

内核所用内存的静态部分，比如内核代码、页描述符等数据在引导阶段就分配掉了，并不计入MemTotal里，而是算作Reserved(在dmesg中能看到)。而内核所用内存的动态部分，是通过上文提到的几个接口申请的，其中通过alloc_pages申请的内存有可能未纳入统计，就像黑洞一样。

下面讨论的都是/proc/meminfo中所统计的部分。

##### 1.1 SLAB

通过slab分配的内存被统计在以下三个值中：

- SReclaimable: slab中可回收的部分。调用kmem_getpages()时加上SLAB_RECLAIM_ACCOUNT标记，表明是可回收的，计入SReclaimable，否则计入SUnreclaim。
- SUnreclaim: slab中不可回收的部分。
- Slab: slab中所有的内存，等于以上两者之和。

##### 1.2 VmallocUsed

通过vmalloc分配的内存都统计在/proc/meminfo的 VmallocUsed 值中，但是要注意这个值不止包括了分配的物理内存，还统计了VM_IOREMAP、VM_MAP等操作的值，譬如VM_IOREMAP是把IO地址映射到内核空间、并未消耗物理内存，所以我们要把它们排除在外。从物理内存分配的角度，我们只关心VM_ALLOC操作，这可以从/proc/vmallocinfo中的vmalloc记录看到：

```
# grep vmalloc /proc/vmallocinfo...0xffffc90004702000-0xffffc9000470b000   36864 alloc_large_system_hash+0x171/0x239 pages=8 vmalloc N0=80xffffc9000470b000-0xffffc90004710000   20480 agp_add_bridge+0x2aa/0x440 pages=4 vmalloc N0=40xffffc90004710000-0xffffc90004731000  135168 raw_init+0x41/0x141 pages=32 vmalloc N0=320xffffc90004736000-0xffffc9000473f000   36864 drm_ht_create+0x55/0x80 [drm] pages=8 vmalloc N0=80xffffc90004744000-0xffffc90004746000    8192 dm_table_create+0x9e/0x130 [dm_mod] pages=1 vmalloc N0=10xffffc90004746000-0xffffc90004748000    8192 dm_table_create+0x9e/0x130 [dm_mod] pages=1 vmalloc N0=1...
```

注：/proc/vmallocinfo中能看到vmalloc来自哪个调用者(caller)，那是vmalloc()记录下来的，相应的源代码可见： 
mm/vmalloc.c: vmalloc > __vmalloc_node_flags > __vmalloc_node > __vmalloc_node_range > __get_vm_area_node > setup_vmalloc_vm

通过vmalloc分配了多少内存，可以统计/proc/vmallocinfo中的vmalloc记录，例如：

```
# grep vmalloc /proc/vmallocinfo | awk '{total+=$2}; END {print total}'23375872
```

一些driver以及网络模块和文件系统模块可能会调用vmalloc，加载内核模块(kernel module)时也会用到，可参见 kernel/module.c。

##### 1.3 kernel modules (内核模块)

系统已经加载的内核模块可以用 lsmod 命令查看，注意第二列就是内核模块所占内存的大小，通过它可以统计内核模块所占用的内存大小，但这并不准，因为”lsmod”列出的是[init_size+core_size]，而实际给kernel module分配的内存是以page为单位的，不足 1 page的部分也会得到整个page，此外每个module还会分到一页额外的guard page。下文我们还会细说。

```
# lsmod | lessModule                  Size  Used byrpcsec_gss_krb5        31477  0 auth_rpcgss            59343  1 rpcsec_gss_krb5nfsv4                 474429  0 dns_resolver           13140  1 nfsv4nfs                   246411  1 nfsv4lockd                  93977  1 nfssunrpc                295293  5 nfs,rpcsec_gss_krb5,auth_rpcgss,lockd,nfsv4fscache                57813  2 nfs,nfsv4...
```

lsmod的信息来自/proc/modules，它显示的size包括init_size和core_size，相应的源代码参见：

```
// kernel/module.cstatic int m_show(struct seq_file *m, void *p){...        seq_printf(m, "%s %u",                   mod->name, mod->init_size + mod->core_size);...}
```

注：我们可以在 /sys/module// 目录下分别看到coresize和initsize的值。

kernel module的内存是通过vmalloc()分配的（参见下列源代码），所以在/proc/vmallocinfo中会有记录，也就是说我们可以不必通过”lsmod”命令来统计kernel module所占的内存大小，通过/proc/vmallocinfo就行了，而且还比lsmod更准确，为什么这么说呢？

```
// kernel/module.cstatic int move_module(struct module *mod, struct load_info *info){...        ptr = module_alloc_update_bounds(mod->core_size);...        if (mod->init_size) {                ptr = module_alloc_update_bounds(mod->init_size);...}// 注：module_alloc_update_bounds()最终会调用vmalloc_exec()
```

因为给kernel module分配内存是以page为单位的，不足 1 page的部分也会得到整个page，此外，每个module还会分到一页额外的guard page。 
详见：mm/vmalloc.c: __get_vm_area_node()

而”lsmod”列出的是[init_size+core_size]，比实际分配给kernel module的内存小。我们做个实验来说明：

```
# 先卸载floppy模块$ modprobe -r floppy# 确认floppy模块已经不在了$ lsmod | grep floppy# 记录vmallocinfo以供随后比较$ cat /proc/vmallocinfo > vmallocinfo.1# 加载floppy模块$ modprobe -a floppy# 注意floppy模块的大小是69417字节：$ lsmod | grep floppyfloppy                 69417  0 $ cat /proc/vmallocinfo > vmallocinfo.2# 然而，我们看到vmallocinfo中记录的是分配了73728字节：$ diff vmallocinfo.1 vmallocinfo.268a69> 0xffffffffa03d7000-0xffffffffa03e9000   73728 module_alloc_update_bounds+0x14/0x70 pages=17 vmalloc N0=17# 为什么lsmod看到的内存大小与vmallocinfo不同呢？# 因为给kernel module分配内存是以page为单位的，而且外加一个guard page# 我们来验证一下：$ bc -q69417%40963881    <--- 不能被4096整除69417/409616      <--- 相当于16 pages，加上面的3881字节，会分配17 pages18*4096 <--- 17 pages 加上 1个guard page73728   <--- 正好是vmallocinfo记录的大小
```

所以结论是kernel module所占用的内存包含在/proc/vmallocinfo的统计之中，不必再去计算”lsmod”的结果了，而且”lsmod”也不准。

##### 1.4 HardwareCorrupted

当系统检测到内存的硬件故障时，会把有问题的页面删除掉，不再使用，/proc/meminfo中的HardwareCorrupted统计了删除掉的内存页的总大小。相应的代码参见 mm/memory-failure.c: memory_failure()。

##### 1.5 PageTables

Page Table用于将内存的虚拟地址翻译成物理地址，随着内存地址分配得越来越多，Page Table会增大，/proc/meminfo中的PageTables统计了Page Table所占用的内存大小。

注：请把Page Table与Page Frame（页帧）区分开，物理内存的最小单位是page frame，每个物理页对应一个描述符(struct page)，在内核的引导阶段就会分配好、保存在mem_map[]数组中，mem_map[]所占用的内存被统计在dmesg显示的reserved中，/proc/meminfo的MemTotal是不包含它们的。（在NUMA系统上可能会有多个mem_map数组，在node_data中或mem_section中）。 
而Page Table的用途是翻译虚拟地址和物理地址，它是会动态变化的，要从MemTotal中消耗内存。

(区分 Page Table 和 Page Frame，简单来说 Page Frame 是记录所有物理页的，Page Table 是记录虚拟地址和物理地址之间的映射关系的)

##### 1.6 KernelStack

每一个用户线程都会分配一个kernel stack（内核栈），内核栈虽然属于线程，但用户态的代码不能访问，只有通过系统调用(syscall)、自陷(trap)或异常(exception)进入内核态的时候才会用到，也就是说内核栈是给kernel code使用的。在x86系统上Linux的内核栈大小是固定的8K或16K（可参阅我以前的文章：[内核栈溢出](http://linuxperf.com/?p=116)）。

Kernel stack（内核栈）是常驻内存的，既不包括在LRU lists里，也不包括在进程的RSS/PSS内存里，所以我们认为它是kernel消耗的内存。统计值是/proc/meminfo的KernelStack。

##### 1.7 Bounce

有些老设备只能访问低端内存，比如16M以下的内存，当应用程序发出一个I/O 请求，DMA的目的地址却是高端内存时（比如在16M以上），内核将在低端内存中分配一个临时buffer作为跳转，把位于高端内存的缓存数据复制到此处。这种额外的数据拷贝被称为“bounce buffering”，会降低I/O 性能。大量分配的bounce buffers 也会占用额外的内存。

#### 2. 用户进程

/proc/meminfo统计的是系统全局的内存使用状况，单个进程的情况要看/proc//下的smaps等等。

##### 2.1 Hugepages

Hugepages在/proc/meminfo中是被独立统计的，与其它统计项不重叠，既不计入进程的RSS/PSS中，又不计入LRU Active/Inactive，也不会计入cache/buffer。如果进程使用了Hugepages，它的RSS/PSS不会增加。

注：不要把 Transparent HugePages (THP)跟 Hugepages 搞混了，THP的统计值是/proc/meminfo中的”AnonHugePages”，在/proc//smaps中也有单个进程的统计，这个统计值与进程的RSS/PSS是有重叠的，如果用户进程用到了THP，进程的RSS/PSS也会相应增加，这与Hugepages是不同的。

在/proc/meminfo中与Hugepages有关的统计值如下：

```
MemFree: 570736 kB...HugePages_Total: 0HugePages_Free: 0HugePages_Rsvd: 0HugePages_Surp: 0Hugepagesize: 2048 kB
```

HugePages_Total 对应内核参数 vm.nr_hugepages，也可以在运行中的系统上直接修改 /proc/sys/vm/nr_hugepages，修改的结果会立即影响空闲内存 MemFree的大小，因为HugePages在内核中独立管理，只要一经定义，无论是否被使用，都不再属于free memory。在下例中我们设置256MB(128页)Hugepages，可以立即看到Memfree立即减少了262144kB（即256MB）：

```
# echo 128 > /proc/sys/vm/nr_hugepages# cat /proc/meminfo...MemFree: 308592 kB...HugePages_Total: 128HugePages_Free: 128HugePages_Rsvd: 0HugePages_Surp: 0Hugepagesize: 2048 kB
```

使用Hugepages有三种方式： 
(详见 <https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt>)

1. mount一个特殊的 hugetlbfs 文件系统，在上面创建文件，然后用mmap() 进行访问，如果要用 read() 访问也是可以的，但是 write() 不行。
2. 通过shmget/shmat也可以使用Hugepages，调用shmget申请共享内存时要加上 SHM_HUGETLB 标志。
3. 通过 mmap()，调用时指定MAP_HUGETLB 标志也可以使用Huagepages。

用户程序在申请Hugepages的时候，其实是reserve了一块内存，并未真正使用，此时/proc/meminfo中的 HugePages_Rsvd 会增加，而 HugePages_Free 不会减少。

```
HugePages_Total: 128HugePages_Free: 128HugePages_Rsvd: 128HugePages_Surp: 0Hugepagesize: 2048 kB
```

等到用户程序真正读写Hugepages的时候，它才被消耗掉了，此时HugePages_Free会减少，HugePages_Rsvd也会减少。

```
HugePages_Total: 128HugePages_Free: 0HugePages_Rsvd: 0HugePages_Surp: 0Hugepagesize: 2048 kB
```

我们说过，Hugepages是独立统计的，如果进程使用了Hugepages，它的RSS/PSS不会增加。下面举例说明，一个进程通过mmap()申请并使用了Hugepages，在/proc//smaps中可以看到如下内存段，VmFlags包含的”ht”表示Hugepages，kernelPageSize是2048kB，注意RSS/PSS都是0：

```
...2aaaaac00000-2aaabac00000 rw-p 00000000 00:0c 311151 /anon_hugepage (deleted)Size: 262144 kBRss: 0 kBPss: 0 kBShared_Clean: 0 kBShared_Dirty: 0 kBPrivate_Clean: 0 kBPrivate_Dirty: 0 kBReferenced: 0 kBAnonymous: 0 kBAnonHugePages: 0 kBSwap: 0 kBKernelPageSize: 2048 kBMMUPageSize: 2048 kBLocked: 0 kBVmFlags: rd wr mr mw me de ht...
```

##### 2.2 AnonHugePages

AnonHugePages统计的是Transparent HugePages (THP)，THP与Hugepages不是一回事，区别很大。

上一节说过，Hugepages在/proc/meminfo中是被独立统计的，与其它统计项不重叠，既不计入进程的RSS/PSS中，又不计入LRU Active/Inactive，也不会计入cache/buffer。如果进程使用了Hugepages，它的RSS/PSS不会增加。

而AnonHugePages完全不同，它与/proc/meminfo的其他统计项有重叠，首先它被包含在AnonPages之中，而且在/proc//smaps中也有单个进程的统计，与进程的RSS/PSS是有重叠的，如果用户进程用到了THP，进程的RSS/PSS也会相应增加，这与Hugepages是不同的。下例截取自/proc//smaps中的一段：

```
7efcf0000000-7efd30000000 rw-p 00000000 00:00 0 Size:            1048576 kBRss:              313344 kBPss:              313344 kBShared_Clean:          0 kBShared_Dirty:          0 kBPrivate_Clean:         0 kBPrivate_Dirty:    313344 kBReferenced:       239616 kBAnonymous:        313344 kBAnonHugePages:    313344 kBSwap:                  0 kBKernelPageSize:        4 kBMMUPageSize:           4 kBLocked:                0 kBVmFlags: rd wr mr mw me dc ac hg mg
```

THP也可以用于shared memory和tmpfs，缺省是禁止的，打开的方法如下（详见 <https://www.kernel.org/doc/Documentation/vm/transhuge.txt>）：

- mount时加上”huge=always”等选项
- 通过/sys/kernel/mm/transparent_hugepage/shmem_enabled来控制

因为缺省情况下shared memory和tmpfs不使用THP，所以进程之间不会共享AnonHugePages，于是就有以下等式： 
【/proc/meminfo的AnonHugePages】==【所有进程的/proc//smaps中AnonHugePages之和】 
举例如下：

```
# grep AnonHugePages /proc/[1-9]*/smaps | awk '{total+=$2}; END {print total}'782336# grep AnonHugePages /proc/meminfo AnonHugePages:    782336 kB
```

##### 2.3 LRU

LRU是Kernel的页面回收算法(Page Frame Reclaiming)使用的数据结构，在[解读vmstat中的Active/Inactive memory](http://linuxperf.com/?p=97)一文中有介绍。Page cache和所有用户进程的内存（kernel stack和huge pages除外）都在LRU lists上。

LRU lists包括如下几种，在/proc/meminfo中都有对应的统计值(anon 表示堆栈的内存，file 表示文件相关部分的内存)：

LRU_INACTIVE_ANON – 对应 Inactive(anon) 
LRU_ACTIVE_ANON – 对应 Active(anon) 
LRU_INACTIVE_FILE – 对应 Inactive(file) 
LRU_ACTIVE_FILE – 对应 Active(file) 
LRU_UNEVICTABLE – 对应 Unevictable

注：

- Inactive list里的是长时间未被访问过的内存页，Active list里的是最近被访问过的内存页，LRU算法利用Inactive list和Active list可以判断哪些内存页可以被优先回收。
- 括号中的 anon 表示匿名页(anonymous pages)。 
  用户进程的内存页分为两种：file-backed pages（与文件对应的内存页），和anonymous pages（匿名页），比如进程的代码、映射的文件都是file-backed，而进程的堆、栈都是不与文件相对应的、就属于匿名页。file-backed pages在内存不足的时候可以直接写回对应的硬盘文件里，称为page-out，不需要用到交换区(swap)；而anonymous pages在内存不足时就只能写到硬盘上的交换区(swap)里，称为swap-out。
- 括号中的 file 表示 file-backed pages（与文件对应的内存页）。
- Unevictable LRU list上是不能pageout/swapout的内存页，包括VM_LOCKED的内存页、SHM_LOCK的共享内存页（又被统计在”Mlocked”中）、和ramfs。在unevictable list出现之前，这些内存页都在Active/Inactive lists上，vmscan每次都要扫过它们，但是又不能把它们pageout/swapout，这在大内存的系统上会严重影响性能，设计unevictable list的初衷就是避免这种情况，参见： 
  <https://www.kernel.org/doc/Documentation/vm/unevictable-lru.txt>

LRU与/proc/meminfo中其他统计值的关系：

LRU中不包含HugePages_*。 
LRU包含了 Cached 和 AnonPages。

##### 2.4 Shmem

/proc/meminfo中的Shmem统计的内容包括：

shared memory 
tmpfs。 
此处所讲的shared memory又包括：

SysV shared memory [shmget etc.] 
POSIX shared memory [shm_open etc.] 
shared anonymous mmap [ mmap(…MAP_ANONYMOUS|MAP_SHARED…)] 
因为shared memory在内核中都是基于tmpfs实现的，参见： 
<https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt> 
也就是说它们被视为基于tmpfs文件系统的内存页，既然基于文件系统，就不算匿名页，所以不被计入/proc/meminfo中的AnonPages，而是被统计进了：

Cached (i.e. page cache) 
Mapped (当shmem被attached时候) 
然而它们背后并不存在真正的硬盘文件，一旦内存不足的时候，它们是需要交换区才能swap-out的，所以在LRU lists里，它们被放在：

Inactive(anon) 或 Active(anon) 
注：虽然它们在LRU中被放进了anon list，但是不会被计入 AnonPages。这是shared memory & tmpfs比较拧巴的一个地方，需要特别注意。 
或 unevictable （如果被locked的话） 
注意： 
当shmget/shm_open/mmap创建共享内存时，物理内存尚未分配，要直到真正访问时才分配。/proc/meminfo中的 Shmem 统计的是已经分配的大小，而不是创建时申请的大小。

##### 2.5 AnonPages

前面提到用户进程的内存页分为两种：file-backed pages（与文件对应的内存页），和anonymous pages（匿名页）。Anonymous pages(匿名页)的数量统计在/proc/meminfo的AnonPages中。

以下是几个事实，有助于了解Anonymous Pages：

- 所有page cache里的页面(Cached)都是file-backed pages，不是Anonymous Pages。”Cached”与”AnoPages”之间没有重叠。 
  注：shared memory 不属于 AnonPages，而是属于Cached，因为shared memory基于tmpfs，所以被视为file-backed、在page cache里，上一节解释过。
- mmap private anonymous pages属于AnonPages(Anonymous Pages)，而mmap shared anonymous pages属于Cached(file-backed pages)，因为shared anonymous mmap也是基于tmpfs的，上一节解释过。
- Anonymous Pages是与用户进程共存的，一旦进程退出，则Anonymous pages也释放，不像page cache即使文件与进程不关联了还可以缓存。
- AnonPages统计值中包含了Transparent HugePages (THP)对应的 AnonHugePages 。参见：

```
fs/proc/meminfo.c:static int meminfo_proc_show(struct seq_file *m, void *v){...#ifdef CONFIG_TRANSPARENT_HUGEPAGE                K(global_page_state(NR_ANON_PAGES)                  + global_page_state(NR_ANON_TRANSPARENT_HUGEPAGES) *                  HPAGE_PMD_NR),...
```

##### 2.6 Mapped

上面提到的用户进程的file-backed pages就对应着/proc/meminfo中的”Mapped”。Page cache中(“Cached”)包含了文件的缓存页，其中有些文件当前已不在使用，page cache仍然可能保留着它们的缓存页面；而另一些文件正被用户进程关联，比如shared libraries、可执行程序的文件、mmap的文件等，这些文件的缓存页就称为mapped。

/proc/meminfo中的”Mapped”就统计了page cache(“Cached”)中所有的mapped页面。”Mapped”是”Cached”的子集。

因为Linux系统上shared memory & tmpfs被计入page cache(“Cached”)，所以被attached的shared memory、以及tmpfs上被map的文件都算做”Mapped”。

进程所占的内存页分为anonymous pages和file-backed pages，理论上应该有： 
【所有进程的PSS之和】 == 【Mapped + AnonPages】。 
然而我实际测试的结果，虽然两者很接近，却总是无法精确相等，我猜也许是因为进程始终在变化、采集的/proc/[1-9]*/smaps以及/proc/meminfo其实不是来自同一个时间点的缘故。

##### 2.7 Cached

Page Cache里包括所有file-backed pages，统计在/proc/meminfo的”Cached”中。

- Cached是”Mapped”的超集，就是说它不仅包括mapped，也包括unmapped的页面，当一个文件不再与进程关联之后，原来在page cache中的页面并不会立即回收，仍然被计入Cached，还留在LRU中，但是 Mapped 统计值会减小。【ummaped = (Cached – Mapped)】
- Cached包含tmpfs中的文件，POSIX/SysV shared memory，以及shared anonymous mmap。 
  注：POSIX/SysV shared memory和shared anonymous mmap在内核中都是基于tmpfs实现的，参见： 
  <https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt>

“Cached”和”SwapCached”两个统计值是互不重叠的，源代码参见下一节。所以，Shared memory和tmpfs在不发生swap-out的时候属于”Cached”，而在swap-out/swap-in的过程中会被加进swap cache中、属于”SwapCached”，一旦进了”SwapCached”，就不再属于”Cached”了。

##### 2.8 SwapCached

我们说过，匿名页(anonymous pages)要用到交换区，而shared memory和tmpfs虽然未统计在AnonPages里，但它们背后没有硬盘文件，所以也是需要交换区的。也就是说需要用到交换区的内存包括：”AnonPages”和”Shmem”，我们姑且把它们统称为匿名页好了。

交换区可以包括一个或多个交换区设备（裸盘、逻辑卷、文件都可以充当交换区设备），每一个交换区设备都对应自己的swap cache，可以把swap cache理解为交换区设备的”page cache”：page cache对应的是一个个文件，swap cache对应的是一个个交换区设备，kernel管理swap cache与管理page cache一样，用的都是radix-tree，唯一的区别是：page cache与文件的对应关系在打开文件时就确定了，而一个匿名页只有在即将被swap-out的时候才决定它会被放到哪一个交换区设备，即匿名页与swap cache的对应关系在即将被swap-out时才确立。

并不是每一个匿名页都在swap cache中，只有以下情形之一的匿名页才在：

- 匿名页即将被swap-out时会先被放进swap cache，但通常只存在很短暂的时间，因为紧接着在pageout完成之后它就会从swap cache中删除，毕竟swap-out的目的就是为了腾出空闲内存； 
  【注：参见mm/vmscan.c: shrink_page_list()，它调用的add_to_swap()会把swap cache页面标记成dirty，然后它调用try_to_unmap()将页面对应的page table mapping都删除，再调用pageout()回写dirty page，最后try_to_free_swap()会把该页从swap cache中删除。】
- 曾经被swap-out现在又被swap-in的匿名页会在swap cache中，直到页面中的内容发生变化、或者原来用过的交换区空间被回收为止。 
  【注：当匿名页的内容发生变化时会删除对应的swap cache，代码参见mm/swapfile.c: reuse_swap_page()。】

/proc/meminfo中的SwapCached背后的含义是：系统中有多少匿名页曾经被swap-out、现在又被swap-in并且swap-in之后页面中的内容一直没发生变化。也就是说，如果这些匿名页需要被swap-out的话，是无需进行I/O write操作的。

“SwapCached”不属于”Cached”，两者没有交叉。参见：

```
fs/proc/meminfo.c:static int meminfo_proc_show(struct seq_file *m, void *v){...        cached = global_page_state(NR_FILE_PAGES) -                        total_swapcache_pages() - i.bufferram;...}
```

“SwapCached”内存同时也在LRU中，还在”AnonPages”或”Shmem”中，它本身并不占用额外的内存。

##### 2.9 Mlocked

“Mlocked”统计的是被mlock()系统调用锁定的内存大小。被锁定的内存因为不能pageout/swapout，会从Active/Inactive LRU list移到Unevictable LRU list上。也就是说，当”Mlocked”增加时，”Unevictable”也同步增加，而”Active”或”Inactive”同时减小；当”Mlocked”减小的时候，”Unevictable”也同步减小，而”Active”或”Inactive”同时增加。

“Mlocked”并不是独立的内存空间，它与以下统计项重叠：LRU Unevictable，AnonPages，Shmem，Mapped等。

##### 2.10 Buffers

“Buffers”表示块设备(block device)所占用的缓存页，包括：直接读写块设备、以及文件系统元数据(metadata)比如SuperBlock所使用的缓存页。它与“Cached”的区别在于，”Cached”表示普通文件所占用的缓存页。参见我的另一篇文章<http://linuxperf.com/?p=32>

“Buffers”所占的内存同时也在LRU list中，被统计在Active(file)或Inactive(file)。

注：通过阅读源代码可知，块设备的读写操作涉及的缓存被纳入了LRU，以读操作为例，do_generic_file_read()函数通过 mapping->a_ops->readpage() 调用块设备底层的函数，并调用 add_to_page_cache_lru() 把缓存页加入到LRU list中。参见： 
filemap.c: do_generic_file_read > add_to_page_cache_lru

### 其它问题

#### DirectMap

/proc/meminfo中的DirectMap所统计的不是关于内存的使用，而是一个反映TLB效率的指标。TLB(Translation Lookaside Buffer)是位于CPU上的缓存，用于将内存的虚拟地址翻译成物理地址，由于TLB的大小有限，不能缓存的地址就需要访问内存里的page table来进行翻译，速度慢很多。为了尽可能地将地址放进TLB缓存，新的CPU硬件支持比4k更大的页面从而达到减少地址数量的目的， 比如2MB，4MB，甚至1GB的内存页，视不同的硬件而定。”DirectMap4k”表示映射为4kB的内存数量， “DirectMap2M”表示映射为2MB的内存数量，以此类推。所以DirectMap其实是一个反映TLB效率的指标。

#### Dirty pages到底有多少？

/proc/meminfo 中有一个Dirty统计值，但是它未能包括系统中全部的dirty pages，应该再加上另外两项：NFS_Unstable 和 Writeback，NFS_Unstable是发给NFS server但尚未写入硬盘的缓存页，Writeback是正准备回写硬盘的缓存页。即：

系统中全部dirty pages = ( Dirty + NFS_Unstable + Writeback )

注1：NFS_Unstable的内存被包含在Slab中，因为nfs request内存是调用kmem_cache_zalloc()申请的。

注2：anonymous pages不属于dirty pages。 
参见mm/vmscan.c: page_check_dirty_writeback() 
“Anonymous pages are not handled by flushers and must be written from reclaim context.”

#### 为什么 Active(anon)+Inactive(anon) 不等于AnonPages？

因为Shmem(即Shared memory & tmpfs) 被计入LRU Active/Inactive(anon)，但未计入 AnonPages。所以一个更合理的等式是：

Active(anon)+Inactive(anon) = AnonPages + Shmem

但是这个等式在某些情况下也不一定成立，因为：

- 如果shmem或anonymous pages被mlock的话，就不在Active(non)或Inactive(anon)里了，而是到了Unevictable里，以上等式就不平衡了；
- 当anonymous pages准备被swap-out时，分几个步骤：先被加进swap cache，再离开AnonPages，然后离开LRU Inactive(anon)，最后从swap cache中删除，这几个步骤之间会有间隔，而且有可能离开AnonPages就因某些情况而结束了，所以在某些时刻以上等式会不平衡。

注：参见mm/vmscan.c: shrink_page_list()： 
它调用的add_to_swap()会把swap cache页面标记成dirty，然后调用try_to_unmap()将页面对应的page table mapping都删除，再调用pageout()回写dirty page，最后try_to_free_swap()把该页从swap cache中删除。

#### 为什么 Active(file)+Inactive(file) 不等于Mapped？

1. 因为LRU Active(file)和Inactive(file)中不仅包含mapped页面，还包含unmapped页面；
2. Mapped中包含”Shmem”(即shared memory & tmpfs)，这部分内存被计入了LRU Active(anon)或Inactive(anon)、而不在Active(file)和Inactive(file)中。

#### 为什么 Active(file)+Inactive(file) 不等于 Cached？

1. 因为”Shmem”(即shared memory & tmpfs)包含在Cached中，而不在Active(file)和Inactive(file)中；
2. Active(file)和Inactive(file)还包含Buffers。
   - 如果不考虑mlock的话，一个更符合逻辑的等式是： 
     Active(file) + Inactive(file) + Shmem == Cached + Buffers
   - 如果有mlock的话，等式应该如下（mlock包括file和anon两部分，/proc/meminfo中并未分开统计，下面的mlock_file只是用来表意，实际并没有这个统计值）： 
     Active(file) + Inactive(file) + Shmem + mlock_file == Cached + Buffers

注： 
测试的结果以上等式通常都成立，但内存发生交换的时候以上等式有时不平衡，我猜可能是因为有些属于Shmem的内存swap-out的过程中离开Cached进入了Swapcached，但没有立即从swap cache删除、仍算在Shmem中的缘故。

#### Linux的内存都用到哪里去了？

尽管不可能精确统计Linux系统的内存，但大体了解还是可以的。

**kernel内存的统计方式应该比较明确，即** 
Slab+ VmallocUsed + PageTables + KernelStack + HardwareCorrupted + Bounce + X

- 注1：VmallocUsed其实不是我们感兴趣的，因为它还包括了VM_IOREMAP等并未消耗物理内存的IO地址映射空间，我们只关心VM_ALLOC操作，（参见1.2节），所以实际上应该统计/proc/vmallocinfo中的vmalloc记录，例如（此处单位是byte）：

```
# grep vmalloc /proc/vmallocinfo | awk '{total+=$2}; END {print total}'23375872
```

- 注2：kernel module的内存被包含在VmallocUsed中，见1.3节。
- 注3：X表示直接通过alloc_pages/__get_free_page分配的内存，没有在/proc/meminfo中统计，不知道有多少，就像个黑洞。

**用户进程的内存主要有三种统计口径：** 
\1. 围绕LRU进行统计 
(Active + Inactive + Unevictable) + (HugePages_Total * Hugepagesize)

1. 围绕Page Cache进行统计 
   当SwapCached为0的时候，用户进程的内存总计如下： 
   (Cached + AnonPages + Buffers) + (HugePages_Total * Hugepagesize) 
   当SwapCached不为0的时候，以上公式不成立，因为SwapCached可能会含有Shmem，而Shmem本来被含在Cached中，一旦swap-out就从Cached转移到了SwapCached，可是我们又不能把SwapCached加进上述公式中，因为SwapCached虽然不与Cached重叠却与AnonPages有重叠，它既可能含有Shared memory又可能含有Anonymous Pages。
2. 围绕RSS/PSS进行统计 
   把/proc/[1-9]*/smaps 中的 Pss 累加起来就是所有用户进程占用的内存，但是还没有包括Page Cache中unmapped部分、以及HugePages，所以公式如下： 
   ΣPss + (Cached – mapped) + Buffers + (HugePages_Total * Hugepagesize)

**所以系统内存的使用情况可以用以下公式表示：**

```
MemTotal = MemFree +(Slab+ VmallocUsed + PageTables + KernelStack + HardwareCorrupted + Bounce + X) + [Active + Inactive + Unevictable + (HugePages_Total * Hugepagesize)]MemTotal = MemFree +(Slab+ VmallocUsed + PageTables + KernelStack + HardwareCorrupted + Bounce + X)+ [Cached + AnonPages + Buffers + (HugePages_Total * Hugepagesize)]MemTotal = MemFree +(Slab+ VmallocUsed + PageTables + KernelStack + HardwareCorrupted + Bounce + X)+[ΣPss + (Cached – mapped) + Buffers + (HugePages_Total * Hugepagesize)]
```

## mdstat

```
root@lt:/proc# cat mdstat Personalities : unused devices: <none>
```

## misc

```
root@lt:/proc# cat misc  57 memory_bandwidth 58 network_throughput 59 network_latency 60 cpu_dma_latency227 mcelog236 device-mapper223 uinput  1 psaux200 tun237 loop-control228 hpet229 fuse 61 ecryptfs231 snapshot184 microcode 62 rfkill 63 vga_arbiter
```

## modules

系统已加载的内核模块列表

```
root@lt:/proc# cat modules xt_nat 16384 3 - Live 0xffffffffc04bd000xt_tcpudp 16384 9 - Live 0xffffffffc04b8000veth 16384 0 - Live 0xffffffffc04b3000ipt_MASQUERADE 16384 4 - Live 0xffffffffc04ae000nf_nat_masquerade_ipv4 16384 1 ipt_MASQUERADE, Live 0xffffffffc047c000xfrm_user 32768 1 - Live 0xffffffffc04a5000xfrm_algo 16384 1 xfrm_user, Live 0xffffffffc04a0000iptable_nat 16384 1 - Live 0xffffffffc049b000nf_conntrack_ipv4 16384 2 - Live 0xffffffffc0477000nf_defrag_ipv4 16384 1 nf_conntrack_ipv4, Live 0xffffffffc03bf000nf_nat_ipv4 16384 1 iptable_nat, Live 0xffffffffc033d000xt_addrtype 16384 2 - Live 0xffffffffc0338000iptable_filter 16384 1 - Live 0xffffffffc0333000ip_tables 28672 2 iptable_nat,iptable_filter, Live 0xffffffffc0493000xt_conntrack 16384 1 - Live 0xffffffffc048e000x_tables 36864 7 xt_nat,xt_tcpudp,ipt_MASQUERADE,xt_addrtype,iptable_filter,ip_tables,xt_conntrack, Live 0xffffffffc0484000nf_nat 24576 3 xt_nat,nf_nat_masquerade_ipv4,nf_nat_ipv4, Live 0xffffffffc03a3000nf_conntrack 106496 5 nf_nat_masquerade_ipv4,nf_conntrack_ipv4,nf_nat_ipv4,xt_conntrack,nf_nat, Live 0xffffffffc045c000br_netfilter 20480 0 - Live 0xffffffffc02a2000bridge 110592 1 br_netfilter, Live 0xffffffffc0440000stp 16384 1 bridge, Live 0xffffffffc0298000llc 16384 2 bridge,stp, Live 0xffffffffc0224000aufs 212992 256 - Live 0xffffffffc036e000rfcomm 69632 0 - Live 0xffffffffc03ad000bnep 20480 2 - Live 0xffffffffc02a8000bluetooth 491520 10 rfcomm,bnep, Live 0xffffffffc03c7000iosf_mbi 16384 0 - Live 0xffffffffc0321000crct10dif_pclmul 16384 0 - Live 0xffffffffc0326000crc32_pclmul 16384 0 - Live 0xffffffffc032e000ghash_clmulni_intel 16384 0 - Live 0xffffffffc0199000aesni_intel 172032 0 - Live 0xffffffffc0343000aes_x86_64 20480 1 aesni_intel, Live 0xffffffffc02c6000lrw 16384 1 aesni_intel, Live 0xffffffffc029d000gf128mul 16384 1 lrw, Live 0xffffffffc0253000glue_helper 16384 1 aesni_intel, Live 0xffffffffc022d000ablk_helper 16384 1 aesni_intel, Live 0xffffffffc0232000cryptd 20480 3 ghash_clmulni_intel,aesni_intel,ablk_helper, Live 0xffffffffc021e000serio_raw 16384 0 - Live 0xffffffffc0194000nfsd 294912 2 - Live 0xffffffffc02d8000i2c_piix4 24576 0 - Live 0xffffffffc02cd000auth_rpcgss 61440 1 nfsd, Live 0xffffffffc02b6000nfs_acl 16384 1 nfsd, Live 0xffffffffc02b10008250_fintek 16384 0 - Live 0xffffffffc0174000nfs 245760 0 - Live 0xffffffffc025b000lockd 94208 2 nfsd,nfs, Live 0xffffffffc023b000grace 16384 2 nfsd,lockd, Live 0xffffffffc016f000sunrpc 327680 6 nfsd,auth_rpcgss,nfs_acl,nfs,lockd, Live 0xffffffffc01cd000fscache 65536 1 nfs, Live 0xffffffffc01bc000mac_hid 16384 0 - Live 0xffffffffc00ac000parport_pc 32768 0 - Live 0xffffffffc0144000ppdev 20480 0 - Live 0xffffffffc013b000lp 20480 0 - Live 0xffffffffc00d0000parport 45056 3 parport_pc,ppdev,lp, Live 0xffffffffc012f000cirrus 28672 2 - Live 0xffffffffc01b4000syscopyarea 16384 1 cirrus, Live 0xffffffffc01ad000sysfillrect 16384 1 cirrus, Live 0xffffffffc01a6000sysimgblt 16384 1 cirrus, Live 0xffffffffc019f000ttm 94208 1 cirrus, Live 0xffffffffc017c000drm_kms_helper 126976 1 cirrus, Live 0xffffffffc014f000drm 344064 5 cirrus,ttm,drm_kms_helper, Live 0xffffffffc00da000psmouse 114688 0 - Live 0xffffffffc00b3000floppy 77824 0 - Live 0xffffffffc0098000pata_acpi 16384 0 - Live 0xffffffffc0090000
```

## mounts

记录挂载的文件系统，对于每个文件系统，都包括设备，安装点，文件系统类型，权限和两个标志。

```
root@lt:/proc# cat mounts sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0udev /dev devtmpfs rw,relatime,size=4073364k,nr_inodes=1018341,mode=755 0 0devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0tmpfs /run tmpfs rw,nosuid,noexec,relatime,size=817744k,mode=755 0 0/dev/disk/by-uuid/d30edfb8-0b5d-4359-8abd-638530e9ddd7 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0none /sys/fs/cgroup tmpfs rw,relatime,size=4k,mode=755 0 0none /sys/fs/fuse/connections fusectl rw,relatime 0 0none /sys/kernel/debug debugfs rw,relatime 0 0none /sys/kernel/security securityfs rw,relatime 0 0none /run/lock tmpfs rw,nosuid,nodev,noexec,relatime,size=5120k 0 0cgroup /sys/fs/cgroup/cpuset cgroup rw,relatime,cpuset 0 0none /run/shm tmpfs rw,nosuid,nodev,relatime 0 0cgroup /sys/fs/cgroup/cpu cgroup rw,relatime,cpu 0 0none /run/user tmpfs rw,nosuid,nodev,noexec,relatime,size=102400k,mode=755 0 0none /sys/fs/pstore pstore rw,relatime 0 0cgroup /sys/fs/cgroup/cpuacct cgroup rw,relatime,cpuacct 0 0cgroup /sys/fs/cgroup/memory cgroup rw,relatime,memory 0 0cgroup /sys/fs/cgroup/devices cgroup rw,relatime,devices 0 0cgroup /sys/fs/cgroup/freezer cgroup rw,relatime,freezer 0 0cgroup /sys/fs/cgroup/net_cls cgroup rw,relatime,net_cls 0 0cgroup /sys/fs/cgroup/blkio cgroup rw,relatime,blkio 0 0cgroup /sys/fs/cgroup/perf_event cgroup rw,relatime,perf_event 0 0cgroup /sys/fs/cgroup/net_prio cgroup rw,relatime,net_prio 0 0cgroup /sys/fs/cgroup/hugetlb cgroup rw,relatime,hugetlb 0 0systemd /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,name=systemd 0 0rpc_pipefs /run/rpc_pipefs rpc_pipefs rw,relatime 0 0/dev/disk/by-uuid/d30edfb8-0b5d-4359-8abd-638530e9ddd7 /var/lib/docker/aufs ext4 rw,relatime,errors=remount-ro,data=ordered 0 0none /var/lib/docker/aufs/mnt/a2bd2ad0483ee6b8e510a25816ef23f7e4c02f84315c822044a051e6d5c5aabb aufs rw,relatime,si=d89a2bc240af5572,dio,dirperm1 0 0none /var/lib/docker/aufs/mnt/4b4871dd9043f4a3f34957b1cdc7ab0422a3abc204ce36d21e82348133285f06 aufs rw,relatime,si=d89a2bc240af1572,dio,dirperm1 0 0gvfsd-fuse /run/user/111/gvfs fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=111,group_id=116 0 0shm /var/lib/docker/containers/ef43287086b5025a2e79da1d4fa199327748ddbe2ee688149eceae3d52837411/shm tmpfs rw,nosuid,nodev,noexec,relatime,size=65536k 0 0shm /var/lib/docker/containers/592dcf6d7cf8b7243283ac803f4e824dcb07cc0970db7d3a2d0b483b8af1777c/shm tmpfs rw,nosuid,nodev,noexec,relatime,size=65536k 0 0nsfs /run/docker/netns/3bfc466d2ce2 nsfs rw 0 0nsfs /run/docker/netns/21ff697e4399 nsfs rw 0 0gvfsd-fuse /run/user/0/gvfs fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=0,group_id=0 0 0
```

## net

一个目录包含了当前网络栈的信息，特别是/proc/net/nf_conntrack列出了存在的网络连接(对跟踪路由特别有用，因为iptables转发被用于重定向网络连接)。

使用 man 8 netstat 命令可以查看这些文件的说明

```
/proc/net/dev -- device information/proc/net/raw -- raw socket information/proc/net/tcp -- TCP socket information/proc/net/udp -- UDP socket information/proc/net/igmp -- IGMP multicast information/proc/net/unix -- Unix domain socket information/proc/net/ipx -- IPX socket information/proc/net/ax25 -- AX25 socket information/proc/net/appletalk -- DDP (appletalk) socket information/proc/net/nr -- NET/ROM socket information/proc/net/route -- IP routing information/proc/net/ax25_route -- AX25 routing information/proc/net/ipx_route -- IPX routing information/proc/net/nr_nodes -- NET/ROM nodelist/proc/net/nr_neigh -- NET/ROM neighbours/proc/net/ip_masquerade -- masqueraded connections/proc/net/snmp -- statistics
```

## partitions

一个设备号、尺寸与/dev名的列表，内核用于辨别已存在的硬盘分区。

```
root@lt:/proc# cat partitions major minor  #blocks  name   8        0  209715200 sda   8        1  192937984 sda1   8        2          1 sda2   8        5   16774144 sda5
```

## pci

系统知道的所有PCI设备的列表。

## rtc

包含时钟信息的文件。

## scsi

给出任何通过SCSI或RAID控制器挂接的设备的信息。 
\1. scsi: kernel 知道的所有 scsi 设备的列表 
\2. drivername: 各种scsi驱动器品牌名称

## self

指向执行命令的进程（该进程在 /proc 文件系统中对应的目录）。

## slabinfo

系统的 slab(Linux内核频繁使用的对象) 信息

```
root@lt:/proc# cat slabinfo slabinfo - version: 2.1# name            <active_objs> <num_objs> <objsize> <objperslab> <pagesperslab> : tunables <limit> <batchcount> <sharedfactor> : slabdata <active_slabs> <num_slabs> <sharedavail>nf_conntrack_ffff880226a20000    200    200    320   25    2 : tunables    0    0    0 : slabdata      8      8      0nf_conntrack_ffff8802269d0000    200    200    320   25    2 : tunables    0    0    0 : slabdata      8      8      0nf_conntrack_ffffffff81cda040   1304   1425    320   25    2 : tunables    0    0    0 : slabdata     57     57      0au_finfo             378    378    192   21    1 : tunables    0    0    0 : slabdata     18     18      0au_icntnr           2604   2604    768   21    4 : tunables    0    0    0 : slabdata    124    124      0au_dinfo            2752   2752    128   32    1 : tunables    0    0    0 : slabdata     86     86      0nfsd4_openowners       0      0    440   37    4 : tunables    0    0    0 : slabdata      0      0      0nfs_direct_cache       0      0    208   39    2 : tunables    0    0    0 : slabdata      0      0      0nfs_commit_data       23     23    704   23    4 : tunables    0    0    0 : slabdata      1      1      0nfs_inode_cache        0      0   1024   32    8 : tunables    0    0    0 : slabdata      0      0      0rpc_inode_cache       50     50    640   25    4 : tunables    0    0    0 : slabdata      2      2      0fscache_cookie_jar     46     46     88   46    1 : tunables    0    0    0 : slabdata      1      1      0....
```

## swaps

活动交换分区的信息，如尺寸、优先级等。

```
root@lt:/proc# cat swaps Filename                Type        Size    Used    Priority/dev/sda5                               partition   16774140    2928    -1
```

## sys

动态可配置的内核选项. 其下的目录对应与内核区域，包含了可读与可写的虚拟文件（virtual file）.

- debug

- dev

- fs

- kernel

   

  ​

  - core_pattern
  - domainname
  - file-max
  - file-nr
  - hostname
  - inode-max
  - inode-nr
  - osrelease
  - ostype
  - panic
  - real-root-dev
  - securelevel
  - version

- net

- proc

- sunrpc

- vm

## sysvipc

包括共享内存与进程间通信 (IPC)信息

## tty

包含当前终端信息; /proc/tty/driver是可利用的tty类型列表，其中的每一个是该类型的可用设备列表。

## uptime

内核启动后经过的秒数与idle模式的秒数

```
root@lt:/proc# cat uptime 1912808.25 15230268.58
```

## version

包含Linux内核版本，发布号（distribution number）, 编译内核的gcc版本，其他相关的版本

```
root@lt:/proc# cat versionLinux version 3.19.0-39-generic (buildd@lcy01-26) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) ) #44~14.04.1-Ubuntu SMP Wed Dec 2 10:00:35 UTC 2015
```

## cpuinfo

CPU 的架构信息

包含CPU信息, 诸如厂商（vendor），型号 (family, model，model names), 速度, 缓存大小, 逻辑核数 , 物理核数, CPU flags，以及BogoMips.对于多核CPU，/proc/cpuinfo的逻辑核数”siblings”与物理核数”cpu cores”分别表示:

```
"siblings" = (HT per CPU package) * (# of cores per CPU package)"cpu cores" = (# of cores per CPU package)
```

CPU package是指单独封装的一颗CPU。这可以区分超线程与双核，例如每颗CPU超线程数量为siblings / CPU cores. 如果二者的值相等，则CPU不支持超线程.[4]

## stat

/proc/stat 文件中记录了系统级别的 CPU 使用情况，详情参考 man 5 proc 中的 /proc/stat 一节相关内容

```
root@lt:/proc# cat /proc/stat cpu  974810 12318 376047 1174749637 23034 59 80434 328987 0 0cpu0 165506 13 55664 146774966 2695 0 1886 32230 0 0cpu1 104088 1232 47239 146858688 1902 0 1006 29984 0 0cpu2 90905 3297 26879 146930266 2168 0 309 17637 0 0cpu3 440120 2457 164169 146090941 5390 58 75840 213276 0 0cpu4 60158 1351 27066 146973208 3577 1 675 16088 0 0cpu5 48447 1013 24909 147017455 3634 0 274 8732 0 0cpu6 36383 669 15782 147045688 1699 0 270 6765 0 0cpu7 29200 2283 14336 147058422 1965 0 172 4272 0 0intr 141068717 5 10 0 0 279 0 3 0 1 0 0 0 144 0 972593 0 0 0 0 0 0 0 0 0 0 46007582 375 0 2132 6 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0ctxt 276128188btime 1523838836processes 175019procs_running 1procs_blocked 0softirq 153770620 1 24114791 23 95015675 972664 0 38 20016677 34301 13616450
```

第一行是总的 CPU 使用统计，cpuN 是对应 core 的统计，衡量单位为 USER_HZ，大多数架构下是 100/秒。

CPU 参数说明(所有的数据都是从系统启动开始累计到当前时刻的数据)

| index | 参数       | 示例值     | 解析                                                         |
| ----- | ---------- | ---------- | ------------------------------------------------------------ |
| 2     | user       | 974810     | 处于用户态的运行时间，不包含 nice值为负进程。                |
| 3     | nice       | 12318      | nice值为负的进程所占用的CPU时间                              |
| 4     | system     | 376047     | 处于核心态的运行时间                                         |
| 5     | idle       | 1174749637 | 除硬盘IO等待时间以外其它等待时间                             |
| 6     | iowait     | 23034      | 除IO等待时间以外的其它等待时间iowait (12256) 从系统启动开始累计到当前时刻，IO等待时间(since 2.5.41) |
| 7     | irq        | 59         | 硬中断时间(since 2.6.0-test4)                                |
| 8     | softirq    | 80434      | 软中断时间                                                   |
| 9     | steal      | 328987     | 虚拟环境下运行其它操作系统的时间                             |
| 10    | guest      | 0          | 在 Linux 内核控制下运行客户操作系统的虚拟 CPU 的时间         |
| 11    | guest_nice | 0          | 运行 niced 客户机的时间（在 Linux 内核控制下运行客户操作系统的虚拟 CPU 的时间） |

| 参数                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| intr                    | 这行给出中断的信息，第一个为自系统启动以来，发生的所有的中断的次数；然后每个数对应一个特定的中断自系统启动以来所发生的次数。 |
| ctxt                    | 给出了自系统启动以来CPU发生的上下文交换的次数。              |
| btime                   | 给出了从系统启动到现在为止的时间，单位为秒。                 |
| processes (total_forks) | 自系统启动以来所创建的任务的个数目。                         |
| procs_running           | 当前运行队列的任务的数目。                                   |
| procs_blocked           | 当前被阻塞的任务的数目。                                     |

CPU 活动时间为： 
totalCPUTime = user + nice + system + idle + iowait + irq + softirq + stealstolen + guest

USER_HZ

| ARCH             | 说明                                |
| ---------------- | ----------------------------------- |
| ia64             | configurable (it matches CONFIG_HZ) |
| alpha            | 1024                                |
| 其它（x86/arm…） | hardcoded to 100                    |

## sysrq-trigger

SysRq 经常被称为 Magic System Request，可以用来在系统挂起等极端环境下搜集系统信息并对系统进行控制。

它被定义为一系列按键组合，在 2.6 版本以后的 kernel 中，还额外提供了 /proc/sysrq-trigger 接口。 
模拟 kernel panic

### 启用

#### 确认内核支持

```
# grep “ CONFIG_MAGIC_SYSRQ ” /boot/config-`uname – r`  CONFIG_MAGIC_SYSRQ=y
```

#### sysctl 启用功能

```
# sysctl -w kernel.sysrq=1 kernel.sysrq = 1 
```

通过把” kernel.sysrq = 1 ”设置到 /etc/sysctl.conf 中，可以使 SysRq 在下次系统重启后仍然生效。

#### 确认生效

```
# cat /proc/sys/kernel/sysrq 1
```

### 查看输出

#### 本地终端

SysRq 默认会根据 console_loglevel 输出到本地终端。只要 console_loglevel 大于 default_message_loglevel，SysRq 信息就会输出到本地控制台终端。它在测试的时候都好用，但一到关键时刻，系统挂起无法切换，大量输出造成滚屏，以及信息无法记录等问题随之而来。

#### 输出到 syslog

根据 syslog 的默认配置，SysRq 默认会记录到 /var/log/messages（ubuntu 14.04 下是 kern.log），并且这里记录的信息与 console_loglevel 无关，基本是完整的。但是由于负责记录日志的 syslogd 本身也是一个用户进程，在执行后面即将介绍的 SysRq-E, SysRq-I 时也会被终结，这就意味着 syslog 记录的信息在一定情况下将不再完整。同时由于系统挂起时查看 syslog 日志本身就是一件难上加难的事，这里记录的信息往往只能用在系统恢复过后的故障分析，对故障发生时的实时诊断并没有太大的帮助。

#### 通过 netconsole 输出

利用 netconsole 功能，可以获得一个通过远程 syslog 服务器输出的显示终端，服务器的 SysRq 输出将被远程的 syslog 服务器捕获。在服务器挂起，无法查看 syslog 日志，同时也无法切换到本地控制台终端的时候，这种形式的输出就显得举足轻重。与本地终端类似，只要 console_loglevel 大于 default_message_loglevel，SysRq 信息就会通过 netconsole 输出到远程。它在大多数情况都生效，哪怕是内核网络部分出现问题。因为 netconsole 的网络发送部分是独立存在的，并不依赖于网卡驱动程序。

#### 输出到串口终端

要想通过串口获取 SysRq 输出，首先需要在 grub 的 kernel 行添加类似 ” console=ttyS0,115200 ” 的串口输出配置，重启服务器以启用内核串口输出。然后从另一台主机上用串口线连接服务器，并用 minicom 等程序捕获其输出。这是一种通常的使用方式。然而利用 Serial over IP 产品，管理员无需现身嘈杂的机房就能通过网络获得服务器的串口输出，查看并截取字符形式的输出。这是相对现代的使用方式。通过这两种方式，我们都可以方便的获取到 SysRq 在串口上输出。

### 功能键组合

关于 sysrq 的使用方式，主要有两种，一是快捷键，alt+syrq+[key]，这里的 [key] 指的是键盘上的按键；二是通过 /proc/sysrq 接口，直接将对应的 [key] 发送到 /prc/sysrq-trigger 文件即可，具体的信息可以通过 /var/log/kern.log 查看。

#### 安全重启系统

到目前为止，我们可见到的大多数 SysRq 推荐用法都是系统挂起后的安全重启，用此方法来避免数据丢失。这个 SysRq 序列是 R-E-I-S-U-B 。要知道，该序列早在 SysRq 首次于 Linux 实现的 2.1.43 内核中就存在了。它基本等价于 reboot 命令，会依次停止系统上运行的进程，回写磁盘缓冲区，再安全的重启系统。需要注意的是，E 会向除 init 以外所有进程发送可捕获的 SIGTERM 信号，这就意味着程序可能需要一定时间来进行结束进程前的善后处理，视系统负载和任务数量，这个时间可能会达到几十秒。 I 发送的是不可捕获的 SIGKILL 信号，相对而言没有更多的延迟。同时，S 和 U 这两个动作均与磁盘相关。当系统具有一定负载时，这两个动作均不会立即完成，而是需要一定的时间，通常为几秒钟。所以，R-E-I-S-U-B 这个序列的推荐使用方式是：R – 1 秒 – E – 30 秒 – I – 10 秒 – S – 5 秒 – U – 5 秒 – B，而不是一气呵成地按下这六个键，试想一次正常的 reboot 命令也不是在一瞬间完成的吧。

| 快捷键 | 说明                                    |
| ------ | --------------------------------------- |
| R      | 把键盘设置为 ASCII 模式                 |
| E      | 向除 init 以外所有进程发送 SIGTERM 信号 |
| I      | 向除 init 以外所有进程发送 SIGKILL 信号 |
| S      | 磁盘缓冲区同步                          |
| U      | 重新挂载为只读模式                      |
| B      | 立即重启系统                            |

#### 恢复系统挂起

K 和 F 仅结束符合特定条件的进程。 K 只结束与当前控制台相关的进程组。 K 代表 saK，saK 的全称为 Secure Access Key 。

| 快捷键 | 说明                           |
| ------ | ------------------------------ |
| E      | 向所有进程发送 SIGTERM 信号    |
| I      | 向所有进程发送 SIGKILL 信号    |
| K      | 结束与当前控制台相关的全部进程 |
| F      | 人为触发 OOM Killer            |

#### 获取系统信息

| 快捷键 | 说明                    |
| ------ | ----------------------- |
| M      | 打印内存使用信息        |
| P      | 打印当前 CPU 寄存器信息 |
| T      | 打印进程列表            |
| W      | 打印 CPU 信息           |

#### 其它功能键组合

| 快捷键 | 说明                   |
| ------ | ---------------------- |
| H      | 帮助                   |
| C      | 触发 Crashdump         |
| N      | 降低实时任务运行优化级 |
| O      | 关机                   |

### 示例

触发 Crashdump

```
ehco c > /proc/sysrq-trigger
```

### 参考

[利用 SysRq 键排除和诊断系统故障](https://www.ibm.com/developerworks/cn/linux/l-cn-sysrq/index.html)

## $PID/

$PID 表示进程号，/proc/$PID/ 目录下保存的是该进程的状态信息，包括启动命令，内存，CPU，用户，网络，IO 等内容。

可以通过 man 5 proc 查看详细说明

### autogroup

task group 
参考 <http://man7.org/linux/man-pages/man7/sched.7.html>

### auxv

包含传递给进程的ELF解释器信息，格式是每一项都是一个unsigned long长度的ID加上一个unsigned long长度的值。最后一项以连续的两个0x00开头。

```
root@lt:/proc/6606# hexdump -x auxv 0000000    0021    0000    0000    0000    c000    c75f    7ffe    00000000010    0010    0000    0000    0000    fbff    078b    0000    00000000020    0006    0000    0000    0000    1000    0000    0000    00000000030    0011    0000    0000    0000    0064    0000    0000    00000000040    0003    0000    0000    0000    0040    0040    0000    00000000050    0004    0000    0000    0000    0038    0000    0000    00000000060    0005    0000    0000    0000    000a    0000    0000    00000000070    0007    0000    0000    0000    e000    5efa    7f91    00000000080    0008    0000    0000    0000    0000    0000    0000    00000000090    0009    0000    0000    0000    a7c0    0045    0000    000000000a0    000b    0000    0000    0000    0000    0000    0000    000000000b0    000c    0000    0000    0000    0000    0000    0000    000000000c0    000d    0000    0000    0000    0000    0000    0000    000000000d0    000e    0000    0000    0000    0000    0000    0000    000000000e0    0017    0000    0000    0000    0000    0000    0000    000000000f0    0019    0000    0000    0000    68d9    c75f    7ffe    00000000100    001f    0000    0000    0000    8fef    c75f    7ffe    00000000110    000f    0000    0000    0000    68e9    c75f    7ffe    00000000120    0000    0000    0000    0000    0000    0000    0000    00000000130
```

### comm/cmdline/exe

- comm: 进程的命令名(不包括启动时的姿势)
- cmdline: 启动进程时执行的命令（包括可执行文件路径和参数）,如果这个进程是zombie进程，则这个文件没有任何内容。
- exe: 最初的可执行文件的符号链接, 如果它还存在的话。

示例： 
通过 “./sysmon” 启动一个进程的话，那么 comm 的内容为 “sysmon”，而 cmdline 的内容为 “./sysmon”

### cwd

进程的当前工作目录的软链接

### cgroup

进程的 cgroup 设置

```
# cat cgroup 12:name=systemd:/user/0.user/14.session11:hugetlb:/user/0.user/14.session10:net_prio:/user/0.user/14.session9:perf_event:/user/0.user/14.session8:blkio:/user/0.user/14.session7:net_cls:/user/0.user/14.session6:freezer:/user/0.user/14.session5:devices:/user/0.user/14.session4:memory:/user/0.user/14.session3:cpuacct:/user/0.user/14.session2:cpu:/user/0.user/14.session1:cpuset:/
```

### cpuset

man 7 cpuset 
进程 cpuset 目录相对于 cpuset 根文件系统的路径

### coredump_filter

man 5 core 
从 2.6.23 内核开始，该文件用来控制在进程发生 crash 的时候，具体需要 dump 哪些内存的信息。 
该文件的值是内存映射类型的比特掩码，如果对应的比特位被设置为 1，则该类型的内存映射的内容将会被 dump 出来。以下是各个比特位对应的类型：

| bit  | description                              |
| ---- | ---------------------------------------- |
| 0    | 匿名私有映射                             |
| 1    | 匿名共享映射                             |
| 2    | file-backed private mappings             |
| 3    | file-backed shared mappings              |
| 4    | ELF headers(since Linux 2.6.24)          |
| 5    | private huge pages. (since Linux 2.6.28) |
| 6    | shared huge pages. (since Linux 2.6.28)  |
| 7    | private DAX pages. (since Linux 4.4)     |
| 8    | shared DAX pages. (since Linux 4.4)      |

默认情况下，只会设置 0, 1, 4(内核启用 CONFIG_CORE_DUMP_DEFAULT_ELF_HEADERS)， 5 这几个比特位。其默认值可以通过 coredump_filter 启动选项来修改。

e.g.

```
$ echo 0x7 > /proc/self/coredump_filter$ ./some_program
```

### environ

影响进程的环境变量的名字和值.

### fd

进程打开的每一个文件，都会对应这个目录里的的一个文件。0-标准输入，1-标准输出，2-标准错误输出。

### fdinfo

进程打开的文件描述符的状态信息，每个文件描述符对应该目录下的一个文件，以文件描述符命名。

### io

进程的 io 统计数据

```
# cat iorchar: 3580671745wchar: 1328893447syscr: 9689523syscw: 419361read_bytes: 0write_bytes: 0cancelled_write_bytes: 0
```

| 项目                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| rchar                 | 读取的字符数。这个数字只表示进程引起的读取操作，是对进程的 read 和 pread 操作的简单统计，无论是否实际从物理磁盘读取的都统计在内（比如说从终端的读取也包含在内） |
| wchar                 | 写入的字符数，详细说明同上。                                 |
| syscr                 | 读取系统调用。试图统计 IO 操作数（read/pread）               |
| syscw                 | 写入系统调用。试图统计 IO 操作数（write/pwrite）             |
| read_bytes            | 试图统计实际从存储层读取的字节数。对于块设备文件系统来说，这个数字才是精确的统计。 |
| write_bytes           | 试图统计实际写入到存储层的字节数                             |
| cancelled_write_bytes | 统计进程调用 write/pwrite 之后，但数据还没有实际写入就被中断时，没有写入的字节数，并不是一个精确值。 |

### latency

显示哪些代码造成的延时比较大（要使用这个feature，需要执行“echo 1 > /proc/sys/kernel/latencytop”）。

```
# cat /proc/2948/latencyLatency Top version : v0.130667 10650491 4891 poll_schedule_timeout do_sys_poll SyS_poll system_call_fastpath 0x7f636573dc1d8 105 44 futex_wait_queue_me futex_wait do_futex SyS_futex system_call_fastpath 0x7f6365a167bc
```

每一行前三个数字分别是后面代码执行的次数，总共执行延迟时间（单位是微秒）和最长执行延迟时间（单位是微秒），后面则是代码完整的调用栈。

### limits

进程的 limits 配置，包括最大打开文件数量，最高优先级，最大堆栈数，最大进程数等等。

Soft Limit表示kernel设置给资源的值，Hard Limit表示Soft Limit的上限，而Units则为计量单元。

```
# cat limits Limit                     Soft Limit           Hard Limit           Units     Max cpu time              unlimited            unlimited            seconds   Max file size             unlimited            unlimited            bytes     Max data size             unlimited            unlimited            bytes     Max stack size            8388608              unlimited            bytes     Max core file size        0                    unlimited            bytes     Max resident set          unlimited            unlimited            bytes     Max processes             31823                31823                processes Max open files            1024                 4096                 files     Max locked memory         65536                65536                bytes     Max address space         unlimited            unlimited            bytes     Max file locks            unlimited            unlimited            locks     Max pending signals       31823                31823                signals   Max msgqueue size         819200               819200               bytes     Max nice priority         0                    0                    Max realtime priority     0                    0                    Max realtime timeout      unlimited            unlimited            us 
```

### maps

包含进程的内存区域映射信息，记录了进程的虚拟内存的详细使用情况。其中注意的一点是[stack:]是线程的堆栈信息，对应于/proc/[pid]/task/[tid]/路径。

- 地址: 表示一段虚拟内存地址范围
- 权限: 虚拟内存的权限，r=读, w=写, x=可执行, s=共享, p=私有
- 偏移量:
- 设备: 映像文件所在的磁盘的主设备号和次设备号
- 节点: 映像文件的的磁盘节点（inode）
- 路径: 文件/库/堆栈的路径

```
7fcb8a41f000-7fcb8a42a000 r-xp 00000000 08:01 2887874                    /lib/x86_64-linux-gnu/libnss_files-2.19.so7fcb8a42a000-7fcb8a629000 ---p 0000b000 08:01 2887874                    /lib/x86_64-linux-gnu/libnss_files-2.19.so7fcb8a629000-7fcb8a62a000 r--p 0000a000 08:01 2887874                    /lib/x86_64-linux-gnu/libnss_files-2.19.so7fcb8a62a000-7fcb8a62b000 rw-p 0000b000 08:01 2887874                    /lib/x86_64-linux-gnu/libnss_files-2.19.so7fcb8a62b000-7fcb8a636000 r-xp 00000000 08:01 2887884                    /lib/x86_64-linux-gnu/libnss_nis-2.19.so7fcb8a636000-7fcb8a835000 ---p 0000b000 08:01 2887884                    /lib/x86_64-linux-gnu/libnss_nis-2.19.so7fcb8a835000-7fcb8a836000 r--p 0000a000 08:01 2887884                    /lib/x86_64-linux-gnu/libnss_nis-2.19.so7fcb8a836000-7fcb8a837000 rw-p 0000b000 08:01 2887884                    /lib/x86_64-linux-gnu/libnss_nis-2.19.so7fcb8a837000-7fcb8a84e000 r-xp 00000000 08:01 2887868                    /lib/x86_64-linux-gnu/libnsl-2.19.so7fcb8a84e000-7fcb8aa4d000 ---p 00017000 08:01 2887868                    /lib/x86_64-linux-gnu/libnsl-2.19.so7fcb8aa4d000-7fcb8aa4e000 r--p 00016000 08:01 2887868                    /lib/x86_64-linux-gnu/libnsl-2.19.so
```

### map_files

映射的文件，即进程将使用到的二进制和动态库等，映射到该进程的内存虚拟地址的地址段信息。

```
root@lt:/proc/6606# ll -h map_files/total 0dr-x------ 2 root root  0 May 25 14:24 ./dr-xr-xr-x 9 root root  0 May 25 14:07 ../lr-------- 1 root root 64 May 25 17:58 400000-82e000 -> /vob/golang/src/github.com/Lt0/sysmon/sysmon*lr-------- 1 root root 64 May 25 17:58 7f915e9cb000-7f915eb86000 -> /lib/x86_64-linux-gnu/libc-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915eb86000-7f915ed85000 -> /lib/x86_64-linux-gnu/libc-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915ed85000-7f915ed89000 -> /lib/x86_64-linux-gnu/libc-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915ed89000-7f915ed8b000 -> /lib/x86_64-linux-gnu/libc-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915ed90000-7f915eda9000 -> /lib/x86_64-linux-gnu/libpthread-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915eda9000-7f915efa8000 -> /lib/x86_64-linux-gnu/libpthread-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915efa8000-7f915efa9000 -> /lib/x86_64-linux-gnu/libpthread-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915efa9000-7f915efaa000 -> /lib/x86_64-linux-gnu/libpthread-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915efae000-7f915efd1000 -> /lib/x86_64-linux-gnu/ld-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915f1d0000-7f915f1d1000 -> /lib/x86_64-linux-gnu/ld-2.19.so*lr-------- 1 root root 64 May 25 17:58 7f915f1d1000-7f915f1d2000 -> /lib/x86_64-linux-gnu/ld-2.19.so*lr-------- 1 root root 64 May 25 17:58 82e000-b55000 -> /vob/golang/src/github.com/Lt0/sysmon/sysmon*lr-------- 1 root root 64 May 25 17:58 b55000-b92000 -> /vob/golang/src/github.com/Lt0/sysmon/sysmon*root@lt:/proc/6606# 
```

（1）mount ID：mount的唯一标识符（可以在umount（2）之后重用）。

​               （2）父ID：父安装的ID（或安装树顶部的self）。

​               （3）major：minor：文件系统上文件的st_dev值（参见stat（2））。

​               （4）root：文件系统中mount的根目录。

​               （5）挂载点：相对于进程根的挂载点。

​               （6）mount选项：per-mount选项。

​               （7）可选字段：“tag [：value]”形式的零个或多个字段。

​               （8）分隔符：标记可选字段的结尾。

​               （9）filesystem type：文件系统的名称，格式为“type [.subtype]”。

​               （10）mount source：特定于文件系统的信息或“none”。

​               （11）超级选项：每超级块选项。 

### mem

一个二进制图像(image)表示进程的虚拟内存, 只能通过 ptrace 化进程访问.

### mountinfo

该文件包含挂载点的信息，其每一行内容如下：

```
36 35 98:0 /mnt1 /mnt2 rw,noatime master:1 - ext3 /dev/root rw,errors=continue(1)(2)(3)   (4)   (5)      (6)      (7)   (8) (9)   (10)         (11)
```

1. mount ID: moun t的唯一标识符（可以在umount（2）之后重用）。
2. 父 ID: 上层挂载点的 ID（或跟挂载点本身）。
3. major：minor：文件系统上文件的st_dev值（参见stat（2））。
4. root：文件系统中 mount 的根目录。
5. 挂载点：相对于进程根的挂载点。
6. mount选项：per-mount选项。
7. 可选字段：“tag [：value]” 形式的零个或多个字段。
8. 分隔符：标记可选字段的结尾。
9. filesystem type：文件系统的名称，格式为 “type [.subtype]”。
10. mount source：特定于文件系统的信息或 “none”。
11. super 选项：per-superblock。

### mounts

当前进程的 mount 命名空间中的所有文件系统的列表。 记录格式同 fstab（5）。

从内核版本2.6.15开始，该文件是可轮询的：打开文件进行读取后，对该文件进行更改（即文件系统挂载或卸载）会导致select（2）将文件描述符标记为可读，并且poll（2）和epoll_wait（2）将文件标记为具有错误条件。 
详细内容参考 namespaces(7)。

### mountstats

该文件导出有关进程的 mount 命名空间中的挂载点的信息（统计信息，配置信息）。 
文件中的行内容说明如下：

```
device /dev/sda7 mounted on /home with fstype ext3 [statistics](       1      )            ( 2 )             (3 ) (4)
```

1. 设备名称（如果没有相应的设备，则为“nodevice”）。
2. 文件系统中的挂载点。
3. 文件系统类型。
4. 可选的统计信息和配置信息。 目前（如在Linux 2.6.26中），只有NFS文件系统通过此字段导出信息。

此文件只能由进程所有者读取。

详细内容参考 namespaces(7)。

### net

该目录是进程所在的网络命名空间内的网络信息。

### numa_maps

详细内容参考 man 7 numa

该文件用于显示进程的 NUMA 内存策略和内存分配情况。 
每一行显示进程的一段内存的 NUMA 内存状态，第一个字段是内存虚拟地址的起始地址，第二个字段是内存策略，具体的字段说明如下：

| 字段              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| <首字段>          | 内存范围的起始地址                                           |
| <第二字段>        | 对该范围内存生效的内存策略，请注意，有效策略不一定是该内存范围的进程安装的策略。 具体而言，如果进程为该范围安装了“default”策略，则该范围的有效策略将是进程策略，其可能是也可能不是“default”。 |
| N=                | 各个 NUMA 节点上分配的内存页数。                             |
| file=             | 内存范围所对应的文件。 如果文件被映                          |
| heap              | 由堆使用的内存                                               |
| stack             | 由栈使用的内存                                               |
| huge              | Huge 内存范围，显示的是大页内存而不是常规页。                |
| anon=             | 该范围内的匿名内存页数                                       |
| dirty=            | 需要回写磁盘的页数                                           |
| mapped=           | 映射的内存页面总数（如果和 dirty 以及 anon 的页数不同的话）。 |
| mapmax=           | 扫描期间遇到的最大mapcount（映射单个页面的进程数）。 这可以用作在 给定存储器范围内发生的共享程度的指示符。 |
| swapcache=        | 在交换设备上具有关联条目的页数。                             |
| active=           | 活动列表中的页数。仅当与此范围内的页数不同时，才会显示此字段。 这意味着存储 器范围中存在一些非活动页面，这些页面很快就会被交换器从内存中删除。 |
| writeback=        | 当前正在回写到磁盘的页数                                     |
| kernelpagesize_kB | 内核页大小                                                   |

内容示例：

```
7fb8f0c75000 default file=/lib/x86_64-linux-gnu/libselinux.so.1 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=47fb8f0c76000 default7fb8f0c78000 default file=/lib/x86_64-linux-gnu/ld-2.23.so\040(deleted) mapped=38 mapmax=46 N0=38 kernelpagesize_kB=4
```

### oom_xxx

```
root@lt:/proc/6606# cat oom_adj 0root@lt:/proc/6606# cat oom_score0root@lt:/proc/6606# cat oom_score_adj0root@lt:/proc/6606#
```

### personality

该只读文件公开进程的执行域，由 personality(2) 设置。 该值以十六进制表示法显示。 
访问该文件的权限由 ptrace 访问模式 PTRACE_MODE_ATTACH_FSCREDS 检查控制; 见ptrace（2）。

### root

该进程所能看到的根路径的符号链接。如果没有chroot监狱，那么进程的根路径是/.

### setgroups

man 7 user_namespaces

allow: 允许包含进程pid的用户命名空间中的进程使用setgroups（2）系统调用 
deny: 在该用户命名空间中不允许setgroups（2） 
请注意，如果没有设置 /proc/[pid]/gid_map，则无论该文件的值是什么，都不允许调用setgroups（2）

### sched

// 注下面的时间或时刻都是从 rq->clock 中获得的，而这个值是由update_rq_clock底层cpu来更新的。并且很多信息是需要内核配置 CONFIG_SCHEDSTATS 才有。 
大多数字段的计算在 sched.c及sched_fair.c 里，在这两个文件里搜索相应的字段就能得到相应的计算方法。

```
    "se.exec_start": "进程最近一次被调度的时间（起始时间点），单位 ns",     "se.vruntime": "虚拟运行时间，cfs 调度用",     "se.sum_exec_runtime": "处于运行状态的时长的总时长",     "se.statistics.sum_sleep_runtime": "处于 sleep 状态的总时长",     "se.avg_overlap": "",     "se.avg_wakeup": "当前进程最近一次调用 try_to_wake_up 为止的单次执行时间",     "se.avg_running": "平均单次执行时间",     "se.statistics.wait_start": "最近一次进入 wait 队列的时刻",     "se.statistics.sleep_start": "最近一次被设置为 S 状态的时刻",     "se.statistics.block_start": "最近一次被设置为 D 状态的时刻",     "se.statistics.sleep_max": "处于 S 状态时间最长的一次的时长",     "se.statistics.block_max": "处于 D 状态时间最长的一次的时长",     "se.statistics.exec_max": "最长单次执行时间",     "se.statistics.slice_max": "曾经获得时间片的最长时间（当 cpu load 过高时开始计算）",     "se.statistics.wait_max": "最长在就绪队列里的等待时间",     "se.statistics.wait_sum": "累计在就绪队列里的等待时间",     "se.statistics.wait_count": "累计等待次数 （被出队的次数） cfs使用，当被选中做下一个待运行进程时（set_next_entity），等待结束",     "se.statistics.iowait_sum": "IO 等待总时长",     "se.statistics.iowait_count": "IO 等待次数，io_schedule 调用次数",     "sched_info.bkl_count": "此进程大内核锁调用次数",     "se.nr_migrations": "需要迁移当前进程到其他 cpu 时累加此字段",     "se.statistics.nr_migrations_cold": "2.6.32 代码里赋值都是 0",     "se.statistics.nr_failed_migrations_affine": "进程设置了 cpu 亲和，进程迁移时检查失败的次数",     "se.statistics.nr_failed_migrations_running": "由于进程处于 runing 状态，不运行迁移的次数",     "se.statistics.nr_failed_migrations_hot": "当前进程因为是 cache hot 导致迁移失败的次数",     "se.statistics.nr_forced_migrations": "在当前进程 cache hot 下，由于负载均衡尝试多次失败，强行进行迁移的次数",     "se.statistics.nr_wakeups": "被唤醒的次数",     "se.statistics.nr_wakeups_sync": "同步唤醒次数，即 a 唤醒 b，a 立刻睡眠，b 被唤醒的次数 /* waker goes to sleep after wakup */",     "se.statistics.nr_wakeups_migrate": "被唤醒得到调度的当前 cpu，不是之前睡眠的 cpu 的次数",     "se.statistics.nr_wakeups_local": "被本地唤醒的次数",     "se.statistics.nr_wakeups_remote": "远程唤醒累计次数",     "se.statistics.nr_wakeups_affine": "考虑了任务的 cache 亲和性的唤醒次数",     "se.statistics.nr_wakeups_affine_attempts": "尝试进行考虑了任务的 cache 亲和性的唤醒次数",     "se.statistics.nr_wakeups_passive": "2.6.32 代码里赋值都是 0",     "se.statistics.nr_wakeups_idle": "2.6.32 代码里赋值都是 0",     "avg_atom": "进程平均切换耗时",     "avg_per_cpu": "如果本进程曾经被推或者拉到其他 cpu 上，则计算每个 cpu 上的平均耗时",     "nr_switches": "主动切换和被动切换的累计次数",     "nr_voluntary_switches": "主动切换次数",     "nr_involuntary_switches": "被动切换次数",     "se.load.weight": "负载均衡相关的权重",     "se.avg.load_sum": "",     "se.avg.util_sum": "",     "se.avg.load_avg": "",     "se.avg.util_avg": "",     "se.avg.last_update_time": "",     "policy": "进程属性，0 为非实时",     "prio": "动态优先级",     "clock-delta": "连续两次调用 cpu_clock() 之间的时间差，单位为 ns",  // https://stackoverflow.com/questions/15024601/what-is-clock-delta-in-proc-pid-sched    "mm->numa_scan_seq": "",     "numa_pages_migrated": "",     "numa_preferred_nid": "",     "total_numa_faults": "", 
```

### smaps

该文件记录了进程使用的库/堆栈/二进制文件等在虚拟内存中的地址段，以及这些库/堆栈/二进制文件等的详细内存使用信息。

kernel 必须配置 enable 了 CONFIG_PROC_PAGE_MONITOR 选项才能看到 smaps 信息。

首行内容（形如 00400000-0082e000 r-xp 00000000 08:01 2761308 ）

| 内容      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| File      | 映射的文件，中括号括起来的表示不是真正的文件，[stack:N] 表示线程号为 N 的线程对应的栈在内存中的映射情况 |
| StartAddr | “映射的起始虚拟地址”,                                        |
| EndAddr   | “映射的结束虚拟地址”,                                        |
| Perm      | “虚拟内存的权限，r=读, w=写, x=可执行, s=共享, p=私有”,      |
| Offset    | “偏移量,如果这段内存是从文件里映射过来的，则偏移量为这段内容在文件中的偏移量。如果不是从文件里面映射过来的则为 0”, |
| Dev       | “映像文件所在的磁盘的主设备号和次设备号”,                    |
| INode     | “映像文件的的磁盘节点（inode）”,                             |

| 选项           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| Size           | “虚拟内存大小”,                                              |
| Rss            | “实际占用的物理内存大小，使用的共享库所占用的内存会全部计算在里边, Rss = Shared_Clean + Shared_Dirty + Private_Clean + Private_Dirty “, |
| Pss            | “实际占用的物理内存大小，对于该进程用到的共享库，会根据使用该库的进程数量，按比例显示该进程占用的内存”, |
| SharedClean    | “没有更改过的共享页大小，当发生换页时不用写回磁盘”,          |
| SharedDirty    | “更改过的共享页大小，当发生换页时需要写回磁盘”,              |
| PrivateClea    | : “没有更改过的私有页大小，当发生换页时不用写回磁盘”,        |
| PrivateDirty   | “更改过的私有页大小，当发生换页时需要写回磁盘”,              |
| Referenced     | “”,                                                          |
| Anonymous      | “匿名页大小，包含 AnonHugePages”,                            |
| AnonHugePages  | “Transparent HugePages (THP)，THP 与 Hugepages 不是一回事，Hugepages 在 /proc/meminfo中是被独立统计的，而 AnonHugePages 是被包含在 AnonPages 之中的，与进程的 RSS/PSS 是有重叠的”, |
| SharedHugetlb  | “Shared Hugetlb”,                                            |
| PrivateHugetlb | “Private Hugetlb”,                                           |
| Swap           | “交换到 swap 分区的大小”,                                    |
| SwapPss        | “Swap Pss”,                                                  |
| KernelPageSize | “内核的内存页面大小”,                                        |
| MMUPageSize    | “体系结构 MMU 一个页面大小”,                                 |
| Locked         | “被 mlock() 系统调用锁定的内存大小。被锁定的内存因为不能 pageout/swapout，会从 Active/Inactive LRU list 移到 Unevictable LRU list 上。”, |
| VmFlags        | “VmFlags field represents the kernel flags associated with the particular virtual memory area in two letter encoded manner. The codes are the following: ” + smapsHdrVmFlags, |

VmFlags 说明

| 符号 | 说明                                    |
| ---- | --------------------------------------- |
| rd   | readable                                |
| wr   | writable                                |
| ex   | executable                              |
| sh   | shared                                  |
| mr   | may read                                |
| mw   | may write                               |
| me   | may execute                             |
| ms   | may share                               |
| gd   | stack segment grows down                |
| pf   | pure PFN range                          |
| dw   | disabled write to the mapped file       |
| lo   | pages are locked in memory              |
| io   | memory mapped I/O area                  |
| sr   | sequential read advise provided         |
| rr   | random read advise provided             |
| dc   | do not copy area on fork                |
| de   | do not expand area on remapping         |
| ac   | area is accountable                     |
| nr   | swap space is not reserved for the area |
| ht   | area uses huge tlb pages                |
| nl   | non-linear mapping                      |
| ar   | architecture specific flag              |
| dd   | do not include area into core dump      |
| sd   | soft-dirty flag                         |
| mm   | mixed map area                          |
| hg   | huge page advise flag                   |
| nh   | no-huge page advise flag                |
| mg   | mergeable advise flag                   |

示例:

```
00400000-0082e000 r-xp 00000000 08:01 2761308                            /vob/golang/src/github.com/Lt0/sysmon/sysmonSize:               4280 kBRss:                4024 kBPss:                4024 kBShared_Clean:          0 kBShared_Dirty:          0 kBPrivate_Clean:      4024 kBPrivate_Dirty:         0 kBReferenced:         4024 kBAnonymous:             0 kBAnonHugePages:         0 kBSwap:                  0 kBKernelPageSize:        4 kBMMUPageSize:           4 kBLocked:                0 kBVmFlags: rd ex mr mw me dw sd
```

### stack

当前进程的内核调用栈信息，只有内核编译时打开了CONFIG_STACKTRACE编译选项，才会生成这个文件。

```
root@lt:/proc/6606# cat stack [<ffffffff810ede4e>] futex_wait_queue_me+0xde/0x140[<ffffffff810ee9e2>] futex_wait+0x182/0x290[<ffffffff810f0ace>] do_futex+0xde/0x5d0[<ffffffff810f1031>] SyS_futex+0x71/0x150[<ffffffff817b738d>] system_call_fastpath+0x16/0x1b[<ffffffffffffffff>] 0xffffffffffffffffroot@lt:/proc/6606# 
```

### stat

```
# cat /proc/6606/stat6606 (sysmon) S 18299 18299 26897 34822 18299 1077960960 21127758 57743985 0 0 13051 32320 11661 34755 20 0 24 0 189531797 1024700416 5871 18446744073709551615 4194304 8576880 140732243338944 140732243338336 4566419 0 0 0 2143420159 18446744073709551615 0 0 17 4 0 0 0 0 0 11882496 12128640 41033728 140732243347465 140732243347474 140732243347474 140732243349487 0
```

| Index     | Field           | 示例值                                    | 说明                                                         |
| --------- | --------------- | ----------------------------------------- | ------------------------------------------------------------ |
| 1         | pid             | 6606                                      | 进程号                                                       |
| 2         | comm            | (sysmon)                                  | 可执行文件名                                                 |
| 3         | state           | S                                         | 任务状态，R:runnign, S:sleeping (TASK_INTERRUPTIBLE), D:disk sleep (TASK_UNINTERRUPTIBLE), T: stopped/tracing stop, Z:zombie, X:dead |
| 4         | ppid            | 18299                                     | 父进程 ID                                                    |
| 5         | pgid            | 18299                                     | 线程组 ID                                                    |
| 6         | sid             | 26897                                     | 会话组 ID                                                    |
| 7         | tty_nr          | 34822                                     | 进程使用的 tty 终端的设备号                                  |
| 8         | tty_pgrp        | 18299                                     | tty 的 pgrp                                                  |
| 9         | flags           | 1077960960                                | 进程标志位                                                   |
| 10        | min_flt         | 21127758                                  | 该任务不需要从硬盘拷数据而发生的缺页（次缺页）的次数         |
| 11        | cmin_flt        | 57743985                                  | 累计的该任务的所有的waited-for进程曾经发生的次缺页的次数目   |
| 12        | maj_flt         | 0                                         | 该任务需要从硬盘拷数据而发生的缺页（主缺页）的次数           |
| 13        | cmaj_flt        | 0                                         | 累计的该任务的所有的waited-for进程曾经发生的主缺页的次数目   |
| 14        | utime           | 13051                                     | 在用户态运行的时间，以 USER_HZ 衡量                          |
| 15        | stime           | 32320                                     | 在核心态运行的时间，以 USER_HZ 衡量                          |
| 16        | cutime          | 11661                                     | 累计的该任务的所有的 waited-for 进程曾经在用户态运行的时间，包括子进程，以 USER_HZ 衡量 |
| 17        | cstime          | 34755                                     | 累计的该任务的所有的 waited-for 进程曾经在核心态运行的时间，包括子进程，以 USER_HZ 衡量 |
| 18        | priority        | 20                                        | 任务的动态优先级                                             |
| 19        | nice            | 0                                         | 任务的静态优先级                                             |
| 20        | num_threads     | 24                                        | 线程数                                                       |
| 21        | it_real_value   | 0                                         | 已废弃，总是为 0                                             |
| 22        | start_time      | 189531797                                 | 启动该进程的时间点，从系统开机后开始算，比如 init 进程的 start_time 可能为 2，2.6 以前的内核中，以 jiffies 衡量，2.6 及以后的内核中，以 USER_HZ 衡量，一般为 100/秒 |
| 23        | vsize           | 1024700416(B)                             | 虚拟地址空间大小                                             |
| 24        | rss             | 5871(page)                                | 该任务当前驻留物理地址空间的大小，这些页可能用于代码，数据和栈。 |
| 25        | rlim            | 18446744073709551615(bytes)               | 该任务能驻留物理地址空间的最大值                             |
| 26        | start_code      | 4194304                                   | 程序文本段可执行地址的起始位置                               |
| 27        | end_code        | 8576880                                   | 程序文本段可执行地址的结束位置                               |
| 28        | start_stack     | 140732243338944                           | 主进程栈的起始地址                                           |
| 29        | esp             | 140732243338336                           | esp(32 位堆栈指针) 的当前值, 与在进程的内核堆栈页得到的一致. |
| 30        | eip             | 4566419                                   | 指向将要执行的指令的指针, EIP(32 位指令指针)的当前值.        |
| 31        | pending         | 0                                         | 排队中的信号的掩码                                           |
| 32        | block_sig       | 0                                         | 阻塞了的信号的掩码                                           |
| 33        | sigign          | 0                                         | 忽略了的信号的掩码                                           |
| 34        | sigcatch        | 2143420159                                | 被捕获的信号的掩码                                           |
| 35        | -               | 18446744073709551615                      | 占位符，曾经是 wchan 地址，现在使用 /proc/PID/wchan 代替     |
| 36        | -               | 0                                         | 占位符                                                       |
| 37        | -               | 0                                         | 占位符                                                       |
| 38        | exit_signal     | 17                                        | 进程退出时向父进程发送的信号                                 |
| 39        | task_cpu        | 4                                         | 运行在哪个CPU上                                              |
| 40        | rt_priority     | 0                                         | 实时进程的相对优先级别                                       |
| 41        | policy          | 0                                         | 进程的调度策略，0=非实时进程，1=FIFO实时进程；2=RR实时进程   |
| 42        | blkio_ticks     | 0                                         | 花在等待磁盘 IO 上的时间                                     |
| 43        | gtime           | 0                                         | 任务在客户机运行的时间（花费在虚拟 CPU 上运行客户操作系统的时间），以 USER_HZ 衡量 |
| 44        | cgtime          | 0                                         | 子进程花在客户机运行的时间，以 USER_HZ 衡量                  |
| 45        | start_data      | 11882496                                  | 软件的 data+bss 所在的地址起始位置                           |
| 46        | end_data        | 12128640                                  | 软件的 data+bss 所在的地址结束位置                           |
| start_brk | 41033728        | 软件的堆可以通过 brk() 进行扩展的起始地址 |                                                              |
| arg_start | 140732243347465 | 软件的堆可以通过 brk() 进行扩展的结束地址 |                                                              |
| arg_end   | 140732243347474 | 软件的命令行放置的起始地址                |                                                              |
| env_start | 140732243347474 | 软件的命令行放置的结束地址                |                                                              |
| env_end   | 140732243349487 | 软件的环境变量放置的起始地址              |                                                              |
| exit_code | 0               | 软件的环境变量放置的结束地址              |                                                              |

进程的 CPU 占用时间（该值包括其所有线程的 CPU 时间）： 
processCPUTime = utime + stime + cutime + cstime

线程的 CPU 占用时间： 
threadCPUTime = utime + stime

### statm

进程所占用内存大小的统计信息，包含七个值，度量单位是page（page大小可通过getconf PAGESIZE得到）。

```
# cat /proc/2948/statm  72362 12945 4876 569 0 24665 0
```

各个值含义： 
a）进程占用的总的内存； 
b）进程当前时刻占用的物理内存； 
c）同其它进程共享的内存； 
d）进程的代码段； 
e）共享库（从2.6版本起，这个值为0）； 
f）进程的堆栈； 
g）dirty pages（从2.6版本起，这个值为0）。

### status

进程的状态信息。其很多内容与/proc/[pid]/stat和/proc/[pid]/statm，但是却是以一种更清晰地方式展现出来。

| Field                      | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Name                       | 可执行文件的文件名                                           |
| Umask                      | 文件模式创建掩码                                             |
| State                      | 进程状态，（R: 运行, S: 睡眠, D: 不可中断的睡眠, Z: 僵尸, T: 出于被跟踪状态或停止状态） |
| Tgid                       | 线程组 ID                                                    |
| Ngid                       | NUMA 组 ID (没有的话为 0)                                    |
| Pid                        | 进程 ID                                                      |
| PPid                       | 父进程 ID                                                    |
| TracerPid                  | 正在踪该进程的进程 ID (没有的话为 0)                         |
| Uid                        | Real, effective, saved set, and file system UIDs             |
| Gid                        | Real, effective, saved set, and file system GIDs             |
| FDSize                     | 当前分配的文件描述符插槽的数量                               |
| Groups                     | 其它组名列表                                                 |
| NStigd                     | 后代命名空间线程组 ID 层次结构                               |
| NSpid                      | 后代命名空间进程 ID 层次结构                                 |
| NSpgid                     | 后代命名空间进程组 ID 层次结构                               |
| NSsid                      | 后代命名空间会话 ID 层次结构                                 |
| VmPeak                     | 峰值虚拟内存大小                                             |
| VmSize                     | 虚拟地址空间大小，单位为 KB                                  |
| VmLck                      | 已经锁住的物理内存的大小。锁住的物理内存不能交换到硬盘 (locked_vm) |
| VmPin                      | 固定的内存大小（pinned memory size），这些页无法被移动，因为需要被直接访问 |
| VmHWM                      | RSS 峰值大小（高峰）                                         |
| VmRSS                      | 内存部分的大小。 它包含以下三个部分（VmRSS = RssAnon + RssFile + RssShmem） |
| RssAnon                    | 匿名常驻内存的大小                                           |
| RssFile                    | 常驻文件映射的大小                                           |
| RssShmem                   | 常驻共享内存的大小（包括SysV shm，tmpfs 映射和匿名共享内存的映射） |
| VmData                     | 私有数据段的大小                                             |
| VmStk                      | 堆栈段的大小                                                 |
| VmExe                      | 文本段的大小                                                 |
| VmLib                      | 共享库代码的大小                                             |
| VmPTE                      | 该进程的所有页表的大小，单位：KB                             |
| VmPMD                      | Size of second-level page tables (since Linux 4.0).          |
| VmSwap                     | 交换出内存的匿名私有页的大小（不包括 shmem 交换所使用的内存） |
| HugetlbPages               | hugetlb 内存部分的大小                                       |
| CoreDumping                | 正在转储的进程内存（杀死该进程可能会产生中断的 core）        |
| Threads                    | 当前进程的线程数                                             |
| SigQ                       | 当前处在队列中的信号/队列一共可以存储多少信号                |
| SigPnd                     | 当前线程 pending 的信号的掩码                                |
| ShdPnd                     | 表示整个进程 pending 的信号的掩码                            |
| SigBlk                     | 被阻塞的信号的掩码                                           |
| SigIgn                     | 被忽略的信号的掩码                                           |
| SigCgt                     | 被捕获到的信号的掩码                                         |
| CapInh                     | 能被当前进程执行的程序的继承的能力的掩码                     |
| CapPrm                     | 进程能够使用的能力的掩码，可以包含CapEff中没有的能力，这些能力是被进程自己临时放弃的，CapEff是CapPrm的一个子集，进程放弃没有必要的能力有利于提高安全性 |
| CapEff                     | 进程的有效能力的掩码                                         |
| CapBnd                     | 能力边界集的掩码                                             |
| NoNewPrivs                 | no_new_privs, like prctl(PR_GET_NO_NEW_PRIV, …)              |
| Seccomp                    | seccomp 模式，like prctl(PR_GET_SECCOMP, …)                  |
| Cpus_allowed               | 可能运行该进程的内核掩码，16 进制，转成二进制之后，标志位为 1 的表示允许，标志位为 0 的表示不允许 |
| Cpus_allowed_list          | 可能运行该进程的内核列表                                     |
| Mems_allowed               | 该进程可能使用的内存节点的掩码                               |
| Mems_allowed_list          | 该进程可能使用的内存节点的列表                               |
| voluntary_ctxt_switches    | 主动切入/切出 CPU 的次数                                     |
| nonvoluntary_ctxt_switches | 被动切入/切出 CPU 的次数                                     |

示例输出：

```
# cat status Name:   sysmonState:  S (sleeping)Tgid:   6606Ngid:   0Pid:    6606PPid:   18299TracerPid:  0Uid:    0   0   0   0Gid:    0   0   0   0FDSize: 64Groups: 0 VmPeak:   984292 kBVmSize:   984292 kBVmLck:         0 kBVmPin:         0 kBVmHWM:     23024 kBVmRSS:     23024 kBVmData:   970264 kBVmStk:       136 kBVmExe:      4280 kBVmLib:      2012 kBVmPTE:       204 kBVmSwap:        0 kBThreads:    22SigQ:   0/31823SigPnd: 0000000000000000ShdPnd: 0000000000000000SigBlk: 0000000000000000SigIgn: 0000000000000000SigCgt: ffffffffffc1feffCapInh: 0000000000000000CapPrm: 0000003fffffffffCapEff: 0000003fffffffffCapBnd: 0000003fffffffffSeccomp:    0Cpus_allowed:   ffCpus_allowed_list:  0-7Mems_allowed:   00000000,00000001Mems_allowed_list:  0voluntary_ctxt_switches:    1305nonvoluntary_ctxt_switches: 221
```

### syscall

当前进程正在执行的系统调用

举例如下：

```
$ cat /proc/2406/syscall202 0xab3730 0x0 0x0 0x0 0x0 0x0 0x7ffff7f6ec68 0x455bb3
```

第一个值是系统调用号（202代表poll），后面跟着 6 个系统调用的参数值（位于寄存器中），最后两个值依次是堆栈指针和指令计数器的值。

如果当前进程虽然阻塞，但阻塞函数并不是系统调用，则系统调用号的值为 -1，后面只有堆栈指针和指令计数器的值。

如果进程没有阻塞，则这个文件只有一个 running 的字符串。

内核编译时打开了 CONFIG_HAVE_ARCH_TRACEHOOK 编译选项，才会生成这个文件。 

### task

一个目录包含了硬链接到该进程启动的任何任务。 
该目录包含进程的所有线程信息，每个线程都对应一个以线程 ID 命名的目录，目录中的内容是该线程的相关信息。

### timers

### uid_map

uid_map 文件公开了用户 ID 从进程 pid 的用户命名空间到打开 uid_map 的进程的用户命名空间的映射（但是请参阅下面的这一点的限定条件）。

换句话说，根据读取进程的用户命名空间的用户ID映射，在从特定uid_map文件读取时，位于不同用户命名空间中的进程可能会看到不同的值。

uid_map文件中的每一行指定两个用户命名空间之间的一系列连续用户ID的一对一映射。（首次创建用户命名空间时，此文件为空。） 
每行中由被空格分隔的三个数字组成。 前两个数字指定两个用户命名空间中每个用户名称的起始用户 ID。 第三个数字指定映射范围的长度。 
字段解释如下：

1. 进程pid的用户命名空间中用户标识范围的开始。
2. 由字段一映射指定的用户ID的用户ID范围的开始。 如何解释字段2取决于打开uid_map的进程和进程pid是否在同一个用户命名空间中，如下所示： 
   a. 如果两个进程位于不同的用户名称空间中：字段2是打开uid_map的进程的用户名称空间中一系列用户标识的开头。 
   b. 如果两个进程在同一个用户命名空间中：字段2是进程pid的父用户命名空间中用户ID范围的开始。 这种情况启用uid_map的开启者（这里的常见情况是打开/ proc / self / uid_map）来查看用户ID到创建此用户命名空间的进程的用户命名空间的映射。
3. 在两个用户名称空间之间映射的用户标识范围的长度。

### wchan

当进程sleep时，kernel当前运行的函数。

```
root@lt:/proc/6606# cat wchan futex_wait_queue_me
```

# 参考

- [linux/Documentation/filesystems/proc.txt](https://github.com/torvalds/linux/blob/master/Documentation/filesystems/proc.txt)
- [Linux Filesystem Hierarchy - /proc](http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)
- [The /proc filesystem](http://www.comptechdoc.org/os/linux/howlinuxworks/linux_hlproc.html)
- [wiki - procfs](https://zh.wikipedia.org/wiki/Procfs)
- [进程切换：自愿(VOLUNTARY)与强制(INVOLUNTARY)](http://linuxperf.com/?p=209)
- [GNU/Linux下的/proc/[pid\]目录下的文件分析](https://github.com/NanXiao/gnu-linux-proc-pid-intro)
- [Linux中 /proc/[pid\] 目录各文件简析](https://www.hi-linux.com/posts/64295.html)