---
layout: post
title:  "Linux gperftools高性能多线程"
categories: Linux
tags: Linux C grerftools 高性能 多线程 
author: DJY
---

* content
{:toc}
# 概述

------

gperftools 包含高性能并支持多线程的 malloc 实现，以及内存和 CPU 性能分析工具。 
可以用来提高 malloc 性能，也可以用来分析CPU 性能和内存使用状况，或者检查堆泄漏 。

# 安装可能需要的软件

------

## 编译所需软件

```
apt-get install -y m4 autoconf libtool make gcc g++ libunwind8 libunwind8-dev
```

## 性能分析所需软件

```
apt-get install -y graphviz gv kcachegrind
```

# 编译安装 gperftools

------

如果要生成 profile 和使用 pprof 来分析性能，则需要在运行程序的设备上安装 gperftools。否则只需要编译得到动态库或静态库，将其链接进自己的程序即可

## 编译安装

要生成 profile 和使用 pprof 来分析性能，则需要在运行程序的设备上安装 gperftools。

gperftools git：

```
https://github.com/gperftools/gperftools.git
```

```
unzip gperftools-master.zipcd gperftools-master./autogen.sh./configure --prefix=/usrmake -j32make install
```

## 编译选项

| 选项            | 描述                       |
| --------------- | -------------------------- |
| –disable-shared | 不编译动态库               |
| –prefix=/usr    | 将所有的东西安装到 /usr 下 |
| –libexecdir=DIR | 设置可执行文件安装目录     |
| –libdir=DIR     | 设置 libraries 安装目录    |

# 链接 gperftools 到程序

------

## 动态库

编译好 gperftools 并安装之后，可以直接通过 -ltcmalloc 或 -lprofiler 分别链接 tcmalloc 或 profiler 到自己的程序。这两个选项可以分开或同时使用。

## 静态库

要将 tcmalloc 或 profiler 的静态库链接到自己的程序，除了链接所需的 .a 文件之后，还需要链接下列库：

```
-lstdc++ -lm
```

如果是用 gcc 来编译，则需要加上下列选项，因为根据 gperftools 的 github 主页上的说明，gcc 在编译的时候会做一些优化以确保使用其内建的 malloc，而不是链接进去的 tcmalloc，所以为了避免这个问题，需要加上以下选项：

```
-fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free 
```

### 仅链接 tcmalloc

```
gcc -o zpagent ./obj/*.o ./lib/*.a -lpthread ./lib/libtcmalloc.a -lstdc++ -lm -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free 
```

### 仅链接 profiler

```
gcc -o zpagent ./obj/*.o ./lib/*.a -lpthread ./lib/libprofiler.a -lstdc++ -lm
```

### 同时链接 tcmalloc 和 profiler

如果要同时链接 tcmalloc 和 profiler，不能通过分别链接 libtcmalloc.a 和 libprofiler.a 的方式，只能通过直接链接 lib/libtcmalloc_and_profiler.a 的方式。

```
gcc -o zpagent ./obj/*.o ./lib/*.a -lpthread ./lib/libtcmalloc_and_profiler.a -lstdc++ -lm -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free 
```

# 生成分析文件

------

heap 检查:

```
gcc [...] -o myprogram -ltcmallocHEAPCHECK=normal ./myprogram
```

Heap 分析:

```
gcc [...] -o myprogram -ltcmallocHEAPPROFILE=/tmp/netheap ./myprogram
```

Cpu 分析:

```
gcc [...] -o myprogram -lprofilerCPUPROFILE=/tmp/profile ./myprogram
```

- CPUPROFILE_FREQUENCY=x 
  设置每秒取样数，默认为 100，超过 1000 则和 1000 没有明显差别。在运行程序的终端 export 设置。

# CPU Profile 示例 - 分析部分代码

通过 ProfilerStart(“/tmp/prof”) 和 ProfilerStop() 设置需要分析的代码起始/终止位置，然后重新编译，就可以只分析这部分代码的 CPU 性能

## 代码

```
    ProfilerStart("/tmp/prof");    strcpy(zp_conf.zp_file, "/fdsk/zp.conf");    zp_conf.interval = 1;    zpagent_config_init();    ProfilerStop();
```

## 生成 profile

执行下列命令可以生成 zpagent 的 profile 到 /tmp/profile。

```
CPUPROFILE=/tmp/profile ./zpagent
```

## 分析 profile

------

生成的 profile 可以使用 pprof 来进行分析，可以通过 text 和图形的方式显示出来。 
图形下查看，如果是远程，则 gv 响应速度最快，callgrind 则可以分析调用关系，web 下是直接生成 svg 矢量图的方式查看，字体渲染效果最好，如果是本地，则响应速度也很快。

通用的查看分析结果的方式为：

```
pprof zpagent /tmp/prof.out 
```

进入 pprof 之后，输入 web/gv/callgrind 则表示通过 web/gv/callgrind 等方式进行查看。

Note:

1. 如果要用 web/gv/callgrind 方式查看，则需要在设备上安装 web browerser/gv/kcachegrind。
2. 如果编译安装 gperftools 的时候，没有指定 –prefix=/usr，则默认会安装到 /usr/local 下，即 pprof 安装到 /usr/local/bin/ 下，libraries 则安装到 /usr/local/lib/ 下，这个时候执行 pprof 会提示找不到动态库，可以通过设置环境变量的方式解决：

```
export LD_LIBRARY_PATH=/usr/local/lib
```

### text

#### 显示结果

```
pprof --text zpagent /tmp/prof.out 
```

#### 效果

```
Using local file zpagent.Using local file /tmp/prof.out.Total: 70 samples       9  12.9%  12.9%        9  12.9% re_node_set_contains       7  10.0%  22.9%        7  10.0% re_node_set_add_intersect       6   8.6%  31.4%        6   8.6% check_node_accept       6   8.6%  40.0%        7  10.0% update_regs       4   5.7%  45.7%        4   5.7% __GI___strcmp_ssse3       4   5.7%  51.4%       10  14.3% build_sifted_states       3   4.3%  55.7%        6   8.6% avl_probe       3   4.3%  60.0%        3   4.3% proceed_next_node       3   4.3%  64.3%        3   4.3% re_acquire_state       3   4.3%  68.6%        3   4.3% re_node_set_compare       2   2.9%  71.4%        4   5.7% do_free_helper (inline)       2   2.9%  74.3%        2   2.9% re_node_set_insert.part.15       2   2.9%  77.1%       24  34.3% re_search_internal       2   2.9%  80.0%       17  24.3% sift_states_backward       2   2.9%  82.9%        2   2.9% tcmalloc::CentralFreeList::Populate       2   2.9%  85.7%        6   8.6% update_cur_sifted_state       1   1.4%  87.1%        1   1.4% GetSizeWithCallback (inline)       1   1.4%  88.6%        1   1.4% SLL_PopRange (inline)       1   1.4%  90.0%        1   1.4% SLL_SetNext (inline)       1   1.4%  91.4%        1   1.4% __GI_memset       1   1.4%  92.9%        1   1.4% __memcpy_sse2       1   1.4%  94.3%        1   1.4% check_matching       1   1.4%  95.7%        4   5.7% re_node_set_insert       1   1.4%  97.1%        1   1.4% transit_state       1   1.4%  98.6%       55  78.6% zpagent_config_get_ap_id       1   1.4% 100.0%        2   2.9% zpagent_config_zone_insert_ap       0   0.0% 100.0%        1   1.4% SLL_Push (inline)       0   0.0% 100.0%       70 100.0% __libc_start_main       0   0.0% 100.0%       55  78.6% __regexec       0   0.0% 100.0%        3   4.3% __regfree       0   0.0% 100.0%       70 100.0% _start       0   0.0% 100.0%        9  12.9% add_epsilon_src_nodes       0   0.0% 100.0%        1   1.4% avl_create       0   0.0% 100.0%        4   5.7% do_free (inline)       0   0.0% 100.0%        4   5.7% do_free_with_callback (inline)       0   0.0% 100.0%        2   2.9% do_malloc (inline)       0   0.0% 100.0%        2   2.9% do_malloc_or_cpp_alloc (inline)       0   0.0% 100.0%        2   2.9% do_malloc_small (inline)       0   0.0% 100.0%        1   1.4% do_realloc (inline)       0   0.0% 100.0%        1   1.4% do_realloc_with_callback (inline)       0   0.0% 100.0%        2   2.9% free_dfa_content       0   0.0% 100.0%        1   1.4% free_state       0   0.0% 100.0%        1   1.4% ltegw_avl_create       0   0.0% 100.0%        6   8.6% ltegw_avl_insert       0   0.0% 100.0%       70 100.0% main       0   0.0% 100.0%       29  41.4% prune_impossible_nodes       0   0.0% 100.0%        1   1.4% single_list_find       0   0.0% 100.0%        1   1.4% single_list_insert_head       0   0.0% 100.0%        4   5.7% tc_free       0   0.0% 100.0%        2   2.9% tc_malloc       0   0.0% 100.0%        1   1.4% tc_realloc       0   0.0% 100.0%        2   2.9% tcmalloc::CentralFreeList::FetchFromOneSpansSafe       0   0.0% 100.0%        2   2.9% tcmalloc::CentralFreeList::RemoveRange       0   0.0% 100.0%        2   2.9% tcmalloc::ThreadCache::Allocate (inline)       0   0.0% 100.0%        2   2.9% tcmalloc::ThreadCache::Deallocate (inline)       0   0.0% 100.0%        2   2.9% tcmalloc::ThreadCache::FetchFromCentralCache       0   0.0% 100.0%        1   1.4% tcmalloc::ThreadCache::FreeList::PopRange (inline)       0   0.0% 100.0%        1   1.4% tcmalloc::ThreadCache::FreeList::Push (inline)       0   0.0% 100.0%        1   1.4% tcmalloc::ThreadCache::ListTooLong       0   0.0% 100.0%        1   1.4% tcmalloc::ThreadCache::ReleaseToCentralCache       0   0.0% 100.0%       10  14.3% zpagent_config_ap_cb_create_and_add       0   0.0% 100.0%        3   4.3% zpagent_config_avl_cmp_ap       0   0.0% 100.0%       70 100.0% zpagent_config_init       0   0.0% 100.0%        4   5.7% zpagent_config_is_zone_begin       0   0.0% 100.0%        1   1.4% zpagent_config_list_ap_cmp       0   0.0% 100.0%       70 100.0% zpagent_config_parse       0   0.0% 100.0%        1   1.4% zpagent_config_read_line       0   0.0% 100.0%        4   5.7% zpagent_config_regex       0   0.0% 100.0%        1   1.4% zpagent_config_ue_db_create
```

#### 说明

| 分析样本数 | 样本所占百分比 | 目前为止打印的样本所占百分比 | 这个函数的及其调用的子函数的样本数 | 这个函数的及其调用的子函数的样本数所占百分比 | 函数名               |
| ---------- | -------------- | ---------------------------- | ---------------------------------- | -------------------------------------------- | -------------------- |
| 9          | 12.9%          | 12.9%                        | 9                                  | 12.9%                                        | re_node_set_contains |

### gv

#### 打开

```
pprof -gv zpagent /tmp/prof.out    -- 直接显示分析图像pprof --gv --focus=zpagent_config_get_ap_id zpagent /tmp/prof.out     -- 以 zpagent_config_get_ap_id 为中心进行分析pprof --gv --ignore=zpagent_config_get_ap_id agent /tmp/prof.out     -- 忽略 zpagent_config_get_ap_id
```

#### 效果

![title](https://woniuxiang.space/api/file/getImage?fileId=586e96a518f9e3001900001e)

#### 说明

每个节点包含下列内容：

- 函数名
- 本地时间
- 累积时间

分析通过采样方法进行，默认情况下，我们每秒采样100个样本。 因此，输出中的一个时间单位对应于大约10毫秒的执行时间。

“本地”时间是执行直接包含在过程（以及内联到过程中的任何其他过程）中的指令所花费的时间。 “累积”时间是“本地”时间和在任何被调用者中花费的时间的总和。 如果累积时间与本地时间相同，则不打印。

比如，对于 zpagent_config_get_ap_id 这个节点，1( 1.4% )表示这个函数内部代码及内联到这个函数的代码所花费的时间（1 个样本时间）及百分比（1/55 = 1.4%），另外 54 个样本时间花费在 regex 上。55 表示 regex + zpagent_config_get_ap_id 内部及内联过程所话费的时间总数。

### callgrind

#### 将 profile 转化为 callgrind 格式

```
pprof --callgrind zpagent /tmp/prof.out >/tmp/zpagent.callgrind
```

#### 打开

```
kcachegrind /tmp/callgrind.res
```

#### 效果

![title](https://woniuxiang.space/api/file/getImage?fileId=586e92f018f9e3001900001d)

### web

#### 打开

```
pprof --web zpagent /tmp/prof.out
```

#### 效果

![title](https://woniuxiang.space/api/file/getImage?fileId=586e9a3818f9e30019000020)

#### 说明

查看方式同 gv。

# Heap Checker

## 检查 service

    对于 upstart 管理的 service，需要在 /etc/init/service_name.conf 文件中执行二进制文件前加上 “export HEAPCHECK=normal”，另外，**必须手动处理 SIGTERM 信号，否则 stop 服务之后无法生成分析文件**，原因是程序在接收到 SIGTERM 信号之后并不是正常退出，没有执行 exit(), 故而不会进行 heap 检查。

### 示例 - service casa-mobile

    casa-mobile 服务的可执行文件是 virtcasa，virtcasa 进程在起来之后会根据配置文件以不同的参数启动多个子进程以实现不同的功能服务。简而言之，这是一个多进程单线程的服务框架。

#### 修改程序

    对 virtcasa 服务进行 heap 检查： 
    在 virtcasa 中添加函数：

```
void virtcasa_sinal_handle(int sig){    switch(sig) {        case SIGTERM:            printf("get SIGTERM, exitting...\n");            exit(0);        break;    }}
```

    然后在 main 函数的第一行调用：

```
signal(SIGTERM, virtcasa_sinal_handle);
```

- 如果还有其它地方设置 SIGTERM 的处理，需要注释掉其它地方的处理，只保留这一个。
- 如果不希望检查所有的 virtcasa 进程，只希望对指定的子进程进行检查，可以在子进程对应的函数中调用 “signal(SIGTERM, virtcasa_sinal_handle);” 而不是在 main 函数中调用。

#### 修改 Makefile

    由于 virtcasa 引用的库数目众多，且 Makefile 复杂，不能确保 libtcmalloc 一定作为最后一个库引用，所以删除 Makefile 中的 libtcmalloc 库的引用在调试时使用 LD_PRELOAD 加载 tcmalloc 进行调试。

    修改 casa/nfv/src/virtcasa/Makefile 文件，将

```
LDLIBS += $(NFV_LDLIBS) $(LTE_LIBS) -l:libtcmalloc.a -lstdc++ -l:libevent.a -l:libprotobuf-c.a -lglib-2.0 -lpthread -lrt -l:libhiredis.a -l:libcares.a
```

    中的 -l:libtcmalloc.a 删除：

```
LDLIBS += $(NFV_LDLIBS) $(LTE_LIBS) -lstdc++ -l:libevent.a -l:libprotobuf-c.a -lglib-2.0 -lpthread -lrt -l:libhiredis.a -l:libcares.a
```

#### 重新编译

    修改完代码和 Makefile 之后需要重新编译 virtcasa

#### 修改配置文件

    修改 virtcasa 的服务配置文件，其服务配置文件为 /etc/init/casa-mobile.conf，将

```
exec ${NFV_BIN_PATH}/virtcasa${VC_SUFFIX}
```

    修改为：

```
exec env LD_PRELOAD=/usr/lib/libtcmalloc.so HEAPCHECK=strict HEAP_CHECK_DUMP_DIRECTORY=/fdsk ${NFV_BIN_PATH}/virtcasa${VC_SUFFIX}
```

    只需要设置这里的环境变量即可对这里启动的这个进程及其子进程生效（使用 execvp 启动子进程）。

#### 运行

    重新编译后启动服务，运行服务，再停止服务，即可在 /tmp 下生成 virtcasa-xxx.heap 文件。

## Tips

    在 heap checker 中，即使对程序开启了 checker，但程序没有内存泄漏的话，是不会生成 xxx.heap 文件的，如果程序没有分配内存也不会生成 xxx.heap 文件，只有在程序有内存泄漏的时候才会生成 xxx.heap 文件。另外，如果程序 malloc 了一段内存，在程序退出之前一直有指针指向这段内存，那么即使在程序退出时没有 free 这段内存也是不会生成 xxx.heap 文件的。

# 参考

- [gperftoos - wiki](https://github.com/gperftools/gperftools/wiki)
- [gperftools/gperftools ](https://github.com/gperftools/gperftools)
- [用gperftools对C/C++程序进行profile ](http://airekans.github.io/cpp/2014/07/04/gperftools-profile)
- [Gperftools - CPU Profiler ](https://gperftools.github.io/gperftools/cpuprofile.html)
- [Gperftools - Heap Profiler ](https://gperftools.github.io/gperftools/heapprofile.html)
- [Gperftools - Heap Leak Checker ](https://gperftools.github.io/gperftools/heap_checker.html)