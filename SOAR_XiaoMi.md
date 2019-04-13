---
title: soar 简单使用
tags:                      
- soar
categories:                
- 教程
date: 2018-11-5
---

SOAR 是小米开源的一款SQL优化工具
> <github.com/XiaoMi/soar>

## 编译安装

**注：编译安装没有成功，使用的二进制包**

### 基本依赖

```cnf
Go 1.10+
git
```

### 编译生成二进制文件

```bash
go get -d github.com/XiaoMi/soar
cd ${GOPATH}/src/github.com/XiaoMi/soar
make
```

## 安装验证

```bash
echo 'select * from film' | ./soar
```

## 升级 Go-1.10.1 on CentOS 6

> <https://tecadmin.net/install-go-lang-on-centos/>

### 下载安装包

```bash
go version
    go version go1.9.7 linux/amd64

wget https://dl.google.com/go/go1.10.1.linux-amd64.tar.gz

tar -xvf go1.10.1.linux-amd64.tar.gz
cp -r go /usr/local
cd /usr/local/go
```

### 设置Go语言环境变量

需要设置3个环境变量: GOROOT，GOPATH和PATH。

GOROOT是系统上安装Go软件包的位置
> export GOROOT=/usr/local/go

GOPATH是工作目录的位置。例如，我的项目目录是 ~/Projects/Proj1
> export GOPATH=$HOME/apps/app1

现在将PATH变量设置为二进制系统
> export PATH=$GOPATH/bin:$GOROOT/bin:$PATH

以上环境仅适用于您当前的会话。可以添加至 `~/.bash_profile`

### 验证

```bash
go version
    go version go1.10.1 linux/amd64

go env
```

## 下载二进制安装包

**此方法直接下载，添加执行权限即可运行**<br>
> 172.16.130.163:/home/soar/

```bash
wget https://github.com/XiaoMi/soar/releases/download/${tag}/soar.linux-amd64 -O soar
chmod a+x soar

# 如：
wget https://github.com/XiaoMi/soar/releases/download/v0.8.1/soar.linux-amd64 -O soar
chmod a+x soar

./soar --help
```

## 安装验证

```bash
echo 'select * from film' | ./soar
```

## 相关选项

```sql
./soar --help
Usage of ./soar:
  -allow-drop-index
        AllowDropIndex, 允许输出删除重复索引的建议
  -allow-online-as-test
        AllowOnlineAsTest, 允许线上环境也可以当作测试环境
  -blacklist string
        blacklist中的SQL不会被评审，可以是指纹，也可以是正则
  -config string
        Config file path
  -conn-time-out int
        ConnTimeOut, 数据库连接超时时间，单位秒 (default 3)
  -delimiter string
        Delimiter, SQL分隔符 (default ";")
  -drop-test-temporary
        DropTestTemporary, 是否清理测试环境产生的临时库表 (default true)
  -dry-run
        是否在预演环境执行 (default true)
  -explain
        Explain, 是否开启Explain执行计划分析 (default true)
  -explain-format string
        ExplainFormat [json, traditional] (default "traditional")
  -explain-max-filtered float
        ExplainMaxFiltered, filtered大于该配置给出警告 (default 100)
  -explain-max-keys int
        ExplainMaxKeyLength, 最大key_len (default 3)
  -explain-max-rows int
        ExplainMaxRows, 最大扫描行数警告 (default 10000)
  -explain-min-keys int
        ExplainMinPossibleKeys, 最小possible_keys警告
  -explain-sql-report-type string
        ExplainSQLReportType [pretty, sample, fingerprint] (default "pretty")
  -explain-type string
        ExplainType [extended, partitions, traditional] (default "extended")
  -explain-warn-access-type string
        ExplainWarnAccessType, 哪些access type不建议使用 (default "ALL")
  -explain-warn-extra string
        ExplainWarnExtra, 哪些extra信息会给警告 (default "Using temporary,Using filesort")
  -explain-warn-scalability string
        ExplainWarnScalability, 复杂度警告名单, 支持O(n),O(log n),O(1),O(?) (default "O(n)")
  -explain-warn-select-type string
        ExplainWarnSelectType, 哪些select_type不建议使用
  -ignore-rules string
        IgnoreRules, 忽略的优化建议规则 (default "COL.011")
  -index-prefix string
        IdxPrefix (default "idx_")
  -list-heuristic-rules
        ListHeuristicRules, 打印支持的评审规则列表
  -list-report-types
        ListReportTypes, 打印支持的报告输出类型
  -list-rewrite-rules
        ListRewriteRules, 打印支持的重写规则列表
  -list-test-sqls
        ListTestSqls, 打印测试case用于测试
  -log-level int
        LogLevel, 日志级别, [0:Emergency, 1:Alert, 2:Critical, 3:Error, 4:Warning, 5:Notice, 6:Informational, 7:Debug] (default 3)
  -log-output string
        LogOutput, 日志输出位置 (default "/dev/stderr")
  -markdown-extensions int
        MarkdownExtensions, markdown转html支持的扩展包, 参考blackfriday (default 94)
  -markdown-html-flags int
        MarkdownHTMLFlags, markdown转html支持的flag, 参考blackfriday
  -max-column-count int
        MaxColCount, 单表允许的最大列数 (default 40)
  -max-distinct-count int
        MaxDistinctCount, 单条SQL中Distinct的最大数量 (default 5)
  -max-group-by-cols-count int
        MaxGroupByColsCount, 单条SQL中GroupBy包含列的最大数量 (default 5)
  -max-in-count int
        MaxInCount, IN()最大数量 (default 10)
  -max-index-bytes int
        MaxIdxBytes, 索引总长度限制 (default 3072)
  -max-index-bytes-percolumn int
        MaxIdxBytesPerColumn, 索引中单列最大字节数 (default 767)
  -max-index-cols-count int
        MaxIdxColsCount, 复合索引中包含列的最大数量 (default 5)
  -max-index-count int
        MaxIdxCount, 单表最大索引个数 (default 10)
  -max-join-table-count int
        MaxJoinTableCount, 单条SQL中JOIN表的最大数量 (default 5)
  -max-pretty-sql-length int
        MaxPrettySQLLength, 超出该长度的SQL会转换成指纹输出 (default 1024)
  -max-query-cost int
        MaxQueryCost, last_query_cost 超过该值时将给予警告 (default 9999)
  -max-subquery-depth int
        MaxSubqueryDepth (default 5)
  -max-total-rows int
        MaxTotalRows, 计算散粒度时，当数据行数大于MaxTotalRows即开启数据库保护模式，不计算散粒度 (default 9999999)
  -max-varchar-length int
        MaxVarcharLength (default 1024)
  -online-dsn string
        OnlineDSN, 线上环境数据库配置, username:********@ip:port/schema
  -only-syntax-check
        OnlySyntaxCheck, 只做语法检查不输出优化建议
  -print-config
        Print configs
  -profiling
        Profiling, 开启数据采样的情况下在测试环境执行Profile
  -query string
        Queries for analyzing
  -query-time-out int
        QueryTimeOut, 数据库SQL执行超时时间，单位秒 (default 30)
  -report-css string
        ReportCSS, 当ReportType为html格式时使用的css风格，如不指定会提供一个默认风格。CSS可以是本地文件，也可以是一个URL
  -report-javascript string
        ReportJavascript, 当ReportType为html格式时使用的javascript脚本，如不指定默认会加载SQL pretty使用的javascript。像CSS一样可以是本地文件，也可以是一个URL
  -report-title string
        ReportTitle, 当ReportType为html格式时，HTML的title (default "SQL优化分析报告")
  -report-type string
        ReportType, 化建议输出格式，目前支持: json, text, markdown, html等 (default "markdown")
  -rewrite-rules string
        RewriteRules, 生效的重写规则 (default "delimiter,orderbynull,groupbyconst,dmlorderby,having,star2columns,insertcolumns,distinctstar")
  -sampling
        Sampling, 数据采样开关
  -sampling-statistic-target int
        SamplingStatisticTarget, 数据采样因子，对应postgres的default_statistics_target (default 100)
  -show-last-query-cost
        ShowLastQueryCost
  -show-warnings
        ShowWarnings
  -spaghetti-query-length int
        SpaghettiQueryLength, SQL最大长度警告，超过该长度会给警告 (default 2048)
  -table-allow-charsets string
        TableAllowCharsets (default "utf8,utf8mb4")
  -table-allow-engines string
        TableAllowEngines (default "innodb")
  -test-dsn string
        TestDSN, 测试环境数据库配置, username:********@ip:port/schema
  -trace
        Trace, 开启数据采样的情况下在测试环境执行Trace
  -unique-key-prefix string
        UkPrefix (default "uk_")
  -verbose
        Verbose
  -version
        Print version info
```

## 测试 一

### 常用命令

> <https://github.com/XiaoMi/soar/blob/master/doc/cheatsheet.md>

```sql
echo "select d.PROC_END_USERID,sum(cnt)
from (
select w.PROC_END_USERID, count(*) cnt
from edoc_base a
join edoc_base_workflow w
where w.EDOC_BASE_ID = a.ID
group by w.PROC_END_USERID
union all
select w.PROC_END_USERID, count(*) cn
from edoc_base a
join edoc_base_sendorgan c
on c.EDOC_BASE_ID = a.ID and a.DISPATCH_ORGAN = 'union'
join edoc_base_workflow w
on w.EDOC_BASE_ID = a.ID
group by w.PROC_END_USERID) d
group by d.PROC_END_USERID" | ./soar
```

### 下面为结果

#### 注：

以下的所有内容均为输出，直到 **测试 二** 为止<br>
soar 为了方便查看，自动进行了 markdown 格式的处理

```sql
# Query: EE5F2060C2AABDBB

☆ ☆ ☆ ☆ ☆ 0分

SELECT  
  d. PROC_END_USERID, SUM( cnt)
FROM  
  (
SELECT  
  w. PROC_END_USERID, COUNT( *
) cnt

FROM  
  edoc_base  a

  JOIN  edoc_base_workflow  w

WHERE  
  w. EDOC_BASE_ID  = a. ID

GROUP BY  
  w. PROC_END_USERID

UNION ALL  
SELECT  
  w. PROC_END_USERID, COUNT( *
) cn

FROM  
  edoc_base  a

  JOIN  edoc_base_sendorgan  c
 on  c. EDOC_BASE_ID  = a. ID  
  AND  a. DISPATCH_ORGAN  = 'union'
  JOIN  edoc_base_workflow  w
 on  w. EDOC_BASE_ID  = a. ID

GROUP BY  
  w. PROC_END_USERID) d

GROUP BY  
  d. PROC_END_USERID
```

## 建议使用AS关键字显示声明一个别名

* **Item:**  ALI.001

* **Severity:**  L0

* **Content:**  在列或表别名(如"tbl AS alias")中, 明确使用AS关键字比隐含别名(如"tbl alias")更易懂。

## 最外层SELECT未指定WHERE条件

* **Item:**  CLA.001

* **Severity:**  L4

* **Content:**  SELECT语句没有WHERE子句，可能检查比预期更多的行(全表扫描)。对于SELECT COUNT(\*)类型的请求如果不要求精度，建议使用SHOW TABLE STATUS或EXPLAIN替代。

## 在不同的表中GROUP BY或ORDER BY

* **Item:**  CLA.006

* **Severity:**  L4

* **Content:**  这将强制使用临时表和filesort，可能产生巨大性能隐患，并且可能消耗大量内存和磁盘上的临时空间。

## 请为GROUP BY显示添加ORDER BY条件

* **Item:**  CLA.008

* **Severity:**  L2

* **Content:**  默认MySQL会对'GROUP BY col1, col2, ...'请求按如下顺序排序'ORDER BY col1, col2, ...'。如果GROUP BY语句不指定ORDER BY条件会导致无谓的排序产生，如果不需要排序建议添加'ORDER BY NULL'。

## 使用SUM(COL)时需注意NPE问题

* **Item:**  FUN.006

* **Severity:**  L1

* **Content:**  当某一列的值全是NULL时，COUNT(COL)的返回结果为0,但SUM(COL)的返回结果为NULL,因此使用SUM()时需注意NPE问题。可以使用如下方式来避免SUM的NPE问题: SELECT IF(ISNULL(SUM(COL)), 0, SUM(COL)) FROM tbl

## 同一张表被连接两次

* **Item:**  JOI.002

* **Severity:**  L4

* **Content:**  相同的表在FROM子句中至少出现两次，可以简化为对该表的单次访问。

## MySQL对子查询的优化效果不佳

* **Item:**  SUB.001

* **Severity:**  L4

* **Content:**  MySQL将外部查询中的每一行作为依赖子查询执行子查询。 这是导致严重性能问题的常见原因。这可能会在 MySQL 5.6版本中得到改善, 但对于5.1及更早版本, 建议将该类查询分别重写为JOIN或LEFT OUTER JOIN。

## 不建议在子查询中使用函数

* **Item:**  SUB.006

* **Severity:**  L2

* **Content:**  MySQL将外部查询中的每一行作为依赖子查询执行子查询，如果在子查询中使用函数，即使是semi-join也很难进行高效的查询。可以将子查询重写为OUTER JOIN语句并用连接条件对数据进行过滤。

---
---

## 测试 二

### 配置文件

> <https://github.com/XiaoMi/soar/blob/master/doc/config.md>

```bash
$ cat << eof > soar.yaml
# 线上环境配置
online-dsn:
  addr: 172.16.50.102:3333
  schema: oa
  user: root
  password: password
  disable: false

# 测试环境配置
test-dsn:
  addr: 172.16.50.102:3333
  schema: oa
  user: root
  password: password
  disable: false

# 是否允许测试环境与线上环境配置相同
allow-online-as-test: true

# 是否清理测试时产生的临时文件
drop-test-temporary: true

# 语法检查小工具
only-syntax-check: false
sampling-data-factor: 100
sampling: true

# 日志级别，
# [0:Emergency, 1:Alert, 2:Critical, 3:Error, 4:Warning, 5:Notice, 6:Informational, 7:Debug]
log-level: 7
log-output: soar.log

# 优化建议输出格式
report-type: markdown
ignore-rules:
- ""
blacklist: soar.blacklist

# 启发式算法相关配置
max-join-table-count: 5
max-group-by-cols-count: 5
max-distinct-count: 5
max-index-cols-count: 5
max-total-rows: 9999999
spaghetti-query-length: 2048
allow-drop-index: false

# EXPLAIN相关配置
explain-sql-report-type: pretty
explain-type: extended
explain-format: traditional
explain-warn-select-type:
- ""
explain-warn-access-type:
- ALL
explain-max-keys: 3
explain-min-keys: 0
explain-max-rows: 10000
explain-warn-extra:
- ""
explain-max-filtered: 100
explain-warn-scalability:
- O(n)
query: ""
list-heuristic-rules: false
list-test-sqls: false
verbose: true
eof
```

### 测试内容

#### 更新 2018-10-31

命令来源：<br>
> <https://github.com/XiaoMi/soar/blob/master/doc/cheatsheet.md>

#### 一、

将相关日志输出到 `soar.log`，可以指定日志的级别<br>
关于日志的级别，上面的配置文件里面有<br>

* 0 Emergency
* 1 Alert
* 2 Critical
* 3 Error
* 4 Warning
* 5 Notice
* 6 Informational
* 7 Debug

```bash
echo "select * from mysql.user" | ./soar -log-output=soar1.log -log-level=7

more soar1.log
```

#### 二、

根据文档指定数据库进行连接，相关后台操作在debug日志里面有详细描述<br>
debug输出详细信息在本文最后的附加内容，可以搜索 **soar2.log** <br>
相关的输出内容，在附加内容里面的 **output2.log**

```sql
echo '' > soar2.log

echo "select d.PROC_END_USERID,sum(cnt)
from (
select w.PROC_END_USERID, count(*) cnt
from edoc_base a
join edoc_base_workflow w
where w.EDOC_BASE_ID = a.ID
group by w.PROC_END_USERID
union all
select w.PROC_END_USERID, count(*) cn
from edoc_base a
join edoc_base_sendorgan c
on c.EDOC_BASE_ID = a.ID and a.DISPATCH_ORGAN = 'union'
join edoc_base_workflow w
on w.EDOC_BASE_ID = a.ID
group by w.PROC_END_USERID) d
group by d.PROC_END_USERID" | ./soar -test-dsn="root:'password'@172.16.50.102:3333/oa" -allow-online-as-test -log-output=soar2.log -log-level=7
```

此处仅对日志内容进行简单分析

#### 日志分析

1. 程序通过执行 `select @@version` 判断实例是否可以正常使用
2. 执行了一些看不懂的操作
3. 对相应的OA库执行 `show create database oa`
4. 生成测试库 `optimizer_YQaWTm7nXCJXqE7w`
5. 对相关的表执行 `show create table tablename`
6. 在测试库里生成相同的表
7. 对表执行 `show table status where name = 'edoc_base_workflow'`
8. 随后执行了一串看不懂的操作，似乎执行了大量的改写的SQL
9. 对相关的索引与主键进行分析，比如提示索引列过长 varchar(500)
10. drop生成的临时库，清除相关数据

#### 三、

下面这条命令，使用上面的配置文件 `soar.yaml`<br>
此处的日志内容，也在本文最后的附加内容，搜索 **soar3.log** <br>
相关输出在附加内容后面的 **output3.log

**

```sql
echo "select d.PROC_END_USERID,sum(cnt)
from (
select w.PROC_END_USERID, count(*) cnt
from edoc_base a
join edoc_base_workflow w
where w.EDOC_BASE_ID = a.ID
group by w.PROC_END_USERID
union all
select w.PROC_END_USERID, count(*) cn
from edoc_base a
join edoc_base_sendorgan c
on c.EDOC_BASE_ID = a.ID and a.DISPATCH_ORGAN = 'union'
join edoc_base_workflow w
on w.EDOC_BASE_ID = a.ID
group by w.PROC_END_USERID) d
group by d.PROC_END_USERID" | ./soar -config=soar.yaml
```

---
---

## 测试 三

### explain输出信息分析

```sql
$ ./soar -report-type explain-digest << EOF
+------+-------------+------------+--------+----------------------------------------------------------------+----------------------------------+---------+-------------------+--------+-----------------------------------------------------------+
| id   | select_type | table      | type   | possible_keys                                                  | key                              | key_len | ref               | rows   | Extra                                                     |
+------+-------------+------------+--------+----------------------------------------------------------------+----------------------------------+---------+-------------------+--------+-----------------------------------------------------------+
|    1 | PRIMARY     | <derived2> | ALL    | NULL                                                           | NULL                             | NULL    | NULL              | 976531 | Using temporary; Using filesort                           |
|    2 | DERIVED     | w          | index  | EDOC_BASE_WORKFLOW_BASEID                                      | key_PROC_END_USERID_EDOC_BASE_ID | 135     | NULL              | 976510 | Using index                                               |
|    2 | DERIVED     | a          | eq_ref | PRIMARY,PK_EDOC_BASE                                           | PRIMARY                          | 66      | oa.w.EDOC_BASE_ID |      1 | Using index                                               |
|    3 | UNION       | a          | ref    | PRIMARY,PK_EDOC_BASE,uni2,IDX_EDOCBASE_DIS_,key_DISPATCH_ORGAN | key_DISPATCH_ORGAN               | 1003    | const             |     21 | Using where; Using index; Using temporary; Using filesort |
|    3 | UNION       | c          | ref    | key_EDOC_BASE_ID                                               | key_EDOC_BASE_ID                 | 67      | oa.a.ID           |      1 | Using index                                               |
|    3 | UNION       | w          | eq_ref | EDOC_BASE_WORKFLOW_BASEID                                      | EDOC_BASE_WORKFLOW_BASEID        | 66      | oa.a.ID           |      1 |                                                           |
+------+-------------+------------+--------+----------------------------------------------------------------+----------------------------------+---------+-------------------+--------+-----------------------------------------------------------+
EOF
```

### 输出内容

## Explain信息

| id | select\_type | table | partitions | type | possible_keys | key | key\_len | ref | rows | filtered | scalability | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1  | PRIMARY | *<derived2>* | NULL | ALL | NULL | NULL | NULL | NULL | 0 | 0.00% | ☠️ **O(n)** | Using temporary; Using filesort |
| 2  | DERIVED | *w* | NULL | index | EDOC\_BASE\_WORKFLOW\_BASEID | key\_PROC\_END\_USERID\_EDOC\_BASE\_ID | 135 | NULL | 0 | 0.00% | ☠️ **O(n)** | Using index |
| 2  | DERIVED | *a* | NULL | eq\_ref | PRIMARY,<br>PK\_EDOC\_BASE | PRIMARY | 66 | oa.w.EDOC\_BASE\_ID | 0 | 0.00% | ☠️ **O(n)** | Using index |
| 3  | UNION | *a* | NULL | ref | PRIMARY,<br>PK\_EDOC\_BASE,<br>uni2,<br>IDX\_EDOCBASE\_DIS\_,<br>key\_DISPATCH\_ORGAN | key\_DISPATCH\_ORGAN | 1003 | const | 0 | 0.00% | ☠️ **O(n)** | Using where; Using index; Using temporary; Using filesort |
| 3  | UNION | *c* | NULL | ref | key\_EDOC\_BASE\_ID | key\_EDOC\_BASE\_ID | 67 | oa.a.ID | 0 | 0.00% | ☠️ **O(n)** | Using index |
| 3  | UNION | *w* | NULL | eq\_ref | EDOC\_BASE\_WORKFLOW\_BASEID | EDOC\_BASE\_WORKFLOW\_BASEID | 66 | oa.a.ID | 0 | 0.00% | ☠️ **O(n)** |  |

### Explain信息解读

#### SelectType信息解读

* **PRIMARY**: 最外层的select.

* **DERIVED**: 用于from子句里有子查询的情况. MySQL会递归执行这些子查询, 把结果放在临时表里.

* **UNION**: UNION中的第二个或后面的SELECT查询, 不依赖于外部查询的结果集.

#### Type信息解读

* ☠️ **ALL**: 最坏的情况, 从头到尾全表扫描.

* **index**: 全表扫描, 只是扫描表的时候按照索引次序进行而不是行. 主要优点就是避免了排序, 但是开销仍然非常大.

* **eq_ref**: 除const类型外最好的可能实现的连接类型. 它用在一个索引的所有部分被连接使用并且索引是UNIQUE或PRIMARY KEY, 对于每个索引键, 表中只有一条记录与之匹配. 例:'SELECT * FROM ref_table,tbl WHERE ref_table.key_column=tbl.column;'.

* **ref**: 连接不能基于关键字选择单个行, 可能查找到多个符合条件的行. 叫做ref是因为索引要跟某个参考值相比较. 这个参考值或者是一个数, 或者是来自一个表里的多表查询的结果值. 例:'SELECT * FROM tbl WHERE idx_col=expr;'.

#### Extra信息解读

* ☠️ **Using temporary**: 表示MySQL在对查询结果排序时使用临时表. 常见于排序order by和分组查询group by.

* ☠️ **Using filesort**: MySQL会对结果使用一个外部索引排序,而不是从表里按照索引次序读到相关内容. 可能在内存或者磁盘上进行排序. MySQL中无法利用索引完成的排序操作称为'文件排序'.

* **Using index**: 只需通过索引就可以从表中获取列的信息, 无需额外去读取真实的行数据. 如果查询使用的列值仅仅是一个简单索引的部分值, 则会使用这种策略来优化查询.

* **Using where**: WHERE条件用于筛选出与下一个表匹配的数据然后返回给客户端. 除非故意做的全表扫描, 否则连接类型是ALL或者是index, 且在Extra列的值中没有Using Where, 则该查询可能是有问题的.

---
---

## 附加内容

**附加内容均为过长的日志内容，使用搜索查看**
由于debug日志过长，本文已将其删除

* soar2.log
* output2.log
* output3.log
* soar3.log


### output2.log

Query: EE5F2060C2AABDBB

☆ ☆ ☆ ☆ ☆ 0分

```sql

SELECT  
  d. PROC_END_USERID, SUM( cnt)
FROM  
  (
SELECT  
  w. PROC_END_USERID, COUNT( *
) cnt

FROM  
  edoc_base  a

  JOIN  edoc_base_workflow  w

WHERE  
  w. EDOC_BASE_ID  = a. ID

GROUP BY  
  w. PROC_END_USERID

UNION ALL  
SELECT  
  w. PROC_END_USERID, COUNT( *
) cn

FROM  
  edoc_base  a

  JOIN  edoc_base_sendorgan  c
 on  c. EDOC_BASE_ID  = a. ID  
  AND  a. DISPATCH_ORGAN  = 'union'
  JOIN  edoc_base_workflow  w
 on  w. EDOC_BASE_ID  = a. ID

GROUP BY  
  w. PROC_END_USERID) d

GROUP BY  
  d. PROC_END_USERID
```

## 建议使用AS关键字显示声明一个别名

* **Item:**  ALI.001

* **Severity:**  L0

* **Content:**  在列或表别名(如"tbl AS alias")中, 明确使用AS关键字比隐含别名(如"tbl alias")更易懂。

## 最外层SELECT未指定WHERE条件

* **Item:**  CLA.001

* **Severity:**  L4

* **Content:**  SELECT语句没有WHERE子句，可能检查比预期更多的行(全表扫描)。对于SELECT COUNT(\*)类型的请求如果不要求精度，建议使用SHOW TABLE STATUS或EXPLAIN替代。

## 在不同的表中GROUP BY或ORDER BY

* **Item:**  CLA.006

* **Severity:**  L4

* **Content:**  这将强制使用临时表和filesort，可能产生巨大性能隐患，并且可能消耗大量内存和磁盘上的临时空间。

## 请为GROUP BY显示添加ORDER BY条件

* **Item:**  CLA.008

* **Severity:**  L2

* **Content:**  默认MySQL会对'GROUP BY col1, col2, ...'请求按如下顺序排序'ORDER BY col1, col2, ...'。如果GROUP BY语句不指定ORDER BY条件会导致无谓的排序产生，如果不需要排序建议添加'ORDER BY NULL'。

## 使用SUM(COL)时需注意NPE问题

* **Item:**  FUN.006

* **Severity:**  L1

* **Content:**  当某一列的值全是NULL时，COUNT(COL)的返回结果为0,但SUM(COL)的返回结果为NULL,因此使用SUM()时需注意NPE问题。可以使用如下方式来避免SUM的NPE问题: SELECT IF(ISNULL(SUM(COL)), 0, SUM(COL)) FROM tbl

## 同一张表被连接两次

* **Item:**  JOI.002

* **Severity:**  L4

* **Content:**  相同的表在FROM子句中至少出现两次，可以简化为对该表的单次访问。

## MySQL对子查询的优化效果不佳

* **Item:**  SUB.001

* **Severity:**  L4

* **Content:**  MySQL将外部查询中的每一行作为依赖子查询执行子查询。 这是导致严重性能问题的常见原因。这可能会在 MySQL 5.6版本中得到改善, 但对于5.1及更早版本, 建议将该类查询分别重写为JOIN或LEFT OUTER JOIN。

## 不建议在子查询中使用函数

* **Item:**  SUB.006

* **Severity:**  L2

* **Content:**  MySQL将外部查询中的每一行作为依赖子查询执行子查询，如果在子查询中使用函数，即使是semi-join也很难进行高效的查询。可以将子查询重写为OUTER JOIN语句并用连接条件对数据进行过滤。

---
---

### output3.log

Query: EE5F2060C2AABDBB

☆ ☆ ☆ ☆ ☆ 0分

```sql

SELECT  
  d. PROC_END_USERID, SUM( cnt)
FROM  
  (
SELECT  
  w. PROC_END_USERID, COUNT( *
) cnt

FROM  
  edoc_base  a

  JOIN  edoc_base_workflow  w

WHERE  
  w. EDOC_BASE_ID  = a. ID

GROUP BY  
  w. PROC_END_USERID

UNION ALL  
SELECT  
  w. PROC_END_USERID, COUNT( *
) cn

FROM  
  edoc_base  a

  JOIN  edoc_base_sendorgan  c
 on  c. EDOC_BASE_ID  = a. ID  
  AND  a. DISPATCH_ORGAN  = 'union'
  JOIN  edoc_base_workflow  w
 on  w. EDOC_BASE_ID  = a. ID

GROUP BY  
  w. PROC_END_USERID) d

GROUP BY  
  d. PROC_END_USERID
```

## Explain信息

| id | select\_type | table | partitions | type | possible_keys | key | key\_len | ref | rows | filtered | scalability | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1  | PRIMARY | *<derived2>* | NULL | ALL |  | NULL |  |  | ☠️ **1028950** | ☠️ **100.00%** | ☠️ **O(n)** | Using temporary; Using filesort |
| 2  | DERIVED | *w* | NULL | ALL | EDOC\_BASE\_WORKFLOW\_BASEID | NULL |  |  | ☠️ **1028928** | ☠️ **100.00%** | ☠️ **O(n)** | Using temporary; Using filesort |
| 2  | DERIVED | *a* | NULL | eq\_ref | PRIMARY,<br>PK\_EDOC\_BASE | PRIMARY | 66 | oa.w.EDOC\_BASE\_ID | 1 | ☠️ **100.00%** | ☠️ **O(n)** | Using index |
| 3  | UNION | *a* | NULL | ref | PRIMARY,<br>PK\_EDOC\_BASE,<br>IDX\_EDOCBASE\_DIS\_,<br>key\_DISPATCH\_ORGAN | key\_DISPATCH\_ORGAN | 1003 | const | 22 | ☠️ **100.00%** | ☠️ **O(n)** | Using where; Using index; Using temporary; Using filesort |
| 3  | UNION | *c* | NULL | ref | key\_EDOC\_BASE\_ID | key\_EDOC\_BASE\_ID | 67 | oa.a.ID | 1 | ☠️ **100.00%** | ☠️ **O(n)** | Using index |
| 3  | UNION | *w* | NULL | eq\_ref | EDOC\_BASE\_WORKFLOW\_BASEID | EDOC\_BASE\_WORKFLOW\_BASEID | 66 | oa.a.ID | 1 | ☠️ **100.00%** | ☠️ **O(n)** | NULL |

### Explain信息解读

#### SelectType信息解读

* **PRIMARY**: 最外层的select.

* **DERIVED**: 用于from子句里有子查询的情况. MySQL会递归执行这些子查询, 把结果放在临时表里.

* **UNION**: UNION中的第二个或后面的SELECT查询, 不依赖于外部查询的结果集.

#### Type信息解读

* ☠️ **ALL**: 最坏的情况, 从头到尾全表扫描.

* **eq_ref**: 除const类型外最好的可能实现的连接类型. 它用在一个索引的所有部分被连接使用并且索引是UNIQUE或PRIMARY KEY, 对于每个索引键, 表中只有一条记录与之匹配. 例:'SELECT * FROM ref_table,tbl WHERE ref_table.key_column=tbl.column;'.

* **ref**: 连接不能基于关键字选择单个行, 可能查找到多个符合条件的行. 叫做ref是因为索引要跟某个参考值相比较. 这个参考值或者是一个数, 或者是来自一个表里的多表查询的结果值. 例:'SELECT * FROM tbl WHERE idx_col=expr;'.

#### Extra信息解读

* **Using filesort**: MySQL会对结果使用一个外部索引排序,而不是从表里按照索引次序读到相关内容. 可能在内存或者磁盘上进行排序. MySQL中无法利用索引完成的排序操作称为'文件排序'.

* **Using temporary**: 表示MySQL在对查询结果排序时使用临时表. 常见于排序order by和分组查询group by.

* **Using index**: 只需通过索引就可以从表中获取列的信息, 无需额外去读取真实的行数据. 如果查询使用的列值仅仅是一个简单索引的部分值, 则会使用这种策略来优化查询.

* **Using where**: WHERE条件用于筛选出与下一个表匹配的数据然后返回给客户端. 除非故意做的全表扫描, 否则连接类型是ALL或者是index, 且在Extra列的值中没有Using Where, 则该查询可能是有问题的.

## 为oa库的edoc\_base表添加索引

* **Item:**  IDX.001

* **Severity:**  L2

* **Content:**  为列ID添加索引,散粒度为: 100.00%; 为列DISPATCH\_ORGAN添加索引,散粒度为: 0.72%;

* **Case:** ALTER TABLE \`oa\`.\`edoc\_base\` add index \`idx\_ID\_DISPATCH\_ORGAN\` (\`ID\`,\`DISPATCH\_ORGAN\`(382)) ;

## 建议使用AS关键字显示声明一个别名

* **Item:**  ALI.001

* **Severity:**  L0

* **Content:**  在列或表别名(如"tbl AS alias")中, 明确使用AS关键字比隐含别名(如"tbl alias")更易懂。

## 最外层SELECT未指定WHERE条件

* **Item:**  CLA.001

* **Severity:**  L4

* **Content:**  SELECT语句没有WHERE子句，可能检查比预期更多的行(全表扫描)。对于SELECT COUNT(\*)类型的请求如果不要求精度，建议使用SHOW TABLE STATUS或EXPLAIN替代。

## 在不同的表中GROUP BY或ORDER BY

* **Item:**  CLA.006

* **Severity:**  L4

* **Content:**  这将强制使用临时表和filesort，可能产生巨大性能隐患，并且可能消耗大量内存和磁盘上的临时空间。

## 请为GROUP BY显示添加ORDER BY条件

* **Item:**  CLA.008

* **Severity:**  L2

* **Content:**  默认MySQL会对'GROUP BY col1, col2, ...'请求按如下顺序排序'ORDER BY col1, col2, ...'。如果GROUP BY语句不指定ORDER BY条件会导致无谓的排序产生，如果不需要排序建议添加'ORDER BY NULL'。

## 使用SUM(COL)时需注意NPE问题

* **Item:**  FUN.006

* **Severity:**  L1

* **Content:**  当某一列的值全是NULL时，COUNT(COL)的返回结果为0,但SUM(COL)的返回结果为NULL,因此使用SUM()时需注意NPE问题。可以使用如下方式来避免SUM的NPE问题: SELECT IF(ISNULL(SUM(COL)), 0, SUM(COL)) FROM tbl

## 同一张表被连接两次

* **Item:**  JOI.002

* **Severity:**  L4

* **Content:**  相同的表在FROM子句中至少出现两次，可以简化为对该表的单次访问。

## MySQL对子查询的优化效果不佳

* **Item:**  SUB.001

* **Severity:**  L4

* **Content:**  MySQL将外部查询中的每一行作为依赖子查询执行子查询。 这是导致严重性能问题的常见原因。这可能会在 MySQL 5.6版本中得到改善, 但对于5.1及更早版本, 建议将该类查询分别重写为JOIN或LEFT OUTER JOIN。

## 不建议在子查询中使用函数

* **Item:**  SUB.006

* **Severity:**  L2

* **Content:**  MySQL将外部查询中的每一行作为依赖子查询执行子查询，如果在子查询中使用函数，即使是semi-join也很难进行高效的查询。可以将子查询重写为OUTER JOIN语句并用连接条件对数据进行过滤。

