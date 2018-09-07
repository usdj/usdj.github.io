---
layout: post
title:  "Linux library"
categories: Linux
tags: Linux C library
author: DJY
---

* content
{:toc}
# 静态库

静态库的名称形式为 libname.a，每个静态库文件即为一个归档文件，其中包含了多个 .o 模块。

## 创建和维护

ar 命令可以用来创建和维护静态库，其通用形式为：

```
ar options archive object-files...
```

### 创建归档

```
cc -g -c mod1.c mod2.c mod3.car r libdemo.a mod1.o mod2.o mo3.orm mod1.o mod2.o mod3.o
```

### 查看归档文件中的模块

```
ar tv libdemo.a     -- 可以列出 libdemo.a 这个静态库中的目录表
```

### 从归档文件中删除一个模块

```
ar d libdemo.a mod3.o
```

## 使用静态库

使用静态库 libdemo.a 编译可执行程序 prog

```
cc -g -c prog.ccc -g -o prog prog.o libdemo.a
```

### 自动查找库

或是将静态库放在链接器搜索的其中一个标准目录中（如 /usr/lib），然后使用 -l 选项指定库名（即库的文件名去除了 lib 前缀和 .a 后缀）。

```
cc -g -o prog prog.o -ldemo
```

### 指定查找的目录

如果库不在链接器搜索的目录中，可以用 -L 选项指定链接器应该搜索这个额外的目录

```
cc -g -o prog prog.o -Lmylibdir -ldemo
```

一个动态库可以包含很多目标模块，但链接器只会包含那些程序需要的模块。

# 动态库

- 共享库的编译必须使用未知独立的代码，这在大多数架构上都会带来性能开销，因为它需要使用额外的一个寄存器。
- 在运行时必须要执行符号重定位。在符号重定位期间，需要将对共享库中每个符号（变量或函数）的引用修改成符号在虚拟内存中的实际运行时位置。由于存在这个重定位的过程，与静态链接程序相比，一个使用共享库的程序或多或少需要花费一些时间来执行这个过程。

## 手动加载和使用（模块化）

    以下代码演示了主程序如何加载一个模块并运行模块中的函数，模块中如何加载主程序，并尝试运行主程序中的函数但是失败。

需要注意的是，dlopen 打开的 so 模块之后，主程序中的全局变量可以被 so 模块访问，同一份代码，编译成多个 so 模块之后，在同一个进程中加载这些 so 模块，代码中的全局变量对各个 so 模块来说是私有的（废话……，可我也不知道为什么一开始想岔了以为它们共享同一个）

### 代码

#### 主函数代码

```
root@ltepc7:~/dlopen# cat main.c#include <stdio.h>#include <dlfcn.h>int ncall(int a, int b);int main() {    printf("in main\n");    void *handle;    typedef int (*FUNC_PTR)(int, int);    FUNC_PTR func;    char *error;    handle = dlopen("./a.so", RTLD_LAZY);    if (!handle) {        fprintf(stderr, "%s\n", dlerror());        return -1;    }    printf("load a.so success\n");    dlerror();    func = (FUNC_PTR)dlsym(handle, "mcall");    if ((error = dlerror()) != NULL) {        fprintf(stderr, "%s\n", error);        return -1;    }    printf("get func mcall success\n");    func(2, 3);    func = (FUNC_PTR)dlsym(handle, "callmain");        if ((error = dlerror()) != NULL) {                fprintf(stderr, "%s\n", error);                return -1;        }        printf("get func callmain success\n");        func(5, 3);}int ncall(int a, int b) {    printf("%d - %d = %d\n", a, b, a-b);}
```

#### 模块 a.so 代码 a.c

```
root@ltepc7:~/dlopen# cat a.c#include <stdio.h>#include <dlfcn.h>int mcall(int a, int b) {    printf("%d + %d = %d\n", a, b, a+b);}int callmain() {    void *handle;        typedef int (*FUNC_PTR)(int, int);        FUNC_PTR func;        char *error;        handle = dlopen(NULL, RTLD_LAZY | RTLD_GLOBAL);        if (!handle) {                fprintf(stderr, "%s\n", dlerror());                return -1;        }        printf("load main as so success: %s\n", dlerror());        func = (FUNC_PTR)dlsym(handle, "main");        if ((error = dlerror()) != NULL) {                fprintf(stderr, "try to get func ncall: %s\n", error);        }        func = (FUNC_PTR)dlsym(RTLD_DEFAULT, "ncall");        if ((error = dlerror()) != NULL) {                fprintf(stderr, "try to get func ncall: %s\n", error);        return -1;        }        printf("get func ncall success\n");        func(5, 3);}
```

### 编译

#### 主程序

```
gcc main.c -fPIC -ldl -o main    -- -ldl 是必须的参数，在使用模块加载等函数的时候需要
```

#### 模块 a.so

```
gcc a.c -D_GNU_SOURCE -fPIC -shared -o a.so    -- -fPIC 表示编译生成与位置无关的代码    -- -shared 表示编译为动态库文件，不加这个参数编译出来的 .so 文件无法加载    -- 这里的 -D_GNU_SOURCE 表示定义 _GNU_SOURCE 测试特性，当 dlsym 用 RTLD_DEFAULT 作为第一个参数的时候，需要定义这个特性    -- 不能假 -ldl 参数，加了之后编译出来的 a.so 文件会导致主函数崩溃，原因未知
```

执行结果

```
root@ltepc7:~/dlopen# ./main in mainload a.so successget func mcall success2 + 3 = 5get func callmain successload main as so success: (null)try to get func ncall: ./main: undefined symbol: maintry to get func ncall: ./a.so: undefined symbol: ncall
```