# RabbitMQ

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200628030707025.png" alt="image-20200628030707025" style="zoom:50%;" />

1.Message

​		消息：消息是不具名的，它由消息头和消息题组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括：routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出消息可能持久性存储）等

2.Publisher

​		消息的生产者：也是一个向Exchange发布消息的客户端应用程序

3.Consumer

​		消息消费者，表示一个从消息队列中取得消息的客户端应用程序

4.Exchange

交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列

三种常见的交换类型

* direct 发布与订阅、完全匹配。点对点，根据路由键将消息精准的投递到特定的消息队列中

  ​		如果是其他类型的路由键，则一律丢弃到 black hole 中

* fanout 广播

  ​		exchange 将接收到的消息同步到所有与该Exchange相连的消息队列中

* topic 主题，匹配规则

  ​		判断 被传递的消息的路由键与exchange 和 queue 之间绑定的路由键 是否匹配，匹配则发送到该queue

  ​		路由键可以带通配符 * 

5.Binding

​		绑定，用于消息队列 queue 和 交换器 exchange之间的关联。一个绑定就是基于路由键将 exchange 和 queue连接起来的路由规则，所以可以将交换器理解成一个由绑定所构成的路由表

6.Queue

 		消息队列。用来保存消息直接发送给消费者。他是消息的容器，也是消息的重点，一个消息可以投入一个或者多个消息队列。消息将一直在队列里面，等待消费者连接到这个队列将其取走

7.Routing-key

​		路由键。RobbitMQ 决定消息该投递到哪个队列的规则（也可以理解为队列的名称，路由键是key，队列是value）

 * Queue通过路由键绑定到 Exchange 

 * 消息发送到MQ服务器时，消息将拥有一个路由键，即使这个路由键是null，RabbitMQ也会将其和绑定使用的路由键进行匹配

 * 如果相匹配，消息将会投递到该消息队列

 * 如果不匹配，消息将会进入black hole

 * 路由键的匹配规则

   ```xml
   * 代表 0 或者 n 个字符，不包括特殊字符，例如 . -
   # 代表全部字符，包含- . _ 等 特殊字符
   ```

8.Connection

​		连接。指RabbitMQ服务器和服务建立的TCP连接

9.Channel

​		信道。是TCP里面的虚拟连接，一条TCP连接上创建多条新到是没有问题的

10.Virthal Host

​		虚拟主机。表示一批Exchange Queue 和 相关对象。虚拟主机是共享相同身份认证和加密环境的独立服务器域

每个vHost本质上就是一个mini版的RabbitMQ服务器，拥有自己的Queue Exchange 绑定和权限机制

vHost是AMQP（高息消息队列协议）概念的基础，必须在连接时制定，RabbitMQ默认的vHost是 /

11.Broker

​		表示消息队列服务器实体，多个Virtual Host 组成一个Broker

一个Broker可以为一个进程，或者实例

## 1. 交换器和队列的关系

​		交换器是通过路由键和队列绑定在一起的，如果消息拥有的Routing-key 跟 Queue Exchange 的路由键匹配，那么该消息就会被路由到该绑定的队列中

​		也就是说，消息到队列的过程中，消息会首先经过交换器，接下来交换器再通过routing-key 匹配分发消息到具体的队列中

​		路由键可以理解为匹配规则

## 2. RabbitMQ 为什么需要信道，为什么不是TCP直接通信

* Tcp的创建和销毁开销特别大。创建需要3次握手，销毁需要4次分手
* 如果不用信道，那应用程序就会以TCP连接Rabbit，高峰时每秒成千上万条连接会造成资源巨大浪费，而且操作系统每秒处理Tcp连接数也是有限的，必定造成性能瓶颈
* 信道的原理是一个线程一条信道，多个线程多条信道共用一条Tcp连接。一条Tcp连接可以容纳无限的信道，即使每秒成千上万的请求也不会成为性能的瓶颈
* 端口的数量限制了Tcp连接的个数



## 3. RabbitMQ 的安装

### （1）安装运行时环境Erlang

### （2） RabbitMQ是根据主机的名称进行解析的，所以需要配置主机名称（最优先解析主机名称）

```shell
vi /etc/sysconfig/network  # 配置主机名称

# Created by anaconda
HOSTNAME=RabbitMQ_1

vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 
192.168.2.10 RabbitMQ_1	# 将ip地址与主机名绑定，也可以绑定域名
```



### （3）安装依赖

```shell
[root@localhost ~]# yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC unixODBC-devel
上次元数据过期检查：0:03:39 前，执行于 2020年06月28日 星期日 02时17分52秒。
软件包 make-1:4.2.1-10.el8.x86_64 已安装。
软件包 gcc-8.3.1-5.el8.0.2.x86_64 已安装。
软件包 gcc-c++-8.3.1-5.el8.0.2.x86_64 已安装。
软件包 kernel-devel-4.18.0-193.6.3.el8_2.x86_64 已安装。
软件包 m4-1.4.18-7.el8.x86_64 已安装。
软件包 ncurses-devel-6.1-7.20180224.el8.x86_64 已安装。
软件包 openssl-devel-1:1.1.1c-15.el8.x86_64 已安装。
软件包 unixODBC-2.3.7-1.el8.x86_64 已安装。
软件包 unixODBC-devel-2.3.7-1.el8.x86_64 已安装。
依赖关系解决。
无需任何处理。
完毕！
# 这里yum源默认使用CentOS默认源
```

* kernel-devel 内核开发工具包
* ncurses-devel 内存开发工具包
* openssl-devel 加密协议开发工具包
* unixODBC 桥连接
```shell
#Erlang src 压缩的时候没有使用gzip压缩，所以解压的时候不用带参数z
tar -xvf 
./configure --prefix=/usr/local/erlang --with-ssl --enable-threands --enable-smp-suppor --enable-kernel-poll --enable-hipe --without-javac
# --with-ssl 提供加密
# --enable-threands 开启多线程高并发处理
# --enable-smp-suppor smp协议支持
# --enable-kernel-poll 提供内核池处理能力
# --withoutjavac 不提供javac编译器 解析java代码，纯erlang效率更高
#
# 出现以下警告不影响使用
configure: WARNING: No OpenGL headers found, wx will NOT be usable
configure: WARNING: No GLU headers found, wx will NOT be usable
./configure: line 4659: wx-config: command not found
configure: WARNING:
                wxWidgets must be installed on your system.

                Please check that wx-config is in path, the directory
                where wxWidgets libraries are installed (returned by
                'wx-config --libs' or 'wx-config --static --libs' command)
                is in LD_LIBRARY_PATH or equivalent variable and
                wxWidgets version is 2.8.4 or above.

*********************************************************************
**********************  APPLICATIONS DISABLED  **********************
*********************************************************************

jinterface     : Java compiler disabled by user

*********************************************************************
*********************************************************************
**********************  APPLICATIONS INFORMATION  *******************
*********************************************************************

wx             : No OpenGL headers found, wx will NOT be usable
No GLU headers (glu.h) found, wx will NOT be usable
wxWidgets not found, wx will NOT be usable

*********************************************************************
*********************************************************************
**********************  DOCUMENTATION INFORMATION  ******************
*********************************************************************

documentation  : 
                 fop is missing.
                 Using fakefop to generate placeholder PDF files.

*********************************************************************

# 编译完成之后自动安装
make && make install

# 配置Erlang环境变量
vi /etc/profile

export PATH=$PATH:/usr/local/erlang/bin
# 我的实际配置
export JAVA_HOME=/usr/local/jdk1.8.0_251
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin:/usr/local/erlang/bin
```

### （4） 安装RabbitMQ

```shell
# 解压时不要带z参数
tar -xvf rabbitmq-server-generic-unix-3.8.5.tar.xz

[root@localhost sbin]# pwd
/opt/rabbitmq_server-3.8.5/sbin
[root@localhost sbin]# ll
总用量 40
-rwxr-xr-x. 1 root root 1236 6月  15 09:20 rabbitmqctl		# 管理
-rwxr-xr-x. 1 root root  990 6月  15 09:20 rabbitmq-defaults
-rwxr-xr-x. 1 root root 1245 6月  15 09:20 rabbitmq-diagnostics
-rwxr-xr-x. 1 root root 6357 6月  15 09:20 rabbitmq-env
-rwxr-xr-x. 1 root root 1241 6月  15 09:20 rabbitmq-plugins # 安装插件
-rwxr-xr-x. 1 root root 1240 6月  15 09:20 rabbitmq-queues
-rwxr-xr-x. 1 root root 7033 6月  15 09:20 rabbitmq-server # 启动RabbitMQ 
-rwxr-xr-x. 1 root root 1241 6月  15 09:20 rabbitmq-upgrade

# 查看可以安装的插件
[root@localhost sbin]# ./rabbitmq-plugins list
Listing plugins with pattern ".*" ...
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status: [failed to contact rabbit@localhost - status not shown]
 |/
[  ] rabbitmq_amqp1_0                  3.8.5
[  ] rabbitmq_auth_backend_cache       3.8.5
[  ] rabbitmq_auth_backend_http        3.8.5
[  ] rabbitmq_auth_backend_ldap        3.8.5
[  ] rabbitmq_auth_backend_oauth2      3.8.5
[  ] rabbitmq_auth_mechanism_ssl       3.8.5
[  ] rabbitmq_consistent_hash_exchange 3.8.5
[  ] rabbitmq_event_exchange           3.8.5
[  ] rabbitmq_federation               3.8.5
[  ] rabbitmq_federation_management    3.8.5
[  ] rabbitmq_jms_topic_exchange       3.8.5
[  ] rabbitmq_management               3.8.5  # 需要安装的插件，复制插件的名字 rabbitmq_management
[  ] rabbitmq_management_agent         3.8.5
[  ] rabbitmq_mqtt                     3.8.5
[  ] rabbitmq_peer_discovery_aws       3.8.5
[  ] rabbitmq_peer_discovery_common    3.8.5
[  ] rabbitmq_peer_discovery_consul    3.8.5
[  ] rabbitmq_peer_discovery_etcd      3.8.5
[  ] rabbitmq_peer_discovery_k8s       3.8.5
[  ] rabbitmq_prometheus               3.8.5
[  ] rabbitmq_random_exchange          3.8.5
[  ] rabbitmq_recent_history_exchange  3.8.5
[  ] rabbitmq_sharding                 3.8.5
[  ] rabbitmq_shovel                   3.8.5
[  ] rabbitmq_shovel_management        3.8.5
[  ] rabbitmq_stomp                    3.8.5
[  ] rabbitmq_top                      3.8.5
[  ] rabbitmq_tracing                  3.8.5
[  ] rabbitmq_trust_store              3.8.5
[  ] rabbitmq_web_dispatch             3.8.5
[  ] rabbitmq_web_mqtt                 3.8.5
[  ] rabbitmq_web_mqtt_examples        3.8.5
[  ] rabbitmq_web_stomp                3.8.5
[  ] rabbitmq_web_stomp_examples       3.8.5

# 开启该插件
[root@localhost sbin]#  ./rabbitmq-plugins enable rabbitmq_management
Enabling plugins on node rabbit@localhost:
rabbitmq_management
The following plugins have been configured:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@localhost...
The following plugins have been enabled:
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch

set 3 plugins.
Offline change; changes will take effect at broker restart.

# 配置RabbitMQ插件，前提是上一步打开了这个插件 ，这里的etc目录是RabbitMQ 目录下的etc目录

[root@localhost rabbitmq]# vi /opt/rabbitmq_server-3.8.5/etc/rabbitmq/enabled_plugins 
```

### （4）启动

```shell
# 前台启动
[root@localhost sbin]# ./rabbitmq-server

  ##  ##      RabbitMQ 3.8.5
  ##  ##
  ##########  Copyright (c) 2007-2020 VMware, Inc. or its affiliates.
  ######  ##
  ##########  Licensed under the MPL 1.1. Website: https://rabbitmq.com

  Doc guides: https://rabbitmq.com/documentation.html
  Support:    https://rabbitmq.com/contact.html
  Tutorials:  https://rabbitmq.com/getstarted.html
  Monitoring: https://rabbitmq.com/monitoring.html

  Logs: /opt/rabbitmq_server-3.8.5/var/log/rabbitmq/rabbit@localhost.log
        /opt/rabbitmq_server-3.8.5/var/log/rabbitmq/rabbit@localhost_upgrade.log

  Config file(s): (none)

  Starting broker... completed with 3 plugins.
  
  # 后台启动
  ./rabbitmq-server -detached
  
  #带提示信息的启动
  ./rabbitmqctl start_app
  
  # 此时启动会有问题，需要创建一个rabbitmq-env.conf文件
  vi /opt/rabbitmq_server-3.8.5/etc/rabbitmq/rabbitmq-env.conf
  # 在 rabbitmq-env.conf 写入以下内容
  # HOSTNAME=rabbit@之前配置的主机名称，如果出错，可以配置为localhost
  HOSTNAME=rabbit@RabbitMQ_1 # 注意是rabbit 不是 rabbitmq@
  # 如果出错，可以改为
  HOSTNAME=RabbitMQ_1
  
  # 启动成功
  [root@RabbitMQ_1 sbin]# ./rabbitmq-server 

  ##  ##      RabbitMQ 3.8.5
  ##  ##
  ##########  Copyright (c) 2007-2020 VMware, Inc. or its affiliates.
  ######  ##
  ##########  Licensed under the MPL 1.1. Website: https://rabbitmq.com

  Doc guides: https://rabbitmq.com/documentation.html
  Support:    https://rabbitmq.com/contact.html
  Tutorials:  https://rabbitmq.com/getstarted.html
  Monitoring: https://rabbitmq.com/monitoring.html

  Logs: /opt/rabbitmq_server-3.8.5/var/log/rabbitmq/rabbit@RabbitMQ_1.log
        /opt/rabbitmq_server-3.8.5/var/log/rabbitmq/rabbit@RabbitMQ_1_upgrade.log

  Config file(s): (none)

  Starting broker... completed with 3 plugins.
```

### （5）启动成功之后会开放两个端口

* 5672 客户端使用Tcp协议访问的端口
* 15672 客户端使用http访问的服务 （开放这个端口需要让 rabbitmq_management 插件生效之后，才会开放）
  * 此时可以通过http访问服务，但是还登陆不了
  * 只提供了唯一的用户名 guest 密码 guest

### （6）有时开启RabbitMQ之后，不会监听5672端口，但是http的15672端口可以正常访问

```shell
[root@RabbitMQ_1 rabbitmq_server-3.8.5]# netstat -anp|grep 3568
tcp        0      0 0.0.0.0:15672           0.0.0.0:*               LISTEN      3568/b
eam.smp       
# 偶尔会没有下边这条记录0 0.0.0.0:5672 ，说明没有监听该端口，java连接不上
tcp        0      0 0.0.0.0:5672            0.0.0.0:*               LISTEN      3568/b
eam.smp       
tcp        0      0 0.0.0.0:25672           0.0.0.0:*               LISTEN      3568/b
eam.smp       
tcp        0      0 127.0.0.1:57473         127.0.0.1:4369          ESTABLISHED 3568/b
eam.smp       
tcp        0      0 192.168.2.10:15672      192.168.2.1:54594       ESTABLISHED 3568/b
eam.smp       
tcp        0      0 192.168.2.10:5672       192.168.2.1:54200       ESTABLISHED 3568/b
eam.smp       
tcp6       0      0 :::5673                 :::*                    LISTEN      3568/b
eam.smp       
unix  3      [ ]         STREAM     CONNECTED     47595    3568/beam.smp    

# 创建这个配置文件
vi /opt/rabbitmq_server-3.8.5/etc/rabbitmq/rabbitmq.config

# rabbitmq.config 中的内容,配置其监听的端口，方式一
[root@RabbitMQ_1 rabbitmq_server-3.8.5]# cat /opt/rabbitmq_server-3.8.5/etc/rabbitmq/rabbitmq.config             
[
  {rabbit, [
    {tcp_listeners, [{"0.0.0.0",5672 },
                     {"::",      5673}]}
  ]}
].

# 方式二
root@c8f26efd278c:/etc/rabbitmq# cat rabbitmq.conf
loopback_users.guest = false
listeners.tcp.default = 5672
management.tcp.port = 15672
```



## 4. RabbitMQ 账户管理

```shell
# 需要在RabbitMQ 服务启动的情况下才能添加用户
./rabbitmqctl add_user luo a1! # 用户名 密码
Adding user "luo" ...
# 创建好之后，该用户默认是无角色无权限
# 授予角色权限
./rabbitmqctl set_user_tags luo administrator
Setting tags for user "luo" to [administrator] ...

# 设置用户可以在哪一台主机访问broker，可以提供的功能以及权限
./rabbitmqctl set_permissions -p "/" luo ".*" ".*" ".*"
 # -p "虚拟机名称"
 # ".*" 代表授予所有权限 分为三级，

```

## 5. RabbitMQ 默认不会创建队列

RabbitMQ 默认启动的时候不会做队列的处理

创建Queue的方式

* 手动写代码创建Queue

  ​	主动创建：不推荐，因为手动创建的Queue未必有监听

  ​	如果再没有监听的情况下创建队列，浪费内存，

* 使用注册监听的方式创建Queue （推荐）

  ​	注册监听一个Consumer，consumer就会监听MQ的一个队列Queue

  ​	RabbitMQ会检查这个Queue、Exchange是否存在，如果不存在RabbitMQ就会帮我们创建

## 5. Exchange 交换机

### D 代表（durable）长期持久化的交换器，不会被删除

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200628224119089.png" alt="image-20200628224119089"  />

### 查看当前交换器绑定的队列

![image-20200628224304868](/Users/luo/Library/Application Support/typora-user-images/image-20200628224304868.png)

### AD 代表 autoDelete，当连接本队列的所有consumer关闭时，这个队列就会被删除

### D 代表durable ，持久化队列，不会被删除 

![image-20200628224641328](/Users/luo/Library/Application Support/typora-user-images/image-20200628224641328.png)



### （1）direct 交换器

* 点对点交换器，发送者发送消息的Routing-key 只能投递到一个Queue中

* 发消息的时候必须制定该消息要投递到哪一个队列中

* 这里的点对点指的是publisher 对 Queue，不是publisher 对 consumer

* 1个Queue只有唯一一个与之对应的Routing-key，且是一个固定值

* 如果有多个consumer连接同一个Queue，RabbitMQ默认会使用公平调度（轮询机制），但不是绝对轮询，消息会优先给空闲的consumer，如果消费者的性能不相同，高性能的消费者会处理更多的消息

### （2） fanout 交换器

* 扇形交换器，实质上做的事情就是广播，他会把消息发送到所有绑定该交换器的队列中
* 如果有多个consumer连接同一个Queue，RabbitMQ默认会使用公平调度（轮询机制），但不是绝对轮询，消息会优先给空闲的consumer，如果消费者的性能不相同，高性能的消费者会处理更多的消息
* provider 发送消息时不用管 exchange的类型，和往常一样直接发送即可

### （3） Topic 交换器

* 既可以实现点对点传输（direct），也可以实现点对面传输（fanout）

* 允许在routing-key 中出现匹配规则

* 路由键的写法和包名的写法相同 com.xxx.service.impl.UserService

* 在绑定的时候可以出现下面特殊符号

  ```java
  * : 代表一个单词（两个. 之间的内容）
  # : 代表0个或者多个字符（适用于- _ 等作为分隔符）
  ```

* 如果有多个consumer连接同一个Queue，RabbitMQ默认会使用公平调度（轮询机制），但不是绝对轮询，消息会优先给空闲的consumer，如果消费者的性能不相同，高性能的消费者会处理更多的消息

## 6. 自动配置原理

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	@ConditionalOnMissingBean(ConnectionFactory.class)
	protected static class RabbitConnectionFactoryCreator {
		
    // 创建连接工厂对象
    //RabbitProperties properties 中封装了rabbitMQ的所有配置
		@Bean
		public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties properties,
				ObjectProvider<ConnectionNameStrategy> connectionNameStrategy) throws Exception {
    }
  }
  
  @Configuration(proxyBeanMethods = false)
	@Import(RabbitConnectionFactoryCreator.class)
	protected static class RabbitTemplateConfiguration {
    
    
    @Bean
		@ConditionalOnMissingBean
		public RabbitTemplateConfigurer rabbitTemplateConfigurer(RabbitProperties properties,
				ObjectProvider<MessageConverter> messageConverter,
				ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers) {
			RabbitTemplateConfigurer configurer = new RabbitTemplateConfigurer();
      // 可以在这里设置消息转换器，默认使用jdk的序列化来转换消息，可以改为jackson序列化的方式，直接创建自己的MEssageConverter 并放到IOC容器中，即可自动使用
			configurer.setMessageConverter(messageConverter.getIfUnique());
			configurer
					.setRetryTemplateCustomizers(retryTemplateCustomizers.orderedStream().collect(Collectors.toList()));
			configurer.setRabbitProperties(properties);
			return configurer;
		}
    
    
    // RabbitTemplate 可用于直接操作rabbitMQ，可直接发送，接收消息
    @Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnMissingBean(RabbitOperations.class)
		public RabbitTemplate rabbitTemplate(RabbitTemplateConfigurer configurer, ConnectionFactory connectionFactory) {
			RabbitTemplate template = new RabbitTemplate();
			configurer.configure(template, connectionFactory);
			return template;
		}
    
    // AmqpAdmin 是rabbitMQ的系统管理组件，声明队列，创建交换器等
    		@Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnProperty(prefix = "spring.rabbitmq", name = "dynamic", matchIfMissing = true)
		@ConditionalOnMissingBean
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
			return new RabbitAdmin(connectionFactory);
		}
  }
  
}
```

