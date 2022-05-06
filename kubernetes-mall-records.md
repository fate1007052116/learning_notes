# 谷粒商城部署到kubernetes实战笔记

### 1.mysql不能往nfs写入数据的问题

```shell
mkdir /data/{mysql_slave_1,mysql_slave_2,mysql_master_1,mysql_master_2}/{data,conf,logs} -p

luo@node1:/data$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#


/data/mysql_master_1/data *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_master_1/conf *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_master_1/logs *(rw,sync,no_subtree_check,no_root_squash)

/data/mysql_master_2/data *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_master_2/conf *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_master_2/logs *(rw,sync,no_subtree_check,no_root_squash)

/data/mysql_slave_1/data *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_slave_1/conf *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_slave_1/logs *(rw,sync,no_subtree_check,no_root_squash)

/data/mysql_slave_2/data *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_slave_2/conf *(rw,sync,no_subtree_check,no_root_squash)
/data/mysql_slave_2/logs *(rw,sync,no_subtree_check,no_root_squash)


luo@node1:/data$ systemctl enable --now nfs-server
luo@node1:/data$ systemctl enable --now rpcbind

# 递归授权
luo@node1:/data$ sudo chmod 777 /data/ -R
```

在客户机通过

mount -o rw -t nfs 192.168.192.204:/mnt/cephfs /mnt/nfs

命令将网络文件mount到本地。执行完成之后，目录是可以访问了，但无法写入。

分析：

   用户对目录的权限受两方面约束：NFS认证权限、Posix权限；

   NFS权限：

   NFS服务器器中exports中配置额读写、只读权限

   Posix权限：

   发现exports目录权限中，参数no_root_squash的其作用是：NFS客户端使用共享目录的用户，如果是root 的话，所有的操作均在服务器端映射为root用户，拥有共享目录的root权限！

   默认情况使用的是相反参数root_squash：在登入 NFS 主机export目录的使用者如果当root时，那么这个使用者的权限将被压缩成为匿名使用者，通常他的 UID 与 GID 都会变成 nobody 那个身份。
因为客户端是使用root登录的，自然权限被压缩为nobody了，难怪无法写入。

将配置信息改为：

/mnt/cephfs 192.168.192.0/8(rw,no_root_squash)
据说有点不安全，但问题是解决了。 

```shell
# 成功解析到服务
[luo@master ~]$ dig -t A mysql-master-1-service.default.svc.cluster.local. @10.244.0.77


mysql-master-1-service.default.svc.cluster.local. 30 IN	A 10.244.2.19

;; Query time: 17 msec
;; SERVER: 10.244.0.77#53(10.244.0.77)
;; WHEN: Wed Apr 14 23:07:35 CST 2021
;; MSG SIZE  rcvd: 153


# pod 之间可以通过service的名字来互相ping通
kubectl exec -it ping -- ping mysql-master-1-service.default.svc.cluster.local
PING mysql-master-1-service.default.svc.cluster.local (10.244.2.20): 56 data bytes
64 bytes from 10.244.2.20: seq=0 ttl=62 time=1.164 ms
64 bytes from 10.244.2.20: seq=1 ttl=62 time=0.803 ms
64 bytes from 10.244.2.20: seq=2 ttl=62 time=0.763 ms


```

### 2.创建mysql

```yaml
---
# 用于集群内部互相连接使用
apiVersion: v1
kind: Service
metadata:
  name: mysql-master-service
spec:
  clusterIP: None
  selector:
    app: mysql-master
  ports:
    - port: 33060
      targetPort: 3306
---
# 用于使用mysql工具连接到数据库
apiVersion: v1
kind: Service
metadata:
  name: mysql-master-service-node-port
spec:
  type: NodePort
  selector:
    app: mysql-master
  ports:
    - port: 33060
      targetPort: 3306
      nodePort: 30002
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-master-stateful-set
spec:
  selector:
    matchLabels:
      app: mysql-master
  serviceName: mysql-master-service
  template:
    metadata:
      labels:
        app: mysql-master
    spec:
      volumes:
        - name: host-path-volume
          hostPath:
            path: /Users/luo/docker/mysql-master/data
            type: DirectoryOrCreate
      containers:
        - name: mysql-master
          image: mysql:8.0.21
          imagePullPolicy: IfNotPresent
#          env:
#            - name: MYSQL_ROOT_PASSWORD  # 第一次初始化的时候要打开注释
#              value: "123456"
          ports:
            - containerPort: 3306
              name: mysql-port
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: host-path-volume
          readinessProbe:
            tcpSocket:
              port: 3306    # 基于端口的就绪检测
            initialDelaySeconds: 3
            periodSeconds: 1


```

使用一个临时`mysql`来测试mysql连接情况

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-client
spec:
  containers:
    - name: mysql-master
      image: mysql:8.0.21
      imagePullPolicy: IfNotPresent
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"
      ports:
        - containerPort: 3306
          name: mysql-port
```

```shell
# 查看目前已有的pod
kubectl get all

NAME                                    READY   STATUS    RESTARTS   AGE
pod/mysql-client                        1/1     Running   0          6h45m
pod/mysql-master-stateful-set-0         1/1     Running   0          15m

NAME                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/kubernetes                       ClusterIP   10.96.0.1        <none>        443/TCP           6h52m
service/mysql-master-service             ClusterIP   None             <none>        33060/TCP         15m
service/mysql-master-service-node-port   NodePort    10.105.46.182    <none>        33061:32656/TCP   15m

NAME                                         READY   AGE
statefulset.apps/mysql-master-stateful-set   1/1     15m

# 使用临时创建出来的mysql，用其命令行工具连接具体要用的mysql
# 注意这里使用的端口是 mysql-master-stateful-set 的 template 中 的 container 自己的端口，与service的端口并没有任何关系，service只相当于一个dns代理的作用
# 这一步成功可以确认 pod 和 headless service 正常运行
kubectl exec -it mysql-client  -- mysql -h mysql-master-service -P 3306 -uroot -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 34
Server version: 8.0.21 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```

### 3.部署nacos

> 这里有详细的pod之间通过`service`来通信的官方文档：`https://kubernetes.io/zh/docs/concepts/services-networking/connect-applications-service/`

```properties
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
# 需要打开数据库配置
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
# 注意这里使用的是 headless的service名字，如果是夸命名空间连接service，则还需要添加命名空间作为前缀
db.url.0=jdbc:mysql://mysql-master-service:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
# 不要忘记了设置密码
db.user.0=root
db.password.0=123456
```

```shell
# 查看文档结构
ll                                                                                       
total 56
-rw-r--r--  1 luo  staff   112B  4 30 10:24 Dockerfile
-rw-r--r--@ 1 luo  staff    16K 12 15 15:29 LICENSE
-rw-r--r--@ 1 luo  staff   1.3K  5 14  2020 NOTICE
drwxr-xr-x  6 luo  staff   192B  4 30 10:23 nacos

# 查看nacos的 Dockerfile
cat Dockerfile                                                                                    
FROM openjdk:8
COPY ./nacos /usr/local/nacos
ENTRYPOINT ["/usr/local/nacos/bin/startup.sh", "-m", "standalone"]
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nacos-deployment
spec:
  selector:
    matchLabels:
      app: nacos
  template:
    metadata:
      name: nacos-container
      labels:
        app: nacos
    spec:
      initContainers:
        - name: test-mysql
          image: busybox
            # 只有解析到 dns my-service 才会跳出本 init 容器
          command: [ 'sh','-c','until nslookup mysql-master-service; do echo waiting for mysql-master-service; sleep 2;done;' ]
      containers:
        - name: nacos
          imagePullPolicy: IfNotPresent
          image: nacos
          command:
            - sh
            - -c
            - "/usr/local/nacos/bin/startup.sh -m standalone ; sleep 2 ; tail -f /usr/local/nacos/logs/start.out"
            # 这样做的好处是使用 kubectl logs -f 就可以直接看到日志 
---
apiVersion: v1
kind: Service
metadata:
  name: nacos-service
spec:
  selector:
    app: nacos
  type: NodePort
  ports:
    - port: 8888
      targetPort: 8848
      nodePort: 30001
```



