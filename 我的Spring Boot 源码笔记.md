# 我的Spring Boot 源码笔记

## 1. @SpringBootApplication 下的自动配置注解
@EnableAutoConfiguration

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration
```

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered{
     
  /**
	 * Return the {@link AutoConfigurationEntry} based on the {@link AnnotationMetadata}
	 * of the importing {@link Configuration @Configuration} class.
	 * @param annotationMetadata the annotation metadata of the configuration class
	 * @return the auto-configurations that should be imported
	 * 它的这个方法用于，将所有需要导入的组件以全限定路径的方式返回，这些组件就会被添加到spring IOC 容器中。
   * 这些全限定路径是很多的自动配置类，就是给容器中导入这个场景需要的所有组件，并配置好这些组件
	 */
	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 这个方法用于获取自动配置类
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
  
  /**
	 * Return the auto-configuration class names that should be considered. By default
	 * this method will load candidates using {@link SpringFactoriesLoader} with
	 * {@link #getSpringFactoriesLoaderFactoryClass()}.
	 * @param metadata the source metadata
	 * @param attributes the {@link #getAttributes(AnnotationMetadata) annotation
	 * attributes}
	 * @return a list of candidate configurations
	 */
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    // 继续往下调用
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
    // 上面方法完成之后，就能获取到EnableAutoConfiguration.class(类名)对应的值，接下来就可以把他们添加到IOC容器中
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
      
}

```

```java
public final class SpringFactoriesLoader {

	/**
	 * 通过这个类获取自动配置类的名字列表
	 * 列表放在META-INF/spring.factories中
	 * Load the fully qualified class names of factory implementations of the
	 * given type from {@value #FACTORIES_RESOURCE_LOCATION}, using the given
	 * class loader.
	 * @param factoryType the interface or abstract class representing the factory
	 * @param classLoader the ClassLoader to use for loading resources; can be
	 * {@code null} to use the default
	 * @throws IllegalArgumentException if an error occurs while loading factory names
	 * @see #loadFactories
	 */
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    // factoryTypeName 实际上就是类名EnableAutoConfiguration.class
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}
  
  private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
      // 配置文件位置在这里，扫描所有jar包，类路径下的这个文件
      // FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
        // 把扫描到的这些文件的内容包装成Properties对象
        // 
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
}
```



* 列表放在META-INF/spring.factories中
* 准确位置是在 /spring-boot-autoconfigure/2.3.0.RELEASE/spring-boot-autoconfigure-2.3.0.RELEASE.jar!/META-INF/spring.factories 
* 就将所有类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration 的值添加到了IOC容器中

![image-20200714223217079](/Users/luo/Library/Application Support/typora-user-images/image-20200714223217079.png)

```properties
# 每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration

```



### 上面找到了自动配置类的全限定路径，接下来通过自动配置类的全限定路径调用对应的自动配置类，这些全都是自动配置类

  ![image-20200714131406676](/Users/luo/Library/Application Support/typora-user-images/image-20200714131406676.png)

### 以 org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration 为例解释自动配置

```java

@Configuration(proxyBeanMethods = false)// 表明这是一个配置类，和我们以前编写的配置类一样，也可以给容器添加组件
@EnableConfigurationProperties(ServerProperties.class)//启用指定类的（这里是ServerProperties类）～ConfigurationProperties功能(@ConfigurationProperties注解功能)；将配置类中对应的值和ServerProperties绑定起来；并把ServerProperties对象加入到IOC容器中
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) //判断当前应用是不是一个web应用，只有当是一个web应用时，整个配置类里面的配置才会生效。spring framework底层@Conditional 注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效
@ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有这个类；CharacterEncodingFilter是springMVC中乱码解决的拦截过滤器
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)	//判断配置文件中是否存在某个配置(server.servlet.encoding)。matchIfMissing = true 代表如果不存在，也代表判断是成立的。也就是说，即使配置文件中没有配置server.servlet.encoding.enabled=true，也会默认生效
public class HttpEncodingAutoConfiguration {
  
  private final Encoding properties;//已经和springBoot的配置文件映射

  // 只有一个有参构造器的情况下，会从IOC容器中获取对象，传入构造器
  // 而参数 ServerProperties 在这个类的第二个注解@EnableConfigurationProperties(ServerProperties.class) 中已经创建，可直接从IOC容器中获取。也就是说，通过只有一个有参构造器和属性的情况下，可以省略，这个属性的@AutoWired注解
  public HttpEncodingAutoConfiguration(ServerProperties properties) {
		this.properties = properties.getServlet().getEncoding();
	}
  
  
  @Bean //给容器中添加一个组件
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());//这个组件中的某些值需要从properties中获取
		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
		return filter;
	}
}
```

### 所有在配置文件中能配置的属性都是在xxxProperties类中封装着，配置文件能配置什么就参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true) //从配置文件(*.yml,*.properties)中获取指定的值和bean的属性进行绑定，我们第一章就用过这个注解
public class ServerProperties {
  
}
```

根据当前不同条件的判断，决定这个配置类是否生效，当条件都满足时，自动配置类才生效

一旦这个配置类生效；这个配置类就会给IOC容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，而这些类的每一个属性又是和配置文件绑定的

## 2. SpringBoot的精髓

1. spring Boot 在启动的时候会加载大量的自动配置类

2. 我们看我们需要的功能有没有springBoot默认写好的自动配置类

3. 如果有，我们再来看自动配置类中到底配置了哪些组件（只要我们要用的组件有，就不需要再来配置了）

4. 如果没有，就需要我们手写一个配置类

5. 给容器中自动配置类添加组件的时候，会从properties类获取某些属性。我们就可以在配置文件中指定这些属性的值

* xxxAutoConfiguration ：自动配置类
* 给容器中添加组件
* 也会有对应的xxxProperties 类封装配置文件中相关的属性

## 细节

1. @Conditional 派生注解（Spring注解版原生的@Conditional作用）

   必须是@Conditional 指定的条件成立，才给容器中添加组件，配置类里面的所有内容才会生效

### 判断哪些自动配置类生效

```properties
debug=true # 开启SpringBoot的debug模式，可以查看哪些配置类生效
```

```shell

============================
CONDITIONS EVALUATION REPORT
============================


Positive matches:	# 已经启用的自动配置类
-----------------

   AopAutoConfiguration matched:
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

   AopAutoConfiguration.ClassProxyingConfiguration matched:
      - @ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice' (OnClassCondition)
      - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)


Negative matches: # 没有启用的自动配置类
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

   AopAutoConfiguration.AspectJAutoProxyingConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice' (OnClassCondition)

```

## 3. springBoot的启动

### （1）创建SpringBootApplication 对象

```java
public class SpringApplication {
  
  
  public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args){
		// 1.创建springBootApplication对象
    // 2.运行SpringBootApplication对象
    return new SpringApplication(primarySources).run(args);
	}
  
  public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}
  
// 最终会调用这个构造器来创建SpringBootApplication对象  
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
  	// 保存主配置类，方便以后调用。主配置类，就是@SpringBootApplication注解修饰的springboot的启动类
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
  	// 判断当前应用是不是一个web应用（如果有servlet.class之类的就判断为是一个web应用）
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		
  	// 所以最终就是：从类路径下的META-INF/spring.factories中找到所有的ApplicationContextInitializer；然后保存起来
  	setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
  // 和setInitializers()方法类似，从类路径下的META-INF/spring.factories中找到所有的ApplicationListener；然后保存起来
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
  	// 从多个配置类中，找到有main方法的类当作主配置类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
  
  
  	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
      // SpringFactoriesLoader.loadFactoryNames() 跳转
    
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
  
}  
```

```java
public final class SpringFactoriesLoader {	
  
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
    // 调用 loadSpringFactories()
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}
  
  private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
      // FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			// 省略
	}
}
```

加载的部分ApplicationContextInitializer

```properties
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

```

加载的全部ApplicationContextInitializer

![springBoot启动时加载的initializers](/Users/luo/Documents/开发笔记/images/springBoot启动时加载的initializers.png)

![Snip20200720_1](/Users/luo/Documents/开发笔记/images/Snip20200720_1.png)

### （2）运行SpringBootApplication

```java
public class SpringApplication {


public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();	//配置java awt 图形相关
  // 从"META-INF/spring.factories"下获取SpringApplicationRunListeners
		SpringApplicationRunListeners listeners = getRunListeners(args);
  // 回调所有 SpringApplicationRunListeners 的 starting() 方法
		listeners.starting();
		try {
      // 封装命令行参数，springboot启动类的args 参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
      // 准备环境
      /*
      		环境创建完成之后，回调所有 SpringApplicationRunListeners 的 environmentPrepared()方法，表示环境准备完成
      */
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
      
			configureIgnoreBeanInfo(environment);
      // 在控制台打印springBoot的图标
			Banner printedBanner = printBanner(environment);
      /**
       * 创建IOC容器
       * 根据应用类型创建web的IOC容器还是普通的IOC容器
       * 判断之后，用BeanUtils通过反射创建IOC容器
       */
			context = createApplicationContext();
      // 做异常分析报告
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
      // 准备上下文，传入IOC,环境，监听器，等
      /*
      *  1.将环境保存到IOC容器中
      *  2.applyInitializers() 回调之前保存的所有的initializer的 initialize()方法
      *  3.调用所有之前加载的 SpringApplicationRunListener 对象的 contextPrepared()方法
      *  3.注册命令行参数到IOC容器中
      *  4.调用所有之前加载的 SpringApplicationRunListener 对象的 contextLoaded()方法
      */
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			// 刷新IOC容器；IOC容器初始化：加载IOC中的所有组件@Bean @Component 等等，创建对象
      // 如果是Web应用，还会创建嵌入式的tomcat
      // 配置类，组件，自动配置类
      refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
  // springboot启动之后，返回IOC容器
		return context;
	}
  
  
  	// 获取SpringApplicationRunListeners
  	private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
	}
  
  
  	// 准备环境
  	private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		ConfigurationPropertySources.attach(environment);
      // 回调 所有 SpringApplicationRunListeners 的 environmentPrepared()方法
		listeners.environmentPrepared(environment);
		bindToSpringApplication(environment);
      // 判断是不是用户自定义的环境，如果是自定义环境，则转为自定义环境
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
					deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
}
```

## 4. 自定义场景启动器starter

### （1）如何编写自动配置

```java
@Configuration // 指定这个类是一个配置类
@ConditionalXXX // 在指定条件成立的情况下，这个自动配置类生效
@AutoConfigureAfter // 指定自动配置类的顺序

@Bean // 给IOC容器中添加组件
@ConfigurationProperties // 结合相关的 xxxProperties类来绑定相关的配置
@EnableConfigurationProperties // 让xxxProperties类生效，并且加入到IOC容器中

// 要实现：starter启动的时候，不需要做任何配置，配置类就已经配置好了
// 将需要启动就自动加载的配置类（自动配置类），在/META-INF/spring.factories 中进行配置
// 示例
```

```properties
# Auto Configure
# \ 代表换行 
# , 为数组
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration
```

### （2）设计模式

自定义启动器只做依赖导入，只存放/META-INF/下的文件

专门写一个自动配置模块

启动器依赖自动配置；这样别人只需要引入自定义启动器（starter）

自定义启动器的命名规范:  **mybatis-spring-boot-starter**

### 5. 实现接口，Spring初始化完成时会自动执行 afterPropertiesSet（）

```java
@Component
public class OssConfig implements InitializingBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        
    }
}
```

