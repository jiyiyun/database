Redis集群部署文档(centos6系统)
---
``` shell
（要让集群正常工作至少需要3个主节点，在这里我们要创建6个redis节点，其中三个为主节点，三个为从节点，对应的redis节点的ip和端口对应关系如下）
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
127.0.0.1:7003
127.0.0.1:7004
127.0.0.1:7005

3：创建集群需要的目录
mkdir -p /usr.local/cluster
cd /usr.local/cluster
mkdir 7000
mkdir 7001
mkdir 7002
mkdir 7003
mkdir 7004
mkdir 7005
 
4：修改配置文件redis.conf
cp /usr/local/redis3.0/redis.conf  /usr.local/cluster
vi redis.conf
##修改配置文件中的下面选项
port 7000
daemonize yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
##修改完redis.conf配置文件中的这些配置项之后把这个配置文件分别拷贝到7000/7001/7002/7003/7004/7005目录下面
cp /usr/local/cluster/redis.conf /usr/local/cluster/7000
cp /usr/local/cluster/redis.conf /usr/local/cluster/7001
cp /usr/local/cluster/redis.conf /usr/local/cluster/7002
cp /usr/local/cluster/redis.conf /usr/local/cluster/7003
cp /usr/local/cluster/redis.conf /usr/local/cluster/7004
cp /usr/local/cluster/redis.conf /usr/local/cluster/7005
 
##注意：拷贝完成之后要修改7001/7002/7003/7004/7005目录下面redis.conf文件中的port参数，分别改为对应的文件夹的名称
 5：分别启动这6个redis实例
cd /usr/local/cluster/7000
redis-server redis.conf
cd /usr/local/cluster/7001
redis-server redis.conf
cd /usr/local/cluster/7002
redis-server redis.conf
cd /usr/local/cluster/7003
redis-server redis.conf
cd /usr/local/cluster/7004
redis-server redis.conf
cd /usr/local/cluster/7005
redis-server redis.conf
 
 
##启动之后使用命令查看redis的启动情况ps -ef|grep redis
如下图显示则说明启动成功
 
6：执行redis的创建集群命令创建集群
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
```
6.4 再次执行第6步的命令，正常执行
---
!(http://img.blog.csdn.net/20150121173116437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveHU0NzA0MzgwMDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

输入yes，然后配置完成。
!(http://img.blog.csdn.net/20150121173132250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveHU0NzA0MzgwMDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

至此redis集群即搭建成功！

7：使用redis-cli命令进入集群环境
---
``` shell
redis-cli -c -p 7000
```
参考资料
---
[redis3.0.0 集群安装详细步骤]http://blog.csdn.net/xu470438000/article/details/42971091
