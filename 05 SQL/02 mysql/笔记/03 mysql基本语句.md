## 03 mysql基本语句

```toc
```
## show 基本用法
```mysql
show databases; 		   查看所有数据库
show databases\G; 		   查看所有数据库  转换为行显示
show schemas;			   查看所有数据库
show tables;			   查看所有表
show variables;			   查看所有变量
show variables like '%char%';			   查看包含char的变量
show variables like 'autocommit';		   查看autocommit变量
show engines;			   查看存储引擎
show status;			   查看数据库状态

SHOW CREATE DATABASE	   显示创建数据库的MySQL语句；  
SHOW CREATE TABLE 		   显示创建表的MySQL语句；  
SHOW GRANTS 			   用来显示授予用户（所有用户或特定用户）的安全权限；  

SHOW ERRORS				   用来显示服务器错误消息
SHOW WARNINGS			   用来显示服务器警告消息

可以通过help show;查看帮助
```


## 库操作
### 创建
```mysql
创建 da02 库
mysql> create database da02;

创建库并指定默认字符集
mysql> create database da02 default charset gbk;

创建库 如果存在不报错(if not exists)
mysql> create database if not exists da02 default character set utf8;
```
### 查看创建库语句
```mysql
mysql> show create database da01;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| da01     | CREATE DATABASE `da01` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.00 sec)

/*!40100 DEFAULT CHARACTER SET latin1 */  
这里是默认添加的语句 当执行create database da01时。
实际上执行的是 create database da01 DEFAULT CHARACTER SET latin1；
```

这里的latin1 又是怎么来的呢？
查看一下mysql库的默认字符集
```mysql
mysql> show variables like '%char%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_database   | latin1 					      |
| character_set_filesystem | binary                           |
| character_set_results    | utf8                             |
| character_set_server     | latin1                           |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysql/share/charsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.01 sec)
```
那这里又是怎么设置的呢？
实际上是通过[[02-2 mysql数据库物理文件介绍(日志，存储文件以及配置文件说明)#mysql配置文件#my cnf|my.cnf]]设置的。

### 更改数据库信息
```mysql
更改db01默认字符集
alter database db01 default character set gbk;
alter database db01 default charset utf8;
```

### 删除数据库
```mysql
drop database 库名
drop database db01;
```


## 表操作
### 创建表
create table 表名(字段 数据类型,字段 数据类型(字符长度) 约束条件 .....);
示例：
```mysql
创建 t1 表
mysql> create table t1(id int(5) key,name varchar(10) not null);
```

### 查看表结构
desc t1;
describe t1;
```mysql
mysql> desc t1;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(5)      | NO   | PRI | NULL    |       |
| name  | varchar(10) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.01 sec)

```
### 查看创建表语句
```mysql 
mysql> show create table t1;
+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                           |
+-------+----------------------------------------------------------------------------------------------------------------------------------------+
| t1    | CREATE TABLE `t1` (
  `id` int(5) NOT NULL,
  `name` varchar(10) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+----------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### 更新表属性信息(alter table)
```mysql
增加一列成为第一列 
alter table t1 add sex int first;

在id后面增加一列id2
alter table t1 add id2 int after id;

删除id2列
alter table t1 drop id2;

修改列名和数据类型 id改为ID 类型int 改为bigint
alter table t1 change id ID bigint;

修改列的数据类型 
alter table t1 modify ID int;

修改表的存储引擎
alter table t2 engine MyISAM;

修改表的默认字符集
alter table t2 default charset=utf8;


```
### 重命名或移动表
```mysql
移动并重命名表
rename table db01.t1 to db02.t11;
或者
alter table db01.t1 rename db02.t11;

重命名
rename table t1 to t11;
alter table t1 rename t11;
```
### 删除表
```mysql
drop table 表名；
```
### 增加记录
```mysql
insert into 表名 set 字段1=xx 字段2=xx;
insert into t1 set id=1,name='aaa';

insert into 表名 values (值1，值2)，(值1，值2);
insert into t1 values(2,'bbb'),(3,'ccc');

insert into 表名 (指定字段1，指定字段2)  values (值1，值2)，(值1，值2);
insert into t1(id,name) values(2,'bbb'),(3,'ccc');
insert into t1(name) values('bbb'),('ccc');


insert into t3 select * from t1; (t3 和t1 的表结构完全相同)
insert into t4 (id,name) select * from t1;(t4的id,name字段数据来源于t1)
```

### 删除记录
```mysql
delete from 表名;
delete from t1;

delete from 表名 where 条件;
delete from t1 where id=1;

truncate 表名
truncate t1;  //truncate 不能添加where条件 只能删除所有记录
```

### 更新记录
```mysql
update 表名 set 字段1=新值，字段2=新值,... where 条件；
update t1 set id=10 ,name=a10 where id=1;

```


### delete/truncate/drop 区别
![[Pasted image 20220708180501.png]]


## 用户管理

## 参考链接
 [MySQL中show语法使用总结](https://www.cnblogs.com/saneri/p/6963583.html)
 [MySQL查看表结构命令](http://c.biancheng.net/view/7199.html#:~:text=%E5%9C%A8%20MySQL%20%E4%B8%AD%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%20DESCRIBE%20%E5%92%8C%20SHOW,CREATE%20TABLE%20%E5%91%BD%E4%BB%A4%E6%9D%A5%E6%9F%A5%E7%9C%8B%E6%95%B0%E6%8D%AE%E8%A1%A8%E7%9A%84%E7%BB%93%E6%9E%84%E3%80%82%20DESCRIBE%EF%BC%9A%E4%BB%A5%E8%A1%A8%E6%A0%BC%E7%9A%84%E5%BD%A2%E5%BC%8F%E5%B1%95%E7%A4%BA%E8%A1%A8%E7%BB%93%E6%9E%84%20DESCRIBE%2FDESC%20%E8%AF%AD%E5%8F%A5%E4%BC%9A%E4%BB%A5%E8%A1%A8%E6%A0%BC%E7%9A%84%E5%BD%A2%E5%BC%8F%E6%9D%A5%E5%B1%95%E7%A4%BA%E8%A1%A8%E7%9A%84%E5%AD%97%E6%AE%B5%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%8C%85%E6%8B%AC%E5%AD%97%E6%AE%B5%E5%90%8D%E3%80%81%E5%AD%97%E6%AE%B5%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E3%80%81%E6%98%AF%E5%90%A6%E4%B8%BA%E4%B8%BB%E9%94%AE%E3%80%81%E6%98%AF%E5%90%A6%E6%9C%89%E9%BB%98%E8%AE%A4%E5%80%BC%E7%AD%89%EF%BC%8C%E8%AF%AD%E6%B3%95%E6%A0%BC%E5%BC%8F%E5%A6%82%E4%B8%8B%EF%BC%9A)
[MySQL删除表操作(delete、truncate、drop的区别)](https://blog.csdn.net/z_ryan/article/details/81913481)