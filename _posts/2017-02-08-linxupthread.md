---
layout: post
title:  "Linux C pthread"
categories: Linux
tags: Linux C pthread 多线程
author: DJY
---

* content
{:toc}
# 编译

------

编译时，若源文件调用了 pthread 相关的函数，则要在 Makefile 中加上 -lpthread 选项以包含 pthread 库。

# 创建线程

------

## 函数原型

```
#include <pthread.h>int pthread_create(pthread_t *thread, const pthread_attr_t *attr,void *(*start_routine) (void *), void *arg);
```

- attr为设置新线程属性
- start_routine为在新线程中执行的函数，该函数必须得是void *pthread_function(void *arg) {}，即接收和返回的数据格式均为空指针
- arg为传递给start_routine的参数。线程的退出调用的是pthread_exit(“return data”)。 
  注：子线程中用execl打开一个新进程，新进程退出会导致创建新线程的程序退出。

## 示例

```
pthread_t a_thread;void *thread_result;int ret;ret = pthread_create(&a_thread, NULL, thread_function, (void *)message); 
```

# 退出线程

------

## 1. 线程 A 取消线程 B

### 函数原型

```
#include <pthread.h>int pthread_setcancelstate(int state, int *oldstate);int pthread_setcanceltype(int type, int *oldtype);int pthread_cancel(pthread_t thread);void pthread_testcancel(void);
```

pthread_cancel() 调用可以只只退出指定线程，其它线程不受影响。

线程A要取消线程B，则要在B线程中设置允许退出，（A向B发送退出信号，B中若有预先定义接收到退出信号的行为，则B可以在接收到信号后退出），然后A在需要B退出时调用pthread_cancel(a_thread)函数通知B线程退出。

### 工作方式

pthread_cancel() 是让线程 B 在断点处自动退出，POSIX 中的大多数系统调用中都包含断点。当然也可以用 pthread_testcancel() 来手动设置断点，如果现成 A 使用 pthread_cancel() 通知了线程 B，现成在运行中如果没有遇到系统调用中的断电，则会在执行到 pthread_cancel() 的时候退出。

### 使用方式

在线程 B 中设置允许退出及退出方式：

```
res = pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);if (res != 0) {    perror("Thread pthread_setcancelstate failed)");    exit(1);}res = pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);//如果设置 type 为 THREAD_CANCEL_ASYNCHRONOUS，线程并不会等待命中取消点才结束，而是立马结束
```

线程 A 调用 pthread_cancel(thread_B) 通知线程 B 退出。

## 2. 给线程发送退出信号

### 函数原型

```
#include <signal.h>int pthread_kill(pthread_t thread, int sig);
```

pthread_kill 用来给指定现成发送信号，由于 SIGKILL 发送给线程之后，线程默认的操作是执行 exit()，这个操作会导致创建该线程的进程退出。

## 3. 线程主动退出

### 函数原型

```
#include <pthread.h>void pthread_exit(void *retval);
```

当前运行的线程调用 pthread_exit() 的话，只退出当前线程，其它现成不受影响。

## 4. 退出整个进程

### 函数原型

```
#include <stdlib.h>void exit(int status);
```

任何一个线程调用 exit() 都会导致整个进程结束，这个进程下的所有线程也就结束了，这些线程的堆上和栈上的内存会由系统回收。

## 获取子线程函数的退出信息

```
#include <pthread.h>int pthread_join(pthread_t thread, void **retval);
```

### 示例

```
pthread_t a_thread;void *thread_result;int res;res = pthread_create(&a_thread, NULL, thread_function, (void *)message); //创建新线程，新线程中运行thread_function函数res = pthread_join(a_thread, &thread_result) //获取线程a_thread中的返回信息，即thread_function函数中用pthread_exit()返回的参数void *thread_function(void *arg) {    pthread_exit("return data");}
```

# 相关函数

------

- pthread_create

- pthread_join

- pthread_cancel 
  向线程发送退出请求，至于是否退出和什么时候退出取决于线程中的两个参数：可退出性的 state 和 type 
  一个线程的退出 state，取决于 pthreadcancelstate，可以是 enable（新线程的默认值）或 disable。如果一个线程的可退出 state 设置为 disable，则推出请求会放到队列里去直到该线程的可退出设置为 enable。如果一个线程的可退出配置为 enable，则其退出方式取决决定了什么时候退出。

  成功返回 0，有错误则返回一个非 0 值。返回 ESRCH 的错误表示在 thread ID 中找不到线程。

- pthread_exit

- pthread_kill

- pthread_cancel 
  – 只在指定线程的退出点退出线程

- pthread_testcancel

- pthread_setcancelstate

- pthread_setcanceltype

- pthread_key_create

- pthread_self

- pthread_equal 
  – 比较两个 pthread，如果不同则返回 0，相同则返回非 0

- gettid 
  返回线程 id，在一个单线程的进程中，这个线程和进程拥有相同的 id，gettid 的返回值和 getpid 的返回值相同；在一个多线程的进程中，每个线程都有独立的 tid，拥有相同的 pid。

Note 
man threads 可以查看退出点

# 线程私有数据（TSD）

------

    线程私有数据（Thread-specific Data，或 TSD）。

## 使用方式

    首先创建允许所有线程访问的全局变量 key，然后在任意一个线程中调用 pthread_key_create 接口创建这个 可以，然后各个子线程用 pthread_setspecific 和 get_specific 接口使用这个 key 来存储和读取数据，每个线程的操作都是在自己的私有线程数据内完成的。 

## 相关接口

### 创建 TSD

    Posix定义了两个API分别用来创建和注销TSD：

```
int pthread_key_create(pthread_key_t *key, void (*destr_function) (void *))
```

    该函数从TSD池中分配一项，将其值赋给key供以后访问使用。如果destr_function不为空，在线程退出（pthread_exit()）时将以key所关联的数据为参数调用destr_function()，以释放分配的缓冲区。 
不论哪个线程调用pthread_key_create()，所创建的key都是所有线程可访问的，但各个线程可根据自己的需要往key中填入不同的值，这就相当于提供了一个同名而不同值的全局变量。

### 注销 TSD

    注销一个TSD采用如下API：

```
int pthread_key_delete(pthread_key_t key)
```

    这个函数并不检查当前是否有线程正使用该TSD，也不会调用清理函数（destr_function），而只是将TSD释放以供下一次调用pthread_key_create()使用。在LinuxThreads中，它还会将与之相关的线程数据项设为NULL（见”访问”）。

### 读写 TSD

    TSD的读写都通过专门的Posix Thread函数进行，其API定义如下：

```
int  pthread_setspecific(pthread_key_t  key,  const   void  *pointer)void * pthread_getspecific(pthread_key_t key)
```

    写入（pthread_setspecific()）时，将pointer的值（不是所指的内容）与key相关联，而相应的读出函数则将与key相关联的数据读出来。数据类型都设为void *，因此可以指向任何类型的数据。

## 代码示例

```
#include <stdio.h>#include <pthread.h>pthread_key_t   *key = NULL;void echomsg(int t){        printf("destructor excuted in thread %x,param=%x\n",pthread_self(),t);}void * child1(void *arg){        printf("%s: key: %x, sizeof(key): %d\n", __FUNCTION__, key, sizeof(key));        int tid=pthread_self();        printf("thread %x enter\n",tid);        pthread_setspecific(*key,(void *)tid);        printf("thread %x returns %x\n",tid,pthread_getspecific(*key));}void * child2(void *arg){        int tid=pthread_self();        printf("thread %x enter\n",tid);        pthread_setspecific(*key,(void *)tid);        printf("thread %x returns %x\n",tid,pthread_getspecific(*key));}int main(void){        int tid1,tid2;        printf("hello\n");        printf("%s: key: %x, sizeof(key): %d\n", __FUNCTION__, key, sizeof(key));        if (!key) {                printf("key need to be init\n");        }        key = (pthread_key_t *)malloc(sizeof(pthread_key_t));        pthread_key_create(key, NULL);        pthread_create(&tid1,NULL,child1,NULL);        pthread_create(&tid2,NULL,child2,NULL);        sleep(1);        //pthread_key_delete(key);        printf("main thread exit\n");        return 0;}
```

### 执行输出

```
hellomain: key: 0, sizeof(key): 8key need to be initchild1: key: 1ade010, sizeof(key): 8thread 6dbda700 enterthread 6dbda700 returns 6dbda700thread 6d3d9700 enterthread 6d3d9700 returns 6d3d9700main thread exit
```

# 其它

------

## linux 多线程当前目录

linux中新启动的子线程中，其当前目录是启动该线程的用户目录，而不是其父线程的当前目录

# 参考

------

- [Posix线程编程指南(2) ](https://www.ibm.com/developerworks/cn/linux/thread/posix_threadapi/part2/)
- [POSIX 线程详解 ](https://www.ibm.com/developerworks/cn/linux/thread/posix_thread1/)