---
title: "Mysql配置主从同步"
date: 2017-11-01T10:07:29+08:00
draft: false
---


参考
[Mysql官方文档](https://dev.mysql.com/doc/refman/5.7/en/replication.html)

##主机配置:

**修改My.conf**

```server-id              = 1```
主机Id
```log_bin           		= mast-binlog```
binlog存放位置
```log-bin-index           = master-binlog.index```
存放binlog索引
```expire_logs_days       = 10```
binlog过期时间
```max_binlog_size        = 200M```
binlog文件大小
```binlog_do_db           = database```
binlog只对那些库启用

确保skip-networking主服务器上未启用该选项。如果网络被禁用，则从机不能与主机通信，并且复制失败。
确保bind-address主机上没有启用改选项或者修改成0.0.0.0

log_bin和log-bin-index可以自定义存储目录。修改默认存储路径时需要修改mysql的访问权限。

```
vim /etc/apparmor.d/usr.sbin.mysqld
```
配置好My.conf和apparmor后需要重启配置才能生效

在主机上创建一个用户给从机连接

```
CREATE USER 'slave_mysql'@'%' IDENTIFIED BY 'mmp123';
GRANT REPLICATION SLAVE ON *.* TO 'slave_mysql'@'%';
```

创建完用户有刷新权限
```flush privileges;```

锁住数据库用mysqldump将这个数据库备份发给从机，配置好从机再解锁

```
FLUSH TABLES WITH READ LOCK;
UNLOCK TABLES;
```

查看主机当前在用哪个binlog文件写入和写入位置

``SHOW MASTER STATUS;``

```
File                   postion   Binglog-Do-DB
master-bin.00001    	004       database
```
		
从机配置

**修改My.conf**
```

```server-id = 2```
从机ID不能和主机一样
```replicate-do-db=database```
需要备份那些库
```relay_log    = server-relay-bin```
中继日志存放位置。从机从主机获取数据后先会放入中继日志，然后从中继中读取sql语句。

配置好my.conf后重启从机，然后在从机配置主机信息

```

[CHANGE MASTER TO 完整语法介绍](https://dev.mysql.com/doc/refman/5.7/en/change-master-to.html)

从机配置
https://mysql.az/2015/04/12/change-binary-log-and-relay-log-location-to-home/

查看Binlog
binlog无法查看解决办法
mysqlbinlog: [ERROR] unknown variable 'default-character-set=utf8'
mysqlbinlog --no-defaults --set-charset=utf8 master-bin.080541
查看Row模式下的binlog
mysqlbinlog  --no-defaults --set-charset=utf8 -v --base64-output=DECODE-ROWS master-bin.080541

https://dev.mysql.com/doc/refman/5.7/en/mysqlbinlog-row-events.html

创建过滤规则
CHANGE REPLICATION FILTER filter[, filter][, ...]

filter:
    REPLICATE_DO_DB = (db_list)
  | REPLICATE_IGNORE_DB = (db_list)
  | REPLICATE_DO_TABLE = (tbl_list)
  | REPLICATE_IGNORE_TABLE = (tbl_list)
  | REPLICATE_WILD_DO_TABLE = (wild_tbl_list)
  | REPLICATE_WILD_IGNORE_TABLE = (wild_tbl_list)
  | REPLICATE_REWRITE_DB = (db_pair_list)
  
  CHANGE REPLICATION FILTER
    REPLICATE_IGNORE_TABLE = ()

db_list:
    db_name[, db_name][, ...]

tbl_list:
    db_name.table_name[, db_table_name][, ...]
wild_tbl_list:
    'db_pattern.table_pattern'[, 'db_pattern.table_pattern'][, ...]

db_pair_list:
    (db_pair)[, (db_pair)][, ...]

db_pair:
    from_db, to_db
    
    CHANGE REPLICATION FILTER

   REPLICATE_WILD_IGNORE_TABLE = ('db1.new%', 'db2.new%');
   
   日志清理
   PURGE MASTER LOGS BEFORE DATE_SUB(CURRENT_DATE, INTERVAL 2 DAY)
http://www.ttlsa.com/mysql/mysql-5-7-enhanced-multi-thread-salve/
从机开启多线程
STOP SLAVE;
SET GLOBAL master_info_repository = 'TABLE' 
SET GLOBAL relay_log_info_repository = 'TABLE'
SET GLOBAL relay_log_recovery = 
SET GLOBAL slave_parallel_type=LOGICAL_CLOCK;
\
START slave




https://www.percona.com/blog/2013/02/08/how-to-createrestore-a-slave-using-gtid-replication-in-mysql-5-6/

同步出错解决方案
The slave is connecting using CHANGE MASTER TO MASTER_AUTO_POSITION = 1, but the master has purged binary logs containing GTIDs that the slave requires.

这个是由于在主库上执行过purge binary logs，然后当从库change master的时候，却要执行那些事务。
你可以在主库上先查找哪些gtid被purge了。
show global variables like 'gtid_purged';
然后拿着这个value，去从库上依次
stop slave;
set global gtid_purged = 'xxx'; # xxx是你主库上查到的value。
start slave;
这样能跳过执行被主库已经purge的事务了。
如果出现
@@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_EXECUTED is empty.
reset master后在执行

