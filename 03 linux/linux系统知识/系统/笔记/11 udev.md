```toc
title: "## udev" 
```

**首先思考一个问题，为什么需要管理设备的创建以及权限？**

内核通常仅根据设备被发现的先后顺序给设备文件命名，因此___很难___在设备文件与物理硬件之间建立___稳定的对应关系___。
而根据设备的___物理属性或配置特征___创建有意义的符号链接名称或网络接口名称，就可以在物理设备与设备文件之间___建立稳定的对应关系___。
在linux中，实现在物理设备与设备文件之间__建立稳定的对应关系__的方式之一就是udev。
![[udev流程图.excalidraw]]
## udev 描述

udev守护进程([systemd-udevd.service(8)](http://www.jinbuguo.com/systemd/systemd-udevd.service.html#))从内核接收设备的插入，拔出，改变状态等事件，并根据这些事件的各种属性，到规则库中进行匹配，以确定触发事件的设备。被匹配成功的规则有可能提供额外的设备信息，这些信息可能会被记录到udev的数据库中，也可能会被用于创建符号链接。

udev处理的所有设备信息都存储在udev数据库中， 并且会发送给可能的设备事件的订阅者。 可以通过 libudev 库访问udev数据库以及设备事件源。



## **使用udev的好处**

**设定设备特定名称**：udev能够在/dev目录下创建额外的符号链接、重命名网络接口等。

**动态管理**：当设备添加 / 删除时，udev 的守护进程侦听来自内核的 uevent，以此添加或者删除 /dev下的设备文件，所以 udev 只为已经连接的设备产生设备文件，而不会在 /dev下产生大量虚无的设备文件。

**自定义命名规则**：通过 Linux 默认的规则文件，udev 在 /dev/ 里为所有的设备定义了内核设备名称，比如 /dev/sda、/dev/hda、/dev/fd等等。由于 udev 是在用户空间 (user space) 运行，Linux 用户可以通过自定义的规则文件，灵活地产生标识性强的设备文件名，比如 /dev/boot_disk、/dev/root_disk、/dev/color_printer等等。

**设定设备的权限和所有者 / 组**：udev 可以按一定的条件来设置设备文件的权限和设备文件所有者 / 组

## udev规则

## 配置文件

### `所有的规则文件统一按照文件名的字典顺序处理`

也就是说假设有如下规则文件

```bash
liuwh@liuwh-PC ~/Desktop> cd /usr/lib/udev/rules.d/
liuwh@liuwh-PC /u/l/u/rules.d> ls
01-md-raid-creating.rules             75-net-description.rules
39-usbmuxd.rules                      75-probe_mtd.rules
40-usb_modeswitch.rules               77-mm-cinterion-port-types.rules
50-firmware.rules                     77-mm-dell-port-types.rules
50-udev-default.rules                 77-mm-ericsson-mbm.rules
55-dm.rules                           77-mm-fibocom-port-types.rules
.........
```

按照字典序会优先读取01-md-raid-creating.rules，随后39-usbmuxd.rules，依次类推。

但是后面的规则文件如果重复的话，会覆盖之前的规则文件，也就是说假如01-md-raid-creating.rules文件中有针对sdb1设备的特殊操作，39-usbmuxd.rules也有针对sdb1设备的操作，那么39的规则会覆盖01的规则。

所以如果想要使得规则文件生效，我们一般是写99-name.rules  其中的name是标识用。

### `对于不同目录下的同名规则文件，以优先级最高的目录为准`

- 系统规则目录  /usr/lib/udev/rules.d 	（优先级低）
- 运行时规则目录 /run/udev/rules.d	 	  （优先级中）
- 本机规则目录 /etc/udev/rules.d			 （优先级高）

 如果系统管理员想要屏蔽 `/usr/lib/` 目录中的某个规则文件， 那么最佳做法是在 `/etc/` 目录中创建一个指向 `/dev/null` 的同名符号链接， 即可彻底屏蔽 `/usr/lib/` 目录中的同名文件。

### 编写配置文件

1. 规则文件必须以 `.rules` 作为后缀名，否则将被忽略。
2. 规则文件中以 "`#`" 开头的行以及空行将被忽略
3. 其他不以 "`#`" 开头的非空行，每行必须至少包含一个"键-值"对。
4. "键"有两种类型：匹配与赋值。
5. 每条规则都是由一系列逗号分隔的"键-值"对组成。
6. 如果某条规则的所有匹配键的值都匹配成功，那么就表示此条规则匹配成功， 也就是此条规则中的所有赋值键都会被赋予指定的值。
7. 根据操作符的不同，每个键都对应着一个唯一的操作。

#### 键

下面的"键"可用于匹配。注意，其中的某些键还可以针对父设备进行匹配，而不仅仅是生成设备事件的那个设备自身。 如果在同一条规则中有多个键可以针对父设备进行匹配，那么仅在所有这些键都同时成功匹配同一个父设备时，才算匹配成功。 Linux通过[[13 sys目录#sysfs|sysfs]]以树状结构展示设备，例如硬盘是SCSI设备的孩子、SCSI设备又是ATA控制器的孩子、 ATA控制器又是PCI总线的孩子。而你经常需要从父设备那里引用信息， 比如硬盘的序列号就是通过父设备(SCSI设备)展现的。

```bash
udevadm info --query=all --name=sda
```

| 匹配键              | 解释                                                                                                                                               |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| ACTION          | 事件 (uevent) 的行为，例如：add( 添加设备 )、remove( 删除设备 )。                                                                                  |
| KERNEL          | 匹配设备的内核名称。"内核名称"是指设备在sysfs里的名称，也就是默认的设备文件名称，例如"sda"。我理解就是内核分配的设备名称，不是udev规则分配的名字   |
| DEVPATH         | 匹配设备的路径(也就是该设备在sysfs文件系统下的相对路径)。举例：/dev/sda1 对应的 devpath 是 /block/sda/sda1 (一般对应着 /sys/block/sda/sda1 目录)。 |
| SUBSYSTEM       | 设备的子系统名称，例如：sda 的子系统为 block。                                                                                                     |
                                                                                                                                                  |
#### 操作符

| 操作符 | 匹配或赋值 | 解释                            |
| --- | ----- | ----------------------------- |
| ==  | 匹配    | 等于                            |
| !=  | 匹配    | 不等于                           |
| =   | 赋值    | 为键赋予指定的值。 此键之前的值(可能是个列表)将被丢弃。 |
| +=  | 赋值    | 在键的现有值列表中增加此处指定的值。            |
| -=  | 赋值    | 在键的现有值列表中删除此处指定的值             |
| :=  | 赋值    | 为键赋予指定的值，并视为最终值，也就是禁止被继续修改。   |

**配置文件示例**

```bash
liuwh@liuwh-PC /u/l/u/rules.d> cat 99-laptop-mode.rules 
ACTION=="change", SUBSYSTEM=="power_supply", ENV{POWER_SUPPLY_NAME}=="|AC|ACAD", RUN+="lmt-udev auto"
ACTION=="add|remove", SUBSYSTEM=="machinecheck", RUN+="lmt-udev auto"
ACTION=="add", SUBSYSTEM=="usb", RUN+="lmt-udev force"

# Run a particular module only
#ACTION=="add", SUBSYSTEM=="usb", RUN+="lmt-udev force modules=runtime-pm devices=%k"

```

## 相关命令

### dmesg

### [[l#lsblk|lsblk]]

### [[u#udevadm|udevadm]]

## 参考链接：

1. udev 中文手册 <http://www.jinbuguo.com/systemd/udev.html>译者：**[金步国](http://www.jinbuguo.com/)**
2. udev 入门：管理设备事件的 Linux 子系统<https://zhuanlan.zhihu.com/p/51984452>
3. udev规则以及编写<https://www.cnblogs.com/fah936861121/p/6496608.html>
4.udev udevadm介绍及linux设备重命名和自动挂载应用实例分析 <https://blog.csdn.net/li_wen01/article/details/89435306>
