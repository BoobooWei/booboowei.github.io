---
title: DBA的日常工作
---

```bash

1.找到高资源消耗的sql
按照sql语句的成本查询前10名sql
select * from (select SQL_TEXT,OPTIMIZER_COST from v$sqlarea order by 2 desc nulls last) where rownum<11;
按照sql的物理读取查询前10名sql
select * from (select SQL_TEXT,DISK_READS from v$sqlarea order by 2 desc nulls last) where rownum<11;
按照sql的执行次数查询前10名sql
select * from (select SQL_TEXT,EXECUTIONSS from v$sqlarea order by 2 desc nulls last) where rownum<11;
按照sql的逻辑读的次数查询前10名sql
select * from (select SQL_TEXT,BUFFER_GETS from v$sqlarea order by 2 desc nulls last) where rownum<11;

2.抓取即时sql
col username for a20
col machine for a20
select username,machine,sid,serial#,sql_hash_value from v$session where username='SCOTT'

select sql_text from v$sqlarea where hash_value=52081782;

3.确定sql语句中涉及到的每个表的数据量
SQL> select count(*) from scott.stg_cusfund;

  COUNT(*)
----------
    838756

Elapsed: 00:00:00.06
SQL> select count(*) from scott.stg_settlement;

  COUNT(*)
----------
     21163

4.sql语句where条件中涉及到的列是否有索引
select table_name,column_name,index_name from user_ind_columns where lower(column_name)='recdate';

5.确定执行计划中是否合理使用了索引
set autot trace exp

6.使用sql强制（sql暗示）指定使用索引
SELECT 
      REPLACE(t.custfundacctincomp, ' ', ''),
      substr('20110512', 0, 6),
      SUM(t.equamt) equamt,
      20,
      SUM(ts.fee),
      SUM(ts.compfee),
      SUM(t.dayplmarkmkt)
FROM stg_cusfund t
LEFT JOIN stg_settlement ts ON TRIM(t.recdate) = TRIM(ts.recdate)
                            AND (TRIM(t.custfundacctincomp) = TRIM('0' || ts.custfundacctincomp) 
                                 OR
                                 TRIM(t.custfundacctincomp) = TRIM(ts.custfundacctincomp))
WHERE 
t.recdate >= substr('20110512', 0, 6) || '01'
AND t.recdate <= substr('20110512', 0, 6) || '31'
GROUP BY t.custfundacctincomp;

7.分析执行计划
读取执行计划的方法：从里向外，从上向下
查看索引的使用
索引的算法
查看表连接的算法

------------------------------------------------------------------------------------------
SELECT /*+use_hash(t,ts)*/
      REPLACE(t.custfundacctincomp, ' ', ''),
      substr('20110512', 0, 6),
      SUM(t.equamt) equamt,
      20,
      SUM(ts.fee),
      SUM(ts.compfee),
      SUM(t.dayplmarkmkt)
FROM stg_cusfund t
LEFT JOIN stg_settlement ts ON TRIM(t.recdate) = TRIM(ts.recdate)
                            AND (TRIM(t.custfundacctincomp) = TRIM('0' || ts.custfundacctincomp) 
                                 OR
                                 TRIM(t.custfundacctincomp) = TRIM(ts.custfundacctincomp))
WHERE 
t.recdate >= substr('20110512', 0, 6) || '01'
AND t.recdate <= substr('20110512', 0, 6) || '31'
GROUP BY t.custfundacctincomp;
------------------------------------------------------------------------------------------
SELECT 
      REPLACE(t.custfundacctincomp, ' ', ''),
      substr('20110512', 0, 6),
      SUM(t.equamt) equamt,
      20,
      SUM(ts.fee),
      SUM(ts.compfee),
      SUM(t.dayplmarkmkt)
FROM stg_cusfund t
LEFT JOIN stg_settlement ts ON TRIM(t.recdate) = TRIM(ts.recdate)
                            AND TRIM(t.custfundacctincomp) = TRIM(ts.custfundacctincomp)
WHERE 
t.recdate >= substr('20110512', 0, 6) || '01'
AND t.recdate <= substr('20110512', 0, 6) || '31'
GROUP BY t.custfundacctincomp;
------------------------------------------------------------------------------------------
通过操作系统高cpu消耗的pid反向获取sql语句:
select sql_text from v$sqlarea where hash_value in (select sql_hash_value from v$session s,v$process p where s.paddr=p.addr and p.spid=&pid); 

&pid --> 来自于top结果

sql语句的优化器模式：
RBO：基于规则的优化模式，根据sql语句的上下文生成一成不变的稳定的执行计划
CBO：基于成本的优化模式，根据sql语句的成本筛选执行计划

计算sql的成本：了解影响成本的因素，影响执行计划
create tablespace tbs1 datafile '/u01/app/oracle/oradata/orcl/tbs01.dbf' size 100m;

create table scott.t01 (x int, y varchar2(50)) pctfree 99 tablespace tbs1;

begin
  for i in 1..20000 loop
    insert into scott.t01 values (i,rpad('A',50,'A'));
  end loop;
  commit;
end;
/


SQL> analyze table scott.t01 compute statistics;

1.t01高水位以下的块的数量：
SQL> select blocks from dba_tables where table_name='T01';

    BLOCKS
----------
     20297

2.全表扫描时IO的吞吐量
select indx,ksppinm,KSPPDESC from x$ksppi where ksppinm like '%_db_file_optimizer_read_count%';

select KSPPSTDVL from X$KSPPCV where indx=1156;

_db_file_optimizer_read_count=8

3.成本计算公式:
cost=(io时间消耗+cpu时间消耗)/单块读取的时间
    = ceil(((20297/8)*(10+8192*8/4096)+(147943868/3074074.07))/12) = 5502
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
io时间消耗=io的次数*每次io的时间=(20297/8)*(10+8192*8/4096)
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～

读8个块怎么读？
先找到第一个然后连续读8个，硬盘上找到一个8k需要10毫秒，每返回一个8k需要2毫秒

查看系统统计信息:
select * from aux_stats$
SNAME		PNAME			  PVAL1 
--------------- -------------------- ---------- 
SYSSTATS_MAIN	CPUSPEEDNW	     3074.07407 --> cpu主频
SYSSTATS_MAIN	IOSEEKTIM		     10 --> io探查时间
SYSSTATS_MAIN	IOTFRSPEED		   4096 --> io传输速度，每毫秒4k
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
cpu时间消耗=语句在cpu上循环调用的次数/cpu的主频=147943868/3074074.07
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～

explain plan for select * from scott.t01;
SQL> select cpu_cost from plan_table;

  CPU_COST
----------
 147943868

20297是什么？怎么得到的？
begin
  for i in 1..5000 loop
    insert into scott.t01 values (i,rpad('A',50,'A'));
  end loop;
  commit;
end;
/

*在评估执行计划之前，先要确定统计信息的有效性!统计信息是老的执行计划就可能是错的！
查看对象被分析的时间：
select table_name,to_char(last_analyzed,'yyyy-mm-dd hh24:mi:ss') from user_tables;
查看N天以来没有分析过的表：
select table_name,to_char(last_analyzed,'yyyy-mm-dd hh24:mi:ss') from user_tables
where last_analyzed<sysdate-&n;
-----------------------------------------------
如何收集对象的统计信息：dbms_stats
收集单张表的统计信息:
begin
  DBMS_STATS.GATHER_TABLE_STATS('SCOTT','T01',estimate_percent=>10,degree=>8,cascade=>true);
end;
/

导出统计信息：
1.创建保存统计信息的表
begin
  DBMS_STATS.CREATE_STAT_TABLE ('SCOTT','STATSTAB');
end;
/

exec dbms_stats.delete_table_stats('SCOTT','T01');

2.导出对象或者用户下所有对象的统计信息：
begin
  DBMS_STATS.EXPORT_TABLE_STATS ('SCOTT','T01',stattab=>'STATSTAB');
end;
/

3.导入统计信息
begin
  DBMS_STATS.IMPORT_TABLE_STATS ('SCOTT','T01',stattab=>'STATSTAB');
end;
/

4.将统计信息导入到另一个数据库
在源将统计信息表，导出
exp scott/tiger tables=statstab file=1.dmp
导入到目标库
imp scott/tiger file=1.dmp full=y

begin
  DBMS_STATS.IMPORT_TABLE_STATS ('SCOTT','T01',stattab=>'STATSTAB');
end;
/

跨用户迁移统计信息需要修改c5列
begin
  DBMS_STATS.IMPORT_TABLE_STATS ('BLAKE','T01',stattab=>'STATSTAB');
end;
/

5.收集用户下所有表的统计信息:
begin
DBMS_STATS.GATHER_SCHEMA_STATS (
ownname=>'SCOTT',
estimate_percent=> DBMS_STATS.AUTO_SAMPLE_SIZE,
method_opt=>'FOR ALL INDEXED COLUMNS',
degree=>8,
cascade=>true,
options=>'GATHER AUTO');
end;
/

options=>'GATHER' --> 重新分析所有表
options=>'GATHER EMPTY' --> 只分析没有统计信息的表
options=>'GATHER STALE' --> 分析数据变化超过10%的表
options=>'GATHER AUTO' --> GATHER EMPTY + GATHER STALE
-------------------------------------------------------------------------

```
