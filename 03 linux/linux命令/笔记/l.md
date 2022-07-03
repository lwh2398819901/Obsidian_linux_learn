## l
```toc
```
## lsblk
<iframe 
 height=500
 width=900  
src="https://linux.cn/article-4734-1.html"　
>
</iframe>

```bash
root@uos-PC /h/u/Desktop# lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0         7:0    0  99.3M  1 loop /snap/core/11743
loop1         7:1    0  61.9M  1 loop /snap/core20/1169
loop2         7:2    0  61.8M  1 loop /snap/core20/1081
loop3         7:3    0  99.4M  1 loop /snap/core/11993
loop4         7:4    0  99.5M  1 loop /snap/core/11798
sda           8:0    0 931.5G  0 disk 
├─sda1        8:1    0    16M  0 part 
├─sda2        8:2    0 195.3G  0 part /media/uos/2E50C0F250C0C237
└─sda3        8:3    0 736.2G  0 part /media/uos/ext4
nvme0n1     259:0    0 238.5G  0 disk 
├─nvme0n1p1 259:1    0   300M  0 part /boot/efi
├─nvme0n1p2 259:2    0   1.5G  0 part /boot
├─nvme0n1p3 259:3    0    15G  0 part /
├─nvme0n1p4 259:4    0    15G  0 part 
├─nvme0n1p5 259:5    0 181.7G  0 part /data
├─nvme0n1p6 259:6    0    14G  0 part /recovery
└─nvme0n1p7 259:7    0    11G  0 part [SWAP]
```