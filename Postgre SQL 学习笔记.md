# Postgre SQL 学习笔记

## 一、ubuntu 安装

> 参考：https://www.postgresql.org/download/linux/ubuntu/

```shell
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql
```

```shell
# 安装完成之后，会自动创建一个 postgres 用户
# 更改其系统密码
suod passwd postgres
# 切换到改用户
su postgres
# 使用 postgre sql 客户端命令行工具
postgres@ubuntu:~$ psql
psql (14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1))
Type "help" for help.

postgres=#
#  更改密码
postgres=# alter user postgres with password '1234'
```

### 1、允许任意ip连接数据库

```shell
luo@ubuntu:/etc/postgresql/14/main$ ll
total 68
drwxr-xr-x 3 postgres postgres  4096 Apr 26 19:34 ./
drwxr-xr-x 3 postgres postgres  4096 Apr 26 17:53 ../
drwxr-xr-x 2 postgres postgres  4096 Apr 26 17:53 conf.d/
-rw-r--r-- 1 postgres postgres   315 Apr 26 17:53 environment
-rw-r--r-- 1 postgres postgres   143 Apr 26 17:53 pg_ctl.conf
-rw-r----- 1 postgres postgres  5076 Apr 26 19:34 pg_hba.conf
-rw-r----- 1 postgres postgres  1636 Apr 26 17:53 pg_ident.conf
-rw-r--r-- 1 postgres postgres 29063 Apr 26 18:02 postgresql.conf
-rw-r--r-- 1 postgres postgres   317 Apr 26 17:53 start.conf
luo@ubuntu:/etc/postgresql/14/main$ sudo vi postgresql.conf
# 解除以下的注释
listen_addresses = '*'          # what IP address(es) to listen on;
```

