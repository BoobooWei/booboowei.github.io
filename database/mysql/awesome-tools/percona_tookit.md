---
title: Percona Toolkit工具明细说明
---

# 数据库版本和工具版本要求

| 数据库  | 版本         | *Percona Toolkit 版本（采用最新小版本）* | *Percona Toolkit 小版本*要求     |
| ------- | ------------ | ---------------------------------------- | -------------------------------- |
| MySQL   | 5.5          | *Percona Toolkit 2.2*                    | *Percona Toolkit 2.2.1以上版本*  |
| MySQL   | 5.6          | *Percona Toolkit 2.2*                    | *Percona Toolkit 2.2.1以上版本*  |
| MySQL   | 5.7          | *Percona Toolkit 2.2*                    | *Percona Toolkit 2.2.17以上版本* |
| MySQL   | 8.0          | *Percona Toolkit 3.0*                    | *Percona Toolkit 3.0.1*          |
| MariaDB | MariaDB 10.4 | *Percona Toolkit 3.0*                    | *Percona Toolkit 3.2.0*          |



- [Percona Toolkit迭代日志](https://www.percona.com/doc/percona-toolkit/LATEST/release_notes.html#v0-9-5-released-2011-08-04)
- [Percona Toolkit支持的平台和版本](https://www.percona.com/services/policies/percona-software-platform-lifecycle)

# Percona Toolkit工具细节说明

Percona Toolkit 工具执行需要两个用户：

1. 数据库用户：授权如下

   ```sql 
   CREATE USER 'solardb'@'%' IDENTIFIED BY 'SolarDB@123';
   GRANT ALL ON *.* TO 'solardb'@'%' with grant options;
   FLUSH PRIVILEGES;
   ```

2. 工具概览

https://www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html

| 工具命令 | 工具作用                 | 备注                                          |
| -------- | ------------------------ | --------------------------------------------- |
| 开发类   | pt-duplicate-key-checker | 列出并删除重复的索引和外键                    |
|          | pt-online-schema-change  | 在线修改表结构                                |
|          | pt-query-advisor         | 分析查询语句，并给出建议，有bug 已废弃        |
|          | pt-show-grants           | 规范化和打印权限                              |
|          | pt-upgrade               | 在多个服务器上执行查询，并比较不同            |
| 性能类   | pt-index-usage           | 分析日志中索引使用情况，并出报告              |
|          | pt-pmp                   | 为查询结果跟踪，并汇总跟踪结果                |
|          | pt-visual-explain        | 格式化执行计划                                |
|          | pt-table-usage           | 分析日志中查询并分析表使用情况 pt 2.2新增命令 |
| 配置类   | pt-config-diff           | 比较配置文件和参数                            |
|          | pt-mysql-summary         | 对mysql配置和status进行汇总                   |
|          | pt-variable-advisor      | 分析参数，并提出建议                          |
| 监控类   | pt-deadlock-logger       | 提取和记录mysql死锁信息                       |
|          | pt-fk-error-logger       | 提取和记录外键信息                            |
|          | pt-mext                  | 并行查看status样本信息                        |
|          | pt-query-digest          | 分析查询日志，并产生报告 常用命令             |
|          | pt-trend                 | 按照时间段读取slow日志信息 已废弃             |
| 复制类   | pt-heartbeat             | 监控mysql复制延迟                             |
|          | pt-slave-delay           | 设定从落后主的时间                            |
|          | pt-slave-find            | 查找和打印所有mysql复制层级关系               |
|          | pt-slave-restart         | 监控salve错误，并尝试重启salve                |
|          | pt-table-checksum        | 校验主从复制一致性                            |
|          | pt-table-sync            | 高效同步表数据                                |
| 系统类   | pt-diskstats             | 查看系统磁盘状态                              |
|          | pt-fifo-split            | 拟切割文件并输出                              |
|          | pt-summary               | 收集和显示系统概况                            |
|          | pt-stalk                 | 出现问题时，收集诊断数据                      |
|          | pt-sift                  | 浏览由pt-stalk创建的文件 pt 2.2新增命令       |
|          | pt-ioprofile             | 查询进程IO并打印一个IO活动表 pt 2.2新增命令   |
| 实用类   | pt-archiver              | 将表数据归档到另一个表或文件中                |
|          | pt-find                  | 查找表并执行命令                              |
|          | pt-kill                  | Kill掉符合条件的sql 常用命令                  |
|          | pt-align                 | 对齐其他工具的输出 pt 2.2新增命令             |
|          | pt-fingerprint           | 将查询转成密文 pt 2.2新增命令                 |

# 实践

## 在线DDL

注意:

```bash
1. {{ }} 或者 xx代表要修改的位置
2. --noversion-check 如果是非原生的MySQL，例如各大云产商的，必须要使用该参数
3. --charset=utf8 根据数据库使用的字符集选择合适的，否则容易造成乱码
4. --alter="xxx,xxx,xxx" 一张表多个变更使用逗号分割
```


### 场景一

{% note info %} 
一张表多个变更
{% endnote %}

```bash
[root@build-center cloudcare_dba]# cat qinxi/alter_table_hana_20191126.sh
#!/bin/bash
host=xx
port=xx
dbname=xx
user=xx
password=xx
  
table=xx
pt-online-schema-change --user=${user} --port=${port} --host=${host} --password=${password} --alter="add column update_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,drop index IX_ST_ENTRY_DETAIL_SRCNO_DETAILID,drop index auto_shard_key_ORG_ID,drop index auto_shard_key_WAREHOUSE_ID,drop index IX_ST_ENTRY_DETAIL_ITEMCODE,drop index IX_ST_ENTRY_DETAIL_INOUT_TYPE,drop index IX_ST_ENTRY_DETAIL_TOP_ITEMTYPE_ID,drop index IX_SOURCE_RECEIPT_TYPE,drop index IX_ST_ENTRY_DETAIL_ET_OPER_TIME,drop index IX_ST_ENTRY_DETAIL_ET_CREATE_TIME,drop index IX_ST_ENTRY_DETAIL_CALCULATE_ID,add index index_item_code(ITEM_CODE),add index index_trunk_id(TRUNK_ID),add index index_receipt_id(SOURCE_RECEIPT_ID),add index index_oper_time(OPER_TIME),add index IX_st_entry_detail_et_SOURCE_RECEIPT_NO(SOURCE_RECEIPT_NO),add index IX_st_entry_detail_et_ORG_ID(ORG_ID),add index IX_st_entry_detail_et_WAREHOUSE_ID(WAREHOUSE_ID),add index index_update_time(update_time),add index IX_st_entry_detail_et_IS_BOX(IS_BOX),add index IX_st_entry_detail_et_SOURCE_RECEIPT_TYPE(SOURCE_RECEIPT_TYPE),add index IX_st_entry_detail_et_CREATE_TIME(CREATE_TIME)" D=${dbname},t=${table} --no-version-check --execute
```

### 场景2

{% note info %} 
多张表变更
{% endnote %}


```bash 
#!/bin/bash
# pt-osc scripts
# {{ time_string }}
# Solar 混合云管理平台     
host={{ host }}
port={{ port }}
dbname={{ dbname }}
user={{ user }}
password={{ password }}
 

table={{ sql.table_name1 }}
pt-online-schema-change --user=${user} --port=${port} --host=${host} --password=${password} --alter="{{ sql.alter_cmd }}" D=${dbname},t=${table} --no-version-check --execute --charset=utf8

table={{ sql.table_name2 }}
pt-online-schema-change --user=${user} --port=${port} --host=${host} --password=${password} --alter="{{ sql.alter_cmd }}" D=${dbname},t=${table} --no-version-check --execute --charset=utf8


```


### 场景2

{% note info %} 
生产使用的自动生成PT脚本
{% endnote %}


python自动转换脚本：

```python 
# -*-coding:utf8 -*-
# Build-in Modules
import json
import re
import time
 
# 3rd-part Modules
import argparse
import sqlparse
from jinja2 import Template
 
 
class SQLParseHelper:
    """
    解析SQL
    """
 
    def __init__(self, sql_str=None, split_str=';'):
        self.sql_str = sql_str
        self.split_str = split_str
        self.sql_list = []
        if self.split_str == ';':
            # print(sqlparse.split(self.sql_str))
            self.sql_list = list(filter(
                lambda y: y != '', sqlparse.split(self.sql_str)
            ))
        else:
            self.sql_list = list(
                filter(
                    lambda y: y != '',
                    map(lambda x: x.strip(), self.sql_str.split(self.split_str))
                ))
        # print(self.sql_list)
 
    def get_length(self):
        """
        sql文件中的SQL数量
        :return:
        """
        return len(self.sql_list)
 
    def get_format(self, sql):
        """
        根据选项格式化sql。
        可用选项记录在“ SQL语句格式”中。
        除格式化选项外，该函数还接受关键字“ encoding”，该关键字确定语句的编码。
        返回值：    格式化的SQL语句为字符串。
        :return:
        """
        format_sql = sqlparse.format(sql, reindent=True, keyword_case='upper', strip_comments=True)
        # print(format_sql)
        return format_sql
 
    def get_init(self, sql):
        tokens = []
        parsed = sqlparse.parse(sql)
        stmt = parsed[0].tokens
        for token in stmt:
            tokens.append([token.ttype, token.value])
 
        sql = sqlparse.sql.Statement(stmt)
        sql_type = sql.get_type()
        return sql_type, tokens
 
    def get_type(self, tokens):
        # print(tokens)
        return tokens[0][0][1] if len(tokens[0][0]) > 1 else 'Others'
 
    def sql_statement(self, tokens):
        return tokens[0][1] if len(tokens[0]) > 1 else 'Others'
 
    def get_table(self, tokens):
        """
        如何从tokens中获取表名？
        :param tokens:
        :return:
        """
        tables = []
        try:
            # 找到 Keyword 为 TABLE的关键字的索引位 + 2
            sql_words = list(map(lambda x: x[1], tokens))
            index_num = sql_words.index('TABLE')
            table = sql_words[index_num + 2]
        except Exception as e:
            print(str(e))
        else:
            tables.append(table)
        return tables
 
 
class PTHelper:
    def __init__(self, sql):
        self.sql = sql
 
    def alter_sql_parse(self):
        """
        对同一张表按照原来的语句顺序用逗号连接,所有双引号替换为单引号
        [
                {"table_name": "frm_user",
                "alter_cmd": "DROP INDEX account_idx,ADD UNIQUE INDEX uniq_account(account)"
                "sql_from":[{
                    "original_sql": "alter table frm_user drop index account_idx;",
                    "format_sql": "ALTER TABLE frm_user\nDROP INDEX account_idx;",
                    "sql_type": "DDL",
                    "sql_statement": "ALTER"
                },
                {
                    "original_sql": "alter table frm_user add unique index uniq_account(account);",
                    "format_sql": "ALTER TABLE frm_user ADD UNIQUE INDEX uniq_account(account);",
                    "sql_type": "DDL",
                    "sql_statement": "ALTER"
                }]
        ]
 
        :return:
        """
        self.result = []
        for _sql in self.sql["data"]:
            # 正则匹配出 table_name 和 alter_cmd
            sql_str = _sql["format_sql"].replace('`', '')
            # print(sql_str)
            match_obj = re.match(
                r'ALTER TABLE(.*)[\n]?(ADD|ALTER|CHANGE|CHARACTER|CONVERT|DISABLE|ENABLE|DROP|FORCE|LOCK|MODIFY|ORDER|RENAME|WITHOUT|WITH)[\n]?(.*);',
                sql_str)
            if match_obj:
                table_name = match_obj.group(1).strip()
                alter_cmd_key = match_obj.group(2).strip()
                alter_cmd_option = match_obj.group(3).strip()
                # print(table_name)
                # print(alter_cmd_key)
                # print(alter_cmd_option)
                self.result.append(
                    {
                        "table_name": table_name.strip(),
                        "alter_cmd": "{} {}".format(alter_cmd_key.strip(),
                                                    alter_cmd_option.strip().replace('"', "'")),
                        "sql_from": _sql
                    }
                )
            else:
                print("不能正常匹配")
                self.result.append(
                    {
                        "table_name": "",
                        "alter_cmd": "",
                        "sql_from": _sql
                    }
                )
 
        # print(json.dumps(self.result, indent=2, ensure_ascii=False))
        return self.result
 
    def sql_to_pt(self):
        """
        组成pt脚本
        :return:
        """
        table_names = set(map(lambda x: x["table_name"], self.result))
        # print(table_names)
        out = []
        for table in table_names:
            # print(table)
            info = list(filter(lambda x: x["table_name"] == table, self.result))
            # print(json.dumps(info, indent=2))
            out.append(
                {
                    "table_name": table,
                    "alter_cmd": ','.join(list(map(lambda x: x["alter_cmd"], info))),
                    "sql_from": info
                }
            )
 
        # print(json.dumps(out, indent=2, ensure_ascii=False))
        return out
 
 
class GetScripts:
    def __init__(self, sql_list, **kwargs):
        self.render_data = {"sql_list": sql_list,
                            "time_string": time.strftime('%Y%m%d%H%M%S', time.localtime(time.time())),
                            "host": kwargs['host'],
                            "port": kwargs['port'],
                            "dbname": kwargs['dbname'],
                            "user": kwargs['user'],
                            "password": kwargs['password'],
                            }
 
    def render_template(self):
        template_data = """#!/bin/bash
# pt-osc scripts
# {{ time_string }}
# Solar 混合云管理平台     
host={{ host }}
port={{ port }}
dbname={{ dbname }}
user={{ user }}
password={{ password }}
 
{% for sql in sql_list %}
table={{ sql.table_name }}
pt-online-schema-change --user=${user} --port=${port} --host=${host} --password=${password} --alter="{{ sql.alter_cmd }}" D=${dbname},t=${table} --noversion-check --execute --charset=utf8
{% endfor %}
"""
        template = Template(template_data)
        return template.render(**self.render_data)
 
    def maker(self):
        data = self.render_template()
        return data
 
 
class SQLCheck:
    """
    检测三个条件:
    1. 检查语句中是否包含数据库名，如果包含则Fail;
    2. 检查语句结尾是否为分号，如果是则True；
    3. 判断SQL是否都为DDL Alter 语句，如果是则True；
 
    [
      {
        "original_sql": "ALTER TABLE `aia_use_app_tag_log`   ADD  INDEX `idx_app_use_time` (`use_time`)",
        "format_sql": "ALTER TABLE `aia_use_app_tag_log` ADD INDEX `idx_app_use_time` (`use_time`)",
        "sql_type": "DDL",
        "sql_statement": "ALTER",
        "check_alter": "Pass",
        "check_database": "Pass",
        "check_semicolon": "Fail",
        "check_comma": "Fail",
      }
    ]
    """
 
    def __init__(self, in_data_list):
        self.in_data_list = in_data_list
 
    def check_database(self, _data, pass_num, fail_num):
        """
        检查语句中是否包含数据库名，如果包含则Fail
        语句中不可以包含.
        """
        if _data["format_sql"].find('.') < 0:
            _data["check_database"] = "Pass"
            pass_num += 1
        else:
            _data["check_database"] = "Fail"
            fail_num += 1
        out_data = _data
        return pass_num, fail_num, out_data
 
    def check_semicolon(self, _data, pass_num, fail_num):
        """
        检查语句结尾是否为分号，如果是则True
        """
        if _data["format_sql"].find(';') >= 0:
            _data["check_semicolon"] = "Pass"
            pass_num += 1
        else:
            _data["check_semicolon"] = "Fail"
            fail_num += 1
        out_data = _data
        return pass_num, fail_num, out_data
 
    def check_alter(self, _data, pass_num, fail_num):
        """
        判断SQL是否都为DDL Alter 语句
        """
        if _data["sql_type"] == "DDL" and _data["sql_statement"] == "ALTER":
            _data["check_alter"] = "Pass"
            pass_num += 1
        else:
            _data["check_alter"] = "Fail"
            fail_num += 1
        out_data = _data
        return pass_num, fail_num, out_data
 
    def main(self):
        out_data_list = []
        pass_num = 0
        fail_num = 0
        for _data in self.in_data_list:
            pass_num, fail_num, out_data = self.check_alter(_data, pass_num, fail_num)
            pass_num, fail_num, out_data = self.check_database(_data, pass_num, fail_num)
            pass_num, fail_num, out_data = self.check_semicolon(_data, pass_num, fail_num)
            out_data_list.append(out_data)
 
        return {
            "pass_num": pass_num,
            "fail_num": fail_num,
            "data": out_data_list
        }
 
 
def start_up(**kwargs):
    # 1. 解析SQL SQLParseHelper()
    api = SQLParseHelper(sql_str=kwargs["sql_str"], split_str=";")
    sql_num = api.get_length()
    data = []
    for sql in api.sql_list:
        format_sql = api.get_format(sql)
        result = api.get_init(format_sql)
        data.append({
            "original_sql": sql,
            "format_sql": format_sql.strip(),
            "sql_type": api.get_type(tokens=result[1]),
            "sql_statement": api.sql_statement(tokens=result[1]),
        })
    new_result = {
        "num": sql_num,
        "data": data
    }
    # print(json.dumps(new_result, indent=2, ensure_ascii=False))
 
    # 2. 判断SQL是否都为DDL Alter 语句 check_alter()
    check_api = SQLCheck(new_result["data"])
    check_result = check_api.main()
    # print(json.dumps(check_result, indent=2, ensure_ascii=False))
    if check_result["fail_num"] > 0:
        print("检测不通过")
        print(json.dumps(
            list(filter(
                lambda x: x["check_alter"] == 'Fail' or x["check_database"] == 'Fail' or x[
                    "check_semicolon"] == 'Fail' or x["check_comma"] == "Fail",
                check_result["data"]
            ))
            , indent=2, ensure_ascii=False))
        # print(json.dumps(check_result, indent=2, ensure_ascii=False))
        exit()
    else:
        pass
        # print("检测通过")
        # print(json.dumps(check_result, indent=2, ensure_ascii=False))
 
    # 3. 转PT PTHelper()
    pt_api = PTHelper(check_result)
    pt_api.alter_sql_parse()
    pt_sql_list = pt_api.sql_to_pt()
 
    # 4. 渲染脚本
    scripts_api = GetScripts(pt_sql_list, **kwargs)
    data = scripts_api.maker()
    return data
 
 
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='''MySQL Alter语句自动转 PT脚本 小工具
    支持的Alter类型：ADD|ALTER|CHANGE|CHARACTER|CONVERT|DISABLE|ENABLE|DROP|FORCE|LOCK|MODIFY|ORDER|RENAME|WITHOUT|WITH ；
    不支持带库名；
    不支持一条sql多个变。
    输出结果
    #!/bin/bash
    # pt-osc scripts
    # 20200522190824
    # Solar 混合云管理平台     
    host=localhost
    port=3306
    dbname=test01
    user=root
    password=lsdkjkdjfkdf
 
 
    table=frm_user
    pt-online-schema-change --user=${user} --port=${port} --host=${host} --password=${password} --alter="DROP INDEX account_idx,ADD UNIQUE INDEX uniq_account(account)" D=${dbname},t=${table} --no-version-check --execute --charset=utf8
 
    table=aia_present_point_exchange_user
    pt-online-schema-change --user=${user} --port=${port} --host=${host} --password=${password} --alter="ADD COLUMN end_date DATETIME NULL COMMENT '有效期结束' AFTER start_date" D=${dbname},t=${table} --no-version-check --execute --charset=utf8
    "
 
    Example：
    python3 get_sql2pt.py --InFile demo/mysql_alter_demo.sql --OutFile demo/mysql_alter_pt.sh --Host localhost --Port 3306 --DBName test01 --User root --PassWord lsdkjkdjfkdf
    ''', formatter_class=argparse.RawTextHelpFormatter)
    parser.add_argument("--InFile", help="MySQL Alter SQL源文件 必要参数")
    parser.add_argument("--OutFile",
                        default='sql2pt-{}'.format(time.strftime('%Y%m%d%H%M%S', time.localtime(time.time()))),
                        help="转换后的脚本名 非参数")
    parser.add_argument("--Host", help="MySQL服务器连接地址")
    parser.add_argument("--Port", help="MySQL服务器监听端口")
    parser.add_argument("--DBName", help="MySQL访问的数据库名")
    parser.add_argument("--User", help="MySQL服务器登陆用户名")
    parser.add_argument("--PassWord", help="MySQL服务器登陆用户密码")
 
    args = parser.parse_args()
    if args.InFile and args.Host and args.Port and args.DBName and args.User and args.PassWord:
        sql_str = open(args.InFile, 'r', encoding='utf-8').read().replace('#', '')
        params = {
            "sql_str": sql_str,
            "out_file": args.OutFile,
            "host": args.Host,
            "port": args.Port,
            "dbname": args.DBName,
            "user": args.User,
            "password": args.PassWord
        }
        result = start_up(**params)
        with open(args.OutFile, 'w', encoding='utf-8') as f:
            f.write(result)
```

## 归档

```bash 
pt-archiver  --source h=${host},P=3306,u=${user},p=${password},D=${database},t=${table} --dest h=${host},P=3306,u=${user},p=${password},D=${database},t=${table}  --no-check-charset --where "use_time < '2020-01-01'" --progress 5000 --limit=1000 --txn-size=1000 --statistics --bulk-insert --bulk-delete --charset 'utf8' --noversion-check
```

参数说明：

`--source`: 源端
`--dest`: 目标端
`--dry-run`: debug模式，尝试运行。


