---
title: RAC
---

```bash
第三节点添加
集群环境
dns:172.25.5.11；域名:johnny.com
node1:(172.25.5.12 rac1)  (192.168.48.10 rac1-priv)  (172.25.5.20 rac1-vip)
node2:(172.25.5.13 rac2)  (192.168.48.11 rac1-priv)  (172.25.5.21 rac2-vip)
node3:(172.25.5.10 rac3)  (192.168.48.12 rac1-priv)  (172.25.5.22 rac3-vip)

修改系统内核参数
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

shell限制
vi /etc/security/limits.conf
------------------------------------------
#Oracle configure  shell parameters
oracle soft nofile 65536
oracle hard nofile 65536
oracle soft nproc 16384
oracle hard nproc 16384
#grid configure  shell parameters
grid soft nofile 65536
grid hard nofile 65536
grid soft nproc 16384
grid hard nproc 16384

node3修改网络配置：添加公网：172.25.5.10；心跳：192.168.48.12
vi /etc/sysconfig/network-scripts/ifcfg-eth1
`````````````````````````````
DEVICE=eth1
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.48.12
HWADDR=52:54:00:01:05:0a
TYPE=Ethernet

vi /etc/sysconfig/network-scripts/ifcfg-eth0
``````````````````````````
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
NETMASK=255.255.255.0
IPADDR=172.25.5.10
HWADDR=52:54:00:00:05:0a
GATEWAY=172.25.5.254
TYPE=Ethernet

网络重启动
service network restart

host文件:
node1：
vi /etc/hosts
`````````````````````
172.25.5.12 rac1.johnny.com rac1
172.25.5.13 rac2.johnny.com rac2
172.25.5.10 rac3.johnny.com rac3
172.25.5.20 rac1-vip.johnny.com rac1-vip
172.25.5.21 rac2-vip.johnny.com rac2-vip
172.25.5.22 rac3-vip.johnny.com rac3-vip
192.168.48.10 rac1-priv.johnny.com rac1-priv
192.168.48.11 rac2-priv.johnny.com rac2-priv
192.168.48.12 rac3-priv.johnny.com rac3-priv

node2：
vi /etc/hosts
`````````````````````
172.25.5.12 rac1.johnny.com rac1
172.25.5.13 rac2.johnny.com rac2
172.25.5.10 rac3.johnny.com rac3
172.25.5.20 rac1-vip.johnny.com rac1-vip
172.25.5.21 rac2-vip.johnny.com rac2-vip
172.25.5.22 rac3-vip.johnny.com rac3-vip
192.168.48.10 rac1-priv.johnny.com rac1-priv
192.168.48.11 rac2-priv.johnny.com rac2-priv
192.168.48.12 rac3-priv.johnny.com rac3-priv

node3：
vi /etc/hosts
`````````````````````
172.25.5.12 rac1.johnny.com rac1
172.25.5.13 rac2.johnny.com rac2
172.25.5.10 rac3.johnny.com rac3
172.25.5.20 rac1-vip.johnny.com rac1-vip
172.25.5.21 rac2-vip.johnny.com rac2-vip
172.25.5.22 rac3-vip.johnny.com rac3-vip
192.168.48.10 rac1-priv.johnny.com rac1-priv
192.168.48.11 rac2-priv.johnny.com rac2-priv
192.168.48.12 rac3-priv.johnny.com rac3-priv

dns服务器

vi /etc/hosts
`````````````````````
172.25.5.12 rac1.johnny.com rac1
172.25.5.13 rac2.johnny.com rac2
172.25.5.10 rac3.johnny.com rac3
172.25.5.20 rac1-vip.johnny.com rac1-vip
172.25.5.21 rac2-vip.johnny.com rac2-vip
172.25.5.22 rac3-vip.johnny.com rac3-vip
192.168.48.10 rac1-priv.johnny.com rac1-priv
192.168.48.11 rac2-priv.johnny.com rac2-priv
192.168.48.12 rac3-priv.johnny.com rac3-priv

修改主机名hostname
命令修改
hostname rac3.johnny.com
配置文件修改
vi /etc/sysconfig/network
````````````````````````
HOSTNAME=rac3.johnny.com

vi /etc/resolv.conf
search johnny.com
nameserver 172.25.5.11

添加dns
解析
vi /var/named/chroot/var/named/johnny.com.zone
``````````````````````````````````````````````````````````````````````````````````````````````````````````
$TTL    86400
@               IN SOA  dns.johnny.com.      root.johnny.com. (
                                        42              ; serial (d. adams)
                                        3H              ; refresh
                                        15M             ; retry
                                        1W              ; expiry
                                        1D )            ; minimum
@       IN      NS      dns.oracle.com.
rac1    IN      A       172.25.5.12
rac2    IN      A       172.25.5.13
rac3    IN      A       172.25.5.10
scan    IN      A       172.25.5.195
scan    IN      A       172.25.5.196
scan    IN      A       172.25.5.197
反响解析
vi /var/named/chroot/var/named/5.25.172.local
````````````````````````````````````````````````````````````````````````````````````````````````````````
$TTL    86400
@       IN      SOA     dns.johnny.com. root.johnny.com.  (
                                      1997022700 ; Serial
                                      28800      ; Refresh
                                      14400      ; Retry
                                      3600000    ; Expire
                                      86400 )    ; Minimum
@        IN      NS      dns.johnny.com.
10      IN      PTR     rac3.johnny.com
12      IN      PTR     rac1.johnny.com
13      IN      PTR     rac2.johnny.com
195     IN      PTR     scan.johnny.com
196     IN      PTR     scan.johnny.com
197     IN      PTR     scan.johnny.com

重启动
service named restart

共享磁盘
搜索iscsi：
iscsiadm -m discovery -t sendtargets -p 172.25.5.14 -l
添加udev
vi /etc/udev/rules.d/60-raw.rules
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
ACTION=="add", KERNEL=="sda1", RUN+="/bin/raw /dev/raw/raw1 %N"
ACTION=="add", KERNEL=="sda2", RUN+="/bin/raw /dev/raw/raw2 %N"
ACTION=="add", KERNEL=="sda3", RUN+="/bin/raw /dev/raw/raw3 %N"
ACTION=="add", KERNEL=="sda5", RUN+="/bin/raw /dev/raw/raw4 %N"
ACTION=="add", KERNEL=="sda6", RUN+="/bin/raw /dev/raw/raw5 %N"
KERNEL=="raw1", MODE="0660", GROUP="asmadmin", OWNER="grid"
KERNEL=="raw2", MODE="0660", GROUP="asmadmin", OWNER="grid"
KERNEL=="raw3", MODE="0660", GROUP="asmadmin", OWNER="grid"
KERNEL=="raw4", MODE="0660", GROUP="asmadmin", OWNER="grid"
KERNEL=="raw5", MODE="0660", GROUP="asmadmin", OWNER="grid"
启动udev
start_udev
检验裸设备是否成功加载
raw -qa
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[root@rac3 tmp]# raw -qa
/dev/raw/raw1:	bound to major 8, minor 1
/dev/raw/raw2:	bound to major 8, minor 2
/dev/raw/raw3:	bound to major 8, minor 3
/dev/raw/raw4:	bound to major 8, minor 5
/dev/raw/raw5:	bound to major 8, minor 6
清理oracle虚拟机环境建立node3
su - root(oracle虚拟机)
rm -rf /etc/ora*
rm -rf /u01/*
rm -rf /usr/local/bin/*

userdel -r oracle

groupdel dba
groupdel asmdba
groupdel asmadmin
groupdel oinstall
groupdel oracle

添加组
groupadd -g 501 oinstall
groupadd -g 502 dba
groupadd -g 503 oper
groupadd -g 504 asmadmin
groupadd -g 505 asmdba
groupadd -g 506 asmoper

添加用户
useradd -u 501 -g oinstall -G dba,oper,asmdba oracle
useradd -u 502 -g oinstall -G asmadmin,asmdba,asmoper grid

修改用户口令
passwd oracle 
--> oracle
passwd grid   
--> grid

修改/u01权限
mkdir -p /u01/grid
chown -R grid:oinstall /u01/

mkdir /u01/app/oracle
chown -R oracle:oinstall /u01/app/
chmod -R 775 /u01/
*如果grid与orcle安装在同一文件系统下面，如上，在进行权限修改的时候要注意顺序，先进行/u01赋权，再进行/u01/db赋权，否则会被覆盖。

修改用户环境变量
su - grid
vi .bashrc
`````````````````````````````
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/grid
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_OWNER=oracle
export ORACLE_SID=+ASM3 #rac2节点为 export ORACLE_SID=+ASM2
export ORACLE_TERM=vt100
export THREADS_FLAG=native
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
export LANG=en_US
alias sqlplus='rlwrap sqlplus'
alias lsnrctl='rlwrap lsnrctl'
alias asmcmd='rlwrap asmcmd'

su - oracle
vi .bashrc
`````````````````````````````````````````
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_OWNER=oracle
export ORACLE_SID=johnny3 
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
建立信任关系
su - oracle
node1:172.25.5.10
su - oracle
ssh-keygen -t rsa
ssh-keygen -t dsa
cd .ssh
cat *.pub > authorized_keys

su - oracle
node2:172.25.5.12
scp authorized_keys oracle@172.25.5.10:/home/oracle/.ssh/keys_dbs

su - oracle
node3:172.25.5.10
cat keys_dbs >> authorized_keys
scp authorized_keys oracle@172.25.5.13:/home/oracle/.ssh/
scp authorized_keys oracle@172.25.5.12:/home/oracle/.ssh/

node1、node2、node3都检测下
ssh rac1 date
ssh rac3 date
ssh rac2 date
ssh rac1-priv date
ssh rac2-priv date
ssh rac3-priv date

做好以上准备工作可以进行初步检测
在node1
su - grid
cd /tmp/grid(自己的grid解压包下)
./runcluvfy.sh stage -pre crsinst -n rac1,rac2 -fixup -verbose

node3安装软件
rpm cvuqdisk-1.0.9-1.rpm  -ivh

node3执行脚本
/tmp/CVU_11.2.0.4.0_grid/runfixup.sh

怎么检测针对15000
dns需要修改named.zone
zone "." IN {
        type hint;
//	file "named.ca";
        file "/dev/null";
};

每个节点vi /etc/resolv.conf添加
options rotate
options timeout:2
options attempts:5

所有准备工作做完后可以进行node3添加
检测硬件信息
node1
su - oracle
cluvfy stage -post hwos -n rac3 -verbose
基本可以忽略（教室虚拟机）
内核参数
node1
cluvfy stage -pre nodeadd -n rac3 -verbose
基本可以忽略（教室虚拟机）
软件包信息
node1
cluvfy comp peer -refnode rac2 -n rac3 -orainv oinstall -osdba oinstall-verbose
基本可以忽略（教室虚拟机）

执行数据备份(本人未测试)
/app/grid/bin/ocrconfig -manualbackup
/app/grid/bin/ocrdump /tmp/ocrdump_ocr.bak

Grid 层面添加新节点
node1 
su - grid
cd $ORACLE_HOME/oui/bin
忽略pre-checks（建议先不要设置，在检查一遍在进行设置）
export IGNORE_PREADDNODE_CHECKS=Y

./addNode.sh "CLUSTER_NEW_NODES={rac3}" "CLUSTER_NEW_VIRTUAL_HOSTNAMES={rac3-vip}"
最后需要按照提示执行脚本

oracle软件添加
node1
su - oracle
./addNode.sh "CLUSTER_NEW_NODES={rac3}"
最后执行脚本

dbca层面添加数据库配置
dbca -silent -addInstance -nodeList rac3 -gdbName johnny -instanceName johnny3 -sysDBAUserName sys -sysDBAPassword uplooking

大功告成功后可以进行测试
sqlplus / as sysdba
SQL> select INST_ID,status from gv$instance;

   INST_ID STATUS
---------- ------------
	 1 OPEN
	 3 OPEN
	 2 OPEN

```
















































