---
layout: post
title:  "Linux notify"
categories: Linux
tags: Linux notify 监控
author: DJY
---

* content
{:toc}
### 一、概述

     从 Linux 2.6.13 内核开始，Linux 就推出了 inotify，允许监控程序打开一个独立文件描述符，并针对事件集监控一个或者多个文件，例如打开、关闭、移动/重命名、删除、创建或者改变属性。

     inotify 在监控文件的时候，有一个独立文件描述符来对所有的事件和文件进行监控，然后对于每个具体监控的文件或事件，也都有一个对应的文件描述符存储在 inotify_event 的结构体中。在监控事件消息的时候，是向 inotify 总的独立文件描述符读取事件。

### 二、inotify 的头文件

```
root@vm1:/fdsk# find /usr/include/ -name inotify.h/usr/include/linux/inotify.h/usr/include/x86_64-linux-gnu/sys/inotify.h/usr/include/x86_64-linux-gnu/bits/inotify.h
```

### 三、数据结构体

```
struct inotify_event{  int wd;               /* Watch descriptor.  针对具体的监控文件的文件描述*/  uint32_t mask;        /* Watch mask.  监控事件掩码*/  uint32_t cookie;      /* Cookie to synchronize two events.  */  uint32_t len;         /* Length (including NULs) of name.  */  char name __flexarr;  /* Name.  */};
```

### 四、事件掩码说明

```
/* Supported events suitable for MASK parameter of INOTIFY_ADD_WATCH.  */#define IN_ACCESS        0x00000001     /* File was accessed.  */#define IN_MODIFY        0x00000002     /* File was modified.  */#define IN_ATTRIB        0x00000004     /* Metadata changed.  */#define IN_CLOSE_WRITE   0x00000008     /* Writtable file was closed.  */#define IN_CLOSE_NOWRITE 0x00000010     /* Unwrittable file closed.  */#define IN_CLOSE         (IN_CLOSE_WRITE | IN_CLOSE_NOWRITE) /* Close.  */#define IN_OPEN          0x00000020     /* File was opened.  */#define IN_MOVED_FROM    0x00000040     /* File was moved from X.  */#define IN_MOVED_TO      0x00000080     /* File was moved to Y.  */#define IN_MOVE          (IN_MOVED_FROM | IN_MOVED_TO) /* Moves.  */#define IN_CREATE        0x00000100     /* Subfile was created.  */#define IN_DELETE        0x00000200     /* Subfile was deleted.  */#define IN_DELETE_SELF   0x00000400     /* Self was deleted.  */#define IN_MOVE_SELF     0x00000800     /* Self was moved.  *//* Events sent by the kernel.  */#define IN_UNMOUNT       0x00002000     /* Backing fs was unmounted.  */#define IN_Q_OVERFLOW    0x00004000     /* Event queued overflowed.  */#define IN_IGNORED       0x00008000     /* File was ignored.  *//* Helper events.  */#define IN_CLOSE         (IN_CLOSE_WRITE | IN_CLOSE_NOWRITE)    /* Close.  */#define IN_MOVE          (IN_MOVED_FROM | IN_MOVED_TO)          /* Moves.  *//* Special flags.  */#define IN_ONLYDIR       0x01000000     /* Only watch the path if it is a                                           directory.  */#define IN_DONT_FOLLOW   0x02000000     /* Do not follow a sym link.  */#define IN_EXCL_UNLINK   0x04000000     /* Exclude events on unlinked                                           objects.  */#define IN_MASK_ADD      0x20000000     /* Add to the mask of an already                                           existing watch.  */#define IN_ISDIR         0x40000000     /* Event occurred against dir.  */#define IN_ONESHOT       0x80000000     /* Only send event once.  *//* All events which a program can wait on.  */#define IN_ALL_EVENTS    (IN_ACCESS | IN_MODIFY | IN_ATTRIB | IN_CLOSE_WRITE  \                          | IN_CLOSE_NOWRITE | IN_OPEN | IN_MOVED_FROM        \                          | IN_MOVED_TO | IN_CREATE | IN_DELETE               \                          | IN_DELETE_SELF | IN_MOVE_SELF)
```

### 五、用于 inotify 的 API

     Inotify 提供一个简单的 API，使用最小的文件描述符，并且允许细粒度监控。与 inotify 的通信是通过系统调用实现。可用的函数如下所示：

inotify_init 
     – 是用于创建一个 inotify 实例的系统调用，并返回一个指向该实例的文件描述符。

inotify_init1 
     与 inotify_init 相似，并带有附加标志。如果这些附加标志没有指定，将采用与 inotify_init 相同的值。

inotify_add_watch 
     增加对文件或者目录的监控，并指定需要监控哪些事件。标志用于控制是否将事件添加到已有的监控中，是否只有路径代表一个目录才进行监控，是否要追踪符号链接，是否进行一次性监控，当首次事件出现后就停止监控。

inotify_rm_watch 
     从监控列表中移出监控项目。

read 
     读取包含一个或者多个事件信息的缓存。

close 
    关闭文件描述符，并且移除所有在该描述符上的所有监控。当关于某实例的所有文件描述符都关闭时，资源和下层对象都将释放，以供内核再次使用。

因此，典型的监控程序需要进行如下操作： 
     使用 inotify_init 打开一个文件描述符 
     1. 添加一个或者多个监控 
     2. 等待事件 
     3. 处理事件，然后返回并等待更多事件

    当监控不再活动时，或者接到某个信号之后，关闭文件描述符，清空，然后退出。 
在下一部分中，您将看到可以监控的事件，它们如何在简单的程序中运行。最后，您将看到事件监控如何进行。

### 六、代码示例

```
#include <stdio.h>#include <unistd.h>#include <sys/inotify.h>void main(){    int ifd = -1, wfd = -1, read_num = -1;    char buf[1024];    //初始化出独立的监控文件描述符    ifd = inotify_init();    if (ifd < 0) {        perror("inotify_init failed");    }    //添加需要监控的文件，返回对应的文件描述符    wfd = inotify_add_watch(ifd,"file.txt", IN_ATTRIB | IN_MOVE_SELF);    if (wfd < 0){        perror("inotify_add_watch failed");        return -1;    }    //进入死循环读取事件，没有事件的时候读取操作被堵塞，有事件的时候，将读取到的事件消息传给 process_event 函数进行处理    while (1) {        read_num = read(ifd, buf, sizeof(buf));        if (read_num > 0){            pevent = (struct inotify_event *) &buf;            process_event(pevent);        } else {            perror("read inotify event failed");            sleep(1);        }    }}//事件处理函数，将发生的事件简单打印出来void process_event(struct inotify_event *pevent){    switch(pevent->mask & (IN_ATTRIB | IN_MOVE_SELF)) {        case (IN_ATTRIB):            printf("Warning: The watched file was deleted\n");            break;        case IN_MOVE_SELF:            printf("Warning: The watched file was moved\n");            break;        default:            perror("Unknow inotify mask");            break;    }}
```

### 参考

- [用 inotify 监控 Linux 文件系统事件](https://www.ibm.com/developerworks/cn/linux/l-inotify/)