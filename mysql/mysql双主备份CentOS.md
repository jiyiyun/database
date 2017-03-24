搭建mysql双主备份(centos系统)
---

1 . 安装软件

```txt
[root@ ~]# yum install mariadb-server python-pymysql -y
```

2 . 将默认my.cnf替换并编辑

```txt
[root@ ~]# cp /usr/share/mariadb/my-large.cnf /etc/my.cnf
cp: overwrite ‘/etc/my.cnf’? y
```
编辑<code>/etc/my.cnf</code>文件

```txt
更改数据目录

[client]
#password       = your_password
port            = 3306
socket          = /data/mysql/mysql.sock


[mysqld]
datadir=/data/mysql
log-error       = /var/log/mariadb/mariadb.log
port            = 3306
socket          = /data/mysql/mysql.sock

server-id       = 1565                        #server-id 不能冲突
log-bin=mysql-bin                             #已开启，如果没有开启要开启
```
3 . 创建mysql数据目录并初始化
---

```txt
mkdir -p /data/mysql
```

```txt
[root@1565 ~]# mysql_install_db --user=mysql --data=/data/mysql --basedir=/usr/
```
4 . 在两台主机上创建用于备份的帐号
---

```txt
MariaDB [(none)]>create user repl;
MariaDB [(none)]>GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%' IDENTIFIED BY 'repl';
MariaDB [(none)]> FLUSH PRIVILEGES;
```
查看帐号

```txt
MariaDB [(none)]> select User,Host,Password From mysql.user;
+------+-----------+-------------------------------------------+
| User | Host      | Password                                  |
+------+-----------+-------------------------------------------+
| repl | %         | *A424E797037BF97C19A2E88CF7891C5C2038C039 |
+------+-----------+-------------------------------------------+
7 rows in set (0.00 sec)
```
5. 锁表
---

```txt
MariaDB [(none)]>  FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.01 sec)
```
6 . 查看bin-log 文件名称和Position号
---

```txt
MariaDB [(none)]> show master status\G
*************************** 1. row ***************************
            File: mysql-bin.000007
        Position: 327
    Binlog_Do_DB: 
Binlog_Ignore_DB: 
1 row in set (0.00 sec)
```
- File文件名称为   mysql-bin.000007
- Position 号码为  327

下面对接的时候要用这两个数字

7 . 对接两个数据库(A对接B的IP，B对接A的IP)
---

A节点

```txt
change master to master_host='192.168.15.66',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='mysql-bin.000007',
master_log_pos=327;
```

```txt
MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.15.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000007
          Read_Master_Log_Pos: 327
               Relay_Log_File: 1565-relay-bin.000002
                Relay_Log_Pos: 537
        Relay_Master_Log_File: mysql-bin.000007
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

- Slave_IO_Running: Yes
- Slave_SQL_Running: Yes

主从对接成功

B节点操作类似。注意修改IP和bin-log文件名称，Position号

8 . 表解锁
---

```txt
MariaDB [(none)]> UNLOCK TABLES;
Query OK, 0 rows affected (0.00 sec)
```

9 . 测试同步
---

A节点
```txt
MariaDB [(none)]> create database mysql_node_A;
Query OK, 1 row affected (0.00 sec)
```

B节点查看

```txt
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mysql_node_A       |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.01 sec)
```

B 节点创建

```txt
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mysql_node_A       |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.01 sec)

MariaDB [(none)]> create database mysql_node_B;
Query OK, 1 row affected (0.00 sec)
```
A节点查看

```txt
MariaDB [(none)]> UNLOCK TABLES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> create database mysql_node_A;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mysql_node_A       |
| mysql_node_B       |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)
```


10 . 附录：
---

1 . 设置了server_id选项的可以从<code>show variables like 'server_id';</code>查看server_id

```txt
MariaDB [(none)]> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 1565  |
+---------------+-------+
1 row in set (0.00 sec)
```
2 . 进程查看

```txt
MariaDB [(none)]> show processlist \G 
*************************** 1. row ***************************
      Id: 9
    User: system user
    Host: 
      db: NULL
 Command: Connect
    Time: 1689
   State: Waiting for master to send event
    Info: NULL
Progress: 0.000
*************************** 2. row ***************************
      Id: 10
    User: system user
    Host: 
      db: NULL
 Command: Connect
    Time: 911
   State: Slave has read all relay log; waiting for the slave I/O thread to update it
    Info: NULL
Progress: 0.000
*************************** 3. row ***************************
      Id: 11
    User: repl
    Host: 192.168.15.66:51382
      db: NULL
 Command: Binlog Dump
    Time: 1683
   State: Master has sent all binlog to slave; waiting for binlog to be updated
    Info: NULL
Progress: 0.000
*************************** 4. row ***************************
      Id: 12
    User: root
    Host: localhost
      db: NULL
 Command: Query
    Time: 0
   State: init
    Info: show processlist
Progress: 0.000
4 rows in set (0.01 sec)
```

3 . 停止主从备份

```txt
MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.01 sec)
```
4 . 重置mysql数据库
```
MariaDB [(none)]> reset slave;
Query OK, 0 rows affected (0.00 sec)
```
5 . 可以选择那些数据库不用同步

```txt
[mysqld]
log-bin = /var/log/mysql/master1565-bin          #指定bin-log位置
server-id = 10065                                 #server-id保持全局唯一就行了
binlog-ignore-db = mysql,information_schema      #这行忽略对这两个库操作的日志
slave-skip-errors = all                          #忽略所有复制产生的错误
```