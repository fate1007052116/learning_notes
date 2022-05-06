# CentOS 8 安装 MySql 8.0.20

## 1. 初始化

   ```shell
   useradd mysql
   chown mysql:mysql /opt/mysql -R
   # 初始化mysql
   [root@RabbitMQ_1 bin]# ./mysqld --initialize
   2020-07-09T17:32:17.827736Z 0 [System] [MY-013169] [Server] /opt/mysql/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.20) initializing of server in progress as process 2036
   2020-07-09T17:32:17.885388Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
   2020-07-09T17:32:20.975672Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
   2020-07-09T17:32:23.284873Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: O2kTl!kniEt=   # 临时密码在这里
   
   # 启动MySql服务
   [root@RabbitMQ_1 bin]# ./mysqld
   2020-07-09T17:33:36.977217Z 0 [System] [MY-010116] [Server] /opt/mysql/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.20) starting as process 2129
   2020-07-09T17:33:37.002251Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
   2020-07-09T17:33:37.624998Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
   2020-07-09T17:33:37.849649Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: '/tmp/mysqlx.sock'
   2020-07-09T17:33:38.071621Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
   2020-07-09T17:33:38.088026Z 0 [System] [MY-010931] [Server] /opt/mysql/mysql-8.0.20-linux-glibc2.12-x86_64/bin/mysqld: ready for connections. Version: '8.0.20'  socket: '/tmp/mysql.sock'  port: 0  MySQL Community Server - GPL.
   
   # mysql 配置文件目录添加如下命令行跳过密码：/etc/my.cnf
   
   vim /etc/my.cnf 
   
   # my.cnf  中的内容
   [mysqld]
   basedir=/opt/mysql/mysql-8.0.20-linux-glibc2.12-x86_64/
   datadir=/opt/mysql/mysql-8.0.20-linux-glibc2.12-x86_64//data
   socket=/tmp/mysql.sock
   user=mysql
   port=3306
   skip-grant-tables
   
   # 跳过密码可以登陆，但是不可以修改密码
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'orochi0208';
   ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
   ```

## 2. 登陆异常

[mysql 报 error while loading shared libraries: libtinfo.so.5 解决办法](https://www.cnblogs.com/kaishirenshi/p/12667004.html)       

```shell
[root@RabbitMQ_1 bin]# ./mysql -uroot -p
./mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directoryl
# 即使 yum install -y libtinfo.so.5 也依旧不能解决
# 解决办法：
sudo ln -s  /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5
```
   linux安装MySQL报 error while loading shared libraries: libtinfo.so.5 解决办法

   MySQL 我采用的是 Linux- Generic 包安装，其中详细略过不表。一顿操作之后，终于到将 mysql 服务启动。但是到了连接服务的时候却报错了。

   mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory

   解决办法：
   sudo ln -s  /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5

## 3. 修改密码

   ```shell
   # 通过 ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root'; 命令来修改密码
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password 
   BY 'orochi0208';                                                           
   Query OK, 0 rows affected (0.00 sec)
   ```

## 4. 授权远程访问

   ```shell
   # 创建远程账户
   mysql> create user 'root'@'%' identified with mysql_native_password by 'orochi0208';                                                                  
   Query OK, 0 rows affected (0.00 sec)
   
   # 授予账户权限
   mysql> grant all privileges on *.* to 'root'@'%' with grant option;
   Query OK, 0 rows affected (0.00 sec)
   
   # 刷新权限
   flush privileges;
   Query OK, 0 rows affected (0.01 sec)
   ```

   