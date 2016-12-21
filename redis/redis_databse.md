提问：怎样重启redis而不丢失数据
===
Redis介绍
---
Redis是当前比较热门的NOSQL系统之一，它是一个key-value存储系统。和Memcache类似，但很大程度补偿了Memcache的不足，它支持存储的value类型相对更多，包括string、list、set、zset和hash。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作。在此基础上，Redis支持各种不同方式的排序。

和Memcache一样，Redis数据都是缓存在计算机内存中，不同的是，Memcache只能将数据缓存到内存中，无法自动定期写入硬盘，这就表示，一断电或重启，内存清空，数据丢失。所以Memcache的应用场景适用于缓存无需持久化的数据。而Redis不同的是它会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，实现数据的持久化。

1. 下载redis
---
提前安装好gcc，在centos中g++ 安装方式yum install gcc-g++
https://redis.io/download
2. 编译安装
---
```shell
 2000  tar -xvf redis-3.2.6.tar.gz 
 2001  cd redis-3.2.6
 2002  ls
 2003  make
 2004  make test
 2005  apt install tcl
 2006  apt install recipe
 2009  make test
 2010  make install
 2019  cd /usr/local/
 2020  ls
 2021  ls redis
 2023  cd redis/bin/
 2024  ls
 2026  redis-server
 root@ubuntu3160:/usr/local/redis/bin# redis-server    
46416:C 21 Dec 20:18:03.353 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
46416:M 21 Dec 20:18:03.355 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 46416
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

46416:M 21 Dec 20:18:03.363 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
46416:M 21 Dec 20:18:03.364 # Server started, Redis version 3.2.6
46416:M 21 Dec 20:18:03.364 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
46416:M 21 Dec 20:18:03.364 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
46416:M 21 Dec 20:18:03.365 * The server is now ready to accept connections on port 6379
^C46416:signal-handler (1482322764) Received SIGINT scheduling shutdown...
46416:M 21 Dec 20:19:24.160 # User requested shutdown...
46416:M 21 Dec 20:19:24.160 * Saving the final RDB snapshot before exiting.
46416:M 21 Dec 20:19:24.160 * DB saved on disk
46416:M 21 Dec 20:19:24.160 # Redis is now ready to exit, bye bye...
```
配置redis
---
``` shell
首先编辑conf文件，将daemonize属性改为yes（表明需要在后台运行）
cd etc/
Vi redis.conf
b)再次启动redis服务，并指定启动服务配置文件
redis-server /usr/local/redis/etc/redis.conf
服务端启动成功后，执行redis-cli启动Redis 客户端，查看端口号
# redis-cli
127.0.0.1:6379> 
127.0.0.1:6379> 

redis-cli.exe是一个客户端程序，命令行下运行，简单的一个Set/Get
redis 127.0.0.1:6379> set mykey "this is a value"
OK
redis 127.0.0.1:6379> get mykey
"this is a value"
redis 127.0.0.1:6379>
```
redis常用命令
---
``` shell
Redis：
Redis-server /usr..../redis.conf 启动redis服务，并指定配置文件
Redis-cli 启动redis 客户端
Pkill redis-server 关闭redis服务
Redis-cli shutdown 关闭redis客户端
Netstat -tunpl|grep 6379 查看redis 默认端口号6379占用情况
```
关闭Redis服务
---
``` shell
root@ubuntu:/home/t/redis-3.2.6# pkill redis-server
root@ubuntu:/home/t/redis-3.2.6# netstat -tunpl |grep 6379
root@ubuntu:/home/t/redis-3.2.6# pstree -p |grep redis
root@ubuntu:/home/t/redis-3.2.6# redis-cli 
Could not connect to Redis at 127.0.0.1:6379: Connection refused
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected> exit
```
``` shell
Redis服务端默认连接端口是6379.
mysql 服务端默认连接端口是3306
Mogodb 服务端默认连接端口是27017，还有28017。
```
redis常用配置
---
``` shell
Redis的配置文件中有哪些配置呢？

daemonize 如果需要在后台运行，把该项改为yes

pidfile 配置多个pid的地址 默认在/var/run/redis.pid

bind 绑定ip，设置后只接受来自该ip的请求

port 监听端口，默认是6379

loglevel 分为4个等级：debug verbose notice warning

logfile 用于配置log文件地址

databases 设置数据库个数，默认使用的数据库为0

save 设置redis进行数据库镜像的频率。

rdbcompression 在进行镜像备份时，是否进行压缩

dbfilename 镜像备份文件的文件名

Dir 数据库镜像备份的文件放置路径

Slaveof 设置数据库为其他数据库的从数据库

Masterauth 主数据库连接需要的密码验证

Requriepass 设置 登陆时需要使用密码

Maxclients 限制同时使用的客户数量

Maxmemory 设置redis能够使用的最大内存

Appendonly 开启append only模式

Appendfsync 设置对appendonly.aof文件同步的频率（对数据进行备份的第二种方式）

vm-enabled 是否开启虚拟内存支持   （vm开头的参数都是配置虚拟内存的）

vm-swap-file 设置虚拟内存的交换文件路径

vm-max-memory 设置redis使用的最大物理内存大小

vm-page-size 设置虚拟内存的页大小

vm-pages 设置交换文件的总的page数量

vm-max-threads 设置VM IO同时使用的线程数量

Glueoutputbuf 把小的输出缓存存放在一起

hash-max-zipmap-entries 设置hash的临界值

Activerehashing 重新hash
```
参考资料
---
[Redis安装部署与维护详解]http://www.open-open.com/lib/view/open1426468117367.html