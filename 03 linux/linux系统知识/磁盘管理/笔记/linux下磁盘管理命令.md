## linux下磁盘管理的命令
```toc 
```

### 分区
fdisk
gdisk
parted
### 信息
[[l#lsblk|lsblk]]
[[b#blkid|blkid]]
df
du
### uuid
[[linux下磁盘管理命令#linux ext2 3 4设置uuid|uuidgen]]

### 数据读写
dd
shred
### 其他
partprobe
[[linux下磁盘管理命令#dumpe2fs|dumpe2fs]]
swapon
swapoff
quota
fsck ：检查分区表
badblocks
fuser
tune2fs
e2label
hdparm
[[linux下磁盘管理命令#resize2fs|resize2fs]]
![[inode,block,superblock#查看inode相关命令]]



### dumpe2fs
日志文件系统ext3/ext4　查看日志所占大小和inode  

![[Pasted image 20211012133237.png]]


<iframe 
 height=500
 width=800  
src="https://blog.csdn.net/test_soy/article/details/48182145"　
>
</iframe>



### linux ext2/3/4设置uuid


<iframe 
 height=500
 width=800  
src="https://www.cnblogs.com/yjken/p/3922063.html"　
>
</iframe>

![[Pasted image 20211101104907.png]]


### resize2fs
https://www.cnblogs.com/wj78080458/p/9927058.html

调整ext2\ext3\ext4文件系统的大小，它可以放大或者缩小没有挂载的文件系统的大小。如果文件系统已经挂载，它可以扩大文件系统的大小，前提是内核支持在线调整大小。


**-d**debug-flags

打开各种resize2fs调试特性，如果它们已经编译成二进制文件的话。调试标志应该通过从以下列表中添加所需功能的数量来计算：

2，调试块重定位。

4，调试iNode重定位。

8，调试移动inode表。

**-f**

强制执行，覆盖一些通常强制执行的安全检查。

**-F**

执行之前，刷新文件系统的缓冲区

**-M**

将文件系统缩小到最小值

**-p**

显示已经完成任务的百分比

**-P**

显示文件系统的最小值

**-S**RAID-stride

resize2fs程序将启发式地确定在创建文件系统时指定的RAID步长。此选项允许用户显式地指定RAID步长设置，以便由resize2fs代替。


