# Zookeeper 安装

```shell
# 进入zookeeper 的conf目录
[root@localhost conf]# cd /opt/zookeeper-3.6.1-bin/conf
[root@localhost conf]# ll
总用量 12
-rw-r--r--. 1 luo luo  535 4月  21 10:59 configuration.xsl
-rw-r--r--. 1 luo luo 3435 4月  21 10:59 log4j.properties
-rw-r--r--. 1 luo luo 1148 4月  21 10:59 zoo_sample.cfg
# 拷贝一份配置文件
[root@localhost conf]# cp zoo_sample.cfg zoo.cfg
[root@localhost conf]# cd ..
# 创建数据目录
[root@localhost zookeeper-3.6.1-bin]# mkdir /opt/zookeeper-3.6.1-bin/data/
# 配置刚创建的data目录作为数据存放目录
[root@localhost zookeeper-3.6.1-bin]# vi /opt/zookeeper-3.6.1-bin/conf/zoo.cfg
[root@localhost zookeeper-3.6.1-bin]# 

# The number of milliseconds of each tick
tickTime=2000                                                              
# The number of ticks that the initial                                     
# synchronization phase can take                                           
initLimit=10                                                               
# The number of ticks that can pass between                                
# sending a request and getting an acknowledgement                         
syncLimit=5                                                                
# the directory where the snapshot is stored.                              
# do not use /tmp for storage, /tmp here is just                           
# example sakes.                                                           
dataDir=/opt/zookeeper-3.6.1-bin/data          # 配置数据存放目录                            
# the port at which the clients will connect                               
clientPort=2181                                                            
# the maximum number of client connections.                                
# increase this if you need to handle more clients                         
#maxClientCnxns=60  

# 启动Zookeeper
[root@localhost bin]# ./zkServer.sh start                                  
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper-3.6.1-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

# 使用zkCli.sh 连接zookeeper
[root@localhost bin]# ./zkCli.sh -server 192.168.2.20:2181 
```

## Zookeeper 集群安装

```shell
# 需要在每个zookeeper的数据目录（这里是/opt/zookeeper-3.6.1-bin/data/）创建myid 文件
# myid 文件中有zookeeper的id，id要求是自然数和不重复
vi /opt/zookeeper-3.6.1-bin/data/myid
1

# 在zoo.cfg中添加集群配置
[root@localhost zookeeper-3.6.1-bin]# vi /opt/zookeeper-3.6.1-bin/conf/zoo.cfg 

# 修改配置文件zoo.cfg - 设置监听客户端、投票、选举端口
server.1=192.168.2.20:2881:3881		# 对应myid 的值 是 1
server.2=192.168.2.20:2882:3882		# 对应myid 的值 是 2
server.3=192.168.2.20:2883:3883		# 对应myid 的值 是 3


# 为每一台zookeeper设置id
[root@localhost zookeeper_cluster]# pwd
/opt/zookeeper_cluster
[root@localhost zookeeper_cluster]# tail zoo1/data/myid
1
[root@localhost zookeeper_cluster]# tail zoo2/data/myid
2
[root@localhost zookeeper_cluster]# tail zoo3/data/myid
3
# 每台的投票、选举端口
[root@localhost zookeeper_cluster]# cat zoo1/conf/zoo.cfg | grep server
server.1=192.168.2.20:2881:3881                                            
server.2=192.168.2.20:2882:3882                                            
server.3=192.168.2.20:2883:3883                                            
[root@localhost zookeeper_cluster]# cat zoo2/conf/zoo.cfg | grep server
server.1=192.168.2.20:2881:3881                                            
server.2=192.168.2.20:2882:3882                                            
server.3=192.168.2.20:2883:3883                                            
[root@localhost zookeeper_cluster]# cat zoo3/conf/zoo.cfg | grep server
server.1=192.168.2.20:2881:3881                                            
server.2=192.168.2.20:2882:3882                                            
server.3=192.168.2.20:2883:3883                                            
[root@localhost zookeeper_cluster]# 

# 配置客户端端口                                                          
[root@localhost zookeeper_cluster]# cat zoo1/conf/zoo.cfg | grep clientPort                                                     
clientPort=2181                                                            
[root@localhost zookeeper_cluster]# cat zoo2/conf/zoo.cfg | grep clientPort                                                               
clientPort=2182                                                            
[root@localhost zookeeper_cluster]# cat zoo3/conf/zoo.cfg | grep clientPort                                                                  
clientPort=2183 

# 数据目录
[root@localhost zookeeper_cluster]# cat zoo1/conf/zoo.cfg|grep dataDir=
dataDir=/opt/zookeeper_cluster/zoo1/data
[root@localhost zookeeper_cluster]# cat zoo2/conf/zoo.cfg|grep dataDir=
dataDir=/opt/zookeeper_cluster/zoo2/data
[root@localhost zookeeper_cluster]# cat zoo3/conf/zoo.cfg|grep dataDir=
dataDir=/opt/zookeeper_cluster/zoo3/data
```



# zookeeper 配置

* 需要在 zookeeper/conf/下，将zoo_sample.cfg 文件 拷贝 位zoo.cfg，对其进行相应的配置，主要配置dataDir = zookeeper/data/  data 文件夹需要自己手动创建

* Zookeeper/bin/  ./zkServer.sh start 启动服务端

* Zookeeper/bin/  ./zkCli.sh -server 192.168.2.20:2181 启动客户端，连接指定服务端

* 每个服务端需要一个唯一标识id，这个id存放于dataDir 下的 myid文件中，即zookeeper/data/myid  (ID要求是不重复的自然数) myid文件中的内容直接为id即可，比如 1,2,3....

* 在zookeeper/conf/zoo.cfg 文件下添加（每个zookeeper中都需要有这部分的配置）注意没有空格

# 同一台服务器的客户端监听端口、投票端口、选举端口不可以重复（找错找了一下午）

* 客户端监听端口

* clientPort=2181 服务器一
  服务器ip:投票端口，选举端口

  server.1=192.168.2.10:2881:3881
  server.2=192.168.2.20:2882:3882
  server.3=192.168.2.30:2883:3883

* clientPort=2182 服务器二
  server.1=192.168.2.10:2881:3881
  server.2=192.168.2.20:2882:3882
  server.3=192.168.2.30:2883:3883
  
* clientPort=2183 服务器三
  server.1=192.168.2.10:2881:3881
  server.2=192.168.2.20:2882:3882
  server.3=192.168.2.30:2883:3883

​       这里的server.id   id就是myid文件中的id

## 任何zookeeper客户端工具都可以连接zookeeper集群

# zookeeper 命令

## Ls /path  必须指定path，或者 ls / 

## Create (-e 创建临时节点)(-s 创建序列化节点，默认) /path (data 该节点中的数据)

	 create /luo 在根下创建永久节点luo
	 Create -e /temp  tempData在根下创建临时节点temp，temp节点的数据是tempData
	
	 create -s /luo_sequence luo_seq_data  在根下创建永久序列化节点luo_sequence 该节点的数据为luo_seq_data
	 Created /luo_sequence0000000002 创建完成，该节点的序列号是2
	 create -e -s /temp_seq temp_seq_data 在根下创建临时序列化节点temp_seq，节点的数据是 temp_seq_data，创建完成Created /temp_seq0000000003

## get (-s 返回详细信息) /path  获取节点中的数据 

* 存放的数据
*  cZxid:创建时zxid（znode每次改变时递增的事务id）
* ctime：创建时间戳
* mZxid：最近一次更新的zxid
* mtime：最近一次更新的时间戳
* pZxid：子节点的zxid（如果没有子节点，则显示本节点的zxid）
* cversion：子节点的更新次数
* dataversion：节点数据的更新次数
* aclVersion：节点ACL（授权信息）的更新次数
* ephemeraIOwner：如果该节点为ephemeral节点（临时，生命周期与session一样，zookeeper客户端连接到zookeeper集群的session），ephemeralOwner 值表示该节点绑定的sessionId，如果该节点不是ephemeral节点，ephemeralOwner值为0
* dataLength：节点数据的字节数
* NumChildren：子节点的数量

## set /path data

* 设置该节点的数据

## delete /path 删除节点

* 删除该节点



## 即使是在不同主机，投票端口和选举端口必须唯一

## 在不同主机，客户端的端口号可以一致，前提是ip不同



clientPort=2181
server.1=192.168.2.10:2881:3881
server.2=192.168.2.20:2882:3882
server.3=192.168.2.30:2883:3883

