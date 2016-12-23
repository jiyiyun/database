mysql连接数查看
---

- show processlist     #查看当前所有连接资料

``` shell
mysql> show processlist;
+----+------+-----------+------+---------+------+----------+------------------+
| Id | User | Host      | db   | Command | Time | State    | Info             |
+----+------+-----------+------+---------+------+----------+------------------+
|  8 | root | localhost | NULL | Query   |    0 | starting | show processlist |
+----+------+-----------+------+---------+------+----------+------------------+
1 row in set (0.00 sec)
```
- show status; 查看状态

``` shell
MariaDB [(none)]> show status;
+------------------------------------------+-------------+
| Variable_name                            | Value       |
+------------------------------------------+-------------+
| Threads_connected                        | 1           |
| Threads_created                          | 36921       |
| Threads_running                          | 1           |
+------------------------------------------+-------------+
```
- mysql> show variables; #查看最大连接数

``` shell
| max_connections                                          | 151 
| max_delayed_threads                                      | 20 
```
``` shell
修改/etc/mysql/my.cnf  或者windows my.ini
#max_connections        = 100
set-variable=max_usr_connections=30   #单用户连接数
set-variable=max_connections=800      #全局的限制连接数
```