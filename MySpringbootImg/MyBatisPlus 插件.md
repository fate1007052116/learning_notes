# MyBatisPlus 插件

## 1. 临时配置法

### （1） MybatisAutoConfiguration自动配置类

```java
@org.springframework.context.annotation.Configuration
@ConditionalOnClass({ SqlSessionFactory.class, SqlSessionFactoryBean.class })
@ConditionalOnSingleCandidate(DataSource.class)
@EnableConfigurationProperties(MybatisProperties.class)
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration implements InitializingBean {  
	@Bean
  @ConditionalOnMissingBean // 可以在IOC中放入SqlSessionFactory对象，自己进行配置
  public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
  }
}
```

### （2）最简单的spring使用mybatisPlus插件的方式

不推荐，像半结合产品

```java
// MybatisSqlSessionFactoryBuilder 重写了 SqlSessionFactoryBuilder 的部分方法
public class MybatisSqlSessionFactoryBuilder extends SqlSessionFactoryBuilder {
}
```



```java
@Configuration
public class MybatisPlusConfiguration {


    @Bean
    public SqlSessionFactory MybatisSqlSessionFactory(){
        InputStream in = null;
        try {
             in = Resources.getResourceAsStream("myBatis-config.xml");
        } catch (IOException e) {
            e.printStackTrace();
        }
        MybatisSqlSessionFactoryBuilder builder = new MybatisSqlSessionFactoryBuilder();

        return builder.build(in);
    }

}
```

### （3） 配置了之后的日志

已经使用了自定义的 MybatisSqlSessionFactory

```
MybatisAutoConfiguration#sqlSessionFactory:
      Did not match: 
         - @ConditionalOnMissingBean (types: org.apache.ibatis.session.SqlSessionFactory; SearchStrategy: all) found beans of type 'org.apache.ibatis.session.SqlSessionFactory' MybatisSqlSessionFactoryBuilder (OnBeanCondition)

```

## 2. springBoot整合mybatis plus

mybatisPlus插件 的 MybatisSqlSessionFactoryBean 实现了和 mybatis 的 SqlSessionFactoryBean 类一样的接口

```java
public class MybatisSqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
}
```

```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {
}
```



```java
@Configuration
public class MybatisPlusConfiguration {

    @Bean
    public SqlSessionFactory myBatisSqlSessionFactoryBean(DataSource dataSource){
				//
        MybatisSqlSessionFactoryBean bean = new MybatisSqlSessionFactoryBean();

        bean.setDataSource(dataSource);
      
        SqlSessionFactory sqlSessionFactory = null;
        try {
            sqlSessionFactory= bean.getObject();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return sqlSessionFactory;
    }

}
```

```shell
# mybatis 自动配置的 sqlSessionFactory 被我们自定义的 sqlSessionFactory所替换
MybatisAutoConfiguration#sqlSessionFactory:
      Did not match:
         - @ConditionalOnMissingBean (types: org.apache.ibatis.session.SqlSessionFactory; SearchStrategy: all) found beans of type 'org.apache.ibatis.session.SqlSessionFactory' myBatisSqlSessionFactoryBean (OnBeanCondition)

```

## 3. SQL注入的原理

前面我们已经知道，MP在启动后会将BaseMapper中的一系列的方法注册到meppedStatements中，那么究竟是如

何注入的呢？流程又是怎么样的？下面我们将一起来分析下。

在MP中，ISqlInjector负责SQL的注入工作，它是一个接口，AbstractSqlInjector是它的实现类，实现关系如下：

```java
/**
 * SQL 自动注入器接口
 *
 * @author hubin
 * @since 2016-07-24
 */
public interface ISqlInjector {

    /**
     * 检查SQL是否注入(已经注入过不再注入)
     *
     * @param builderAssistant mapper 信息
     * @param mapperClass      mapper 接口的 class 对象
     */
    void inspectInject(MapperBuilderAssistant builderAssistant, Class<?> mapperClass);
}
```

```java
public abstract class AbstractSqlInjector implements ISqlInjector {
/**
 * SQL 自动注入器
 *
 * @author hubin
 * @since 2018-04-07
 */
    @Override
    public void inspectInject(MapperBuilderAssistant builderAssistant, Class<?> mapperClass) {
        Class<?> modelClass = extractModelClass(mapperClass);
        if (modelClass != null) {
            String className = mapperClass.toString();
            Set<String> mapperRegistryCache = GlobalConfigUtils.getMapperRegistryCache(builderAssistant.getConfiguration());
          // 检查是否已经注入
            if (!mapperRegistryCache.contains(className)) {
                List<AbstractMethod> methodList = this.getMethodList(mapperClass);
                if (CollectionUtils.isNotEmpty(methodList)) {
                    TableInfo tableInfo = TableInfoHelper.initTableInfo(builderAssistant, modelClass);
                    // 没有注入的时候，循环注入自定义方法
                    methodList.forEach(m -> m.inject(builderAssistant, mapperClass, modelClass, tableInfo));
                } else {
                    logger.debug(mapperClass.toString() + ", No effective injection method was found.");
                }
                mapperRegistryCache.add(className);
            }
        }
    }
}
```

```java
/**
 * 抽象的注入方法类
 *
 * @author hubin
 * @since 2018-04-06
 */
public abstract class AbstractMethod implements Constants {

	    /**
     * 注入自定义方法
     */
    public void inject(MapperBuilderAssistant builderAssistant, Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        this.configuration = builderAssistant.getConfiguration();
        this.builderAssistant = builderAssistant;
        this.languageDriver = configuration.getDefaultScriptingLanguageInstance();
        /* 注入自定义方法 */
        injectMappedStatement(mapperClass, modelClass, tableInfo);
    }
  
    /** 到这里，查看哪些类实现了这个方法
     * 注入自定义 MappedStatement
     *
     * @param mapperClass mapper 接口
     * @param modelClass  mapper 泛型
     * @param tableInfo   数据库表反射信息
     * @return MappedStatement
     */
    public abstract MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo);

}
```

**查看哪些类实现了 抽象类AbstractMethod 的 injectMappedStatement() 方法**

![Snip20200725_1](/Users/luo/Documents/开发笔记/images/Snip20200725_1.png)

**使用SelectById类进行举例**

```java
public class SelectById extends AbstractMethod {

    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
        SqlMethod sqlMethod = SqlMethod.SELECT_BY_ID;
      // 生成SQL语句
        SqlSource sqlSource = new RawSqlSource(configuration, String.format(sqlMethod.getSql(),
            sqlSelectColumns(tableInfo, false),
            tableInfo.getTableName(), tableInfo.getKeyColumn(), tableInfo.getKeyProperty(),
            tableInfo.getLogicDeleteSql(true, true)), Object.class);
      // 将SQL语句加入到容器中，以后直接执行
        return this.addSelectMappedStatementForTable(mapperClass, getMethod(sqlMethod), sqlSource, tableInfo);
    }
}
```

![Snip20200725_2](/Users/luo/Documents/开发笔记/images/Snip20200725_2.png)

## 4. 操作Oracle

### （1）创建表

```sql
-- Oracle 是大小写敏感的，我们创自己写Sql脚本创建表的时候Oracle会自动将我们的表名，字段名转成大写,但是 Oracle 同样支持"" 语法，将表名或字段名加上""后，Oracle不会将其转换成大写

-- 建议创建的时候不要添加双引号，让oracle自动全部转大写
create table tb_user(
	user_id NUMBER(20) VISIBLE not null,
	user_name varchar2(255 BYTE) VISIBLE,
	true_name varchar2(255 BYTE) VISIBLE,
	password varchar2(255 BYTE) VISIBLE,
	email varchar2(255 BYTE) VISIBLE
)

alter table tb_user
add (age NUMBER(2))

-- 创建序列
create sequence seq_tb_user start with 1 increment by 1

```

### （2）安装jdbc驱动包

1. 因为oracle驱动在maven仓库上没有，所以手动安装到本地仓库

```shell
# 切换到maven目录
luo@luodeMacBook-Pro bin % cd /Volumes/OS/javaEE/maven/apache-maven-3.6.3/bin

# 安装ojdbc10，用不了，建议使用ojdbc8.jar
luo@luodeMacBook-Pro bin % ./mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc10 -Dversion=10.0.1 -Dpackaging=jar -Dfile=/Volumes/OS/javaEE/jar/ojdbc10.jar

# 我只是用ojdbc8.jar 替换了maven 本地仓库中的ojdbc10.jar 所以懒得改
```

2. 导入刚安装的ojdbc坐标

   ```shell
   		<dependency>
   			<groupId>com.oracle</groupId>
   			<artifactId>ojdbc10</artifactId>
   			<version>10.0.1</version>
   		</dependency>
   ```

3. 遇到的错误

   ```shell
   ORACLE数据库---"ORA-00942: 表或视图不存在
   ```

   * Oracle 是大小写敏感的，我们创自己写Sql脚本创建表的时候Oracle会自动将我们的表名，字段名转成大写,

     ```sql
     create table T_WindRadar  (
        wr_id                VARCHAR2(64)                    not null,
        wr_reciveTime        DATE,
        wr_image             BLOB,
        constraint PK_T_WINDRADAR primary key (wr_id)
     );
     ```

     

   * 但是 Oracle 同样支持"" 语法，将表名或字段名加上""后，Oracle不会将其转换成大写

     ```sql
     create table "T_WindRadar"  (
        "wr_id"                VARCHAR2(64)                    not null,
        "wr_reciveTime"        DATE,
        "wr_image "            BLOB,
        constraint PK_T_WINDRADAR primary key (wr_id)
     );
     ```

     

   * 如果加上了"",那么我们采用一般的SQL语句查询则会产生“ORA-00942: 表或视图不存在 ”，因此SQL脚本中需要将表名也加上""。

     ```sql
     select * from  "T_WindRadar";
     ```

     

## 5. 配置性能分析插件

### （1）导入性能分析插件

```xml
        <!--sql 分析打印插件-->
        <dependency>
            <groupId>p6spy</groupId>
            <artifactId>p6spy</artifactId>
            <version>3.8.2</version>
        </dependency>
```

### （2）配置数据源

```properties
spring:
  datasource:
  # 改一
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver 
    password: orochi0208
    username: root
    # 改二
    url: jdbc:p6spy:mysql://127.0.0.1:3306/home?useUnicode=true&characterEncoding=utf-8
```

### （3）resources 下放入 spy.properties 文件

```properties

module.log=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger


#日志输出到控制台，解开注释就行了
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger

# 指定输出文件位置
logfile=sql.log

# 使用日志系统记录 sql
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,batch,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒

```

