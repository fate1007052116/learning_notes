# Spring Boot 日志

spring framework 默认使用 commons-logging (spring framework 5.0 没有依赖这个日志包)

spring boot 默认使用slf4j ，但是又依赖于Spring framework，所以Spring boot 在导入Spring framework的时候排除了commons-logging的依赖

| 日志门面                       | 描述                    |
| ------------------------------ | ----------------------- |
| JCL(Jakarta Commons Logging)   | 最后一次更新于2014年    |
| SLF4J(simple logging for java) | spring boot 默认日志api |
| jboss-logging                  | 不是给一般的程序员用    |

| 日志实现 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| log4j    | 较早开发，没用使用公共的API日志抽象，需要通过适配器类来使用log4j |
| JUL      | Java 临时开发的日志工具类，不好用                            |
| log4j2   | apache开发的log4j升级版，加入很多重大功能，但是由于太新，缺乏大部分框架的支持 |
| logback  | slf4j就是专门针对logback而创建的，完美兼容                   |

### 1. spring boot 用一个启动器来专门记录日志



![日志依赖](/Users/luo/Documents/开发笔记/MySpringbootImg/日志依赖.png)

Logback.xml 直接就被日志框架识别了

Logback-spring.xml 日志框架就不直接加载日志的配置项，由SpringBoot

```xml
<layout class="ch.qos.logback.classic.PatternLayout">
            <!--            当从logback.xml更改为logback-spring.xml之后，可以解锁高级功能
                        日志记录会根据当前的环境（生产环境、开发环境、测试环境）来输出相应的格式
                        但是如果切换为logback.xml 则不能解析<springProfile>标签，且报错
            -->
            <springProfile name="dev">
                <!--                dev环境下执行这个
                           开发环境下我不关注日期，就不显示日期
                -->
                <pattern>[%thread] %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <!--                非dev环境下执行这个-->
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
```





| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

### 2. 实验：将默认的logback实现替换为log4j实现

​	本身没有什么意义，因为sl4j就是log4j的升级版

```xml
<!--pom配置-->
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <!--				排除log4j到slf4j 的适配层
                （原本调用log4j完成记录日志的，有了这个适配层且排除log4j之后，会调用这个适配层，适配层再调用slf4j api）
                狸猫（sl4j）换太子（log4j）
                -->
                <exclusion>
                    <artifactId>log4j-to-slf4j</artifactId>
                    <groupId>org.apache.logging.log4j</groupId>
                </exclusion>
                <!--				排除logback实现类-->
                <exclusion>
                    <artifactId>logback-classic</artifactId>
                    <groupId>ch.qos.logback</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- https://mvnrepository.com/artifact/log4j/log4j -->
        <!--<dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>-->

        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
        <dependency>
            <!--			添加slf4j到log4j的适配层
                sl4j调用log4j完成日志记录
            -->
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
        </dependency>
```

使用log4j 替换logback之后的依赖图

![将logback替换为log4j](/Users/luo/Documents/开发笔记/images/将logback替换为log4j.png)

### 3. 实验：将默认的sl4j+logback 替换为 log4j2

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-logging</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
```



![logback替换为log4j2](/Users/luo/Documents/开发笔记/images/logback替换为log4j2.png)