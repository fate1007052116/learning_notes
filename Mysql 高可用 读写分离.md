# Mysql 高可用，读写分离



## 一、配置mysql主从机器

### 1、mysql主从复制原理

* 主机只能有一台，而从机可以有多台
* 当主从关系搭建完成之后，从机复制主机的所有操作，但是对于主从关系确立之前的数据，从机全部不复制
* 流程
1. 主机将所有的写操作写入到一个binary log 文件中

2. 从机不能直接访问主机

3. 从机去读取主机的binary log日志，并写入到从机本机的中继日志 relay log

4. 从机再执行relay log日志，这样从机就能复制主机的所有记录

   ![Snip20200726_1](/Users/luo/Documents/开发笔记/images/Snip20200726_1.png)

#### （1）主机配置


```shell
# 这样拷贝，容器无法解析my.cnf文件，直接使用docker磁盘映射以解决

# 从docker中拷贝my.cnf到物理机，发现文件打不开
luo@luodeMacBook-Pro ~ % docker cp mysql_1:/etc/mysql/my.cnf /Volumes/OS/

# 换一下位置，再
root@d61093c2f81d:/# cp /etc/mysql/my.cnf /opt/

# 这样就可以编辑 物理机上的/Volumes/OS/my.cnf了
luo@luodeMacBook-Pro ~ % docker cp mysql_1:/opt/my.cnf /Volumes/OS/

# 再把编辑完的my.cnf拷贝回容器
luo@luodeMacBook-Pro ~ % docker cp /Volumes/OS/my.cnf mysql_1:/opt/

# 放回原位置
root@d61093c2f81d:/# cp /opt/my.cnf /etc/mysql/
```



修改my.cnf文件，在[mysqld]加入下面的内容：

```shell
[mysqld]
# 主服务器唯一ID
server_id=1  # 在mysql 5.7 注意是 下划线
# 启用二进制日志
# 会生成相应的文件mysql-master-bin.000001
log-bin=mysql-master-bin
# 设置不要复制的数据库（可以复制多个）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
# 设置需要复制的数据库
# 主机不需要在搭建主从环境之前，就将要复制的数据库建好，因为建好之后，从机不能复制创建数据库的binary log（此时主从关系还未创建），但是主从关系搭建之后，从机可以复制主机的binary log，但是却没有对应的数据库用来执行该binary log，然后报错
binlog-do-db=home #需要复制的主数据库的名字
# 设置logbin格式
binlog_format=STATEMENT
# binlog_format有三种格式
# STATEMENT : 将所有写操作写入到binary log日志中；缺点会造成数据不一致 insert into user(id,date) values(0,now())，比如说主机调用now()函数
# ROW : 行模式，只记录每一行的改变；但如果是全表更新，行模式就要记录每一行，从机要执行每一行，效率低
#MIXED : 混合模式，解决了一部分数据不一致的问题；缺点：不能识别系统变量 @@host name
```

启动主机时，查看日志发现my.cnf未生效

```shell
# 查看日志
luo@luodeMacBook-Pro ~ % docker logs mysql_1
# 意思是这个文件所有人可写，不安全，所以这个文件里面的配置信息被忽略
mysqld: [Warning] World-writable config file '/etc/mysql/my.cnf' is ignored.

# 改变权限即可解决
# 无论在物理机上执行还是在容器中执行，都没有效果，无法改变权限，这个应该只有在mac上才会出现这个问题
luo@luodeMacBook-Pro ~ % chmod 664 /Volumes/extend/temp/mysql_1/conf/my.cnf

# 无法改变权限，且容器没有vi编辑器，可以将本地盘映射到容器的其他位置，再在容器中将物理机配置好的文件拷贝到/etc/mysql/my.cnf,并修改相应权限
luo@luodeMacBook-Pro conf % docker create -p 3301:3306 --name mysql_1 -v /Volumes/extend/temp/mysql_1/conf:/opt -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

# 将物理机映射到容器的my.cnf 配置文件拷贝到容器的 /etc/mysql/目录下
# 需要提前删除 /etc/mysql/* 的文件
root@fbde82fb9a02:/opt# cp -r /opt/* /etc/mysql/

root@fbde82fb9a02:/# chmod 644 /etc/mysql/*
```



#### （2）从机my.cnf配置

```shell
[mysqld]
# 从机服务器唯一ID
server_id=2
# 启用中继日志
relay-log=mysql-relay
# 开启只读模式
read-only=1
```

#### （3）主机创建账号给从机复制binary log

```shell
# 只能在主机上直接登陆进行授权，不可以通过 mycat 登陆再授权
root@b6f68efb9d75:/# mysql -uroot -p123456

# 'slave'@'%' 从机账号为slave，可以在任何ip登陆
# slave 的密码为123456
mysql> grant replication slave on *.* to 'slave'@'%' identified by '123123';
Query OK, 0 rows affected, 1 warning (0.00 sec)

# 这是没有配置成功，检查my.cnf 的权限是不是chmod 644 my.cnf 
# my.cnf 中的自定义配置 是否在 [mysqld] 后面
mysql> show master status; 
Empty set (0.00 sec)

# 配置成功的情况
mysql> show master status;
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB                                | Executed_Gtid_Set |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| mysql-master-bin.000001 |      438 | home         | mysql,sys,information_schema,performance_schema |                   |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
1 row in set (0.00 sec)

# 记录下 File mysql-master-bin.000001
# 记录 	 Position 438

# 执行完这个步骤之后不要再操作主机了，不然数据不一致
```

#### （4）配置从机，从主机同步数据

```shell
change master to
master_host='172.17.0.2',
master_user='slave',
master_password='123123',
master_log_file='mysql-master-bin.000001',
master_log_pos=438;
```

```shell
# 执行效果
# 停止从主机拷贝数据
mysql> stop slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

# 重新皮质主从关系
mysql> reset master;
Query OK, 0 rows affected (0.00 sec)

# 从主机的特定日志文件的特定位置开始复制数据
# 可以多次执行，相当于一个还原点
mysql> change master to
    -> master_host='172.17.0.2',
    -> master_user='slave',
    -> master_password='123123',
    -> master_log_file='mysql-master-bin.000001',
    -> master_log_pos=438;
Query OK, 0 rows affected, 1 warning (0.01 sec)

# 启动从机服务
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

# 查看从机状态
mysql> show slave status;
# 数据太多，按照列的方式显示
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-master-bin.000001
          Read_Master_Log_Pos: 438
               Relay_Log_File: mysql-relay.000002
                Relay_Log_Pos: 327
        Relay_Master_Log_File: mysql-master-bin.000001
             Slave_IO_Running: Yes	# 这两项都为yes，则主从复制已经搭建好 
            Slave_SQL_Running: Yes	# 
```

#### （5）注意

* 从机也可以写入数据，但是从机的数据不会同步到主机

* 往从机写入数据，如果之后从机再从主机同步数据的时候有冲突，则同步会终止


#### （6）通过mycat开启读写分离

* 测试是否已经开启读写分离

  ```sql
  -- 在mycat上插入数据
  insert into testTable 
  values(4,@@hostname)
  
  
  select * from testTable
  where id = 4
  ```
  
  主机中的数据
  ![Snip20200727_1](/Users/luo/Documents/开发笔记/images/Snip20200727_1.png)

  从机中的数据
  

![Snip20200727_2](/Users/luo/Documents/开发笔记/images/Snip20200727_2.png)

​		此时mycat中读出的数据

![Snip20200727_3](/Users/luo/Documents/开发笔记/images/Snip20200727_3.png)

​		发现此时，mycat的读和写都是在主机上完成，原因是没有配置

* 修改**Mycat**的配置文件**schema.xml**,修改<dataHost>的balance属性，通过此属性配置读写分离的类型 

  负载均衡类型，目前的取值有4 种： 

  （1）balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上。 

  （2）balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模		 式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡。 

  ​		 只能有一台写主机，stand by writeHost 是写主机的备份，当写主机down时，stand by writeHost 立刻接替写主机的工作

  

  （3）balance="2"，所有**读**操作都随机的在 writeHost、readhost 上分发。 （测试用）

  （4）balance="3"，所有读请求随机的分发到 readhost 执行，writerHost 不负担读压力 



#### （7）注意`my.cnf`中的`Server_id=1`

```sql
show slave status;

Fatal error: The slave I/O thread stops because master and slave have equal MySQL server ids;
these ids must be different for replication to work (or the --replicate-same-server-id option must be used on slave but this does not always make sense; please check the manual before using it).

```



### 2、允许root账户远程访问

```shell
mysql> grant all PRIVILEGES on *.* to root@'%' identified by '1234' with grand option;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>
```



### 3、双主双从

### （1）双主机配置

如果是在`docker`环境中搭建，建议，先启动`mysql`容器之后，再来修改`mysql.cnf`，否则`mysql root`账户只能在容器里面登录

可以添加所有`mysql`配置文件的公共部分

```shell
[client]
# 注意不是 utf-8
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
# 注意 skip-name-resolve 一定要加，不然连接mysql会超级慢
skip-name-resolve
```



**配置完my.cnf之后，记得要重启呀**

> 注意： `[mysqld]`只能有一个，需要手动合并

Master 1 配置

```shell
[mysqld]
# 主服务器唯一ID
server-id=1
# 是否为只读模式：否
read-only=0
# 启用二进制日志
# 会生成相应的文件mysql-master-bin.000001
log-bin=mysql-master-bin
# 设置不要复制的数据库（可以复制多个）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
# 设置需要复制的数据库
# 主机不需要在搭建主从环境之前，就将要复制的数据库建好，因为建好之后，从机不能复制创建数据库的binary log（此时主从关系还未创建），但是主从关系搭建之后，从机可以复制主机的binary log，但是却没有对应的数据库用来执行该binary log，然后报错
binlog-do-db=home #需要复制的主数据库的名字
# 设置logbin格式
binlog_format=STATEMENT

# 额外配置
# 主机在作为从数据库的时候，如果有写操作，也要更新二进制日志文件
log-slave-updates
# 表示自增长字段每次递增的量，指自增字段的起始值，其默认值时1，取值范围1-65535
auto-increment-increment=2
# 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是 1-65535
# 这个值 master1 master2 要区分开来
# server-id 也需要区分
auto_increment-offset=1
```

Master 2 配置

```shell
[mysqld]
# 主服务器唯一ID
server-id=3
# 启用二进制日志
# 会生成相应的文件mysql-master-bin.000001
log-bin=mysql-master-bin
# 设置不要复制的数据库（可以复制多个）
binlog-ignore-db=mysql
binlog-ignore-db=sys
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
# 设置需要复制的数据库
# 主机不需要在搭建主从环境之前，就将要复制的数据库建好，因为建好之后，从机不能复制创建数据库的binary log（此时主从关系还未创建），但是主从关系搭建之后，从机可以复制主机的binary log，但是却没有对应的数据库用来执行该binary log，然后报错
binlog-do-db=home #需要复制的主数据库的名字
# 设置logbin格式
binlog_format=STATEMENT

# 额外配置
# 主机在作为从数据库的时候，如果有写操作，也要更新二进制日志文件
log-slave-updates
# 表示自增长字段每次递增的量，指自增字段的起始值，其默认值时1，取值范围1-65535
auto-increment-increment=2
# 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是 1-65535
# 这个值 master1 master2 要区分开来
# server-id 也需要区分
auto_increment-offset=2
```

### （2）双从机配置

slave 1 配置

```shell
[mysqld]
# 从机服务器唯一ID
server-id=2
# 启用中继日志
relay-log=mysql-relay
```

Slave 2 配置

```shell
[mysqld]
# 从机服务器唯一ID
server-id=4
# 启用中继日志
relay-log=mysql-relay
```



### （3）创建备用主从

```shell
# 备用 master2
luo@luodeMacBook-Pro ~ % docker create --name mysql_3 -p 3303:3306 -v /Volumes/extend/temp/mysql_1/conf:/opt/ -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

# 备用 slave2

luo@luodeMacBook-Pro ~ % docker create --name mysql_4 -p 3304:3306 -v /Volumes/extend/temp/mysql_2/conf:/opt/ -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

### （4）授权

在两台主机上建立账号并授权slave

```sql
-- 因为master1的授权sql前面有，所以这里只显示 master2 授权 slave2 的sql
-- 多个授权sql都可以一样，但是不安全
grant replication slave on *.* to 'slave2'@'%' identified by '123123';

-- 刷新权限
FLUSH PRIVILEGES;
```

### （5）重制master 1 

```shell
mysql> reset master;
Query OK, 0 rows affected (0.01 sec)

mysql> show master status;
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB                                | Executed_Gtid_Set |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| mysql-master-bin.000001 |      154 | home         | mysql,sys,information_schema,performance_schema |                   |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
1 row in set (0.00 sec)
```

### （6）重制 master 2

```shell
mysql> reset master;
Query OK, 0 rows affected (0.01 sec)

mysql> show master status;
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| File                    | Position | Binlog_Do_DB | Binlog_Ignore_DB                                | Executed_Gtid_Set |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
| mysql-master-bin.000001 |      154 | home         | mysql,sys,information_schema,performance_schema |                   |
+-------------------------+----------+--------------+-------------------------------------------------+-------------------+
1 row in set (0.00 sec)
```



### （7）slave 1 连接 master 1

Slave 1 复制 master1

```shell
## 停止slave，停止之前的主从复制
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

# 重制slave
mysql> reset slave;
Query OK, 0 rows affected (0.01 sec)

change master to
master_host='172.17.0.2',
master_user='slave',
master_password='123123',
master_log_file='mysql-master-bin.000001',
master_port=3001,
master_log_pos=154;  
# 和 master 1 的位置一致

# 启动 slave
mysql> start slave;

# 查看slave状态
mysql> show  slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-master-bin.000001
          Read_Master_Log_Pos: 154
               Relay_Log_File: mysql-relay.000002
                Relay_Log_Pos: 327
        Relay_Master_Log_File: mysql-master-bin.000001
             Slave_IO_Running: Yes # 主从配置成功
            Slave_SQL_Running: Yes
```



### （8）slave 2 连接 master 2

slave 2 复制 master 2

```shell
## 停止slave
mysql> stop slave;
Query OK, 0 rows affected (0.00 sec)

# 重制slave
mysql> reset slave;
Query OK, 0 rows affected (0.01 sec)

# 这里我改过账号为 slave2
# ip 也需要改，使用 docker inspect mysql_3 查看ip
change master to
master_host='172.17.0.5',
master_user='slave2',
master_password='123123',
master_log_file='mysql-master-bin.000001',
master_log_pos=154;  
# 这里需要和 master 2的位置一致

# 记得开启
mysql> start slave;

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.5
                  Master_User: slave2
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-master-bin.000001
          Read_Master_Log_Pos: 154
               Relay_Log_File: mysql-relay.000002
                Relay_Log_Pos: 327
        Relay_Master_Log_File: mysql-master-bin.000001
             Slave_IO_Running: Yes	# 备用从机 成功配置
            Slave_SQL_Running: Yes
```

### （9）两个主机相互复制

master 2 复制 master 1 

```shell
# 在master 2 上执行与 slave1连接master1 相同的命令
change master to
master_host='172.17.0.2',
master_user='slave',
master_password='123123',
master_log_file='mysql-master-bin.000001',
master_log_pos=154;  

# 记得开启
mysql> start slave;

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-master-bin.000001
          Read_Master_Log_Pos: 154
               Relay_Log_File: bbbc2d270540-relay-bin.000002
                Relay_Log_Pos: 327
        Relay_Master_Log_File: mysql-master-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

master 1 复制 master 2

```shell
# 在master 1 上执行与 slave 2 连接master 2 相同的命令
change master to
master_host='172.17.0.5',
master_user='slave2',
master_password='123123',
master_log_file='mysql-master-bin.000001',
master_log_pos=154;  

mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.5
                  Master_User: slave2
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-master-bin.000001
          Read_Master_Log_Pos: 154
               Relay_Log_File: 04566d88c2ac-relay-bin.000002
                Relay_Log_Pos: 327
        Relay_Master_Log_File: mysql-master-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Ye
```

### （10）注意

1. master 1 中不能有数据库，否则删掉数据库，重新来一遍

2. mysql 双主双从，配置了之后，slave1只能同步master1的数据，slave1不能同步master2的数据，slave2 可以同步master1 和 master2的数据。master1 和 master 2 之间可以互相同步，就只是slave1  不可以同步 master 2；重启master 1 解决问题

   **原因出在master1，因为改变slave1作为master2 的从机，发现slave2没有问题，master配置了my.cnf 的 log-slave-updates 及其以后的内容，没有重启使得my.cnf生效，所以master1 接收到来自master 2的binary log文件之后，没有进行持久化，自然slave1 也收不到经过master 1转发的master2的日志文件，所以slave1不能同步master2 的写操作**

### （11）配置mycat

1. balance="1": 全部的readHost与stand by writeHost参与select语句的负载均衡。 
2. writeType="0": 所有写操作发送到配置的第一个writeHost，第一个master 1 挂了切到还生存的第二个  master 2
3. writeType="1"，所有写操作都随机的发送到配置的writeHost，1.5 以后废弃不推荐 
4. writeHost，重新启动后以切换后的为准，切换记录在配置文件中:dnindex.properties 。 
5. switchType="1": 1 默认值，自动切换。 
6. -1 表示不自动切换 
7. 2 基于MySQL 主从同步的状态决定是否切换。

```xml
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<!-- 心跳检测，空闲时发送sql给数据库，检查数据库是否down -->
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts 配置写主机 -->
		<writeHost host="hostM1" url="172.17.0.2:3306" user="root" password="123456">
			<!-- can have multi read hosts 配置读主机-->
			<readHost host="hostS1" url="172.17.0.3:3306" user="root" password="123456" />
		</writeHost>
		<!-- 配置双主双从 -->
		<writeHost host="hostM2" url="172.17.0.5:3306" user="root" password="123456">
			<!-- can have multi read hosts 配置读主机-->
			<readHost host="hostS2" url="172.17.0.6:3306" user="root" password="123456" />
		</writeHost>
	</dataHost>
```

### （12）高可用细节

* 当master 1 挂掉之后，master 2作为写主机，slave 2 作为读主机，slave 1 也挂了
* master 1 再次上电之后，master 1 就沦为备用master 了，写操作在master 2 上进行



## 二、mycat

### 1、修改配置文件

* Server.xml

  ```xml
  	<!-- 设置登陆mycat的账户密码 -->
  	<user name="mycatRoot">
  		<property name="password">123456</property>
  		<!-- 整个 mycat 对java暴露的 逻辑库 -->
  		<property name="schemas">TESTDB</property>
  	</user>
  ```

* schema.xml

  ```xml
  <!-- schema 标签中的内容为示例数据库连接，全部删除 -->
  ```

* 配置mysql

  ```shell
  # 主mysql
  luo@luodeMacBook-Pro ~ % docker create -p 3301:3306 --name mysql_1 -v /Volumes/extend/temp/mysql_1/conf:/etc/mysql/ -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
  
  # 从mysql
  luo@luodeMacBook-Pro ~ % docker create -p 3302:3306 --name mysql_2 -v /Volumes/extend/temp/mysql_2/conf:/etc/mysql/ -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
  
  # 实际执行的创建容器
  luo@luodeMacBook-Pro ~ % docker create -p 3301:3306 --name mysql_1 -v /Volumes/extend/temp/mysql_1/conf:/opt/ -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
  
  luo@luodeMacBook-Pro ~ % docker create -p 3302:3306 --name mysql_2 -v /Volumes/extend/temp/mysql_2/conf:/opt/ -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
  
  ```

* 启动mycat

  ```shell
  # 前台启动
  /usr/local/mycat/bin/mycat console
  
  # 后台启动
  /usr/local/mycat/bin/mycat start
  ```

* 使用mysql客户端工具通过mycat连接mysql

```shell
  # 只适用于mysql:5.7 
  # mysql:8.0.21 老是提示密码错误 
  # -umycat 是登陆mycat，mycat的账户
  # -p123456 是登陆 mycat的密码
  # -h 172.17.0.4 是mycat的ip地址
  # -P 8066 是mycat对外提供数据库访问的端口
  # -DTESTDB 是 server.xml中设置的逻辑数据库的名字
  # --default_auth=mysql_native_pasowrd Mysql 8的缺省加密方式已经改为caching_sha2_password，而MyCat对此尚不支持。为此，需加上--default_auth=mysql_native_pasowrd选项，但是看不出效果

  root@bae3ade0efdf:/# mysql -umycat -p123456 -h 172.17.0.4 -P 8066 -DTESTDB --default_auth=mysql_native_pasowrd

  # 发现mycat对外只有一个server.xml中的逻辑数据库 TESTDB
  mysql> show databases;
  +----------+
  | DATABASE |
  +----------+
  | TESTDB   |
  +----------+
  1 row in set (0.00 sec)
```



### 2、分库

Localhost 1 , localhost 2 两个主机上必须在启动mycat之前就都已经创建好home数据库，且名字一样

```xml
<!-- dataNode="dn1" 设置 server.xml中的 TESTDB 默认节点是dn1 -->
	<schema name="home" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
	<!-- 如果SQL中操作的是customer表，则会走db2节点，若不是customer表，走的是dn1节点 -->
		<table name="customer" dataNode="dn2"></table>
	</schema>
	<!-- database 需要和 数据库中的库名相同 -->
	<dataNode name="dn1" dataHost="localhost1" database="home" />
	<!-- 即使使用了分库，数据库的名字也必须相同，也就意味着这两个库在启动mycat之前就需要被创建好，
		不可以通过mycat来创建home数据库
	-->
	<dataNode name="dn2" dataHost="localhost2" database="home" />

	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<!-- 心跳检测，空闲时发送sql给数据库，检查数据库是否down -->
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts 配置写主机 -->
		<writeHost host="hostM1" url="172.17.0.2:3306" user="root" password="123456">
			<!-- can have multi read hosts 配置读主机-->
			<readHost host="hostS1" url="172.17.0.3:3306" user="root" password="123456" />
		</writeHost>
		<!-- 配置双主双从 -->
		<writeHost host="hostM2" url="172.17.0.5:3306" user="root" password="123456">
			<!-- can have multi read hosts 配置读主机-->
			<readHost host="hostS2" url="172.17.0.6:3306" user="root" password="123456" />
		</writeHost>
	</dataHost>


	<dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat> 
		<!-- 记得改 host 的名字 -->
		<writeHost host="hostM3" url="172.17.0.7:3306" user="root" password="123456">

		</writeHost>
	</dataHost>
```

在mycat 中执行customer相关的操作就会自动跳转到 localhost 2 上

而非 customer 操作就会走localhost 1 

在使用Navicat 客户端工具连接mycat 操作时，customer 表 和 其他表会轮流刷新，但是不影响使用



## 三、关于主库已经存有很多数据，要为其新增一个从库

```sql

# 可以看到 master 已经走出很远了（假设），bin 日志已经很大，position 也已经很大
# master 中有数据库，也有表，表中含有大量数据
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-master-bin.000001
         Position: 1045
     Binlog_Do_DB: test_db
 Binlog_Ignore_DB: mysql,sys,information_schema,performance_schema
Executed_Gtid_Set:
1 row in set (0.00 sec)

# 此时启动 slave，
# 创建要同步的数据库
# 使用 navicat 从 master 同步该数据库下的所有表结构到 slave
# 使用 navicat 从 master 同步该数据库下的所有数据到 slave
# 此时设置 slave 来同步master
change master to
master_host='192.168.50.200',
master_user='slave22',
master_password='1234',
master_log_file='mysql-master-bin.000001', # 注意和 master 一致  
master_port=3011,
master_log_pos=1045; # 注意和 master 一致  

# 然后再启动 slave
start slave;

# 此时可以对 master 进行增删改了

# master 更新之后，发现slave同步成功
show slave status;

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.50.200
                  Master_User: slave22
                  Master_Port: 3011
                Connect_Retry: 60
              Master_Log_File: mysql-master-bin.000001
          Read_Master_Log_Pos: 1045
               Relay_Log_File: 2383cf0b8bdd-relay-bin.000002
                Relay_Log_Pos: 614
        Relay_Master_Log_File: mysql-master-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```









## 四、sharding-proxy配置读写分离

> 本实例基于`单主单从mysql`
>
> 参考：https://shardingsphere.apache.org/document/4.1.1/cn/quick-start/sharding-proxy-quick-start/

### 1、下载mysql驱动

```shell
pwd
/home/luo/sharding_proxy/apache-shardingsphere-4.1.0-sharding-proxy-bin/lib
# 下载 mysql 驱动
wget https://archive.apache.org/dist/shardingsphere/4.1.0/apache-shardingsphere-4.1.0-sharding-proxy-bin.tar.gz
```



### 2、配置数据库

```shell
luo@ubuntu ~/s/apache-shardingsphere-4.1.0-sharding-proxy-bin> cat conf/config-master_slave.yaml

######################################################################################################
#
# If you want to connect to MySQL, you should manually copy MySQL driver to lib directory.
#
######################################################################################################

# 数据库名称
schemaName: test_db
#
dataSources:
# 配置数据源，名字随意（master_ds、slave_ds_0）
  master_ds:
    url: jdbc:mysql://127.0.0.1:3001/test_db?serverTimezone=UTC&useSSL=false
    username: root
    password: 1234
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
  slave_ds_0:
    url: jdbc:mysql://127.0.0.1:3002/test_db?serverTimezone=UTC&useSSL=false
    username: root
    password: 1234
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
#  slave_ds_1:
#    url: jdbc:mysql://127.0.0.1:3306/demo_ds_slave_1?serverTimezone=UTC&useSSL=false
#    username: root
#    password:
#    connectionTimeoutMilliseconds: 30000
#    idleTimeoutMilliseconds: 60000
#    maxLifetimeMilliseconds: 1800000
#    maxPoolSize: 50
#
masterSlaveRule:
  name: ms_ds
  # 配置主机所使用的数据源名称（上面 dataSources 下的字段）
  masterDataSourceName: master_ds
  slaveDataSourceNames:
  # 配置从机所使用的数据源名称
    - slave_ds_0
#    - slave_ds_1
```



### 3、配置对后端java程序暴露的逻辑账号

```shell
luo@ubuntu ~/s/a/conf> cat server.yaml

authentication:
  users:
    luo:  # 账号
      password: 1234 # 密码
#    sharding:
#      password: sharding
#      authorizedSchemas: sharding_db
props:
#  max.connections.size.per.query: 1
#  acceptor.size: 16  # The default value is available processors count * 2.
#  executor.size: 16  # Infinite by default.
#  proxy.frontend.flush.threshold: 128  # The default value is 128.
#    # LOCAL: Proxy will run with LOCAL transaction.
#    # XA: Proxy will run with XA transaction.
#    # BASE: Proxy will run with B.A.S.E transaction.
#  proxy.transaction.type: LOCAL
#  proxy.opentracing.enabled: false
#  proxy.hint.enabled: false
#  query.with.cipher.column: true
# 额外配置控制台显示sql
  sql.show: true
```



### 4、启动 sharing-proxy

```shell
# 指定端口为 4000
luo@ubuntu ~/s/a/bin> ./start.sh 4000
```



## 五、分库分表

> 参考：https://shardingsphere.apache.org/document/4.1.1/cn/manual/sharding-jdbc/configuration/config-yaml/
>
> 注意本配置文件中是`algorithmExpression`而不是`algorithmInlineExpression`。
>
> 超级注意⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️
>
> ```shell
> shardingColumn: create_user
>       # 注意：这里的 列名 必须在表中存在，如果不存在，sharding-proxy 并不会报错，
>       # 只是单纯的分库策略无效，一条记录会在每一库中都插入
>       # 举例：如果表中的 create_user 字段写成了其他名字，sharding-proxy并不会报错
>       # 反而会使得该表的分库策略无效，一个order记录会在每一个库中都存在
> ```
>
> 

### 1、配置

```shell
# sharding-proxy 对外暴露的 逻辑数据库的名称
schemaName: test_db

dataSources:
  ds_0:
    url: jdbc:mysql://127.0.0.1:3001/test_db?serverTimezone=UTC&useSSL=false
    username: root
    password: 1234
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
  ds_1:
    url: jdbc:mysql://127.0.0.1:3011/test_db?serverTimezone=UTC&useSSL=false
    username: root
    password: 1234
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
    
    
# 配置分库分表规则 
shardingRule:
  tables:
    # 对哪一张表进行分库分表
    or_order:
      # 这里极力建议表的 后缀数字从 0开始计数。
      # 只要分表策略是对任何正整数求余数，那么结果就一定是从0开始
      # 从 1开始只会增加不必要的麻烦
      # 要求 ds_0 数据源下面有 or_order_0 表 和 or_order_1 表
      # 要求 ds_1 数据源下面有 or_order_0 表 和 or_order_1 表
      actualDataNodes: ds_${0..1}.or_order_${0..1}
      # 分表策略
      tableStrategy:
        inline:
          # 根据表的哪一列进行分表
          shardingColumn: id
          # 设置分表的算法
          # 注意，如果是 一个数据库下有两张表，则两张表的后缀数字必须是 0 或 1
          # 因为这里的分表策略是对2求余数，结果只能是 0 或 1，所以表的名字也必须是 0 或 1
          algorithmExpression: or_order_${id % 2}
      keyGenerator:
        # id 的生成策略，使用雪花算法       
        # sharding-proxy 的雪花算法要求id的类型为 bigint
        type: SNOWFLAKE
        column: id

    or_order_item:
      actualDataNodes: ds_${0..1}.or_order_item_${0..1}
      tableStrategy:
        inline:
          # 两张表使用相同的分表策略，同一个create_user创建的订单就能都分配到 or_order_1 和 or_order_item_1
          # 或者 or_order_2 和 or_order_item_2 表中了      
          shardingColumn: id
          algorithmExpression: or_order_item_${id % 2}
      keyGenerator:
        type: SNOWFLAKE
        column: id

  bindingTables:
    # 表单的绑定策略，绑定了的话，如果在 ds_1 数据库中找到了记录，
    # 那么在其关联的订单项也必然在 ds_1 数组哭中，不用进行跨库 join 查询
    - or_order,or_order_item
  
  # 配置分库的策略    
  defaultDatabaseStrategy:
    inline:
      shardingColumn: create_user
      # 注意：这里的 列名 必须在表中存在，如果不存在，sharding-proxy 并不会报错，
      # 只是单纯的分库策略无效，一条记录会在每一库中都插入
      algorithmExpression: ds_${create_user % 2}
#  defaultTableStrategy:
#    none:
```

### 2、启动sharding-proxy

### 3、并在sharding-proxy中创建相应的表

> 也可以在物理数据库中创建相应的物理表`or_order_0`,`or_order_1`,`or_order_item_0`,`or_order_item_1`但是稍微显得麻烦

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : master-node11
 Source Server Type    : MySQL
 Source Server Version : 50736
 Source Host           : 192.168.50.200:3011
 Source Schema         : test_db

 Target Server Type    : MySQL
 Target Server Version : 50736
 File Encoding         : 65001

 Date: 29/04/2022 09:31:39
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for or_order
-- ----------------------------
DROP TABLE IF EXISTS `or_order`;
CREATE TABLE `or_order_0` (
  `id` bigint(20) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `create_user` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

-- ----------------------------
-- Table structure for or_order_item_0
-- ----------------------------
DROP TABLE IF EXISTS `or_order_item`;
CREATE TABLE `or_order_item` (
  `id` bigint(20) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  `create_user` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

SET FOREIGN_KEY_CHECKS = 1;

```



## 六、分库分表的同时配置主从分离

```shell
# 对外的统一数据库的名称
schemaName: sharding-master-slave

dataSources:
  ds0: 
    #driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.50.200:3001/test_db
    username: root
    password: 1234
  ds0_slave0: 
    #driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.50.200:3002/test_db
    username: root
    password: 1234
  # ds0_slave1: 
  #     driverClassName: com.mysql.jdbc.Driver
  #     url: jdbc:mysql://localhost:3306/ds0_slave1
  #     username: root
  #     password: 
  ds1: 
    #driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.50.200:3011/test_db
    username: root
    password: 1234
  ds1_slave0: 
    #driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.50.200:3022/test_db
    username: root
    password: 1234
  # ds1_slave1: 
  #       driverClassName: com.mysql.jdbc.Driver
  #       url: jdbc:mysql://localhost:3306/ds1_slave1
  #       username: root
  #       password: 

shardingRule:  

  defaultDataSourceName: ds0
  # 即使官网上 tables 位于 masterSlaveRules 之前
  # 但是是先有的读写分离，才有的分库分表
  # 所以要先配置 读写分离的 主从规则
  masterSlaveRules:
      master_ds0:
        # master 的数据源
        masterDataSourceName: ds0
        slaveDataSourceNames:
        # master 下的 slave 的数据源
          - ds0_slave0
          # - ds0_slave1
        loadBalanceAlgorithmType: ROUND_ROBIN
      master_ds1:
        masterDataSourceName: ds1
        slaveDataSourceNames: 
          - ds1_slave0
          # - ds1_slave1
        loadBalanceAlgorithmType: ROUND_ROBIN

  tables:
    or_order: 
      # 注意这里不能直接依赖于 dataSource
      # 而是只能依赖于 masterSlaveRules 中的 数据源
      actualDataNodes: master_ds${0..1}.or_order_${0..1}
      # 数据库的分库策略可以提出到公共部分
      # 但是依赖的数据源依然是 masterSlaveRules 中的 数据源
      # databaseStrategy:
      #   inline:
      #     shardingColumn: create_user
      #     algorithmExpression: master_ds${create_user % 2}
      tableStrategy: 
        inline:
          shardingColumn: id
          algorithmExpression: or_order_${id % 2}
      keyGenerator:
        type: SNOWFLAKE
        column: id
    or_order_item:
      # 依赖的数据源依然是 masterSlaveRules 中的 数据源
      actualDataNodes: master_ds${0..1}.or_order_item_${0..1}
      tableStrategy:
        inline:
          shardingColumn: id
          algorithmExpression: or_order_item_${id % 2}  
      keyGenerator:
        type: SNOWFLAKE
        column: id    
  bindingTables:
    - or_order,or_order_item 
  defaultTableStrategy:
    none:
  # 默认的分库策略，注意依赖的 是 masterSlaveRules 中的 数据源
  defaultDatabaseStrategy:
    inline:
      shardingColumn: create_user
      algorithmExpression: master_ds${create_user % 2}  
  defaultKeyGenerator:
    type: SNOWFLAKE
    column: id
  

# props:
  # sql:
    # show: true
```

