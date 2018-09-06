---
layout: post
title:  "定期自动清除服务器缓存"
categories: Linux
tags: Linux shell
author: DJY
---

* content
{:toc}

```
root@unassigned-hostname:/public# cat clean_cache.sh
#!/bin/bash
for ((i=1;i<9999999;i++)) do
echo "*********************************************************" | tee -a /var/log/clean_cache.log
echo "The $i times to clean cache" >> /var/log/clean_cache.log
date | tee -a /var/log/clean_cache.log 
echo 1 > /proc/sys/vm/drop_caches | tee -a /var/log/clean_cache.log
sleep 5d
done
```

```
to free pagecache, use
echo 1 > /proc/sys/vm/drop_caches;

to free dentries and inodes, use 
echo 2 > /proc/sys/vm/drop_caches;

to free pagecache, dentries and inodes, use 
echo 3 >/proc/sys/vm/drop_caches.
```

- date 打印当前时间  
- tee -a /var/log/clean_cache.log 把运行结果打印输出到log文件，自定义  
- sleep 5d 5天清理一次，可更改设置时间，s/m/h/d
2 上面脚本开机自启(ubuntu系统)
```
vi /etc/init.d/rc.local  在最后加入如下
cd /public/;/public/clean_cache.sh
```
