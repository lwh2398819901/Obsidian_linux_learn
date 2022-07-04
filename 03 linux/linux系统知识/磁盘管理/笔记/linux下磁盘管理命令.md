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
[[d#dumpe2fs|dumpe2fs]]
swapon
swapoff
quota
fsck ：检查分区表
badblocks
fuser
tune2fs
e2label
hdparm
[[r#resize2fs|resize2fs]]
![[inode,block,superblock#查看inode相关命令]]



### linux ext2/3/4设置uuid


<iframe 
 height=500
 width=800  
src="https://www.cnblogs.com/yjken/p/3922063.html"　
>
</iframe>

![[Pasted image 20211101104907.png]]




