# MariaDB

说明： 操作系统是centos7.2, 采用MariaDB10.1, 主备模式采用GTID（2个节点）。可参考： http://www.tuicool.com/articles/raaMfeV

### 安装部分

- repo源的准备

```
CentOS7.2自带repo中MariaDB5.5, 10.1需要单独下载，自行配置好repo
```

- 安装软件

```
yum install MariaDB-server
```

### 配置部分

#### 修改配置文件

*要修改数据目录的话，最好做软链，例如：ln -sv /app/mysql_data/mysql/ /var/lib/mysql 也可以配置datadir*

- 主节点配置

```
在/etc/my.cnf.d/server.cnf配置文件[server]段添加一下内容：
[mysqld]
socket      = /var/lib/mysql/mysql.sock
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 256
sort_buffer_size = 1M
read_buffer_size = 1M
read_rnd_buffer_size = 4M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size= 16M
thread_concurrency = 4
log-bin=mysql-bin
relay-log=relay-log-bin
server-id=1
binlog-format=ROW
log-slave-updates=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-threads=2
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
report-port=3306
report-host=10.214.129.156
character-set-server=utf8
collation-server=utf8_general_ci
```

- 另一台备份节点做如下修改


```
[mysqld]
socket      = /var/lib/mysql/mysql.sock
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 256
sort_buffer_size = 1M
read_buffer_size = 1M
read_rnd_buffer_size = 4M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size= 16M
thread_concurrency = 4
log-bin=mysql-bin
relay-log=relay-log-bin
server-id=2
binlog-format=ROW
log-slave-updates=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-threads=2
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
report-port=3306
report-host=10.214.129.183
character-set-server=utf8
collation-server=utf8_general_ci
```


### 修改客户端登录socket
```
[client]
socket=/var/lib/mysql/mysql.sock
```


#### 启动mysql

```
systemctl start mariadb.service
systemctl enable mariadb.service
```

#### 配置主从

- 主节点：配置同步账号：*GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'backup'@'从节点' IDENTIFIED BY 'admin';* 例如：

```
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'backup'@'10.214.129.183' IDENTIFIED BY 'admin';
```

- 主节点：记录主节点二进制日志文件和位置，*show master status;*，例如：

```
MariaDB [(none)]> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000006 |      788 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

- 备节点：第一次同步,确保 **Slave_IO_Running=Yes Slave_SQL_Running=Yes**,使用*show slave status\G*;查看

```
CHANGE MASTER TO master_host='10.214.129.156',master_user='backup',master_password='admin',MASTER_LOG_FILE='mysql-bin.000006', MASTER_LOG_POS=378;
```

- 备节点: 使用GTID同步

```
stop slave;
#slave_pos模式可以在主节点挂掉后，slave可以成为master，master可以变为slave
change master to master_host='10.214.129.156',master_user='backup',master_password='admin',master_use_gtid=slave_pos;

start slave;
```

### 测试部分

- 在主节点进行增删改查，查看从节点是否同步

示例如下：

```
1. 在主节点创建数据库和表，并插入数据
create database guo_test;
use guo_test;
create table test(id int,name varchar(20),password varchar(20));
insert into test values(1,'zhangsan','123');
insert into test values(2,'lisi','123');
2. 在主节点和从节点查看是否同步
select * from test;
3. 修改表中的一条数据，在查看是否同步
update test set name='admin' where id=2;
```

### 故障恢复部分

- 当主节点出现故障，假设主节点down,此时备节点可以继续进行读写操作，但是当主节点恢复起来的时候，不通同步从节点数据，现解决如下问题：

```
备节点：
GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'backup'@'10.214.129.156' IDENTIFIED BY 'admin';

主节点：
change master to master_host='10.214.129.183',master_user='backup',master_password='admin',master_use_gtid=current_pos;
start slave;
```

- 做完以上操作后，发现主节点故障的这段时间中备节点写入的数据会同步到主节点中。


