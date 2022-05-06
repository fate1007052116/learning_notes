# ActiveMQ

* 访问管理页面 http://ip:8161/admin

* 用户名：admin  密码：admin

  在/activeMQ/conf/users.properties 修改用户名和密码

  ```proper
  admin=admin
  ```

  

* activeMQ是jetty提供的Http服务，启动较慢

* 可以到activeMQ/conf/jetty/activemq.xml 修改activeMQ监听的端口

  ```xml
  <bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
               <!-- the default port number for the web console -->
          <property name="host" value="0.0.0.0"/>
          <property name="port" value="8161"/>
	</bean>
	```

* activeMQ/conf/jetty/activemq.xml 是 activeMQ 的核心配置信息，是提供服务时所使用的配置，可以修改启动的访问端口，即java编程中访问activeMQ的访问端口

  默认端口：61616

  协议：tcp

```xml
<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
        </transportConnectors>
```

# ActiveMQ术语

* Destination 目的地。JMS（消息中间件）Provider负责维护，用于对Message进行管理的对象。MessageProducer需要指定Destincation才能发送消息，MessageReceiver需要指定Destination才能接受消息
* Producer 消息生产者，负责发送Message到目的地
* Consumer | Receiver 消息消费者，负责从目的地中消费【处理｜监听｜订阅】Message
* Message 消息，消息封装一次通信的内容

# 常用API简介

下述API都是借口类型，定义在javax.jms包中，是JMS标准接口定义

* ConnectionFactory 连接工厂，用于创建连接工厂的类型

* Connection  链接，用于建立访问ActiveMQ连接的类型，由链接工厂创建

* Session  会话，一次持久有效有状态的访问，有连接创建

* Destination & Queue 目的地，用于描述本次访问ActiveMQ的消息访问目的地，即ActiveMQ服务中具体队列，由会话创建

  Interface Queeu extends Destination

* MessageProducer 消息生产者，在一次有效会话中，用于发送消息给ActiveMQ服务的工具，由会话创建

* MessageConsumer  消息消费者【消息订阅者，消息处理者】，在一次有效会话中，用于从ActiveMQ服务中获取消息的工具，由会话创建

* Message   消息，通过MessageProducer 向 ActiveMQ 服务发送消息时使用的数据载体对象或者 消息消费者从ActiveMQ服务中获取消息时使用的数据载体对象，是所有消息【文本消息，对象消息等】具体类型的顶级接口，

* 可以通过会话创建或者功过会话从ActiveMQ中获取

# Topic模型

### publish/subscribe 处理模式(Topic)

* 消息生产者（发布）将消息发布到Topic中，同时有多个消息消费者（订阅）消费该消息

* 和点对点方式不同，发布到Topic的消息会被所有订阅者消费
* 当生产者发布消息，不管是否有消费者，都不会保存消息
* 一定要先有消息的消费者，后有消息的生产者

## 注意

### RabbitMq 和 ActiveMQ所绑定的是同一个端口5672

### 当配置过RabbitMQ 改过hostName之后，再用同一机器启动ActiveMQ会无法启动，即使更改了端口

```shell
# 启动时查看日志
./activemq start & tail -f ../data/activemq.log

2020-07-10 05:43:02,196 | INFO  | Closing org.apache.activemq.xbean.XBeanBr
okerFactory$1@20398b7c: startup date [Fri Jul 10 05:42:54 EDT 2020]; root o
f context hierarchy | org.apache.activemq.xbean.XBeanBrokerFactory$1 | main
# 错误信息
2020-07-10 05:43:02,201 | WARN  | Exception encountered during context init
ialization - cancelling refresh attempt: org.springframework.beans.factory.
BeanCreationException: Error creating bean with name 'org.apache.activemq.x
bean.XBeanBrokerService#0' defined in class path resource [activemq.xml]: I
nvocation of init method failed; nested exception is java.net.URISyntaxExce
ption: Illegal character in hostname at index 13: ws://RabbitMQ_1:61614?max
imumConnections=1000&wireFormat.maxFrameSize=104857600 | org.apache.activem
q.xbean.XBeanBrokerFactory$1 | main
```



