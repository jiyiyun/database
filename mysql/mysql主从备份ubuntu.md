mysql主从备份ubuntu
---

- Master1 192.168.100.65
- Master2 192.168.100.66
- Server version: 10.0.29-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

1 . 安装mariadb
---

```txt
root@ubuntu:~# apt install mariadb-server python-pymysql -y
```

2 . 配置50-server.cnf(类似于mariadb-server.cnf,mysql官方文档设置my.cnf相当于同时设置了client和server)
---

```txt
root@ubuntu:~# vi /etc/mysql/mariadb.conf.d/50-server.cnf
```
- 开启log-bin
- 设置server-id

和其他参数

Master1 主机

```txt
[mysqld]
log-bin = /var/log/mysql/master1565-bin          #指定bin-log位置
server-id = 10065                                 #server-id保持全局唯一就行了
binlog-ignore-db = mysql,information_schema      #这行忽略对这两个库操作的日志
slave-skip-errors = all                          #忽略所有复制产生的错误

# bind-address          = 127.0.0.1              #这个注释掉，这个是指定具体的那个IP可以相连
```
Master2 主机

```txt
[mysqld]
log-bin = /var/log/mysql/master1566-bin
server-id = 10066                                  #server-id全局唯一
binlog-ignore-db = mysql,information_schema 
slave-skip-errors = all

# bind-address          = 127.0.0.1               #注释掉
```
3 . 重启数据库
---

```txt
root@ubuntu:~# systemctl restart mysql
```

4 . 创建用于同步mysql帐号,(创建帐号repl，仅有REPLICATION SLAVE 权限)
---

```txt
MariaDB [(none)]> create user repl;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%' IDENTIFIED BY 'repl';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

！！！！ 5到10是排错，请不要操作，从第11开始操作
===

```txt
********以下是排错不要操作********以下是排错不要操作********以下是排错不要操作********
```

5 . 两台主机使用relp帐号互联

master1 主机进入mysql(master必须指向对端66)

```txt
change master to master_host='192.168.100.66',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='master-bin.000001',
master_log_pos=0;
```
master2 主机进入mysql (master必须指向对端65)

```txt
change master to master_host='192.168.100.65',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='master-bin.000001',
master_log_pos=0;
```

6 . 重启数据库

```txt
root@ubuntu:~# systemctl restart mysql
```

7 . 查看slave

master1节点

```txt
MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.100.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000001
          Read_Master_Log_Pos: 4
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 4
        Relay_Master_Log_File: master-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: Yes
```

master2 节点

```txt
MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.100.65
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000001
          Read_Master_Log_Pos: 4
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 4
        Relay_Master_Log_File: master-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: Yes
```
找到各自的slave了，但是Slave_IO_Running: No 参数是NO，需要手动开启slave

```txt
MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)
```
再次查看slave

```txt
MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.100.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000001
          Read_Master_Log_Pos: 4
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 4
        Relay_Master_Log_File: master-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: Yes
```
还是没有起来，原因呢？

```txt
MariaDB [(none)]>  show master status;
+-----------------------+----------+--------------+--------------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB         |
+-----------------------+----------+--------------+--------------------------+
| master10065-bin.000002 |      331 |              | mysql,information_schema |
+-----------------------+----------+--------------+--------------------------+
1 row in set (0.00 sec)
```
这里显示 Position为331 ,而我们在对接的时候使用master_log_pos=0;所以要清除以上对接信息重新对接

```txt
MariaDB [(none)]> stop slave;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> reset slave;
Query OK, 0 rows affected (0.00 sec)
```
重启数据库

```txt
root@ubuntu:~# systemctl restart mysql
```
查看是否清除slave

```txt
MariaDB [(none)]> show slave status\G
Empty set (0.00 sec)
```
8 . 进入mysql帐号，对接对方数据库

master1 数据库

```txt
change master to master_host='192.168.100.66',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='master-bin.000001',
master_log_pos=331;
```

master2数据库

```txt
change master to master_host='192.168.100.65',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='master-bin.000001',
master_log_pos=331;
```
9 . 重启数据库

```txt
root@ubuntu:~# systemctl restart mysql
```
10 . 查看数据库对接情况

```txt
MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.100.65
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000001
          Read_Master_Log_Pos: 331
               Relay_Log_File: mysqld-relay-bin.000003
                Relay_Log_Pos: 4
        Relay_Master_Log_File: master-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: Yes

MariaDB [(none)]> show master status\G
*************************** 1. row ***************************
            File: master10066-bin.000005
        Position: 331
    Binlog_Do_DB: 
Binlog_Ignore_DB: mysql,information_schema
1 row in set (0.00 sec)
```
对接失败！！！

再次清除，重新开始

修改50-server.cnf中binlog-ignore-db = mysql,information_schema 参数
```txt
root@ubuntu:~# vi /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
log-bin = /var/log/mysql/master1565-bin
server-id = 10065
#binlog-ignore-db = mysql,information_schema
slave-skip-errors = all
```
将binlog-ignore-db = mysql,information_schema注释掉重启数据库

```txt
root@ubuntu:~# systemctl restart mysql
```
验证对接

```txt
MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.100.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000001
          Read_Master_Log_Pos: 331
               Relay_Log_File: mysqld-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: master-bin.000001
             Slave_IO_Running: No
            Slave_SQL_Running: Yes
```
Slave_IO_Running: No对接不成功

```txt
MariaDB [(none)]> show master status;
+-----------------------+----------+--------------+------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-----------------------+----------+--------------+------------------+
| master10065-bin.000006 |      444 |              |                  |
+-----------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```
这里position已经是444了，

发现这段报错

MariaDB [(none)]> show slave status\G

Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'

找不到日志文件清除配置重新开始

```txt
********以上是排错不要操作********以上是排错不要操作********以上是排错不要操作********
```

!!!最重要的
===

11 . 重新对接数据库
===

先查看master status
---

Master1

```txt
MariaDB [(none)]> show master status\G
*************************** 1. row ***************************
            File: master10065-bin.000008
        Position: 331
    Binlog_Do_DB: 
Binlog_Ignore_DB: 
1 row in set (0.00 sec)
```
Master2

```txt
MariaDB [(none)]> show master status\G
*************************** 1. row ***************************
            File: master10066-bin.000009
        Position: 331
    Binlog_Do_DB: 
Binlog_Ignore_DB: 
1 row in set (0.00 sec)
```

修改对接文件
---

master1

```txt
change master to master_host='192.168.100.66',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='master10066-bin.000009',
master_log_pos=331;
```
master2

```txt
change master to master_host='192.168.100.65',
master_port=3306,
master_user='repl',
master_password='repl', 
master_log_file='master10065-bin.000008',
master_log_pos=331;
```
重点：
- master_log_file=   一定要是对端显示的文件名称，这样才能找到bin-log文件
根据实际情况修改对接文件

启动slave

```txt
MariaDB [(none)]> start slave
```

查看结果 (show slave status\G)
---

master1

```txt
MariaDB [(none)]> start slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.100.66
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master10066-bin.000009
          Read_Master_Log_Pos: 331
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 540
        Relay_Master_Log_File: master10066-bin.000009
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```
master2

```
MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.100.65
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master10065-bin.000008
          Read_Master_Log_Pos: 331
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 540
        Relay_Master_Log_File: master10065-bin.000008
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

看到

- Slave_IO_Running: Yes
- Slave_SQL_Running: Yes

成功了，下来创建数据库验证

验证对接情况
---

master1节点

```txt
MariaDB [(none)]> create database mater_a;
Query OK, 1 row affected (0.00 sec)
```
master2节点

```txt
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mater_a            |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
```

有了，下来反过来验证再在master2上面创建数据库，

master2

```txt
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mater_a            |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> create database master_b;
Query OK, 1 row affected (0.00 sec)
```
创建了master_b数据库，下来去master1验证

```txt
MariaDB [(none)]> create database mater_a;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| master_b           |
| mater_a            |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.00 sec)
```
mysql双备成功