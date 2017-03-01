
一个小时内学习 SQLite 数据库
===

1. 介绍 

SQLite 是一个开源的嵌入式关系数据库，实现自包容、零配置、支持事务的SQL数据库引擎。 其特点是高度便携、使用方便、结构紧凑、高效、可靠。 与其他数据库管理系统不同，SQLite 的安装和运行非常简单，在大多数情况下 - 只要确保SQLite的二进制文件存在即可开始创建、连接和使用数据库。如果您正在寻找一个嵌入式数据库项目或解决方案，SQLite是绝对值得考虑。

2. 安装

SQLite on Windows

进入 SQL 下载页面：http://www.sqlite.org/download.html

下载 Windows 下的预编译二进制文件包：

```txt
sqlite-shell-win32-x86-<build#>.zip
sqlite-dll-win32-x86-<build#>.zip
```
注意: <build#> 是 sqlite 的编译版本号

将 zip 文件解压到你的磁盘，并将解压后的目录添加到系统的 PATH 变量中，以方便在命令行中执行 sqlite 命令。

可选: 如果你计划发布基于 sqlite 数据库的应用程序，你还需要下载源码以便编译和利用其 API

```txt
sqlite-amalgamation-<build#>.zip
SQLite on Linux
```
在 多个 Linux 发行版提供了方便的命令来获取 SQLite：

```txt
/* For Debian or Ubuntu /*
$ sudo apt-get install sqlite3 sqlite3-dev

/* For RedHat, CentOS, or Fedora/*
$ yum install SQLite3 sqlite3-dev
SQLite on Mac OS X
```
如果你正在使用 Mac OS 雪豹或者更新版本的系统，那么系统上已经装有 SQLite 了。

3. 创建首个 SQLite 数据库

现在你已经安装了 SQLite 数据库，接下来我们创建首个数据库。在命令行窗口中输入如下命令来创建一个名为  test.db 的数据库。

```txt
sqlite3 test.db
```
创建表：

```txt
sqlite> create table mytable(id integer primary key, value text);
```
2 columns were created. 

该表包含一个名为 id 的主键字段和一个名为 value 的文本字段。

注意: 最少必须为新建的数据库创建一个表或者视图，这么才能将数据库保存到磁盘中，否则数据库不会被创建。

接下来往表里中写入一些数据：

```txt
sqlite> insert into mytable(id, value) values(1, 'Micheal');
sqlite> insert into mytable(id, value) values(2, 'Jenny');
sqlite> insert into mytable(value) values('Francis');
sqlite> insert into mytable(value) values('Kerk');
```
查询数据：

```txt
sqlite> select * from test;
1|Micheal
2|Jenny
3|Francis
4|Kerk
```
设置格式化查询结果：

```txt
sqlite> .mode column;
sqlite> .header on;
sqlite> select * from test;
id          value
----------- -------------
1           Micheal
2           Jenny
3           Francis
4           Kerk
```
.mode column 将设置为列显示模式，.header 将显示列名。

修改表结构，增加列：

```txt
sqlite> alter table mytable add column email text not null '' collate nocase;;
```
创建视图：

```txt
sqlite> create view nameview as select * from mytable;
```
创建索引：

```txt
sqlite> create index test_idx on mytable(value);
```
4. 一些有用的 SQLite 命令

显示表结构：

```txt
sqlite> .schema [table]
```
获取所有表和视图：

```txt
sqlite > .tables 
```
获取指定表的索引列表：

```txt
sqlite > .indices [table ]
```
导出数据库到 SQL 文件：

```txt
sqlite > .output [filename ] 
sqlite > .dump 
sqlite > .output stdout
```
从 SQL 文件导入数据库：

```txt
sqlite > .read [filename ]
```
格式化输出数据到 CSV 格式：

```txt
sqlite >.output [filename.csv ] 
sqlite >.separator , 
sqlite > select * from test; 
sqlite >.output stdout
```
从 CSV 文件导入数据到表中：

```txt
sqlite >create table newtable ( id integer primary key, value text ); 
sqlite >.import [filename.csv ] newtable 
```
备份数据库：

```txt
/* usage: sqlite3 [database] .dump > [filename] */
sqlite3 mytable.db .dump > backup.sql
```
恢复数据库：

```txt
/* usage: sqlite3 [database ] < [filename ] */ 
sqlite3 mytable.db < backup.sql
```
参考资料
---

- http://www.oschina.net/question/12_53183