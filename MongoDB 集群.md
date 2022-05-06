# MongoDB 集群

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

### 配置完成后，每个节点的配置信息

#### 日志文件路径、数据库路径、端口号不能相同

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

### 初始化副本集

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

#### 默认的副本集初始化配置

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

| 节点   | 说明             |
| ------ | ---------------- |
| 主节点 | 可以进行所有操作 |
|从节点|用于读写分离，可读，不可写|
|仲裁节点|不存放任何业务数据（即使执行rs.slaveOk()也不可以）不可读，不可写。其他节点down了之后，仲裁节点不能被选举为主节点（因为优先级是0）|


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

### 此时连接mongoDB集群的时候需要指定副本集名称

 ![image-20200712211549799](/Users/luo/Library/Application Support/typora-user-images/image-20200712211549799.png)