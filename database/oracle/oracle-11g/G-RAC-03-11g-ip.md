---
title: RAC
---

```bash
查看public ip、private ip:
[root@rac1 bin]# ./oifcfg getif
eth0  136.0.10.0  global  public
eth1  135.0.10.0  global  cluster_interconnect

查看VIP:
[root@rac1 bin]# ./srvctl config vip -node rac1
VIP 存在: 网络编号 1, 托管节点 rac1
VIP 名称: rac1-vip.oracle.com
VIP IPv4 地址: 136.0.10.23
VIP IPv6 地址:
[root@rac1 bin]# ./srvctl config vip -node rac2
VIP 存在: 网络编号 1, 托管节点 rac2
VIP 名称: rac2-vip.oracle.com
VIP IPv4 地址: 136.0.10.24
VIP IPv6 地址:

查看SCAN_IP:
[root@rac1 bin]# ./srvctl config scan -all
SCAN 名称: scan.oracle.com, 网络: 1
子网 IPv4: 136.0.10.0/255.255.255.0/eth0
子网 IPv6:
SCAN 0 IPv4 VIP: 136.0.10.196
SCAN 名称: scan.oracle.com, 网络: 1
子网 IPv4: 136.0.10.0/255.255.255.0/eth0
子网 IPv6:
SCAN 1 IPv4 VIP: 136.0.10.197
SCAN 名称: scan.oracle.com, 网络: 1
子网 IPv4: 136.0.10.0/255.255.255.0/eth0
子网 IPv6:
SCAN 2 IPv4 VIP: 136.0.10.195

停止资源：
./crsctl stop has

修改public ip、private ip:
./oifcfg getif
./oifcfg delif -global eth0
./oifcfg delif -global eth1
./oifcfg setif -global eth0/192.168.xxx.xxx:public
./oifcfg setif -global eth1/172.168.xxx.xxx:cluster_interconnect
./oifcfg getif

修改VIP
./crsctl stop resource ora.rac1.LISTENER_RAC1.lsnr
./crsctl stop resource ora.rac2.LISTENER_RAC2.lsnr
./crsctl stop resource ora.rac1.vip
./crsctl stop resource ora.rac2.vip
./srvctl modify nodeapps -n rac1 -A 192.168.xxx.xxx/255.255.255.0/eth0
./srvctl modify nodeapps -n rac2 -A 192.168.xxx.xxx/255.255.255.0/eth0
./crsctl start resource ora.rac1.vip
./crsctl start resource ora.rac2.vip
./srvctl start listener
./srvctl status listener

修改SCAN IP
./srvctl stop scan
./srvctl stop scan_listener
./srvctl remove scan
./srvctl remove scan_listener
./srvctl add scan -n scan.oracle.cn -k 1 -S 192.168.xxx.xxx/255.255.255.0/eth0
./srvctl add scan_listener
./srvctl start scan
./srvctl start scan_listener

修改参数
alter system set local_listener
alter system set remote_listener
```
