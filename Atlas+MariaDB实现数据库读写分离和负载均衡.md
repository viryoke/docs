# Atlas+MariaDB实现数据库读写分离和负载均衡

## 一、搭建Mariadb主从复制数据库

### 1、使用Docker下载MariaDB镜像

```
docker pull mariadb
```

### 2、在本机上创建创建/opt/docker_mdb1目录和/opt/dcoker_mdb1/data目录，用于映射到docker容器中的数据目录

```
mkdir -p /opt/docker_mdb1
mkdir -p /opt/docker_mdb1/data
```

### 3、修改master主库my.cnf配置文件，并将文件置于/opt/docker_mdb1/目录下

```
server-id               = 1
log_bin                 = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size         = 1024M
```

### 4、根据MariaDB镜像，部署master主库

```
docker run --name master \
-p 13306:3306 \
-v /opt/docker_mdb1/data:/var/lib/mysql \
-v /opt/docker_mdb1/my.cnf:/etc/mysql/my.cnf \
-e MYSQL_ROOT_PASSWORD=root@123456 \
-e TERM=linux \
-d mariadb
```

### 5、同理配置slave1和slave2从库的本地目录和配置文件

```
# slave1
mkdir -p /opt/docker_mdb2
mkdir -p /opt/docker_mdb2/data
# my.cnf配置文件修改如下：
server-id               = 2
log_bin                 = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size         = 1024M

# slave2
mkdir -p /opt/docker_mdb3
mkdir -p /opt/docker_mdb3/data
# my.cnf配置文件修改如下：
server-id               = 3
log_bin                 = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size         = 1024M
```

### 6、根据MariaDB镜像，部署slave1和slave2从库

```
# slave1
docker run --name slave1 \
-p 23306:3306 \
-v /opt/docker_mdb2/data:/var/lib/mysql \
-v /opt/docker_mdb2/my.cnf:/etc/mysql/my.cnf \
-e MYSQL_ROOT_PASSWORD=root@123456 \
-e TERM=linux \
--link master:master_db \
-d mariadb

# slave2
docker run --name slave2 \
-p 33306:3306 \
-v /opt/docker_mdb3/data:/var/lib/mysql \
-v /opt/docker_mdb3/my.cnf:/etc/mysql/my.cnf \
-e MYSQL_ROOT_PASSWORD=root@123456 \
-e TERM=linux \
--link master:master_db \
-d mariadb
```

### 7、连接master主库，查看mysql-bin文件是否启用以及当前记录的位置

```
mysql -h127.0.0.1 -P13306 -uroot -p
# 进入数据库之后查询mysql-bin日志文件信息
show master status;
+------------------+----------+-----------------------+-----------------------+

| File             | Position | Binlog_Do_DB          | Binlog_Ignore_DB      |

+------------------+----------+-----------------------+-----------------------+

| mysql-bin.000001 |      328 |                       |                       |

+------------------+----------+-----------------------+-----------------------+
```

### 8、连接slave1、slave2从库，配置相关参数

```
# 连接数据库后执行以下命令
# 先停止slave
stop slave;

# 根据step7中查出的master信息更改相关项
change master to 
master_host='master_db',
master_user='root',
master_password='root@123456',
master_port=3306,
master_log_file='mysql-bin.000001',
master_log_pos=328;

# 重启slave
start slave;

# 注：为了测试方便，本例使用root作为主从复制账号，实际使用时可在主库上创建一个专用账号用于主从复制，如下：
grant replication slave on *.* to 'sync'@'%' identified by 'sync@123456';
```

### 9、重启slave1、slave2容器，至此一主两从的数据库就搭建完成了

```
docker restart slave1
docker restart slave2
```

## 二、安装并配置Atlas数据库中间件

### 1、根据系统版本，去官网下载对应的安装包

```
# 官网地址
https://github.com/Qihoo360/Atlas/releases
# 本例下载的版本为
Atlas-2.1-debian7.0-x86_64.deb
# 安装参考地址
https://github.com/Qihoo360/Atlas/blob/master/README_ZH.md
```

### 2、安装Atlas

```
dpkg -i Atlas-2.1-debian7.0-x86_64.deb
```

### 3、配置Atlas

```
# 进入配置文件目录
cd /usr/local/mysql-proxy/conf
# 修改test.cnf配置文件，根据注释的提示分别配置主库、从库地址以及数据库用户名密码即可，其他保持默认
proxy-backend-addresses = 127.0.0.1:13306
proxy-read-only-backend-addresses = 127.0.0.1:23306,127.0.0.1:33306
pwds = root:FAwHrcWPNrc6gKf9d0WkiA==
```

### 4、启动Atlas

```
/usr/local/mysql-proxy/bin/mysql-proxyd test start
```

### 5、根据Atlas开放的端口连接数据库，检测是否已经实现读写分离和负载均衡（测试方法多样，请自行验证）

```
mysql -h127.0.0.1 -P1234 -uroot -p
```

