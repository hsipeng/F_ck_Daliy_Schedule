Docker 是一个开源的引擎，可以轻松的为任何应用创建一个轻量级、可移植、自给自足的容器。

docker 的使用就不多说，直接进入怎么配置mysql 主从复制

## 实现步骤

- 准备两台 MySQL 服务器

- 配置主服务器（Master）

- 配置从服务器（Slave）

- 完成Master和Slave链接

- 测试配置是否成功

## docker 虚拟两台 MySQL 服务器

命令如下：
创建主服务器

```bash
docker run --name mysql_master -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

```

> 通过镜像 mysql:latest ,该景象基于debain os 。启动一个名为 mysql_master 的 MySQL 服务器，端口号是3306，映射的宿主机端口号是3306，root 账号密码是123456

创建从服务器

```bash
docker run --name mysql_slave -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
```

可以使用 docker ps 查看当前运行的容器

下面需要进行一些相关配置

首先我们先逐个获取容器运行的ip地址

```bash
# master
docker inspect --format='{{.NetworkSettings.IPAddress}}' c66b935ea94d

172.17.0.2

# slave

docker inspect --format='{{.NetworkSettings.IPAddress}}' 2bb11aa899e4

172.17.0.3

```

## 配置主服务器（Master）
```bash

docker exec -it c66b935ea94d /bin/bash

```
在安装vim之前要先升级一下软件包

```bash
apt-get update
apt install -y vim

```

之后配置mysql，配置文件在/etc/mysql/my.cnf

```bash
[mysqld]
## 设置server_id，一般设置为IP，同一局域网内注意要唯一
server_id=100
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=edu-mysql-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062

```
在我的容器中，my.cnf 的路径是 /etc/mysql。
配置完成后重启 mysql ，使用如下命令：


```bash
service mysql restart

```

这个命令会使得容器停止，重新启动就可以了。

```bash
docker start c66b935ea94d
```


接下来创建数据同步用户：

```basn
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
flush privileges;
```

## 配置从服务器（Slave）

配置文件 my.cnf

```bash
[mysqld]
## 设置server_id，一般设置为IP,注意要唯一
server_id=101
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=edu-mysql-slave1-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## 防止改变数据(除了特殊的线程)
read_only=1
```


## 完成Master和Slave链接

注意，需要保证 Master 和 Slave 除了不同步的数据库，其他数据库的数据要一致。
在 Master 进入 MySQL， 然后执行命令：

```bash
show master status;
```

记录下 File 和 Position 字段的值，后面会用到。

然后到 Slave 中进入 mysql，执行命令：
```bash
change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='edu-mysql-bin.000001', master_log_pos=929, master_connect_retry=30;
```


> 命令解释：
master_host: Master 的IP地址
master_user: 在 Master 中授权的用于数据同步的用户
master_password: 同步数据的用户的密码
master_port: Master 的数据库的端口号
master_log_file: 指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
master_log_pos: 从哪个 Position 开始读，即上文中提到的 Position 字段的值
master_connect_retry: 当重新建立主从连接时，如果连接失败，重试的时间间隔，单位是秒，默认是60秒。



在 Slave 的 MySQL 终端执行查看主从同步状态

```bash
show slave status \G;

```

SlaveIORunning 和 SlaveSQLRunning 是No，表明 Slave 还没有开始复制过程。相反 SlaveIORunning 和 SlaveSQLRunning 是Yes表明已经开始工作了，因为我已经运行过了，所以我的显示的都是 Yes。

执行以下命令，开始开启主从同步：
```bash
start slave;
```

OK!

> slave 基本命令
start slave; //开启slave
stop slave;  // 停止主从复制
reset slave; // 充值slave 状态。
或者 change master to master_host=' '

