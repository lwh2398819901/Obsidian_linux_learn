## 02 mysql体系结构

```toc
```

## mysql体系结构

mysql是一个单进程，多线程的数据库。
oracle 是多进程的数据库。? (存疑 未验证)

![[Pasted image 20220706151023.png]]

![[Pasted image 20220706162558.png]]

### 1. 连接层

![[Pasted image 20220706162241.png]]

应用程序通过接口（如 ODBC、JDBC）来连接 MySQL，最先连接处理的是连接层。连接层包括通信协议、线程处理、用户名密码认证 3 部分。

- 通信协议负责检测客户端版本是否兼容 MySQL 服务端。
- 线程处理是指每一个连接请求都会分配一个对应的线程，相当于一条 SQL 对应一个线程，一个线程对应一个逻辑 CPU，在多个逻辑 CPU 之间进行切换。
- 密码认证用来验证用户创建的账号、密码，以及 host 主机授权是否可以连接到 MySQL 服务器。

Connection Pool（连接池）属于连接层。由于每次建立连接都需要消耗很多时间，连接池的作用就是将用户连接、用户名、密码、权限校验、线程处理等需要缓存的需求缓存下来，下次可以直接用已经建立好的连接，提升服务器性能。

### 2. SQL层

![[Pasted image 20220706162330.png]]

SQL 层是 MySQL 的核心，MySQL 的核心服务都是在这层实现的。主要包含权限判断、查询缓存、解析器、预处理、查询优化器、缓存和执行计划。

- 权限判断可以审核用户有没有访问某个库、某个表，或者表里某行数据的权限。

- 查询缓存通过 Query Cache 进行操作，如果数据在 Query Cache 中，则直接返回结果给客户端，不必再进行查询解析、优化和执行等过程。

- 查询解析器针对 SQL 语句进行解析，判断语法是否正确。

- 预处理器对解析器无法解析的语义进行处理。

- 查询优化器对 SQL 进行改写和相应的优化，并生成最优的执行计划，就可以调用程序的 API 接口，通过存储引擎层访问数据。

Management Services & Utilities、SQL Interface、Parser、Optimizer 和 Caches & Buffers 属于 SQL 层，详细说明如下表所示。

| 名称                               | 说明                                                                                                                                                                                                                                                                   |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Management Services & Utilities  | MySQL 的系统管理和控制工具，包括备份恢复、MySQL 复制、集群等。                                                                                                                                                                                                                                |
| SQL Interface（SQL 接口）            | 用来接收用户的 SQL 命令，返回用户需要查询的结果。例如 SELECT FROM 就是调用 SQL Interface。                                                                                                                                                                                                        |
| Parser（查询解析器）                    | 在 SQL 命令传递到解析器的时候会被解析器验证和解析，以便 MySQL 优化器可以识别的数据结构或返回 SQL 语句的错误。                                                                                                                                                                                                      |
| Optimizer（查询优化器）                 | SQL 语句在查询之前会使用查询优化器对查询进行优化，同时验证用户是否有权限进行查询，缓存中是否有可用的最新数据。它使用“选取-投影-连接”策略进行查询。<br>例如 `SELECT id, name FROM student WHERE gender = "女";`语句中，SELECT 查询先根据 WHERE 语句进行选取，而不是将表全部查询出来以后再进行 gender 过滤。SELECT 查询先根据 id 和 name 进行属性投影，而不是将属性全部取出以后再进行过滤，将这两个查询条件连接起来生成最终查询结果。 |
| Caches & Buffers（查询缓存)           | 如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。这个缓存机制是由一系列小缓存组成的，比如表缓存、记录缓存、key 缓存、权限缓存等。                                                                                                                                                                                         |

### 3. 存储引擎层

Pluggable Storage Engines 属于存储引擎层。存储引擎层是 MySQL 数据库区别于其他数据库最核心的一点，也是 MySQL 最具特色的一个地方。主要负责 MySQL 中数据的存储和提取。

因为在关系数据库中，数据的存储是以表的形式存储的，所以存储引擎也可以称为表类型（即存储和操作此表的类型）。

需要了解mysql存储引擎的特性，因为mysql支持对每一个表使用不同的存储引擎。

需要特别关注__MyISAM__和__InnoDB__，5.5之后的版本默认是__InnoDB__；

| 名称         | 说明                                                                                                                                                                                  |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MyISAM** | MyISAM存储引擎提供高速存储和检索，以及全文搜索能力。<br>MyISAM在所有MySQL版本里被支持；<br>不支持事务处理；<br>它是MySQL的默认的存储引擎(5.5之前)；                                                                                       |
| MEMORY     | MEMORY存储引擎，别称HEAP存储引擎；<br>提供“内存中”表，将数据存储在内存中。<br>MEMORY存储引擎不支持事务处理；<br>注释：MEMORY存储引擎正式地被确定为HEAP引擎。                                                                                  |
| MERGE      | MRG_MYISAM存储引擎，别名MERGE；<br>MRG_MYISAM存储引擎允许集合将被处理同样的MyISAM表作为一个单独的表。<br>MRG_MYISAM存储引擎不支持事务处理；<br>MySQL的所有版本都支持MRG_MYISAM存储引擎；                                                      |
| ISAM       | Obsolete storage engine, now replaced by MyISAM                                                                                                                                     |
| MRG_ISAM   | Obsolete storage engine, now replaced by MERGE                                                                                                                                      |
| **InnoDB** | InnoDB存储引擎，别名INNOBASE；<br>提供事务安全表；<br>MySQL的所有版本都支持InnoDB存储引擎；<br>它支持事务处理；                                                                                                          |
| BDB        | BDB存储引擎，别名BERKELEYDB；<br>BDB存储引擎提供事务安全表；<br>mysql 5.1以下版本才支持此存储引擎；                                                                                                                  |
| EXAMPLE    | EXAMPLE存储引擎是一个“存根”引擎，它不做什么。你可以用这个引擎创建表，但没有数据被存储于其中或从其中检索。这个引擎的目的是服务，在MySQL源代码中的一个例子，它演示说明如何开始编写新存储引擎。同样，它的主要兴趣是对开发者。                                                                |
| NDB        | NDB存储引擎，别名NDBCLUSTER；NDB Cluster是被MySQL Cluster用来实现分割到多台计算机上的表的存储引擎。<br>它在MySQL-Max 5.1二进制分发版里提供。这个存储引擎当前只被Linux, Solaris, 和Mac OS X 支持。在未来的MySQL分发版中，我们想要添加其它平台对这个引擎的支持，包括Windows。 |
| ARCHIVE    | ARCHIVE存储引擎被用来无索引地，非常小地覆盖存储的大量数据。                                                                                                                                                   |
| CSV        | CSV存储引擎把数据以逗号分隔的格式存储在文本文件中。CSV存储引擎不支持事物处理；                                                                                                                                          |
| BLACKHOLE  | BLACKHOLE存储引擎接受但不存储数据，并且查询也总是返回一个空集；/dev/null storage engine (anything you write to it disappears)                                                                                  |
| FEDERATED  | FEDERATED存储引擎把数据存在远程数据库中。在MySQL 5.1中，它只和MySQL一起工作，使用MySQL C Client API。在未来的分发版中，我们想要让它使用其它驱动器或客户端连接方法连接到另外的数据源。FEDERATED存储引擎支持事务处理；                                                 |

#### **InnoDB**

InnoDB是一个健壮的事务型存储引擎，这种存储引擎已经被很多互联网公司使用，为用户操作非常大的数据存储提供了一个强大的解决方案。

InnoDB还引入了行级锁定和外键约束，在以下场合下，使用InnoDB是最理想的选择：

- 更新密集的表。 InnoDB存储引擎特别适合处理多重并发的更新请求。

- 事务。 InnoDB存储引擎是支持事务的标准MySQL存储引擎。

- 自动灾难恢复。 与其它存储引擎不同，InnoDB表能够自动从灾难中恢复。

- 外键约束。 MySQL支持外键的存储引擎只有InnoDB。

- 支持自动增加列AUTO_INCREMENT属性。

- 从5.7开始innodb存储引擎成为默认的存储引擎。

一般来说，如果需要事务支持，并且有较高的并发读取频率，InnoDB是不错的选择。

#### **MyISAM**

MyISAM表是独立于操作系统的，这说明可以轻松地将其从Windows服务器移植到Linux服务器。

每当我们建立一个MyISAM引擎的表时，就会在本地磁盘上建立三个文件，文件名就是表名。

例如，我建立了一个MyISAM引擎的tb_Demo表，那么就会生成以下三个文件：

- tb_demo.frm，存储表定义。
- tb_demo.MYD，存储数据。
- tb_demo.MYI，存储索引。

MyISAM表无法处理事务，这就意味着有事务处理需求的表，不能使用MyISAM存储引擎。MyISAM存储引擎特别适合在以下几种情况下使用：

1. 选择密集型的表。 MyISAM存储引擎在筛选大量数据时非常迅速，这是它最突出的优点。

2. 插入密集型的表。 MyISAM的并发插入特性允许同时选择和插入数据。

由此看来，MyISAM存储引擎很适合管理服务器日志数据。

#### **MRG_MYISAM**

MRG_MyISAM存储引擎是一组MyISAM表的组合，老版本叫 MERGE 其实是一回事儿。

这些MyISAM表结构必须完全相同，尽管其使用不如其它引擎突出，但是在某些情况下非常有用。

说白了，Merge表就是几个相同MyISAM表的聚合器；Merge表中并没有数据，对Merge类型的表可以进行查询、更新、删除操作，这些操作实际上是对内部的MyISAM表进行操作。

#### **Merge存储引擎的使用场景**

对于服务器日志这种信息，一般常用的存储策略是将数据分成很多表，每个名称与特定的时间段相关。

例如，可以用12个相同的表来存储服务器日志数据，每个表用对应各个月份的名字来命名。当有必要基于所有12个日志表的数据来生成报表，这意味着需要编写并更新多表查询，以反映这些表中的信息。与其编写这些可能出现错误的查询，不如将这些表合并起来使用一条查询，之后再删除Merge表，而不影响原来的数据，删除Merge表只是删除Merge表的定义，对内部的表没有任何影响。

#### **Merge存储引擎的使用方法**

- ENGINE=MERGE， 指明使用MERGE引擎，其实是跟MRG_MyISAM一回事儿，也是对的，在MySQL 5.7已经看不到MERGE了。
- UNION=(t1, t2)， 指明了MERGE表中挂接了些哪表，可以通过alter table的方式修改UNION的值，以实现增删MERGE表子表的功能。比如：

```sql
alter table tb_merge engine=merge union(tb_log1) insert_method=last;
```

- INSERT_METHOD=LAST， INSERT_METHOD指明插入方式，取值可以是： `0` 不允许插入； `FIRST` 插入到UNION中的第一个表； `LAST` 插入到UNION中的最后一个表。

- MERGE表及构成MERGE数据表结构的各成员数据表必须具有完全一样的结构。 每一个成员数据表的数据列必须按照同样的顺序定义同样的名字和类型，索引也必须按照同样的顺序和同样的方式定义。

#### **MEMORY**

使用MySQL Memory存储引擎的出发点是速度，为得到最快的响应时间，采用的逻辑存储介质是系统内存。

虽然在内存中存储表数据确实会提供很高的性能，但当mysqld守护进程崩溃时，所有的Memory数据都会丢失。

获得速度的同时也带来了一些缺陷。

它要求存储在Memory数据表里的数据使用的是长度不变的格式，这意味着不能使用BLOB和TEXT这样的长度可变的数据类型。VARCHAR是一种长度可变的类型，但因为它在MySQL内部当做长度固定不变的CHAR类型，所以可以使用。

一般在以下几种情况下使用Memory存储引擎：

- 目标数据较小，而且被非常频繁地访问。 在内存中存放数据，所以会造成内存的使用，可以通过参数max_heap_table_size控制Memory表的大小，设置此参数，就可以限制Memory表的最大大小。
- 如果数据是临时的，而且要求必须立即可用，那么就可以存放在内存表中。
- 存储在Memory表中的数据如果突然丢失，不会对应用服务产生实质的负面影响。
- Memory同时支持散列索引和B树索引。

B树索引优于散列索引的是，可以使用部分查询和通配查询，也可以使用<、>和>=等操作符方便数据挖掘。

散列索引进行“相等比较”非常快，但是对“范围比较”的速度就慢多了，因此散列索引值适合使用在=和<>的操作符中，不适合在<或>操作符中，也同样不适合用在order by子句中。

#### **CSV**

CSV 存储引擎是基于 CSV 格式文件存储数据。

- CSV 存储引擎因为自身文件格式的原因，所有列必须强制指定 NOT NULL 。
- CSV 引擎也不支持索引，不支持分区。
- CSV 存储引擎也会包含一个存储表结构的 .frm 文件，还会创建一个 .csv 存储数据的文件，还会创建一个同名的元信息文件，该文件的扩展名为 .CSM ，用来保存表的状态及表中保存的数据量。
- 每个数据行占用一个文本行。

因为 csv 文件本身就可以被Office等软件直接编辑，保不齐就有不按规则出牌的情况，如果出现csv 文件中的内容损坏了的情况，也可以使用 CHECK TABLE 或者 REPAIR TABLE 命令检查和修复。

#### **ARCHIVE**

Archive是归档的意思，在归档之后很多的高级功能就不再支持了，仅仅支持最基本的插入和查询两种功能。

在MySQL 5.5版以前，Archive是不支持索引，但是在MySQL 5.5以后的版本中就开始支持索引了。

Archive拥有很好的压缩机制，它使用zlib压缩库，在记录被请求时会实时压缩，所以它经常被用来当做仓库使用。

#### **BLACKHOLE**

MySQL在5.x系列提供了Blackhole引擎–“黑洞”，其作用正如其名字一样：任何写入到此引擎的数据均会被丢弃掉， 不做实际存储；Select语句的内容永远是空。

这和Linux中的 `/dev/null` 文件完成的作用完全一致。

那么， 一个不能存储数据的引擎有什么用呢？

Blackhole虽然不存储数据，但是MySQL还是会正常的记录下Binlog，而且这些Binlog还会被正常的同步到Slave上，可以在Slave上对数据进行后续的处理。

这样对于在Master上只需要Binlog而不需要数据的场合下，balckhole就有用了。

BlackHole 还可以用在以下场景

- 验证语法 验证dump file语法的正确性
- 检测负载 以使用blackhole引擎来检测binlog功能所需要的额外负载
- 检测性能 由于blackhole性能损耗极小，可以用来检测除了存储引擎这个功能点之外的其他MySQL功能点的性能。

#### **PERFORMANCE_SCHEMA**

主要用于收集数据库服务器性能参数。

MySQL用户是不能创建存储引擎为PERFORMANCE_SCHEMA的表，一般用于记录binlog做复制的中继。

在这里有官方的一些介绍: [MySQL Performance Schema](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/refman/5.6/en/performance-schema.html)

#### **FEDERATED**

主要用于访问其它远程MySQL服务器一个代理，它通过创建一个到远程MySQL服务器的客户端连接，并将查询传输到远程服务器执行，而后完成数据存取；在MariaDB上的实现是FederatedX。

#### **其他**

这里列举一些其它数据库提供的存储引擎，OQGraph、SphinxSE、TokuDB、Cassandra、CONNECT、SQUENCE。

#### **常用引擎对比**

不同存储引起都有各自的特点，为适应不同的需求，需要选择不同的存储引擎，所以首先考虑这些存储引擎各自的功能和兼容。
![[Pasted image 20220706161222.png]]

**存储引擎相关操作命令**\
**查看存储引擎**\
使用“SHOW VARIABLES LIKE '%storage_engine%';” 命令在mysql系统变量搜索默认设置的存储引擎，输入语句如下：

```text
mysql> SHOW VARIABLES LIKE '%storage_engine%'; 
+----------------------------------+---------+ 
| Variable_name                    | Value   |
|----------------------------------+---------| 
| default_storage_engine           | InnoDB  | 
| default_tmp_storage_engine       | InnoDB  | 
| disabled_storage_engines         |         | 
| internal_tmp_disk_storage_engine | InnoDB  | 
+----------------------------------+---------+ 
4 rows in set 
Time: 0.005s
```

使用`SHOW ENGINES;`命令显示安装以后可用的所有的支持的存储引擎和默认引擎，后面带上 `\G` 可以列表输出结果，你可以尝试一下`SHOW ENGINES\G;`。

```text
mysql> SHOW ENGINES; 
+--------------------+---------+--------------------------------------+-------------+--------+-----------+ 
| Engine             | Support | Comment                              | Transactions| XA     | Savepoints| 
|--------------------+---------+--------------------------------------+-------------+--------+-----------| 
| InnoDB             | DEFAULT | Supports transactions,               | YES         | YES    | YES       | 
|                    |         | row-level locking, and foreign keys  |             |        |           | 
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables| NO          | NO     | NO        | 
| MEMORY             | YES     | Hash based, stored in memory, useful | NO          | NO     | NO        | 
|                    |         | for temporary tables                 |             |        |           | 
| BLACKHOLE          | YES     | /dev/null storage engine (anything   | NO          | NO     | NO        | 
|                    |         | you write to it disappears)          |             |        |           | 
| MyISAM             | YES     | MyISAM storage engine                | NO          | NO     | NO        |
| CSV                | YES     | CSV storage engine                   | NO          | NO     | NO        | 
| ARCHIVE            | YES     | Archive storage engine               | NO          | NO     | NO        | 
| PERFORMANCE_SCHEMA | YES     | Performance Schema                   | NO          | NO     | NO        | 
| FEDERATED          | NO      | Federated MySQL storage engine       | <null>      | <null> | <null>    | 
+--------------------+---------+--------------------------------------+-------------+--------+-----------+
```

由上面命令输出，可见当前系统的默认数据表类型是InnoDB。当然，我们可以通过修改数据库配置文件中的选项，设定默认表类型。

**设置存储引擎**\
对上面数据库存储引擎有所了解之后，你可以在[[02-2 mysql数据库物理文件介绍(日志，存储文件以及配置文件说明)#mysql配置文件#my cnf|my.cnf]]配置文件中设置你需要的存储引擎，这个参数放在 [mysqld] 这个字段下面的 default_storage_engine 参数值，例如下面配置的片段

```text
[mysqld] 
default_storage_engine=CSV
```

**在创建表的时候，对表设置存储引擎**\
例如：

```text
 CREATE TABLE `user` (
   `id`     int(100) unsigned NOT NULL AUTO_INCREMENT,
   `name`   varchar(32) NOT NULL DEFAULT '' COMMENT '姓名',
   `mobile` varchar(20) NOT NULL DEFAULT '' COMMENT '手机',
   PRIMARY KEY (`id`) 
 )ENGINE=InnoDB;
```

在创建用户表 user 的时候，SQL语句最后 ENGINE=InnoDB 就是设置这张表存储引擎为 InnoDB。

### **如何选择合适的存储引擎**

可以根据上文中的[常用引擎对比](https://zhuanlan.zhihu.com/p/59056833/edit#%E5%B8%B8%E7%94%A8%E5%BC%95%E6%93%8E%E5%AF%B9%E6%AF%94)来选择你使用的存储引擎。\
使用哪种引擎需要根据需求灵活选择，一个数据库中多个表可以使用不同的引擎以满足各种性能和实际需求。\
使用合适的存储引擎，将会提高整个数据库的性能。\
下面提供几个选择标准，然后按照标准，根据实际情况，选择对应的存储引擎即可：

1. 是否需要支持事务；
2. 是否需要使用热备；
3. 崩溃恢复，能否接受崩溃；
4. 是否需要外键支持；
5. 存储的限制；
6. 对索引和缓存的支持。

### 4. 文件系统层

文件系统层主要是将数据库的数据存储在操作系统的文件系统之上，并完成与存储引擎的交互。



## 参考链接
[mysql 体系结构](http://c.biancheng.net/view/7939.html)

