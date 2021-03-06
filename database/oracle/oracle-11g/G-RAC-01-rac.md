---
title: RAC
---

```bash
################################################################################
配置重点：
1.所有节点都要关闭防火墙和SeLinux
2.每个节点都要有至少两个物理网卡
3.所有节点的public IP指定网关
4.安装集群之前VIP不能ping通
5.配置所有节点的grid用户的信任关系、oracle用户的信任关系
6.配置NTP
7.配置DNS
8.配置共享存储
################################################################################
规划两个节点的IP地址：
vi /etc/hosts
--------------------------------------------------------------------------------
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# Public Network - (eth0)
172.25.0.12 rac1.example.com rac1
172.25.0.13 rac2.example.com rac2

# Private Interconnect - (eth1)
10.25.0.12 rac1-priv.exampl.com rac1-priv
10.25.0.13 rac2-priv.exampl.com rac2-priv

# Public Virtual IP (eth0:xx)
172.25.0.15 rac1-vip.example.com rac1-vip
172.25.0.16 rac2-vip.example.com rac2-vip
--------------------------------------------------------------------------------

按照hosts文件配置两个节点的网卡，public ip一定要指定网关！
rac1-->eth0
--------------------------
# Virtio Network Device
DEVICE=eth0
BOOTPROTO=static
ONBOOT=yes
TYPE=Ethernet
NETMASK=255.255.255.0
IPADDR=172.25.0.12
HWADDR=52:54:00:00:00:0c
GATEWAY=172.25.0.254
--------------------------

rac1-->eth1
--------------------------
# Virtio Network Device
DEVICE=eth1
BOOTPROTO=static
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=10.25.0.12
HWADDR=52:54:00:01:00:0c
TYPE=Ethernet
--------------------------

[root@rac1 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 52:54:00:00:00:0C  
          inet addr:172.25.0.12  Bcast:172.25.0.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe00:c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1075 errors:0 dropped:0 overruns:0 frame:0
          TX packets:161 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:131689 (128.6 KiB)  TX bytes:27050 (26.4 KiB)

eth1      Link encap:Ethernet  HWaddr 52:54:00:01:00:0C  
          inet addr:10.25.0.12  Bcast:10.25.0.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe01:c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:264 errors:0 dropped:0 overruns:0 frame:0
          TX packets:33 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:22346 (21.8 KiB)  TX bytes:6092 (5.9 KiB)

[root@rac1 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.25.0.0       0.0.0.0         255.255.255.0   U     0      0        0 eth1
172.25.0.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     0      0        0 eth1
0.0.0.0         172.25.0.254    0.0.0.0         UG    0      0        0 eth0

###############################################################################
rac2-->eth0
--------------------------
# Virtio Network Device
DEVICE=eth0
BOOTPROTO=static
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=172.25.0.13
HWADDR=52:54:00:00:00:0d
GATEWAY=172.25.0.254
TYPE=Ethernet
--------------------------

rac2-->eth1
--------------------------
# Virtio Network Device
DEVICE=eth1
BOOTPROTO=static
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=10.25.0.13
HWADDR=52:54:00:01:00:0d
TYPE=Ethernet
--------------------------

[root@rac2 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 52:54:00:00:00:0D  
          inet addr:172.25.0.13  Bcast:172.25.0.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe00:d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:41 errors:0 dropped:0 overruns:0 frame:0
          TX packets:59 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3066 (2.9 KiB)  TX bytes:10323 (10.0 KiB)

eth1      Link encap:Ethernet  HWaddr 52:54:00:01:00:0D  
          inet addr:10.25.0.13  Bcast:10.25.0.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe01:d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:5 errors:0 dropped:0 overruns:0 frame:0
          TX packets:30 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:260 (260.0 b)  TX bytes:5333 (5.2 KiB)

[root@rac2 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.25.0.0       0.0.0.0         255.255.255.0   U     0      0        0 eth1
172.25.0.0      0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     0      0        0 eth1
0.0.0.0         172.25.0.254    0.0.0.0         UG    0      0        0 eth0
###############################################################################
创建组:
groupadd -g 501 oinstall
groupadd -g 502 dba
groupadd -g 503 oper
groupadd -g 504 asmadmin
groupadd -g 505 asmdba
groupadd -g 506 asmoper
创建用户
useradd -u 502 -g oinstall -G dba,asmadmin,asmdba,asmoper grid
useradd -u 501 -g oinstall -G dba,oper,asmdba,asmadmin oracle
修改用户口令
passwd grid
passwd oracle

修改grid用户配置文件
vi .bashrc
----------------------------------------------------------
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/grid
export ORACLE_OWNER=oracle
export ORACLE_SID=+ASM1 #rac2节点为ORACLE_SID=+ASM2
export ORACLE_TERM=vt100
export THREADS_FLAG=native
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
export LANG=en_US
alias sqlplus='rlwrap sqlplus'
alias lsnrctl='rlwrap lsnrctl'
alias asmcmd='rlwrap asmcmd'
----------------------------------------------------------

修改oracle用户配置文件
vi .bashrc
----------------------------------------------------------
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export ORACLE_OWNER=oracle
export ORACLE_SID=orcl1
export ORACLE_TERM=vt100
export THREADS_FLAG=native
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
export EDITOR=vi
export SQLPATH=/home/oracle
export LANG=en_US
alias sqlplus='rlwrap sqlplus'
alias lsnrctl='rlwrap lsnrctl'
alias rman='rlwrap rman'
alias dgmgrl='rlwrap dgmgrl'
----------------------------------------------------------
###############################################################################
修改主机shell限制
vi /etc/security/limits.conf
------------------------------------------
#grid & oracle configure shell parameters
grid soft nofile 65536
grid hard nofile 65536
grid soft nproc 16384
grid hard nproc 16384

oracle soft nofile 65536
oracle hard nofile 65536
oracle soft nproc 16384
oracle hard nproc 16384
------------------------------------------
修改主机内核参数
vi /etc/sysctl.conf
------------------------------------------
kernel.shmmax = 4294967296
kernel.shmmni = 4096
kernel.shmall = 2097152
kernel.sem = 250 32000 100 128
fs.file-max = 6815744
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
------------------------------------------
使内核参数生效
/sbin/sysctl -p
###############################################################################
yum -y install scsi-target-utils

vi /etc/tgt/targets.conf
----------------------------------------
<target iqn.2016-10-25.com.oracle.blues:luns1>
backing-store /dev/sda5
initiator-address 172.25.x.12
initiator-address 172.25.x.13
</target>
----------------------------------------

安装、配置 openfiler:
Services标签页 --> iSCSI target server --> Enabled
System标签页 --> 开放ip --> Network Access Configuration --> 172.25.0.0/255.255.255.0
Volumes标签页 --> 创建物理卷
Volumes标签页 --> 创建卷组 --> Volume Groups
Volumes标签页 --> 创建逻辑卷 --> Add Volume
Volumes标签页 --> 添加IQN --> iSCSI Targets --> Target Configuration
Volumes标签页 --> 映射LUN --> iSCSI Targets --> LUN Mapping

在rac1和rac2发现openfiler服务器:
rac1:
iscsiadm -m discovery -t sendtargets -p 172.25.xxx.14 -l
rac2:
iscsiadm -m discovery -t sendtargets -p 172.25.xxx.14 -l

在rac1对发现的共享磁盘进行分区:
分成7个区，3个1G的，4个4G的分区！
fdisk -l 
cat /proc/partitions

在rac1,rac2将磁盘分区初始化为rawdevice:
vi /etc/udev/rules.d/60-raw.rules
---------------------------------------------------------------
ACTION=="add", KERNEL=="sda1", RUN+="/bin/raw /dev/raw/raw1 %N"
ACTION=="add", KERNEL=="sda2", RUN+="/bin/raw /dev/raw/raw2 %N"
ACTION=="add", KERNEL=="sda3", RUN+="/bin/raw /dev/raw/raw3 %N"
ACTION=="add", KERNEL=="sda5", RUN+="/bin/raw /dev/raw/raw4 %N"
ACTION=="add", KERNEL=="sda6", RUN+="/bin/raw /dev/raw/raw5 %N"
ACTION=="add", KERNEL=="sda7", RUN+="/bin/raw /dev/raw/raw6 %N"
ACTION=="add", KERNEL=="sda8", RUN+="/bin/raw /dev/raw/raw7 %N"
KERNEL=="raw[1-7]", MODE="0660", GROUP="asmadmin", OWNER="grid"
---------------------------------------------------------------
重新启动udev
start_udev
###############################################################################
配置NTP：server --> rac1
vi /etc/sysconfig/ntpd
--------------------------------------------
OPTIONS="-x -u ntp:ntp -p /var/run/ntpd.pid"

vi /etc/ntp.conf
------------------------------------------------------
restrict 127.0.0.1 
restrict -6 ::1
restrict 172.25.0.0 mask 255.255.255.0 nomodify notrap 
server 127.127.1.0
fudge   127.127.1.0 stratum 10
keys /etc/ntp/keys

重新启动ntp服务
service ntpd restart
chkconfig ntpd on

INFO: PRVF-5408 : NTP Time Server ".LOCL." is common only to the following nodes "rac1"
INFO: PRVF-5416 : Query of NTP daemon failed on all nodes
INFO: Clock synchronization check using Network Time Protocol(NTP) failed
INFO: PRVF-9652 : Cluster Time Synchronization Services check failed


client --> rac2
vi /etc/sysconfig/ntpd
--------------------------------------------
OPTIONS="-x -u ntp:ntp -p /var/run/ntpd.pid"

vi /etc/ntp.conf
--------------------------------------------
server 172.25.0.12
restrict 172.25.0.12 mask 255.255.255.255 nomodify notrap noquery

客户端与服务端时间同步命令	（大概要5分钟后，客户端才能与之同步）
重新启动ntp服务
service ntpd restart
chkconfig ntpd on
ntpdate 172.25.0.12 (ntp server ip)
###############################################################################
配置grid用户的信任关系
su - grid
ssh-keygen -t rsa
ssh-keygen -t dsa
cd .ssh
cat *.pub > authorized_keys

scp authorized_keys grid@172.25.x.13:/home/grid/.ssh/keys_dbs
cat keys_dbs >> authorized_keys
scp authorized_keys grid@172.25.x.12:/home/grid/.ssh/

ssh rac1 date
ssh rac2 date
ssh rac1-priv date
ssh rac2-priv date
--------------------------------------------------------------
配置oracle用户的信任关系
su - oracle
ssh-keygen -t rsa
ssh-keygen -t dsa
cd .ssh
cat *.pub > authorized_keys

scp authorized_keys oracle@172.25.x.13:/home/oracle/.ssh/keys_dbs
cat keys_dbs >> authorized_keys
scp authorized_keys oracle@172.25.x.12:/home/oracle/.ssh/

ssh rac1 date
ssh rac2 date
ssh rac1-priv date
ssh rac2-priv date
###############################################################################
配置DNS：server --> rac1
yum -y install bind bind-chroot caching-nameserver

配置DNS
# cd /var/named/chroot/etc
# cp -p named.caching-nameserver.conf named.conf
# vi /var/named/chroot/etc/named.conf 

-----------------------------------------------------------------------------
 options {
         listen-on port 53 { any; };
         listen-on-v6 port 53 { ::1; };
         directory       "/var/named";
         dump-file       "/var/named/data/cache_dump.db";
         statistics-file "/var/named/data/named_stats.txt";
         memstatistics-file "/var/named/data/named_mem_stats.txt"; 

         allow-query     { any; };
         allow-query-cache { any; };
 };
 logging {
         channel default_debug {
                 file "data/named.run";
                 severity dynamic;
         };
 };
 view localhost_resolver {
         match-clients      { any; };
         match-destinations { any; };
          recursion yes;
         include "/etc/named.zones";
 };

-----------------------------------------------------------------------------
# cp -p named.rfc1912.zones named.zones
vi /var/named/chroot/etc/named.zones
-----------------------------------------------------------------------------
zone "." IN {
        type hint;
        file "/dev/null";
};

zone "oracle.com" IN {
        type master;
        file "oracle.com.zone";
        allow-update { none; };
}; 

zone "x.25.172.in-addr.arpa" IN {
        type master;
        file "x.25.172.local";
        allow-update { none; };
};
-----------------------------------------------------------------------------

# cd /var/named/chroot/var/named
# cp -p named.zero oracle.com.zone
# cp -p named.local x.25.172.local 

vi /var/named/chroot/var/named/oracle.com.zone
-----------------------------------------------------------------------------
$TTL    86400
@               IN SOA  dns.oracle.com.      root.oracle.com. (
                                        42              ; serial (d. adams)
                                        3H              ; refresh
                                        15M             ; retry
                                        1W              ; expiry
                                        1D )            ; minimum
@       IN      NS      dns.oracle.com.
rac1    IN      A       172.25.0.12
rac2    IN      A       172.25.0.13
scan    IN      A       172.25.0.141
scan    IN      A       172.25.0.142
scan    IN      A       172.25.0.143 
-----------------------------------------------------------------------------

vi /var/named/chroot/var/named/x.25.172.local
-----------------------------------------------------------------------------
$TTL    86400
@       IN      SOA     dns.oracle.com. root.oracle.com.  (
                                      1997022700 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
@       IN      NS      dns.oracle.com.
12       IN      PTR     rac1.oracle.com.
13       IN      PTR     rac2.oracle.com.
141     IN      PTR     scan.oracle.com.
142     IN      PTR     scan.oracle.com.
143     IN      PTR     scan.oracle.com. 
-----------------------------------------------------------------------------

重新启动服务
service named restart
chkconfig named on

配置客户端：rac1 & rac2
vi /etc/resolv.conf
------------------------
search oracle.com
nameserver 172.25.0.12‪
------------------------

域名解析测试：
nslookup scan.oracle.com
nslookup 172.25.x.141
nslookup 172.25.x.142
nslookup 172.25.x.143
###############################################################################
创建相关目录：
mkdir -p /u01/grid
chown -R grid:oinstall /u01/grid
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01/app/
chmod -R 775 /u01/
###############################################################################
将grid安装包上传到rac1:
scp p13390677_112040_Linux-x86-64_3of7.zip grid@172.25.0.12:/home/grid
解压缩：
su - grid
unzip p13390677_112040_Linux-x86-64_3of7.zip

所有节点都要安装校验磁盘组需要的rpm包：
rpm -ivh /home/grid/grid/rpm/cvuqdisk-1.0.9-1.rpm

校验集群安装的可行性：
cd grid/
./runcluvfy.sh stage -pre crsinst -n rac1,rac2 -fixup -verbose

/u01/grid/bin/cluvfy stage -post crsinst -n rac1,rac2 -verbose
###############################################################################
Checking the file "/etc/resolv.conf" to make sure only one of domain and search entries is defined
File "/etc/resolv.conf" does not have both domain and search entries defined
Checking if domain entry in file "/etc/resolv.conf" is consistent across the nodes...
domain entry in file "/etc/resolv.conf" is consistent across nodes
Checking if search entry in file "/etc/resolv.conf" is consistent across the nodes...
search entry in file "/etc/resolv.conf" is consistent across nodes
Checking file "/etc/resolv.conf" to make sure that only one search entry is defined
All nodes have one search entry defined in file "/etc/resolv.conf"
Checking all nodes to make sure that search entry is "oracle.com" as found on node "rac2"
All nodes of the cluster have same value for 'search'
Checking DNS response time for an unreachable node
  Node Name                             Status                  
  ------------------------------------  ------------------------
  rac2                                  passed                  
  rac1                                  passed                  
The DNS response time for an unreachable node is within acceptable limit on all nodes

File "/etc/resolv.conf" is consistent across nodes

Check: Time zone consistency 
Result: Time zone consistency check passed

Pre-check for cluster services setup was unsuccessful. 
Checks did not pass for the following node(s):
	rac2
###############################################################################
Network Time Protocol (NTP) - This task verifies cluster time synchronization on clusters that use Network Time Protocol (NTP).  Error: 
 - 
PRVF-5408 : NTP Time Server ".LOCL." is common only to the following nodes "rac1"  - Cause:  One or more nodes in the cluster do not synchronize with the NTP Time Server indicated.  - Action:  At least one common NTP Time Server is required for a successful Clock Synchronization check. If there are none, reconfigure all of the nodes in the cluster to synchronize with at least one common NTP Time Server. 
 - 
PRVF-5416 : Query of NTP daemon failed on all nodes  - Cause:  An attempt to query the NTP daemon using the 'ntpq' command failed on all nodes.  - Action:  Make sure that the NTP query command 'ntpq' is available on all nodes and make sure that user running the CVU check has permissions to execute it. 

  Check Failed on Nodes: [rac2] Check Succeeded On Nodes: [rac1]  
Verification result of failed node: rac2 

Back to Top

[root@rac1 bin]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*LOCAL(0)        .LOCL.           5 l   55   64  377    0.000    0.000   0.001

[root@rac2 bin]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*rac1.example.co LOCAL(0)         6 u   58  256  377    0.169   -0.006   0.026

###############################################################################
[ohasd(6187)]CRS-1301:Oracle High Availability Service started on node rac2.
2016-01-26 10:34:03.441:
[ohasd(6187)]CRS-8017:location: /etc/oracle/lastgasp has 2 reboot advisory log files, 0 were announced and 0 errors occurred
2016-01-26 10:34:06.939:
[/u01/grid/bin/orarootagent.bin(6227)]CRS-2302:Cannot get GPnP profile. Error CLSGPNP_NO_DAEMON (GPNPD daemon is not running).
2016-01-26 10:34:13.363:
[ohasd(6187)]CRS-2302:Cannot get GPnP profile. Error CLSGPNP_NO_DAEMON (GPNPD daemon is not running).
2016-01-26 10:34:13.444:
[gpnpd(6328)]CRS-2328:GPNPD started on node rac2.
2016-01-26 10:34:15.818:
[cssd(6393)]CRS-1713:CSSD daemon is started in clustered mode
2016-01-26 10:34:17.661:
[ohasd(6187)]CRS-2767:Resource state recovery not attempted for 'ora.diskmon' as its target state is OFFLINE
2016-01-26 10:34:17.661:
[ohasd(6187)]CRS-2769:Unable to failover resource 'ora.diskmon'.
2016-01-26 10:34:36.814:
[gpnpd(6328)]CRS-2332:Error pushing GPnP profile to "mdns:service:gpnp._tcp.local.://rac1:48895/agent=gpnpd,cname=rac-cluster,host=rac1,pid=3204/gpnpd h:rac1 c:rac-cluster".
###############################################################################
如果root脚本失败，删除节点资源
# /u01/app/11.2.0/grid/crs/install/roothas.pl -deconfig -force -verbose
# /u01/app/11.2.0/grid/crs/install/rootcrs.pl -deconfig -force -verbose

校验grid安装是否成功：
cd /u01/grid/bin
./crs_stat -t

[grid@rac1 bin]$ ./crs_stat -t
Name                           Target     State      Host      
------------------------------ ---------- ---------  -------   
ora.CRS.dg                     ONLINE     ONLINE     rac1      
ora.LISTENER.lsnr              ONLINE     ONLINE     rac1      
ora.LISTENER_SCAN1.lsnr        ONLINE     ONLINE     rac1      
ora.LISTENER_SCAN2.lsnr        ONLINE     ONLINE     rac2      
ora.LISTENER_SCAN3.lsnr        ONLINE     ONLINE     rac1      
ora.asm                        ONLINE     ONLINE     rac1      
ora.cvu                        ONLINE     ONLINE     rac2      
ora.gsd                        OFFLINE    OFFLINE              
ora.net1.network               ONLINE     ONLINE     rac1      
ora.oc4j                       ONLINE     ONLINE     rac2      
ora.ons                        ONLINE     ONLINE     rac1      
ora.rac1.ASM1.asm              ONLINE     ONLINE     rac1      
ora.rac1.LISTENER_RAC1.lsnr    ONLINE     ONLINE     rac1      
ora.rac1.gsd                   OFFLINE    OFFLINE              
ora.rac1.ons                   ONLINE     ONLINE     rac1      
ora.rac1.vip                   ONLINE     ONLINE     rac1      
ora.rac2.ASM2.asm              ONLINE     ONLINE     rac2      
ora.rac2.LISTENER_RAC2.lsnr    ONLINE     ONLINE     rac2      
ora.rac2.gsd                   OFFLINE    OFFLINE              
ora.rac2.ons                   ONLINE     ONLINE     rac2      
ora.rac2.vip                   ONLINE     ONLINE     rac2      
ora.registry.acfs              ONLINE     ONLINE     rac1      
ora.scan1.vip                  ONLINE     ONLINE     rac1      
ora.scan2.vip                  ONLINE     ONLINE     rac2      
ora.scan3.vip                  ONLINE     ONLINE     rac1  
###############################################################################

备份ocr
/u01/grid/bin/ocrconfig -export /home/grid/ocr19.bak

安装数据软件：
./runInstaller

安装数据软件时如果不能发现集群节点：CRS="true"
vi /u01/app/oraInventory/ContentsXML/inventory.xml
-------------------------------------------------------------------------------
<HOME_LIST>
<HOME NAME="Ora11g_gridinfrahome1" LOC="/u01/grid" TYPE="O" IDX="1" CRS="true">
   <NODE_LIST>
      <NODE NAME="rac1"/>
      <NODE NAME="rac2"/>
   </NODE_LIST>
</HOME>
</HOME_LIST>
-------------------------------------------------------------------------------
###############################################################################
在grid用户下，创建新的磁盘组：
su - grid
asmca 

###############################################################################
在grid用户下，配置监听
su - grid
netca

###############################################################################
在oracle用户下创建数据库：
su - oracle
dbca

###############################################################################
如果oracle用户不能发现磁盘组，请检查文件$ORACLE_HOME/bin/oracle 的权限 6751
[root@rac2 bin]# ll oracle
-rwxrwxr-x 1 oracle oinstall 239627031 Jan 26 12:56 oracle

[root@rac2 bin]# chmod 6751 oracle
[root@rac2 bin]# ll oracle
-rwsr-s--x 1 oracle oinstall 239627031 Jan 26 12:56 oracle
###############################################################################
SQL> select inst_id,status from gv$instance;

   INST_ID STATUS
---------- ------------
	 1 OPEN
	 2 OPEN

SQL> show parameter cluster

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
cluster_database		     boolean	 TRUE
cluster_database_instances	     integer	 2
cluster_interconnects		     string
###############################################################################
使crs_stat显示资源完整名称
vi rac_stat.sh
-----------------------------------------------------------------------------------------
awk \
  'BEGIN {printf "%-30s %-10s %-10s %-10s\n","Name                          ","Target    ","State     ","Host   ";
          printf "%-30s %-10s %-10s %-10s\n","------------------------------","----------", "---------","-------";}'

/u01/grid/bin/crs_stat | awk \
'BEGIN { FS="=| ";state = 0;}
  $1~/NAME/ {appname = $2; state=1};
  state == 0 {next;}
  $1~/TARGET/ && state == 1 {apptarget = $2; state=2;}
  $1~/STATE/ && state == 2 {appstate = $2; apphost = $4; state=3;}
  state == 3 {printf "%-30s %-10s %-10s %-10s\n", appname,apptarget,appstate,apphost; state=0;}'
-----------------------------------------------------------------------------------------
chmod +x rac_stat.sh
###############################################################################
启动和停止集群资源：
/u01/grid/bin/crs_stop -all
/u01/grid/bin/crs_start -all

启动和停止集群后台核心进程：
/u01/grid/bin/crsctl stop cluster -f
/u01/grid/bin/crsctl stop has -f

/u01/grid/bin/crsctl start has
启动has的另一种方法：
/etc/init.d/init.ohasd run

/u01/grid/bin/crsctl start cluster

查看集群后台核心进程状态：
/u01/grid/bin/crsctl check crs

CRS-4638: Oracle High Availability Services is online
CRS-4537: Cluster Ready Services is online
CRS-4529: Cluster Synchronization Services is online
CRS-4533: Event Manager is online

/u01/grid/bin/crsctl check css
/u01/grid/bin/crsctl check cluster

查看时间同步服务状态：
/u01/grid/bin/crsctl check ctss

查看事件管理（Event Manager）服务状态：
/u01/grid/bin/crsctl check evm
###############################################################################
管理rac的集群注册盘(ocr:Oracle Cluster Registry)和表决盘(voting disk)
查看ocr的位置：
/u01/grid/bin/ocrcheck

Status of Oracle Cluster Registry is as follows :
	 Version                  :          3
	 Total space (kbytes)     :     262120
	 Used space (kbytes)      :       3156
	 Available space (kbytes) :     258964
	 ID                       : 1545614171
	 Device/File Name         :       +crs
                                    Device/File integrity check succeeded

                                    Device/File not configured

                                    Device/File not configured

                                    Device/File not configured

                                    Device/File not configured

	 Cluster registry integrity check succeeded

	 Logical corruption check succeeded

查看voting disk的位置：
/u01/grid/bin/crsctl query css votedisk
##  STATE    File Universal Id                File Name Disk group
--  -----    -----------------                --------- ---------
 1. ONLINE   24f90820042f4fc8bf19f63921a19837 (/dev/raw/raw1) [CRS]
Located 1 voting disk(s).

手工备份ocr:
ocrconfig -local -manualbackup
 
ocrconfig -export /home/oracle/ocr.bak

备份ASM实例的参数文件：
su - grid
sqlplus / as sysasm
create pfile='/home/grid/spfilebak.ora' from spfile;

模拟raw1损坏：dd写10M 0
重新启动集群！

停止crs
crsctl stop crs -f
以独占模式启动crs
./crsctl start crs -excl -nocrs
通过grid用户登录创建ASM磁盘组:asmca不可用就使用手工创建！
su - grid
sqlplus / as sysasm
create diskgroup data1 external redundancy disk '/dev/raw/raw1','/dev/raw/raw2','/dev/raw/raw3' attribute 'compatible.asm'='11.2.0.4.0';

alter diskgroup data1 set attribute 'compatible.asm'='11.2.0.4.0';

修改所有节点的ocr.loc文件：
vi /etc/oracle/ocr.loc
------------------------------
ocrconfig_loc=+AAA

恢复ocr:
ocrconfig -import /home/oracle/ocr.bak
create spfile='+TEST' from pfile='/home/grid/spfilebak.ora'

恢复VOTE磁盘：
/u01/grid/bin/crsctl replace votedisk +data1

重新启动集群！
###############################################################################
asm磁盘组中各种文件的管理：
su - grid
asmcmd -p 是一个操作asm磁盘组的命令行工具，可以查看、拷贝、删除磁盘组中的数据

增加控制文件：
asmcmd -p
ASMCMD [+data/orcl/controlfile] > cp Current.260.902153423 /home/grid/control.ctl
copying +data/orcl/controlfile/Current.260.902153423 -> /home/grid/control.ctl
ASMCMD [+data/orcl/controlfile] > 
ASMCMD [+data/orcl/controlfile] > cp /home/grid/control.ctl +data/orcl/controlfile/
copying /home/grid/control.ctl -> +data/orcl/controlfile/control.ctl

alter system set control_files='','','' scope=spfile;

增加日志组
alter database add logfile '+DATA/' size 100m;

创建表空间：
create tablespace mytbs datafile '+DATA/' size 50m;
###############################################################################
查看asm空间分配：
[grid@rac2 bin]$ kfed read /dev/raw/raw1 | grep size
kfdhdb.secsize:                     512 ; 0x0b8: 0x0200
kfdhdb.blksize:                    4096 ; 0x0ba: 0x1000
kfdhdb.ausize:                  1048576 ; 0x0bc: 0x00100000
kfdhdb.dsksize:                     954 ; 0x0c4: 0x000003ba

[grid@rac2 bin]$ ./srvctl config database -d orcl
Database unique name: orcl
Database name: orcl
Oracle home: /u01/app/oracle/product/11.2.0/db_1
Oracle user: oracle
Spfile: +DATA/orcl/spfileorcl.ora
Domain: 
Start options: open
Stop options: immediate
Database role: PRIMARY
Management policy: AUTOMATIC
Server pools: orcl
Database instances: orcl1,orcl2
Disk Groups: DATA
Mount point paths: 
Services: 
Type: RAC
Database is administrator managed

cat init+ASM1.ora
-------------------------------------------
+ASM1.asm_diskgroups='DATA'#Manual Mount
+ASM2.asm_diskgroups='DATA'#Manual Mount
*.asm_diskstring='/dev/raw/*'
*.asm_power_limit=1
*.diagnostic_dest='/u01/app/oracle'
*.instance_type='asm'
*.large_pool_size=12M
*.remote_login_passwordfile='EXCLUSIVE'
-------------------------------------------
###############################################################################
在老节点为新的实例准备undo和redo：
create undo tablespace undotbs4 datafile '+DATA/' size 50m;

alter database add logfile thread 4 '+DATA/' size 50m;
alter database add logfile thread 4 '+DATA/' size 50m;
alter database add logfile thread 4 '+DATA/' size 50m;

在老节点为新的实例准备参数：
alter system set instance_number=4 scope=spfile sid='orcl4';
alter system set thread=4 scope=spfile sid='orcl4';
alter system set undo_tablespace=undotbs4 scope=spfile sid='orcl4';

在老节点为新的实例启用日志进程：
alter database enable thread 4；

为新实例准备pfile：
vi $ORACLE_HOME/dbs/initorcl4.ora
----------------------------------
spfile='+DATA/orcl/spfileorcl.ora'
#########################################################################################
create diskgroup data2 external redundancy disk '/dev/raw/raw4','/dev/raw/raw5' attribute 'compatible.asm'='11.2.0.4.0';
create diskgroup data3 external redundancy disk '/dev/raw/raw6','/dev/raw/raw7' attribute 'compatible.asm'='11.2.0.4.0';

SQL> alter diskgroup DATA2 mount;
SQL> alter diskgroup DATA3 mount;

SQL> startup nomount
SQL> create spfile='+DATA2/orcl/spfileorcl.ora' from pfile;
SQL> shutdown abort

rac1:
vi $ORACLE_HOME/dbs/initorcl1.ora
----------------------------------------
spfile='+DATA2/orcl/spfileorcl.ora'
rac2:
vi $ORACLE_HOME/dbs/initorcl2.ora
----------------------------------------
spfile='+DATA2/orcl/spfileorcl.ora'

SQL> startup nomount

RMAN> restore controlfile from '/home/oracle/rmanbk/ORCL_1416338340_2_1_20160602.bkp';
RMAN> alter database mount;

run{
set newname for datafile 1 to '+data2/';
set newname for datafile 2 to '+data2/';
set newname for datafile 3 to '+data2/';
set newname for datafile 4 to '+data2/';
set newname for datafile 5 to '+data2/';
set newname for tempfile 1 to '+data2/';
restore database;
switch datafile all;
switch tempfile all;
}

alter database rename file '/u01/app/oracle/oradata/orcl/redo01.log' to '+DATA2/orcl/onlinelog/redo01.log';
alter database rename file '/u01/app/oracle/oradata/orcl/redo02.log' to '+DATA2/orcl/onlinelog/redo02.log';
alter database rename file '/u01/app/oracle/oradata/orcl/redo03.log' to '+DATA2/orcl/onlinelog/redo03.log';

alter database open resetlogs;

将数据库转换成rac模式，增加rac所需要的表空间、日志文件、初始化参数
alter system set cluster_database=true scope=spfile;
alter system set cluster_database_instances=2 scope=spfile;

alter system set instance_number=1 scope=spfile sid='orcl1';
alter system set thread=1 scope=spfile sid='orcl1';
alter system set undo_tablespace=undotbs1 scope=spfile sid='orcl1';

alter system set db_create_file_dest='+DATA2';

create undo tablespace undotbs2 datafile size 50M;

alter database add logfile thread 2 size 50m;
alter database add logfile thread 2 size 50m;
alter database add logfile thread 2 size 50m;

alter system set instance_number=2 scope=spfile sid='orcl2';
alter system set thread=2 scope=spfile sid='orcl2';
alter system set undo_tablespace=undotbs2 scope=spfile sid='orcl2';

alter database enable thread 2;

rac1:
export ORACLE_SID=orcl1
SQL> shutdown immediate
SQL> startup
rac2:
export ORACLE_SID=orcl2
SQL> startup

SQL> select inst_id,status from gv$instance;

   INST_ID STATUS
---------- ------------
	 2 OPEN
	 1 OPEN

配置OEM：
emca -config dbcontrol db -repos recreate

vi /u01/app/oracle/product/11.2.0/db_1/install/portlist.ini

emctl status dbconsole
emctl start dbconsole
emctl stop dbconsole
################################################################################
su - grid
lsnrctl status LISTENER
lsnrctl status LISTENER_SCAN1
lsnrctl status LISTENER_SCAN2
lsnrctl status LISTENER_SCAN3

lsnrctl service LISTENER_SCAN1
lsnrctl service LISTENER_SCAN2
lsnrctl service LISTENER_SCAN3

su - oracle
$ORACLE_HOME/bin/srvctl add service -d racdb -s hrsrv -r racdb1,racdb2
$ORACLE_HOME/bin/srvctl add service -d racdb -s hrsrv -r racdb1 -a racdb2
$ORACLE_HOME/bin/srvctl add service -d racdb -s hrsrv -r racdb2 -a racdb1
$ORACLE_HOME/bin/srvctl add service -d racdb -s hrsrv -r racdb1

$ORACLE_HOME/bin/srvctl start service -d racdb -s hrsrv                    
```

