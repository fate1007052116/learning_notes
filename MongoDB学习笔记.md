##  一、MongoDB 安装



### 1、使用docker安装

> https://registry.hub.docker.com/_/mongo-express
>
> https://registry.hub.docker.com/_/mongo

```shell
# 创建一个网卡
docker network create --driver bridge --subnet 192.168.10.0/24 --gateway 192.168.10.254 myNet

# 不设置密码
# 如果要设置账号密码，可以加入以下变量
# MONGO_INITDB_ROOT_USERNAME: root
# MONGO_INITDB_ROOT_PASSWORD: example
docker run --name mongo -p 27017:27017 -d --net myNet --ip 192.168.10.10  mongo


# 其中 mongoDB的ip地址是使用 
docker inspect mongo # 看出的，以及可以看到 mongo 使用的网卡类型位 bridge
# 连接mongoDB 时也不用设置密码
docker run --net myNet --ip 192.168.10.20 -e ME_CONFIG_MONGODB_SERVER=192.168.10.10 -e MONGO_INITDB_ROOT_USERNAME='' -e MONGO_INITDB_ROOT_PASSWORD=''  -p 8081:8081 -d --name mongo-express mongo-express
Welcome to mongo-express
```





```shell
vi /etc/yum.repos.d/mongodb-org-4.2.repo
# 输入以下内容
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
# 安装
yum install -y mongodb-org

# 网不好，直接在任意位置创建data/db 目录，db目录用来存放mongodb的数据文件
```

### 2、MongoDB中的数据类型

1. String 最常用，在mongoDB中只有UTF-8编码的字符串才是合法的
2. Integer 根据所采用的服务器可分为32位或64位
3. Boolean
4. Double
5. Min/Max keys 将一个值与Bson（二进制的Json）元素的最低值和最高值相对比
6. Array 用于将数组或列表或者多个值存储为一个键
7. Timestamp 时间戳。记录文档修改或者添加具体的时间
8. Object 用于内嵌文档，文档嵌套文档
9. Null 用于创建空值
10. Symbol 符号，该数据类型基本等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言
11. 日期时间，用Unix时间格式来存取当前日期或者时间，也可以指定自己的日期时间：创建Date对象，传入年月日信息
12. Object ID  对象ID，用于创建文档的Id
13. Binary Data 二进制数据，用于存储二进制数据
14. Code 代码类型，用于在文档中存储javascript代码
15. Regular expression 正则表达式类型，存储正则表达式

### 3、启动mongoDB

```shell
# 需要指定mongoDB的数据文件存储路径，默认会去/data/db/下寻找，但是我们没有创建这个目录，因此需要通过 
# --dbpath 来指定数据库的位置
luo@luodeMacBook-Pro bin % ./mongod --dbpath /opt/mongo/data/db
```

### 4、通过配置文件来加载启动参数

配置文件的扩展名应该为conf，撇脂文件中使用key=value 结构，在执行mongoDB时通过--config参数来指定需要加载的配置文件

### 5、常见的启动参数

```shell
--quiet 安静输出
--port 指定服务端口，默认27017
--bind 绑定服务ip，若绑定127.0.0.1，则只能本机访问
--logpath 指定mongoDB的日志文件，注意指定的是文件而不是目录
--logappend 使用追加的方式写日志
--fork 守护进程的方式运行mongoDB，创建服务器进程
--auth 启动验证
--config 指定配置文件的路径，注意不是目录
--journal 启用日志选项，mongoDB的数据操作将会写入奥journal文件夹的文件里
```

我才mongoDB目录下创建了一个conf目录，用于存放mongoDB的配置文件mongoldb.conf

配置文件中的参数不要加 --

```shell
# monfoDB.conf 配置文件的内容

# 一定要有/data/db 目录，少了db 都启动不了
dbpath=/opt/mongo/data/db
# 指定的是日志文件的名字
logpath=/opt/mongo/logs/mongoDB.log
logappend=true
# 可以改端口
port=27017
# 绑定ip（若不设置绑定的ip，默认只有127.0.0.1可以连接mongod）
bind_ip=0.0.0.0

# 启用在后台运行mongos，或者mongod进程为守护进程模式
fork=true


# 启动mongoDB
luo@luodeMacBook-Pro mongo % ./bin/mongod --config /opt/mongo/conf/mongoDB.conf
about to fork child process, waiting until server is ready for connections.
forked process: 1562
child process started successfully, parent exiting

```

### 6、关闭mongoDB 

1. Control + c 会等待当前进行中的操作完成，之后mongoDB才会退出，所以是安全的关闭方式
2. 使用kill 命令关闭：然后需要删除data/db/mongod.lock 文件，否则下次无法启动，会造成数据损坏，因此不建议使用
3. 使用mongoDB的函数关闭
   1. db.shutdownServer()
   2. db.runCommand("shutdown")
   3. 上面两个方法都需要在admin库中执行，都是安全关闭的方式

```shell
# 使用函数关闭mongoDB需要使用客户端工具(Mongo)先连接上mongoDB，切换到admin数据库，再执行函数
luo@luodeMacBook-Pro bin % /opt/mongo/bin/mongo			# 使用客户端工具连接mongoDB
MongoDB shell version v4.2.8
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("4124234f-6743-4f32-b75e-53670812e5b4") }
MongoDB server version: 4.2.8
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings:
2020-07-02T10:26:10.497+0800 I  CONTROL  [initandlisten]
2020-07-02T10:26:10.498+0800 I  CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2020-07-02T10:26:10.499+0800 I  CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2020-07-02T10:26:10.500+0800 I  CONTROL  [initandlisten]
2020-07-02T10:26:10.501+0800 I  CONTROL  [initandlisten]
2020-07-02T10:26:10.503+0800 I  CONTROL  [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
MongoDB Enterprise > use admin	# 切换到admin库
switched to db admin	
# mongoDB执行函数需要在函数之前加 db.
MongoDB Enterprise > db.shutdownServer()		# 执行关闭mongoDB函数
2020-07-02T12:02:55.593+0800 I  NETWORK  [js] DBClientConnection failed to receive message from 127.0.0.1:27017 - HostUnreachable: Connection closed by peer
server should be down...
# 此时mogoDB已经被关闭，所以客户端工具mongo连接不上mongoDB
2020-07-02T12:02:55.598+0800 I  NETWORK  [js] trying reconnect to 127.0.0.1:27017 failed
2020-07-02T12:02:55.598+0800 I  NETWORK  [js] reconnect 127.0.0.1:27017 failed failed
MongoDB Enterprise >

# 连接时指定ip
luo@luodeMacBook-Pro bin % ./mongo -host 127.0.0.1 -port 27017
```

4. 使用mongod命令来关闭mongoDB

```shell
# 4.2.8 版本已经不能这样关闭
luo@luodeMacBook-Pro bin % ./mongod --shutdown ../data/db
Error parsing command line: unrecognised option '--shutdown'
try './mongod --help' for more information
```

### 7、mongoDB 用户权限管理

1. read 		只读指定的数据库

2. readWrite 读写指定的数据库

3. dbAdmin 允许用户在指定的数据库中执行管理函数，如索引的创建、删除、查改、查看、统计、访问system.profile

4. userAdmin 允许用户向 system.user集合写入，可以找指定数据库里创建、删除、管理用户

5. clusterAdmin 只在admin数据库中可用，赋予用户所有分片、复制集相关函数的管理权限

6. readAnyDatabase 只在admin数据库中可用，赋予用户所有数据库读的权限

7. readWriteAnyDatabase 只在admin数据库中可用，赋予用户所有数据库的读写权限

8. dbAdminAnyDatabase 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限

9. userAdminAnyDatabase 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限

10. root  只在admin数据库中可用，超级账号 

### 8、mongo 用户的使用

* mogoDB有一个管理员组，这个组专门为管理普通用户而设计的，管理员通常没有对数据库的读写权限，只有操作用户的权限，因此只需要赋予管理员userAdminAnyDatabase 角色即可

* 管理员账户必须在admin数据库下创建 

### 9、创建DB管理员用户

```shell
# 先切换到admin数据库
MongoDB Enterprise > use admin
switched to db admin
# 查看admin库中的用户(users 是复数)
MongoDB Enterprise > db.system.users.find()
# 创建用户
 MongoDB Enterprise > db.createUser({user:"luo",pwd:"a1!",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
Successfully added user: {
	"user" : "luo",
	"roles" : [						# 数组的形式，可以添加多个角色
		{
			"role" : "userAdminAnyDatabase",  # 该用户的角色
			"db" : "admin"										# 该用户能操作的数据库
		}
	]
}
# 重启后生效
```

### 10、使用权限方式启动MongoDB

在默认的情况下，mongoDB是不开启用户认证的，添加用户之后，需要开启用户认证机制，在mongodb.conf下添加启动参数auth=true即可

```shell
dbpath=/opt/mongo/data/db
# 指定的是日志文件的名字
logpath=/opt/mongo/logs/mongoDB.log
# 可以改端口
port=27017
# 绑定ip
bind_ip=0.0.0.0
fork=true
logappend=true
# 开启用户认证
auth=true
```

### 11、要使用mongoDB的函数，需要登陆

```shell
# db.auth("用户名","密码") 因为luo是管理员，只能在admin库下登陆
MongoDB Enterprise > db.auth("luo","a1!")
1
```

### 12、使用用户管理员账户创建普通用户

普通账户由管理员创建，通常需要指定操作哪一个数据库

```shell
# 创建数据库
# use 命令切换数据库的时候，如果该数据库不存在，则创建该数据库
MongoDB Enterprise > use home
switched to db home
# 创建普通用户 
MongoDB Enterprise > db.createUser({user:"gao",pwd:"a1!",roles:[{role:"readWrite",db:"home"}]})
Successfully added user: {
	"user" : "gao",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "home"
		}
	]
}

# 未登陆的情况下插入数据到MongoDB的home库
MongoDB Enterprise > db.home.insert({id:"1000"})
WriteCommandError({
	"ok" : 0,
	"errmsg" : "command insert requires authentication",	# 未登陆权限不足
	"code" : 13,
	"codeName" : "Unauthorized"
})
# 使用新创建的gao用户登陆(只能在admin库下登陆)
MongoDB Enterprise > db.auth("gao","a1!")
1
MongoDB Enterprise > use home
switched to db home
MongoDB Enterprise > db.home.insert({id:"10000"})
WriteResult({ "nInserted" : 1 })

# 查询刚刚插入的数据
MongoDB Enterprise > db.home.find()
{ "_id" : ObjectId("5efdbed2d7e79364698a1405"), "id" : "10000" }
```

### 13、更新用户角色

* 如果对已经存在的角色做修改，要求当前用户需要拥有userAdminAnyDatabase 或者更高的权限
* 设置了新的角色，原来的角色会被删除
* 更新角色，若要保留原的用户的角色，需要在更新用户角色的同时再写一遍原来的角色
* 需要切换到admin库才能操作

```shell
# 在保留userAdminAnyDatabase角色的前提下，给luo用户添加readWrite的角色
MongoDB Enterprise > db.updateUser("luo",{roles:[{role:"readWrite",db:"home"},{role:"userAdminAnyDatabase",db:"admin"}]})
MongoDB Enterprise >

# 用户角色更新成功
MongoDB Enterprise > show users
{
	"_id" : "admin.gao",
	"userId" : UUID("36ca34ad-ebec-4d4f-8d80-280edba1638d"),
	"user" : "gao",
	"db" : "admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "home"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
{
	"_id" : "admin.luo",
	"userId" : UUID("391a69b2-61ed-4497-89e4-eac970eb6cc0"),
	"user" : "luo",
	"db" : "admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "home"
		},
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
```

### 14、更新用户密码

* 使用db.updateUser() 更新用户密码

  ```shell
  # 管理员才可以更改用户密码。
  # 即使是普通用户在自己的库、admin库，也不能更改自己的密码
  MongoDB Enterprise > db.updateUser("gao",{pwd:"111"})
  ```

  

* db.changeUserPassword() 这个函数只能更新密码

  ```shell
  # 管理员才可以更改普通用户的密码
  MongoDB Enterprise > db.changeUserPassword("gao","a1!")
  ```

### 15、删除用户

```shell
# 需要切换到创建用户时所指定的数据库才可以删除用户（非必须）
# 管理员才可以删除用户
MongoDB Enterprise > use admin
switched to db admin
MongoDB Enterprise > db.dropUser("gao")
true
MongoDB Enterprise > show users
{
	"_id" : "admin.luo",
	"userId" : UUID("391a69b2-61ed-4497-89e4-eac970eb6cc0"),
	"user" : "luo",
	"db" : "admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "home"
		},
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
```

### 16、创建数据库

```shell
# 如果当前数据不存在，则创建数据库之后再切换到该数据库
use home

# 查看已有数据库
# 如果开启了认证，则需要认证之后才可以查看数据库
# 管理员能看到所有用户数据库
# 用户只能看到自己能操作的数据库
# 这个命令只显示已经使用过的数据库，use dbx 只是切换，而没有具体使用过的数据库，是不会显示的，因为use dbx 只在内存中创建该数据库，没有将该库持久化到磁盘中，只有网dbx里写入数据，才会将该库持久化到磁盘中
MongoDB Enterprise > show databases
admin   0.000GB
config  0.000GB
home    0.000GB
local   0.000GB
```

### 17、删除数据库

```shell
# 需要切换到要删除的数据库
# 需要是dbAdmin，且该dbAdmin管理该数据库
MongoDB Enterprise > use home
switched to db home
# 删除当前所在的数据库
db.updateUser("yan",{roles:[{role:"readwriteAnyDatabase",db:"admin"}]})
MongoDB Enterprise > db.updateUser("luo",{roles:[{role:"dbAdmin",db:"home"},{role:"userAdminAnyDatabase",db:"admin"}]})
MongoDB Enterprise > use home
switched to db home
MongoDB Enterprise > db.dropDatabase()
{ "dropped" : "home", "ok" : 1 }

# 删除数据库之后，该数据的角色依然保留
MongoDB Enterprise > show users
{
	"_id" : "admin.gao",
	"userId" : UUID("ef327f6e-359b-48fd-bfc1-312a04b5ee52"),
	"user" : "gao",
	"db" : "admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "home"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
{
	"_id" : "admin.luo",
	"userId" : UUID("391a69b2-61ed-4497-89e4-eac970eb6cc0"),
	"user" : "luo",
	"db" : "admin",
	"roles" : [
		{
			"role" : "dbAdmin",
			"db" : "home"
		},
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}

# 赋予用户任何库的读写权限
MongoDB Enterprise > db.updateUser("gao",{roles:[{role:"readWriteAnyDatabase",db:"admin"}]})
```







## 二、mongoDB 中的集合操作

### 1、集合对应sql中的表

db.createCollection(collectionName,options)

Options:可选参数，指定内存大小及索引选项，可以是以下参数

1. Capped (boolean) 可选，如果为true，则创建固定的集合（固定大小的集合，当达到最大容量时，会自动覆盖最早的文档）

   为true时，需要指定大小size

2. autoindexid (boolean) 可选，如果为true，自动再_id字段创建索引，默认为false

3. size （数值）可选，为固定集合指定一个最大值（单位字节）

   如果capped为true，则必须指定该字段

4. max （数值）可选，指定固定集合中包含文档的最大数量

   **插入文档时，mongoDB首先检查固定集合的size字段，再检查max字段**

### 2、使用默认的集合

```shell
#mongoDB中，我们可以不用创建集合，当我们插入一些数据时，会自动创建集合并且会使用数据库的名字作为集合的名称
# 没有集合的情况下，插入数据
MongoDB Enterprise > db.home.insert({userName:"罗俊华"})
WriteResult({ "nInserted" : 1 })
# 查看插入的信息
MongoDB Enterprise > db.home.find()
{ "_id" : ObjectId("5efde45790c094b3fbc33346"), "userName" : "罗俊华" }

# 查看当前库中的集合
MongoDB Enterprise > show collections
home	# 默认的集合的名字和库的名字一样
```

* 创建不带参数的集合

  ```shell
  # 如果开启认证，则需要使用具有dbAdmin权限的用户来创建集合
  # db.createCollection("集合的名字")
  MongoDB Enterprise > db.createCollection("myCol")
  { "ok" : 1 }
  # 查看创建的集合
  MongoDB Enterprise > show collections
  home
  myCol
  ```

* 创建带参数的集合

  ```shell
  # 创建名为userCollection的集合，固定大小1024B，为_id 字段创建索引，最大文档数量1000
  MongoDB Enterprise > db.createCollection("userCollection",{capped:true,size:1024,autoIndexId:true,max:1000})
  {
  	"note" : "the autoIndexId option is deprecated and will be removed in a future release",
  	"ok" : 1
  }
  #v 查看
  MongoDB Enterprise > show collections
  home
  myCol
  userCollection
  ```

* 查看集合

```shell
show tables
show collecion
# 删除集合
# db.集合名字.drop()
MongoDB Enterprise > db.userCollection.drop()
true
```

## 三、MongoDB中的文档操作

* mongoDB 中文档时指多个key及其关联的value有序的放置在一起就是文档，其实指的就是数据

* mongoDB 中文档的数据结构和json基本一致，所以存储在集合中的数据都是bson形式（二进制的json）

```shell
# 插入数据到myCol集合中
MongoDB Enterprise > db.myCol.insert({name:"程序员",description:"喜欢敲代码",tags:["厉害","爱思考"]})
# 查看当前集合中的所有数据
MongoDB Enterprise > db.myCol.find()
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "喜欢敲代码", "tags" : [ "厉害", "爱思考" ] }

# 使用save函数插入文档
# 使用语法和insert一样，名字不一样
MongoDB Enterprise > db.myCol.save({name:"高",description:"女朋友",tag:["喜欢睡觉"]})
WriteResult({ "nInserted" : 1 })
# 查看
MongoDB Enterprise > db.myCol.find()
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "喜欢敲代码", "tags" : [ "厉害", "爱思考" ] }
{ "_id" : ObjectId("5efdebb71b467537e9fe174b"), "name" : "高", "description" : "女朋友", "tag" : [ "喜欢睡觉" ] }

# 使用insertOne() mongoDB 3.2版本之后才有
MongoDB Enterprise > db.myCol.insertOne({name:"小高高"})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5efdec4f1b467537e9fe174c")
}
#查看
MongoDB Enterprise > db.myCol.find()
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "喜欢敲代码", "tags" : [ "厉害", "爱思考" ] }
{ "_id" : ObjectId("5efdebb71b467537e9fe174b"), "name" : "高", "description" : "女朋友", "tag" : [ "喜欢睡觉" ] }
{ "_id" : ObjectId("5efdec4f1b467537e9fe174c"), "name" : "小高高" }
```

### 1、一次插入多个文档

```shell
# 向集合中批量插入多个文档时，需要使用数组来存放文档
# 只有insert() save() 函数可以批量插入 insertOne() 不可以
MongoDB Enterprise > db.myCol.insert([{name:"1"},{name:"2"},{name:"3"}])
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 3,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
# 查询
MongoDB Enterprise > db.myCol.find()
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "喜欢敲代码", "tags" : [ "厉害", "爱思考" ] }
{ "_id" : ObjectId("5efdebb71b467537e9fe174b"), "name" : "高", "description" : "女朋友", "tag" : [ "喜欢睡觉" ] }
{ "_id" : ObjectId("5efdec4f1b467537e9fe174c"), "name" : "小高高" }
{ "_id" : ObjectId("5efdefdd1b467537e9fe174d"), "name" : "1" }
{ "_id" : ObjectId("5efdefdd1b467537e9fe174e"), "name" : "2" }
{ "_id" : ObjectId("5efdefdd1b467537e9fe174f"), "name" : "3" }
MongoDB Enterprise >

# insertMany() mongoDB 3.2之后才有
MongoDB Enterprise > db.myCol.insertMany([{name:"100"},{name:"200"},{name:"300"}])
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5efdf0e21b467537e9fe1750"),
		ObjectId("5efdf0e21b467537e9fe1751"),
		ObjectId("5efdf0e21b467537e9fe1752")
	]
}

# mongoDB插入数据时，并不会因为某条数据插入失败而停止或者回滚事务，但我们可以使用try{}catch(){}来捕获异常信息
MongoDB Enterprise > try{db.home.insertMany([{id:45,name:"你好"},{id:23,name:"乌拉"}]);}catch(e){print(e);}
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5f0a8b0bffae3bffd23f4c7f"),
		ObjectId("5f0a8b0bffae3bffd23f4c80")
	]
}
```

### 2、通过变量插入文档

* Mongo shell (目前使用的mongo客户端工具)允许我们定义变量，所有变量的类型为var类型，也可以忽略变量的类型（省略var）
* 变量中赋值符号后面则需要使用小括号来表示变量中的值
* 我们可以将变量作为任意插入文档的函数的参数
* 语法 **变量名=({变量值})**
* 变量仅在本次会话中生效，重连客户端就没了
```shell
# 定义变量 (var 已省略)
MongoDB Enterprise > document=({title:"spring cloud",tags:["spring cloud","netFlix"]})
{ "title" : "spring cloud", "tags" : [ "spring cloud", "netFlix" ] }

# 将变量插入到集合
MongoDB Enterprise > db.myCol.insertOne(document)
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5efdf2f91b467537e9fe1753")
}

# 定义变量，该变量包含多个文档
MongoDB Enterprise > doc=([{title:"hello"},{title:"你好"},{title:"morning"}])
[
	{
		"title" : "hello"
	},
	{
		"title" : "你好"
	},
	{
		"title" : "morning"
	}
]
# 通过变量一次插入多个文档
MongoDB Enterprise > db.myCol.insertMany(doc)
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5efdf3df1b467537e9fe1754"),
		ObjectId("5efdf3df1b467537e9fe1755"),
		ObjectId("5efdf3df1b467537e9fe1756")
	]
}
```

### 3、更新集合中的文档

#### (1) update()

  更新已存在的文档

	语法：**db.CollectionName.update({查询条件},{更新内容},{更新参数（可选）})**

```shell
# 查询未更新前的数据
MongoDB Enterprise > db.myCol.find({name:"高"})
{ "_id" : ObjectId("5efdebb71b467537e9fe174b"), "name" : "高", "description" : "女朋友", "tag" : [ "喜欢睡觉" ] }

# 执行更新操作(原来的数据会被完全删除，此次更新中没有的数据，未来也不会有)
MongoDB Enterprise > db.myCol.update({name:"高"},{name:"你还好吗 阿燕"})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 更新之后
MongoDB Enterprise > db.myCol.find({name:"你还好吗 阿燕"})
{ "_id" : ObjectId("5efdebb71b467537e9fe174b"), "name" : "你还好吗 阿燕" }
```

#### (2) 在更新的基础上保留原来的数据

* 更新操作符
* $set 操作符：用来指定一个键并更新key的值，若key不存在，则创建
* $inc 操作符：可以对文档某个数字型的值（只能为满足要求的数字），进行增减的操作
* 语法：**db.collectionName.update({查询条件},{更新操作符:{更新内容}})**

```shell
# 更新前
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "喜欢敲代码", "tags" : [ "厉害", "爱思考" ] }

# 执行更新操作
MongoDB Enterprise > db.update({name:"程序员"},{$set:{description:"天天熬夜"}})

# 更新后
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "天天熬夜", "tags" : [ "厉害", "爱思考" ] }

# 给现有的文件添加number属性
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$set:{number:07}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 添加属性之后
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "天天熬夜", "tags" : [ "厉害", "爱思考" ], "number" : 7 }

# 实现批量更新


```

#### (3)批量更新多个文档（数据）

语法：**db.collectionName.update({查询条件},{更新操作符:{更新内容}},{multi:true})**

Multi:true 代表一次更新多个文档

```shell
# 更新前
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1757"), "name" : "luo", "size" : 100 }
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1758"), "name" : "luo", "size" : 200 }
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1759"), "name" : "luo", "size" : 300 }

# 执行更新
MongoDB Enterprise > db.myCol.update({name:"luo"},{$set:{size:199}},{multi:true})
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })

# 结果
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1757"), "name" : "luo", "size" : 199 }
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1758"), "name" : "luo", "size" : 199 }
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1759"), "name" : "luo", "size" : 199 }
```

#### (4)操作符

1. $set 操作符：用来指定一个键并更新key的值，若key不存在，则创建
2. $inc 操作符：可以对文档某个数字型的值（只能为满足要求的数字），进行增减的操作 （size:"10"）这样是无法更新

```shell
# number更新前
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "天天熬夜", "tags" : [ "厉害", "爱思考" ], "number" : 7 }

# 更新 为name="程序员" 的文件的number属性增加10 (如果是减小，则添加 - 负号即可)
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$inc:{number:10}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 结果
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "天天熬夜", "tags" : [ "厉害", "爱思考" ], "number" : 17 }
```
3. $unset 操作符： 主要用来删除key

```shell
   # 删除前
   MongoDB Enterprise > db.myCol.find({name:"程序员"})
   { "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "description" : "天天熬夜", "tags" : [ "厉害", "爱思考" ], "number" : 18 }
   
   # 删除
   # $unset:{description:"这里只是一个占位符，可以写任意，只为满足格式"}
   MongoDB Enterprise > db.myCol.update({name:"程序员"},{$unset:{description:998}})
   WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
   
   # 删除后
   MongoDB Enterprise > db.myCol.find({name:"程序员"})
   { "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "厉害", "爱思考" ], "number" : 18 }
```

4. $push 操作符：向文档的某个数组类型添加一个数组元素，不过滤重复的数据，若添加时key存在，要求key的类型必须是数组；若key不存在，则创建数组类型的key
```shell
# 向数组添加元素之前
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "厉害", "爱思考" ], "number" : 18 }

# 添加数组中的属性(文档中该key数组(tags)已经存在)
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$push:{tags:"没事喜欢敲代码"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 结果
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "厉害", "爱思考", "没事喜欢敲代码" ], "number" : 18 }

# 该key不存在，添加操作
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$push:{hobby:"爱好买电子产品"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 添加后
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "厉害", "爱思考", "没事喜欢敲代码" ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }
```

5. $pop 操作符：删除数组元素

   * 1表示从数组的尾部删除	（正数代表从前往后，从1开始）
* -1表示删除数组的第一个元素 （负数表示从后往前，从-1开始）
   * 一次只能从数组中删除一个元素

```shell
# 删除之前
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "厉害", "爱思考", "没事喜欢敲代码" ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }

# 执行删除数组中的队后一个元素，1代表删除最后一个元素
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$pop:{tags:1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 结果
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "厉害", "爱思考" ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }

# 删除数组的第一个元素 -1
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$pop:{tags:-1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
MongoDB Enterprise > db.myCol.find({name:"程序员"})

# 结果
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "爱思考" ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }
```

6. $pull 操作符：从数组中删除满足条件的元素

```shell
# 删除前
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ "爱思考" ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }

# 删除与名称相同的数组元素
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$pull:{tags:"爱思考"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 结果
MongoDB Enterprise > db.myCol.find({name:"程序员"})
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }
```

7. $pullAll 操作符：从数组中删除满足条件的多个元素

```shell
# 删除前
MongoDB Enterprise > db.myCol.find({title:"java"})
{ "_id" : ObjectId("5efe0b341b467537e9fe175a"), "title" : "java", "version" : [ "javaME", "javaSE", "javaEE" ] }

# 执行删除数组中的多个元素
MongoDB Enterprise > db.myCol.update({title:"java"},{$pullAll:{version:["javaME","javaSE"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 结果
MongoDB Enterprise > db.myCol.find({title:"java"})
{ "_id" : ObjectId("5efe0b341b467537e9fe175a"), "title" : "java", "version" : [ "javaEE" ] }
```

8. $rename 操作符：对key进行重新命名

```shell
# 更新前
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ ], "number" : 18, "hobby" : [ "爱好买电子产品" ] }

# 更新操作 update({name:"程序员"},{$rename:{旧key的名字:"新key的名字"}})
MongoDB Enterprise > db.myCol.update({name:"程序员"},{$rename:{number:"num"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 结果
{ "_id" : ObjectId("5efdeadd1b467537e9fe174a"), "name" : "程序员", "tags" : [ ], "hobby" : [ "爱好买电子产品" ], "num" : 18 }
```

   

### 4、save()函数替换文档

* save() 根据_id相同，通过传入的文档来替换已有的文档

* 语法：**save({文档})**

* 新插入的文档_id 和 原有文档的 _id 相同，才能做替换

  

```shell
# 替换前
MongoDB Enterprise > db.myCol.find({title:"java"})
{ "_id" : ObjectId("5efe0b341b467537e9fe175a"), "title" : "java", "version" : [ "javaEE" ] }

# 执行替换(必须_id完全一致才能做替换操作)
MongoDB Enterprise > db.myCol.save({"_id" : ObjectId("5efe0b341b467537e9fe175a"),content:"学java"})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

# 原有的内容被完全替换(已被删除)
MongoDB Enterprise > db.myCol.find({title:"java"})
# 替换后的内容
MongoDB Enterprise > db.myCol.find({content:"学java"})
{ "_id" : ObjectId("5efe0b341b467537e9fe175a"), "content" : "学java" }
```

### 5、删除mongoDB中的文档

1. remove()

   * 可以删除集合中指定的文档，符合条件的文档都会被删除
* 语法：**db.collectionName.remove({删除条件},{删除参数（可选）})**
   * **已经不推荐使用了，因为要清理磁盘，而且好像不可以清理磁盘了**
* 可以使用该文档的objectId作为删除条件
   * remove()不会真正的释放磁盘空间，删除之后，需要继续执行db.repairDatabase()来回收磁盘空间
   * 释放磁盘空间还需要有dbAdmid权限才能执行
   * MongoDB Enterprise > db.repairDatabase() 好像没有这个函数了

```shell
# 删除前
MongoDB Enterprise > db.myCol.find({title:"morning"})
{ "_id" : ObjectId("5efdf3df1b467537e9fe1756"), "title" : "morning" }

# 根据_id 做删除操作
MongoDB Enterprise > db.myCol.remove({"_id" : ObjectId("5efdf3df1b467537e9fe1756")})
WriteResult({ "nRemoved" : 1 })

# 删除后
MongoDB Enterprise > db.myCol.find({title:"morning"})

# 如果使用的条件在集合中可以匹配多条数据，remove()会删除所有满足条件的数据
# 删除前
MongoDB Enterprise > db.myCol.find({size:199})
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1757"), "name" : "luo", "size" : 199 }
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1758"), "name" : "luo", "size" : 199 }
{ "_id" : ObjectId("5efdfb5e1b467537e9fe1759"), "name" : "luo", "size" : 199 }

# 移除size = 199 的文档
MongoDB Enterprise > db.myCol.remove({size:199})
WriteResult({ "nRemoved" : 3 })

# 删除成功
MongoDB Enterprise > db.myCol.find({size:199})

# 可以在remove()函数中给定justOne参数，表示只删除第一条，参数无论给定什么，都只删除第一条
# 删除前
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efe83386bc564cce0175ebe"), "name" : "luo", "age" : 18 }
{ "_id" : ObjectId("5efe83386bc564cce0175ebf"), "name" : "luo", "age" : 20 }
{ "_id" : ObjectId("5efe83386bc564cce0175ec0"), "name" : "luo", "age" : 30 }

# 参数给定的是2，但仍然只删除了第一条数据
MongoDB Enterprise > db.myCol.remove({name:"luo"},{justOne:2})
WriteResult({ "nRemoved" : 1 })

# 删除第一条数据之后
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efe83386bc564cce0175ebf"), "name" : "luo", "age" : 20 }
{ "_id" : ObjectId("5efe83386bc564cce0175ec0"), "name" : "luo", "age" : 30 }
```

2. deleteOne()

   * 官方推荐，每次只删除满足条件的第一条数据
   * 语法：**db.collectionName.deleteOne({删除条件})**
   * 删除之后同时还会释放磁盘空间

```shell
# 删除前
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efe83386bc564cce0175ebf"), "name" : "luo", "age" : 20 }
{ "_id" : ObjectId("5efe83386bc564cce0175ec0"), "name" : "luo", "age" : 30 }
{ "_id" : ObjectId("5efe86b1db47f7578a39d79d"), "name" : "luo", "age" : 35 }

# 执行删除
MongoDB Enterprise > db.myCol.deleteOne({name:"luo"})
{ "acknowledged" : true, "deletedCount" : 1 }

# 删除之后
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efe83386bc564cce0175ec0"), "name" : "luo", "age" : 30 }
{ "_id" : ObjectId("5efe86b1db47f7578a39d79d"), "name" : "luo", "age" : 35 }
```

3. deleteMany()
   * 删除满足条件的所有数据
```shell
# 删除前
MongoDB Enterprise > db.myCol.find({name:"luo"})
{ "_id" : ObjectId("5efe83386bc564cce0175ec0"), "name" : "luo", "age" : 30 }
{ "_id" : ObjectId("5efe86b1db47f7578a39d79d"), "name" : "luo", "age" : 35 }
{ "_id" : ObjectId("5efe8915db47f7578a39d79e"), "name" : "luo", "age" : 20 }

# 执行删除
MongoDB Enterprise > db.myCol.deleteMany({name:"luo"})
{ "acknowledged" : true, "deletedCount" : 3 }

# 删除后
MongoDB Enterprise > db.myCol.find({name:"luo"})
```

4. 删除集合中的所有文档

```shell
# 使用deleteMany({})删除该集合中的所有文档
MongoDB Enterprise > db.myCol.find()
{ "_id" : ObjectId("5efe8a93db47f7578a39d79f"), "name" : "luo" }
{ "_id" : ObjectId("5efe8a93db47f7578a39d7a0"), "name" : "gao" }

# 执行删除
MongoDB Enterprise > db.myCol.deleteMany({})
{ "acknowledged" : true, "deletedCount" : 2 }

# 删除后
MongoDB Enterprise > db.myCol.find()

# 使用remove() 删除该集合中的所有文档（不推荐）
MongoDB Enterprise > db.myCol.remove({})
WriteResult({ "nRemoved" : 13 })
```

   

### 6、查询集合中的文档

#### 1. find()

* 语法：**find({查询条件（可选）},{指定投影的key（可选）})**
* 投影的key，在SQL中是投影的列，相当于select * 
* pretty()可以使用格式化的方式来显示所有文档

```shell
# 使用了pretty() 看起来更舒服
MongoDB Enterprise > db.myCol.find().pretty()
{
	"_id" : ObjectId("5efe922cdb47f7578a39d7a1"),
	"name" : "luo",
	"age" : 20,
	"tag" : [
		"喜欢敲代码",
		"玩电脑"
	]
}
{ "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }

# 添加查询条件
MongoDB Enterprise > db.myCol.find({name:"luo"}).pretty()
{
	"_id" : ObjectId("5efe922cdb47f7578a39d7a1"),
	"name" : "luo",
	"age" : 20,
	"tag" : [
		"喜欢敲代码",
		"玩电脑"
	]
}
```

#### 2. findOne()

* 只返回满足条件的第一条数据。如果未做投影操作，该方法则自带格式化功能 pretty()

* 语法： **findOne({查询条件（可选）},{投影操作（可选）})**

  ```shell
  # 集合中有多条文档
  MongoDB Enterprise > db.myCol.find()
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
  { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
  
  # findOne()只返回其中第一条数据，且自带pretty()格式化
  MongoDB Enterprise > db.myCol.findOne()
  {
  	"_id" : ObjectId("5efe922cdb47f7578a39d7a1"),
  	"name" : "luo",
  	"age" : 20,
  	"tag" : [
  		"喜欢敲代码",
  		"玩电脑"
  	]
  }
  
  # 条件查询结果有多条
  MongoDB Enterprise > db.myCol.find({name:"luo"})
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
  
  # 但findOne()仍然只返回一条数据
  MongoDB Enterprise > db.myCol.findOne({name:"luo"})
  {
  	"_id" : ObjectId("5efe922cdb47f7578a39d7a1"),
  	"name" : "luo",
  	"age" : 20,
  	"tag" : [
  		"喜欢敲代码",
  		"玩电脑"
  	]
  }
  ```

  

#### 3. 模糊查询

* 在mongoDB中可以通过 // 与 ^$实现模糊查询

* 注意使用模糊查询时查询条件不能放到双引号或者单引号中

  ```shell
  # 模糊查询name中含有o的数据
  MongoDB Enterprise > db.myCol.find({name:/o/})
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
  
  # 使用正则表达式(以ao结尾)
  # ^ 锚定行首
  # $ 锚定行尾
  MongoDB Enterprise > db.myCol.find({name:/ao$/})
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
  { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
  ```

#### 4. 投影操作

* 在find()函数中我们可以指定需要投影的key

* 语法：**find({查询条件},{投影的key名字:1（显示该列）｜0（隐藏该列）})**

* _id 默认都会显示

  ```shell
  # 两个key都隐藏
  MongoDB Enterprise > db.myCol.find({name:"luo"},{name:0,age:0})
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5") }
  
  #  两个key都显示
  MongoDB Enterprise > db.myCol.find({name:"luo"},{name:1,age:1})
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20 }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
  
  # 像select key 一样只投影需要显示的列
  MongoDB Enterprise > db.myCol.find({name:"luo"},{name:1})
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo" }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo" }
  
  # 一个显示，一个隐藏是不允许的
  MongoDB Enterprise > db.myCol.find({name:"luo"},{name:1,age:0})
  Error: error: {
  	"ok" : 0,
  	"errmsg" : "Projection cannot have a mix of inclusion and exclusion.",
  	"code" : 2,
  	"codeName" : "BadValue"
  }
  
  # 隐藏id
  MongoDB Enterprise > db.myCol.find({},{_id:0})
  { "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "name" : "gao", "age" : 18 }
  { "name" : "高高", "age" : 18 }
  { "name" : "luo", "age" : 40 }
  { "name" : "hao", "age" : 25 }
  
  # findOne()也可以做投影
  MongoDB Enterprise > db.myCol.findOne({},{_id:0})
  { "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  ```

#### 5. Find()条件操作符

* 条件操作符用于比较两个表达式并从mongoDB集合中获取数据

* 语法：**find({key:{操作符:条件}})**

* 语法：**findOne({key:{操作符:条件}})**

  1. $gt 大于操作符(greater than)

     可以判断数字或者日期

     ```shell
     # 全部数据
     MongoDB Enterprise > db.myCol.find()
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
     
     # 查询age>20的文档
     MongoDB Enterprise > db.myCol.find({age:{$gt:20}})
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
     ```

     

  2. $lt 小于操作符 (less than)

     ```shell
     # 查询age<25的文档
     MongoDB Enterprise > db.myCol.find({age:{$lt:25}})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
     ```

     

  3. $gte 大于等于操作符 (greater than equals)

     ```shell
     # age>=25
     MongoDB Enterprise > db.myCol.find({age:{$gte:25}})
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
     ```

     

  4. $lte 小于等于 (less than equals)

     ```shell
     # age <= 18
     MongoDB Enterprise > db.myCol.find({age:{$lte:18}})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
     ```

     

  5. $eq 等于 (equals)

     ```shell
     # age = 20 
     MongoDB Enterprise > db.myCol.find({age:{$eq:20}})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
     ```

     

  6. $ne 不等于 (no equals)

     ```shell
     # age != 25
     MongoDB Enterprise > db.myCol.find({age:{$ne:25}})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     ```

     

  7. $and 与逻辑

     语法：**find({$and:[{条件一},{条件二},{条件三}]})**

     ```shell
     # age = 40 && name = "luo"
     MongoDB Enterprise > db.myCol.find({$and:[{name:"luo"},{age:40}]})
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     
     # 如果在查询中给定了多个查询条件，条件之间默认的关系就是and关系
     # 只适用于对同一属性（age）的不同条件（>= <=）判断时有效
     # age >=20 && age <=25
     MongoDB Enterprise > db.myCol.find({age:{$gte:20,$lte:25}})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
     { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
     ```

     

  8. $.or 

     语法：**find({$or:[{条件一},{条件二},{条件三}]})**

     ```shell
     # name = "gao" || age > 25
     MongoDB Enterprise > db.myCol.find({$or:[{name:{$eq:"gao"}},{age:{$gt:25}}]})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     
     # 默认是等于逻辑，所以eq可以省略
     MongoDB Enterprise > db.myCol.find({$or:[{name:"gao"},{age:25}]})
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
     
     # 这样会报错
     MongoDB Enterprise > db.myCol.find({$or:[{$eq:{name:"gao"}},{$eq:{age:25}}]})
     Error: error: {
     	"ok" : 0,
     	"errmsg" : "unknown top level operator: $eq",
     	"code" : 2,
     	"codeName" : "BadValue"
     }
     ```

     

  9. $and $or 联合使用

     ```shell
     # name = "高高" && age = 18 || age > 25
     MongoDB Enterprise > db.myCol.find({$or:[{$and:[{name:"高高"},{age:{$eq:18}}]},{age:{$gt:25}}]})
     { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     ```

     

  10. $type 操作符

      基于bson类型来检索集合中匹配的数据类型，并返回结果
	   ```shell
    # 使用typeof 关键字查看数据类型
      MongoDB Enterprise > typeof 123
      number
      MongoDB Enterprise > typeof "你好"
      string
      
      # 查询键 name 的类型是数字类型的文档
      MongoDB Enterprise > db.myCol.find({name:{$type:"number"}})
      { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
     ```
  
### 7、Limit函数与Skip函数

#### 1. limit() 限制查询到的文档的条数

    语法：**db.collectionName.find().limit(要读取文件的个数)**

```shell
# 全部数据
MongoDB Enterprise > db.myCol.find()
{ "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
{ "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
{ "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
{ "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
{ "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
{ "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }


# 只要查询到的结果的前两条数据
MongoDB Enterprise > db.myCol.find().limit(2)
{ "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
{ "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
```

#### 2. 跳过指定数量的数据

    语法：**db.collectionName.find().skip(要跳过数据的条数)**

```shell
# 跳过查询结果的前三条数据
MongoDB Enterprise > db.myCol.find().skip(3)
{ "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
{ "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
{ "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
```
 3. skip() 结合 limit() 达到分页查询的效果 （不推荐）

    因为会扫描全部文档后再返回结果，效率低

```shell
#  skip() limit()不分前后
MongoDB Enterprise > db.myCol.find().skip(3).limit(2)
{ "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
{ "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
```

### 8、MongoDB中的排序

* sort() 函数可以通过参数来指定要排序的字段

* 并用 1 、-1来指定排序的方式（其中1为升序，-1为降序）

* 语法：**db.collectionName.find().sort({排序的key:1})**

  ```shell
  # 根据年龄做升序排列
  MongoDB Enterprise > db.myCol.find().sort({age:1})
  { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
  { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
  
  # 降序
  MongoDB Enterprise > db.myCol.find().sort({age:-1})
  { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
  { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
  { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
  { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18 }
  { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
  ```

  

### 四、MongoDB 中的日期操作

#### 1、日期处理

```shell
# 日期是object类型的
MongoDB Enterprise > typeof new Date()
object

# 插入日期
MongoDB Enterprise > db.home.insertOne({name:"luo",date:new Date()})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5f049f600cbe129b81ec9134")
}

# 查询
# mongoDB中的时间会比当前系统时间少8小时，因为使用的是UTC时间，中国东八区，比UTC快8小时，所以会比当前系统时间少8小时
MongoDB Enterprise > db.home.find()
{ "_id" : ObjectId("5f049f600cbe129b81ec9134"), "name" : "luo", "date" : ISODate("2020-07-07T16:14:24.877Z") }

# 插入日期的时候通过构造器指定日期
# 2020-9-26T10:12:35Z 日期需要带T Z 否则插入的不是标准的日期格式
MongoDB Enterprise > db.home.insertOne({name:"gao",date:new Date("2020-9-26T10:12:35Z")})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5f04a2490cbe129b81ec9135")
}

# 使用函数插入日期（对日期的要求比较低，可以省略TZ，年月日之间甚至可以不要连字符）
# 位数不够的用0补足
MongoDB Enterprise > typeof ISODate
function
MongoDB Enterprise > db.home.insertOne({name:"high",date:ISODate("2010-01-02T14:08:04Z")})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5f04a2fd0cbe129b81ec9136")
}

# 松散的插入日期
MongoDB Enterprise > db.home.insertOne({name:"iso",date:ISODate("19990208 124523")})
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5f04a3bb0cbe129b81ec9137")
}
# 松散插入数据正常
{ "_id" : ObjectId("5f04a3bb0cbe129b81ec9137"), "name" : "iso", "date" : ISODate("1999-02-08T12:45:23Z") }

# 使用日期作为查询条件
MongoDB Enterprise > db.home.find({date:{$eq:ISODate("19990208 124523")}})
{ "_id" : ObjectId("5f04a3bb0cbe129b81ec9137"), "name" : "iso", "date" : ISODate("1999-02-08T12:45:23Z") }

# 非等式查询（不给定时分秒，则用00:00:00计数）
MongoDB Enterprise > db.home.find({date:{$gt:ISODate("19990208")}})
{ "_id" : ObjectId("5f049f600cbe129b81ec9134"), "name" : "luo", "date" : ISODate("2020-07-07T16:14:24.877Z") }
{ "_id" : ObjectId("5f04a2490cbe129b81ec9135"), "name" : "gao", "date" : ISODate("2020-09-26T10:12:35Z") }
{ "_id" : ObjectId("5f04a2fd0cbe129b81ec9136"), "name" : "high", "date" : ISODate("2010-01-02T14:08:04Z") }
{ "_id" : ObjectId("5f04a3bb0cbe129b81ec9137"), "name" : "iso", "date" : ISODate("1999-02-08T12:45:23Z") }
```

#### 2、使用聚合取出部分日期

```shell
# 取出 年、月、日
MongoDB Enterprise > db.home.aggregate([{$project:{年份:{$year:"$date"},月份:{$month:"$date"},日:{$dayOfMonth:"$date"},_id:0}}])
{ "年份" : 2020, "月份" : 7, "日" : 7 }
{ "年份" : 2020, "月份" : 9, "日" : 26 }
{ "年份" : 2010, "月份" : 1, "日" : 2 }
{ "年份" : 1999, "月份" : 2, "日" : 8 }

# 取出 时、分、秒、毫秒
MongoDB Enterprise > db.home.aggregate([{$project:{_id:0,时:{$hour:"$date"},分:{$minute:"$date"},秒:{$second:"$date"},毫秒:{$millisecond:"$date"}}}])
{ "时" : 16, "分" : 14, "秒" : 24, "毫秒" : 877 }
{ "时" : 10, "分" : 12, "秒" : 35, "毫秒" : 0 }
{ "时" : 14, "分" : 8, "秒" : 4, "毫秒" : 0 }
{ "时" : 12, "分" : 45, "秒" : 23, "毫秒" : 0 }

# 显示星期、全年的第几周、全年中的第几天
# $dayOfWeek 星期天为1，星期六为7
# $week 全年第几周，从0开始
MongoDB Enterprise > db.home.aggregate([{$project:{_id:0,星期:{$dayOfWeek:"$date"},全年第几周:{$week:"$date"},全年的第几天:{$dayOfYear:"$date"}}}])
{ "星期" : 3, "全年第几周" : 27, "全年的第几天" : 189 }
{ "星期" : 7, "全年第几周" : 38, "全年的第几天" : 270 }
{ "星期" : 7, "全年第几周" : 0, "全年的第几天" : 2 }
{ "星期" : 2, "全年第几周" : 6, "全年的第几天" : 39 }
```

#### 3、自定义日期格式

```shell
MongoDB Enterprise > db.home.aggregate([{$project:{自定义日期格式:{$dateToString:{format:"%Y年%m月%d日 %H-%M-%S",date:"$date"}},_id:0,name:1}}])
{ "name" : "luo", "自定义日期格式" : "2020年07月07日 16-14-24" }
{ "name" : "gao", "自定义日期格式" : "2020年09月26日 10-12-35" }
{ "name" : "high", "自定义日期格式" : "2010年01月02日 14-08-04" }
{ "name" : "iso", "自定义日期格式" : "1999年02月08日 12-45-23" }
MongoDB Enterprise >
```

### 五、java无法连接MongoDB

```shell
MongoDB Enterprise >  db.system.version.find()
{ "_id" : "featureCompatibilityVersion", "version" : "4.2" }
{ "_id" : "authSchema", "currentVersion" : 5 }
MongoDB Enterprise > var schema = db.system.version.findOne({"_id" : "authSchema"})
MongoDB Enterprise > schema.currentVersion = 3
3
# 重启
```











### 六、MongoDB中的索引

* mongoDB中会自动为文档中的_id（文档中的主键）创建索引，与关系型数据库的主键索引类似

* createIndex() 函数可以为其他的key创建索引

* 创建索引的时候需要指定排序规则（1为升序，-1降序创建索引）

* dbAdmin才可以创建索引

* 语法：**db.collectionName.createIndex({创建索引的key:排序规则,...},{创建索引的参数（可选参数）})**

* 参数说明

  1. Background （boolean，默认false）后台建立索引，以便创建索引时不阻止其他数据库活动

     不给定该参数的时候，mongoDB在创建索引的时候，会阻塞进程来创建索引，阻塞期间所有对于mongoDB的请求都无法进行

  2. unique （boolean，默认false） 创建唯一索引

     保证索引对应的key不会出现相同的值（比如_id索引就是唯一索引）

     db.collectionName.createIndex({索引key:排序规则},{unique:true})

     **如果唯一索引所在的字段有重复的数据写入或者这个字段已经包含重复的数据时，抛出异常**

     ```shell
      MongoDB Enterprise > db.myCol.createIndex({name:1},{unique:true})
     {
  
   	"createdCollectionAutomatically" : false,
   	"numIndexesBefore" : 1,
   	"numIndexesAfter" : 2,
   	"ok" : 1
     }
  
     # 查看唯一索引
     MongoDB Enterprise > db.myCol.getIndexes()
     [
     	{
     		"v" : 2,
     		"key" : {
     			"_id" : 1
     		},
     		"name" : "_id_",
     		"ns" : "home.myCol"
     	},
     	{
     		"v" : 2,
     		"unique" : true,	# 唯一索引
     		"key" : {
     			"name" : 1
     		},
     		"name" : "name_1",
     		"ns" : "home.myCol"
     	}
     ]
     ```
  
     
  
  3. name（String）指定索引的名称，如果未指定mongoDB会生成一个索引字段的名称和排序顺序串联
  
  4. partialFilterExpression（数据类型：document）如果指定mongoDB指挥满足过滤表达式的记录
  
     * 只对符合条件的文档创建索引（3.2版本支持）
  
     * MongoDB的部分索引只为哪些再一个集合中，满足指定的筛选条件的文档创建索引
  
     * 由于部分索引是一个集合文档的一个子集，因此部分索引具有较低的存储需求
  
     * 并降低了索引创建和维护的性能成本
  
     * 部分索引通过过滤条件来创建，可以为mongoDB支持的所有索引类型创建部分索引
  
     * 语法：**db.collectionName.createIndex({索引名:排序规则},{partialFilterExpression:{{key名:{匹配条件:条件值}}}})**
  
     * 查询时，当查询条件满足部分索引的创建条件时，才会走查询索引
  
     * 如果指定partialFilterExpression 和 唯一约束(unique)，那么唯一约束只适用于满足筛选条件的文档，具有唯一约束的部分索引不会阻止不符合唯一约束且不符合过滤条件的文档的插入。也就是说，此时的唯一约束只对满足了部分索引的文档生效
  
       ```shell
       # 为年纪大于25的文档的age创建索引
       MongoDB Enterprise > db.myCol.createIndex({age:1},{partialFilterExpression:{age:{$gt:25}}})
       {
       	"createdCollectionAutomatically" : false,
       	"numIndexesBefore" : 2,
       	"numIndexesAfter" : 3,
       	"ok" : 1
       }
       
       # 查看部分索引
       MongoDB Enterprise > db.myCol.getIndexes()
       [
       	{
       		"v" : 2,
       		"key" : {
       			"_id" : 1
       		},
       		"name" : "_id_",
       		"ns" : "home.myCol"
       	},
       	{
       		"v" : 2,
       		"key" : {
       			"age" : 1
       		},
       		"name" : "age_1",
       		"ns" : "home.myCol",
       		"partialFilterExpression" : {	#部分索引
       			"age" : {
       				"$gt" : 25
       			}
       		}
       	}
       ]
     ```
  
  ​     
  
  5. sparse （boolean，默认false）对文档中不存在的字段数据不启用索引（稀疏索引）
  
  6. expireAfterSeconds (Integer) 指定索引的过期时间
  
  7. storageEngine    document类型允许用户配置索引的额外存储引擎

```shell
# 后台为name创建索引 
MongoDB Enterprise > db.myCol.createIndex({name:1},{background:true})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}

# 查看索引
MongoDB Enterprise > db.myCol.getIndexes()
[
	{
		"v" : 2,
		"key" : {		# 主键自带索引
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "home.myCol"		# 当前索引所在集合的路径(库.集合)
	},
	{
		"v" : 2,
		"key" : {
			"name" : 1	# 自定义创建的索引 1代表升序
		},
		"name" : "name_1",	# 索引的名字
		"ns" : "home.myCol", # 当前索引所在集合的路径(库.集合)
		"background" : true
	}
]

# 第二种查看索引的方式
MongoDB Enterprise > db.myCol.getIndexSpecs()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "home.myCol"
	},
	{
		"v" : 2,
		"key" : {
			"name" : 1
		},
		"name" : "name_1",
		"ns" : "home.myCol",
		"background" : true
	}
]

# 查看集合中的索引key(有索引的key)
MongoDB Enterprise > db.myCol.getIndexKeys()
[ { "_id" : 1 }, { "name" : 1 } ]

# 查看索引的大小（单位：字节）
# db.collectionName.totalIndexSize(detail（可选参数，默认false）)
# 如果detail是0或者false以外的数据：那么会显示该集合中每个索引的大小以及集合中索引的总大小
# 如果detail是0活着false：只显示该集合中所有索引的总大小
MongoDB Enterprise >  db.myCol.totalIndexSize()
57344

# 显示每个索引的大小
MongoDB Enterprise > db.myCol.totalIndexSize(true)
_id_	36864
name_1	20480
57344
```

#### 1、修改、删除索引

```shell
# mongoDB没有单独的修改索引的函数，如果要修改某个索引，需要先删除旧的索引，再创建新的索引

# 查看集合中索引的名称
MongoDB Enterprise > db.myCol.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "home.myCol"
	},
	{
		"v" : 2,
		"key" : {
			"name" : 1
		},
		"name" : "name_1", # 索引的名称
		"ns" : "home.myCol",
		"background" : true
	}
]


# 删除集合中的指定索引
MongoDB Enterprise > db.myCol.dropIndex("name_1")
{ "nIndexesWas" : 2, "ok" : 1 }

# 删除全部索引 _id 索引不会被删除
MongoDB Enterprise > db.myCol.dropIndexes()
{
	"nIndexesWas" : 1,
	"msg" : "non-_id indexes dropped for collection",
	"ok" : 1
}
```

#### 2、重建索引

* 重建索引可以减少索引的存储空间，减少索引碎片，优化索引查询效率

* 一般在数据大量变化之后，使用重建索引来提升索引的性能

* 重建索引是删除原索引，再重新创建的过程，不建议反复使用

* 该集合中的所有所有索引都会被重建

  ```shell
  MongoDB Enterprise > db.myCol.reIndex()
  {
  	"nIndexesWas" : 1,
  	"nIndexes" : 1,
  	"indexes" : [
  		{
  			"v" : 2,
  			"key" : {
  				"_id" : 1
  			},
  			"name" : "_id_",
  			"ns" : "home.myCol"
  		}
  	],
  	"ok" : 1
  }
  ```

  

#### 3、索引类型

* MongoDB支持多种索引的类型，每中类型的索引有不用的使用场合
1. 单字段索引（single field index）

   索引中只包含了一个key，查询时，可以加速对该字段的各种查询请求

   mongoDB默认的 _id 的索引也是这种类型的

   **db.collectionName.createIndex({索引key:排序规则}) 来创建单字段索引 **

   ```shell
   MongoDB Enterprise > db.myCol.createIndex({name:-1})
   {
   	"createdCollectionAutomatically" : false,
   	"numIndexesBefore" : 1,
   	"numIndexesAfter" : 2,
   	"ok" : 1
   }
   ```

   

2. 交叉索引

   * 为一个集合的多个字段分别建立索引，在查询的时候通过多个字段作为查询条件
   * 在查询文档时，若查询条件中包含一个交叉索引key或者在一次查询中使用了多个交叉索引key作为查询条件都会触发交叉索引
   * 两个或两个以上的索引，他们之间就是交叉索引

   ```shell
   # 再添加一个单字段索引
   MongoDB Enterprise > db.myCol.createIndex({age:1})
   {
   	"createdCollectionAutomatically" : false,
   	"numIndexesBefore" : 2,
   	"numIndexesAfter" : 3,
   	"ok" : 1
   }
   
   # 多个单字段索引自动组合为交叉索引
   MongoDB Enterprise > db.myCol.getIndexes()
   [
   	{
   		"v" : 2,
   		"key" : {
   			"_id" : 1
   		},
   		"name" : "_id_",
   		"ns" : "home.myCol"
   	},
   	{
   		"v" : 2,
   		"key" : {
   			"name" : -1
   		},
   		"name" : "name_-1",	# 单字段索引
   		"ns" : "home.myCol"
   	},
   	{
   		"v" : 2,
   		"key" : {
   			"age" : 1
   		},
   		"name" : "age_1", # 单字段索引
   		"ns" : "home.myCol"
   	}
   ]
   ```

   

3. 复合索引

   * 复合索引是单字段索引的升级版，针对多个字段联合创建索引

   * 先按照一个字段排序，第一个字段相同的文档，再按照第二个字段排序，以此类推

   * 语法：**db.collectionName.createIndex({第一个索引key:排序规则,第二个索引key:排序规则})**

   * 复合索引不光满足多字段组合起来的查询，也能满足所有能匹配符号索引前缀的查询

     ```shell
     # 默认按照name的升序排序，当name相同时，再按照age的降序排序
     MongoDB Enterprise > db.myCol.createIndex({name:1,age:-1})
     {
     	"createdCollectionAutomatically" : false,
     	"numIndexesBefore" : 1,
     	"numIndexesAfter" : 2,
     	"ok" : 1
     }
     # 复合索引 name.age
     # 查询 name 触发复合索引
     # 查询 name.age 触发复合索引
     # 查询 age 不会触发复合索引
     
     # 查看复合索引 
     MongoDB Enterprise > db.myCol.getIndexes()
     [
     	{
     		"v" : 2,
     		"key" : {
     			"_id" : 1
     		},
     		"name" : "_id_",
     		"ns" : "home.myCol"
     	},
     	{
     		"v" : 2,
     		"key" : {
     			"name" : 1,  # 复合索引包含两个key，name升序
     			"age" : -1   # age降序
     		},
     		"name" : "name_1_age_-1",
     		"ns" : "home.myCol"
     	}
     ]
     ```

     

4. 多key索引

   * 当索引的字段为数组时，创建出来的索引就是多key索引

   * 多key索引会为数组的每一个元素建立一条索引

   * 如果指定的key不是数组，创建的索引就是单字段索引

   * **db.collectionName.createIndex({数组key名:排序规则})**

     ```shell
     MongoDB Enterprise > db.myCol.find()
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
     { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
     { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : [ "好看", "好玩" ] }
     { "_id" : ObjectId("5efe94a0db47f7578a39d7a5"), "name" : "luo", "age" : 40 }
     { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
     { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
     
     # 为数组tag 创建索引
     MongoDB Enterprise > db.myCol.createIndex({tag:1})
     {
     	"createdCollectionAutomatically" : false,
     	"numIndexesBefore" : 1,
     	"numIndexesAfter" : 2,
     	"ok" : 1
     }
     # 查看索引
     MongoDB Enterprise > db.myCol.getIndexes()
     [
     	{
     		"v" : 2,
     		"key" : {
     			"_id" : 1
     		},
     		"name" : "_id_",
     		"ns" : "home.myCol"
     	},
     	{
     		"v" : 2,
     		"key" : {
     			"tag" : 1
     		},
     		"name" : "tag_1",
     		"ns" : "home.myCol"
     	}
     ]
     ```

     

5. 覆盖索引查询

   覆盖查询是以下的查询

   * 所有的查询字段是索引的一部分
   * 所有的查询返回字段在同一个索引中
   * 也就时说，查询的条件和查询所返回的字段全部来源于索引

   * 由于所有出现在查询中的字段是索引的一部分，MongoDB无需在整个数据文档中检索匹配查询条件和返回使用相同索引和查询结果

   * 因为索引存在于RAM中，从索引中获取数据比通过扫描文档读取数据要快得多

   * 对于复合索引查询，MongoDB不会去数据库文件中查找。相反，他会从索引中提取数据，这是非常快的数据查询

   * 由于我们的索引中不包含_id 字段， _id 在查询中会默认返回，我们可以在mongoDB中排除它，因为在索引中找到 _id 之后，还会去磁盘中根据 _id 去查找数据，投影的时候排除 _id ，在索引中查询到数据之后就能立刻返回数据，不用再去磁盘中查找，速度极快

   ```shell
   # 创建复合索引
   MongoDB Enterprise > db.myCol.createIndex({name:1,age:1})
   {
   	"createdCollectionAutomatically" : false,
   	"numIndexesBefore" : 1,
   	"numIndexesAfter" : 2,
   	"ok" : 1
   }
   
   # 覆盖索引查询
   # 查询条件是name（索引），返回结果_id name age 也都是索引，所以是复合索引查询
   # 想要隐藏_id 且只显示name、和age，必须是要隐藏的key在前，显示的key在后，否则异常
   MongoDB Enterprise > db.myCol.find({name:"luo"},{_id:0,name:1,age:1})
   { "name" : "luo", "age" : 20 }
   ```

   

#### 4、查询计划

* 在MongoDB中通过explain()函数启动执行计划，我们可以通过查询计划分析索引的使用情况，可以通过查看详细的查询计划来决定如何优化

* 语法：**db.collectionName.find().explain()**

* 删除索引，方便查看查询计划

  ```shell
  # 没有索引的情况下，查看查询计划
  MongoDB Enterprise > db.myCol.find({age:{$gt:25}}).explain()
  {
  	"queryPlanner" : {				# 查询信息
  		"plannerVersion" : 1,
  		"namespace" : "home.myCol",
  		"indexFilterSet" : false,
  		"parsedQuery" : {
  			"age" : {			# 查询条件
  				"$gt" : 25
  			}
  		},
  		"queryHash" : "4BB283C4",
  		"planCacheKey" : "4BB283C4",
  		"winningPlan" : {			# 查询过程
  			"stage" : "COLLSCAN",		# collscan 全集合的扫描，对应SQL的全表扫描，没有走索引
  			"filter" : {				# 过滤数据的条件
  				"age" : {
  					"$gt" : 25
  				}
  			},
  			"direction" : "forward"
  		},
  		"rejectedPlans" : [ ]
  	},
  	"serverInfo" : {
  		"host" : "luodeMacBook-Pro.local",
  		"port" : 27017,
  		"version" : "4.2.8",
  		"gitVersion" : "43d25964249164d76d5e04dd6cf38f6111e21f5f"
  	},
  	"ok" : 1
  }
  
  # 为age创建索引
  MongoDB Enterprise > db.myCol.createIndex({age:1},{background:true})
  {
  	"createdCollectionAutomatically" : false,
  	"numIndexesBefore" : 1,
  	"numIndexesAfter" : 2,
  	"ok" : 1
  }
  
  # 有了索引之后，再查询
  MongoDB Enterprise > db.myCol.find({age:{$gt:25}}).explain()
  {
  	"queryPlanner" : {
  		"plannerVersion" : 1,
  		"namespace" : "home.myCol",
  		"indexFilterSet" : false,
  		"parsedQuery" : {
  			"age" : {
  				"$gt" : 25
  			}
  		},
  		"queryHash" : "4BB283C4",
  		"planCacheKey" : "DF7FEF1F",
  		"winningPlan" : {
  			"stage" : "FETCH",
  			"inputStage" : {
  				"stage" : "IXSCAN",	#此时走的就是索引查询
  				"keyPattern" : {
  					"age" : 1
  				},
  				"indexName" : "age_1",
  				"isMultiKey" : false,
  				"multiKeyPaths" : {
  					"age" : [ ]
  				},
  				"isUnique" : false,
  				"isSparse" : false,
  				"isPartial" : false,
  				"indexVersion" : 2,
  				"direction" : "forward",
  				"indexBounds" : {
  					"age" : [
  						"(25.0, inf.0]"
  					]
  				}
  			}
  		},
  		"rejectedPlans" : [ ]
  	},
  	"serverInfo" : {
  		"host" : "luodeMacBook-Pro.local",
  		"port" : 27017,
  		"version" : "4.2.8",
  		"gitVersion" : "43d25964249164d76d5e04dd6cf38f6111e21f5f"
  	},
  	"ok" : 1
  }
  ```

#### 5、使用索引的注意事项

  * 索引虽然加快了查询速度，但是也是有代价的

  * 索引文件本身也要消耗存储空间

  * 同时还会加重插入、删除、修改记录时的负担（数据的修改，索引也要随之修改）

  * 数据库在运行时也需要消耗资源来维护索引

    
#### 6、不建议创建索引的情况

* 几千条以下的数据（2000条作为分界线）

如果复合索引能用的话，复合索引的效率>交叉索引的效率

#### 7、复合索引的字段排列顺序

当复合索引的内容包含匹配条件以及范围条件的时候，比如用户名（匹配条件）和年龄（范围条件），那么匹配条件应该放在范围条件之前

#### 8、索引的限制

* 索引有额外的开销，更新数据时，索引也需要一并更新
* 内存限制：索引存储在RAM中，如果索引大小超过内存的限制，MongoDB会删除一些索引，导致性能下降
* 查询限制：索引不能用于以下查询的使用
  * 正则表达式查询（行首锚定除外^）及非操作符，$nin,$not等
  * 算数运算符 如  $mod
  * **检测查询语句是否支持索引是很重要的，用.find().explain()**
* 最大范围
  * 集合中的索引不能超过64个
  * 索引名的长度不能超过128个字符
  * 一个复合索引最多可以有31个字段

## 七、MongoDB中的正则表达式查询

* 语法一：**db.collectionName.find({字段名:正则表达式})**

* 语法二：**db.collectionName.find({字段名:{$regex:正则表达式,$options:{正则选项}}})**

* 正则选项:

   1. i：不区分大小写
2. m：多行查找，如果选项里不存在换行符号（例如：/n）或者条件上没有(start,end)，该选项没有任何效果
   3. x：非转译的空白字符将被忽略。需要$regex与$options语法
4. s：允许点字符，（即.）匹配包含换行符在内的所有字符。 需要$regex与$options语法
   5. i,m,x,s可以组合使用
6. 参数需要用""括起,$options:"imxs"
   
```shell
   # 查询l开头
MongoDB Enterprise > db.myCol.find({name:/^l/})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   
   # 方式二（全写）
   MongoDB Enterprise > db.myCol.find({name:{$regex:/^l/}})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   
   # 查询以uo结尾
   MongoDB Enterprise > db.myCol.find({name:/uo$/})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   
   # 方式二 （全写）
   MongoDB Enterprise > db.myCol.find({name:{$regex:/uo$/}})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   
   # 字段包含ao
   MongoDB Enterprise > db.myCol.find({name:/ao/})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
   { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
   
   # 使用参数
   # i 忽略大小写
   MongoDB Enterprise > db.myCol.find({name:/luo/i})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   { "_id" : ObjectId("5f04697c0cbe129b81ec9132"), "name" : "LuoJun" }
   
   # 使用参数（全写）
   # 忽略大小写
   MongoDB Enterprise > db.myCol.find({name:{$regex:/luo/,$options:"i"}})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   { "_id" : ObjectId("5f04697c0cbe129b81ec9132"), "name" : "LuoJun" }
   
   # 以l开头且以o结尾
   MongoDB Enterprise > db.myCol.find({name:/^l.*o$/})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   
   # 查询以l或者g开头
   MongoDB Enterprise > db.myCol.find({name:{$in:[/^l/,/^g/]}})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
   
   # 不以l开头的数据
   MongoDB Enterprise > db.myCol.find({name:{$not:/^l/}})
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : [ "好看", "好玩" ] }
   { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
   { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
   { "_id" : ObjectId("5f04697c0cbe129b81ec9132"), "name" : "LuoJun" }
   
   # 不以l或者g开头的数据
   MongoDB Enterprise > db.myCol.find({name:{$nin:[/^l/,/^g/]}})
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : [ "好看", "好玩" ] }
   { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
   { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
   { "_id" : ObjectId("5f04697c0cbe129b81ec9132"), "name" : "LuoJun" }
```

#### 1、MongoDB中的聚合查询

* 使用aggregate()函数完成聚合查询

* aggregate() 主要用于统计、平均值、求和等，返回计算后的数据结果

* 语法：**db.collectionName.aggregate([{$group:{_id:"$分组键名","$分组键名",...,别名:{聚合运算:"$运算列"}}},{条件筛选:{键名:{运算条件:运算值}}}])**

  * $group 分组处理


| SQL操作/函数 |              MongoDB聚合操作:              |
| :----------: | :----------------------------------------: |
|    where     | $match（$match在$group前，就相当于where）  |
|   group by   |                   $group                   |
|    having    | $match（$match在$group后，就相当于having） |
|    select    |                  $project                  |
|   order by   |                   $sort                    |
|    limit     |                   $limit                   |
|    sum()     |                    $sum                    |
|   count()    |                    $sum                    |
|     join     |         $lookup（mongoDB3.2新增）          |

1. $sum 求和，查询集合中一共有多少个文档

   ```shell
   # 相当于 select count() as count from myCol
   # $group 分组，代表聚合的分组条件（聚合查询中都必须有$group）
   # _id:null 对_id 字段不做分组处理，所有文档在同一个大组中
   #		分组的字段，相当于 group by dept 中的dept部分
   #		如果根据某字段的值进行分组，则定义_id:"$字段名"，所以此案例中的null代表一个固定的字面值null
   # $sum:1 每查询到一条数据做1的递增，相当于 SQL中的sum()
   # count:{$sum:1} 返回结果的字段名，将$sum统计的数据的列名命名为count，相当于select count() as count 中的count
   MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:null,count:{$sum:1}}}])
   { "_id" : null, "count" : 6 }
   
   # 将所有文档中的age累加求和
   # select sum(age) as totalAge from myCol
   # $sum:"$age" 从age列中取值做sum()运算
   # $age 代表文档中age的值
   MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:null,totalAge:{$sum:"$age"}}}])
   { "_id" : null, "totalAge" : 93 }
   
   # 根据name列做分组，求总年龄
   # select name as "_id", sum(age) as "totalAge" from myCol group by name
   MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:"$name",totalAge:{$sum:"$age"}}}])
   { "_id" : "gao", "totalAge" : 18 }
   { "_id" : "hao", "totalAge" : 25 }
   { "_id" : "高高", "totalAge" : 18 }
   { "_id" : 123, "totalAge" : 12 }
   { "_id" : "LuoJun", "totalAge" : 0 }
   { "_id" : "luo", "totalAge" : 20 }
   ```

2. $match 条件过滤 where / having

   **之所以可以区分where/having 是因为$match 和 $group前后是linux管道的关系（后一个操作从前一个操作的输出结果获取数据）**

   Where/having 的效果完全不一样

```shell
# $match 在$group之前表示where
# select sum(age) as totalAge from myCol where age > 20
# 先过滤数据，再分组
MongoDB Enterprise > db.myCol.aggregate([{$match:{age:{$gt:20}}},{$group:{_id:null,totalAge:{$sum:"$age"}}}])
{ "_id" : null, "totalAge" : 25 }

# $match 在 $group之后表示having
# 先分组，再过滤
# select sum(age) as totalAge from myCol group by name having totalAge > 20
MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:"$name",totalAge:{$sum:"$age"}}},{$match:{totalAge:{$gt:20}}}])
{ "_id" : "hao", "totalAge" : 25 }
```

3. $max 取最大值

```shell
# select max(age) as maxAge from myCol 
MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:null,maxAge:{$max:"$age"}}}])
{ "_id" : null, "maxAge" : 25 }
```

4. $min 查询最小值
```shell
# select min(age) as minAge from myCol
MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:null,minAge:{$min:"$age"}}}])
{ "_id" : null, "minAge" : 12 }
```

5. $avg 求平均值

   ```shell
   # select avg(age) as averageAge from myCol
   MongoDB Enterprise > db.myCol.aggregate([{$group:{_id:null,averageAge:{$avg:"$age"}}}])
   { "_id" : null, "averageAge" : 18.6 }
   ```

6. $push 统计结果并返回数组

   ```shell
   # 查询myCol集合，按照age分组并返回他们的name，如果age相同，则使用数组放回他们的name
   # nameArray:{$push:"$name"} 把同组的name的值放到nameArray数组中
   MongoDB Enterprise > db.myCol.aggregate({$group:{_id:"$age",nameArray:{$push:"$name"}}})
   { "_id" : 18, "nameArray" : [ "gao", "高高" ] }
   { "_id" : 12, "nameArray" : [ 123 ] }
   { "_id" : 20, "nameArray" : [ "luo", "张三" ] } #luo和张三 的age相同，name不同，name用数组返回
   { "_id" : 25, "nameArray" : [ "hao" ] }
   { "_id" : null, "nameArray" : [ "LuoJun" ] }
   ```

7. $unwind 数组字段拆分

   ```shell
   # 将数组字段（tag）拆分显示
   MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"}])
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : "喜欢敲代码" }
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : "玩电脑" }
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : "好看" }
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : "好玩" }
   
   # 全部数据
   MongoDB Enterprise > db.myCol.find()
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : [ "喜欢敲代码", "玩电脑" ] }
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a2"), "name" : "gao", "age" : 18 }
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : [ "好看", "好玩" ] }
   { "_id" : ObjectId("5efe96c1db47f7578a39d7a6"), "name" : "hao", "age" : 25 }
   { "_id" : ObjectId("5eff1333688c066442509563"), "name" : 123, "age" : 12 }
   { "_id" : ObjectId("5f04697c0cbe129b81ec9132"), "name" : "LuoJun" }
   { "_id" : ObjectId("5f04886a0cbe129b81ec9133"), "name" : "张三", "age" : 20 }
   ```
#### 2、聚合查询中的管道

* aggregate([{},{},{}])，数组中的每一个{}与下一个{}相连的","都可以看作是一个管道
* 例如 db.myCol.aggregate([{$group:{_id:null,sumAge:{$sum:"$age"}}},{$match:{sumAge:{$gt:20}}}])的$match和$group之间就是管道，$match中的 totalAge就是通过管道，从$group中传过来的
* 只能用于计算当前聚合管道的文档，不能处理其他文档


8. $project 聚合投影

   ```shell
   # 初级数据
   MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"}])
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : "喜欢敲代码" }
   { "_id" : ObjectId("5efe922cdb47f7578a39d7a1"), "name" : "luo", "age" : 20, "tag" : "玩电脑" }
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : "好看" }
   { "_id" : ObjectId("5efe9493db47f7578a39d7a4"), "name" : "高高", "age" : 18, "tag" : "好玩" }
   
   # 使用管道连接并使用$project 处理
   # nameTile:$"$name" 将管道传递过来的name提取出来并命名为nameTitle
   # _id:0 不显示_id
   MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"},{$project:{_id:0,nameTitle:"$name",tagTitle:"$tag"}}])
   { "nameTitle" : "luo", "tagTitle" : "喜欢敲代码" }
   { "nameTitle" : "luo", "tagTitle" : "玩电脑" }
   { "nameTitle" : "高高", "tagTitle" : "好看" }
   { "nameTitle" : "高高", "tagTitle" : "好玩" }
   ```
   
9. $toLower $toUpper

   ```shell
   MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"},{$project:{_id:0,upper_Name:{$toUpper:"$name"},lower_work:{$toLower:"$work"}}}])
   { "upper_Name" : "LUO", "lower_work" : "student" }
   { "upper_Name" : "LUO", "lower_work" : "student" }
   { "upper_Name" : "高高", "lower_work" : "" }
   { "upper_Name" : "高高", "lower_work" : "" }
   ```

10. $concat 拼接字符串

    ```shell
    MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"},{$project:{_id:0,concat_string:{$concat:["$name","-","$tag"]}}}])
    { "concat_string" : "luo-喜欢敲代码" }
    { "concat_string" : "luo-玩电脑" }
    { "concat_string" : "高高-好看" }
    { "concat_string" : "高高-好玩" }
    ```

11. $substr:["$要截取的字符串",start,end]

    ```shell
    # 如果要截取的字符串包含中文字符，则start 和 end 只能是3的倍数，因为UTF-8的中文字符占3字节
    # 因为$substr只能截取ASCII码的数据，如果要截取中文字符则需要使用$substrCP 
    MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"},{$project:{_id:0,name_prefix:{$substr:["$name",0,3]}}}])
    { "name_prefix" : "luo" }
    { "name_prefix" : "luo" }
    { "name_prefix" : "高" }
    { "name_prefix" : "高" }
    
    # 截取中文字符
    MongoDB Enterprise > db.myCol.aggregate([{$unwind:"$tag"},{$project:{_id:0,name_prefix:{$substrCP:["$name",0,2]}}}])
    { "name_prefix" : "lu" }
    { "name_prefix" : "lu" }
    { "name_prefix" : "高高" }
    { "name_prefix" : "高高" }
    ```

12. $project-算数运算

    在$project中可以通过MongoDB的算数操作符对投影的内容中运算处理
    
    ```shell
    # 获取到age，并将age+10 作为age_plus_10返回 age_plus_10:{$add:["$age",10]
    # $add:["$要进行计算的列",要添加的数值（也可以是另一个文档中的内容）]
    MongoDB Enterprise > db.myCol.aggregate([{$project:{_id:0,new_name:"$name",age_plus_10:{$add:["$age",10]}}}])
    { "new_name" : "luo", "age_plus_10" : 30 }
    { "new_name" : "gao", "age_plus_10" : 28 }
    { "new_name" : "高高", "age_plus_10" : 28 }
    { "new_name" : "hao", "age_plus_10" : 35 }
    { "new_name" : 123, "age_plus_10" : 22 }
    { "new_name" : "LuoJun", "age_plus_10" : null }
    { "new_name" : "张三", "age_plus_10" : 30 }
    
    # 去除age为null的数据
    # $ne no equals
    MongoDB Enterprise > db.myCol.aggregate([{$project:{_id:0,new_name:"$name",age_plus_10:{$add:["$age",10]}}},{$match:{age_plus_10:{$ne:null}}}])
    { "new_name" : "luo", "age_plus_10" : 30 }
    { "new_name" : "gao", "age_plus_10" : 28 }
    { "new_name" : "高高", "age_plus_10" : 28 }
    { "new_name" : "hao", "age_plus_10" : 35 }
    { "new_name" : 123, "age_plus_10" : 22 }
    { "new_name" : "张三", "age_plus_10" : 30 }
    
    # $subtract:["$",要减的数字或者文档]
    MongoDB Enterprise > db.myCol.aggregate([{$project:{_id:0,subtract_age:{$subtract:["$age",10]}}}])
    { "subtract_age" : 10 }
    { "subtract_age" : 8 }
    { "subtract_age" : 8 }
    { "subtract_age" : 15 }
    { "subtract_age" : 2 }
    { "subtract_age" : null }
    { "subtract_age" : 10 }
    
    # 乘法操作
    # $multiply
    MongoDB Enterprise > db.myCol.aggregate([{$project:{_id:0,new_age:{$multiply:["$age",2]}}}])
    { "new_age" : 40 }
    { "new_age" : 36 }
    { "new_age" : 36 }
    { "new_age" : 50 }
    { "new_age" : 24 }
    { "new_age" : null }
    { "new_age" : 40 }
    
    # 除法操作 $divide
    MongoDB Enterprise > db.myCol.aggregate([{$project:{_id:0,divide_age:{$divide:["$age",3]}}}])
    { "divide_age" : 6.666666666666667 }
    { "divide_age" : 6 }
    { "divide_age" : 6 }
    { "divide_age" : 8.333333333333334 }
    { "divide_age" : 4 }
    { "divide_age" : null }
    { "divide_age" : 6.666666666666667 }
    
    # 取模操作
    MongoDB Enterprise > db.myCol.aggregate([{$project:{_id:0,mod_3_age:{$mod:["$age",3]}}}])
    { "mod_3_age" : 2 }
    { "mod_3_age" : 0 }
    { "mod_3_age" : 0 }
    { "mod_3_age" : 1 }
    { "mod_3_age" : 0 }
    { "mod_3_age" : null }
    { "mod_3_age" : 2 }
    ```
    
    
    
    
    
    
    
    
    
    ## 八、MongoDB 集群
    
    ```shell
    [root@localhost bin]# ./mongod --config ../conf/mongod.conf                
    ./mongod: error while loading shared libraries: libnetsnmpmibs.so.35: canno
    t open shared object file: No such file or directory
    # 启动mongoDB时 如果报如下错误：
    
       error while loading shared libraries: libnetsnmpmibs.so.31: cannot open shared object file: No such file or directory
    
    该error 是因为未装net-snmp
    直接连接外网的Linux服务器可直接使用：yum install net-snmp
    ```
    
    ```shell
    # 配置文件
    dbpath=/opt/mongodb_cluster/replica_sets/mongodb_27017/data/db                        
    # 指定的是日志文件的名字                                                              
    logpath=/opt/mongodb_cluster/replica_sets/mongodb_27017/log/mongoDB.log               
    logappend=true                                                                        
    # 可以改端口                                                                          
    port=27017                                                                            
    # 绑定ip（若不设置绑定的ip，默认只有127.0.0.1可以连接mongod）                         
    bind_ip=0.0.0.0                                                                       
                                                                                          
    # 启用在后台运行mongos，或者mongod进程为守护进程模式                                  
    fork=true 
    # 设置副本集名称，在各个配置文件中，其值必须相同
    replSet=my_replica_sets
    ```
    
    ### 1、配置完成后，每个节点的配置信息
    
    日志文件路径、数据库路径、端口号不能相同
    
    ```shell
    # 节点一
    [root@localhost replica_sets]# cat mongodb_27017/conf/mongod.conf                     
    # 配置文件                                                                            
    dbpath=/opt/mongodb_cluster/replica_sets/mongodb_27017/data/db                        
    # 指定的是日志文件的名字                                                              
    logpath=/opt/mongodb_cluster/replica_sets/mongodb_27017/log/mongoDB.log               
    logappend=true                                                                        
    # 可以改端口                                                                          
    port=27017                                                                            
    # 绑定ip（若不设置绑定的ip，默认只有127.0.0.1可以连接mongod）                         
    bind_ip=0.0.0.0                                                                       
                                                                                          
    # 启用在后台运行mongos，或者mongod进程为守护进程模式                                  
    fork=true 
    # 设置副本集名称，在各个配置文件中，其值必须相同
    replSet=my_replica_sets
    
    # 节点二
    [root@localhost replica_sets]# cat mongodb_27018/conf/mongod.conf 
    # 配置文件
    dbpath=/opt/mongodb_cluster/replica_sets/mongodb_27018/data/db                        
    # 指定的是日志文件的名字                                                              
    logpath=/opt/mongodb_cluster/replica_sets/mongodb_27018/log/mongoDB.log               
    logappend=true                                                                        
    # 可以改端口                                                                          
    port=27018                                                                            
    # 绑定ip（若不设置绑定的ip，默认只有127.0.0.1可以连接mongod）                         
    bind_ip=0.0.0.0                                                                       
                                                                                          
    # 启用在后台运行mongos，或者mongod进程为守护进程模式                                  
    fork=true 
    # 设置副本集名称，在各个配置文件中，其值必须相同
    replSet=my_replica_sets
    
    # 节点三
    [root@localhost replica_sets]# cat mongodb_27019/conf/mongod.conf 
    # 配置文件
    dbpath=/opt/mongodb_cluster/replica_sets/mongodb_27019/data/db                        
    # 指定的是日志文件的名字                                                              
    logpath=/opt/mongodb_cluster/replica_sets/mongodb_27019/log/mongoDB.log               
    logappend=true                                                                        
    # 可以改端口                                                                          
    port=27019                                                                            
    # 绑定ip（若不设置绑定的ip，默认只有127.0.0.1可以连接mongod）                         
    bind_ip=0.0.0.0                                                                       
                                                                                          
    # 启用在后台运行mongos，或者mongod进程为守护进程模式                                  
    fork=true 
    # 设置副本集名称，在各个配置文件中，其值必须相同
    replSet=my_replica_sets
    [root@localhost replica_sets]# 
    ```
    
    ### 2、初始化副本集
    
    ```shell
    
    # 只需要初始化一个节点，（这个节点将成为主节点），再通过主节点添加其他仲裁节点，从节点
    MongoDB Enterprise > rs.initiate()                                                    
    {
            "info2" : "no configuration specified. Using a default configuration for the s
    et",
            "me" : "localhost.localdomain:27017",
            "ok" : 1,
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594553072, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            },
            "operationTime" : Timestamp(1594553072, 1)
    }
    MongoDB Enterprise my_replica_sets:OTHER> 
    MongoDB Enterprise my_replica_sets:PRIMARY>	# 初始化之后按一下回车就能变成 PRIMARY
    
    # 在主节点上添加其他节点
    MongoDB Enterprise my_replica_sets:PRIMARY> rs.add("192.168.2.30:27018")              
    {
            "ok" : 1,
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594555774, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            },
            "operationTime" : Timestamp(1594555774, 1)
    }
    
    # rs.add("192.168.2.30:27018",true) # 是添加仲裁节点
    # 添加仲裁节点
    MongoDB Enterprise my_replica_sets:PRIMARY> rs.addArb("192.168.2.30:27019")           
    {
            "ok" : 1,
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594555981, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            },
            "operationTime" : Timestamp(1594555981, 1)
    }
    
    # 添加错了可以这样删除节点
    MongoDB Enterprise my_replica_sets:PRIMARY> rs.remove("192.168.2.30:27019")           
    {
            "ok" : 1,
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594555906, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            },
            "operationTime" : Timestamp(1594555906, 1)
    }
    ```
    
    ### 3、默认的副本集初始化配置
    
    ```shell
    MongoDB Enterprise my_replica_sets:PRIMARY> rs.config()                               
    {
            "_id" : "my_replica_sets",
            "version" : 1,
            "protocolVersion" : NumberLong(1),
            "writeConcernMajorityJournalDefault" : true,
            "members" : [
                    {
                            "_id" : 0,
                            "host" : "localhost.localdomain:27019",
                            "arbiterOnly" : false,
                            "buildIndexes" : true,
                            "hidden" : false,
                            "priority" : 1,		# 主节点down之后，从新选举主节点的优先级，节点优先级
                            "tags" : {
                                    
                            },
                            "slaveDelay" : NumberLong(0),
                            "votes" : 1
                    }
            ],
            "settings" : {
                    "chainingAllowed" : true,
                    "heartbeatIntervalMillis" : 2000,
                    "heartbeatTimeoutSecs" : 10,
                    "electionTimeoutMillis" : 10000,
                    "catchUpTimeoutMillis" : -1,
                    "catchUpTakeoverDelayMillis" : 30000,
                    "getLastErrorModes" : {
                            
                    },
                    "getLastErrorDefaults" : {
                            "w" : 1,
                            "wtimeout" : 0
                    },
                    "replicaSetId" : ObjectId("5f0af36ef60cfe786c784b39")
            }
    }
    # 查看节点的状态
    MongoDB Enterprise my_replica_sets:PRIMARY> rs.status()                               
    {
            "set" : "my_replica_sets",
            "date" : ISODate("2020-07-12T12:14:31.571Z"),
            "myState" : 1,
            "term" : NumberLong(3),
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "heartbeatIntervalMillis" : NumberLong(2000),
            "majorityVoteCount" : 2,
            "writeMajorityCount" : 2,
            "optimes" : {
                    "lastCommittedOpTime" : {
                            "ts" : Timestamp(1594556061, 1),
                            "t" : NumberLong(3)
                    },
                    "lastCommittedWallTime" : ISODate("2020-07-12T12:14:21.601Z"),
                    "readConcernMajorityOpTime" : {
                            "ts" : Timestamp(1594556061, 1),
                            "t" : NumberLong(3)
                    },
                    "readConcernMajorityWallTime" : ISODate("2020-07-12T12:14:21.601Z"),
                    "appliedOpTime" : {
                            "ts" : Timestamp(1594556061, 1),
                            "t" : NumberLong(3)
                    },
                    "durableOpTime" : {
                            "ts" : Timestamp(1594556061, 1),
                            "t" : NumberLong(3)
                    },
                    "lastAppliedWallTime" : ISODate("2020-07-12T12:14:21.601Z"),
                    "lastDurableWallTime" : ISODate("2020-07-12T12:14:21.601Z")
            },
            "lastStableRecoveryTimestamp" : Timestamp(1594556001, 1),
            "lastStableCheckpointTimestamp" : Timestamp(1594556001, 1),
            "electionCandidateMetrics" : {
                    "lastElectionReason" : "electionTimeout",
                    "lastElectionDate" : ISODate("2020-07-12T12:07:31.541Z"),
                    "electionTerm" : NumberLong(3),
                    "lastCommittedOpTimeAtElection" : {
                            "ts" : Timestamp(0, 0),
                            "t" : NumberLong(-1)
                    },
                    "lastSeenOpTimeAtElection" : {
                            "ts" : Timestamp(1594555634, 1),
                            "t" : NumberLong(2)
                    },
                    "numVotesNeeded" : 1,
                    "priorityAtElection" : 1,
                    "electionTimeoutMillis" : NumberLong(10000),
                    "newTermStartDate" : ISODate("2020-07-12T12:07:31.543Z"),
                    "wMajorityWriteAvailabilityDate" : ISODate("2020-07-12T12:07:31.573Z")
            },
            "members" : [		# 副本集的信息
                    {
                            "_id" : 0,
                            "name" : "localhost.localdomain:27017",	
                            "health" : 1,
                            "state" : 1,
                            "stateStr" : "PRIMARY",		# 主节点
                            "uptime" : 421,
                            "optime" : {
                                    "ts" : Timestamp(1594556061, 1),
                                    "t" : NumberLong(3)
                            },
                            "optimeDate" : ISODate("2020-07-12T12:14:21Z"),
                            "syncingTo" : "",
                            "syncSourceHost" : "",
                            "syncSourceId" : -1,
                            "infoMessage" : "",
                            "electionTime" : Timestamp(1594555651, 1),
                            "electionDate" : ISODate("2020-07-12T12:07:31Z"),
                            "configVersion" : 5,
                            "self" : true,
                            "lastHeartbeatMessage" : ""
                    },
                    {
                            "_id" : 1,
                            "name" : "192.168.2.30:27018",
                            "health" : 1,
                            "state" : 2,
                            "stateStr" : "SECONDARY",  # 从节点
                            "uptime" : 295,
                            "optime" : {
                                    "ts" : Timestamp(1594556061, 1),
                                    "t" : NumberLong(3)
                            },
                            "optimeDurable" : {
                                    "ts" : Timestamp(1594556061, 1),
                                    "t" : NumberLong(3)
                            },
                            "optimeDate" : ISODate("2020-07-12T12:14:21Z"),
                            "optimeDurableDate" : ISODate("2020-07-12T12:14:21Z"),
                            "lastHeartbeat" : ISODate("2020-07-12T12:14:29.654Z"),
                            "lastHeartbeatRecv" : ISODate("2020-07-12T12:14:30.671Z"),
                            "pingMs" : NumberLong(0),
                            "lastHeartbeatMessage" : "",
                            "syncingTo" : "localhost.localdomain:27017",
                            "syncSourceHost" : "localhost.localdomain:27017",
                            "syncSourceId" : 0,
                            "infoMessage" : "",
                            "configVersion" : 5
                    },
                    {
                            "_id" : 2,
                            "name" : "192.168.2.30:27019",
                            "health" : 0,
                            "state" : 8,
                            "stateStr" : "(not reachable/healthy)",		# 异常节点（重启该节点之后就正常了）
                            "uptime" : 0,
                            "lastHeartbeat" : ISODate("2020-07-12T12:14:29.655Z"),
                            "lastHeartbeatRecv" : ISODate("2020-07-12T12:13:03.644Z"),
                            "pingMs" : NumberLong(0),
                            "lastHeartbeatMessage" : "Error connecting to 192.168.2.30:270
    19 :: caused by :: Connection refused",
                            "syncingTo" : "",
                            "syncSourceHost" : "",
                            "syncSourceId" : -1,
                            "infoMessage" : "",
                            "configVersion" : -1
                    }
            ],
            "ok" : 1,
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594556061, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            },
            "operationTime" : Timestamp(1594556061, 1)
    }                            
    
    # 重启 192.168.2.30:27019 之后
    MongoDB Enterprise my_replica_sets:PRIMARY> rs.status()                               
    {
            "set" : "my_replica_sets",
            "date" : ISODate("2020-07-12T12:18:58.453Z"),
            "myState" : 1,
            "term" : NumberLong(3),
            "syncingTo" : "",
            "syncSourceHost" : "",
            "syncSourceId" : -1,
            "heartbeatIntervalMillis" : NumberLong(2000),
            "majorityVoteCount" : 2,
            "writeMajorityCount" : 2,
            "optimes" : {
                    "lastCommittedOpTime" : {
                            "ts" : Timestamp(1594556331, 1),
                            "t" : NumberLong(3)
                    },
                    "lastCommittedWallTime" : ISODate("2020-07-12T12:18:51.640Z"),
                    "readConcernMajorityOpTime" : {
                            "ts" : Timestamp(1594556331, 1),
                            "t" : NumberLong(3)
                    },
                    "readConcernMajorityWallTime" : ISODate("2020-07-12T12:18:51.640Z"),
                    "appliedOpTime" : {
                            "ts" : Timestamp(1594556331, 1),
                            "t" : NumberLong(3)
                    },
                    "durableOpTime" : {
                            "ts" : Timestamp(1594556331, 1),
                            "t" : NumberLong(3)
                    },
                    "lastAppliedWallTime" : ISODate("2020-07-12T12:18:51.640Z"),
                    "lastDurableWallTime" : ISODate("2020-07-12T12:18:51.640Z")
            },
            "lastStableRecoveryTimestamp" : Timestamp(1594556301, 1),
            "lastStableCheckpointTimestamp" : Timestamp(1594556301, 1),
            "electionCandidateMetrics" : {
                    "lastElectionReason" : "electionTimeout",
                    "lastElectionDate" : ISODate("2020-07-12T12:07:31.541Z"),
                    "electionTerm" : NumberLong(3),
                    "lastCommittedOpTimeAtElection" : {
                            "ts" : Timestamp(0, 0),
                            "t" : NumberLong(-1)
                    },
                    "lastSeenOpTimeAtElection" : {
                            "ts" : Timestamp(1594555634, 1),
                            "t" : NumberLong(2)
                    },
                    "numVotesNeeded" : 1,
                    "priorityAtElection" : 1,
                    "electionTimeoutMillis" : NumberLong(10000),
                    "newTermStartDate" : ISODate("2020-07-12T12:07:31.543Z"),
                    "wMajorityWriteAvailabilityDate" : ISODate("2020-07-12T12:07:31.573Z")
            },
            "members" : [
                    {
                            "_id" : 0,
                            "name" : "localhost.localdomain:27017",
                            "health" : 1,
                            "state" : 1,
                            "stateStr" : "PRIMARY",
                            "uptime" : 688,
                            "optime" : {
                                    "ts" : Timestamp(1594556331, 1),
                                    "t" : NumberLong(3)
                            },
                            "optimeDate" : ISODate("2020-07-12T12:18:51Z"),
                            "syncingTo" : "",
                            "syncSourceHost" : "",
                            "syncSourceId" : -1,
                            "infoMessage" : "",
                            "electionTime" : Timestamp(1594555651, 1),
                            "electionDate" : ISODate("2020-07-12T12:07:31Z"),
                            "configVersion" : 5,
                            "self" : true,
                            "lastHeartbeatMessage" : ""
                    },
                    {
                            "_id" : 1,
                            "name" : "192.168.2.30:27018",
                            "health" : 1,
                            "state" : 2,
                            "stateStr" : "SECONDARY",
                            "uptime" : 562,
                            "optime" : {
                                    "ts" : Timestamp(1594556331, 1),
                                    "t" : NumberLong(3)
                            },
                            "optimeDurable" : {
                                    "ts" : Timestamp(1594556331, 1),
                                    "t" : NumberLong(3)
                            },
                            "optimeDate" : ISODate("2020-07-12T12:18:51Z"),
                            "optimeDurableDate" : ISODate("2020-07-12T12:18:51Z"),
                            "lastHeartbeat" : ISODate("2020-07-12T12:18:57.847Z"),
                            "lastHeartbeatRecv" : ISODate("2020-07-12T12:18:56.867Z"),
                            "pingMs" : NumberLong(0),
                            "lastHeartbeatMessage" : "",
                            "syncingTo" : "localhost.localdomain:27017",
                            "syncSourceHost" : "localhost.localdomain:27017",
                            "syncSourceId" : 0,
                            "infoMessage" : "",
                            "configVersion" : 5
                    },
                    {
                            "_id" : 2,
                            "name" : "192.168.2.30:27019",
                            "health" : 1,
                            "state" : 7,
                            "stateStr" : "ARBITER",	# 仲裁节点
                            "uptime" : 60,
                            "lastHeartbeat" : ISODate("2020-07-12T12:18:57.889Z"),
                            "lastHeartbeatRecv" : ISODate("2020-07-12T12:18:57.459Z"),
                            "pingMs" : NumberLong(0),
                            "lastHeartbeatMessage" : "",
                            "syncingTo" : "",
                            "syncSourceHost" : "",
                            "syncSourceId" : -1,
                            "infoMessage" : "",
                            "configVersion" : 5
                    }
            ],
            "ok" : 1,
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594556331, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            },
            "operationTime" : Timestamp(1594556331, 1)
    }
    ```
    
    | 节点     | 说明                                                         |
    | -------- | ------------------------------------------------------------ |
    | 主节点   | 可以进行所有操作                                             |
    | 从节点   | 用于读写分离，可读，不可写                                   |
    | 仲裁节点 | 不存放任何业务数据（即使执行rs.slaveOk()也不可以）不可读，不可写。其他节点down了之后，仲裁节点不能被选举为主节点（因为优先级是0） |
    
    
    ```shell
    # 在从节点进行操作 192.168.2.30:27018
    MongoDB Enterprise my_replica_sets:SECONDARY> show dbs                                
    2020-07-12T08:27:26.435-0400 E  QUERY    [js] uncaught exception: Error: listDatabases
     failed:{
            "operationTime" : Timestamp(1594556841, 1),
            "ok" : 0,
            "errmsg" : "not master and slaveOk=false", # 虽然主节点已经将本节点添加为从节点，但是本节点还没有确认，所以show dbs 失败
            "code" : 13435,
            "codeName" : "NotMasterNoSlaveOk",
            "$clusterTime" : {
                    "clusterTime" : Timestamp(1594556841, 1),
                    "signature" : {
                            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                            "keyId" : NumberLong(0)
                    }
            }
    } :
    _getErrorWithCode@src/mongo/shell/utils.js:25:13
    Mongo.prototype.getDBs/<@src/mongo/shell/mongo.js:135:19
    Mongo.prototype.getDBs@src/mongo/shell/mongo.js:87:12
    shellHelper.show@src/mongo/shell/utils.js:906:13
    shellHelper@src/mongo/shell/utils.js:790:15
    @(shellhelp2):1:1
    
    # 确认本节点是从节点
    # 取消从节点的确认 rs.slaveOk(false) 
    MongoDB Enterprise my_replica_sets:SECONDARY> rs.slaveOk() 
    # 此时命令可以正常执行
    MongoDB Enterprise my_replica_sets:SECONDARY> show dbs                                
    admin   0.000GB
    config  0.000GB
    home    0.000GB
    local   0.000GB
    ```
    
    ### 4、此时连接mongoDB集群的时候需要指定副本集名称
    
     ![image-20200712211549799](/Users/luo/Library/Application Support/typora-user-images/image-20200712211549799.png)
