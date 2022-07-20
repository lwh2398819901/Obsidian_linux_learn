## 01 mysql数据库安装与登录

```toc
```

## 数据库版本

- 社区版：MySQL Community Edition (GPL)

1. 可以看做企业版本的广泛体验版，未经各个系统平台的压力测试和性能测试
2. 基于gpl协议发布，可以随意下载使用
3. 没有官方技术支持服务

- 企业版：MySQL Enterprise Edition(commercial)
  1. 提供比较全面的高级功能，管理工具以及技术支持
  2. 安全性，稳定性，可扩展性比较好
- 集群版：MySQL Cluster CGE(commercial)

## 下载与安装

mysql 官网 ：<https://www.mysql.com/>
mysql 官网安装手册:<https://dev.mysql.com/doc/relnotes/mysql/5.6/en/>
![[Pasted image 20220703205349.png]]
![[Pasted image 20220703205427.png]]
![[Pasted image 20220703212739.png]]

### 源码包安装

通过官网下载源码包后，在linux平台下编译安装
<https://dev.mysql.com/doc/refman/5.6/en/source-installation.html>

#### 先决条件

- CMAKE
- MAKE
- GCC
- perl
- 足够的内存
- ncurses库

![[Pasted image 20220704151522.png]]

#### 配置

<https://dev.mysql.com/doc/refman/5.6/en/source-configuration-options.html>
![[Pasted image 20220704151817.png]]
![[Pasted image 20220704154133.png]]

1. 编写cmake.sh文件

```bash
#!/bin/bash
cmake . \
-DCMAKE_INSTALL_PREFIX=/mysql31 \  //按照需求修改
-DMYSQL_DATADIR=/mysql31/data \    //按照需求修改
-DMYSQL_TCP_PORT=3307 \			   //按照需求修改
-DMYSQL_UNIX_ADDR=/mysql31/mysql31.sock \   //按照需求修改
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci
```

2.添加执行权限

```bash
	chmod a+x ./cmake.sh
```

3.执行配置

```bash
	./cmake.sh
```

#### 编译 && 安装

```bash
	make && make install
```

#### 后续配置

与二进制安装方式类似

### 二进制安装

#### RPM安装

#### DPKG安装

#### 基于glbc版本

安装手册：<https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html>

- MySQL 依赖于该`libaio` 库。如果未在本地安装此库，则数据目录初始化和后续服务器启动步骤将失败。如有必要，请使用适当的包管理器安装它。例如，在基于 Yum 的系统上：

  ```terminal
  $> yum search libaio  # search for info
  $> yum install libaio # install library
  ```

  或者，在基于 APT 的系统上：

  ```terminal
  $> apt-cache search libaio # search for info
  $> apt-get install libaio1 # install library
  ```

要安装和使用 MySQL 二进制发行版，命令序列如下所示：

```bash

//安装
$> groupadd mysql   							//创建mysql组（可不创建）
$> useradd -r -g mysql -s /bin/false mysql 		//创建mysql用户 并且设置为不可登录
$> mkdir /usr/local/mysql 						//创建目录
$> tar zxvf _/path/to/mysql-VERSION-OS_.tar.gz 	//解压mysql tar包
$> cp  -r ./mysql-VERSION-OS/*  /usr/local/mysql	//拷贝mysql tar包下所有文件到/usr/local/mysql下
$> chown -R mysql.mysql /usr/local/mysql   		//更改目录权限



//初始化mysql服务
确保当前系统没有/etc/my.cnf文件 有的话删除
$> cd /user/local/mysql 						//进入mysql目录
$> scripts/mysql_install_db --user=mysql 		//初始化数据库
# $> bin/mysqld_safe --user=mysql & 				//后台启动
# Next command is optional 
$> cp support-files/mysql.server /etc/init.d/mysql.server  //拷贝服务配置(uos下启动失败)
$> service mysql.server start

$> ss -naltp |grep mysql 						//查看端口是否绑定成功
$> ps aux |grep mysql 							//查看mysql是否启动



//设置密码
$> ./bin/mysqladmin -u root password 'new-password' //给mysql的 root用户设置密码
$> ./bin/mysqladmin -u root -h uos-pc password 'new-password' //给mysql的 root主机用户设置密码,注意 这里主机要通过hostname 获取名称，并且 在/etc/hosts中有记录



//安全初始化数据库
$> ./bin/mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MySQL to secure it, we'll need the current
password for the root user.  If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):  //输入上一步设置的root密码
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n   //是否修改root密码  不修改
 ... skipping.

By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y   //是否删除匿名用户  是
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n //不允许root用户远程登录  一般选是，测试时选择否
 ... skipping.

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y  //删除测试库？ 是
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y  //现在重新加载权限表？是
 ... Success!


All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

Cleaning up...


//设置mysql命令行环境变量
$> vim ~/.bashrc 
export PATH=$PATH:/usr/local/mysql/bin
$> source ~/.bashrc 
```

## 登录连接
```
//测试登录
$> mysql -u root -p
```
本地远程连接阿里云服务器上的MySQL数据库实现：<https://blog.csdn.net/hongfei568718926/article/details/107888670>

### c API链接
https://www.jianshu.com/p/8229bf719163
```c
#include <iostream>
#include <mysql/mysql.h> //mysql提供的函数接口头文件

using namespace std;

int main() {
    const char* host = "39.107.72.14"; //主机名
    const char* user = "root"; //用户名
    const char* pwd = "123"; //密码
    const char* dbName = "testDB"; //数据库名称
    int port = 3306; //端口号

    // 创建mysql对象
    MYSQL *sql = nullptr;
    sql = mysql_init(sql);
    if (!sql) {
        cout << "MySql init error!" << endl;
		return -1;
    }
    
    // 连接mysql
    sql = mysql_real_connect(sql, host, user, pwd, dbName, port, nullptr, 0);
    if (!sql) {
        cout << "MySql Connect error!" << endl;
	   return -1;
    }

    // 执行命令
    int ret = mysql_query(sql, "select * from t1");
    if (ret) {
        cout << "error!" << endl;
    } else {
        cout << "success!" << endl;
    }

    //获取结果集  
	MYSQL_RES*res;
    if (!(res = mysql_store_result(sql)))   //获得sql语句结束后返回的结果集  
    {
        printf("Couldn't get result from %s\n", mysql_error(sql));
        return -1;
    }


    //打印数据行数  
    printf("number of dataline returned: %d\n", mysql_affected_rows(sql));

    //获取字段的信息  
    char *str_field[32];  //定义一个字符串数组存储字段信息  
    for (int i = 0; i<3; i++)  //在已知字段数量的情况下获取字段名  
    {
        str_field[i] = mysql_fetch_field(res)->name;
    }
    for (int i = 0; i<3; i++)  //打印字段  
        printf("%10s\t", str_field[i]);
    printf("\n");
    //打印获取的数据  
	MYSQL_ROW column; //一个行数据的类型安全(type-safe)的表示，表示数据行的列  
    while (column = mysql_fetch_row(res))   //在已知字段数量情况下，获取并打印下一行  
    {
        printf("%10s\t%10s\t%10s\n", column[0], column[1], column[2]);  //column是列数组  
    }
    
    // 关闭mysql
    mysql_close(sql);
    return 0;
}

```

## mysql客户端工具
![[Pasted image 20220705232729.png]]
## mysqladmin
![[Pasted image 20220705232918.png]]

## mysql 启动过程
![[Pasted image 20220705234919.png]]
![[Pasted image 20220705234606.png]]

## 排错
![[Pasted image 20220705235716.png]]

### 错误日志位置
/usr/local/mysql/data/xxx.err


## 参考链接
https://blog.csdn.net/weixin_42734930/article/details/81743047