---
title: MySQL 5.7 导入大数据sql文件时经常报错(max_allowed_packet)解决办法
---

## 报错1

mysql默认配置，导入大数据sql时经常报错（超过100万条数据的表）：

```bash
[Err] 1153 – Got a packet bigger than ‘max_allowed_packet’ bytes
```

解决办法，找到配置文件，如：

max_allowed_packet = 1M

把max_allowed_packe 改成sql文件的大小，比如：500M

重启mysql

## 报错2

```bash
ERROR 2013 lost connection to MySQL server during query
```

可以先查看一下目前服务器的 max_allowed_packet 参数值是多少，命令为show variables like '%max_allowed_pack%'
如果是该参数值过小导致，则可以将配置文件中的 max_allowed_packet 值调大，例如调整至500M
第二，再检查 “wait_timeout”参数 ，是否是数据导入时间超过了数据库默认的超时时间，如果对比下来超时时间低于 数据导入时间，那么将该参数调高
