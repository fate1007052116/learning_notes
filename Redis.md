# Redis

redis的副版本号如果是奇数，代表是测试版，偶数是稳定版

6.0.5 副版本号为0，稳定版

3.1.2 副版本号1，测试版

## 一、安装

```shell
# 安装gcc编译器
yum install -y gcc-c++
# 进入到解压后的redis目录，编译
make
# 安装到指定的目录 PREFIX 后跟要安装到的目录
make install PREFIX=/usr/local/redis

# 将redis添加到环境变量中，这样可以在任何环境操作redis命令 （可选操作）
[root@RabbitMQ_1 bin]# pwd
/usr/local/redis/bin
[root@RabbitMQ_1 bin]# vi /etc/profile


export REDIS_HOME=/usr/local/redis/bin
export PATH=$PATH:$REDIS_HOME

[root@RabbitMQ_1 bin]# source /etc/profile

# 启动redis （前台启动）
redis-server 
# 将默认的启动方式改为后台启动
# 需要到redis的源码压缩包解压后的目录将redis.conf 拷贝到redis的bin目录中
cp /opt/redis-6.0.5/redis.conf /usr/local/redis/bin/
# 编辑已安装redis/bin目录下的redis.conf
vi /usr/local/redis/bin/redis.conf
# 将 daemonize no 改为 daemonize yes

################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes

# 通过指定刚刚配置的redis.conf 文件来后台启动redis
[root@RabbitMQ_1 /]# redis-server /usr/local/redis/bin/redis.conf 
9269:C 29 Jun 2020 03:59:32.812 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
9269:C 29 Jun 2020 03:59:32.813 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=9269, just started
9269:C 29 Jun 2020 03:59:32.813 # Configuration loaded

# 使用 redis自带的客户端工具连接redis  /usr/local/redis/bin/redis-cli 测试是否配置成功

[root@RabbitMQ_1 /]# redis-cli
127.0.0.1:6379> ping	# 输入ping ，返回PONG，说明连接成功
PONG
127.0.0.1:6379> 
# 关闭
./redis-cli studown
```

## 二、Redis 数据类型

### 1.字符串类型

Redis 字符串是字节序列，redis字符串是二进制安全的，这意味着他们有一个已知的长度没有任何特殊字符终止，所以可以存储任何东西，单个value 512MB是上限

```shell
127.0.0.1:6379> set luo junhua # set key value 将key value 放入redis
OK
127.0.0.1:6379> get luo
"junhua"

# incr (increment) 可以使得改key对应的value+1，前提该key的value是整数类型
127.0.0.1:6379> set num 2
OK
127.0.0.1:6379> incr num
(integer) 3

# increby 可以指定参数一次增加的数值，返回增加后的数值
127.0.0.1:6379> incrby num 10
(integer) 13

# decr (decrement)
127.0.0.1:6379> decr num
(integer) 12

# decreby
127.0.0.1:6379> decrby num 5
(integer) 7

# incrbyfloat 对浮点数做增量处理
127.0.0.1:6379> set pi 3.14
OK
127.0.0.1:6379> incrbyfloat pi 10
"13.14"

# append 向key-value 末尾追加value。如果key不存在则将该key的值设置为value (当成set命令来处理)，返回值是最佳字符串的总长度
127.0.0.1:6379> append luo 99
(integer) 8
127.0.0.1:6379> get luo
"junhua99"
	# key 不存在的情况
    127.0.0.1:6379> append gao kongyan
    (integer) 7
    127.0.0.1:6379> get gao
    "kongyan"

# mget / mset 作用于get /set 方法相似，但是可以同时 获得/设置 多个值
127.0.0.1:6379> mset key1 a key2 b key3 c
OK
127.0.0.1:6379> mget key1 key2 key3
1) "a"
2) "b"
3) "c"

# del 删除key (可同时删除多个)
127.0.0.1:6379> del key1 key2
(integer) 2

# 删除redis当前库中的所有数据，redis默认有16个库，这里删除的只是第一个库中的数据
127.0.0.1:6379> flushdb 
OK
127.0.0.1:6379> get luo
(nil) # 代表null
```

### 2.Hash（hash表）

Redis的hash是key-value的集合。redis的hash值是字符串字段和字符串值之间的映射，因此他们被用来表示对象

```shell
# hset key field value  存储一个hash key-value 集合
127.0.0.1:6379> hset user userName luo
(integer) 1

# hget key field	获取一个hash表中 一个key的值
127.0.0.1:6379> hget user userName
"luo"

# hmset hashTableName field1 value1 field2 value2 ..  创建一个hash表，并存储一个或者多个field-value
127.0.0.1:6379> hmset user userName luo age 18 
OK

#hmget hashTableName field1 field2 获取一个hash表中的多个属性的值
127.0.0.1:6379> hmget user userName age
1) "luo"
2) "18"

# hexists hashTableName field 判断这个hash表中的这个属性是否存在
127.0.0.1:6379> hexists user userName
(integer) 1		# 1代表存在，0代表不存在

# hdel hashTableName field1 field2 删除这个hash表中的属性
127.0.0.1:6379> hmget user userName age sex
1) "luo"
2) "18"
3) "male"
127.0.0.1:6379> hdel user userName sex
(integer) 2
127.0.0.1:6379> hmget user userName age sex
1) (nil)
2) "18"
3) (nil)

# hgetall hashTableName 获取hash表中所有属性及其value
127.0.0.1:6379> hgetall user 
1) "age" #属性名称
2) "18"	#属性值
3) "userName"
4) "luo"
5) "sex"
6) "male"

# hkeys hashTableName 只返回这个hash表中属性的名字（不返回属性的值）
127.0.0.1:6379> hkeys user
1) "age"
2) "userName"
3) "sex"

# hvals  hashTableName 只返回这个hash表中属性的值（不返回属性的名称）
127.0.0.1:6379> hvals user
1) "18"
2) "luo"
3) "male"

# hlen hashTableName 返回当前hash表中有几个属性
127.0.0.1:6379> hlen user
(integer) 3
```

### 3.链表

* Redis的链表是简单的字符串链表，排序插入顺序，可以添加元素到redis链表的头部或者尾部
* 每个value会单独的放在一个节点Node中
* 整个链表的名称就是一个key，这个key代表整个链表

```shell
# lpush 向链表的左侧添加（头部添加）
# rpush 右侧添加（尾部添加）
127.0.0.1:6379> lpush list1 redis 	# 当前还没有list1这个链表，链表会被自动创建
(integer) 1
127.0.0.1:6379> lpush list1 activeMQ
(integer) 2
127.0.0.1:6379> lpush list1 rabbitMQ
(integer) 3
127.0.0.1:6379> lrange list1 0 10
1) "rabbitMQ"
2) "activeMQ"
3) "redis"
127.0.0.1:6379> rpush list1 java
(integer) 4
127.0.0.1:6379> rpush list1 c
(integer) 5
127.0.0.1:6379> lrange list1 0 10
1) "rabbitMQ"
2) "activeMQ"
3) "redis"
4) "java"
5) "c"

# lpop listName 从左边取出一个元素，并在链表中删除这个元素
127.0.0.1:6379> lpop list1
"rabbitMQ"
127.0.0.1:6379> lrange list1 0 10
1) "activeMQ"
2) "redis"
3) "java"
4) "c"

# rpop listName 从右边取出一个元素，并在链表中删除这个元素
127.0.0.1:6379> rpop list1
"c"
127.0.0.1:6379> lrange list1 0 10
1) "activeMQ"
2) "redis"
3) "java"

# llen listName 返回链表中元素的个数，相当于 select count(*)
127.0.0.1:6379> llen list1
(integer) 3

# lrange listName start end 遍历链表，不改变任何数据，索引从0开始
127.0.0.1:6379> lrange list1 0 10
1) "activeMQ"
2) "redis"
3) "java"
# lrange 也支持负数作为索引，链表最后一个元素的索引为-1，从右往左依次递减，-2，-3，-4
		# -2 表示从右往左数的第二个元素
		# -1 表示从右往左数的第一个元素
		# 当start 为 负数的时候，end也必须为负数（为0也不可以），否则start 就在end的后面，遍历结果为empty
127.0.0.1:6379> lrange list1 -2 -1
1) "redis"
2) "java"

# lindex listName indexNumber 用于将链表当作数组来使用，用来返回指定索引的元素，索引从0开始
	# 没有rindex命令
127.0.0.1:6379> lindex list1 0
"activeMQ"

# lset 用于设置指定下表元素的值（赋值操作）
127.0.0.1:6379> lrange list1 0 10
1) "activeMQ"
2) "redis"
3) "java"
127.0.0.1:6379> lset list1 1 newElement
OK
127.0.0.1:6379> lrange list1 0 10
1) "activeMQ"
2) "newElement"
3) "java"
```

### 4.set

* redis的set是字符串的无序集合

* set中不允许有重复的value

```shell
# sadd setName value1 value2 value3 添加一个string元素到对应set集合中，
# 如果这个元素在集合中已经存在，则返回0
127.0.0.1:6379> sadd set1 v1 v2 v3
(integer) 3
127.0.0.1:6379> sadd set1 v1
(integer) 0	#v1已经在set1集合中存在，所以返回0

# scard setName 返回set中元素的个数
# 如果set不存在或者set集合中元素为0个，则返回0
127.0.0.1:6379> scard set1
(integer) 3
127.0.0.1:6379> scard lalala
(integer) 0	# set 不存在

# smembers setName 返回set中所有的元素，结果是无序的
127.0.0.1:6379> smembers set1
1) "v2"
2) "v1"
3) "v3"

# sIsMember setName value 判断value是否存在于这个set集合中
# 0表示set不存在或者set结合中不存在这个value
127.0.0.1:6379> sismember set1 v1
(integer) 1
127.0.0.1:6379> sismember set1 v4
(integer) 0

# 从set中移除给定的元素
# 如果这个元素在set集合中不存在或者这个set集合不存在，则返回0
127.0.0.1:6379> srem set1 v2
(integer) 1
127.0.0.1:6379> smembers set1
1) "v1"
2) "v3"
```

### 5.SortedSet（有序集合）zset

* redis的有序集合（zset）类似与redis的集合（set），字符串不能重复的集合
* 相比set，提供了一个排序的功能
* 但是要依赖于给元素提供一个排序标记（score）

```shell
# zadd zSetName score value 如果score一致，会按照value的字典顺序进行排序，
# 		如果score的值不一样，则会按照score的值进行排序
127.0.0.1:6379> zadd zset1 0 redis
(integer) 1
127.0.0.1:6379> zadd zset1 0 rabbitMQ
(integer) 1
127.0.0.1:6379> zadd zset1 0 activeMQ
(integer) 1
127.0.0.1:6379> zadd zset1 1 bbu
(integer) 1
127.0.0.1:6379> zrange zset1 0 10
1) "activeMQ"
2) "rabbitMQ"
3) "redis"
4) "bbu"	# 虽然字典顺序是b，但是score是1，这是主要根据score来排序

# 小技巧：zrange zset1 0 -1 可以遍历所有集合中的元素，-1代表从右往左数，即倒数第一个元素
# zrange setName start end 遍历元素，类似 lrange setName start end 
127.0.0.1:6379> zrange zset1 0 -1
1) "activeMQ"
2) "rabbitMQ"
3) "redis"
4) "bbu"
# 可以添加withscores 参数，连同score一起输出
127.0.0.1:6379> zrange zset1 0 -1 withscores
1) "activeMQ"
2) "0"
3) "rabbitMQ"
4) "0"
5) "redis"
6) "0"
7) "bbu"
8) "1"

# zremrangebyscore zSetName min max 根据score的范围来删除元素【闭区间】
127.0.0.1:6379> zremrangebyscore zset1 0 0
(integer) 3
127.0.0.1:6379> zrange zset1 0 -1
1) "bbu"
```

## 三、Redis 中的其他命令

```shell
# 切换数据库
127.0.0.1:6379> select 1
OK

# 测试是否连接redis，如果已连接，则会返回pong
127.0.0.1:6379> ping
PONG

# 使用echo命令也可以测试是否连接redis，echo "string" 如果返回 "string" 则说明已连接
127.0.0.1:6379> echo hello
"hello"

# keys * 可以返回当前redis库（默认共有16个库）中所有key的名称，可以使用 * 作为通配符
127.0.0.1:6379> keys *
1) "set1"
2) "list1"
3) "zset1"

# exists strKey 判断一个字符串类型的key是否存在
127.0.0.1:6379> exists luo
(integer) 1

# expire key times（秒） 设置一个key的过期时间，时间到达后会自动删除key和value
127.0.0.1:6379> expire luo 30 
(integer) 1


# ttl keyName 查看key的剩余过期时间（已设置过key的过期时间才有剩余过期时间），
# 如果返回的是正数，则表示该key的剩余过期时间（秒）
127.0.0.1:6379> ttl luo
(integer) 27
127.0.0.1:6379> ttl luo
(integer) 26
127.0.0.1:6379> ttl luo
(integer) 26
127.0.0.1:6379> ttl luo
(integer) 25
# 如果返回-1 则表示该key没有设置过期时间
127.0.0.1:6379> ttl luo
(integer) -1
# 如果返回-2 则表示该key-value已经被删除
127.0.0.1:6379> ttl luo
(integer) -2
127.0.0.1:6379> get luo	#失效时间已到，所以无法获取该key 
(nil)

# persist keyName 取消该key的过期时间，转为持久化key（没有过期时间）
127.0.0.1:6379> set luo 1111
OK
127.0.0.1:6379> expire luo 30
(integer) 1
127.0.0.1:6379> ttl luo
(integer) 28
127.0.0.1:6379> ttl luo
(integer) 27
127.0.0.1:6379> persist luo
(integer) 1
127.0.0.1:6379> ttl luo
(integer) -1	# 没有设置过期时间
#前提是要在 该key剩余时间>0  的情况下设置为持久化key
127.0.0.1:6379> ttl luo
(integer) -1										# 默认持久化节点
127.0.0.1:6379> expire luo 10   # 设置过期时间
(integer) 1											
127.0.0.1:6379> persist luo			# 过了10秒以上再设置为持久化key
(integer) 0											# 返回0，说明转为持久化节点失败
127.0.0.1:6379> ttl luo
(integer) -2										# key已经过期且被删除
127.0.0.1:6379> get luo
(nil)														# 已被删除而无法获取

# select dbIndex 切换数据库（redis默认有16个库【0,15】），通过下标切换到该库
# 每个库之间是隔离的
127.0.0.1:6379> select 15
OK
127.0.0.1:6379[15]> keys *		# 可见新切换的库15，没有key
(empty array)

# move keyName dbIndex 转移当前key到指定下表的数据库中
127.0.0.1:6379> keys *
1) "set1"
2) "list1"
3) "zset1"
127.0.0.1:6379> move list1 15		# 将库0中的 list1 移动到 库15中
(integer) 1
127.0.0.1:6379> select 15			#切换到库15
OK
127.0.0.1:6379[15]> keys *		# 库15多出来的key
1) "list1"
127.0.0.1:6379[15]> lrange list1 0 -1
1) "activeMQ"
2) "newElement"
3) "java"
127.0.0.1:6379[15]> select 0	# 切换到库 1
OK
127.0.0.1:6379> lrange list1 0 -1 # 库1中的list1已经被移走，不存在
(empty array)

# dbsize 返回当前库中key的数量，数量和keys * 命令返回结果的个数一致
127.0.0.1:6379> keys *
1) "set1"
2) "zset1"
127.0.0.1:6379> dbsize 
(integer) 2

# info 获取当前服务器信息的统计

# flushdb 删除当前库中所有的key

# flushall 删除所有库（16个库）中的key

# quit 退出

# 使用redis-cli 工具正常退出redis
[root@RabbitMQ_1 bin]# redis-cli shutdown
```

## 四、Redis.conf 配置文件

```shell
# 配置redis默认是后台启动
################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes

# 设置redis端口
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379

# 绑定监听本级物理网卡的ip地址，若开放所有ip（0.0.0.0），则任何连接到此redis的客户端都可以操作redis
# 因为redis是基于ip来进行权限控制的
# bind 0.0.0.0 监听本机所有物理网卡的ip
bind 127.0.0.1	# 不能绑定本地回环地址

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes	# 改为no，

# 配置日志等级【debug、verbose、notice、warning】
loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""

# 配置dataBase的个数
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16

# redis 的持久化方案（RDB方案）
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed  900秒内若有一个key改变，就执行持久化
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000

# 使用RDB备份数据到磁盘时，生成的数据文件是什么
# The filename where to dump the DB
dbfilename dump.rdb

# 上面数据文件存放的目录
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./

# AOF 开启/关闭 AOF和RDB一样是一个备份机制（可以同时开启）
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no

# 配置AOF的文件名
# The name of the append only file (default: "appendonly.aof")
appendfilename "appendonly.aof"


# 开启/关闭 redis集群
################################ REDIS CLUSTER  ###############################

# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes
```

## 五、redis中的数据持久化

将内存中的数据持久化到磁盘中

### 1. RDB方式

* 对内存中的数据库状态进行快照
* 将Redis在内存中的数据库状态保存到磁盘里面，RDB文件是一个经过压缩的二进制文件，通过该文件可以还原生成RDB文件时的数据库状态
* 默认情况下，持久化到dump.rdb文件（dbfilename dump.rdb），并在redis重启后，自动读取其中的文件
* 据悉，通常情况下一千万的字符串类型key，1GB的快照文件，同步到内存中的时间是20-30秒

#### （1）RDB生成的方式

默认在/usr/local/redis/bin/目录下会有一个dump.rdb文件（之前在redis.conf中配置过），如果没有，则在redis-cli中使用save命令，就可以生成dump.rdb文件了

##### 【1】执行命令手动生成RDB文件

有两个命令可以生成RDB文件
* save 命令会阻塞Redis服务进程，直到RDB文件创建完毕为止，在服务进程阻塞期间，服务器不能处理任何命令请求
* bgsave (background save)派生出一个子进程，然后由子进程负责创建RDB文件，服务器（父进程）继续处理命令请求，创建RDB文件结束之前，客户端发送的bgsave 和 save 命令会被服务器拒绝

##### 【2】通过配置文件自动生成

* 可以设置服务器配置的save选项，让服务器每隔一段时间自动执行一次bgsave命令

* 可以通过save选项设置多个保存条件，但只要其中任意一个条件被满足，服务器就会执行bgsave命令

  ```shell
  # redis.conf 中的配置项
  
  # 例如，这里写的是save，但实际是通过bgsave来完成持久化操作的
  save 900 1	# 900秒内且有一个key改变，就执行持久化
  save 300 10 # 300秒内且有 10 个key改变，就执行持久化
  save 60 10000 # 60秒内且有10000 个 key改变，就执行持久化
  
  # 配置dump.rdb 文件存放的位置，这样会导致redis-server 在哪一个目录中启动，dump.rdb文件就会存放到这个路径，存放的位置根据启动的路径
  dir ./  # 备份文件存放在启动redis时所在的目录
  ```

### 2. AOF方式

* AOF持久化方式在redis中默认是关闭的，需要修改配置文件开启该方式

* AOF：把每条命令都写入文件，类似mysql的binlog日志

* AOF方式：是通过保存redis服务器所==***执行写命令来记录数据库状态的文件***==，查询操作不会写入AOF文件 

* AOF文件刷新的方式

   1. Appendfsync always 

      每提交一个修改命令都调用fsync刷新到AOF文件，非常慢，非常安全

   2. appendfsync everysec （默认）

      每秒钟都调用fsync刷新到AOF文件，很快，但可能会丢失一秒以内的数据

   3. Appendfsync no
   
      依靠OS进行刷新，redis不主动刷新AOF，这样最快，安全性差

```shell
# 开启AOF
appendonly yes
appendfilename "appendonly.aof"
# AOF 刷新策略
# appendfsync always
appendfsync everysec
# appendfsync no
```

* AOF 数据恢复方式

  服务器在启动时，通过载入和执行 **appendonly.aof** 文件中保存的命令来还原服务器关闭之前的数据库状态，具体过程

  1. 载入 **appendonly.aof** 文件
  2. 创建模拟客户端
  3. 从AOF文件中读取一条指令
  4. 使用模拟客户端执行命令
  5. 循环读取并执行命令，直到全部完成
  6. 如果同时启用了RBD和AOF方式，AOF优先，启动时只加载AOF文件恢复数据

## 六、安装Redis集群

Redis 3.0.0之后支持集群，集群要求集群节点中必须支持主备模式，也就是**说集群中的主节点（Master）至少有一个从节点**

每一个蓝色的圈都代表一个redis集群中的主节点。他们任何两个节点之间都是相互连通的。客户端可以与任何一个节点相连接，然后就可以访问集群中的任何一个节点。对其进行存取和其他操作

### Redis集群架构图

![image-20200630111138034](/Users/luo/Library/Application Support/typora-user-images/image-20200630111138034.png)

### redis集群的容错机制

* Redis 之间通过互相的ping-pong 判断是否节点可以连接上。如果有一半以上的节点去ping 一个节点的时候没有回应，集群就认为这个节点宕机了，然后去连接它的从节点。如果某个节点和所有从节点全部挂掉，我们集群就进入fail 状态。
* 还有就是如果有一半以上的主节点宕机，那么我们集群同样进入fail 了状态。这就是我们的redis 的投票机制，具体原
  理如下图所示

![image-20200630111837994](/Users/luo/Library/Application Support/typora-user-images/image-20200630111837994.png)

投票过程是集群中所有master 参与,如果半数以上master 节点与master 节点通信超时(cluster-node-timeout),认为当前master 节点挂掉.

### 什么时候整个集群不可用(cluster_state:fail)?

1.  如果集群任意master 挂掉,且当前master 没有slave。此时集群进入fail 状态,也可
      		以理解成集群的slot 映射[0-16383]不完整时进入fail 状态。
2. 如果集群超过半数以上master 挂掉，无论是否有slave，集群进入fail 状态.

## 七、Redis-cluster 数据存储

redis无论有多少节点，最终都会给每个节点分配一些槽，槽的范围是[0,16383]

当我们的存取的key 到达的时候，redis 会根据crc16 的算法得出一个结果，然后把结果对16384 求余数，这样每个key 都会对应一个编号在0-16383 之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。

![image-20200630122750344](/Users/luo/Library/Application Support/typora-user-images/image-20200630122750344.png)

在Node1 执行set name kevin

1. 使用CRC16 算法对key 进行计算，得到一个数字，然后对数字进行取余。
   CRC16 : name = 26384
   26384%16384 = 10000
2. 查找到包含10000 插槽的节点，比如是node2，自动跳转到node2
3. 在node2 上执行set name kevin 命令完成数据的插入
4. 如果在node1 上执行get name，先使用CRC16 算法对key 进行计算，再使用16384 取余，得到插槽的下标，然后跳到拥有该插槽的node2 中执行get name 命令，并返回结果。

## 八、Redis colony

如果使用已经使用过的单机版创建集群时，需要删除dump.rdb 与apeendonly.aof 文件。因为初始化的时候会执行dump.rdb apeendonly.aof 之一，redis将数据写入集群的时候需要知道slot所在redis节点，此时不知道这些信息，所以会初始化失败

redis集群的安装需要依赖与redis src解压后的一个ruby脚本来进行安装

```shell
[root@RabbitMQ_1 src]# pwd
/opt/redis-6.0.5/src
[root@RabbitMQ_1 src]# ll *.rb
-rwxrwxr-x. 1 root root 3600 6月   9 06:19 redis-trib.rb
```

因为这个rb脚本是基于ruby语言编写的，所以要安装ruby运行时依赖

```shell
# 安装ruby 环境
yum install -y ruby

# 安装ruby的包管理器
yum install -y rubygems

# 这个redis-trib.rb脚本的执行需要依赖于一些其他的ruby 包所以我们还要下载一个redis-4.2.1.gem
https://rubygems.org/gems/redis/versions/4.2.1		#(点击右下角的下载)
```

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200630153947977.png" alt="image-20200630153947977" style="zoom:50%;" />

```shell
# 安装刚上传的redis-4.2.1.gem
-rw-r--r--.  1 root root     58880 6月  30 03:38 redis-4.2.1.gem
[root@RabbitMQ_1 opt]# gem install redis-4.2.1.gem 
Successfully installed redis-4.2.1
1 gem installed

# 先启动6个redis实例
# 因为没有实例，先创建一个redis-cluster 文件夹
# 可以复制已安装的redis来快速创建集群，记得删除 dump.rdb 与apeendonly.aof
# 配置redis.conf 开启redis集群
cluster-enabled yes
# 因为是伪集群，所以每个redis的端口号不可以完全相同
port 8001
```

我的例子是依次每个redis使用端口8001，8002，8003，8004，8005，8006

```shell
# 把redis源码压缩包中的创建集群的ruby 脚本复制到redis-cluster 中
[root@RabbitMQ_1 src]# cp /opt/redis-6.0.5/src/redis-trib.rb /opt/redis-cluster/

# 启动六个redis实例，创建一个sh脚本，一键启动
cd redis_01
./redis-server redis.conf
cd ..
cd redis_02
./redis-server redis.conf
cd ..
cd redis_03
./redis-server redis.conf
cd ..
cd redis_04
./redis-server redis.conf
cd ..
cd redis_05
./redis-server redis.conf
cd ..
cd redis_06
./redis-server redis.conf
cd ..

[root@RabbitMQ_1 redis-cluster]# ll
总用量 8
drwxr-xr-x. 2 root root  190 6月  30 04:45 redis_01
drwxr-xr-x. 2 root root  190 6月  30 04:46 redis_02
drwxr-xr-x. 2 root root  190 6月  30 04:46 redis_03
drwxr-xr-x. 2 root root  190 6月  30 04:47 redis_04
drwxr-xr-x. 2 root root  190 6月  30 04:47 redis_05
drwxr-xr-x. 2 root root  190 6月  30 04:48 redis_06
-rwxr-xr-x. 1 root root 3600 6月  30 04:54 redis-trib.rb
-rwxr-xr-x. 1 root root  264 6月  30 04:59 startAll.sh
[root@RabbitMQ_1 redis-cluster]# ./startAll.sh 
2141:C 30 Jun 2020 05:00:22.845 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2141:C 30 Jun 2020 05:00:22.845 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=2141, just started
2141:C 30 Jun 2020 05:00:22.845 # Configuration loaded
2143:C 30 Jun 2020 05:00:22.866 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2143:C 30 Jun 2020 05:00:22.866 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=2143, just started
2143:C 30 Jun 2020 05:00:22.866 # Configuration loaded
2149:C 30 Jun 2020 05:00:22.886 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2149:C 30 Jun 2020 05:00:22.886 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=2149, just started
2149:C 30 Jun 2020 05:00:22.886 # Configuration loaded
2155:C 30 Jun 2020 05:00:22.908 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2155:C 30 Jun 2020 05:00:22.908 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=2155, just started
2155:C 30 Jun 2020 05:00:22.908 # Configuration loaded
2161:C 30 Jun 2020 05:00:22.927 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2161:C 30 Jun 2020 05:00:22.927 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=2161, just started
2161:C 30 Jun 2020 05:00:22.927 # Configuration loaded
2167:C 30 Jun 2020 05:00:22.950 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2167:C 30 Jun 2020 05:00:22.950 # Redis version=6.0.5, bits=64, commit=00000000, modified=0, pid=2167, just started
2167:C 30 Jun 2020 05:00:22.950 # Configuration loaded
# 查看redis是否启动成功
[root@RabbitMQ_1 redis-cluster]# ps aux | grep redis
root       2142  0.1  1.1  61996  9700 ?        Ssl  05:00   0:00 ./redis-server 127.0.0.1:8001 [cluster]		# 相比单机版redis，多出了cluster标志
root       2148  0.1  1.1  61996  9596 ?        Ssl  05:00   0:00 ./redis-server 127.0.0.1:8002 [cluster]
root       2154  0.1  1.1  61996  9700 ?        Ssl  05:00   0:00 ./redis-server 127.0.0.1:8003 [cluster]
root       2160  0.1  1.1  61996  9708 ?        Ssl  05:00   0:00 ./redis-server 127.0.0.1:8004 [cluster]
root       2162  0.1  1.1  61996  9692 ?        Ssl  05:00   0:00 ./redis-server 127.0.0.1:8005 [cluster]
root       2172  0.1  1.1  61996  9684 ?        Ssl  05:00   0:00 ./redis-server 127.0.0.1:8006 [cluster]
root       2178  0.0  0.1  12320  1052 pts/0    S+   05:00   0:00 grep --color=auto redis


# 通过redis安装包自带的脚本来创建集群(redis 3.0.0版本可用)
# --replicas 1 代表一主一备的集群（1个master，1个slave）
./redis-trib.rb create --replicas 1 192.168.2.10:8001 192.168.2.10:8002 192.168.2.10:8003 192.168.2.10:8004 192.168.2.10:8005 192.168.2.10:8006
WARNING: redis-trib.rb is not longer available!
You should use redis-cli instead.

All commands and features belonging to redis-trib.rb have been moved
to redis-cli.
In order to use them you should call redis-cli with the --cluster
option followed by the subcommand name, arguments and options.

Use the following syntax:
redis-cli --cluster SUBCOMMAND [ARGUMENTS] [OPTIONS]

Example:	# 他都已经帮我们写好了，命令，随便用哪个redis客户端工具执行一下即可
redis-cli --cluster create 192.168.2.10:8001 192.168.2.10:8002 192.168.2.10:8003 192.168.2.10:8004 192.168.2.10:8005 192.168.2.10:8006 --cluster-replicas 1

To get help about all subcommands, type:
redis-cli --cluster help

# 直接执行还不可以，因为之前改了端口，需要在使用客户端工具连接redis的时候指定ip+端口
redis-cli -h IP -p 6379，-h后面接redis服务器的IP地址，-p后面接端口

# 使用真实ip还执行不了命令，因为在redis.conf中bind 127.0.0.1 只能通过这个ip来连接redis
./redis-cli -h 127.0.0.1 -p 8001 --cluster create 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003 127.0.0.1:8004 127.0.0.1:8005 127.0.0.1:8006 --cluster-replicas 1
[ERR] Node 127.0.0.1:8001 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0. #报错，提示要清空数据库
# 这就是没删dump.rdb apeendonly.aof 的后果

# 错误配置集群（只用配置一次），如果使用127.0.0.1作为redis的地址，则外网无法访问
[root@RabbitMQ_1 redis_01]# ./redis-cli -h 127.0.0.1 -p 8001 --cluster create 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003 127.0.0.1:8004 127.0.0.1:8005 127.0.0.1:8006 --cluster-replicas 1

# 完美配置 (配置时，如果被拒绝连接，则需要将所有redis的redis.conf中的 bind 127.0.0.1 注释,也可能时redis集群没有启动)
[root@RabbitMQ_1 redis_01]# ./redis-cli -h 127.0.0.1 -p 8001 --cluster create 192.168.2.10:8001 192.168.2.10:8002 192.168.2.10:8003 192.168.2.10:8004 192.168.2.10:8005 192.168.2.10:8006 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.2.10:8005 to 192.168.2.10:8001
Adding replica 192.168.2.10:8006 to 192.168.2.10:8002
Adding replica 192.168.2.10:8004 to 192.168.2.10:8003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: d191b45516d18d74d0666143b9c93674dfc054b0 192.168.2.10:8001
   slots:[0-5460] (5461 slots) master
M: cb9f1d2b0f3c833980868dd89c40997901ac3668 192.168.2.10:8002
   slots:[5461-10922] (5462 slots) master
M: 4745a4e8d16acb2aff689a9a78d4e4b6c3748745 192.168.2.10:8003
   slots:[10923-16383] (5461 slots) master
S: cc6ac64397c8a159d8ad667e36fa5ad7a3ae2d34 192.168.2.10:8004
   replicates d191b45516d18d74d0666143b9c93674dfc054b0
S: 502e8a8e88c47d7a04942671a6e3a8090dd36658 192.168.2.10:8005
   replicates cb9f1d2b0f3c833980868dd89c40997901ac3668
S: c8c45d3471816d9f35e64e77f5f5dea7b0187ba8 192.168.2.10:8006
   replicates 4745a4e8d16acb2aff689a9a78d4e4b6c3748745
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 192.168.2.10:8001)
M: d191b45516d18d74d0666143b9c93674dfc054b0 192.168.2.10:8001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: cb9f1d2b0f3c833980868dd89c40997901ac3668 192.168.2.10:8002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: cc6ac64397c8a159d8ad667e36fa5ad7a3ae2d34 192.168.2.10:8004
   slots: (0 slots) slave
   replicates d191b45516d18d74d0666143b9c93674dfc054b0
S: c8c45d3471816d9f35e64e77f5f5dea7b0187ba8 192.168.2.10:8006
   slots: (0 slots) slave
   replicates 4745a4e8d16acb2aff689a9a78d4e4b6c3748745
S: 502e8a8e88c47d7a04942671a6e3a8090dd36658 192.168.2.10:8005
   slots: (0 slots) slave
   replicates cb9f1d2b0f3c833980868dd89c40997901ac3668
M: 4745a4e8d16acb2aff689a9a78d4e4b6c3748745 192.168.2.10:8003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 集群建立之后，每个redis的目录下都会多出一个nodes.conf文件，这个是集群配置文件
# 如果集群配置不成功，可以删除每个redis的nodes.conf文件

# 使用客户端工具连接集群的时候一定要指定ip、端口号、-c 参数，没有-c参数则无法写入key-value
# -c 参数表示当前连接的是一个集群
[root@RabbitMQ_1 redis_01]# ./redis-cli -c -p 8006
```

### 1、使用Jedis操作Redis注意

```shell
redis.clients.jedis.exceptions.JedisDataException: DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 
1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 
2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 
3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 
4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.

# 需要关闭redis的保护模式，否则jedis无法连接redis
[root@RabbitMQ_1 bin]# ./redis-cli 
127.0.0.1:6379> config set protected-mode "no" # 临时有效
OK


[root@RabbitMQ_1 redis-cluster]# netstat -anp | grep redis
tcp        0      0 127.0.0.1:8001          0.0.0.0:*               LISTEN      2137/./redis-server 
tcp        0      0 127.0.0.1:8002          0.0.0.0:*               LISTEN      2143/./redis-server 
tcp        0      0 127.0.0.1:8003          0.0.0.0:*               LISTEN      2149/./redis-server 
tcp        0      0 127.0.0.1:8004          0.0.0.0:*               LISTEN      2155/./redis-server 
tcp        0      0 127.0.0.1:8005          0.0.0.0:*               LISTEN      2161/./redis-server 
tcp        0      0 127.0.0.1:8006          0.0.0.0:*               LISTEN      2167/./redis-server 
tcp        0      0 192.168.2.10:6379       0.0.0.0:*               LISTEN      2547/./redis-server 
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      2547/./redis-server 
tcp 

# 还需要配置redis.conf，否则redis不监听本机的物理网卡
# 或者直接注释 #bind 127.0.0.1
bind 0.0.0.0
```



## 九、lua脚本

> 客户端输出了服务器返回的通用错误消息，注意这是一个动态抛出的异常，Redis 会保护主线程不会因为脚本的错误而导致服务器崩溃，近似于在脚本的外围有一个很大的 try catch 语句包裹。在 lua 脚本执行的过程中遇到了错误，同 redis 的事务一样，那些通过 **redis.call 函数已经执行过的指令对服务器状态产生影响是无法撤销**的，在编写 lua 代码时一定要小心，避免没有考虑到的判断条件导致脚本没有完全执行。
>
> 　　lua 原生没有提供 try catch 语句，异常包裹语句究竟是用什么来实现的呢？lua 的替代方案是内置 pcall(f) 函数调用。pcall 的意思是 protected call，会让 f 函数运行在保护模式下，f 如果出现了错误，pcall 调用会返回 false 和错误信息。而普通的 call(f) 调用在遇到错误时只会向上抛出异常。在 Redis 的源码中可以看到 lua 脚本的执行被包裹在 pcall 函数调用中。
>
> 　　Redis 在 lua 脚本中除了提供了 redis.call 函数外，同样也提供了 redis.pcall 函数。前者遇到错误向上抛出异常，后者会返回错误信息。使用时一定要注意 call 函数出错时会中断脚本的执行，为了保证脚本的原子性，要谨慎使用。　　





```lua
eval "for b=1, table.getn( ARGV),1  do if tonumber(redis.call('get',KEYS[b])) < tonumber(ARGV[b]) then  error(-1)  end b = b + 1 end for c=1,table.getn(ARGV),1 do redis.call('decrby' , KEYS[c],ARGV[c]) end return 100  " 2 1 2 7 3
```





## 十、支持部分事务

### 1、redis的事务

可以一次执行多个命令，本质是一组命令的集合。一个事务中的命令都会被序列化，按顺序的串行化执行而不被其他命令插入，不许阻塞

一个队列中，一次性，顺序性，拍他性的执行一系列命令

```java
redisTemplate.multi();	  // 标记一个事务块的开始

redisTemplate.exec();			// 执行所有事物块内的命令
redisTemplate.discard(); // 取消事务，放弃执行块内的所有命令
redisTemplate.watch("key"); // 监视一个或多个key，如果在事务执行之前，这个（或这些）key，被其他命令所改动，那么事务将被打断
redisTemplate.unwatch();   // 取消 watch 命令对所有key的监视

```

```shell
127.0.0.1:6379[11]> keys *
(empty array)
127.0.0.1:6379[11]> MULTI
OK
127.0.0.1:6379[11](TX)> set k1 10			# 事务期间，保证隔离性，其他会话无法看到未提交的的数据
QUEUED
127.0.0.1:6379[11](TX)> set k2 20
QUEUED
127.0.0.1:6379[11](TX)> set k3 30
QUEUED
127.0.0.1:6379[11](TX)> EXEC
1) OK
2) OK
3) OK
```



```shell
127.0.0.1:6379[11]> MULTI
OK
127.0.0.1:6379[11](TX)> acbd k1				# 输入一条错误的命令，事物块内的命令，只要有一个未能通过语法检查，那么所有的命令都会失败
(error) ERR unknown command `acbd`, with args beginning with: `k1`,
127.0.0.1:6379[11](TX)> set k4 v4
QUEUED
127.0.0.1:6379[11](TX)> set k5 v5
QUEUED
127.0.0.1:6379[11](TX)> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379[11]> get k4
(nil)
127.0.0.1:6379[11]> get k5
(nil)
```



```shell
127.0.0.1:6379[11]> flushdb
OK
127.0.0.1:6379[11]> set k1 v1
OK
127.0.0.1:6379[11]> MULTI
OK
127.0.0.1:6379[11](TX)> INCR k1  # k1 的值是一个 字符串，不能 + 1，这条命令能通过语法检查，但是在实际执行的时候会报错
QUEUED
127.0.0.1:6379[11](TX)> set k2 22
QUEUED
127.0.0.1:6379[11](TX)> set k3 33
QUEUED
127.0.0.1:6379[11](TX)> EXEC
1) (error) ERR value is not an integer or out of range  # 实际执行的时候就报错了
2) OK
3) OK
127.0.0.1:6379[11]> get k2   # 但是执行时的报错，并不会影响后续命令的执行
"22"
127.0.0.1:6379[11]> get k3
"33"
127.0.0.1:6379[11]> get k1
"v1"
```

### 2、watch

监视一个或多个key，如果在事务执行之前，这个（或这些）key，被其他命令所改动，那么事务将被打断

正常普通的事务

会话一：

```shell
127.0.0.1:6379[11]> flushdb
OK
127.0.0.1:6379[11]> set balance 100
OK
127.0.0.1:6379[11]> set debt 0
OK
127.0.0.1:6379[11]> multi
OK
127.0.0.1:6379[11](TX)> decrby balance 10
QUEUED
127.0.0.1:6379[11](TX)> incrby debt 10			# 事务执行期间，有其他人对 balance 进行更改
QUEUED
# 会话二：开始

# 会话一：结束
127.0.0.1:6379[11](TX)> EXEC  # 可以看出，没有加 watch 的情况下，当 exec 提交执行命令的队列之后，对现有最新的数据进行操作
1) (integer) 190  # 有返回结果，执行成功
2) (integer) 10
127.0.0.1:6379[11]> mget balance debt
1) "190"
2) "10"
```

会话二：

```shell
127.0.0.1:6379[11]> get balance
"100"
127.0.0.1:6379[11]> set balance 200
OK
```



使用`watch`之后

会话三：

```shell
127.0.0.1:6379[11]> flushdb
OK
127.0.0.1:6379[11]> set balance 100
OK
127.0.0.1:6379[11]> set debt 0
OK
127.0.0.1:6379[11]> watch balance debt
OK
127.0.0.1:6379[11]> MULTI
OK
127.0.0.1:6379[11](TX)> decrby balance 50
QUEUED
127.0.0.1:6379[11](TX)> incrby debt 50
QUEUED
# 会话四：开始

# 会话四：结束
127.0.0.1:6379[11](TX)> EXEC										# 一旦执行了 exec ，之前加的监控锁（watch）都被取消了
(nil)   # 执行事务，结果返回 nil，表示事务失败
127.0.0.1:6379[11]> mget balance debt
1) "99"
2) "1"

127.0.0.1:6379[11]> MULTI
OK
127.0.0.1:6379[11](TX)> incr balance
QUEUED
127.0.0.1:6379[11](TX)> decr debt
QUEUED
127.0.0.1:6379[11](TX)> EXEC # 可见 watch，只能一次性的作用于一个事务
1) (integer) 100
2) (integer) 0
```

会话四：

```shell
127.0.0.1:6379[11]> get balance
"100"
127.0.0.1:6379[11]> get debt
"0"
127.0.0.1:6379[11]> decrby balance 1
(integer) 99
127.0.0.1:6379[11]> incrby debt 1
(integer) 1
127.0.0.1:6379[11]> mget balance debt
1) "99"
2) "1"
```

得出结论：`watch`有点像`cas`锁的概念，添加了`watch`之后，只有事务提交前后被`watch`的值没有变化过，这个事务才会执行成功

>  一旦执行了 exec ，之前加的监控锁（watch）都被取消了
>
> `watch`命令，类似乐观锁，事务提交`exec`时，如果`key`的值已经被别的客户端所改变，比如某个`list`已被别的客户端（会话）`push/pop`过了，整个事务队列都不会被执行
>
> 通过`watch`命令在事务执行之前监控了多个`keys`，倘若在`watch`之后，有任何`key`的值发生了变化，`exec`命令执行的事务都将被放弃，同时返回`NullMulti-bulk`应答以通知调用者事务执行失败

### 3、事务的三个阶段

开启：用`Multi`开启一个事务

入队：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的队列里面（此时有语法检查，如果语法检查失败，那么整个事务都将失败）

执行：由`exec`命令触发事务的执行（此时如果已入队的命令都通过了语法检查，但是在执行时抛出异常，则其他命令不会回滚）

### 4、特性

>单独的隔离操作：事务中的所有命令都会被序列化、按顺序的执行。事务在执行过程中，不会被其他客户端（会话）发来的命令请求所打断
>
>没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际的执行，也就不存在“事务内的查询要看到事务内的更新，在事务外看不到”这个让人万分头疼的问题
>
>不保证原子性：`redis`同一个事务中如果有一个命令执行失败，其后面的命令仍然会执行，没有回滚





## 十一、发布订阅

`redis`的发布订阅是一种进程间的消息通信模式：发送者（`pub`）发送消息，订阅者（`sub`）接收消息
