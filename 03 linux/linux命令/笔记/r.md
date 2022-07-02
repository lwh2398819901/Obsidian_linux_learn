## route
***route***
```
uos@uos-PC ~/Desktop> route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.10.77.1      0.0.0.0         UG    100    0        0 eno1
10.10.77.0      0.0.0.0         255.255.255.0   U     100    0        0 eno1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```
***route -n***  
-n ： 将主机名以 IP 的方式显示
```
uos@uos-PC ~/Desktop> route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.10.77.1      0.0.0.0         UG    100    0        0 eno1
10.10.77.0      0.0.0.0         255.255.255.0   U     100    0        0 eno1
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
uos@uos-PC ~/Desktop> 
```

 
   - Destination ：其实就是 Network 的意思； 
   - Gateway ：就是该接口的 Gateway 那个 IP 啦！若为 0.0.0.0 表示需要额外的 IP； 
   -  Genmask ：就是 Netmask 啦！与 Destination 组合成为一部主机或网域； 
   -  Flags ：共有多个旗标可以来表示该网域或主机代表的意义： 
   			-   U：代表该路由可用； 
   			-   G：代表该网域需要经由 Gateway 来帮忙转递； 
   			-   H：代表该行路由为一部主机，而非一整个网域； 
   - Iface ：就是 Interface (接口) 的意思。