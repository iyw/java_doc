## mysql.57 多源复制搭建

#### 机器列表
```
master1: 172.17.0.4
master2: 172.17.0.3

汇总库: slave: 172.17.0.2
```

#### master 配置(多个master注意server-id不能一样)
```
[mysql]
default-character-set=utf8
[mysqld]
server-id=1001
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-bin=mysql-bin
binlog-format=ROW
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
sync_binlog =1
log-slave-updates
```

#### slave 配置
```
conf.d/
├── ignore.cnf
├── mysqld.cnf
└── sq_order.cnf
```
配置1:(mysqld.cnf)
```
[mysql]
default-character-set=utf8

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-bin=mysql-bin
binlog-format=ROW
server_id=1000
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
log-slave-updates
sync_binlog =1
master_info_repository=TABLE 
relay_log_info_repository=TABLE
```

配置2:(ignore.cnf 配置需要过滤的表)
```
[mysqld]
replicate-wild-ignore-table = mysql.%
replicate-wild-ignore-table = test.%
replicate-wild-ignore-table = information_schema.%
replicate-wild-ignore-table = performance_schema.%
replicate-wild-ignore-table = sq_order_global.%
replicate-wild-ignore-table = %.driver_order
```

配置3:(sq_order.cnf多源复制配置)
```
[mysqld]
replicate-rewrite-db=sq_order_0->sq_order
replicate-rewrite-db=sq_order_1->sq_order
replicate-rewrite-db=sq_order_2->sq_order
replicate-rewrite-db=sq_order_3->sq_order
.....

replicate-rewrite-db=sq_order_511->sq_order
```
为了方便、写了个脚本
```sh
#!/bin/bash
  echo "[mysqld]" >> sq_order.cnf;
  for((i=0; $i<512; i++)); do
    echo "replicate-rewrite-db=sq_order_${i}->sq_order" >> sq_order.cnf;
  done
```

#### 授权

主库(172.17.0.3,172.17.0.4)上分别执行
```
grant replication slave,replication client on *.* to 'slave'@'172.17.0.2' identified by '123456';
flush privileges
show master status；
```

从库上执行
```
stop slave;
reset slave;
change master to master_host='172.17.0.4',master_port=3306,master_user='slave',master_password='123456',master_log_file='mysql-bin.000002',master_log_pos=2816661 for channel'm1';
change master to master_host='172.17.0.3',master_port=3306,master_user='slave',master_password='123456',master_log_file='mysql-bin.000003',master_log_pos=2829761 for channel 'm2';
start slave;
```

大功告成!!!
