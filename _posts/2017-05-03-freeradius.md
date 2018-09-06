---
layout: post
title:  "Freeradius服务器配置"
categories: Linux
tags: Linux freeradius
author: DJY
---

* content
{:toc}
##  Freeradius服务器配置

1 Enviroment

```
apt-get -y install gcc make libssl-dev libtalloc-dev libcrypto++-dev
```
2 Download packet
```
http://freeradius.org/releases/
#tar xzvf freeradius-server-3.0.15.tar.gz
#cd cd freeradius-server-3.0.15/
#./configure
```
3 Install
```
make
make install 
radiusd -x
```
> radiusd: error while loading shared libraries: libfreeradius-radius-020210.so: cannot open shared object file: No such file or directory 	#ldconfig