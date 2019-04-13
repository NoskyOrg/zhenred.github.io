---
title: Percona Toolkit
tags:                      
- Percona Toolkit
categories:                
- 教程
date: 2018-11-5
---

此内容整理自中研软学习PPT

[在线文档](https://www.percona.com/doc/percona-toolkit/LATEST/index.html)

## 业务使用场景

- 线上集群新上线一张表，需要导入2亿条数据
- 扩容后数据一致性校验&修复
- 定期发送慢查询报告
- 业务异常的慢查询
- 在线添加、删除索引、字段
- kill连接对服务器信息进行汇总当系统出问题的时候收集重要的系统信息

## 安装

```bash
yum -y install percona-toolkit

or download with
https://www.percona.com/downloads/percona-toolkit/LATEST/
```

---

### 根据功能的不同，可以分为多个不同的类别

## 开发类举例

### pt-show-grants

- Usage: pt-show-grants [OPTIONS] [DSN]
- 规范化列出MySQL所有用户及权限，便于复制、比较和版本控制
- $ pt-show-grants

### pt-duplicate-key-checker

- Usage: pt-duplicate-key-checker [OPTIONS] [DSN]
- 用来检查mysql表是否有重复的索引和外键。
- 这个工具审查`show create table`的结果，
- 如果发现不同索引包含相同的列，或者一个覆盖索引最左边的列和其他索引一样，它会打印出可疑的索引，默认只会针对同一种类型的索引做比较
- 它也可以用来查找重复的外键，一个重复的外键必须是在同一个表中和其他外键包含相同的列，而且参考的父表也相同。

> $ pt-duplicate-key-checker --host localhost


### pt-upgrade

- Usage: pt-upgrade [OPTIONS] LOGS|RESULTS DSN [DSN]
- 在多个服务器上执行查询，并比较不同，可以比较慢日志、查询日志、binlog、tcpdump、raw；也可以保存分析的结果，与另一台主机进行比较。
- a：直接比较2台机器上slow的不同之处：
    - $ pt-upgrade master slave /var/lib/mysql/chen.slow
- b：先把master的slow保存后，再和slave进行比较不同：
    - $ mkdir host1_results
    - $ pt-upgrade h=master --save-results host1_results/
    - /var/lib/mysql/chen.slow
    - $ pt-upgrade host1_results/ h=slave
  
### pt-online-schema-change

- Usage: pt-online-schema-change [OPTIONS] DSN
- 不阻塞读写的方式在线改变表结构，执行前务必检查备份。

### pt-online-schema-change

#### 在线给表添加一个列

- $ pt-online-schema-change --alter "ADD COLUMN c1 INT" D=test,t=t1 --execute

#### 原理

1. 创建一张一样的新表
2. 修改新表表结构
3. 创建触发器，将老表正在变更的数据同步到新表上
4. Copy 数据到新表
5. Rename 新表
6. Drop 旧表
7. Drop 触发器

## 性能类举例

### pt-table-usage

- Usage: pt-table-usage [OPTIONS] [FILES]
- 分析日志中查询并分析表使用情况
- $ pt-table-usage /var/lib/mysql/chen.slow

### pt-visual-explain

- Usage: pt-visual-explain [OPTIONS] [FILES]
- 格式化执行计划
- $ mysql -e "explain select * from test.t" | pt-visual-explain
- $ mysql -e "explain select id from test.t" | pt-visual-explain

### pt-index-usage

- Usage: pt-index-usage [OPTIONS] [FILES]
- 分析日志中索引使用情况，并出报告
- $ pt-index-usage /var/lib/mysql/chen.slow --host localhost

### pt-pmp

- Usage: pt-pmp [OPTIONS] [FILES]
- 进行堆栈跟踪，并汇总跟踪结果。它可以在Linux上创建和总结完整的进程堆栈跟踪。堆栈跟踪的总结可以成为诊断进程等待的一个非常宝贵的工具。
- $ pt-pmp -b /usr/sbin/mysqld
- $ pt-pmp -p 1930

## 配置类举例

### pt-config-diff

- Usage: pt-config-diff [OPTIONS] CONFIG CONFIG [CONFIG...]
- 比较MySQL配置文件的不同之处。

### pt-mysql-summary

- Usage: pt-mysql-summary [OPTIONS]
- 返回MySQL的状态和配置统计信息。

#### 举例

- $ pt-mysql-summary --user=root
- 结果

```sql
Instances
MySQL Executable
Slave Hosts
Report On Port 3306
Processlist
Status Counters (Wait 10 Seconds)
Table cache
InnoDB
MyISAM
Security
Encryption
Binary Logging
Noteworthy Variables
Configuration File
```

### pt-variable-advisor

- 分析参数，并提出建议
- 分析本机上的参数设置：
- $ pt-variable-advisor localhost

## 监控类举例

### pt-deadlock-logger

- Usage: pt-deadlock-logger [OPTIONS] DSN
- 提取和记录mysql死锁信息

打印master机器上的死锁信息：

- $ pt-deadlock-logger -uroot -p123 h=master

将master上的死锁信息保存在master的percona_schema库的deadlocks表里：

- $ pt-deadlock-logger -uroot -p123 h=master --dest h=master ,D=percona_schema,t=deadlocks

将检测死锁进程放在后台一直执行：

- $ pt-deadlock-logger --daemonize --run-time=3000 --create-dest-table --dest D=test,t=deadlocks u=root,h=localhost

### pt-mext

- Usage: pt-mext [OPTIONS] --COMMAND
- 并行查看status样本信息
- $ pt-mext -r --mysqladmin ext -i10 -c3
    - --relative Subtract each column from the previous column.（差值）
    - extended-status Gives an extended status message from the server
    - -i, --sleep=# Execute commands repeatedly with a sleep between.
    - -c, --count=# Number of iterations to make. This works with -i

### pt-fk-error-logger

- Usage: pt-fk-error-logger [OPTIONS] [DSN]
- 进行外键错误检查，这个工具会一直运行检查，除非指定运行时间（--run-time）或者次数（--iterations）
- a：一直运行
    - $ pt-fk-error-logger h=master
- b：指定间隔、次数
    - $ pt-fk-error-logger h=master --run-time 1m --iterations 4 --interval 15

## 复制类举例

### pt-heartbeat

- Usage: pt-heartbeat [OPTIONS] [DSN] --update|--monitor|--check|--stop
- 监控mysql复制延迟

#### 原理

pt工具在主库上面创建一张测试表，以一秒的频率去更新这个的记录并把当前时间写入到字段中，
通过分析主从同步过来的时间和当前时间做对比得出时间差。

- 在主库上以后台方式建立heartbeat表并持续更新：
- $ /usr/bin/pt-heartbeat -uroot -p123 -D test --create-table --update -h master --daemonize

- 在从库上监控主从延迟情况：从库执行
    - $ /usr/bin/pt-heartbeat -uroot -p123 -D test --check -h slave --master-server-id=1 （单次）
    - $ /usr/bin/pt-heartbeat -uroot -p123 -D test --monitor -h slave --master-server-id=1 （持续监控，可以加上daemonize参数放在后台一直监控）
- 使用守护进程监控从库并输出日志：从库执行
    - $ /usr/bin/pt-heartbeat -uroot -p123 -D test --monitor -h slave --master-server-id=1 --print-master-server-id --daemonize --log=/tmp/slave-lag.log

### pt-slave-delay

- Usage: pt-slave-delay [OPTIONS] SLAVE_DSN [MASTER_DSN]
- 设定从落后主的时间

#### 原理

- 通过启动和停止复制sql线程来设置从落后于主指定时间
- 默认是基于从上relay日志的二进制日志的位置来判断，因此不需要连接到主服务器，
- 如果IO进程不落后主服务器太多的话，这个检查方式工作很好，
- 如果网络通畅的话，一般IO线程落后主通常都是毫秒级别。
- 一般是通过--delay and --delay"+"--interval来控制。
- --interval是指定检查是否启动或者停止从上sql线程的频繁度，默认的是1分钟检查一次。
- &ensp;
- a：使从落后主10s，并每隔1秒钟检测一次，运行10分钟
    - $ pt-slave-delay --user=root --password=123 --delay 10s --interval 1s --run-time 10m --host=slave
- b：使从落后主1分钟，并每隔1分钟检测一次，运行10分钟
    - $ pt-slave-delay --user=root --password=123 --delay 1m --run-time 10m --host=192.168.1.102
- 从库上执行查看延迟情况：
    - $ /usr/bin/pt-heartbeat -uroot -p123 -D test --monitor -h slave --master-server-id=1 --print-master-server-id
- 从5.6开始mysql在设定主从关系的时候可以通过延迟参数master_delay 指定

### pt-slave-find

- Usage: pt-slave-find [OPTIONS] [DSN]
- 以tree层级的方式查找和打印MySQL的主从关系
    - $ pt-slave-find --user=root --password=123 --host master

### pt-slave-restart

- 主从库都可以执行，变换host即可
- Usage: pt-slave-restart [OPTIONS] [DSN]
- 监控salve错误，如果从库停止尝试重启salve
- a：监视从，跳过1个错误
    - $ pt-slave-restart --user=root --password=123 --host=slave --skip-count=1
- b：监视从，跳过错误代码为1062的错误。
    - $ pt-slave-restart --user=root --password=123 --host=slave --error-numbers=1062

### pt-table-checksum

- Usage: pt-table-checksum [OPTIONS] [DSN]
- 校验主从复制一致性

#### 原理

通过在主上运行该工具计算每个表的散列值，利用主从关系，把同样的计算过程在从库上重放，从而拿到主从各自的散列值，只要比较散列值是否相同就可以了，所以必须保证有个账号可以同时在主从上执行这些操作。

```sql
create database pt charset utf8;
grant update,insert,delete,select,process,super,replication slave on *.* to 'checksum'@'%' identified by 'checksum';
grant all on pt.* to checksum@'%';
```

pass have a picture

### pt-table-checksum

- $ pt-table-checksum --nocheck-binlog-format --create-replicate-table --replicate=pt.checksums -uchecksum -pchecksum -hmaster--recursion-method=hosts

遇到错误：Diffs cannot be detected because no slaves were found.

#### 原因

默认是通过`show slave hosts`的值，默认找不到，可以设置hosts模式，可以在从库的my.cnf添加如下2个参数解决：非动态参数，`需重启`
> report_host=192.168.120.15 
> report_port=3306

参数的目的是上报自己的ip或者主机名和端口，这样就可以在主库上通过`show slave hosts`看到该主库上的从库信息。

- 参数
- --replicate-check-only
- 只输出不一致的表信息

#### 输出内容

- TS：完成检查的时间
- ERRORS：检查时候发生错误和警告的数量
- DIFFS：0表示一致，1表示不一致。当指定--no-replicate-check时，会一直为0，当指定--replicate-check-only会显示不同的信息。
- ROWS ：表的行数。
- CHUNKS ：被划分到表中的块的数目。
- SKIPPED ：由于错误或警告或过大，则跳过块的数目。
- TIME ：执行的时间。
- TABLE ：被检查的表名。

备注：主要观察DIFFS列，标识差异数。

### pt-table-sync

- Usage: pt-table-sync [OPTIONS] DSN [DSN]
- 高效同步表数据，修复的表上必须有主键或者唯一键

先查看哪些不一致的地方：

- $ pt-table-checksum --nocheck-binlog-format --create-replicate-table --replicate=test.checksums -uchecksum -pchecksum -hmaster --recursion-method=hosts

进行修复：

- $ pt-table-sync --replicate=pt.checksums h=master,u=checksum,p=checksum h=slave,u=checksum,p=checksum --execute

第一个h是主，第二个h是从，还可以使用--sync-to-master 指定从的信息：

- $ pt-table-sync --replicate=test.checksums --sync-to-master h=slave,u=checksum,p=checksum --execute

再次查看是否存在不一致：

- $ pt-table-checksum --nocheck-binlog-format --create-replicate-table --replicate=test.checksums -uchecksum -pchecksum -hmaster --recursion-method=hosts


## 系统类举例

### pt-diskstats

- Usage: pt-diskstats [OPTIONS] [FILES]
- 查看系统磁盘IO相关信息。会自动隐藏空闲的磁盘，可以非常方便地快速地深入到输入/输出性能和检查磁盘行为。
- ？帮助菜单
- $ pt-diskstats --interval 1

通过看man或者官方手册查看各个列的含义：

> #ts device rd_s rd_avkb rd_mb_s rd_mrg rd_cnc rd_rt wr_s wr_avkb wr_mb_s wr_mrg wr_cnc wr_rt busy in_prg io_s qtime stime

### pt-fifo-split

- Usage: pt-fifo-split [OPTIONS] [FILE]
- 模拟切割文件并输出

一个窗口进行大文件分割：

- pt-fifo-split --lines 1000000 hugefile.txt --fifo /tmp/fifobig

另一个窗口执行：

```sql
while [ -e /tmp/fifobig]; do
    time mysql -e "set foreign_key_checks=0; set sql_log_bin=0; set unique_checks=0; load data local infile '/tmp/my-fifo' into table load_test fields terminated by '\t' lines terminated by '\n' (col1, col2);"
    sleep 1;
done
```

### pt-summary

- Usage: pt-summary
- 查看系统的相关统计信息。
- $ pt-summary

### pt-stalk

- 出现问题时，收集诊断数据
- 收集的信息非常的全面，包括大量的系统状态（IO、CPU、网卡）等以及大量MySQL信息（processlist、innodb status、variables等）
- function有三种：status、processlist、和自定义脚本
- &ensp;
- 举例：status
- $ pt-stalk --user=root --password=123 --function=status --variable=Threads_running --threshold=5 --cycles=5 --interval=1 --iterations=3 --run-time=30 --sleep=30 --dest=/tmp/pt-stalk --log=/var/log/pt-stalk.log --pid=/var/run/pt-stalk.pid

### pt-sift

- Usage: pt-sift FILE|PREFIX|DIRECTORY
- 浏览由pt-stalk创建的文件

### pt-ioprofile

- Usage: pt-ioprofile [OPTIONS] [FILE]
- 查询进程IO并打印一个IO活动表
    - $ pt-ioprofile -p 7332 -c sizes
    - $ pt-ioprofile -p 7332 -c count
    - $ pt-ioprofile -p 7332 -c times

## 实用类举例

### pt-align

- Usage: pt-align [FILES]
- 在文件中如果行列不是对齐的话，使用该命令查看文件内容可以看到行列自动对齐
- 比较类似 df -P

### pt-archiver

- Usage: pt-archiver [OPTIONS] --source DSN --where WHERE

数据库归档工具，查询源数据，导出数据到文件\表\不做操作，然后删除源数据。小心使用，这个工具在归档数据以后默认会把源数据删除，所以需要手工加上-no-delete，我少有归档需求，而且偶尔用不熟练的话，风险也较大。

注意：表中要有索引 | 主键举例：

- 将t表的行数据输出到一个文件中去，原表数据就清空了，请慎重操作
- $ pt-archiver --user=root --password=123 --charset=utf8 --source h=192.168.120.14,D=pttest,t=t --file '/tmp/%Y-%m-%d-%D.%t' --where "1=1" --limit 1000 --commit-each

#### 常用命令

- 删除老数据（单独删除操作不用指定字符集）：
    - /usr/bin/pt-archiver --source h=localhost,u=root,p=123,P=3306,D=test,t=t --no-check-charset --where ‘a<=376‘--limit 10000 --txn-size 1000 --purge
- 复制数据到其他mysql实例，且不删除source的数据(指定字符集)：
    - /usr/bin/pt-archiver --source h=localhost,u=root,p=1234,P=3306,D=test,t=t1 --dest h=192.168.2.12,P=3306,u=archiver,p=archiver,D=test,t=t1_bak --progress 5000 --where 'mc_id<=125' --statistics --charset=UTF8 --limit=10000 --txn-size 1000 --no-delete
- 复制数据到其他mysql实例，并删source上的旧数据(指定字符集)：
    - /usr/bin/pt-archiver --source h=localhost,u=root,p=1234,P=3306,D=test,t=t1 --dest h=192.168.2.12,P=3306,u=archiver,p=archiver,D=test,t=t1_his --progress 5000 --where "CreateDate <'2017-05-01 00:00:00' " --statistics --charset=UTF8 --limit=10000 --txn-size 1000 --bulk-delete

#### 官方文档说明

> The normal method is to delete every row by its primary key. Bulk deletes might be a lot faster.They also might not be faster if you have a complex WHERE clause.

通常的方法是会通过他们的主键去删除每行数据，批量删除可能会非常快。如果你有复杂的where条件的话，它们也可能不会更快。

- 复制数据到其他mysql实例，不删除source数据，但是使用批量插入dest上新的数据(指定字符集)：
    - /usr/bin/pt-archiver --source h=localhost,u=archiver,p=archiver,P=3306,D=test,t=t1 --dest h=192.168.2.12,P=3306,u=archiver,p=archiver,D=test,t=t1_his --progress 5000 --where "c <'2017-05-01 00:00:00' " --statistics --charset=UTF8 --limit=10000 --txn-size 1000 --no-delete --bulk-insert
    - 测试用的一张只有3列元素的表，共计9万行数据。使用bulk-insert用时7秒钟。而常规insert用时40秒。
- 导出数据到文件：
    - /usr/bin/pt-archiver --source h=10.0.20.26,u=root,p=1234,P=3306,D=test,t=t --file '/root/test.txt' --progress 5000 --where 'a<12000' --no-delete --statistics --charset=UTF8 --limit=10000 --txn-size 1000
- 导出数据到文件并删除数据库的相关行：
    - /usr/bin/pt-archiver --source h=10.0.20.26,u=root,p=1234,P=3306,D=test,t=t --file '/root/test.txt' --progress 5000 --where 'a<12000' --statistics --charset=UTF8 --limit=10000 --txn-size 1000 --purge

Specify at least one of "--dest","--file", or "--purge".
下面几个参数都是互斥的，只能选其一

- "--ignore"and "--replace" are mutually exclusive.
- "--txn-size"and "--commit-each" are mutually exclusive.
- "--low-priority-insert"and "--delayed-insert" are mutually exclusive.
- "--share-lock"and "--for-update" are mutually exclusive.
- "--analyze"and "--optimize" are mutually exclusive.
- "--no-ascend"and "--no-delete" are mutually exclusive.

#### 常用的参数

- --limit10000 每次取1000行数据用pt-archive处理，Number of rows to fetch and archive per statement.
- --txn-size 1000 设置1000行为一个事务提交一次，Number of rows pertransaction.
- --where‘id<3000‘ 设置操作条件
- --progress5000 每处理5000行输出一次处理信息
- --statistics 输出执行过程及最后的操作统计。（只要不加上--quiet，默认情况下pt-archive都会输出执行过程的）
- --charset=UTF8 指定字符集为UTF8
- --bulk-delete 批量删除source上的旧数据(例如每次1000行的批量删除操作)
- --bulk-insert 批量插入数据到dest主机(看dest的general log发现它是通过在dest主机上LOAD DATA LOCAL INFILE插入数据的)
- --replace 将insert into 语句改成replace写入到dest库
- --sleep120 每次归档了limit个行记录后的休眠120秒（单位为秒）
- --file‘/root/test.txt‘
- --purge 删除source数据库的相关匹配记录
- --header 输入列名称到首行（和--file一起使用）
- --no-check-charset 不指定字符集
- --check-columns 检验dest和source的表结构是否一致，不一致自动拒绝执行（不加这个参数也行。默认就是执行检查的）
- --no-check-columns 不检验dest和source的表结构是否一致，不一致也执行（会导致dest上的无法与source匹配的列值被置为null或者0）
- --chekc-interval 默认1s检查一次
- --local 不把optimize或analyze操作写入到binlog里面（防止造成主从延迟巨大）
- --retries 超时或者出现死锁的话，pt-archiver进行重试的间隔（默认1s）
- --no-version-check 目前为止，发现部分pt工具对阿里云RDS操作必须加这个参数
- --analyze=ds 操作结束后，优化表空间（d表示dest，s表示source）

默认情况下，pt-archiver操作结束后，不会对source、dest表执行analyze或optimize操作，因为这种操作费时间，并且需要你提前预估有足够的磁盘空间用于拷贝表。一般建议也是pt-archiver操作结束后，在业务低谷手动执行analyze table用以回收表空间。

- bug：不会迁移`max(id)`那条数据
- 解决办法：找到`# vim /usr/bin/pt-archiver`的6401行，修改`<`为`<=`

```s
$first_sql .= " AND ($col < " . $q->quote_val($val) . ")";
$first_sql .= " AND ($col <= " . $q->quote_val($val) . ")";
```

### pt-find

- Usage: pt-find [OPTIONS] [DATABASES]
- 查找表并执行命令
- a：查找一天前创建的所有innodb引擎的表
    - $ pt-find --ctime +1 --engine innodb
- b：找到大于5M的表
    - $ pt-find --tablesize +5M
- c：在test数据库中找到空表并删除该表：
    - pt-find --empty test --exec-plus "DROP TABLE %s"

### pt-kill

- Usage: pt-kill [OPTIONS] [DSN]
- Kill掉符合条件的sql
- pt-kill 是一个简单而且很实用的查杀mysql线程和查询的工具
- 主要是为了防止一些大/复杂/长时间查询占用数据库及系统资源，而对线上业务造成影响的情况。
- a：杀掉运行时间超过60s的会话
    - $ pt-kill --busy-time 60 --kill
- b：每10s检查Sleep的线程并杀死
    - $ pt-kill --match-commandSleep --kill --victims all --interval 10
- c：打印出所有状态是login的线程
    - $ pt-kill --match-statelogin --print --victims all
- 如何去查找红色命令和状态有哪些值？——官方文档
- d：安装info关键字来查杀会话
    - 注：要严格匹配大小写
    - --ignore-info 不匹配
    - --match-info 匹配
    - info可以使用select、update、insert、delete来进行匹配，并可使用"|"进行多项匹配，如"select|SELECT|delete|DELETE|update|UPDATE"
- e：按照访问来源host/ip查杀线程
    - --ignore-host/--match-host
- f：按照DB来查杀线程
    - --ignore-db/--match-db
- g：按照数据库用户
    - --ignore-user/--match-user
    - Action：
    - --kill 杀掉连接并且退出
    - --kill-query 只杀掉连接执行的语句，但是线程不会被终止
    - --print 打印满足条件的语句

### pt-fingerprint

- Usage: pt-fingerprint [OPTIONS] [FILES]
- 将查询转成密文
- a：转换一个单独的查询：
    - $ pt-fingerprint --query "select id from test.t1;"
- b：转换整个查询文件
    - $ pt-fingerprint /path/to/file.txt

## 日志分析工具 pt-query-digest

> Usage: pt-query-digest [OPTIONS] [FILES] [DSN]

### 常用参数

- --create-review-table：
    - 当使用--review参数把分析结果输出到表中时，如果没有表就自动创建。
- --create-history-table：
    - 当使用--history参数把分析结果输出到表中时，如果没有表就自动创建。
- --filter ：
    - 对输入的慢查询按指定的字符串进行匹配过滤后再进行分析
- --limit：
    - 限制输出结果百分比或数量，默认值是20,即将最慢的20条语句输出，
    - 如果是95%则按总响应时间占比从大到小排序，输出到总和达到95%位置截止。
- --log=s ：
    - 指定输出的日志文件
- --history：
    - 将分析结果保存到表中，分析结果比较详细，
    - 下次再使用--history时，如果存在相同的语句，且查询所在的时间区间和历史表中的不同，则会记录到数据表中，
    - 可以通过查询同一CHECKSUM来比较某类型查询的历史变化。
- --review：
    - 将分析结果保存到表中，这个分析只是对查询条件进行参数化，一个类型的查询一条记录，比较简单。
    - 当下次使用--review时，如果存在相同的语句分析，就不会记录到数据表中。
- --output：
    - 分析结果输出类型，值可以是report(标准分析报告)、slowlog(Mysql slow log)、json、json-anon，
    - 一般使用report，以便于阅读。
- --since：
    - 从该指定日期开始分析。
- --until：
    - 截止时间，配合—since可以分析一段时间内的慢查询。

### 使用示例

1. 分析慢日志日志（——会将分析结果输出到屏幕上）
    - $ pt-query-digest --report cisohm-slow.log
2. 报告最近半个小时的慢日志
    - $ pt-query-digest --report --since 1800s cisohm-slow.log
3. 报告一个时间段的慢查询：
    - $ pt-query-digest --report --since '2018-02-10 00:48:00' --until '2018-08-06 00:00:00' cisohm-slow.log
4. 报告只含select语句的慢查询:
    - $ pt-query-digest --filter '$event->{fingerprint} =~ m/^select/i' --limit=20% cisohm-slow.log
5. 报告针对root用户的慢查询:
    - $ pt-query-digest --filter '($event->{user} || "") =~ m/^root/i' cisohm-slow.log
6. 报告所有的全表扫描或full join的慢查询:
    - $ pt-query-digest --filter '((\$event->{Full_scan} || "") eq "yes") || (($event->{Full_join} || "") eq "yes")' cisohm-slow.log
7. 把查询保存到query_review表
    - $ pt-query-digest --user=root --password=123 --review h=localhost,D=test --no-report --create-review-table cisohm-slow.log
    - select * from query_review limit 1 \G
8. 把查询保存到query_history表
    - $ pt-query-digest --user=root --password=123 --history h=localhost,D=test --no-report --create-history-table cisohm-slow.log
    - select * from query_historylimit 2 \G

9. 分析binlog
    - $ mysqlbinlog on.000010 > mysql-bin000010.sql
    - $ pt-query-digest --type=binlog mysql-bin000010.sql > slow_bin-10.log

## 分析binlog 结果解读

```bash
mysqlbinlog on.000010 > mysql-bin000010.sql
pt-query-digest --type=binlog mysql-bin000010.sql > slow_bin-10.log
```

### 第一部分：总体统计结果

- Overall: 总共有多少条查询，
- Time range: 查询执行的时间范围。
- unique: 唯一查询数量，即对查询条件进行参数化以后，总共有多少个不同的查询
- total: 总计min:最小max: 最大avg:平均
- 95%: 把所有值从小到大排列，位置位于95%的那个数，这个数一般最具有参考价值。
- median: 中位数，把所有值从小到大排列，位置位于中间那个数。

```sql
# 440ms user time, 160ms system time, 24.47M rss, 200.31M vsz
# Current date: Thu Nov  1 19:27:56 2018
# Hostname: lizhenzhen-qyw-1.novalocal
# Files: binlog.sql
# Overall: 29 total, 25 unique, 0.00 QPS, 0.03x concurrency ______________
# Time range: 2018-11-01 09:06:47 to 13:08:11
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time           398s       0     56s     14s     47s     19s       0
# Query size        15.77k       6   3.54k   97.28   84.10  468.15    5.75
# @@session.au           1       1       1       1       1       0       1
# @@session.au           1       1       1       1       1       0       1
# @@session.au           1       1       1       1       1       0       1
# @@session.ch          33      33      33      33      33       0      33
# @@session.ch           1       1       1       1       1       0       1
# @@session.co          33      33      33      33      33       0      33
# @@session.co          33      33      33      33      33       0      33
# @@session.fo           1       1       1       1       1       0       1
# @@session.lc           0       0       0       0       0       0       0
# @@session.ps           9       9       9       9       9       0       9
# @@session.sq           0       0       0       0       0       0       0
# @@session.sq       1.34G   1.34G   1.34G   1.34G   1.34G       0   1.34G
# @@session.un           1       1       1       1       1       0       1
# error code             0       0       0       0       0       0       0
```

### 第二部分：查询分组统计结果

- Response: 总的响应时间。
- time: 该查询在本次分析中总的时间占比。
- calls: 执行次数，即本次分析总共有多少条这种类型的查询语句。
- R/Call: 平均每次执行的响应时间。
- V/M: 响应时间Variance-to-mean的比率（方差对平均值比）
- Item : 查询对象

```sql
# Profile
# Rank Query ID                        Response time Calls R/Call  V/M   I
# ==== =============================== ============= ===== ======= ===== =
#    1 0xF4A1D958057AB044687C628130... 56.0000 14.1%     1 56.0000  0.00 ALTER TABLE edoc_todoworklist
#    2 0x1B2C849F04E4D343616A3AB3F0... 54.0000 13.6%     1 54.0000  0.00 ALTER TABLE edoc_todoworklist
#    3 0x2F46530C584334C2B531268F95... 48.0000 12.1%     1 48.0000  0.00 ALTER TABLE edoc_todoworklist
#    4 0xB4B8D8A74905E4F883EB31C8FA... 46.0000 11.6%     1 46.0000  0.00 ALTER TABLE edoc_todoworklist
#    5 0x7D5AB3CEB681E38FDFF9336B43... 43.0000 10.8%     1 43.0000  0.00 ALTER TABLE edoc_todoworklist
#    6 0xFC8BA0AE4C735D13976B53C3B7... 35.0000  8.8%     1 35.0000  0.00 ALTER TABLE edoc_todoworklist
#    7 0xC957219471F4616EC16250B954... 26.0000  6.5%     1 26.0000  0.00 ALTER TABLE edoc_todoworklist
#    8 0x774B4C62B72AC45394EC0D39A8... 24.0000  6.0%     1 24.0000  0.00 ALTER TABLE edoc_todoworklist
#    9 0x273FBF16C0325C9928128D5EF5... 23.0000  5.8%     1 23.0000  0.00 ALTER TABLE oa.edoc_todoworklist `oa`.`edoc_todoworklist`
#   10 0x16AB63D924713C65EFDB9593E8... 21.0000  5.3%     1 21.0000  0.00 ALTER TABLE edoc_todoworklist
#   11 0xD949F93FAA8A18CCC6CA26C63F... 20.0000  5.0%     1 20.0000  0.00 ALTER TABLE edoc_todoworklist
# MISC 0xMISC                           2.0000  0.5%    18  0.1111   0.0 <12 ITEMS>
```

### 第三部分：每一种查询的详细统计结果

由下面查询的详细统计结果，最上面的表格列出了执行次数、最大、最小、平均、95%等各项目的统计

- ID: 查询的ID号，和上图的Query ID对应
- Databases: 库名
- Users: 各个用户执行的次数（占比）
- Query_time distribution : 查询时间分布, 长短体现区间占比，本例中1s-10s之间查询数量是10s以上的两倍
- Tables: 查询中涉及到的表
- Explain: 示例SQL语句

```sql
# Query 1: 0 QPS, 0x concurrency, ID 0xF4A1D958057AB044687C628130861566 at byte 46819
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: all events occurred at 2018-11-01 11:20:05
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          3       1
# Exec time     14     56s     56s     56s     56s     56s       0     56s
# Query size     0      82      82      82      82      82       0      82
# error code     0       0       0       0       0       0       0       0
# String:
# Databases    oa
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms
#    1s
#  10s+  ################################################################
# Tables
#    SHOW TABLE STATUS FROM `oa` LIKE 'edoc_todoworklist'\G
#    SHOW CREATE TABLE `oa`.`edoc_todoworklist`\G
alter table edoc_todoworklist add key key_mix_2 (WAST_STATE,edoc_base_id,organ_id)\G
```
