# Spring Cloud Alibaba

## 一、spring cloud Alibaba 简介

### 1. 为什么会出现Spring cloud Alibaba

主要是 Spring Cloud Netflix 项目进入维护模式

进入维护模式意味着：

Spring Cloud Netflix 将不再开发新组件

我们都知道Spring Cloud 版本迭代算是比较快的，因而出现了很多重大问题都还来不及修复就又推另外一个Release了，进入维护模式意思就是，目前一直以后一段时间Spring Cloud Netflix提供的服务和功能就这么多了，不再开发新的功能和组件了。以后将以维护和Merge分支Full Request 为主

新组件功能将以其他替代品代替的方式实现

![Snip20200803_20](/Users/luo/Documents/开发笔记/images/Snip20200803_20.png)



## 二、服务注册与配置中心

Nacos (Naming Configuration service)

https://nacos.io/zh-cn/

### 1. 是什么

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台

Nacos：Dynamic Naming and Configuration Service

Nacos：注册中心 + 配置中心的组合

Nacos = Eureka + Config + Bus

### 2. 能干嘛

替换Eureka做服务注册中心

替换Config做服务配置中心

### 3. 运行nacos

```shell
# Linux/Unix/Mac
luo@luodeMacBook-Pro bin % cd /Volumes/extend/docker_images/nacos/bin
luo@luodeMacBook-Pro bin % sh startup.sh -m standalone

# 如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：
bash startup.sh -m standalone

# 查看日志
luo@luodeMacBook-Pro bin % tail -f /Volumes/extend/docker_images/nacos/logs/nacos.log

# 关闭
sh shutdown.sh
```

Nacos web 管理页面，账号nacos，密码 nacos

http://localhost:8848/nacos

### 4. 配置父工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>parent</name>
    <description>Demo project for Spring Boot</description>
    <packaging>pom</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        <java.version>1.8</java.version>
        <spring-cloud-alibaba.version>2.2.1.RELEASE</spring-cloud-alibaba.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>


    <dependencyManagement>
        <dependencies>
            <!--spring boot 2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.9.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```



### 5. 参照官方教程

https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_spring_cloud_alibaba_nacos_discovery

### 6. 各注册中心的对比

![Snip20200804_1](/Users/luo/Documents/开发笔记/images/Snip20200804_2.png)

Nacos 支持两种CAP模型

* AP
* CP

![Snip20200804_3](/Users/luo/Documents/开发笔记/images/Snip20200804_3.png)

![Snip20200804_4](/Users/luo/Documents/开发笔记/images/Snip20200804_5.png)

### 7. Nacos 支持AP 、 CP 的自动切换

C 是所有节点在同一时间看到的数据是一致的

A 是所有的请求都会收到响应（高可用）

#### （1）什么时候选择什么模式？

一般来说，如果不需要存储服务级别的信息且服务实例是通过nacos-client注册的，并且能够保持心跳上报，那么就可以选择AP模式。当前主流的服务，如Spring cloud 和 Dubbo 服务，都适用于AP模式，**AP模式为了服务的高可用性而减弱了一致性，因此AP模式下只支持注册临时实例**。

如果需要在服务级别编辑或者存储配置信息，那么cp是必须的，K8S服务和DNS服务则适用于CP模式。CP模式下则支持注册持久化实例，此时则是以Raft协议为集群运行模式，该模式下注册实例之前必须注册服务，如果服务不存在则，返回错误

#### （2）如何切换

```shell
curl -X PUT "$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP"
```

### 8. 配置Nacos作为配置中心



#### （1）Nacos中应用名匹配规则

​	在 `bootstrap.properties` 中配置 Nacos server 的地址和应用名

```yml
spring:
  application:
    name: nacos-config-client
  profiles:
    active: dev     # 开发环境
  cloud:
    nacos:
      # 服务发现
      discovery:
        server-addr: 127.0.0.1:8848 # nacos注册中心的地址
      # 服务配置
      config:
        server-addr: 127.0.0.1:8848 # nacos配置中心的地址
        file-extension: yaml        # 指定yml格式的配置，这里如果是yml，则nacos配置中心的对应的DataId文件扩展名也必须是yml；不可以混淆使用
        prefix: myDataId
      # 拼接之后的文件名是： myDataId-dev.yaml  # 扩展名必须完全一致，yml 不可以 对应 yaml；
```

​	Nacos中的dataId的组成格式及与Springboot 配置文件中的匹配规则

说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```yml
${prefix}-${spring.profiles.active}.${file-extension}
```

* `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
* `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**，省略会导致一些问题
* `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。
* Data ID 中的扩展名不可以省略，扩展名必须完全相同，yml 对 yml；yaml对yaml，不可以混淆使用
* `coupon-service-dev.yml`的优先级比`coupon-service.yml`高
* 如果本地配置文件`application.yml`和`nacos`配置中心都有的项，优先使用配置中心中的配置

![Snip20200804_7](/Users/luo/Documents/开发笔记/images/Snip20200804_7.png)

#### （2）动态刷新nacos中的配置

在`@Value`所在的类上添加`@RefreshScope`注解即可，`@Value`的内容在`nacos`配置中心中

### 9. 多环境多项目管理

#### （1）面临的问题

【1】实际开发中，通常一个一个系统会准备

* dev 开发环境
* test 测试环境
* prod 生产环境

如何保证指定环境启动时，服务能正确读取到nacos上相应环境的配置文件呢？

【2】一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境。。怎样对这些微服务配置进行管理呢？

#### （2）Nacos图形化管理界面

* 配置管理
* 命名空间

#### （3）Namespace + Group + Data Id

类似java里面的 package 名 和类名，最外层的namespace 是可以用于区分部署环境的，group和data Id逻辑上区分这两个目标对象

三者关系

![Snip20200804_8](/Users/luo/Documents/开发笔记/images/Snip20200804_8.png)

默认情况：

Namespace=public

group=DEFAULT_GROUP

Cluster=DEFAULT

Nacos默认的命名空间是public，Namespace主要用来实现隔离。比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。

Group默认是DEFAULT_GROUP，group 可以把不同的微服务划分到同一个分组里面去

service就是微服务；一个service可以包含多个cluster集群，nacos默认的Cluster是DEFULT，cluster是指对微服务的一个虚拟部分。比方说为了容灾，将service微服务分别部署在了杭州和广州的机房里，这时就可以给杭州机房的service微服务起一个集群名称（hangzhou），给广州机房的service微服务起一个集群名称（guangzhou），还可以尽量让同一个机房的微服务互相调用，以提升性能

最后是instance，就是微服务的实例

#### （4）Data Id 方案

指定spring.profile.active 和 配置文件的`Data ID`来使得不同环境下读取不同的配置

默认namespace + 默认 group + 新建 dev 和 test 两个 Data Id

![Snip20200804_9](/Users/luo/Documents/开发笔记/images/Snip20200804_9.png)

通过`spring.profile.active`属性就可以进行多环境下配置文件的读取

`Bootstrap.yml` 或者`application.yml`中配置什么就是什么

![Snip20200804_10](/Users/luo/Documents/开发笔记/images/Snip20200804_10.png)

#### （5）Group 方案

![Snip20200804_11](/Users/luo/Documents/开发笔记/images/Snip20200804_11.png)



在`spring.cloud.nacos.config.group`中指定组名，即可完成分组配置。不指定则默认使用`DEFAULT_GROUP`分组

为了演示时容易区分，`spring.profiles.active`和`spring.cloud.nacos.config.prefix`相对于`Data ID 方案`都同时做了修改

```yml
spring:
  application:
    name: nacos-config-client
  profiles:
    active: info    # 为了测试Group
  # active: dev     # 开发环境
  # active: test    # 测试环境
  cloud:
    nacos:
      # 服务发现
      discovery:
        server-addr: 127.0.0.1:8848 # nacos注册中心的地址
      # 服务配置
      config:
        server-addr: 127.0.0.1:8848 # nacos配置中心的地址
        file-extension: yaml        # 指定yml格式的配置，这里如果是yml，则nacos配置中心的对应的DataId文件扩展名也必须是yml；不可以混淆使用
      # prefix: myDataId
      # 拼接之后的文件名是： myDataId-dev.yaml  # 扩展名必须完全一致，yml 不可以 对应 yaml；
        prefix: nacos-config-client # 使用分组前缀
      # group: TEST_GROUP           # 配置分组
        group: DEV_GROUP
```

#### （6）namespace方案

每一个微服务之间相互隔离配置，每一个微服务都创建自己的命名空间，只加载自己命名空间下的所有配置

我的chrome好像创建不了命名空间，不要使用浏览器代理翻墙

![Snip20200804_12](/Users/luo/Documents/开发笔记/images/Snip20200804_12.png)

新建命名空间之后，在配置列表中多出了刚刚新建的命名空间

![Snip20200804_13](/Users/luo/Documents/开发笔记/images/Snip20200804_13.png)

记录下命名空间ID

![Snip20200804_15](/Users/luo/Documents/开发笔记/images/Snip20200804_15.png)

![Snip20200804_16](/Users/luo/Documents/开发笔记/images/Snip20200804_16.png)

在`bootstrap.yml`中指定命名空间

因为命名空间相当于java中的 `package name`，所以只要命名空间不重复，不同命名空间之间的`group`和`dataID`可以重复，这里为了省事，三个命名空间`public` `dev-namespace` `test-namespace`三个都有`data ID = nacos-config-client-info.yaml`，`group = DEV_GROUP` ，所以只需要修改命名空间，就可以切换配置文件

```yml
spring:
  application:
    name: nacos-config-client
  profiles:
    active: info    # 为了测试Group
  # active: dev     # 开发环境
  # active: test    # 测试环境
  cloud:
    nacos:
      # 服务发现
      discovery:
        server-addr: 127.0.0.1:8848 # nacos注册中心的地址
      # 服务配置
      config:
        server-addr: 127.0.0.1:8848 # nacos配置中心的地址
        file-extension: yaml        # 指定yml格式的配置，这里如果是yml，则nacos配置中心的对应的DataId文件扩展名也必须是yml；不可以混淆使用
      # prefix: myDataId
      # 拼接之后的文件名是： myDataId-dev.yaml  # 扩展名必须完全一致，yml 不可以 对应 yaml；
        prefix: nacos-config-client # 使用分组前缀
      # group: TEST_GROUP           # 配置分组
        group: DEV_GROUP
        # 指定命名空间  
        namespace: f911a501-b616-4597-bbf0-b082d515a534   # test-namespace f911a501-b616-4597-bbf0-b082d515a534
      # namespace: e9d1bc3d-2870-4b02-bdf7-b51a5c08a1a4   # dev-namespace  e9d1bc3d-2870-4b02-bdf7-b51a5c08a1a4
```

#### （7）多微服务多环境方案

![Snip20200911_1](/Users/luo/Documents/开发笔记/images/Snip20200911_1.png)

`bootstrap.yml`配置

```yaml
spring:
  profiles:
    active: dev
  application:
    name: coupon-service
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yml
        namespace: 4329c12a-5f52-479b-8600-28b7573545b8 # coupon-name-space
        group: ${spring.profiles.active} # 组名随环境的改变而改变
```

#### （8）配置集

将`application.yml`拆分成多个部分

同时加载多个配置集

* 微服务的任何配置信息，任何配置文件都可以放在配置中心中
* 只需要在`bootstrap.yml`中说明加载配置中心中哪些配置文件即可

`bootstrap.yml`

```yaml
spring:
  profiles:
    active: dev
  application:
    name: coupon-service
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yml
        namespace: 4329c12a-5f52-479b-8600-28b7573545b8 # coupon-name-space
        # 组名还可以设置为配置集中的默认组名，此时配置集中的组名可以省略
        group: ${spring.profiles.active} # 组名随环境的改变而改变
        # 所有配置集中的内容拼接好了之后组合成application.yml
        extension-configs:
          - dataId: coupon-datasource.yml
            refresh: true
            group: ${spring.profiles.active}

          - dataId: coupon-jackson.yml
            refresh: true
            group: ${spring.profiles.active}

          - dataId: coupon-service.yml
            refresh: true
            #group: ${spring.profiles.active} # 使用默认组名，可以省略
```

`coupon-datasource.yml`

![Snip20200911_2](/Users/luo/Documents/开发笔记/images/Snip20200911_2.png)

`coupon-jackson.yml`

![Snip20200911_3](/Users/luo/Documents/开发笔记/images/Snip20200911_3.png)

`coupon-service.yml`

![Snip20200911_4](/Users/luo/Documents/开发笔记/images/Snip20200911_4.png)





### 10. 集群与持久化配置

#### （1）说明

nacos默认使用嵌入式数据库实现数据的存储，如果启用多个默认配置下的`nacos`节点，数据存储是存在一致性问题的。为了解决这个问题，nacos采用了**`集中式存储的方式来支持集群化部署，目前只支持mysql的存储`**

nacos支持的三种部署模式

* 单机模式-用于测试和单机使用
* 集群模式-用于生产环境，确保高可用
* 多集群模式-用于多数据中心场景

#### （2）单机模式支持mysql

nacos默认持久化配置

* nacos默认自带的是嵌入式数据库derly
* **数据库迁移到mysql之后，原有的配置不能自动迁移**

https://nacos.io/zh-cn/docs/deployment.html

在0.7版本之前，在单机模式时，nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源的能力，具体步骤

##### 步骤

* mysql 版本 高于 5.6.5

* 使用nacos安装包自带的sql文件初始化mysql数据库，执行数据库初始文件：nacos/conf/nacos-mysql.sql

  ![Snip20200804_18](/Users/luo/Documents/开发笔记/images/Snip20200804_18.png)

执行之后（注意nacos-config数据库要自己手动创建）

![Snip20200804_19](/Users/luo/Documents/开发笔记/images/Snip20200804_19.png)

* 修改 nacos/conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql的数据源url，用户名，密码

  ```shell
  # 配置好了的效果
  [root@RabbitMQ_1 bin]# tail -n 7 /opt/nacos/conf/application.properties                                                                        
  # 配置mysql作为存储持久化配置
  spring.datasource.platform=mysql
  
  db.num=1
  # 如果是登陆本机数据库，没有使用虚拟机，则需要使用ip 127.0.0.1
  ```
# 注意改数据库的名字，默认的数据库是nacos，我的是nacos_config
```properties
  db.url.0=jdbc:mysql://192.168.2.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
  db.user=root
  db.password=orochi0208
```

如果root不能从其他机器登陆，则在数据库中执行一下

  ```  sql

  create user 'root'@'%' identified with mysql_native_password by 'orochi0208';
  
  grant all privileges on *.* to 'root'@'%' with grant option;
  
  flush privileges;
  ```

  再以单机模式启动nacos，nacos所有写嵌入式数据库的数据都写到了mysql

  ```shell
  # 最后一行提示使用外部存储
  [root@RabbitMQ_1 bin]# ./startup.sh -m standalone & tail -f /opt/nacos/logs/start.out
  2020-08-04 10:50:10,324 INFO Nacos started successfully in stand alone mode. use external storage
  ```

#### （3）集群模式部署

https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

![Snip20200804_17](/Users/luo/Documents/开发笔记/images/Snip20200804_17.png)  

##### 【1】预备环境准备

* 64 bit OS Linux/Unix/Mac，推荐使用Linux系统。
* 64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).[配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
* Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi).[配置](https://maven.apache.org/settings.html)。
* 3个或3个以上Nacos节点才能构成集群。

##### 【2】配置集群配置文件

```shell
# 快速查看本机ip
[root@RabbitMQ_1 bin]# hostname -i                                                                                                             
192.168.2.10
```

在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip:port。（请配置3个或3个以上节点）

```shell
[root@RabbitMQ_1 bin]# cp /opt/nacos/conf/cluster.conf.example /opt/nacos/conf/cluster.conf                                                    

# 修改完成之后
# 不可以配置为127.0.0.1
[root@RabbitMQ_1 bin]# tail -n 4 /opt/nacos/conf/cluster.conf
#example
192.168.2.10:3333
192.168.2.10:4444
192.168.2.10:5555
```

##### 【3】修改`start.sh`，使得启动时可以指定端口号

修改前

```shell
# :m:f:s:c:p: 对应着启动时的参数，例如 ./startsh -m standalone 就会触发m参数     
     59 while getopts ":m:f:s:c:p:" opt		
     60 do
     61     case $opt in
     62         m)
     63             MODE=$OPTARG;;
     64         f)
     65             FUNCTION_MODE=$OPTARG;;
     66         s)
     67             SERVER=$OPTARG;;
     68         c)
     69             MEMBER_LIST=$OPTARG;;
     70         p)
     71             EMBEDDED_STORAGE=$OPTARG;;
     72         ?)
     73         echo "Unknown parameter"
     74         exit 1;;
     75     esac
     76 done
     
     145 nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
```

修改后

```shell
     59 # 因为p 被占用了，所以用大写P来指定端口号
     60 while getopts ":m:f:s:c:p:P:" opt
     61 do                              
     62     case $opt in
     63         m)
     64             MODE=$OPTARG;;
     65         f)
     66             FUNCTION_MODE=$OPTARG;;
     67         s)
     68             SERVER=$OPTARG;;
     69         c)
     70             MEMBER_LIST=$OPTARG;;
     71         p)                      
     72             EMBEDDED_STORAGE=$OPTARG;;
     73         P)
     74             PORT=$OPTARG;;
     75         ?)
     76         echo "Unknown parameter"
     77         exit 1;;
     78     esac        
     79 done
     
    145 nohup $JAVA -DServer.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &                                         

```



##### 【4】启动服务器

Linux/Unix/Mac

* Stand-alone mode

```bash
sh startup.sh -m standalone
```

* 集群模式

> 使用内置数据源

```bash
sh startup.sh -p embedded
```

> 使用外置数据源

```bash
sh startup.sh
```

```shell
# 注意，三个集群因为全部文件完全共享，所以不能短时间内同时启动三台nacos，必须等待上一个启动完成之后，再启动下一个
[root@RabbitMQ_1 bin]# ./startup.sh -P 3333 & tail -f ../logs/nacos.log

[root@RabbitMQ_1 bin]# ./startup.sh -P 4444 & tail -f ../logs/nacos.log

[root@RabbitMQ_1 bin]# ./startup.sh -P 5555 & tail -f ../logs/nacos.log
```

![Snip20200805_20](/Users/luo/Documents/开发笔记/images/Snip20200805_20.png)

##### 【5】修改nginx

```shell
    upstream nacos-cluster{
        server 192.168.2.10:3333;
        server 192.168.2.10:4444;
        server 192.168.2.10:5555;
    }

    server {
        listen       2222;
        server_name  192.168.2.10;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
            proxy_pass http://nacos-cluster;
        }
    }
```

##### 【6】启动nginx

```shell
# 创建路径
[root@RabbitMQ_1 sbin]# mkdir -p /var/temp/nginx/ 

[root@RabbitMQ_1 nginx-1.18.0]# pwd
/opt/nginx/nginx-1.18.0

# 配置nginx安装路径
[root@RabbitMQ_1 nginx-1.18.0]# ./configure --prefix=/opt/nginx/runable

# 编译
[root@RabbitMQ_1 nginx-1.18.0]# make  

# 安装
[root@RabbitMQ_1 nginx-1.18.0]# make install  

# 切换到nginx安装目录
[root@RabbitMQ_1 sbin]# pwd                                                                                                                    
/opt/nginx/runable/sbin

# 指定配置文件运行nginx
[root@RabbitMQ_1 sbin]# ./nginx -c /opt/nginx/runable/conf/nginx.conf
```

##### 【7】通过nginx访问nacos集群

http://192.168.2.10:2222/nacos

![Snip20200805_21](/Users/luo/Documents/开发笔记/images/Snip20200805_21.png)







## 三、熔断与限流

Sentinel: 分布式系统的流量防卫兵，代替Netflix 的 Hystrix

### 1. Sentinel 是什么？

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

* **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
* **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
* **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 的主要特性：

![Sentinel 的主要特性](/Users/luo/Documents/开发笔记/images/Sentinel 的主要特性.png)

[https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D](https://github.com/alibaba/Sentinel/wiki/介绍)

![Snip20200805_22](/Users/luo/Documents/开发笔记/images/Snip20200805_22.png)

Sentinel 的开源生态：

![Sentinel 的开源生态](/Users/luo/Documents/开发笔记/images/Sentinel 的开源生态.png)

Sentinel 分为两个部分:

* 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
* 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

解决的问题：

* 服务雪崩
* 服务降级
* 服务熔断
* 服务限流

### 2. 运行

```shell
# Sentinel-dashbord 默认监听8080端口，可以-Dserver.port=8888 来指定端口
luo@luodeMacBook-Pro sentinel % java -jar -Dserver.port=8888 sentinel-dashboard-1.7.2.jar

# 官方启动方法
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar

# 用户名和密码都是 sentinel
```

### 3. 懒加载说明

配置好之后，需要`sentinel`客户端被访问过至少一次，`sentinel dashboard`上面才会监控到数据

### 4. 流控规则

https://github.com/alibaba/Sentinel/wiki/集群流控

**流量控制**（flow control），其原理是监控应用流量的 QPS 或并发线程数等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

`FlowSlot` 会根据预设的规则，结合前面 `NodeSelectorSlot`、`ClusterNodeBuilderSlot`、`StatisticSlot` 统计出来的实时信息进行流量控制。

限流的直接表现是在执行 `Entry nodeA = SphU.entry(resourceName)` 的时候抛出 `FlowException` 异常。`FlowException`是 `BlockException` 的子类，您可以捕捉 `BlockException` 来自定义被限流之后的处理逻辑。

同一个资源可以创建多条限流规则。`FlowSlot` 会对该资源的所有限流规则依次遍历，直到有规则触发限流或者所有规则遍历完毕。

一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：

* `resource`：资源名，即限流规则的作用对象，默认请求路径
* `count`: 限流阈值
* `grade`: 限流阈值类型（QPS 或并发线程数）
* `limitApp`: 流控针对的调用来源，若为 `default` 则不区分调用来源
* `strategy`: 调用关系限流策略
* `controlBehavior`: 流量控制效果（直接拒绝、Warm Up、匀速排队）

![Snip20200805_2](/Users/luo/Documents/开发笔记/images/Snip20200805_2.png)

* 资源名：唯一名称，默认请求路径
* 针对来源：sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）
* 阈值类型/单机阈值
  * QPS：每秒钟的请求数量，当调用该api的QPS达到阈值的时候，进行限流
  * 线程数：当调用该api的线程数达到阈值的时候，进行限流
* 是否集群：不需要集群
* 流控模式：
  * 直接：api达到限流条件时，直接限流
  * 关联：当关联的资源达到阈值时，就限流自己
  * 链路：只记录指定链路上的流量，指定资源从入口资源进来的流量，如果达到阈值，就进行限流（api级别的针对来源）
* 流控效果：（详见4.1.2 QPS流量控制）
  * 快速失败：直接失败，抛出异常
  * Warm up：根据codeFactor（冷加载因子，默认3）的值，从阈值/condeFactor，经过预热时长，才达到设置的QPS阈值
  * 排队等待：

#### 4.1 基于QPS/并发数的流量控制

流量控制主要有两种统计类型，一种是统计并发线程数，另外一种则是统计 QPS。类型由 `FlowRule` 的 `grade` 字段来定义。其中，0 代表根据并发数量来限流，1 代表根据 QPS 来进行流量控制。其中线程数、QPS 值，都是由 `StatisticSlot` 实时统计获取的。

我们可以通过下面的命令查看实时统计信息：

```
curl http://localhost:8719/cnode?id=resourceName
```

输出内容格式如下：

```
idx id     thread  pass  blocked   success  total Rt   1m-pass   1m-block   1m-all   exception
2   abc647    0     46      0         46      46   1     2763       0         2763     0
```

其中：

* thread： 代表当前处理该资源的并发数；
* pass： 代表一秒内到来到的请求；
* blocked： 代表一秒内被流量控制的请求数量；
* success： 代表一秒内成功处理完的请求；
* total： 代表到一秒内到来的请求以及被阻止的请求总和；
* RT： 代表一秒内该资源的平均响应时间；
* 1m-pass： 则是一分钟内到来的请求；
* 1m-block： 则是一分钟内被阻止的请求；
* 1m-all： 则是一分钟内到来的请求和被阻止的请求的总和；
* exception： 则是一秒内业务本身异常的总和。

##### 4.1.1 并发线程数控制

并发数控制用于保护业务线程池不被慢调用耗尽。例如，当应用所依赖的下游应用由于某种原因导致服务不稳定、响应延迟增加，对于调用者来说，意味着吞吐量下降和更多的线程数占用，极端情况下甚至导致线程池耗尽。为应对太多线程占用的情况，业内有使用隔离的方案，比如通过不同业务逻辑使用不同线程池来隔离业务自身之间的资源争抢（线程池隔离）。这种隔离方案虽然隔离性比较好，但是代价就是线程数目太多，线程上下文切换的 overhead 比较大，特别是对低延时的调用有比较大的影响。Sentinel 并发控制不负责创建和管理线程池，而是简单统计当前请求上下文的线程数目（正在执行的调用数目），如果超出阈值，新的请求会被立即拒绝，效果类似于信号量隔离。

例子参见：[ThreadDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/FlowThreadDemo.java)

![Snip20200805_3](/Users/luo/Documents/开发笔记/images/Snip20200805_3.png)

假设线程数为1，所有的请求都可以进入微服务，但是同一时刻只有一个线程为一个请求提供服务，如果此时有其他请求也访问微服务，则这个请求因为没有空闲的线程为其服务，而会被拒绝

##### 4.1.2 QPS流量控制

当 QPS 超过某个阈值的时候，则采取措施进行流量控制。流量控制的效果包括以下几种：**直接拒绝**、**Warm Up**、**匀速排队**。对应 `FlowRule` 中的 `controlBehavior` 字段。

> 注意：若使用除了直接拒绝之外的流量控制效果，则调用关系限流策略（strategy）会被忽略。

###### 直接拒绝（快速失败）

![Snip20200805_2](/Users/luo/Documents/开发笔记/images/Snip20200805_2.png)

**直接拒绝**（`RuleConstant.CONTROL_BEHAVIOR_DEFAULT`）方式是默认的流量控制方式，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出`FlowException`。这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位时。具体的例子参见 [FlowQpsDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/FlowQpsDemo.java)。源码`com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController`

###### Warm Up

Warm Up（`RuleConstant.CONTROL_BEHAVIOR_WARM_UP`）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。详细文档可以参考 [流量控制 - Warm Up 文档](https://github.com/alibaba/Sentinel/wiki/限流---冷启动)，具体的例子可以参见 [WarmUpFlowDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/WarmUpFlowDemo.java)。

`com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController`  

```java
public class WarmUpController implements TrafficShapingController {
    
  	
		public WarmUpController(double count, int warmUpPeriodInSec) {
        this.construct(count, warmUpPeriodInSec, 3); //coldFactor 默认 为 3
    }
}
```

通常冷启动的过程系统允许通过的 QPS 曲线如下图所示：

![image](/Users/luo/Documents/开发笔记/images/warmUp.png)

公式：阈值除以coldFactor（默认为3），经过预热时长后才会达到阈值

![Snip20200805_10](/Users/luo/Documents/开发笔记/images/Snip20200805_10.png)

上图：最开始的阈值 = 单机阈值10  除以 coldFactor（默认为3）= 3，即最开始的阈值是3，经过预热时长5秒之后，达到最大阈值，也就是单机阈值10。

应用场景：秒杀系统再开启的瞬间，会有很多流量上来，很可能会把系统打死，预热方式就是为了保护系统，可慢慢的把流量放进来，慢慢的把阈值增长到设置的阈值。

***

###### 匀速排队

匀速排队（`RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。详细文档可以参考 [流量控制 - 匀速器模式](https://github.com/alibaba/Sentinel/wiki/流量控制-匀速排队模式)，具体的例子可以参见 [PaceFlowDemo](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/PaceFlowDemo.java)。

该方式的作用如下图所示：

![image](/Users/luo/Documents/开发笔记/images/匀速排队.png)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。

![Snip20200805_12](/Users/luo/Documents/开发笔记/images/Snip20200805_12.png)

`com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController`

匀速排队：让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效。

![Snip20200805_11](/Users/luo/Documents/开发笔记/images/Snip20200805_11.png)

设置含义：/testA 每秒1次请求，超过的话就排队等待，等待的超时时间为20000ms

***

#### 4.2 基于调用关系的流量控制

调用关系包括调用方、被调用方；一个方法又可能会调用其它方法，形成一个调用链路的层次关系。Sentinel 通过 `NodeSelectorSlot` 建立不同资源间的调用的关系，并且通过 `ClusterNodeBuilderSlot` 记录每个资源的实时统计信息。

有了调用链路的统计信息，我们可以衍生出多种流量控制手段。

##### 4.2.1 根据调用方限流

`ContextUtil.enter(resourceName, origin)` 方法中的 `origin` 参数标明了调用方身份。这些信息会在 `ClusterBuilderSlot` 中被统计。可通过以下命令来展示不同的调用方对同一个资源的调用数据：

```
curl http://localhost:8719/origin?id=nodeA
```

调用数据示例：

```
id: nodeA
idx origin  threadNum passedQps blockedQps totalQps aRt   1m-passed 1m-blocked 1m-total 
1   caller1 0         0         0          0        0     0         0          0
2   caller2 0         0         0          0        0     0         0          0
```

上面这个命令展示了资源名为 `nodeA` 的资源被两个不同的调用方调用的统计。

流控规则中的 `limitApp` 字段用于根据调用来源进行流量控制。该字段的值有以下三种选项，分别对应不同的场景：

* `default`：表示不区分调用者，来自任何调用者的请求都将进行限流统计。如果这个资源名的调用总和超过了这条规则定义的阈值，则触发限流。
* `{some_origin_name}`：表示针对特定的调用者，只有来自这个调用者的请求才会进行流量控制。例如 `NodeA` 配置了一条针对调用者`caller1`的规则，那么当且仅当来自 `caller1` 对 `NodeA` 的请求才会触发流量控制。
* `other`：表示针对除 `{some_origin_name}` 以外的其余调用方的流量进行流量控制。例如，资源`NodeA`配置了一条针对调用者 `caller1` 的限流规则，同时又配置了一条调用者为 `other` 的规则，那么任意来自非 `caller1` 对 `NodeA` 的调用，都不能超过 `other` 这条规则定义的阈值。

同一个资源名可以配置多条规则，规则的生效顺序为：**{some_origin_name} > other > default**

##### 4.2.2 根据调用链路入口限流：链路限流

`NodeSelectorSlot` 中记录了资源之间的调用链路，这些资源通过调用关系，相互之间构成一棵调用树。这棵树的根节点是一个名字为 `machine-root` 的虚拟节点，调用链的入口都是这个虚节点的子节点。

一棵典型的调用树如下图所示：

```
     	          machine-root
                    /       \
                   /         \
             Entrance1     Entrance2
                /             \
               /               \
      DefaultNode(nodeA)   DefaultNode(nodeA)
```

上图中来自入口 `Entrance1` 和 `Entrance2` 的请求都调用到了资源 `NodeA`，Sentinel 允许只根据某个入口的统计信息对资源限流。比如我们可以设置 `strategy` 为 `RuleConstant.STRATEGY_CHAIN`，同时设置 `refResource` 为 `Entrance1` 来表示只有从入口 `Entrance1` 的调用才会记录到 `NodeA` 的限流统计当中，而不关心经 `Entrance2` 到来的调用。

调用链的入口（上下文）是通过 API 方法 `ContextUtil.enter(contextName)` 定义的，其中 contextName 即对应调用链路入口名称。详情可以参考 [ContextUtil 文档](https://github.com/alibaba/Sentinel/wiki/如何使用#上下文工具类-contextutil)。

一句话：**多个请求调用了同一个微服务**

##### 4.2.3 具有关系的资源流量控制：关联流量控制

当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联。比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。如果放任读写操作争抢资源，则争抢本身带来的开销会降低整体的吞吐量。可使用关联限流来避免具有关联关系的资源之间过度的争抢，举例来说，`read_db` 和 `write_db` 这两个资源分别代表数据库读写，我们可以给 `read_db` 设置限流规则来达到写优先的目的：设置 `strategy` 为 `RuleConstant.STRATEGY_RELATE` 同时设置 `refResource` 为 `write_db`。这样当写库操作过于频繁时，读数据的请求会被限流。

也就是说：**当关联的资源达到阈值的时候，就限流自己。当与A关联的资源B达到阈值之后，就限流A自己**。B惹事，A挂了

![Snip20200805_4](/Users/luo/Documents/开发笔记/images/Snip20200805_4.png)

大批量的访问`/testB`，最终导致`/testA`挂了

![Snip20200805_7](/Users/luo/Documents/开发笔记/images/Snip20200805_7.png)

![Snip20200805_6](/Users/luo/Documents/开发笔记/images/Snip20200805_6.png)

### 5. 降级

#### （1）概述

除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。由于调用关系的复杂性，如果调用链路中的某个资源不稳定，最终会导致请求发生堆积。Sentinel **熔断降级**会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 `DegradeException`）。

**Sentinel 的断路器没有半开状态**



#### （2）降级策略

![Snip20200805_13](/Users/luo/Documents/开发笔记/images/Snip20200805_13.png)

RT（平均响应时间，秒级）

* 平均响应时间`超出阈值`且`在时间窗口内通过的请求>5`两个条件同时满足后，触发降级
* 窗口期过后关闭断路器
* RT最大4900（更大需要通过-Dcsp.sentinel.statistic.max.rt=XXX才能生效）

异常比例（秒级）

* QPS >= 5 且 `异常比例（秒级统计）超过阈值时`，触发降级
* 时间窗口期结束后，关闭降级

异常数（分钟级）

* 异常数（分钟统计）超过阈值时，触发降级
* 时间窗口结束后，关闭降级

#### （3）官方降级策略解释

我们通常用以下几种方式来衡量资源是否处于稳定的状态：

##### 【1】平均响应时间 

* 平均响应时间 (`DEGRADE_GRADE_RT`)：当 1s 内持续进入 N 个请求，对应时刻的平均响应时间（秒级）均超过阈值（`count`，以 ms 为单位），那么在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 `DegradeException`）。注意 Sentinel 默认统计的 RT 上限是 4900 ms，**超出此阈值的都会算作 4900 ms**，若需要变更此上限可以通过启动配置项 `-Dcsp.sentinel.statistic.max.rt=xxx` 来配置。

  ![Snip20200805_1](/Users/luo/Documents/开发笔记/images/Snip20200805_1.png)



![Snip20200805_5](/Users/luo/Documents/开发笔记/images/Snip20200805_5.png)

`/testA`的请求响应时间不能大于200ms，如果大于200ms，接下来的时间窗口1s内就熔断`/testA`，1s过后，恢复正常，重新提供服务

```java
    @GetMapping("/testA")
    public String testA() {


        String name = Thread.currentThread().getName();
        log.info(name + "正在处理 /testA ...");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //log.info(name + "处理完成");
        return "你好---testA";
    }
```

测试，无限循环，每秒有10个线程访问`/testA`，超过5个请求（满足条件一），所以处理一个请求要`1s`超过了响应时长RT`200ms`（满足条件二），这时，浏览器再访问`\testA`就会被拒绝服务。我们希望`200ms`内处理完本次任务，如果`200ms`内没有处理完，在未来`1s`钟的时间窗口期内，断路器打开（保险丝跳闸），微服务不可用。

当jmeter停止之后，请求数量小于5个，条件一不满足，就能恢复访问了

![Snip20200805_9](/Users/luo/Documents/开发笔记/images/Snip20200805_9.png)

***

##### 【2】异常比例

* 异常比例 (`DEGRADE_GRADE_EXCEPTION_RATIO`)：当资源的每秒请求量 >= N（可配置），并且每秒异常总数占通过量的比值超过阈值（`DegradeRule` 中的 `count`）之后，资源进入降级状态，即在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。

  ![Snip20200805_14](/Users/luo/Documents/开发笔记/images/Snip20200805_14.png)

![Snip20200805_15](/Users/luo/Documents/开发笔记/images/Snip20200805_15.png)

```java
    // 测试异常比例
    @GetMapping("/testExceptionRate")
    public String testExceptionRate() {
        
        int a = 10 / 0;

        return "ok";
    }
```



异常比例超过`20%`（条件一），并且`每秒请求数量>=5`（条件二），在接下来的`3s`内触发熔断，3秒过后继续提供服务

![Snip20200805_17](/Users/luo/Documents/开发笔记/images/Snip20200805_17.png)

![Snip20200805_18](/Users/luo/Documents/开发笔记/images/Snip20200805_18.png)

按照上述配置，单独访问一次，必然来一次报错一次（int a = 10/0;），调用一次错一次

开启jmeter之后，直接高并发发送请求，多次调用达到了我们配置的条件了（错误率>20%），断路器开启，微服务降级了，此时访问不再报错，是因为服务降级了

***

##### 【3】异常数

* 异常数 (`DEGRADE_GRADE_EXCEPTION_COUNT`)：当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 `timeWindow` 小于 60s，则结束熔断状态后仍可能再进入熔断状态。
* **时间窗口一定要>=60s**

![Snip20200805_19](/Users/luo/Documents/开发笔记/images/Snip20200805_19.png)

![Snip20200805_23](/Users/luo/Documents/开发笔记/images/Snip20200805_23.png)

验证：第一次访问`http://localhost:8401/testExceptionRate`，绝对报错，因为`int a = 10/0`，我们看到error窗口，但是5次报错之后，进入熔断后降级。时间窗口`70s`之后，熔断关闭

注意：异常降级仅针对业务异常，对 Sentinel 限流降级本身的异常（`BlockException`）不生效。为了统计异常比例或异常数，需要通过 `Tracer.trace(ex)` 记录业务异常。示例：

```
Entry entry = null;
try {
  entry = SphU.entry(key, EntryType.IN, key);

  // Write your biz code here.
  // <<BIZ CODE>>
} catch (Throwable t) {
  if (!BlockException.isBlockException(t)) {
    Tracer.trace(t);
  }
} finally {
  if (entry != null) {
    entry.exit();
  }
}
```

开源整合模块，如 Sentinel Dubbo Adapter, Sentinel Web Servlet Filter 或 `@SentinelResource` 注解会自动统计业务异常，无需手动调用。





### 6. 热点key限流

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

* 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
* 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![sentinel-hot-param-overview-1](/Users/luo/Documents/开发笔记/images/sentinel-hot-param-overview-1.png)

Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。



#### （1）基本使用

兜底方法，分为系统默认和客户端自定义，两种

* 之前的例子中，限流出问题之后，都用`sentinel`系统默认的提示：`Blocked by Sentinel(flow limiting)`
* 我们能不能自定义类似`Hystrix`，当某个方法出问题了，就找对应的兜底降级方法
* **从 `@HystrixCommand`到`@SentinelResource`**

* `com.alibaba.csp.sentinel.slots.block.BlockException`

![Snip20200805_24](/Users/luo/Documents/开发笔记/images/Snip20200805_24.png)

配置说明：`@SentinelResource(value = "hotKeyLimiting",blockHandler = "hotKeyLimiting_blockHandler")`注解对应方法中`第0个参数`（也就是这里的p1）的第一个value超过QPS的单机阈值`1`时，就会报`blockHandler`指定的错误。

注意：资源名对应的是`@SentinelResource`注解中的值，不是`@GetMapping`，不能添加`/`

而且建议`blockHandler`属性一定要添加（兜底方法一定要指定），不然抛出的异常信息会直接返回给前台，不友好

注意2:

* `@SentinelResource`处理的是Sentinel控制台配置的违规情况，由`blockHandler`配置的方法负责兜底处理
* `RuntimeException ` `int a = 10/0`，java运行时异常，`@SentinelResource`不管
* `@SentinelResource`主管配置出错，运行出错该走异常，就走异常

```java
    /**
     * 热点限流
     *
     * @SentinelResource 的value可以是任意，但是要求唯一，为了方便记忆，这里与 @GetMapping 相同，但是没有 斜杠 /
     * 之后在Sentinel中进行配置的时候，/hotKeyLimiting 与 hotKeyLimiting 走的限流规则就分别对应着这里的 @GetMapping @SentinelResource
     * blockHandler 属性和 @HystrixCommand 中的 fallbackMethod 一样，可以指定一个兜底的方法。
     * 区别是，blockHandler 配置之后，只要满足Sentinel dashBoard 中的hot key 限流规则，就会触发其指定的兜底方法
     * 也就是说，本方法抛出的异常 blockHandler不管，blockHandler只管 sentinel dashBoard 的指令
     */
    @GetMapping("/hotKeyLimiting")
    @SentinelResource(value = "hotKeyLimiting", blockHandler = "hotKeyLimiting_blockHandler")
    public String hotKeyLimiting(@RequestParam(value = "p1", required = false) String p1,
                                 @RequestParam(value = "p2", required = false) String p2) {

        log.info(p1 + "\t" + p2);
        
        int a = 10/0; //方法中抛出的异常，不会跳到 blockHandler 指定的方法中，Sentinel 才不管
        
        return "hotKeyLimiting--------ok";

    }

    public String hotKeyLimiting_blockHandler(String p1, String p2, BlockException blockException) {

        return "我是hotKeyLimiting_blockHandler兜底方法，😭😭"; // 默认使用系统的提示：Blocked by Sentinel(flow limiting)
    }
```

测试链接：

* http://localhost:8401/hotKeyLimiting?p1=hello&p2=bye
* http://localhost:8401/hotKeyLimiting?p1=hello
* 其余链接不会出发hot key限流

#### （2）参数额外项

上面我们已经实现了的功能：超过 `1s`一个请求之后，因为达到阈值`1，`所以马上被限流

我们希望添加：`p1`参数当它是某个特殊值的时候，他的限流值和平时的`1`不一样。例如当`p1==5`的时候，他的限流阈值可以达到`200`

注意：参数类型必须和方法的参数类型一致，否则无效；一定要记得点绿色的`添加`按钮，否则无效

![Snip20200805_26](/Users/luo/Documents/开发笔记/images/Snip20200805_26.png)

结论：当`p1 ==5`时，阈值变为`200`

​		  当`p1!=5`时，阈值为`1`

#### （3）官方文档

要使用热点参数限流功能，需要引入以下依赖：

```xml
<!--Sentinel 启动器已经集成了-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-parameter-flow-control</artifactId>
    <version>x.y.z</version>
</dependency>
```

然后为对应的资源配置热点参数限流规则，并在 `entry` 的时候传入相应的参数，即可使热点参数限流生效。

> 注：若自行扩展并注册了自己实现的 `SlotChainBuilder`，并希望使用热点参数限流功能，则可以在 chain 里面合适的地方插入 `ParamFlowSlot`。

那么如何传入对应的参数以便 Sentinel 统计呢？我们可以通过 `SphU` 类里面几个 `entry` 重载方法来传入：

```java
public static Entry entry(String name, EntryType type, int count, Object... args) throws BlockException

public static Entry entry(Method method, EntryType type, int count, Object... args) throws BlockException
```

其中最后的一串 `args` 就是要传入的参数，有多个就按照次序依次传入。比如要传入两个参数 `paramA` 和 `paramB`，则可以：

```
// paramA in index 0, paramB in index 1.
// 若需要配置例外项或者使用集群维度流控，则传入的参数只支持基本类型。
SphU.entry(resourceName, EntryType.IN, 1, paramA, paramB);
```

**注意**：若 entry 的时候传入了热点参数，那么 exit 的时候也一定要带上对应的参数（`exit(count, args)`），否则可能会有统计错误。正确的示例：

```java
Entry entry = null;
try {
    entry = SphU.entry(resourceName, EntryType.IN, 1, paramA, paramB);
    // Your logic here.
} catch (BlockException ex) {
    // Handle request rejection.
  	// Sentinel 默认就是从这里抛出 Blocked by Sentinel(flow limiting) 异常给浏览器的
} finally {
    if (entry != null) {
        entry.exit(1, paramA, paramB);
    }
}
```

对于 `@SentinelResource` 注解方式定义的资源，若注解作用的方法上有参数，Sentinel 会将它们作为参数传入 `SphU.entry(res, args)`。比如以下的方法里面 `uid` 和 `type` 会分别作为第一个和第二个参数传入 Sentinel API，从而可以用于热点规则判断：

```
@SentinelResource("myMethod")
public Result doSomething(String uid, int type) {
  // some logic here...
}
```



#### （3）热点参数规则



热点参数规则（`ParamFlowRule`）类似于流量控制规则（`FlowRule`）：

| 属性              | 说明                                                         | 默认值   |
| ----------------- | ------------------------------------------------------------ | -------- |
| resource          | 资源名，必填                                                 |          |
| count             | 限流阈值，必填                                               |          |
| grade             | 限流模式                                                     | QPS 模式 |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持             | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持   | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持 | 0ms      |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | `false`  |
| clusterConfig     | 集群流控相关配置                                             |          |

我们可以通过 `ParamFlowRuleManager` 的 `loadRules` 方法更新热点参数规则，下面是一个示例：

```
ParamFlowRule rule = new ParamFlowRule(resourceName)
    .setParamIdx(0)
    .setCount(5);
// 针对 int 类型的参数 PARAM_B，单独设置限流 QPS 阈值为 10，而不是全局的阈值 5.
ParamFlowItem item = new ParamFlowItem().setObject(String.valueOf(PARAM_B))
    .setClassType(int.class.getName())
    .setCount(10);
rule.setParamFlowItemList(Collections.singletonList(item));

ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
```

### 7. 系统自适应限流

**不推荐使用，过低的QPS配置相当于系统不可用**

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

![Snip20200805_27](/Users/luo/Documents/开发笔记/images/Snip20200805_27.png)

#### （1）背景

在开始之前，我们先了解一下系统保护的目的：

* 保证系统不被拖垮
* 在系统稳定的前提下，保持系统的吞吐量

长期以来，系统保护的思路是根据硬指标，即系统的负载 (load1) 来做系统过载保护。当系统负载高于某个阈值，就禁止或者减少流量的进入；当 load 开始好转，则恢复流量的进入。这个思路给我们带来了不可避免的两个问题：

* load 是一个“结果”，如果根据 load 的情况来调节流量的通过率，那么就始终有延迟性。也就意味着通过率的任何调整，都会过一段时间才能看到效果。当前通过率是使 load 恶化的一个动作，那么也至少要过 1 秒之后才能观测到；同理，如果当前通过率调整是让 load 好转的一个动作，也需要 1 秒之后才能继续调整，这样就浪费了系统的处理能力。所以我们看到的曲线，总是会有抖动。
* 恢复慢。想象一下这样的一个场景（真实），出现了这样一个问题，下游应用不可靠，导致应用 RT 很高，从而 load 到了一个很高的点。过了一段时间之后下游应用恢复了，应用 RT 也相应减少。这个时候，其实应该大幅度增大流量的通过率；但是由于这个时候 load 仍然很高，通过率的恢复仍然不高。

[TCP BBR](https://en.wikipedia.org/wiki/TCP_congestion_control#TCP_BBR) 的思想给了我们一个很大的启发。我们应该根据系统能够处理的请求，和允许进来的请求，来做平衡，而不是根据一个间接的指标（系统 load）来做限流。最终我们追求的目标是 **在系统不被拖垮的情况下，提高系统的吞吐率，而不是 load 一定要到低于某个阈值**。如果我们还是按照固有的思维，超过特定的 load 就禁止流量进入，系统 load 恢复就放开流量，这样做的结果是无论我们怎么调参数，调比例，都是按照果来调节因，都无法取得良好的效果。

Sentinel 在系统自适应保护的做法是，用 load1 作为启动自适应保护的因子，而允许通过的流量由处理请求的能力，即请求的响应时间以及当前系统正在处理的请求速率来决定。

#### （2）系统规则

系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

* **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
* **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
* **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
* **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
* **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

#### （3）原理

先用经典图来镇楼:

![系统自适应限流 ](/Users/luo/Documents/开发笔记/images/系统自适应限流 .png)

我们把系统处理请求的过程想象为一个水管，到来的请求是往这个水管灌水，当系统处理顺畅的时候，请求不需要排队，直接从水管中穿过，这个请求的RT是最短的；反之，当请求堆积的时候，那么处理请求的时间则会变为：排队时间 + 最短处理时间。

* 推论一: 如果我们能够保证水管里的水量，能够让水顺畅的流动，则不会增加排队的请求；也就是说，这个时候的系统负载不会进一步恶化。

我们用 T 来表示(水管内部的水量)，用RT来表示请求的处理时间，用P来表示进来的请求数，那么一个请求从进入水管道到从水管出来，这个水管会存在 `P * RT`　个请求。换一句话来说，当 `T ≈ QPS * Avg(RT)` 的时候，我们可以认为系统的处理能力和允许进入的请求个数达到了平衡，系统的负载不会进一步恶化。

接下来的问题是，水管的水位是可以达到了一个平衡点，但是这个平衡点只能保证水管的水位不再继续增高，但是还面临一个问题，就是在达到平衡点之前，这个水管里已经堆积了多少水。如果之前水管的水已经在一个量级了，那么这个时候系统允许通过的水量可能只能缓慢通过，RT会大，之前堆积在水管里的水会滞留；反之，如果之前的水管水位偏低，那么又会浪费了系统的处理能力。

* 推论二:　当保持入口的流量是水管出来的流量的最大的值的时候，可以最大利用水管的处理能力。

然而，和 TCP BBR 的不一样的地方在于，还需要用一个系统负载的值（load1）来激发这套机制启动。

> 注：这种系统自适应算法对于低 load 的请求，它的效果是一个“兜底”的角色。**对于不是应用本身造成的 load 高的情况（如其它进程导致的不稳定的情况），效果不明显。**

### 8.  自定义限流处理逻辑

#### （1）按照资源名称进行限流

按照`@SentinelResource(value = "byResource", blockHandler = "byResource_handler")`的资源名称进行限流

```java
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource", blockHandler = "byResource_handler")
    public String byResource() {
        return "一切ok";
    }

    public String byResource_handler(BlockException blockException) {

        log.info(blockException.toString());

        return "异常：" + blockException.toString();
    }
```



![Snip20200805_28](/Users/luo/Documents/开发笔记/images/Snip20200805_28.png)

#### （2）通过访问的URL来限流

通过访问的URL来限流，会返回`Sentinel`自带默认的限流处理信息

```java
    @GetMapping("/service/byUrl")
    @SentinelResource(value = "byUrl")
    public String byUrl() {
        return "/service/byUrl-----ok";
    }
```

按照URL进行限流

![Snip20200805_29](/Users/luo/Documents/开发笔记/images/Snip20200805_29.png)

#### （3）自定义限流处理逻辑

##### 【1】创建自定义限流类

```java
@Slf4j
public class MyBlockHandler {

    /**
     * 必须是静态方法
     */
    public static String customizeGlobalHandler_1(BlockException blockException) {

        log.info(blockException.toString());

        return "customizeGlobalHandler_1，异常：" + blockException.toString() + "😭😭😭";
    }

    public static String customizeGlobalHandler_2(BlockException blockException) {

        log.info(blockException.toString());

        return "customizeGlobalHandler_2，异常：" + blockException.toString() + "😭😭😭";
    }
}
```

##### 【2】在controller中指定自定义的兜底方法

```java
    /**
     * blockHandlerClass 指定 blockHandler方法所在的类，
     * 有了 blockHandlerClass 之后，blockHandler指定的就是在 blockHandlerClass 中，选择方法作为 blockHandler方法
     */
    @GetMapping("/service/byUrl")
    @SentinelResource(value = "byUrl", blockHandlerClass = MyBlockHandler.class, blockHandler = "customizeGlobalHandler_1")
    public String byUrl() {
        return "/service/byUrl-----ok";
    }
```

##### 【3】在sentinel中指定限流规则

![Snip20200806_30](/Users/luo/Documents/开发笔记/images/Snip20200806_30.png)

### 9. 注解支持

Sentinel 提供了 `@SentinelResource` 注解用于定义资源，并提供了 AspectJ 的扩展用于自动定义资源、处理 `BlockException` 等。使用 [Sentinel Annotation AspectJ Extension](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-annotation-aspectj) 的时候需要引入以下依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
    <version>x.y.z</version>
</dependency>
```

#### （1）@SentinelResource 注解

注意：注解方式埋点不支持`private`方法

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

* `value`：资源名称，必需项（不能为空）

* `entryType`：entry 类型，可选项（默认为 `EntryType.OUT`）

* `blockHandler` / `blockHandlerClass`: `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 `BlockException`。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

* ```
  fallback
  ```

  ：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了

   

  ```
  exceptionsToIgnore
  ```

   

  里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：

  * 返回值类型必须与原函数返回值类型一致；
  * 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  * fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

* ```
  defaultFallback
  ```

  （since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所以类型的异常（除了

   

  ```
  exceptionsToIgnore
  ```

   

  里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：

  * 返回值类型必须与原函数返回值类型一致；
  * 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  * defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

* `exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

> 注：1.6.0 之前的版本 fallback 函数只针对降级异常（`DegradeException`）进行处理，**不能针对业务异常进行处理**。

特别地，若 blockHandler 和 fallback 都进行了配置，则被限流降级而抛出 `BlockException` 时只会进入 `blockHandler` 处理逻辑。若未配置 `blockHandler`、`fallback` 和 `defaultFallback`，则被限流降级时会将 `BlockException` **直接抛出**。

示例：

```java
public class TestService {

    // 对应的 `handleException` 函数需要位于 `ExceptionUtil` 类中，并且必须为 static 函数.
    @SentinelResource(value = "test", blockHandler = "handleException", blockHandlerClass = {ExceptionUtil.class})
    public void test() {
        System.out.println("Test");
    }

    // 原函数
    @SentinelResource(value = "hello", blockHandler = "exceptionHandler", fallback = "helloFallback")
    public String hello(long s) {
        return String.format("Hello at %d", s);
    }
    
    // Fallback 函数，函数签名与原函数一致或加一个 Throwable 类型的参数.
    public String helloFallback(long s) {
        return String.format("Halooooo %d", s);
    }

    // Block 异常处理函数，参数最后多一个 BlockException，其余与原函数一致.
    public String exceptionHandler(long s, BlockException ex) {
        // Do some log here.
        ex.printStackTrace();
        return "Oops, error occurred at " + s;
    }
}
```

从 1.4.0 版本开始，注解方式定义资源支持自动统计业务异常，无需手动调用 `Tracer.trace(ex)` 来记录业务异常。Sentinel 1.4.0 以前的版本需要自行调用 `Tracer.trace(ex)` 来记录业务异常。

#### （2）配置

#### （3）AspectJ

若您的应用直接使用了 AspectJ，那么您需要在 `aop.xml` 文件中引入对应的 Aspect：

```xml
<aspects>
    <aspect name="com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect"/>
</aspects>
```

#### （4）Spring AOP

若您的应用使用了 Spring AOP，您需要通过配置的方式将 `SentinelResourceAspect` 注册为一个 Spring Bean：

```java
@Configuration
public class SentinelAspectConfiguration {

    @Bean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }
}
```

我们提供了 Spring AOP 的示例，可以参见 [sentinel-demo-annotation-spring-aop](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-annotation-spring-aop)。er/sentinel-demo/sentinel-demo-annotation-spring-aop)。

### 10. 服务熔断

#### （1）使用`sentinel`自带注解完成熔断降级

使用`@SentinelResource(value = "getPaymentById", fallback = "fallbackHandler")`的`fallback`进行服务降级，`fallback`用于接收方法抛出的运行时异常，对其进行降级处理

作用和`@HystrixCommand(fallback="fallbackHandler")`一致

```java
    @GetMapping("/getPaymentById/{paymentId}")
    @SentinelResource(value = "getPaymentById", fallback = "fallbackHandler")
    public Payment getPaymentById(@PathVariable String paymentId) {

        Payment forObject = this.restTemplate.getForObject("http://" + PaymentServiceName + "/getPaymentById/" + paymentId,
                Payment.class);

        log.info("获取到的订单是：" + forObject);

        if (forObject == null) {
            throw new NullPointerException("未能在订单表中查询到该订单😭😭😭");
        }

        return forObject;

    }

    /**
     * 服务降级fallback
     * 兜底方法的返回类型必须和原方法一致，且传入的参数也必须一致，但可以添加接收 Throwable异常
     * */
    public Payment fallbackHandler(@PathVariable String paymentId, Throwable e) {
        log.info("兜底方法接收到原方法的参数：" + paymentId);
        log.info("兜底方法接收到异常：" + e);

        Payment payment = new Payment();

        payment.setServerInfo("程序异常了，快想办法，😭");

        return payment;
    }
```

#### （2）`sentinel`结合`open feign`

##### 【1】导入 `open feign`坐标

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

【2】配置yml

```yml
spring:
  application:
    name: sentinel-consumer-order-open-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
# sentinel 要使用feign 需要额外配置
feign:
  sentinel:
    enabled: true
```

##### 【3】启用`open feign`

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class SentinelConsumerOrderOpenFeignApp {

    public static void main(String[] args) {
        SpringApplication.run(SentinelConsumerOrderOpenFeignApp.class,args);
    }
}
```



##### 【4】接口

```java
@FeignClient(name = "sentinel-provider-payment-circuit-break",fallback = PaymentServiceImpl.class)
public interface PaymentService {

    /**
     * 如果有 @PathVariable 注解，则必须在 @PathVariable 中指定参数的名字，否则报错
     * provider 那边倒是可以不修改
     * */
    @GetMapping("/getPaymentById/{paymentId}")
    Result<Payment> getPaymentById(@PathVariable("paymentId") String paymentId);

}
```

##### 【5】接口实现类作为`fallback`方法

```java
@Service
public class PaymentServiceImpl implements PaymentService {


    @Override
    public Result<Payment> getPaymentById(String paymentId) {

        ResultEnum other = ResultEnum.Other;
        other.setCode(445);
        other.setMessage("通过open feign 解耦 ribbon 与controller，我是fallback方法，出错了😭😭");

        return new Result<>(other,null);
    }
}
```



#### （3）熔断框架比较

![Snip20200806_1](/Users/luo/Documents/开发笔记/images/Snip20200806_1.png)

### 11. 持久化

将限流配置规则持久化到Nacos中保存，只要刷新`8401`的某个`rest`地址，`sentinel`控制台的流控规则就能看到，只要`nacos`里面的配置不删除，针对`8401`上的`sentinel`流控规则就持续有效

#### （1）步骤

##### 【1】添加依赖

```xml
        <!--        后续持久化的时候会用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

##### 【2】添加nacos数据源配置

```yml
spring:
  application:
    name: sentinel-consumer-order-open-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080
        port: 8720
      datasource:
        ds1:  # 数据源名字随意
          nacos:
            server-addr: 127.0.0.1:8848         # 或者直接配置到nginx，又nginx连接到nacos集群
            dataId: ${spring.application.name}  # 这里需要作为nacos 的 dataId，用于保存sentinel的配置到nacos上，用json的形式
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

##### 【3】nacos配置

注意区分：带参数的`restful`地址，和`@GetMapping`中的`url`保持一致即可

```json
[
  {
  	"resource":"/consumer/getPaymentById/{paymentId}",
    "limitApp":"default",
    "grade":1,
    "count":1,
    "strategy":0,
    "controlBehavior":0,
    "clusterMode":false
	},
  {
    "resource":"/testSentinelEndurance",
    "limitApp":"default",
    "grade":1,
    "count":1,
    "strategy":0,
    "controlBehavior":0,
    "clusterMode":false
  }
]
```

详细解释

* `resource`：资源名称
* `limitApp`：来源应用
* `grade`：阈值类型，0表示线程数，1表示QPS
* `count`：单机阈值
* `strategy`：流控模式，0表示直接，1表示关联，2表示链路
* `controlBehavior`：流控效果，0表示快速失败，1表示`warm up`，2表示排队等待
* `clusterMode`：是否集群

![Snip20200806_3](/Users/luo/Documents/开发笔记/images/Snip20200806_3.png)

##### 【4】查看`controller`资源映射

```java
    @GetMapping("/consumer/getPaymentById/{paymentId}")
    public Result<Payment> getPaymentById(@PathVariable String paymentId){

        Result<Payment> paymentById = paymentService.getPaymentById(paymentId);

        return paymentById;

    }

    @GetMapping("/testSentinelEndurance")
    public String testSentinelEndurance(){
        return "访问ok";
    }
```

### （2）注意

即使做了持久化，但是在`sentinel`上对已经持久化的的配置文件做修改，修改后的数据并不会保存到`nacos`上

## 四、Seata处理分布式事务

### 1. 分布式之后

单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源，业务操作需要调度三个服务来完成。此时**每个服务内部的数据一致性由本地的事务来保证，但是全局事务的一致性问题无法保证**

用户购买赏评的业务逻辑。整体业务是由三个微服务提供支持

* 仓储服务：对给定的商品扣除仓储数量
* 订单服务：根据采购需求创建订单
* 账户服务：从用户账户中扣除余额

架构图

![Snip20200806_4](/Users/luo/Documents/开发笔记/images/Snip20200806_4.png)

总结：一次业务操作需要跨多个数据源或者需要跨多个系统进行远程调用，就会产生分布式事务问题

### 2. Seata 简介

Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

http://seata.io/zh-cn/

#### （1）一个典型的分布式事务过程

分布式事务处理过程的`ID + 三组件`模型

##### 【1】Transaction ID X

全局唯一的事务ID

##### 【2】三组件

* Transaction Coordinator (TC)：维护全局和分支事务的状态，驱动全局事务提交或回滚。
* Transaction Manager (TM)：定义全局事务的范围：开始全局事务、提交或回滚全局事务。
* Resource Manager (RM)：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

#### （2）Seata 是什么？

Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。在 Seata 开源之前，Seata 对应的内部版本在阿里经济体内部一直扮演着分布式一致性中间件的角色，帮助经济体平稳的度过历年的双11，对各BU业务进行了有力的支撑。经过多年沉淀与积累，商业化产品先后在阿里云、金融云进行售卖。2019.1 为了打造更加完善的技术生态和普惠技术成果，Seata 正式宣布对外开源，未来 Seata 将以社区共建的形式帮助其技术更加可靠与完备。

![whatIsSeata](/Users/luo/Documents/开发笔记/images/whatIsSeata.png)

##### 【1】处理过程

1. `TM`向`TC`申请开启一个全局事务，全局事务创建成功并生成一个全局唯一`XID`
2. `XID`在微服务调用链路的上下文中传播
3. `RM`向`TC`注册分支事务，将其纳入`XID`对应全局事务的管辖
4. `TM`向`TC`发起针对`XID`的全局事务提交或者回滚决议
5. `TC`调度`XID`下管辖的全部分支事务完成提交或者回滚请求

##### 【2】SEATA 的分布式交易解决方案

![SEATA 的分布式交易解决方案](/Users/luo/Documents/开发笔记/images/SEATA 的分布式交易解决方案.png)

### 3. 安装seata

**这里的安装教程适用于 seata 0.9 版本，1.0 版本及其以后才支持集群**



#### （1）指定数据源

**修改 conf/file.conf**

主要修改：自定义事务组名称 + 事务日志存储模式为 `db` + 数据库连接信息

* service 模块

  ```shell
  service {
    #vgroup->rgroup
    # 原来默认 vgroup_mapping.my_test_tx_group = "default"
    # 组名可以随便取，建议是 xxx_tx_group
    vgroup_mapping.my_test_tx_group = "fsp_tx_group"
  }
  ```

  

* Store 模块

  ```shell
  # 事务日志存储模块
  ## transaction log store
  store {
    ## store mode: file、db
    # 默认存在文件中
    # mode = "file"
    # 改为存储在数据库中
    mode = "db"
  
  	# 省略
  	
    ## file store
    file {
  		# 省略
    }
      ## database store
    db {
      ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
      datasource = "dbcp"
      ## mysql/oracle/h2/oceanbase etc.
      db-type = "mysql"
      # 配置数据源url、账号、密码，数据源要有相应的数据库
      driver-class-name = "com.mysql.jdbc.Driver"
      url = "jdbc:mysql://127.0.0.1:3306/seata?useSSL=false&useUnicode=true&characterEncoding=utf8"
      user = "root"
      password = "a1!"
  		# 省略
    }
  }
  ```

#### （2）执行建表语句

如果是 `seata 1.0`及其以上，查看README-zh.md，上面有对应的建表语句

https://github.com/seata/seata/blob/develop/script/server/db/mysql.sql

![Snip20200806_5](/Users/luo/Documents/开发笔记/images/Snip20200806_5.png)

![Snip20200806_6](/Users/luo/Documents/开发笔记/images/Snip20200806_6.png)

#### （3）指定注册中心

**修改 conf/registry.conf 文件**

```shell
registry {
	# 指定注册中心
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"
	
  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
  
  # 省略
 }
```

#### （4）启动

注意：如果`mysql`版本不是 `5.x`，把/lib/mysql-connector-java-5.1.30.jar 替换为相应的`mysql`驱动即可

如果`seata 1.x`，需要把 `seata/lib/jdbc/下的驱动拷贝到 seata/lib/ 目录下，否则找不到

![Snip20200806_7](/Users/luo/Documents/开发笔记/images/Snip20200806_7.png)

```shell
Usage: sh seata-server.sh(for linux and mac) or cmd seata-server.bat(for windows) [options]
  Options:
    --host, -h
      The host to bind.
      Default: 0.0.0.0
    --port, -p
      The port to listen.
      Default: 8091
    --storeMode, -m
      log store mode : file、db
      Default: file
    --help

e.g.
# 记得先启动nacos
sh seata-server.sh -p 8091 -h 127.0.0.1 -m file

# 看到这条日志，说明启动完成
2020-08-06 21:26:09.465 INFO [main]io.seata.common.loader.EnhancedServiceLoader.loadFile:236 -load RegistryProvider[Nacos] extension by class[io.seata.discovery.registry.nacos.NacosRegistryProvider]
```

### 4. 业务说明

这里我们创建三个服务，一个订单服务，一个库存服务，一个账户服务

当用户下订单时，会在订单服务中创建一个订单，然后通过远程调用库存服务来扣减下单赏品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成

该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题

#### （1）创建业务数据库

* Seata_order：存储订单的数据库

* Seata_storage：存储库存的数据库

* seata_account：存储账户信息的数据库

  ```sql
  CREATE DATABASE seata_order;
  CREATE DATABASE seata_storage;
  CREATE DATABASE seata_account;
  USE seata_order;
  CREATE TABLE t_order (
  	id BIGINT ( 11 ) NOT NULL auto_increment,
  	user_id BIGINT ( 11 ) COMMENT "用户id",
  	product_id BIGINT ( 11 ) COMMENT "产品id",
  	count INT ( 11 ) COMMENT "数量",
  	money DECIMAL ( 11, 0 ) COMMENT "金额",
  	STATUS INT ( 1 ) COMMENT "订单状态：0：创建中，1：已完结",
  	CONSTRAINT t_order_pk PRIMARY KEY ( id ) 
  ) ;
  USE seata_storage;
  CREATE TABLE t_storage (
  	id BIGINT ( 11 ) auto_increment,
  	product_id BIGINT ( 11 ) COMMENT "产品id",
  	total INT ( 11 ) COMMENT "总库存",
  	used INT ( 11 ) COMMENT "已用库存",
  	residue INT ( 11 ) COMMENT "剩余库存",
  	CONSTRAINT s_storage_pk PRIMARY KEY ( id ) 
  ) ;
  INSERT INTO seata_storage.t_storage
  VALUES( 1, 1, 100, 0, 100 ) ;
  
  USE seata_account;
  CREATE TABLE t_account (
  	id BIGINT ( 11 ) auto_increment COMMENT "id",
  	user_id BIGINT ( 11 ) COMMENT "用户id",
  	total DECIMAL ( 10, 0 ) COMMENT "总额度",
  	used DECIMAL ( 10, 0 ) COMMENT "已用余额",
  	residue DECIMAL ( 10, 0 ) COMMENT "剩余可用额度",
  	CONSTRAINT t_account PRIMARY KEY ( id ) 
  ) ;
  INSERT INTO t_account
  VALUES( 1, 1, 1000, 0, 1000 );
  ```

#### （2）在每个库下都建立一个回滚日志表

建表语句在这里

![Snip20200806_8](/Users/luo/Documents/开发笔记/images/Snip20200806_8.png)

也就是

```sql
-- the table to store seata xid data
-- 0.7.0+ add context
-- you must to init this sql for you business databese. the seata server not need it.
-- 此脚本必须初始化在你当前的业务数据库中，用于AT 模式XID记录。与server端无关（注：业务数据库）
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
drop table `undo_log`;
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

所有数据库，及其各自的表创建完成效果图

![Snip20200806_9](/Users/luo/Documents/开发笔记/images/Snip20200806_9.png)

#### （3）yml配置

```yml
server:
  port: 7001

spring:
  datasource:
    username: root
    password: orochi0208
    url: jdbc:mysql://127.0.0.1:3306/seata_order?useSSL=false&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
  application:
    name: seata-order-7001
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    alibaba:
      seata:
        # 事务组的名称，需要和seata-server 事务组的名称相匹配
        tx-service-group: fsp_tx_group  # 这个group就是 seata/conf/file.conf 中配置的组名

# 开启 open feign 负载均衡
feign:
  hystrix:
    enabled: true

mybatis-plus:
  global-config:
    db-config:
      id-type: auto
      table-prefix: t_


```

#### （4）复制 septa server 的配置文件

将`seata/conf/file.conf`和`seata/conf/registry.conf`拷贝到模块的`resources`目录下

两个文件都是已经修改过的

![Snip20200806_11](/Users/luo/Documents/开发笔记/images/Snip20200806_11.png)

修改 resources/file.conf

```shell
service {
  #vgroup->rgroup
  vgroup_mapping.fsp_tx_group = "default"
}
```

#### （5）问题

当库存和账户金额扣减之后，订单状态并没有设置为已完成，没有从0改为1

而且由于feign的重试机制，账户余额还有可能被多次扣减



#### （6）解决

```java
    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 下订单->减库存->减余额->改状态
     * */
    @GlobalTransactional(name = "suibian",rollbackFor = Exception.class)
    @Override
    public Order createOrder(Order order) {
        // 因为新建时，订单状态为创建中
        order.setStatus(0);

        log.info("开始创建订单："+order);
        /**
         * 因为是自动生成的mapper.xml，记得在插入sql上添加 useGeneratedKeys="true" keyColumn="id" keyProperty="id"
         * */
        this.orderMapper.insert(order);

        log.info("订单创建完成");

        log.info("调用库存微服务，正在执行减库存操作");

        storageService.storageDecrease(order.getProductId(),order.getCount());

        log.info("减库存执行完毕");

        log.info("调用账户微服务，正在扣钱");

        accountService.moneyDecrease(order.getUserId(),order.getMoney());

        log.info("扣钱完成");

        log.info("修改订单状态为已完成");

        Order orderFinished = new Order();

        orderFinished.setId(order.getId());

        orderFinished.setStatus(1);
        order.setStatus(1);
        this.orderMapper.updateByPrimaryKeySelective(orderFinished);

        log.info("订单状态修改完成");

        return order;
    }
```



### 5. 原理

* `TM`开启分布式事务（`TM`向`TC`注册全局事务记录）
* 按照业务场景，编排数据库，服务等事务内资源（`RM`向`TC`汇报资源准备状态）
* `TM`结束分布式事务，事务一阶段结束（`TM`通知`TC`提交/回滚分布式事务）
* `TC`汇总事务信息，决定分布式事务是提交还是回滚
* `TC`通知所有`RM`提交/回滚 资源，事务二阶段结束

### 6. 模式

http://seata.io/zh-cn/docs/overview/what-is-seata.html

AT 模式：（默认）提供无侵入自动补偿的事务模式，目前已支持 MySQL、 Oracle 、PostgreSQL和 TiDB的AT模式，H2 开发中

TCC 模式：支持 TCC 模式并可与 AT 混用，灵活度更高

SAGA 模式：为长事务提供有效的解决方案

XA 模式：支持已实现 XA 接口的数据库的 XA 模式

#### （1）AT 模式

##### 前提

* 基于支持本地 ACID 事务的关系型数据库。
* Java 应用，通过 JDBC 访问数据库。

##### 整体机制

两阶段提交协议的演变：

* 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
* 二阶段：
  * 提交异步化，非常快速地完成。
  * 回滚通过一阶段的回滚日志进行反向补偿。

###### 一阶段

在一阶段，Seata会拦截`业务SQL`

1. 解析 SQL 语意，找到`业务SQL`需要更新的业务数据，在业务数据被更新之前，将其保存为`before image`
2. 执行`业务SQL`，更新业务数据
3. 在业务数据更新之后，将其保存为`after image`，最后生成行锁
4. 以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性

![Snip20200807_1](/Users/luo/Documents/开发笔记/images/Snip20200807_1.png)

![Snip20200807_2](/Users/luo/Documents/开发笔记/images/Snip20200807_2.png)

###### 如果二阶段顺利

因为`业务SQL`在一阶段已经提交数据库，所以`seata`框架只需将**一阶段保存的快照数据和行锁删除，完成数据清理即可**

![Snip20200807_3](/Users/luo/Documents/开发笔记/images/Snip20200807_3.png)

###### 二阶段回滚

二阶段如果是回滚的话，`seata`就需要回滚一阶段已经执行的`业务SQL`，还原业务数据。（反向补偿）

回滚方式便是用`before image`还原业务数据，但在还原前要校验脏写，对比`数据库当前业务数据`和`after image`

如果两份数据完全一致就说明没有脏写，可以还原数据，如果不一致就说明有脏写，出现脏写就要转人工处理

![Snip20200807_4](/Users/luo/Documents/开发笔记/images/Snip20200807_4.png)

### 7. 补充

![Snip20200807_5](/Users/luo/Documents/开发笔记/images/Snip20200807_5.png)

![Snip20200807_6](/Users/luo/Documents/开发笔记/images/Snip20200807_6.png)

## 五、执行流程

### 1. 接口化请求调用

当调用被`@FeignClient`注解修饰的接口时，再框架内部，将请求转换成`Feign`的请求实例`feign.Request`，交由`Fegign`框架处理

### 2. Feign

转化请求`Feign`是一个http请求调用的轻量级框架，可以以java接口注解的方式调用http请求，封装了http调用流程

在注册中心（nacos）中找到该微服务的提供者

```java
/**
 * path 属性为所有mapping方法添加前缀
 * name 调用的微服务名称，之后通过注册中心解析到真正的微服务提供者
 * */
@FeignClient(name = "vod-service",path = "/vod/video")
public interface VodService {

		// 然后再根据这个url来进行服务的调用
    @DeleteMapping("/deleteVideoByVideoSourceId/{videoSourceId}")
    public Result deleteVideoByVideoSourceId(@PathVariable("videoSourceId") String videoSourceId);
}
```

### 3. Hystrix

当目标服务不可用的时候，进行熔断	

熔断处理机制`Feign`的调用关系，会被`Hystrix`代理拦截，对每一个`Feign`调用请求，`Hystrix`都会将其包装成`HystrixCommand`参与`Hystrix`的流程和熔断规则。如果请求判断需要熔断则`Hystrix`直接熔断，抛出异常或者使用`FallbackFactory`返回熔断`Fallback`结果，如果通过，则将调用请求传递给`Ribbon`组件

### 4. Ribbon

负载均衡，选择合适的微服务提供者进行调用

服务地址选择，当请求传递到`Ribbon`之后，`Ribbon`会根据自身维护服务列表，根据服务的服务质量，如平均响应时间，Load等，结合特定的规则，从列表中挑选合适的服务实例，选择好机器之后，然后将机器实例的信息请求传递给HttpClient客户端，`HttpClient`客户端来执行真正的Http接口调用

### 5. http client

通过ip+端口号访问对应的微服务进行服务的调用

Http 客户端，真正执行 Http 调用，根据上层`Ribbon`传递过来的请求，已经指定了服务地址，则`HttpClient`开始执行真正的Http请求



















