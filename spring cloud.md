# spring cloud

## 一、cloud升级

### 1. 服务注册中心

* Eureka: 停止更新，曾经的默认
* Zookeeper: 可以使用老技术
* Consul: 保守
* Nacos: 阿里巴巴，完美替换 Eureka

### 2. 服务调用

* Ribbon: 维护状态
* spring cloud loadBalance: 逐渐取代Ribbon

### 3. 服务调用2

* Feign: 和Ribbon齐名，netflix公司推出，挂了
* OpenFeign: 推荐

### 4. 服务降级

* Hystrix: 官网快挂了，国内大规模使用
* resilience4j: 官网推荐，国外用的人多，国内用的人少
* Alibaba sentinel: 推荐

### 5. 服务网关

* Zuul: 成员分裂
* Zuul2: 等不到它发布了
* Gateway: 主流，重点

### 6. 服务配置

* config：不再使用
* nacos：替换config，后来居上

### 7. 服务总线

* bus：spring原生
* nacos： 

## 二、服务注册中心

### 1. Eureka

#### （1）单机版服务端配置

1. maven 依赖

   ```xml
   		<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-server
   
   			spring-cloud-starter-eureka 适用于2018年的Spring-cloud
   			引入 eureka 注册服务中心
   		-->
   		<dependency>
   			<groupId>org.springframework.cloud</groupId>
   			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   		</dependency>
   ```

   

2. yml

```yml
server:
  port: 7001

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
  	hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      # 设置Eureka Server 交互的地址查询服务和注册服务都需要依赖这个地址
      # 注意，地址容易写错
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
```

3. 开启Eureka注册中心

```java
@SpringBootApplication
/**
 * 开启Eureka的注册中心服务
 * localhost:7001 即可访问web界面
 * */
@EnableEurekaServer
public class CloudEurekaServer7001Application {

	public static void main(String[] args) {
		SpringApplication.run(CloudEurekaServer7001Application.class, args);
	}

}
```

#### （2）单机版客户端配置

1. maven

```xml
        <!--引入Eureka 客户端依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

2. yml配置

```yml
spring:
  application:
    # 微服务名称
    name: cloud-payment-service
    
# 配置Eureka客户端
eureka:
  client:
    # 表示是否将自己注册进EurekaServer，默认true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认true。单节点无所谓，集群必须设置为true，才能配合ribbon配合使用负载均衡
    fetch-registry: true
    service-url:
      # 注册中心的地址
      defaultZone: http://localhost:7001/eureka
```

3. 开启Eureka 客户端服务

```java
@SpringBootApplication
@EnableCaching
/**
 * 开启Eureka 客户端
 * */
@EnableEurekaClient
@MapperScan(basePackages = "com.example.cloudproviderpayment8001.mapper")
public class CloudProviderPayment8001Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudProviderPayment8001Application.class, args);
    }

}
```



#### （3） 集群版Eureka

1. 修改hosts文件如下（非必须）


```shell

sh-3.2# tail /etc/hosts
::1             localhost
# Royal TSX
 127.0.0.1 api.royalapps.com
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal
# End of section

127.0.0.1 eureka7001.com 	# 新增域名
127.0.0.1 eureka7002.com	# 新增域名
```

2. Eureka Server 注册中心一

```yml
server:
  port: 7001

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
  	hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      # 端口为7001的Eureka Server 需要去端口为  7002 Eureka Server 注册 自己 （7001）
      defaultZone: http://eureka7002.com:7002/eureka
```

3. Eureka Server 注册中心二

```yml
server:
  port: 7002

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
  	hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      # 端口为7002的Eureka Server 需要去端口为  7001 Eureka Server 注册 自己 （7002）
      defaultZone: http://eureka7001.com:7001/eureka
```

4. 配置成功

![Snip20200729_4](/Users/luo/Documents/开发笔记/images/Snip20200729_4.png)

![Snip20200729_6](/Users/luo/Documents/开发笔记/images/Snip20200729_6.png)

5. 配置微服务同时在两个注册中心注册自己

```yml
server:
  port: 80
spring:
  application:
    name: cloud-order-service
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      # defaultZone: http://localhost:7001/eureka # 单机版Eureka注册中心配置
      # 集群版Eureka注册中心配置
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

#### （4）搭建三台Eureka Server 作为注册中心

0. 新增host记录

```shell
sh-3.2# tail /etc/hosts
# Royal TSX
 127.0.0.1 api.royalapps.com
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal
# End of section

127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```

1. 注册中心一

```yml
server:
  port: 7001

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
  	hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      # 端口为7001的Eureka Server 需要去端口为  7002 Eureka Server 注册 自己 （7001）
      # defaultZone: http://eureka7002.com:7002/eureka # 二台 Eureka Server 作为注册中心
      # 三台 Eureka Server 搭建注册中心集群
      defaultZone: http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
```

2. 注册中心二

```yml
server:
  port: 7002

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
  	hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      # 端口为7002的Eureka Server 需要去端口为  7001 Eureka Server 注册 自己 （7002）
      # defaultZone: http://eureka7001.com:7001/eureka
      # 三台 Eureka Server 搭建注册中心集群
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7003.com:7003/eureka
```

3. 注册中心三

```yml
server:
  port: 7003

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
  	hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      # 三台 Eureka Server 搭建注册中心集群
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

4. Eureak 客户端

```yml
# 配置Eureka客户端
eureka:
  client:
    # 表示是否将自己注册进EurekaServer，默认true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认true。单节点无所谓，集群必须设置为true，才能配合ribbon配合使用负载均衡
    fetch-registry: true
    service-url:
      # 注册中心的地址
      # defaultZone: http://localhost:7001/eureka # 单机版Eureka注册中心配置
      # 集群版Eureka注册中心配置
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka

```

5. 配置成功截图

![Snip20200729_7](/Users/luo/Documents/开发笔记/images/Snip20200729_7.png)

#### （5）消费者配置

1. 获取服务的名称

![Snip20200729_1](/Users/luo/Documents/开发笔记/images/Snip20200729_1.png)

2. 配置消费者调用

```java

@RestController
@RequestMapping("/consumer")
@Slf4j
public class OrderController {

    /**
     * 单机版
     * */
    //public static final String PAYMENT_URL = "http://127.0.0.1:8001";

    /**
     * 集群版：需要去注册中心获取服务
     * 还需要在restTemplate 上添加 @LoadBalanced 注解
     * 赋予 restTemplate 负载均衡的能力
     * */
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/payment/getAllPayment")
    public CommonResult getAllPayment() {

        log.info("order 中调用 payment 的获取所有 payment 订单服务");
        return this.restTemplate.getForObject(PAYMENT_URL + "/payment/getAllPayment", CommonResult.class);
    }

    @GetMapping("/payment/getPaymentById/{paymentId}")
    public CommonResult<Payment> getPaymentById(@PathVariable Long paymentId) {

        log.info("order 中调用 payment 的根据id查询 payment订单服务");

        return this.restTemplate.getForObject(PAYMENT_URL + "/payment/getPaymentById/" + paymentId, CommonResult.class);
    }

    @PostMapping("/payment/create")
    public CommonResult<Payment> create(Payment payment) {
        /**
         * 注意post方法参数和url是分开的
         * get方式，url中包含了请求参数
         * 发送 post 请求时，接收此请求的controller，需要使用 @RequestBody 来接收参数，否则接收不到
         * */
        log.info("客户要下的支付信息是："+payment);

        return this.restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
    }

}

```

#### （6）规范显示

1. 默认

![Snip20200729_2](/Users/luo/Documents/开发笔记/images/Snip20200729_2.png)

2. 定制显示

```yml
eureka:
  client:
    # 表示是否将自己注册进EurekaServer，默认true
    register-with-eureka: true
    # 是否从EurekaServer抓取已有的注册信息，默认true。单节点无所谓，集群必须设置为true，才能配合ribbon配合使用负载均衡
    fetch-registry: true
    service-url:
      # 注册中心的地址
      # defaultZone: http://localhost:7001/eureka # 单机版Eureka注册中心配置
      # 集群版Eureka注册中心配置
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    # 设置 eureka web页面 服务的名字，与调用此服务的消费者无关，消费者从来不调用 instance-id
    instance-id: payment${server.port}
    # instance-id 之前显示ip地址
    prefer-ip-address: true
```

3. 添加instance-id之后

![Snip20200729_3](/Users/luo/Documents/开发笔记/images/Snip20200729_3.png)

4. 添加ip前缀之后

   因为ip都是本机，所以不显示

![Snip20200729_5](/Users/luo/Documents/开发笔记/images/Snip20200729_5.png)

#### （7）Eureka的自我保护

1. 概述

保护模式主要用于一组客户端和Erueka Server之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server 将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是**不会注销任何微服务**

**某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存**

属于CAP里面的AP分支

2. 为什么会产生Eureka自我保护机制

防止Eureka Client可以正常运行，但是与Eureka Server网络不通的情况下，Eureka Server **不会立刻将Eureka Client 服务剔除**

3. 什么是自我保护模式

默认情况下，如果Eureka Server在一定时间内没有收到某个微服务实例的心跳，Eureka Server将会注销该实例（默认90秒）。但是当网络故障发生（延时，卡顿，拥挤）时，微服务与Eureka Server之间无法正常通信，以上行为可能变得非常危险了--因为微服务本身是健康的，此时不应该注销这个微服务。Eureka 通过自我保护模式来解决这个问题。当Eureka Server节点在短时间内丢失过多客户端时（可能发现了网络分区故障），那么这个节点就进入自我保护模式

**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例**

宁可保留错误的注册信息，也不盲目注销任何可能健康的服务实例

4. Eureka Server 配置

```yml
server:
  port: 7001

eureka:
  instance:
  #eureka 服务端的实例名称，service-url会调用这个名称作为ip
    hostname: localhost
  client:
    # false 表示不向注册中心注册自己
    register-with-eureka: false
    # false 表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka # 单机版
      # 端口为7001的Eureka Server 需要去端口为  7002 Eureka Server 注册 自己 （7001）
      # defaultZone: http://eureka7002.com:7002/eureka # 二台 Eureka Server 作为注册中心
      # 三台 Eureka Server 搭建注册中心集群
      # defaultZone: http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  server:
    # 关闭自我保护机制，保证不可用的服务被及时的清除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

5. Eureka Client 配置

```yml
server:
  port: 80
spring:
  application:
    name: cloud-order-service
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka # 单机版Eureka注册中心配置
      # 集群版Eureka注册中心配置
      # defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    # Eureka 客户端向注册中心发送心跳的时间间隔，单位秒（默认30）
    lease-renewal-interval-in-seconds: 1
    # Eureka 在收到最后一次心跳后等待的时间上限，单位为秒，默认90，超时后将剔除服务
    lease-expiration-duration-in-seconds: 2

```



### 2. DiscoveryClient 

(1)开启

```java
/**
 * 开启服务查看
 * */
@EnableDiscoveryClient
```

(2)使用

```java
@RestController
@RequestMapping("/payment")
@Slf4j
public class PaymentController {
  
    /**
     * 可以向外提供本微服务所提供的服务列表
     * 注入即可使用
     * */
    @Resource
    private DiscoveryClient discoveryClient;
		
      /**
     * 获取本服务所能提供的微服务列表
     * */
    @GetMapping("/discovery")
    public DiscoveryClient getAllDiscoveryClient(){
        List<String> services = this.discoveryClient.getServices();

        for (String service : services) {

            log.info(service);

            List<ServiceInstance> instances = this.discoveryClient.getInstances(service);

            for (ServiceInstance instance : instances) {
                log.info(instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
            }
        }

        return this.discoveryClient;
    }

}
```

### 3. zookeeper

### 4. consul

#### （1）安装启动

```shell
luo@luodeMacBook-Pro /opt % docker pull consul
luo@luodeMacBook-Pro /opt % docker create -p 8500:8500 --name consul consul
```

### 5. 三个注册中心的异同

* C:Consistency 强一致性
* A:Availabbility 可用性
* P:Partition tolerance 分区容错性，集群间的通信故障，导致主数据与副本数据不一致
* CAP理论关注颗粒度是数据，而不是整体系统设计的策略

**最多只能同时较好的满足两个**

CAP理论的核心是：**一个分布式系统不可能同时很好的满足一致性、可用性、分区容错性这三个要求**

因此。根据CAP原理将NoSQL数据库分为了满足CA原则，满足CP原则和满足AP原则三大类

CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大

CP - 满足一致性，分区容忍性的系统，通常性能不是特别高	(Zookeeper,Consul)

AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一点 (Eureka的自我保护机制，好死不如赖活着)

![Snip20200731_14](/Users/luo/Documents/开发笔记/images/Snip20200731_14.png)

![Snip20200731_13](/Users/luo/Documents/开发笔记/images/Snip20200731_13.png)

#### （1）AP 架构 (Eureka)

当网络分区出现后，为了保证可用性，系统b可以返回旧值，保证系统的可用性

结论：**违背了一致性C的要求，只满足可分区和容错即AP**

![Snip20200731_15](/Users/luo/Documents/开发笔记/images/Snip20200731_15.png)

#### （2）CP 架构

>如果集群间主从数据不一致，就应该让存老数据的服务器不可访问来保证访问时数据的一致性

![Snip20200731_16](/Users/luo/Documents/开发笔记/images/Snip20200731_16.png)

#### （3）满足p条件下，c、a二选一

分布式系统一定要满足`P(分区容错)`，因为网络一定会出现问题，

满足分区容错的条件下（p），若满足了可用行（a），但还要满足一致性的话（c）。当一个请求向集群中写数据时，一致性要求要所有节点的数据都写入并同步完成，才能返回成功，若一个节点通信失败，则整体回滚，并返回写入失败。所以cap无法同时做到

CA系统只有单节点系统能够做到

#### （4）分布式系统实现一致性的算法

`raft`算法，`paxos`算法

画图理解`raft`算法 http://thesecretlivesofdata.com/raft/

> 细节请看`谷粒商场 高级篇 184 商城业务-分布式业务-Cap Raft 算法原理` https://www.bilibili.com/video/BV1Tk4y117k1?p=184

***

## 三、服务调用

### 1. Ribbon

Spring Cloud Ribbon 是基于NetFlix Ribbon实现的一套 **客户端、负载均衡的工具**

一句话：**负载均衡 + RestTemplate 调用**

简单的说，Ribbon是NetFlix发布的开源项目，主要功能是提供**客户端的软件负载均衡算法和服务调用**。Ribbon客户端组件提供一系列完善的配置项，如连接超时，重试等。简单的说就是在配置文件中列出 **Load Balancer **（简称LB）后面的所有机器，Ribbon会自动的帮助我们基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法

Ribbon目前也进入了维护模式

#### （1）负载均衡间的区别

* Nginx 是服务器负载均衡，客户端所有的请求都会交给Nginx，然后由Nginx实现请求转发。即负载均衡是由服务端实现的，（集中式LB）
* Ribbon 本地负载均衡，在调用微服务接口的时候，会在注册中心上获取注册信息服务列表之后，缓存到JVM本地，从而在本地实现RPC远程服务调用的技术。（进程内LB）

* 集中式LB：即在服务的消费方和提供方之间使用独立的LB设施（可以是硬件，如F5，也可以是软件，如nginx），由该设施 负责把访问请求通过某种策略发送到服务的提供方
* 进程内LB：将LB逻辑集成到消费方，消费方从服务注册中心获取有哪些地址可用，然后自己再从这些地址中选出一个合适的服务器。Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址

#### （2） 架构说明

Ribbon其实就是一个软负载均衡的客户端组件，它可以和其他所需请求的客户端结合使用，和Eureka结合只是其中的一个实例

#### （3）工作步骤

1. 先选择Eureka Server，它优先选择在同一个区域内负载较少的server
2. 再根据用户指定的策略，再从server取到的服务注册列表中选择一个地址

#### （4）使用

```xml
        <!--引入Eureka 客户端依赖
					已经包含 spring-cloud-starter-netflix-ribbon ，所以不用再次引入ribbon
					-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>    
```

![Snip20200731_17](/Users/luo/Documents/开发笔记/images/Snip20200731_17.png)

#### （5）RestTemplate

```java
    @GetMapping("/payment/getPaymentById/{paymentId}")
    public CommonResult<Payment> getPaymentById(@PathVariable Long paymentId) {

        log.info("order 中调用 payment 的根据id查询 payment订单服务");

        /**
         * restTemplate.getForObject()
         * restTemplate.postForObject()
         * 返回对象为响应体中数据转化成的对象，基本上可以理解为json
         * */
        return this.restTemplate.getForObject(PAYMENT_URL + "/payment/getPaymentById/" + paymentId, CommonResult.class);
    }


    @GetMapping("/payment/getPaymentByIdUsingEntity/{paymentId}")
    public CommonResult<Payment> getPaymentByIdUsingEntity(@PathVariable Long paymentId) {


        ResponseEntity<CommonResult> entity = this.restTemplate.getForEntity(PAYMENT_URL + "/payment/getPaymentById/" + paymentId, CommonResult.class);

        log.info("调用provider之后，结果包含头信息，状态码等"+entity.getStatusCodeValue()+"\t"+entity.getHeaders());
        /**
         * restTemplate.getForEntity()
         * restTemplate.postForEntity()
         * 返回结果包含状态码，头信息，body
         * 真正有效的数据放在body中
         * 返回对象为 ResponseEntity 对象，包含响应中的一些重要信息，比如响应头，响应状态码，响应体等
         * */
        if(entity.getStatusCode().is2xxSuccessful()){
            return entity.getBody();
        }

        return new CommonResult<Payment>(444,"调用provider失败，状态码是："+entity.getStatusCodeValue(),null);


    }
```

#### （6）Ribbon核心组件IRule

IRule：根据特定算法从服务列表中选取一个要访问的服务

注意： AbstractLoadBalancerRule 抽象类持有对 ILoadBalancer 的引用

```java
public abstract class AbstractLoadBalancerRule implements IRule, IClientConfigAware {

    private ILoadBalancer lb;
        
    @Override
    public void setLoadBalancer(ILoadBalancer lb){
        this.lb = lb;
    }
    
    @Override
    public ILoadBalancer getLoadBalancer(){
        return lb;
    }      
}
```



![Snip20200731_18](/Users/luo/Documents/开发笔记/images/Snip20200731_18.png)

* com.netflix.loadbalancer.RoundRobinRule 

  轮询

* com.netflix.loadbalancer.RandomRule 

  随机

* com.netflix.loadbalancer.RetryRule 

  先按照轮询（RoundRobinRule）的策略获取服务，如果获取服务失败则在指定时间内会进行重试

* com.netflix.loadbalancer.WeightedResponseTimeRule 

  对轮询（RoundRobinRule）的扩展，响应速度越快的实例选择权重越大，越容易被选择

* com.netflix.loadbalancer.ResponseTimeWeightedRule 

  对轮询（RoundRobinRule）的扩展，响应速度越快的实例，越容易被选择

* com.netflix.loadbalancer.BestAvailableRule 

  会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

* com.netflix.loadbalancer.AvailabilityFilteringRule 

  先过滤故障实例，再选择并发较小的实例

* com.netflix.loadbalancer.ZoneAvoidanceRule 

  默认规则，复合判断server所在区域的性能和server的可用性，选择服务器

1. 配置时的注意事项

   这个自定义配置类不能放在@ComponentScan 所扫描的当钱包下以及子包下

   否则，我们自定义这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。也就是说，不这样配置，所有调用provider的负载均衡算法都被设置为这一个算法

   因为@SpringBootApplication 注解包含 @ComponentScan，所以Ribbon配置类不能放在@SpringBootApplication注解相同包，或者子包下

```java
public class RibbonClientConfiguration {
  	
  
  @Bean
	@ConditionalOnMissingBean	// 直接放入容器，即可替代这个
	public IRule ribbonRule(IClientConfig config) {
		if (this.propertiesFactory.isSet(IRule.class, name)) {
			return this.propertiesFactory.get(IRule.class, config, name);
		}
		ZoneAvoidanceRule rule = new ZoneAvoidanceRule();
		rule.initWithNiwsConfig(config);
		return rule;
	}
  
  /**
   * 自动配置类已经帮我们创建了 ILoadBalancer ，我们可以在自定义负载均衡器中，直接拿来使用
   * 但是拿不出来，容器中没有是怎么回事
   */
  @Bean
	@ConditionalOnMissingBean
	public ILoadBalancer ribbonLoadBalancer(IClientConfig config,
			ServerList<Server> serverList, ServerListFilter<Server> serverListFilter,
			IRule rule, IPing ping, ServerListUpdater serverListUpdater) {
		if (this.propertiesFactory.isSet(ILoadBalancer.class, name)) {
			return this.propertiesFactory.get(ILoadBalancer.class, config, name);
		}
		return new ZoneAwareLoadBalancer<>(config, rule, ping, serverList,
				serverListFilter, serverListUpdater);
	}
}
```

2. 配置步骤

【1】在springBoot启动类的父包及其以上创建配置类

![Snip20200731_19](/Users/luo/Documents/开发笔记/images/Snip20200731_19.png)

```java
/**
 * 定制化 IRule配置类
 * */
@Configuration
public class MyRuleConfiguration {

    @Bean
    public IRule randomRule(){
        return new RandomRule();
    }

}
```

【2】配置主启动类

```java
@SpringBootApplication
@EnableEurekaClient
/**
 * 主启动类上添加 @RibbonClient 注解
 * 在该服务（CLOUD-PAYMENT-SERVICE）启动的时候就会去加载我们自定义的IRule配置类（MyRuleConfiguration），从而使配置类生效
 * 然后就能使用我们在MyRuleConfiguration类中配置的负载均衡算法
 * */
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyRuleConfiguration.class)
public class CloudConsumerOrder80Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConsumerOrder80Application.class, args);
    }

}
```

3. 负载均衡算法

轮询负载均衡算法：rest接口第几次请求数 % 服务集群总数量 = 实际调用服务器位置下标，每次服务重启之后，rest接口计数从1开始

```java
// 轮询负载均衡算法实现：源码
public class RoundRobinRule extends AbstractLoadBalancerRule {
  			// modulo 是 provider的数量
  			// 多线程操作的CAS锁
  	    private int incrementAndGetModulo(int modulo) {
        for (;;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }
}
```

4. 手写负载均衡算法

【1】如果没有实现IRule接口的前提下，手写这个算法，需要取消RestTemplate 的 @LoadBalanced 注解，才能使用我们自己的负载均衡算法

```java
@Configuration
public class RestTemplateConfiguration {

    /**
     * 创建通过http调用远程服务的模版对象
     * 使用自定义负载均衡算法的时候，需要注释 @LoadBalanced
     * 若使用Ribbon自带的负载均衡算法，则需要添加 @LoadBalanced 注解
     * */
    @Bean
    @LoadBalanced // 我使用的是实现IRule接口，所以没有注释
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.build();
        //return new RestTemplate();
    }
}
```

【2】我的写法

```java
/**
 * 自定义Ribbon 的负载均衡策略
 * 自定义轮询算法
 */
@Component
@Slf4j
public class MyLoadBalanceConfiguration extends AbstractLoadBalancerRule {


    AtomicInteger atomicInteger = new AtomicInteger(0);

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }

    @Override
    public Server choose(Object key) {

        ILoadBalancer loadBalancer = super.getLoadBalancer();

        log.info("获取到的ILoadBalancer = " + loadBalancer);


        List<Server> reachableServers = loadBalancer.getReachableServers();

        int selectedServerIndex = selectServer(reachableServers.size());

        Server server = reachableServers.get(selectedServerIndex);

        log.info("本次实际调用的provider是：" + server.getHost() + ":" + server.getPort());

        return server;
    }

    /**
     * @param totalServerCount provider 总的数量
     * @return 本次调用provider，在list中的索引
     */
    private int selectServer(int totalServerCount) {

        int currentCount = 0;

        int nextCount = 0;
        /**
         * 自己写的第一个CAS轻量级锁
         * */
        do {
            currentCount = atomicInteger.get();

            /**
             * 整数最大是 2147483647 ，这样可以防止溢出
             * */
            nextCount = currentCount == 2147483647 ? 0 : currentCount + 1;


            /**
             * compareAndSet() 设置成功，返回true，取反，false，跳出循环
             * compareAndSet() 设置失败，返回false，取反，true，继续循环
             * */
        } while (!atomicInteger.compareAndSet(currentCount, nextCount));

        int nextIndex = nextCount % totalServerCount;

        log.info("当前是第" + currentCount + "次访问provider");

        return nextIndex;
    }
}
```

springBoot启动类

```java
@SpringBootApplication
@EnableEurekaClient
/**
 * 主启动类上添加 @RibbonClient 注解
 * 在该服务（CLOUD-PAYMENT-SERVICE）启动的时候就会去加载我们自定义的IRule配置类（MyRuleConfiguration），从而使配置类生效
 * 然后就能使用我们在MyRuleConfiguration类中配置的负载均衡算法
 * */
//@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyRuleConfiguration.class)
/**
 * 使用自定义负载均衡算法
 * */
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyLoadBalanceConfiguration .class)

public class CloudConsumerOrder80Application {

    public static void main(String[] args) {
        SpringApplication.run(CloudConsumerOrder80Application.class, args);
    }

}
```

启动类和自定义算法类的位置关系

![Snip20200731_2](/Users/luo/Documents/开发笔记/images/Snip20200731_2.png)

### 2. openFeign

#### （1）概述

Feign 是一个声明式WebService客户端。使用Feign能让编写Web Service 客户端更加简单

它的使用方法是**定义一个服务接口然后在上面添加注解**。Feign也支持可插拔式的编码器和解码器。spring cloud 对Feign 进行了封装，使其支持了Spring Mvc标准注解和HttpMessageConverters。Feign可以与EurekaRibbon组合使用以支持负载均衡

#### （2）Feign能干什么

Feign旨在使编写java http 客户端变得更容易

前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖调用可能不止一处，**往往一个接口被多处调用，所以通常都会针对每个微服务，自行封装一些客户端类来包装这些依赖服务的调用**，所以，Feign在此基础上做了进一步封装，由它来帮助我们定义和实现依赖服务接口的定义。

在Feign的实现下，**我们只需创建一个接口并使用注解的方式来配置它（以前是Dao接口上面标注Mapper注解，现在是一个微服务接口上面标注一个@Feign注解即可）**，即可完成对微服务提供方接口的绑定，简化了使用Spring Cloud Ribbon时，自动封装微服务调用客户端的开发量。

#### （3）Feign集成了Ribbon

利用Ribbon维护了Payment的服务列表信息，并通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，**通过Feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用**

#### （4）Feign 与 OpenFeign

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign 是 Spring Cloud 组件中一个轻量级Restful的Http服务客户端，Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方法是：使用Feign的注解定义的接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign是Spring Cloud在Fegin的基础上支持了SpringMvc的注解，如@RequestMapping等等。OpenFeign的@FeignClient可以解析SpringMvc的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |

#### （5）使用步骤

​	全部都是基于消费者的修改，provider不需要做任何改动

【1】主启动类上添加 @EnableFeignClients注解

```java
@SpringBootApplication
@EnableFeignClients
public class CloudConsumerOrder80EurekaOpenfeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(CloudConsumerOrder80EurekaOpenfeignApplication.class, args);
	}

}
```

【2】创建调用provider的接口，并使用@FeignClient(value = "CLOUD-PAYMENT-SERVICE")注解

```java
@Service
/**
 * 指定服务的 provider，不需要添加http
 * */
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentService {
		// 这个抽象方法和 provider controller 对应的方法一摸一样
    @PostMapping("payment/create")
    CommonResult<Payment> createPayment(@RequestBody Payment payment);
}
```

【3】controller中调用接口

```java
@RestController
@RequestMapping("/consumer")
public class OrderController {

    @Resource
    PaymentService paymentService;

    @PostMapping("/payment/create")
    public CommonResult<Payment> createPayment(Payment payment){


        CommonResult<Payment> payment1 = this.paymentService.createPayment(payment);

        return payment1;
    }

}
```

#### （6）超时配置

我的配置

```yml
server:
  port: 80
spring:
  application:
    name: cloud-consumer-order-service-openfeign
eureka:
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

# 设置 OpenFeign 客户端超时时间（OpenFeign默认支持Ribbon）
feign:
  hystrix:
    enabled: false # Feign是否启用断路器,默认为false
  client:
    config:
      default:
        connectTimeout: 10000 # Feign的连接建立超时时间，默认为10秒
        readTimeout: 6000 #Feign的请求处理超时时间，默认为60

# 设置feign客户端超时时间（OpenFeign默认支持Ribbon），低版本可用
#ribbon:
#  ReadTimeout: 1000     # 处理请求的超时时间，默认为1秒。建立连接后从服务器读取到可用资源所用的时间
#  ConnectTimeout: 4000  # 连接建立的超时时长，默认1秒
```

别人的详细解释

```properties
OpenFeign超时时长设置及详解

概念明确：

1 hystrix可配置的部分

hystrix.command.default.execution.timeout.enable=true //为false则超时控制有ribbon控制，为true则hystrix超时和ribbon超时都是用，但是谁小谁生效，默认为true
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=3000//熔断器的超时时长默认1秒，最常修改的参数
circuitBreaker.requestVolumeThreshold=20 //触发熔断的最小请求次数，默认为20
circuitBreaker.sleepWindowInMilliseconds=5000 //休眠时长，默认为5秒
circuitBreaker.errorThresholdPercentage=50 //触发熔断的失败请求最小占比，默认50%

2 ribbon的可配置部分

# 设置feign客户端超时时间（OpenFeign默认支持Ribbon）
ribbon.ReadTimeout=1000 //处理请求的超时时间，默认为1秒。建立连接后从服务器读取到可用资源所用的时间
ribbon.ConnectTimeout=1000 //连接建立的超时时长，默认1秒
ribbon.MaxAutoRetries=1 //同一台实例的最大重试次数，但是不包括首次调用，默认为1次
ribbon.MaxAutoRetriesNextServer=0 //重试负载均衡其他实例的最大重试次数，不包括首次调用，默认为0次
ribbon.OkToRetryOnAllOperations=false //是否对所有操作都重试，默认false

3 Feign的可配置部分

feign.hystrix.enabled=false //Feign是否启用断路器,默认为false
feign.client.config.default.connectTimeout=10000 //Feign的连接建立超时时间，默认为10秒
feign.client.config.default.readTimeout=60000 //Feign的请求处理超时时间，默认为60
feign.client.config.default.retryer=feign.Retryer.Default //Feign使用默认的超时配置，在该类源码中可见，默认单次请求最大时长1秒，重试5次
 

另外以上各种超时配置，如果都存在，则时间小的配置生效
```

好的，现在来说Feign的超时时长设置：

1 Feign的默认配置，是不启用hystrix，并且Feign的底层是调用ribbon来实现负载均衡的，所以为了不和ribbon的重试机制冲突因此也不会启用重试机制

因此配置Feign是必须要做的就是

```
feign.hystrix.enabled=true //开启Feign的hystrix，这样配置文件中的hystrix的配置才会生效，否则依然是默认的规则
```

之后设置hystrix的相关配置才可以在Feign中生效，因为Feign也调用了hystrix

2 ribbon和hystrix的配置

因为hystrix的超时时长，默认为1秒，太短了！因此我们一般一定会设置hystrix的超时时长

　在上面启用了Feign的hystrix开关后，配置hystrix超时时长

```
execution.isolation.thread.timeoutInMilliseconds=10000 //这里设置了10秒
```

然后看ribbon的超时设置：

ribbon的超时设置无非2个：处理超时和连接超时，默认为1秒和1秒

但是，ribbon是有默认重试的，也是2个：统一实例的重试次数和负载均衡的不同实例的重试次数，默认为1次和0次

也就是说，在同一个实例上建立连接如果失败可以重试1次，处理请求如果失败可以重试1次，但是不包括首次调用，即：实际ribbon的超时时间是 1秒×2+1秒×2，即4秒

之后是，但是上面设置了hystrix超时为10秒，因此ribbon超时优先生效

最后是1个是否对所有操作重试的开关，默认为false,这里解释下什么是所有操作：

即：为false是，GET请求不论是连接失败还是处理失败都会重试，而对于非GET请求只对连接失败进行重试

#### （7）日志打印

Feign提供了日志打印功能，可以通过配置来调整日志级别，从而了解Feign中的Http请求细节

**对Feign接口的调用情况进行监控和输出**

* NONE：默认，不显示任何日志
* BASIC：仅记录请求方法，URL，响应状态码及执行时间
* HEADERS：出了BASIC中定义的信息之外，还有请求和响应的头信息
* FULL：出了HEARDS中定义的信息之外，还有请求和响应的正文及元数据

```java
@Configuration
public class FeignConfiguration {
    
    /**
     * 注意是 feign.Logger.Level 
     * 不是常规的日志
     * */
    @Bean
    public Logger.Level feignLogger(){
        return Logger.Level.FULL;
    }

}
```

还需要在yml文件中配置日志级别

```yml
logging:
  level:
    # Feign 以什么级别监控那个Feign 注解的的接口
    # 这里配置的是 包下所有接口
    com.example.cloudconsumerorder80eurekaopenfeign.service: debug
```

### 3.feign出现字符串格式问题

feign使用中报错处理JSON parse error: Illegal character ((CTRL-CHAR, code 31)): only regular white space...

解决方式锁了好多，最终在发送请求的时候添加了
@Headers({“acceptEncoding: gzip”,“contentType: application/json”})

```java
 @PostMapping(value = "staffVehicle/**/**", consumes = "application/json")
    // 这句是新加的，然后就好了
    @Headers({"acceptEncoding: gzip","contentType: application/json"})
    void saveVehicle(@RequestBody VehicleDoorAliasSaveVo vo);
————————————————
版权声明：本文为CSDN博主「没穿胖次的猴子」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_36175946/article/details/114270579
```

这是fegin的一个bug，目前已经解决了，原文链接https://github.com/OpenFeign/feign/issues/934

后来发现还是有问题。然后根据网上的说法删除feign配置中的compression段

```yaml
# feign 配置
feign:
  sentinel:
    enabled: true
  hystrix:
    enabled: true
  okhttp:
    enabled: true
  httpclient:
    enabled: false
  client:
    config:
      default:
        connectTimeout: 10000
        readTimeout: 60000
 //下面的删除掉，或者设置为false
  compression:
    request:
      enabled: true
    response:
      enabled: true
```

## 四、服务降级

### 1. 分布式系统面临的问题

复杂分布式系统结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免的失败

![Snip20200731_3](/Users/luo/Documents/开发笔记/images/Snip20200731_3.png)

### 2. 服务雪崩

多个服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他服务，这就是所谓的**扇出**。如果扇出的链路上某个微服务的调用和响应时间太长或者不可用，对微服务A的调用就会占用越来越多的系统资源进而引发系统崩溃，所谓的雪崩效应

对于高流量应用来说，单一的后端依赖会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩

### 3. Hystrix简介

Hystrix 是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时，异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失败，避免级联故障，提高分布式系统的弹性**

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过短路器的故障监控（类似于熔断保险丝），**向调用方返回一个符合预期的、可处理的备选响应（FallBack） ,而不是长时间的等待或者抛出调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要的占用，从而避免了故障在分布式系统中蔓延，乃至雪崩。

### 4. Hystrix 重要概念

#### （1）服务降级 FallBack

​		对方系统（某Provider节点）不可用了，你要给我想办法搞个兜底的解决方法

​		服务器繁忙，请稍后重试，不让客户端等待并立即返回一个友好的提示

* 哪些情况会触发FallBack
  * 程序运行异常
  * 超时
  * 服务熔断触发服务降级
  * 线程池、信号量打满也会导致服务降级

#### （2）服务熔断 Break

​			类似保险丝达到最大服务访问之后，直接拒绝访问，拉闸限电，然后调用服务降级的方法返回友好提示，已经		运行中的服务则不受影响

​			服务降级 -> 进而熔断 -> 恢复调用链路

#### （3）服务限流 FlowLimit

​			秒杀、高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

### 5. 高并发压力测试

![Snip20200731_6](/Users/luo/Documents/开发笔记/images/Snip20200731_6.png)

![Snip20200731_7](/Users/luo/Documents/开发笔记/images/Snip20200731_7.png)

可见，tomcat的默认工作线程数倍打满了，没有多余的线程来分解压力和处理

结论：上面还只是provider 8001自己测试，加入此时外部的消费者80也来访问，那么消费者只能等待，最终导致消费端不满意，服务端8001直接被拖死

### 6. 解决的要求

* 超时导致服务器变慢（浏览器转圈）：超时不等待
* 出错（宕机或者程序运行出错）：出错要有兜底
* 解决：
  * 对方服务8001超时了，调用者 80 不能一致卡死等待，必须有服务降级
  * 对方服务8001宕机了，调用者 80 不能一直卡死等待，必须有服务降级
  * 对方服务8001 ok，调用者 80 自己出故障或者有自我要求（自己的等待时间小于服务提供者，调用者只能等待3秒，但是 provider 要5秒才能完成），80 就要自己降级

### 7. 服务降级

#### （1）降级配置

​	修改 @HystrixCommand 之后要手动重启java代码，使用Devtools重启无效

* 8001先从自生找问题

  设置自身调用超时时间的最大值，最大值内可以正常运行，超时了需要有兜底的方法处理，做服务降级FallBack

* 8001 fallback

  * 业务类启用
    * @HystrixCommand 报异常之后该如何处理
      * 一旦调用服务方法失败并抛出了错误信息之后，会自动调用@HystrixCommand标注好的fallbackMethod调用本类中指定的方法
  * 主启动类激活
    * 主启动类上启用 @EnableCircuitBreaker

* 80 fallback

  * 和 8001fallback类似，只不过放在了客户端80

  * 消费端的Hystrix配置需要在yml中开启Hystrix

    ```yml
    # 消费端使用服务降级需要在这里开启
    feign:
      hystrix:
        enabled: true
    ```
    
  * 主启动类上启用 
  
    ```java
    @EnableFeignClients
    @EnableHystrix
    ```

#### （2）目前的问题

​	【1】每个业务方法对应一个兜底方法，代码膨胀

​	【2】controller 和 服务降级 代码高耦和

​	【3】业务代码和兜底方法放在一个类里面，不好

#### （3）解决代码膨胀

使用默认的服务降级方法，如果不使用默认fallback方法，则可以指定fallback方法，二者不冲突

```java
@RestController
@RequestMapping("/consumer/default")
@Slf4j
/**
 * 配置全局服务降级
 * @DefaultProperties(defaultFallback = "timeoutHandler")
 * 可以使得所有有 @HystrixCommand 注解的方法，一旦超时、异常等，就默认调用其指定的兜底方法
 * */
@DefaultProperties(defaultFallback = "globalHandler")
public class DefaultHystrixOrderController {

    @Resource
    PaymentService paymentService;


    @HystrixCommand(fallbackMethod = "timeoutHandler", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
    })
    @GetMapping("/timeout")
    public String timeout() {
        String s = this.paymentService.paymentTimeout();

        log.info("调用超时服务");

        return s;
    }

    public String timeoutHandler() {

        String name = Thread.currentThread().getName();

        return name + "我是消费者80，对方支付系统繁忙，请10秒中之后再重试，😭😭";
    }

    /**
     * 出异常会调用全局fallback
     * */
    @HystrixCommand
    @GetMapping("/add")
    public String add(){
        int i = 10/0;
        return "计算成功";
    }


    /**
     * 全局fallBack方法
     * */
    public String globalHandler() {

        String name = Thread.currentThread().getName();

        return name + "我是消费者80，对方支付系统繁忙，请10秒中之后再重试，😭😭😭😭";
    }


    @GetMapping("/normal")
    public String normal() {
        String s = this.paymentService.paymentNormal();

        log.info("消费者调用正常服务");

        return s;
    }

}
```

#### （4）解耦

消费者调用 provider ，provider 此时宕机

此次在消费者端解决，与provider没有关系

只需要为@Feign客户端定义的接口添加一个服务降级处理的实现类即可实现解耦

【1】服务接口配置

```java
/**
 * 在Feign负载均衡中指定服务降级的fallback方法绑定类
 * */
@Service
@FeignClient(value = "CLOUD-PAYMENT-SERVICE-HYSTRIX",fallback = PaymentServiceWithFallbackImpl.class)
public interface PaymentServiceWithFallback {

    @GetMapping("/payment/paymentTimeout")
    public String paymentTimeout();


    /**
     * 这里超时当错错误处理
     * */
    @GetMapping("/payment/paymentTimeout")
    public String paymentError();

}
```

【2】服务接口实现类写fallback方法

```java
/**
 * 一定要放入IOC容器中
 * */
@Service
/**
 * 不可以使用默认fallback
 * */
@DefaultProperties(defaultFallback = "fallbackInServiceImpl")
@Slf4j
public class PaymentServiceWithFallbackImpl implements PaymentServiceWithFallback {

    /**
     * @FeignClient 修饰接口中的抽象方法服务降级之后，会调用这里重写抽象方法的方法作为fallback方法
     * */
    @Override
    public String paymentTimeout() {

        return "paymentTimeout() 抽象方法 的具体实现，可以定制fallback函数";
    }


    @Override
    public String paymentError() {
        return null;
    }


    public String fallbackInServiceImpl(){

        String info = "通过绑定接口实现类来实现降级,解耦后的降级";
        log.info(info);

        return info;
    }
}
```

【3】controller和普通的controller没有任何区别

Controller 没有做任何的额外设置或者添加

```java

/**
 * 将fallback 服务降级与controller解耦，把降级的业务逻辑写在service业务逻辑中
 *
 * */
@RestController
@Slf4j
@RequestMapping("/consumer/fallbackByService")
public class FallBackInServiceController {

    @Resource
    PaymentServiceWithFallback paymentServiceWithFallback;


    @GetMapping("/timeout")
    public String timeout() {
        String s = this.paymentServiceWithFallback.paymentTimeout();

        log.info("调用超时服务");

        return s;
    }

    @GetMapping("/payError")
    public String payError(){
        String s = this.paymentServiceWithFallback.paymentError();

        return s;
    }


}
```

【4】总结：

此时provider已经down了，但是我们在消费端做了服务降级处理，让消费端在provider端不可用的同时也会获得提示信息，而不会消费端挂起，耗死服务器

### 8. 服务熔断

#### （1）概述

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错而不可用或者响应时间太长时，会进行服务的降级，进而该节点微服务的调用，快速返回错误的响应信息。

当检测到该节点微服务调用响应正常之后，恢复调用链路。

在Spring Cloud框架里，熔断机制是通过Hystrix实现。Hystrix会监控服务间调用的情况。

当失败的调用达到一定的阈值，（默认是5秒内的20次调用失败），就会启动熔断机制，熔断机制的注解是@HystrixCommand

![Snip20200801_1](/Users/luo/Documents/开发笔记/images/Snip20200801_1.png)

#### （2）配置

![Snip20200801_2](/Users/luo/Documents/开发笔记/images/Snip20200801_2.png)

所有的配置参数都可以在 HystrixCommandProperties 中找到

```java
/**
 * Properties for instances of {@link HystrixCommand}.
 * <p>
 * Default implementation of methods uses Archaius (https://github.com/Netflix/archaius)
 */
public abstract class HystrixCommandProperties {
}
```

我的配置

```java
    /***************************         服务熔断     *************************/


    @HystrixCommand(fallbackMethod = "idError", commandProperties = {
            /**
             * 是否启用断路器
             * */
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),

            /**
             * Threshold : 门槛
             * requestVolumeThreshold: 触发熔断器的最小请求次数
             * 如果滚动时间窗（默认10秒），仅收到了9个请求，即使这9个请求都失败了，熔断器也不会打开
             * */
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),

            /**
             * 休眠时间窗口期
             * 该属性用来设置断路器打开之后的休眠时间窗。休眠时间窗结束之后，
             * 会将断路器设置为 "半开" 状态，尝试被熔断的请求，如果依然失败就把断路器继续设置为 "打开" 状态，
             * 如果成果就设置为 "关闭" 状态
             * */
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),

            /**
             * 失败率达到多少之后跳闸
             * 该属性用来设置在滚动时间窗口期，在请求数量超过 circuitBreaker.requestVolumeThreshold 的条件下，
             * 且请求的错误数的百分比超过 60%，就把断路器设置为"打开"状态，
             * 否则设置为"关闭状态"
             * */
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"),
            /**
             * 相当于：在时间窗口期内（10000毫秒），10次请求中，有60%的请求，响应失败，就启动熔断，熔断后，请求走的是fallback
             * 在时间窗口期内（10000毫秒），单个请求让其通过，尝试服务是否已经可以正常使用
             * 如果这单个请求正常执行，则Hystrix则会慢慢的恢复链路
             * 服务降级 -> 熔断 -> 逐渐恢复链路
             * 测试步骤
             * 1. 刷新10次 http://localhost:8001/payment/circuitBreak/selectPaymentById/-4545
             * 2. 进入     http://localhost:8001/payment/circuitBreak/selectPaymentById/2
             * 发现 id 为 2 是正常的，但是依然服务降级，调用的是fallback方法
             * 3. 再刷新一次 http://localhost:8001/payment/circuitBreak/selectPaymentById/2
             * 发现正常
             *
             * 多次错误，然后满满的正确，发现刚开始时不满足条件，就算是正确的访问也不能进行，之后服务就恢复了
             * */
    })
    @Override
    public String getPaymentById(Integer paymentId) {

        if (paymentId < 0) {
            throw new RuntimeException("id 不可以是负数");
        }

        String s = IdUtil.simpleUUID();

        return "根据id：" + paymentId + "查询到的账单是" + s;
    }


    /**
     * fallback方法
     */
    public String idError(Integer paymentId) {

        String error = paymentId + "查询的主键不能是负数";
        log.error(error);

        return error;
    }
```

#### （3）熔断结论

* 熔断打开：请求不再进行调用当前正常服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设置时钟则进入半熔断状态
* 熔断关闭：熔断关闭不会对当前服务进行熔断
* 熔断半开：部分请求根据规则调用当前服务（一般是熔断窗口期内调用一次服务），如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

#### （4）官方熔断流程图

![Snip20200801_3](/Users/luo/Documents/开发笔记/images/Snip20200801_3.png)

#### （5）断路器起作用的条件

涉及到三个参数

* 快照时间窗

  断路器确定是否打开，需要统计一些请求和错误数据，而统计时间范围就是快照窗口期，默认是最近的十秒

* 请求总数阈值

  在快照时间窗内，必须满足请求总数阈值才有资格熔断。默认为20。意味着在10秒内，如果该hystrix命令的调用次数不足20次，即使所有请求都超时或者其他原因失败，断路器都不会打开。

* 错误百分比阈值：当请求总数在快照时间窗内超过了阈值，这时，断路器才会打开。比如发生了30次调用，如果30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认50%阈值的情况下，这时候就会将断路器打开。

```java
            /**
             * 是否启用断路器
             * */
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),

            /**
             * Threshold : 门槛
             * requestVolumeThreshold: 触发熔断器的最小请求次数
             * 如果滚动时间窗（默认10秒），仅收到了9个请求，即使这9个请求都失败了，熔断器也不会打开
             * */
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),

            /**
             * 休眠时间窗口期
             * 该属性用来设置断路器打开之后的休眠时间窗。休眠时间窗结束之后，
             * 会将断路器设置为 "半开" 状态，尝试被熔断的请求，如果依然失败就把断路器继续设置为 "打开" 状态，
             * 如果成果就设置为 "关闭" 状态
             * */
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),

            /**
             * 失败率达到多少之后跳闸
             * 该属性用来设置在滚动时间窗口期，在请求数量超过 circuitBreaker.requestVolumeThreshold 的条件下，
             * 且请求的错误数的百分比超过 60%，就把断路器设置为"打开"状态，
             * 否则设置为"关闭状态"
             * */
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"),
```

#### （6）断路器关闭的条件

* 当满足一定阈值的时候（默认10秒内超过20个请求次数）
* 当失败率达到一定的时候（默认10秒内超过50%的请求失败）
* 达到以上阈值，断路器就会开启
* 当开启的时候，所有的请求都不会转发，而是直接走fallback方法
* 一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，如果失败，继续开启，重复步骤4 和步骤5

#### （7）断路器打开之后

* 再有请求调用的时候，将不会调用正常逻辑业务，而是直接调用降级fallback。通过断路器实现了自动地发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果。

* 如果恢复原来的主逻辑

  Hystrix为我们实现了自动恢复功能	

  当断路器打开，对主逻辑进行熔断之后，断路器进入半开状态，快照窗口期内会释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将闭合，主逻辑恢复；如果这次请求依然有问题，断路器将继续进入打开状态，休眠窗口期重新计时

### 9. 服务限流

​	Alibaba Sentinel 中再做讲解

### 10. Hystrix 工作流程

https://github.com/Netflix/Hystrix/wiki/How-it-Works

![hystrix-command-flow-chart](/Users/luo/Documents/开发笔记/images/hystrix-command-flow-chart.png)

![Snip20200801_4](/Users/luo/Documents/开发笔记/images/Snip20200801_4.png)

 ### 11. Hystrix Dashboard

监控端和被监控端的 spring boot 版本 和 spring cloud 版本必须一致

使用spring 脚手架创建的Spring boot 和 spring cloud 混合工程，spring cloud 在 pom文件中是以 dependenciesManager 标签来引入的，其中携带版本信息，将脚手架创建的工程改为集成父工程时，需要删除此dependenciesManager 标签，不然spring cloud 版本 还是不一致

#### （1）被监控端

```xml
        <!--
        使用Hystrix Dashboard 监控 hystrix ，必须导入此依赖
                还有web启动器
        -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

#### （2）监控端

```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
		</dependency>
```

```java
@SpringBootApplication
@EnableHystrixDashboard
/**
 * 监控wbe页面 http://localhost:9001/hystrix
 * */
public class CloudProvider9001HystrixDashboardApplication {

	public static void main(String[] args) {
		SpringApplication.run(CloudProvider9001HystrixDashboardApplication.class, args);
	}

}
```

![Snip20200802_1](/Users/luo/Documents/开发笔记/images/Snip20200802_1.png)

## 五、服务网关

### 1. gateway 新一代网关

#### （1）概念

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuul网关，但是在2.x版本中，zuul的升级一直跳票，spring cloud最后自己研发了一个网关替代zuul，gateway是 zuul 1.x版本的替代。

Gateway 是在 spring 生态系统之上构建的API网关服务，基于spring 5， spring boot 2.x 和 project Reactor 等技术。

Gateway 旨在提供一种简洁而有效的方式来对API进行路由，以及提供一些强大的过滤功能，例如：熔断、限流、重试等

Spring cloud gateway 作为 spring cloud 生态系统中的网关，目标是替代zuul，在Spring cloud 2.0 以上的版本中，没有对新版本zuul 2.0 以上最新高性能版本进行集成，仍然还是使用zuul 1.x 非 reactor模式的老版本。而为了提升网关的性能，spirng cloud gateway是基于 **WebFlux**框架实现的，而webflux 框架底层则使用了高性能的 reacctor 模式通信框架 Netty

Spring cloud gateway 的目标是提供统一的路由方式且基于filter 链的方式提供网关基本的功能，例如：安全，监控、指标，和限流。

**spring cloud gateway 使用 webflux 中的 reactor-netty 响应式编程组件，底层使用了 netty 通讯框架**

![diagram-microservices-88e01c7d34c688cb49556435c130d352](/Users/luo/Documents/开发笔记/images/diagram-microservices-88e01c7d34c688cb49556435c130d352.svg)

#### （2）特性

* 基于 spring framework 5, project reactor ,spring boot 2.x进行构建
* 动态路由：能够匹配任何请求属性
* 可以对路由指定Predicate（断言）和Filter （过滤器）
* 集成Hystrix的断路器功能
* 集成spring cloud 服务发现功能
* 易于编写Predicate（断言）和Filter（过滤器）
* 请求限流功能
* 支持路径重写

#### （3）gateway 和 zuul 1.x 的区别

spring cloud finchley 正式版之前，spring cloud 推荐的网关是 netflix 提供的zuul

* zuul 1.x 是一个基于 **阻塞I/O**的API gateway
* zuul 1.x  **基于servlet 2.5使用阻塞架构**，它不支持任何长连接（如webSocket），zuul的设计模式和nginx比较像，每次io操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但差别是nginx是用c++实现，zuul用java实现，而jvm本身会有第一次加载较慢的情况，使得zuul的性能相对较差
* zuul 2.x 理念更先进，想基于Netty非阻塞和支持长连接，但是spring cloud 目前还没有整合。zuul 2.x 较 zuul 1.x有较大提升，在性能方面，根据官方提供的基准测试，Spring cloud gateway 的RPS（每秒请求数）是zuul的1.6倍
* spring cloud gateway 建立在 Spring framework，project reactor，Spring boot 2.x之上，使用非阻塞API
* Spring cloud gateway 还支持webSocket，并且与spring 紧密集成有更好的开发体验

#### （4）Zulu 1.x 模型

* F版之前的 spring cloud 中所集成的zuul版本，采用的是tomcat容器，使用的是传统的Servlet IO 处理模型
* Servlet的生命周期，servlet由servlet container 进行生命周期管理
* container 启动时构造servlet对象并调用servlet init()进行初始化
* container 运行时接收请求，并为每一个请求分配一个线程（一般从线程池中获取空闲线程），然后调用service()
* container 关闭时调用servlet destroy()销毁servlet

![Snip20200802_2](/Users/luo/Documents/开发笔记/images/Snip20200802_2.png)

上述模式的缺点：

* servlet 是 一个简单的网络IO模型，当请求进入servlet container时，servlet container 就会为请求绑定一个线程，在**并发不高的场景下**，这种模型是适用的。但是一旦高并发（比如用apache jemeter压），线程数量就会暴涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。在一些简单业务场景下，不希望为每一个request分配一个线程，只需要一个或者几个线程就能应对极大的并发请求，这种业务场景下，servlet没有优势
* 所以zuul 1.x 是**基于servlet之上的一个阻塞式处理模型**，即spring 实现了所有request请求的一个servlet（DispacherServlet）并由该servlet阻塞处理，所以该Spring cloud zuul无法摆脱servlet 模型的弊端

#### （5）Spring webFlux

传统的web框架，比如说：struts2，Spring mvc 等都是基于servlet API与servlet container 基础之上运行的。

但是，**在servlet 3.1之后有了异步非阻塞的支持**。而webFlux是一个典型非阻塞异步的框架，他的核心是基于reactor的相关API实现的。相对于传统web框架来说，他可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程（Spring 5 必须让你使用java8）

Spring webflux 是spring 5.0引入的新的响应式框架，区别于spring mvc，它不需要依赖servlet api，他是**完全异步非阻塞**的，并且有reactor来实现响应式流程规范

#### （6）核心概念

##### 【1】Route 路由

​	路由是构建网关的基本模块，它由ID，目标URI，一系列的断言、过滤器组成，如果断言为true，则匹配该路由

##### 【2】Predicate 断言

​	参看的是java8的 java.util.function.Predicate

​	开发人员可以匹配Http请求中的所有内容（例如请求头或请求参数），**如果请求与断言相匹配则进行路由**

##### 【3】Filter 过滤

​	指的是spring 框架中 GateWayFilter 的实例，使用过滤器，可以请求被路由前或者之后对请求进行修改



##### 【4】总体

web请求，通过一些匹配条件，定位到真正的服务节点，并在这个转发过程的前后，进行一些精细化控制。

Predicate 就是我们的匹配条件；而Filter，就可以理解为一个无所不能的拦截器。

有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

#### （7）工作流程

![spring_cloud_gateway_diagram](/Users/luo/Documents/开发笔记/images/spring_cloud_gateway_diagram.png)

Clients make requests to Spring Cloud Gateway. If the Gateway Handler Mapping determines that a request matches a route, it is sent to the Gateway Web Handler. This handler runs the request through a filter chain that is specific to the request. The reason the filters are divided by the dotted line is that filters can run logic both before and after the proxy request is sent. All “pre” filter logic is executed. Then the proxy request is made. After the proxy request is made, the “post” filter logic is run.

* 客户端向spring cloud gateway 发出请求。然后再gateway handler mapping 找到与请求匹配的路由，并将其发送到gateway handler mapping

* Handler 再通过指定的过滤器来将请求发送到我们实际的服务，执行业务逻辑，然后再返回

* 过滤器之间用虚线分开是因为可能再发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑
* Filter 在 “pre” 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等
* Filter 在 “post” 类型的过滤器中可以做响应内容、响应头的修改、日志的输出、流量监控等

#### （8）核心逻辑

路由转发 和 执行过滤器链

#### （9）运行gateway

```xml
<!--spring-cloud-starter-gateway 不可以引入 spring-boot-starter-web -->
        <!-- 引入gateway依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

#### （10）网关配置

##### 【1】yml文件中配置

```yml
spring:
  application:
    name: cloud-gateway-gateway-9527-consul

  cloud:
    # 配置网关
    gateway:
      routes:
        - id: payment_routh_0               # 路由的id，没有固定的规则，但要求唯一，建议配合服务名
          uri: http://localhost:8001        # 匹配后，提供路由服务的地址
          predicates:
            - Path=/payment/paymentNormal   # 断言，路径相匹配的进行路由

        - id: payment_routh_1
          uri: http://localhost:8001
          predicates:
            - Path=/payment/circuitBreak/selectPaymentById/*
```

##### 【2】代码中注入  RouteLocator 的 Bean

```java
/**
     * 配置了一个 ID 为  path_route_bilibili_1
     * 访问 http://localhost:9527/donghua 就可以跳转到 https://www.bilibili.com/v/douga
     */
    @Bean
    public RouteLocator customizeRouteLocator(RouteLocatorBuilder builder) {

        RouteLocatorBuilder.Builder route1 = builder.routes();

        // 路径不要忘记加斜杠 /

        route1.route("path_route_bilibili_1", r ->
                r.path("/donghua").uri("https://www.bilibili.com/v/douga")
        );

        RouteLocator routeLocator = route1.build();


        return routeLocator;
    }

    @Bean
    public RouteLocator ladyNewsRouteLocator(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();

        RouteLocatorBuilder.Builder route = routes.route("lady_id", r ->
                r.path("/lady").uri("https://news.baidu.com/lady")
        );

        return route.build();

    }
```

缺点：微服务的地址被写死，不能实现负载均衡

#### （11）通过服务名实现动态路由

​	默认情况下GateWay会根据注册中心的服务列表，以注册中心上微服务名为路径，**创建动态路由进行转发，从而实现动态路由功能**

​	此时可以通过gateway来实现负载均衡

![Snip20200802_4](/Users/luo/Documents/开发笔记/images/Snip20200802_4.png)

```java
spring:
  application:
    name: cloud-gateway-gateway-9527

  cloud:
    # 配置网关
    gateway:
      # 配置负载均衡
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      # 配置路由
      routes:
        - id: payment_routh_0               # 路由的id，没有固定的规则，但要求唯一，建议配合服务名
          # uri: http://localhost:8001        # 匹配后，提供路由服务的地址 （写死，不好）
          uri: lb://CLOUD-PAYMENT-SERVICE-HYSTRIX # 匹配后，提供路由服务的地址 （根据注册中心解析服务名，再调用provider）
          predicates:
            - Path=/payment/paymentNormal   # 断言，路径相匹配的进行路由

        - id: payment_routh_1
          #uri: http://localhost:8001
          uri: lb://CLOUD-PAYMENT-SERVICE-HYSTRIX
          predicates:
            - Path=/payment/circuitBreak/selectPaymentById/*
```

#### （12）Predicate

[RoutePredicateFactory](https://docs.spring.io/spring-cloud-gateway/docs/3.0.0-M3/reference/html/#configuring-route-predicate-factories-and-gateway-filter-factories)

Spring Cloud Gateway matches routes as part of the Spring WebFlux `HandlerMapping` infrastructure. Spring Cloud Gateway includes many built-in route predicate factories. All of these predicates match on different attributes of the HTTP request. You can combine multiple route predicate factories with logical `and` statements.

![Snip20200802_3](/Users/luo/Documents/开发笔记/images/Snip20200802_3.png)

<img src="/Users/luo/Documents/开发笔记/images/RoutePredicateFactory.png" alt="RoutePredicateFactory" style="zoom:200%;" />

##### 【1】日期类

生成日期

* BeforeRoutePredicateFactory
* AfterRoutePredicateFactory    可用作微服务提前上线，但到指定时间才开始服务
* BetweenRoutePredicateFactory

```java
		ZonedDateTime now = ZonedDateTime.now();
		System.out.println(now);
```

```yml
            - After=2020-08-02T15:38:27.281+08:00[Asia/Shanghai] # 在这个日期之后才能匹配。生成日期的方法 ZonedDateTime now = ZonedDateTime.now();
						- Before=2020-08-02T15:38:27.281+08:00[Asia/Shanghai]
						- Between=2020-08-02T15:38:27.281+08:00[Asia/Shanghai],2020-08-02T15:38:27.281+08:00[Asia/Shanghai]
```

##### 【2】CookieRoutePredicateFactory

需要两个参数，一个是cookie name，一个是正则表达式

路由规则会通过获取对应的 cookie name值 和正则表达式去匹配，如果匹配上就会执行路由，如果没匹配上就不执行

```yml
            - Cookie=username,luojunhua
```

*  不带cookie的访问

  ```shell
  luo@luodeMacBook-Pro ~ % curl http://localhost:9527/payment/paymentNormal
  404 错误
  ```

* 带cookie访问

  ```shell
  luo@luodeMacBook-Pro ~ % curl http://localhost:9527/payment/paymentNormal --cookie username="luojunhua"
  我是正常的服务（低延迟），由线程：http-nio-8002-exec-9提供服务，共耗时：0毫秒，由8002提供服务%
  ```

##### 【3】HeaderRoutePredicateFactory

两个参数，一个是属性名称，一个是正则表达式，这个属性值和正则表达式匹配则执行

```yml
            - Header=X-Request-id,\d+   # 请求头中含有X-Request-id，并且值为正整数的正则表达式才放行
```

```shell
luo@luodeMacBook-Pro ~ % curl http://localhost:9527/payment/paymentNormal -H "X-Request-id:998"
我是正常的服务（低延迟），由线程：http-nio-8002-exec-7提供服务，共耗时：0毫秒，由8002提供服务%
```

##### 【4】MethodRoutePredicateFactory

```yml
          # - Method=get 只放行get请求
```

```shell
luo@luodeMacBook-Pro ~ % curl http://localhost:9527/payment/paymentNormal
我是正常的服务（低延迟），由线程：http-nio-8001-exec-7提供服务，共耗时：0毫秒，由8001提供服务%
```

##### 【5】QueryRoutePredicateFactory

```yml
            - Query=userId,\d+  # 请求中带有参数userId，且这个参数的值为正整数
```

```shell
luo@luodeMacBook-Pro ~ % curl "http://localhost:9527/payment/paymentNormal?userId=2"
我是正常的服务（低延迟），由线程：http-nio-8002-exec-10提供服务，共耗时：0毫秒，由8002提供服务%
```

##### 【6】HostRoutePredicateFactory

HostRoutePredicate 接收一组参数，一组匹配的域名列表，这个模版是一个ant分隔的模版，用逗号（.）作为分隔符，它通过参数中的主机地址作为匹配规则。

```yml
- Host=*.*.somehost.org,*.*.anotherhost.org
```

##### 【7】总结

Predicate就是为了实现一组匹配规则（与逻辑），让请求过来，并找到对应的Route进行处理

#### （13）GatewayFilter 过滤器

路由过滤器可以用于修改进入的Http请求和返回http响应，路由过滤器只能指定路由进行使用

spring cloud gateway. 内置了多种路由过滤器，他们都有GatewayFilter 工厂产生

##### 【1】生命周期

* 在请求到达service()方法之前
* 在service()响应给浏览器之前

##### 【2】种类

* GatewayFilter
* GlobalFilter

##### 【3】自定义全局过滤器

```java
@Component
@Slf4j
public class GatewayGlobalFilter implements GlobalFilter, Ordered {

    /**
     * 这样才能访问成功
     * http://127.0.0.1:9527/payment/paymentNormal?userName=%E7%88%B7
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        log.info("进入到自定义的全局过滤器");
        // 这个是spring 封装的 request，相比HttpServletRequest而言，操作起来更简单
        ServerHttpRequest request = exchange.getRequest();

        MultiValueMap<String, String> queryParams = request.getQueryParams();

        List<String> userName = queryParams.get("userName");

        String userName1 = queryParams.getFirst("userName");

        if (StringUtils.isEmpty(userName1)) {
            String info = "登陆用户名为null，禁止访问";
            log.info(info);
            exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
            // 结束访问
            return exchange.getResponse().setComplete();
        }
        log.info(userName1 + "，登陆成功");

        // 放行
        return chain.filter(exchange);
    }

    /**
     * 设置过滤器的顺序
     * 数字越小，优先级越高
     * */
    @Override
    public int getOrder() {
        return 10;
    }
}
```



##### 【4】使用 gate自带的过滤器

[一般过滤器](https://docs.spring.io/spring-cloud-gateway/docs/3.0.0-M3/reference/html/#gatewayfilter-factories)

```yml
spring:
  application:
    name: cloud-gateway-gateway-9527

  cloud:
    # 配置网关
    gateway:
      # 配置负载均衡
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      # 配置路由
      routes:
        - id: payment_routh_0               # 路由的id，没有固定的规则，但要求唯一，建议配合服务名
          # uri: http://localhost:8001        # 匹配后，提供路由服务的地址 （写死，不好）
          uri: lb://CLOUD-PAYMENT-SERVICE-HYSTRIX # 匹配后，提供路由服务的地址 （根据注册中心解析服务名，再调用provider）
          predicates:
            - Path=/payment/paymentNormal   # 断言，路径相匹配的进行路由
            - After=2020-08-02T15:38:27.281+08:00[Asia/Shanghai] # 在这个日期之后才能匹配。生成日期的方法 ZonedDateTime now = ZonedDateTime.now();
          # - Cookie=username,luojunhua # curl http://localhost:9527/payment/paymentNormal --cookie username="luojunhua"
          # - Header=X-Request-id,\d+   # 请求头中含有X-Request-id，并且值为正整数的正则表达式才放行
          # - Method=get 只放行get请求
          # - Query=userId,\d+  # 请求中带有参数userId，且这个参数的值为正整数
          filters:
            - AddRequestParameter=redKey, blueValue # 参数中有 redKey=blueValue 才能被放行
            # http://127.0.0.1:9527/payment/paymentNormal?userName=随便&redKey=blueValue

```

[全局过滤器](https://docs.spring.io/spring-cloud-gateway/docs/3.0.0-M3/reference/html/#global-filters)

## 六、配置中心

微服务意味着要将单体应用中的业务拆分成一个个子服务每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的

spring cloud 提供了 ConfigServer 来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个的配置文件管理。。。。。

### 1. 是什么

spring cloud config 为微服务框架中的微服务提供集中化的外部配置支持，配置服务器为**各个不同微服务应用**的所有环境提供了一个**中心化的外部配置**

![Snip20200802_5](/Users/luo/Documents/开发笔记/images/Snip20200802_5.png)

### 2. 服务端、客户端

服务端：也称为**分布式配置中心，他是一个独立的微服务应用**，用来连接配置服务器，并为客户端提供获取配置信息，加密、解密信息等访问接口

客户端：通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心，获取和加载配置信息，配置服务器默认采用git来存储配置信息，这样有助于对配置环境进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

### 3. 能干嘛

* 集中管理配置文件
* 不同环境不同配置，动态化的配置更新，分环境部署，比如 dev/test/prod/beta/release
* 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取自己的配置信息
* 当配置发生变更时，服务不再需要重启即可感知到配置的变化，并应用新的配置
* 将配置信息以Rest接口的形式暴露；port，curl访问刷新均可

### 4. 与gitHub整合配置

由于springCloud config默认使用git来存储配置文件（也有其他方式，比如svn和本地文件），但最推荐的还是git，而且使用的是http/https访问形式

### 5. 注册中心配置步骤

#### （1）github上新建仓库

![spring-cloud-config-repository](/Users/luo/Documents/开发笔记/images/spring-cloud-config-repository.png)

#### （2）克隆仓库到本地

https://github.com/fate1007052116/spring-cloud-config.git

```shell
luo@luodeMacBook-Pro spring-cloud-config % pwd
/Volumes/extend/docker_images/spring-cloud-config
luo@luodeMacBook-Pro spring-cloud-config % git clone https://github.com/fate1007052116/spring-cloud-config.git
```



#### （3）添加配置文件并上传

```shell
luo@luodeMacBook-Pro spring-cloud-config % pwd
/Volumes/extend/docker_images/spring-cloud-config/spring-cloud-config

# 新建配置文件
luo@luodeMacBook-Pro spring-cloud-config % vi application-dev.yml

# 查看配置文件
luo@luodeMacBook-Pro spring-cloud-config % tail application-dev.yml
server:
  port: 998

# 本地添加
luo@luodeMacBook-Pro spring-cloud-config % git add ./
luo@luodeMacBook-Pro spring-cloud-config % git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   application-dev.yml

# 本地提交
luo@luodeMacBook-Pro spring-cloud-config % git commit -m "添加了一个修改端口的yml配置文件"
[master (root-commit) 9eab079] 添加了一个修改端口的yml配置文件
 1 file changed, 3 insertions(+)
 create mode 100644 application-dev.yml
luo@luodeMacBook-Pro spring-cloud-config %

# 推送到远程仓库
luo@luodeMacBook-Pro spring-cloud-config % git push https://github.com/fate1007052116/spring-cloud-config.git master
```

#### （4）配置中心的yml配置

```yml
spring:
  application:
    name: cloud-config-center-3344
  cloud:
    config:
      server:
        git:
          uri: git@https://github.com/fate1007052116/spring-cloud-config.git  # gitHub上同步配置信息的仓库
          # 搜索目录
          search-paths:
            - spring-cloud-config  # 好像就是仓库的名字
      # 读取仓库的配置文件所在分支
      label: master
```

#### （5）主启动类添加注解

```java
@SpringBootApplication
@EnableEurekaClient
/**
 * 开启配置中心服务
 * */
@EnableConfigServer
public class CloudConfigConfig3344Application {

	public static void main(String[] args) {
		SpringApplication.run(CloudConfigConfig3344Application.class, args);
	}

}
```

#### （6）修改hosts文件，模拟

```shell
sh-3.2# tail -n 1 /etc/hosts
127.0.0.1 config-center-3344.com
```

#### （7）配置yml

```yml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center-3344
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/luo_jun99/spring-cloud-config.git
          # 建议使用https方式，而不要使用SSH 方式 连接git/gitee，会出现些莫名其妙的错误 git@gitee.com:luo_jun99/spring-cloud-config.git
          # 搜索目录
          search-paths:
            - /  # yml文件所在的文件夹，不是仓库的名字
          username: 13211634008
          password: OROCHi0208
      # 读取仓库的配置文件所在分支
      label: master

eureka:
  client:
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

#### （8）浏览器访问测试

​	**配置读取规则**

* label：分支
* profile：环境
* application：服务名

 【1】/{label}/{application}-{profile}.yml

* master 分支
  * http://localhost:3344/master/application-dev.yml
  * http://localhost:3344/master/application-test.yml
  * http://localhost:3344/master/application-prod.yml

* dev 分支
  * http://localhost:3344/dev/application-dev.yml
  * http://localhost:3344/dev/application-test.yml
  * http://localhost:3344/dev/application-prod.yml
* 其他分支直接改label即可

【2】/{application}-{profile}.yml

在第一种的基础上省略分支的名称 /{label}，默认读取master分支（yml文件中已经配置）

* http://localhost:3344/application-dev.yml

* http://localhost:3344/application-test.yml
* http://localhost:3344/application-prod.yml

【3】/{application}/{profile}[/{label}]

读取json串，自己解析，label（master）可以省略

* http://localhost:3344/application/prod/master

* http://localhost:3344/application/test/

* http://localhost:3344/application/prod/



### 6. bootstrap.yml

#### （1）区分

​	application.yml 是用户级的资源配置选项

​	bootstrap.yml 是系统级的，**优先级更高**

#### （2）概述

spring cloud 会创建一个 "Bootstrap Context" ，作为spring应用的 “Application Context” 的 **父上下文**。初始化的时候，“Bootstrap Context” 负责从**外部源**加载配置属性并解析配置。这两个上下文共享一个从外部获取的“Environment”

“Bootstrap” 属性有高优先级，默认情况下，他们不会被本地配置覆盖。bootstrap context 和 application context 有着不同的约定，所以新增一个 bootstrap.yml 文件，保证 bootstrap context 和 application context 配置的分离

**要将Client模块下的application.yml 文件 改为 bootstrap.yml ，这是关键**

因为 bootstrap.yml 是比application.yml 先加载的。bootstrap.yml 的优先级高于  application.yml

### 7. 客户端配置步骤

#### 将application.yml 改名为bootstrap.yml

```yml
eureka:
  client:
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
# 连接配置中心
spring:
  application:
    name: cloud-config-client-3355
  cloud:
    config:
      label: master # 分支名称
      name: application # 配置文件名称
      profile: dev      # 配置i文件后缀
      uri: http://localhost:3344/   # 配置中心地址
    #  全部综合起来 = http://localhost:3344/master/application-dev.yml
```

### 8. 分布式配置的动态刷新问题

Spring cloud config 配置中心可以同步 GitHub/gitee 的配置

但是客户端重启之前不能同步 spring cloud config 配置中心的内容

### 9. 手动版的动态刷新

#### （1）在client上配置监控

```xml
<!--		添加actuator监控的坐标才能同步分布式的配置刷新-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
```

暴露监控的端点

```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"	# 暴露所有能监控的数据，健康之类的
```

在业务类上添加 **@RefreshScope** 注解

```java
@RestController
@RefreshScope
public class ClientController {

    @Value("${server.port}")
    Integer port;

    @Value("${user.name}")
    String userName;

    @GetMapping("/info")
    public String info() {
        return userName +"，端口是"+ port;
    }

}
```

#### （2）手动发送post请求到client

需要运维工程师在在github/gitee上更改完配置文件之后，手动发送post请求，配置才能更新

```shell
luo@luodeMacBook-Pro spring-cloud-config % curl -X POST http://localhost:998/actuator/refresh
["config.client.version","user.name"]%                                                                     luo@luodeMacBook-Pro spring-cloud-config %
```

## 七、消息总线

为了弥补spring cloud config 的不足，实现**分布式的自动刷新功能**

使用 spring cloud config + spring cloud bus 可以实现配置的动态刷新

### 1. 概述

spring cloud bus 支持两种消息代理：

* RabbitMq
* Kafka

spring cloud bus 是用来将分布式系统的节点与轻量级消息系统连接起来的框架

**它整合了java事件处理机制和消息中间件的功能**

![Snip20200803_1](/Users/luo/Documents/开发笔记/images/Snip20200803_1.png)

### 2. 能干嘛

spring cloud bus 能管理和传播分布式系统间的消息，就像一个分布式容器，可用于广播状态的更改、事件推送等，也可以当作微服务间的通信信道

![Snip20200803_2](/Users/luo/Documents/开发笔记/images/Snip20200803_2.png)

### 3. 为何被称为总线

#### （1）什么是总线

​	在微服务架构的系统中，通常会使用**轻量级的消息代理**来构建一个**共用的消息主题**，并让系统中所有的微服务实例都连接上来。**由于该主题中产生的消息会被所有的实例监听和消费，所以称它为消息总线**。在总线上的各个实例，都可以方便的广播一些需要让其他连接在该主题上的实例都知道的消息。

#### （2）基本原理

Config Client 实例都监听MQ中的同一个topic（默认是 springCloudBus，rabbitmq交换机列表中可以看到）。当一个服务刷新数据的时候，他会把这个信息放入到topic中，这样其他监听同一个topic的服务就能得到通知，然后去更新自身的配置

### 4. 设计思想

* 利用消息总线触发一个**客户端**/bus/refresh，而刷新所有客户端配置

  ![Snip20200803_5](/Users/luo/Documents/开发笔记/images/Snip20200803_5.png)

* 利用消息总线触发一个**服务端**ConfigServer的/bus/refresh端点，而刷新所有客户端的配置

  ![Snip20200803_6](/Users/luo/Documents/开发笔记/images/Snip20200803_6.png)

显然，触发服务端的方式更符合

通过触发客户端来更新配置不适合的原因：

* 打破了微服务职责单一性，因为微服务本身是业务模块，它本不应该承担通知配置刷新的职责
* 破坏了微服务各节点的对等性
* 有一定的局限性。例如微服务在迁移时，他的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改

### 5. 配置步骤

#### （1）配置中心和客户端都需要添加消息总线rabbitMQ的支持

```xml
    <!--        添加消息总线rabbitMQ的支持-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-bus-amqp</artifactId>
            </dependency>

        <!--		添加actuator监控的坐标才能同步分布式的配置刷新-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

#### （2）配置中心添加rabbitMQ的配置

```yml
# 配置rabbitMQ
rabbitmq:
  host: localhost
  port: 5672
  username: luo
  password: a1!
# rabbitMQ相关配置，暴露bus刷新配置的端点
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
            # 暴露bus 刷新配置的端点
```

#### （3）客户端的 bootstrap.yml 添加rabbitMQ信息

```yml
# 连接配置中心
spring:
  application:
    name: cloud-config-client-3355
  cloud:
    config:
      label: master # 分支名称
      name: application # 配置文件名称
      profile: dev      # 配置i文件后缀
      uri: http://localhost:3344/   # 写死配置中心地址
    # uri: http://CLOUD-CONFIG-CENTER-3344:3344/ # 通过注册中心解析配置中心所在地址
    #  全部综合起来 = http://localhost:3344/master/application-dev.yml
  rabbitmq:   # 注意是在 cloud 之下配置
    host: localhost
    port: 5672
    username: luo
    password: a1!


# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
        # 暴露所有能监控的数据，健康之类的
```

### 6. 全局广播通知

![Snip20200803_7](/Users/luo/Documents/开发笔记/images/Snip20200803_7.png)

运维在GitHub/gitee 上修改完之后，对配置中心（3344）发送post请求，就能同步所有客户端



```shell
# 注意是 bus-refresh，没有使用rabbitMQ+ bus的时候，使用的是 refresh
luo@luodeMacBook-Pro ~ %  curl -X POST "http://localhost:3344/actuator/bus-refresh"
```

### 7. 定点通知

指定某一个实例生效，而不是全部生效

/bus/refresh请求不再发送到具体的服务实例上，而是发给config server， 并通过destination参数来指定需要更新配置的服务或实例

```shell
# 公式 curl -X POST "http://localhost:配置中心的端口号/actuator/bus-refresh/{微服务的实例名字:实例端口}"
luo@luodeMacBook-Pro ~ %  curl -X POST "http://localhost:3344/actuator/bus-refresh/cloud-config-client:3355"
```

其中，实例名字是

```yml
# 连接配置中心
spring:
  application:
    name: cloud-config-client # 微服务的实例名字
```

也可以在Eureka注册中心查看实例的名字

![Snip20200803_8](/Users/luo/Documents/开发笔记/images/Snip20200803_8.png)

### 8. 架构总结

![Snip20200803_9](/Users/luo/Documents/开发笔记/images/Snip20200803_9.png)

## 八、消息驱动

![Snip20200803_10](/Users/luo/Documents/开发笔记/images/Snip20200803_10.png)

### 1. 概述

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型

官方定义：spring cloud stream 是一个用于构建与共享消息传递系统连接高度的可伸缩事件驱动微服务框架，该框架提供了一个灵活的编程模型，它建立在已经建立和熟悉的spring 和最佳实践上。

应用程序可以通过inputs 或者 outpus 来与spring cloud stream 中 binder 对象交互。通过我们配置来绑定binding（绑定），而spring cloud stream 的 binder 对象负责与消息中间件交互。所以，我们只需要搞清楚如何与 Spring cloud stream 交互就可以方便使用消息驱动的方式

通过使用Spring Integration 来连接消息代理中间件以实现消息事件驱动

spring cloud stream 是 为一些provider 的消息中间件产品提供了个性化的自动化配置实现，引用了 发布-订阅、消费组、分区的三个核心概念

**目前仅支持 RabbitMQ Kafka**

### 2. 设计思想

#### （1）标准MQ

* 生产者/消费者之间靠**消息（Message）**媒介传递消息内容

* 消息必须走特定的**消息通道（Message Channel）**

* 消息通道里的消息如何被消费呢？谁负责**收发处理**
	
	消息通道message Chanel 和子接口Subscribable Channel ，由MessageHandler消息处理器所订阅

#### （2）引入 cloud stream 之后

这些中间件的差异性导致，我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我们想往另外一种消息队列进行迁移，这时无疑就是一个灾难，**一大堆东西都要重新推到，重新做**，因为它跟我们的系统耦合了，这时候，Spring cloud stream 给我们提供了一种解耦的方式

![Snip20200803_3](/Users/luo/Documents/开发笔记/images/Snip20200803_3.png)

#### （3）为什么可以屏蔽底层差异

在没有绑定器这个概念的情况下，我们的spring boot 应用要直接与消息中间件进行消息交互的时候，由于各种消息中间件构建的初衷不同，他们的实现细节上会有较大的差异性。通过定义绑定器作为中间层，完美地实现了**应用程序与消息中间件细节之间的隔离**。通过向应用程序暴露统一的Channel通道，使得应用程序不再需要考虑各种不同的消息中间件的实现。

**通过定义绑定器Binder作为中间层，实现了应用程序与消息中间件细节之间的隔离**

#### （4）Binder

在没有绑定器这个概念的情况下，我们的Spring boot 应用要直接与消息中间件进行信息交互的时候，由于各种消息中间件构建的初衷不同，他们的实现细节上会有较大的差异性，通过定义绑定器作为中间层，完美的实现了**应用程序与消息中间件的隔离**。stream 对消息中间件的进一步封装，可以做到代码层面对中间件的无感知，甚至于动态的切换中间件（rabbitMQ切换为kafka），使得微服务开发的高度解耦，服务可以关注更多自己的业务流程

![Snip20200803_4](/Users/luo/Documents/开发笔记/images/Snip20200803_4.png)

#### （5）发布-订阅

Stream 的消息通信方式遵循了发布-订阅模式

Topic 主题进行广播

* 在RabbitMQ中就是Topic Exchange
* 在Kafaka中就是Topic

### 3. stream 标准流程套路

![Snip20200803_11](/Users/luo/Documents/开发笔记/images/Snip20200803_11.png)

#### （1）Binder

很方便的连接中间件，屏蔽差异

#### （2）Channel

通道，是队列Queue中的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置

#### （3）Source 和 Sink

简单的可以理解为参照对象是Spring cloud stream 自身，从stream 发布消息就是输出，接受消息就是输入

### 4. 常用注解

| 组成/注解       | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前仅支持RabbitMQ、Kafaka                           |
| Binder          | Binder 是应用与消息中间件之间的封装，目前实现了RabbitMQ和Kafaka的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息的类型（对应于Kafaka的topic，RabbitMQ的exchange），这些都可以通过配置文件来实现 |
| @Input          | 标识输入通道，通过该输入通道接受消息，并进入应用程序         |
| @Output         | 标识输出通道，发布的消息将通过该通道离开应用程序，前往消息中间件 |
| @StreamListener | 监听队列，用于消费者队列的消息接收                           |
| @EnableBinding  | 将信道和exchange绑定在一起                                   |

### 5. provider 配置

```yml
server:
  port: 8801
spring:
  application:
    name: cloud-provider-stream
  cloud:
    stream:
      binders:                  # 在这里配置需要绑定的rabbitmq的服务信息
        defaultRabbit:          # 表示定一个的名称，用于后面的bindings 整合
          type: rabbit          # 表示组件类型
          environment:          # 设置rabbitmq的相关环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: luo
                password: a1!
      bindings:                 # 服务的整合处理
        output:                 # 这个名字是一个通用的名称
          destination: studyExchange        # 表示要用使用Exchange名称定义
          content-type: application/json    # 设置消息类型，本次为json，文本则为 text/plain
          binder: defaultRabbit             # 设置要绑定的消息服务的具体设置


eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2      # 设置心跳的间隔时间（默认30秒）
    lease-expiration-duration-in-seconds: 5   # 设置过期时间
    instance-id: send-8801.com                # 在消息列表中显示的主机名称
    prefer-ip-address: true                   # 访问路径变为ip地址
```

```java
/**
 * 实现类不需要添加@Service注解
 * @EnableBinding(Source.class) 定义消息的推送管道
 * */
@EnableBinding(Source.class)
@Slf4j
public class MyMessageProviderImpl implements MyMessageProvider {

    /**
     * 消息的发送管道
     * */
    @Autowired
    MessageChannel output;


    @Override
    public String send(String msg) {

        Message<String> build = MessageBuilder.withPayload(msg).build();

        this.output.send(build);

        log.info(msg+"，已发送");

        return null;
    }
}
```

### 6. consumer 配置

```yml
server:
  port: 8802

spring:
  application:
    name: cloud-consumer-stream
  cloud:
    stream:
      binders:                  # 在这里配置需要绑定的rabbitmq的服务信息
        defaultRabbit:          # 表示定一个的名称，用于后面的bindings 整合
          type: rabbit          # 表示组件类型
          environment:          # 设置rabbitmq的相关环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: luo
                password: a1!
      bindings:                 # 服务的整合处理
        input:                  # 这个名字是通道的名称，这里是唯一与provider有区别的地方，provider 这里是output
          destination: studyExchange #表示要使用的交换机
          content-type: application/json
          binder: defaultRabbit


eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2      # 设置心跳的间隔时间（默认30秒）
    lease-expiration-duration-in-seconds: 5   # 设置过期时间
    instance-id: receive-${server.port}.com                # 在消息列表中显示的主机名称
    prefer-ip-address: true                   # 访问路径变为ip地址
```

```java
@Service
@EnableBinding(Sink.class)
@Slf4j
public class MessageConsumerServiceImpl {

    @Value("${server.port}")
    Integer port;

    @StreamListener(Sink.INPUT)
    public void receiveMessage(Message<String> message) {
        log.info("我是消费者：" + port + "接收到的消息是：" + message);
        log.info(message.getPayload());
    }

}
```

### 7. 重复消费

目前，只要provider 发送消息，所有的consumer都能收到该消息，存在重复消费

#### （1）实际案例

比如在一下场景中，订单系统我们做集群部署，都会从RabbitMQ中获取到订单信息，那如果一个订单同时被两个服务获取到，那么就会造成数据错误，我们应该避免这种情况。

这时，**我们可以通过使用Stream中的消息分组来解决**

![Snip20200803_12](/Users/luo/Documents/开发笔记/images/Snip20200803_12.png)

注意在Stream中处于同一个group中的多个消费者是**竞争关系**，就能保证消息只会被其中一个应用消费一次

**不同组是可以全面消费的（重复消费）**

**同一组内会发生竞争关系，只有其中一个可以消费**

这里的组对应的就是RabbitMQ中的Queue队列，同一个队列中的所有消费者是竞争关系

下图为，一个交换机绑定两个队列，每个队列对应一个consumer，所以会有重复消费

![Snip20200803_13](/Users/luo/Documents/开发笔记/images/Snip20200803_13.png)



#### （2）分组原理

微服务的应用放置于同一group中，就可以保证消息只会被其中一个应用消费一次。**不同组是可以同时消费多次，同一个组内会发生竞争关系，只有其中一个可以消费**

#### （3）分组、持久化

```yml
spring:
  application:
    name: cloud-consumer-stream
  cloud:
    stream:
      binders:                  # 在这里配置需要绑定的rabbitmq的服务信息
        defaultRabbit:          # 表示定一个的名称，用于后面的bindings 整合
          type: rabbit          # 表示组件类型
          environment:          # 设置rabbitmq的相关环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: luo
                password: a1!
      bindings:                 # 服务的整合处理
        input:                  # 这个名字是通道的名称，这里是唯一与provider有区别的地方，provider 这里是output
          destination: studyExchange #表示要使用的交换机
          content-type: application/json
          binder: defaultRabbit
          # 微服务的应用放置于同一group中，就可以保证消息只会被其中一个应用消费一次。不同组是可以同时消费多次，同一个组内会发生竞争关系，只有其中一个可以消费
          # 不写组名，默认使用随机匿名的组名
          # 组名就是RabbitMQ中对应的Queue队列名
        # group: group-${server.port} # 演示不同组的重复消费
          group: publicGroup          # 演示同一个组，避免重复消费
          # 指定了分组之后，该分组对应的Queue就转化为Durable 持久化队列，
          # 当没有消费者连接到该Queue时，该Queue接收到的消息会被保存，当消费者连接到Queue时，队列保存的消息将继续发送给消费者
          # 如果没有指定分组，则该分组的数据不会在Queue中进行持久化
```

可见，指定了group之后，RabbitMQ中组名对应的Queue有了D属性（D durable 持久化）

![Snip20200803_14](/Users/luo/Documents/开发笔记/images/Snip20200803_14.png)

## 九、分布式请求链路跟踪

### 1. 概述

#### （1） 解决的问题

在微服务框架中，一个客户端发起的请求在后端系统中会经过多个不同的服务节点调用来协同产生最后的请求结果，每一个前段请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延迟时或错误都会引起整个请求最后的失败

#### （2）是什么

Spring cloud sleuth 提供了一套完整的服务跟踪解决方案，在分布式系统中提供追踪解决方案并且兼容支持了zipkin

### 2. zipkin安装

```shell
# 教程 https://zipkin.io/pages/quickstart.html
# docker 安装
docker run -d -p 9411:9411 openzipkin/zipkin

# 下载	
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
```

### 3. 完整的调用链路

表示一请求链路，一条链路通过Trace id 唯一标识，span标识发起的请求信息，各span通过parent id 关联起来

trace id：类似于树结构的span集合，表示一条调用链路，存在唯一标识

span：表示调用链路的来源，通俗的理解是span就是一次请求信息

![Snip20200803_15](/Users/luo/Documents/开发笔记/images/Snip20200803_15.png)

简图

![Snip20200803_16](/Users/luo/Documents/开发笔记/images/Snip20200803_16.png)

### 4. 配置

除了注册中心，要进行链路监控的微服务都要进行此配置

添加依赖

```xml
<!--        同时引入 spring-cloud-sleuth-zipkin, spring-cloud-starter-sleuth 用于监控-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

修改yml

```yml
spring:
  application:
    # 微服务名称
    name: cloud-payment-service
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      # 采样率介于 0 - 1 之间 ，1 表示 全部采集，一般 0.5 就够了
      probability: 1
```

![Snip20200803_17](/Users/luo/Documents/开发笔记/images/Snip20200803_17.png)

