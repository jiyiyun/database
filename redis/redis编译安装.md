redis安装
---

```txt
  120  tar -xf redis-3.2.8.tar.gz
  122  cd redis-3.2.8
  124  make
  126  sudo yum install gcc tcl ruby
  127  make
  129  cd src/
  131  make all
  132  cd ..
  133  mkdir -p /magedu/redis
  134  sudo mkdir -p /magedu/redis
  137  sudo make test
  138  sudo make PREFIX=/magedu/redis instally
以上只是生成了redis二进制文件，如果不指定安装路径，默认安装在/usr/local/
```

启动redis
---

```txt
$cd /magedu/redis/bin
$cp ~/redis-3.2.8/redis.conf .
$ ./redis-server redis.conf

注意要使用sudo 权限
查看redis版本
$ ./redis-cli -v
redis-cli 3.2.8

登录redis
$ ./redis-cli -p 7000
127.0.0.1:7000> KEYS *

```

逐一启动集群单个节点
---
```txt
下面是一个最少选项的集群的配置文件:

port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
文件中的 cluster-enabled 选项用于开实例的集群模式， 而 cluster-conf-file 选项则设定了保存节点配置文件的路径， 默认值为 nodes.conf.节点配置文件无须人为修改， 它由 Redis 集群在启动时创建， 并在有需要时自动进行更新。

要让集群正常运作至少需要三个主节点，不过在刚开始试用集群功能时， 强烈建议使用六个节点： 其中三个为主节点， 而其余三个则是各个主节点的从节点。

首先， 让我们进入一个新目录， 并创建六个以端口号为名字的子目录， 稍后我们在将每个目录中运行一个 Redis 实例： 命令如下:

mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
在文件夹 7000 至 7005 中， 各创建一个 redis.conf 文件， 文件的内容可以使用上面的示例配置文件， 但记得将配置中的端口号从 7000 改为与文件夹名字相同的号码。

从 Redis Github 页面 的 unstable 分支中取出最新的 Redis 源码， 编译出可执行文件 redis-server ， 并将文件复制到 cluster-test 文件夹， 然后使用类似以下命令， 在每个标签页中打开一个实例：

cd 7000
../redis-server ./redis.conf

启动7001 、7002、7003、7004、7005 
```
启动集群
---
```txt
cd /usr/local/redis3.0/src
./redis-trib.rb  create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
6.1执行上面的命令的时候会报错，因为是执行的ruby的脚本，需要ruby的环境
错误内容：/usr/bin/env: ruby: No such file or directory
所以需要安装ruby的环境，这里推荐使用yum install ruby安装
yum install ruby
 
6.2然后再执行第6步的创建集群命令，还会报错，提示缺少rubygems组件，使用yum安装
 
错误内容：
./redis-trib.rb:24:in `require': no such file to load -- rubygems (LoadError)
from ./redis-trib.rb:24
yum install rubygems
6.3再次执行第6步的命令，还会报错，提示不能加载redis，是因为缺少redis和ruby的接口，使用gem 安装
错误内容：
/usr/lib/ruby/site_ruby/1.8/rubygems/custom_require.rb:31:in `gem_original_require': no such file to load -- redis (LoadError)
from /usr/lib/ruby/site_ruby/1.8/rubygems/custom_require.rb:31:in `require'
from ./redis-trib.rb:25
 
gem install redis
报错redis requires Ruby version >= 2.2.2

# curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
# curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
# curl -L get.rvm.io | bash -s stable

# rvm install 2.3.8

# ruby --version
ruby 2.3.8p459 (2018-10-18 revision 65136) [x86_64-linux]
# gem install redis
Fetching: redis-4.1.0.gem (100%)
Successfully installed redis-4.1.0
Parsing documentation for redis-4.1.0
Installing ri documentation for redis-4.1.0
Done installing documentation for redis after 2 seconds
1 gem installed


# ./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
M: e6de5f74e414e2e6dd1ee69a7f6a316882839008 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: c65c314f45374737a8340c5b353e09cb2f27e337 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: 9bc1a37bcc67ad3c4b92dff5931dd696d658773e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: a7144123832c3aec78350d33538c963106c827b9 127.0.0.1:7003
   replicates e6de5f74e414e2e6dd1ee69a7f6a316882839008
S: fc8fa43c84c94c8921f613ccc2b359fa2046ddde 127.0.0.1:7004
   replicates c65c314f45374737a8340c5b353e09cb2f27e337
S: fc8fa43c84c94c8921f613ccc2b359fa2046ddde 127.0.0.1:7005
   replicates 9bc1a37bcc67ad3c4b92dff5931dd696d658773e
Can I set the above configuration? (type 'yes' to accept):yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: e6de5f74e414e2e6dd1ee69a7f6a316882839008 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: a7144123832c3aec78350d33538c963106c827b9 127.0.0.1:7003
   slots: (0 slots) slave
   replicates e6de5f74e414e2e6dd1ee69a7f6a316882839008
M: 9bc1a37bcc67ad3c4b92dff5931dd696d658773e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   0 additional replica(s)
S: fc8fa43c84c94c8921f613ccc2b359fa2046ddde 127.0.0.1:7004
   slots: (0 slots) slave
   replicates c65c314f45374737a8340c5b353e09cb2f27e337
M: c65c314f45374737a8340c5b353e09cb2f27e337 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
测试redis集群
---

```txt
测试 Redis 集群比较简单的办法就是使用 redis-rb-cluster 或者 redis-cli

[root@bogon bin]# ls
appendonly.aof  nodes.conf       redis-check-aof  redis-cli       redis-server
dump.rdb        redis-benchmark  redis-check-rdb  redis-sentinel
[root@bogon bin]# ./redis-cli -c -p 7000
127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
127.0.0.1:7002> set hello world
-> Redirected to slot [866] located at 127.0.0.1:7000
OK
127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"bar"
127.0.0.1:7002> get hello
-> Redirected to slot [866] located at 127.0.0.1:7000
"world"

```
参考资料
- [redis requires Ruby version >= 2.2.2问题]https://www.cnblogs.com/carryping/p/7447823.html
- [Redis 集群教程]http://www.redis.cn/topics/cluster-tutorial.html
