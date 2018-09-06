---
layout: post
title:  "Tacacsplus服务器配置"
categories: Linux
tags: Linux tacacsplus
author: DJY
---

* content
{:toc}
## Tacacs+服务器配置

### Install

gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3)

1. Eviroment

```
apt-get -y install gcc make flex bison libwrap0-dev
```

2. Download software

```
wget ftp://ftp.shrubbery.net/pub/tac_plus/tacacs+-F4.0.4.28.tar.gz  
tar -zxvf tacacs+-F4.0.4.28.tar.gz
cd tacacs+-F4.0.4.28
./configure
make
make install
```
**PS:only above F4.0.4.28 support ipv6**

3. After install,create two file in /usr/local/bin  

```
#ls /usr/local/bin/tac*
/usr/local/bin/tac_plus  /usr/local/bin/tac_pwd
tac_plus  tacacs plus 可执行文件
tac_pwd – generate DES or MD5 encryption of a password
root@ubuntu:~# /usr/local/bin/tac_plus 
Usage: tac_plus -C <config_file> [-GghiLPstv] [-B <bind address>] [-d <debug level>] [-l <logfile>] [-p <port>] [-u <wtmpfile>]
    -G  stay in foreground; do not detach from the tty
    -g  single thread mode
    -h  display this message
    -i  inetd mode
    -L  lookup peer addresses for logs
    -P  parse the configuration file and exit
    -S  enable single-connection
    -s  refuse SENDPASS
    -t  also log to /dev/console
    -v  display version information
```

4 use tac_pwd encrpt password (option)
```
# /usr/local/bin/tac_pwd
Password to be encrypted: password
VUjB99kC2IGws
```

5 configurate tac_plus.conf file  
```
root@ubuntu:~# cat tac_plus.conf
#set the tacacs key 
key = "tackey"      '>>>>>>>>>>解密tacacs密文要输入这个密钥'
#set the user accounts user details
user = casa {
    default service = permit
    login = cleartext casa
    pap   = cleartext casa
    service = exec {
        priv-lvl = 15
    }
    service = shell {
        priv-lvl = 15
    }
}
user = root {
    default service = permit
    login = cleartext casa
    pap   = cleartext casa
    service = exec {
        priv-lvl = 15
    }
    service = shell {
        priv-lvl = 15
    }   
}
user = test1 {
    default service = permit
    login = cleartext "casa1234"   #for aaa authentication ascii used 
    pap   = cleartext "casa1234"   #key for login default
    service = exec {
                     priv-lvl = 15 #tacacs server return the privilege to user 
    }               
    service = shell {
                     priv-lvl = 15
     }               
}
user = test2 {
    default service = permit
    login = cleartext "casa12345" 
    pap   = cleartext "casa12345"  
    service = exec {
                     priv-lvl = 1 
    }               
    service = shell {
                     priv-lvl = 1
    }                
}
#user = test2 {
#   default service = permit
#   member = admingroup
#   login = cleartext "casa1234"
#}
#set the group details
#group details
# admin group
#group = admingroup {
#   default service = permit
#   service = exec {
#       priv-lvl = 15
#   }
#}
# set enable password
#Enable password setup for users:
user = $enable$ {
    login = cleartext "casa"   '>>>>>>>>>进入casa后enable模式的密码'
}
#set the location of the accounting file
#accounting file = /var/log/tac_plus.log
```
6 start service
```
root@ubuntu:~#/usr/local/bin/tac_plus -C tac_plus.conf 
root@ubuntu:~# ps -x | grep tac
12542 pts/0    S      0:00 tac_plus -C tac_plus.conf
root@ubuntu:~#netstat -na|grep :49
tcp 0 0 0.0.0.0:49 0.0.0.0:* LISTEN
```
> error 
> tac_plus: error while loading shared libraries: libtacacs.so.1: 
> cannot open shared object file: No such file or directory

```
root@ubuntu:/aaa/tacacs+-F4.0.4.26# find -name "libtacacs.so.1"
./.libs/libtacacs.so.1
root@ubuntu:/aaa/tacacs+-F4.0.4.26# cd ./.libs/
root@ubuntu:/aaa/tacacs+-F4.0.4.26/.libs# pwd
/aaa/tacacs+-F4.0.4.26/.libs
root@ubuntu:/aaa/tacacs+-F4.0.4.26/.libs#echo $PWD  > /etc/ld.so.conf.d/tac.conf
root@ubuntu:/aaa/tacacs+-F4.0.4.26/.libs# cat /etc/ld.so.conf.d/tac.conf 
/aaa/tacacs+-F4.0.4.26/.libs
# vi /etc/ld.so.conf
add /usr/local/lib under /etc/ld.so.conf
root@freelinux#ldconfig
```