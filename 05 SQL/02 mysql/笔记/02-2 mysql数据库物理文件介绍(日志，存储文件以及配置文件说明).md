## 02-2 mysql数据库物理文件介绍(日志，存储文件以及配置文件说明)

```toc
```

## mysql日志

### 临时开启日志方式
可以通过启动数据库时候，传参数方式 临时开启日志。具体方法自行百度。

### 日志类型简介
| 日志类型     | 介绍 | 默认值 |
| ------------ | ---- | ------ |
| 错误日志     |   mysqld 启动和停止，以及服务器在运行过程中发生的错误及警告相关信息    |    开启    |
| 一般查询日志 |  建立客户端链接，从客户端收到的所有语句    |      关闭  |
| 二进制日志   |  更改数据的语句，或者主从复制    |      关闭  |
| 中继日志     |   主从复制架构中的从服务器上接收主服务更改   |   关闭     |
| 慢查询日志   | 记录执行时间超过 long_query_time 这个变量定义的时长的查询语句     |     关闭   |
| [[基础知识#DDL\|DDL]] 日志（元数据日志） | [[基础知识#DDL\|DDL]] 语句执行的元数据操作    |        |

#### 错误日志

错误日志记录着 mysqld 启动和停止，以及服务器在运行过程中发生的错误及警告相关信息。当数据库意外宕机或发生其他错误时，我们应该去排查错误日志。

log_error 参数控制错误日志是否写入文件及文件名称，默认情况下，错误日志被写入终端标准输出stderr。当然，推荐指定 log_error 参数，自定义错误日志文件位置及名称。

```
# 指定错误日志位置及名称
vim /etc/my.cnf 
[mysqld] 
log_error = /data/mysql/logs/error.log

相关配置变量说明：
log_error={1 | 0 | /PATH/TO/ERROR_LOG_FILENAME}
定义错误日志文件。作用范围为全局或会话级别，属非动态变量。
```

#### 一般查询日志

一般查询日志又称通用查询日志，是 MySQL 中记录最详细的日志，该日志会记录 mysqld 所有相关操作，当 clients 连接或断开连接时，服务器将信息写入此日志，并记录从 clients 收到的每个 SQL 语句。当你怀疑 client 中的错误并想要确切知道 client 发送给mysqld的内容时，通用查询日志非常有用。

默认情况下，general log 是关闭的，开启通用查询日志会增加很多磁盘 I/O， 所以如非出于调试排错目的，不建议开启通用查询日志。

```bash
# general log相关配置
vim /etc/my.cnf 
[mysqld]
general_log = 0 //默认值是0，即不开启，可设置为1
general_log_file = /data/mysql/logs/general.log //指定日志位置及名称
```

#### 慢查询日志

慢查询日志是用来记录执行时间超过 long_query_time 这个变量定义的时长的查询语句。通过慢查询日志，可以查找出哪些查询语句的执行效率很低，以便进行优化。

与慢查询相关的几个参数如下：

1. slow_query_log ：是否启用慢查询日志，默认为0，可设置为0，1。
2. slow_query_log_file ：指定慢查询日志位置及名称，默认值为host_name-slow.log，可指定绝对路径。
3. long_query_time ：慢查询执行时间阈值，超过此时间会记录，默认为10，单位为s。
4. log_output ：慢查询日志输出目标，默认为file，即输出到文件。

默认情况下，慢查询日志是不开启的，一般情况下建议开启，方便进行慢SQL优化。在配置文件中可以增加以下参数：

```bash
# 慢查询日志相关配置，可根据实际情况修改
vim /etc/my.cnf 
[mysqld] 
slow_query_log = 1
slow_query_log_file = /data/mysql/logs/slow.log
long_query_time = 3
log_output = FILE

```

#### 二进制日志

关于二进制日志，前面有篇文章做过介绍。它记录了数据库所有执行的[[基础知识#DDL\|DDL]] 和[[基础知识#DML|DML]] 语句（除了数据查询语句select、show等），以事件形式记录并保存在二进制文件中。常用于数据恢复和主从复制。

与 binlog 相关的几个参数如下：

- log_bin ：指定binlog是否开启及文件名称。
- server_id ：指定服务器唯一ID，开启binlog 必须设置此参数。
- binlog_format ：指定binlog模式，建议设置为ROW。
- max_binlog_size ：控制单个二进制日志大小，当前日志文件大小超过此变量时，执行切换动作。
- expire_logs_days ：控制二进制日志文件保留天数，默认值为0，表示不自动删除，可设置为0~99。
-

binlog默认情况下是不开启的，不过一般情况下，建议开启，特别是要做主从同步时。

```bash
# binlog 相关配置
vim /etc/my.cnf 
[mysqld]
server-id = 1003306
log-bin = /data/mysql/logs/binlog
binlog_format = row
expire_logs_days = 15
```

#### 中继日志
中继日志用于主从复制架构中的从服务器上，从服务器的 slave 进程从主服务器处获取二进制日志的内容并写入中继日志，然后由 IO 进程读取并执行中继日志中的语句。

relay log 相关参数一般在从库设置，几个相关参数介绍如下：

- relay_log ：定义 relay log 的位置和名称。
- relay_log_purge ：是否自动清空不再需要中继日志，默认值为1(启用)。
- relay_log_recovery ：当 slave 从库宕机后，假如 relay log 损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的 relay log ，并且重新从 master 上获取日志，这样就保证了 relay log 的完整性。默认情况下该功能是关闭的，将 relay_log_recovery 的值设置为1可开启此功能。

relay log 默认位置在数据文件的目录，文件名为 host_name-relay-bin，可以自定义文件位置及名称。

```bash
# relay log 相关配置,从库端设置

vim /etc/my.cnf

[mysqld]
relay_log = /data/mysql/logs/relay-bin
relay_log_purge = 1
relay_log_recovery = 1
```

## mysql数据文件
1. **.frm文件：** 保存每个数据表的元数据(meta)信息，包括表结构的定义等，.frm文件跟数据库存储引擎无关，也就是任何存储引擎的数据表都必须有.frm文件，命名方式为数据表名.frm，如user.frm. .frm文件可以用来在数据库崩溃时恢复表结构。
2. **.MYD文件：** myisam引擎专用，存放myisam表的数据信息，每个表对应一个.MYD文件。
3. **.MYI文件：** myisam引擎专用，存放myisam表的索引信息。一般查询缓存主要来源就是.MYI文件中，每个表对应一个.MYI文件。
4. **.ibd文件：** innoDB引擎存放数据和索引。每个表独占一个idb空间。
5. **.ibdata文件：** innoDB引擎存放数据和索引。共享表空间，所有表共同使用一个（或多个，自行配置），典型的是系统库的表。
6. **db.opt文件：** 每个自建的库都有一个，记录库的默认字符集和校验规则

### mysql配置文件
#### my.cnf
- ***存放位置以及读取规则  按照如下顺序读取***
1. $basedir(mysql安装目录)/my.cnf  
2. $datadir(mysql 数据目录)/my.cnf   
3. /etc/my.cnf   
4. /etc/mysql/my.cnf
5. ~/.my.cnf

- ***如何重新加载配置***
 重启mysqld服务
 systemctl restart mysql.service
- ***配置说明***
https://kalacloud.com/blog/how-to-edit-mysql-configuration-file-my-cnf-ini/


## mysql事务

默认情况下mysql自动提交事务

通过autocommit变量控制是否自动提交。
查看autocommit
```mysql
mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```
autocommit = on 表示自动提交

临时关闭自动提交
```mysql
mysql> set autocommit=OFF;
Query OK, 0 rows affected (0.00 sec)
```

永久关闭自动提交
```bash
# relay log 相关配置,从库端设置

vim /etc/my.cnf

[mysqld]
autocommit=0
or
autocommit=OFF
```

[[基础知识#DDL\|DDL]] 语句执行时，会自动提交上面未提交的事务。

手动控制事务
```bash
commit 提交
rollback 回滚
```


## 参考链接
[MySQL数据库存储引擎简介](https://zhuanlan.zhihu.com/p/59056833)
[MySQL中常见的几种日志汇总](https://www.jb51.net/article/193917.htm#:~:text=MySQL%E4%B8%AD%E5%B8%B8%E8%A7%81%E7%9A%84%E5%87%A0%E7%A7%8D%E6%97%A5%E5%BF%97%E6%B1%87%E6%80%BB%201%20%E9%94%99%E8%AF%AF%E6%97%A5%E5%BF%97%EF%BC%88errorlog%EF%BC%89%202%20%E6%85%A2%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97%EF%BC%88slow%20query%20log%EF%BC%89,3%20%E4%B8%80%E8%88%AC%E6%9F%A5%E8%AF%A2%E6%97%A5%E5%BF%97%EF%BC%88general%20log%EF%BC%89%204%20%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%97%A5%E5%BF%97%EF%BC%88binlog%EF%BC%89%205%20%E4%B8%AD%E7%BB%A7%E6%97%A5%E5%BF%97%EF%BC%88relay%20log%EF%BC%89)
