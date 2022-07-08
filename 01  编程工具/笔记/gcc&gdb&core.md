## gcc

## gdb
## core
关于gdb调试core文件 参考以下链接：
- https://cloud.tencent.com/developer/article/1177442
- https://www.cnblogs.com/dongzhiquan/archive/2012/01/20/2328355.html

### 什么是core文件

当程序运行过程中出现Segmentation fault (core dumped)错误时，程序停止运行，并产生core文件。core文件是程序运行状态的内存映象。
使用gdb调试core文件，可以帮助我们快速定位程序出现段错误的位置。
***当然，可执行程序编译时应加  -g 编译选项，生成调试信息。***

当程序访问的内存超出了系统给定的内存空间，就会产生Segmentation fault (core dumped)，因此，段错误产生的情况主要有： 
（1）访问不存在的内存地址； 
（2）访问系统保护的内存地址； 
（3）数组访问越界等。

core dumped又叫核心转储, 当程序运行过程中发生异常, 程序异常退出时, 由操作系统把程序当前的内存状况存储在一个core文件中, 叫core dumped。

core意指core memory，用线圈做的内存。如今 ，半导体工业澎勃发展，已经没有人用 core memory 了，不过，在许多情况下，人们还是把记忆体叫作 core 。

***一句话简介就是：在一个程序崩溃时，它一般会在指定目录下生成一个core文件。core文件仅仅是一个内存映象(同时加上调试信息)，主要是用来调试的。***

### 控制core文件是否生成

（1）使用ulimit -c命令可查看core文件的生成开关。若结果为0，则表示关闭了此功能，不会生成core文件。

（ 2） 使用ulimit -c filesize命令，可以限制core文件的大小（filesize的单位为KB）。如果生成的信息超过此大小，将会被裁剪，最终生成一个不完整的core文件。在调试此core文 件的时候，gdb会提示错误。比如：ulimit -c 1024。

（3）使用ulimit -c unlimited，则表示core文件的大小不受限制。

在终端通过命令`ulimit -c unlimited`只是临时修改，重启后无效 ，要想永久修改有三种方式： 
- 在/etc/rc.local 中增加一行 ulimit -c unlimited 
- 在/etc/profile 中增加一行 ulimit -c unlimited 
- 在/etc/security/limits.conf最后增加如下两行记录：

```bash
@root soft core unlimited
@root hard core unlimited
```

### core文件的名称和生成路径

core默认的文件名称是core.pid，pid指的是产生段错误的程序的进程号。 默认路径是产生段错误的程序的当前目录。

如果想修改core文件的名称和生成路径，相关的配置文件为：
***
(1) /proc/sys/kernel/core_uses_pid：**控制产生的core文件的文件名中是否添加pid作为扩展，如果添加则文件内容为1，否则为0。
修改文件命令： **echo “1” > /proc/sys/kernel/core_uses_pid**  
或者  
**sysctl -w kernel.core_uses_pid=1 kernel.core_uses_pid = 1**
***
(2) **/proc/sys/kernel/core_pattern：**可以设置格式化的core文件保存的位置和文件名，比如原来文件内容是core-%e。
可以这样修改: echo “/corefile/core-%e-%p-%t” > /proc/sys/kernel/core_pattern 将会控制所产生的core文件会存放到/corefile目录下，产生的文件名为：core-命令名-pid-时间戳。
修改文件： **echo “/corefile/core-%e-%p-%t” > /proc/sys/kernel/core_pattern**  
或者：  
**sysctl -w kernel.core_pattern=/corefile/core-%e-%p-%t kernel.core_pattern = /corefile/core-%e-%p-%t**
***
(3) **在/etc目录中的sysctl.conf文件中最后添加如下内容：
kernel.core_pattern=/corefile/core.%e.%p.%t（生成的core文件需要放置的路径及文件命名格式）
在添加完成后保存并退出，使用source ./sysctl.conf执行sysctl.conf文件，使得设置立即生效**
***
**以下是参数列表:**
- %p - insert pid into filename 添加pid 
- %u - insert current uid into filename 添加当前uid 
- %g - insert current gid into filename 添加当前gid 
- %s - insert signal that caused the coredump into the filename 添加导致产生core的信号
-  %t - insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间 
-  %h - insert hostname where the coredump happened into filename 添加主机名 
-  %e - insert coredumping executable name into filename 添加命令名。 
一般情况下，无需修改，按照默认的方式即可。

ps: 这三种方式我并没有进行实验，只是摘抄了网上的资料 而且对于前两种方式我怀疑是否能够永久生效，因为据我所知 proc目录应该是运行在内存的proc文件系统，关机后是否能保持设置这一点值得怀疑。

### gdb调试core文件的步骤

使用gdb调试core文件来查找程序中出现段错误的位置时，要注意的是可执行程序在编译的时候需要加上-g编译命令选项。

gdb调试core文件的步骤常见的有如下几种，推荐第一种。

**具体步骤一：** （1）启动gdb，进入core文件，命令格式：**gdb [exec file] [core file]**。 用法示例：gdb ./test test.core。

（2）在进入gdb后，查找段错误位置：**where或者bt** 用法示例：

可以定位到源程序中具体文件的具体位置，出现了段错误。

**具体步骤二：** （1）启动gdb，进入core文件，命令格式：**gdb –core=[core file]**。 用法示例：gdb –core=test.core。

（2）在进入gdb后，指定core文件对应的符号表，命令格式：**file [exec file]** . 用法示例：

（3）查找段错误位置：**where或者bt**。 用法示例：


**具体步骤三：** （1）启动gdb，进入core文件，命令格式：**gdb -c [core file]**。 用法示例：gdb -core test.core。 （2）其它步骤同步骤二。
