---
title: DBA的日常工作
---

```bash

内存管理风格的变化
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
oracle 8i
pga  --> manual
bitmap_merge_area_size=1048576
create_bitmap_area_size=8388608
hash_area_size=131072
sort_area_size=65536

sga  --> manual 内存组件的尺寸如果需要改变必须重起实例
shared_pool_size (共享池大小)
db_block_buffers (数据库缓冲区高速缓存)= 10000 (10000*db_block_size)
java_pool_size (java池)
large_pool_size (大池)
log_buffer (日志缓冲区高速缓存)
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
oracle 9i
pga  --> auto
workarea_size_policy= AUTO | manual (排序区的管理)
pga_aggregate_target=200m

sga  --> 半自动
sga_max_size (oracle启动之后sga使用物理内存的上限)
shared_pool_size (共享池大小)
db_cache_size (数据库缓冲区高速缓存)
java_pool_size (java池)
large_pool_size (大池)
*log_buffer (日志缓冲区高速缓存)
log_buffer之外的4个内存池，在不重新启动数据库的情况下大小可调整！尺寸之和不能超过sga_max_size！
管理员使用命令 alter system set .....; 收缩和扩张内存池！
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
oracle 10g
pga  --> auto
workarea_size_policy= AUTO | manual (排序区的管理)
pga_aggregate_target=200m

sga  --> auto 
sga_max_size (oracle启动之后sga使用物理内存的上限)
sga_target = 20G (自动管理的上限)
shared_pool_size (共享池大小)
db_cache_size (数据库缓冲区高速缓存)
java_pool_size (java池)
large_pool_size (大池)
streams_pool_size (流池)
*log_buffer (日志缓冲区高速缓存)

引入3个监视内存工作状态的进程
MMAN:内存管理
MMON:内存监控
MMNL:内存指示灯
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
oracle 11g ： PGA+SGA统一自动管理
memory_max_target :PGA+SGA上限
memory_target	  :PGA+SGA自动上限	
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
SGA自动管理:
sga_max_size --> sga的上限
sga_target   --> sga自动管理的上限

col name for a30
col value for a30
select name,value from v$parameter 
where name in 
('log_buffer',
'db_cache_size',
'shared_pool_size',
'large_pool_size',
'java_pool_size',
'streams_pool_size');

*SGA自动管理模式下，我们可以控制内存组件收缩的下限！
db_cache_size=200m 收缩的下限为200m
shared_pool_size=400m 收缩的下限为400m

内存页面预分配：
SQL> alter system set pre_page_sga=true scope=spfile;
将SGA锁定在物理内存：
SQL> alter system set lock_sga=true scope=spfile;

vi /etc/security/limits.conf
-----------------------------
oracle soft nporc 2047
oracle hard nporc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft memlock unlimited
oracle hard memlock unlimited

查看SGA分配建议
SQL> select SGA_SIZE,ESTD_DB_TIME from v$sga_target_advice;
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
SGA手工管理:
sga_target=0

调整共享池：
查找共享池中的sql语句
select sql_text from v$sqlarea where sql_text like 'select * from scott.%';

避免使用硬编码的字符条件：相似sql是垃圾
select substr(sql_text,1,30),count(*) from v$sqlarea having count(*)>10 group by substr(sql_text,1,30);

查看共享池大小
show parameter shared_pool

shared_pool_reserved_size=20M (共享池保留区大小)
shared_pool_size=400M(共享池大小)

SQL> select ksppinm,indx from x$ksppi where ksppinm like '%reserved%';

KSPPINM 			       INDX
-------------------------------- ----------
_shared_pool_reserved_min_alloc 	182

SQL> select KSPPSTDVL from x$ksppcv where indx=182;

KSPPSTDVL
---------
4400

查看共享池调整建议
SQL> select SHARED_POOL_SIZE_FOR_ESTIMATE,ESTD_LC_MEMORY_OBJECT_HITS from v$shared_pool_advice;

清空共享池：
alter system flush shared_pool;

create or replace procedure p1
is
begin
  null;
end;
/

手工将代码缓存到共享池 : LRU算法不释放keep状态的代码！
exec dbms_shared_pool.keep('P1');

select owner,name,type,kept
from v$db_object_cache
where type IN
('PACKAGE', 'PROCEDURE', 'TRIGGER', 'PACKAGE BODY');

exec dbms_shared_pool.unkeep('P1');
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
调整数据库缓冲区高速缓存：
SQL> show parameter db_cache_size

NAME				     TYPE	 VALUE
------------------------------------ ----------- -----
db_cache_size			     big integer 584M

查看数据库缓冲区高速缓存中的内容：
SQL> select data_object_id from user_objects where object_name='EMP';

DATA_OBJECT_ID
--------------
	 87108

SQL> select OBJD,FILE#,BLOCK# from v$bh where objd=87108;

      OBJD	FILE#	  BLOCK#
---------- ---------- ----------
     87108	    4	     151

清空数据库缓冲区高速缓：
alter system flush buffer_cache;

SET AUTOT[RACE] {OFF | ON | TRACE[ONLY]} [EXP[LAIN]] [STAT[ISTICS]]

SET AUTOT ON
SET AUTOT TRACE
SET AUTOT TRACE EXP
SET AUTOT TRACE STAT
SET AUTOT OFF

使用10046跟踪sql行为：
SQL> grant alter session to scott;
SQL> alter session set events '10046 trace name context forever ,level 12';
SQL> select * from emp;
SQL> alter session set events '10046 trace name context off';
tkprof db01_ora_4371.trc 1.txt

缓冲池的命中率：
SELECT name, 1 - (physical_reads /
(db_block_gets + consistent_gets)) "HIT_RATIO"
FROM v$buffer_pool_statistics
WHERE db_block_gets + consistent_gets > 0;

多缓冲池：

SQL> @?/rdbms/admin/awrrpti.sql
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
```


