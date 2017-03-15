mysql主从复制
---

MySQL的主从复制不是copy，而是通过逻辑的binlog日志复制到要同步的服务器。然后由本地线程读取日志里面的SQL语句，重新应用到MySQL数据库中

mysql数据库支持单向，双向，链式级联，环状等不同业务场景的复制。

- master  ----> slave              一级单slave
- master  ----> slaves             一级多slaves
- master  ----> slave --- slave    多级复制
- master1 <---> master2            双主
- masters ----> slave              多主

- 应用场景1 ： 从服务器作为主服务器的备份
- 应用场景2 ： 主从服务器实现读写分离,(通过应用程序PHP,JAVA等，或者mysql-prox ,amoeba)
- 把多个服务器根据应用的重要性进行拆分访问

PHP 和 JAVA都可以设置通过设置多个连接文件来实现读写分离，当语句关键字为读时连接读库的连接文件，若为update，insert,delete时，连接写库的连接文件

要实现MYSQL的主从复制，必须打开MYSQL的binlog记录功能。可通过在MySQL的配置文件my.cnf中的mysqld的模块([mysqld]标识后面的参数部分)增加“log-bin”参数选项来实现。

```txt
[mysqld]
log-bin = /data/3306/mysql-bin
```
注意一定要放在[mysqld]选项之后才能生效

MySQL主从复制原来重点

* 主从复制的异步的逻辑的SQL语句级的复制
* 复制时，主库有一个I/O线程，从库有两个线程，即I/O和SQL线程
* 实现主从复制的必要条件是主库要开启记录binlog功能
* 作为复制的所有MySQL节点的server-id都不能相同
* binlog 文件只记录对数据库有更改的SQL语句，(来自主库的内容变更)，不记录任何查询(如select 、show)语句



参考资料
---

- 跟老男孩学linux运维 