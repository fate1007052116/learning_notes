Spring Boot Web



```java
public class WebMvcAutoConfiguration {
  
  	// 他个构造器同时从IOC容器中获取两个配置属性ResourceProperties、WebMvcProperties
  	public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
		}
}
```

### 1. SpringBoot 对静态资源映射的规则

WebMvcAutoConfiguration类下的静态内部类 static class WebMvcAutoConfigurationAdapter 的一个方法

```java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
      // 缓存时间
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
      // 所有/webjars/** 的映射请求都去classpath:/META-INF/resources/webjars/ 下查找
			if (!registry.hasMappingForPattern("/webjars/**")) { 
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
      //staticPathPattern 静态资源路径是："/**"
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
        // 添加资源映射this.resourceProperties.getStaticLocations() 的结果是"classpath:/META-INF/resources/","classpath:/resources/", "classpath:/static/", "classpath:/public/"
customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
      
      
   
}
```

可以设置和静态资源有关的配置、缓存时间等

```java

@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
```



webjars：以jar包的方式引入web的静态资源

例如：

```xml
		
<!-- https://mvnrepository.com/artifact/org.webjars.bower/jquery 
			pom文件中引入jquery依赖
-->
		<dependency>
			<groupId>org.webjars.bower</groupId>
			<artifactId>jquery</artifactId>
			<version>1.9.1</version>
		</dependency>
```



![maven引入jquery](/Users/luo/Documents/开发笔记/images/maven引入jquery.png)

这样就可以使用http://localhost:8080/webjars/jquery/1.9.1/jquery.js 访问到静态资源文件了

### 2. /**表示当前项目的任何资源，静态资源的文件夹

```
"classpath:/META-INF/resources/",
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/"
```

### 3. 配置欢迎页映射

WebMvcAutoConfiguration类下的静态内部类 static class EnableWebMvcConfiguration 的一个方法

```java
public static class EnableWebMvcConfiguration{
  
   @Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),// 欢迎页的路径就是上面的静态资源路径
					this.mvcProperties.getStaticPathPattern()); // StaticPathPattern = "/**"
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}
}
```

### 4. thymeleaf

```java
//Thymeleaf配置类属性
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html"
```

### 5. WebMvc视图解析器

WebMvcAutoConfiguration 自动配置类下的 静态类static class WebMvcAutoConfigurationAdapter 中

**视图解析器：根据方法（controler）的返回值得到视图对象（View）视图对象决定如何（渲染、转发、重定向）**

ContentNegotiatingViewResolver：组合所有的视图解析器

**如何添加我们自定义的视图解析器**

我们可以自己给容器中添加一个视图解析器，ContentNegotiatingViewResolver就会自动的将其组合进来

```java
// 自动配置了视图解析器		
		@Bean
		@ConditionalOnBean(ViewResolver.class)
		@ConditionalOnMissingBean(name = "viewResolver", value = ContentNegotiatingViewResolver.class)
		public ContentNegotiatingViewResolver viewResolver(BeanFactory beanFactory) {
			ContentNegotiatingViewResolver resolver = new ContentNegotiatingViewResolver();
			resolver.setContentNegotiationManager(beanFactory.getBean(ContentNegotiationManager.class));
			// ContentNegotiatingViewResolver uses all the other view resolvers to locate
			// a view so it should have a high precedence
			resolver.setOrder(Ordered.HIGHEST_PRECEDENCE);
			return resolver;
		}
```

**真正使用到的视图解析器**

```java
public class ContentNegotiatingViewResolver extends WebApplicationObjectSupport{
  
  @Override
	@Nullable
	public View resolveViewName(String viewName, Locale locale) throws Exception {
		RequestAttributes attrs = RequestContextHolder.getRequestAttributes();
		Assert.state(attrs instanceof ServletRequestAttributes, "No current ServletRequestAttributes");
		List<MediaType> requestedMediaTypes = getMediaTypes(((ServletRequestAttributes) attrs).getRequest());
		if (requestedMediaTypes != null) {
      // 获取候选的视图解析对象
			List<View> candidateViews = getCandidateViews(viewName, locale, requestedMediaTypes);
			// 获得一个最适合的视图对象，并将这个视图对象返回
      View bestView = getBestView(candidateViews, requestedMediaTypes, attrs);
			if (bestView != null) {
				return bestView;
			}
		}

		String mediaTypeInfo = logger.isDebugEnabled() && requestedMediaTypes != null ?
				" given " + requestedMediaTypes.toString() : "";

		if (this.useNotAcceptableStatusCode) {
			if (logger.isDebugEnabled()) {
				logger.debug("Using 406 NOT_ACCEPTABLE" + mediaTypeInfo);
			}
			return NOT_ACCEPTABLE_VIEW;
		}
		else {
			logger.debug("View remains unresolved" + mediaTypeInfo);
			return null;
		}
	}
  
  // 初始化视图解析器
  @Override
	protected void initServletContext(ServletContext servletContext) {
    // 从容器中获取所有的视图解析器，并把视图解析器作为所有要组合的视图解析器
    //BeanFactory就是IOC容器
		Collection<ViewResolver> matchingBeans =
				BeanFactoryUtils.beansOfTypeIncludingAncestors(obtainApplicationContext(), ViewResolver.class).values();
		if (this.viewResolvers == null) {
			this.viewResolvers = new ArrayList<>(matchingBeans.size());
			for (ViewResolver viewResolver : matchingBeans) {
				if (this != viewResolver) {
					this.viewResolvers.add(viewResolver);
				}
			}
		}
		else {
			for (int i = 0; i < this.viewResolvers.size(); i++) {
				ViewResolver vr = this.viewResolvers.get(i);
				if (matchingBeans.contains(vr)) {
					continue;
				}
				String name = vr.getClass().getName() + i;
				obtainApplicationContext().getAutowireCapableBeanFactory().initializeBean(vr, name);
			}

		}
		AnnotationAwareOrderComparator.sort(this.viewResolvers);
		this.cnmFactoryBean.setServletContext(servletContext);
	}
  
}

```



**添加自定义的视图解析器**

```java
package com.example.webrestfulcrud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.servlet.View;
import org.springframework.web.servlet.ViewResolver;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;
import java.util.Map;

@SpringBootApplication
public class WebRestfulCrudApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebRestfulCrudApplication.class, args);
	}

	/**
	 * 将我的自定义视图解析器添加到ICO容器中
	 * alt+shift+n 再按shift切换搜索 搜索 DispatcherServlet
	 * 并在其中的这个方法 打断点
	 * 	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception
	 * @return
	 */
	@Bean
	public ViewResolver myViewResolver(){
		return new MyViewResolver();
	}

	private static class MyViewResolver implements ViewResolver{

		@Override
		public View resolveViewName(String viewName, Locale locale) throws Exception {
			return new View() {
				@Override
				public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

				}
			};
		}
	}
}

```



![我的视图解析器](/Users/luo/Documents/开发笔记/images/我的视图解析器.png)

### 6. Converter,Formatter

Converter: 转换器， 将http请求中的数据封装成对象，交给controller

Formatter: 格式化器，2017-11-2 ----> Date对象

```java
	//可以通过这个添加格式化器，我们只需要将其放在容器中即可	
	@Override
		public void addFormatters(FormatterRegistry registry) {
			ApplicationConversionService.addBeans(registry, this.beanFactory);
		}
```

### 7. HttpMessageConverter 

springMVC用来转换Http请求和响应的；例如：将User对象转为json响应给浏览器

```java
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
		private final ObjectProvider<HttpMessageConverters> messageConvertersProvider;
  	//构造器，每一个对象都需要从IOC容器中取出，所以HttpMessageConverters要从容器中确定
  		public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
        //
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
		}
}
```

**因为上面的方法每次都会获取容器中的所有HttpMessageConverter，所以只需要将自己的HttpMessageConverter放入容器中即可(@Bean,@Compoment)**

```java
@Configuration
public class MessageConvertersConfiguration {
    
    @Bean
    public HttpMessageConverters customConverters(){
        
        HttpMessageConverter<?> additional = null;
        
        HttpMessageConverter<?> another = null;
        
        return new HttpMessageConverters(additional,another);
    }
}
```

### 8. 我们可以配置一个ConfigurableWebBindingInitializer来替换默认的（添加到容器中）

```java
//用于将Http发来的请求绑定到javaBean中	
@Override
		protected ConfigurableWebBindingInitializer getConfigurableWebBindingInitializer(
				FormattingConversionService mvcConversionService, Validator mvcValidator) {
			try {
				return this.beanFactory.getBean(ConfigurableWebBindingInitializer.class);
			}
			catch (NoSuchBeanDefinitionException ex) {
				return super.getConfigurableWebBindingInitializer(mvcConversionService, mvcValidator);
			}
		}
```

## 重点：如何修改SpringBoot的默认配置

1. springBoot在自动配置的时候，先看容器中有没有用户自己配置的(@Bean,@Component)。如果有用户自己配置的，就用用户自己配置的，如果没有，才进行自动配置

   ```java
   	@Bean
   	@ConditionalOnMissingBean(FormContentFilter.class) //当IOC容器中没有这个对象的时候，才执行这个@Bean
   	@ConditionalOnProperty(prefix = "spring.mvc.formcontent.filter", name = "enabled", matchIfMissing = true)
   	public OrderedFormContentFilter formContentFilter() {
   		return new OrderedFormContentFilter();
   	}
   ```

2. 如果有些组件可以有多个(ViewResolver)，spring将用户配置的和自己默认的组合起来

## 扩展SpringMvc

编写一个配置类（@Configuration），实现WebMvcConfigurer接口，不能标注@EnableWebMve

一旦标注了@EnableWebMve,就全面接管SpringMvc,springBoot对springMvc的配置就都不要了，所有都需要我们自己分配

```java
// 为什么一旦标注了@EnableWebMve，就全面接管
// 因为@EnableWebMvc注解会调用后面使用到的DelegatingWebMvcConfiguration类
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}

// 这个类继承WebMvcConfigurationSupport父类
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
}

@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
// IOC容器中没有这个对象WebMvcConfigurationSupport，这个WebMvcAutoConfiguration自动配置类才会生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
  
}
```

WebMvcConfigurationSupport 只是springMvc最基本的功能；

**既保留了所有的自动配置，也能用我们扩展的配置**

```java
// spring 的	WebMvcConfigurer 实现类
// Defined as a nested config to ensure WebMvcConfigurer is not read when not
	// on the classpath
	@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class) //注意导入了这个类
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
    
  }

/**
	 * Configuration equivalent to {@code @EnableWebMvc}.
	 */
	@Configuration(proxyBeanMethods = false)
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {
    
  }
```

**未来会有很多xxxConfiguration帮助我们扩展配置**

```java
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();


	@Autowired(required = false) //从容器中获取所有的WebMvcConfigurer
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);
		}
	}
  
  
  	@Override
	protected void addViewControllers(ViewControllerRegistry registry) {
		this.configurers.addViewControllers(registry);
	}
  
}
```

**容器中所有的WebMvcConfigure都会一起起作用**

```java
//把容器中的WebMvcConfigurer都拿来，且全部调用addViewControllers()	
@Override
	public void addViewControllers(ViewControllerRegistry registry) {
    // 注意这个加强for循环的接口WebMvcConfigurer
		for (WebMvcConfigurer delegate : this.delegates) {
			delegate.addViewControllers(registry);
		}
	}
```



**下面这个我自己写的WebMvcConfigurer实现类，也会被遍历到，也会一起起作用**

**效果：springMvc的配置和我们的Mvc扩展配置都会起作用**

```java
/**
 * 我自己的	WebMvcConfigurer 实现类
 * 使用WebMvcConfigurer可以扩张SpringMvc的功能
 */
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Bean
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        /**
         * 这样可以发送一个/nihao 请求，可以跳转 success.html页面
         * 就没必要单独的为success添加一个Controller了
         */
        registry.addViewController("/nihao").setViewName("success");

    }
}
```

### 9. 国际化

```Java
		@Bean
		@ConditionalOnMissingBean //容器中没有，才会创建国际化解析器，所以我们可以创建自己的国际化解析器
		@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
		public LocaleResolver localeResolver() {
			if (this.mvcProperties.getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
				return new FixedLocaleResolver(this.mvcProperties.getLocale());
			}
			AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
			localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
			return localeResolver;
		}
```

```java
/**
 * 我的国际化解析器
 * 需要创建该对象只不过是在其他地方创建而已
 */
public class MyLocaleResolver implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest request) {

        String language = request.getParameter("language");

        if (!StringUtils.isEmpty(language)) {
            /**
             * 因为传进来的是 zh_CN en_US
             * 所以可以用_进行分割
             */
            String[] s = language.split("_");

            Locale locale = new Locale(s[0],s[1]);
            return locale;
        }
        Locale defaultLocale = Locale.getDefault();
        return defaultLocale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}
```

```java
    /** 创建国际化对象解析器
     * 不可以更改方法名
     * @return
     */
    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
```

### 10. Thymeleaf视图解析器

```java
public class ThymeleafViewResolver extends AbstractCachingViewResolver  implements Ordered {
@Override
    protected View createView(final String viewName, final Locale locale) throws Exception {
        // First possible call to check "viewNames": before processing redirects and forwards
        if (!this.alwaysProcessRedirectAndForward && !canHandle(viewName, locale)) {
            vrlogger.trace("[THYMELEAF] View \"{}\" cannot be handled by ThymeleafViewResolver. Passing on to the next resolver in the chain.", viewName);
            return null;
        }
        // Process redirects (HTTP redirects)
        if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
            vrlogger.trace("[THYMELEAF] View \"{}\" is a redirect, and will not be handled directly by ThymeleafViewResolver.", viewName);
            final String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length(), viewName.length());
            final RedirectView view = new RedirectView(redirectUrl, isRedirectContextRelative(), isRedirectHttp10Compatible());
            return (View) getApplicationContext().getAutowireCapableBeanFactory().initializeBean(view, viewName);
        }
        // Process forwards (to JSP resources)
        if (viewName.startsWith(FORWARD_URL_PREFIX)) {
            // The "forward:" prefix will actually create a Servlet/JSP view, and that's precisely its aim per the Spring
            // documentation. See http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/mvc.html#mvc-redirecting-forward-prefix
            vrlogger.trace("[THYMELEAF] View \"{}\" is a forward, and will not be handled directly by ThymeleafViewResolver.", viewName);
            final String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length(), viewName.length());
            return new InternalResourceView(forwardUrl);
        }
        // Second possible call to check "viewNames": after processing redirects and forwards
        if (this.alwaysProcessRedirectAndForward && !canHandle(viewName, locale)) {
            vrlogger.trace("[THYMELEAF] View \"{}\" cannot be handled by ThymeleafViewResolver. Passing on to the next resolver in the chain.", viewName);
            return null;
        }
        vrlogger.trace("[THYMELEAF] View {} will be handled by ThymeleafViewResolver and a " +
                        "{} instance will be created for it", viewName, getViewClass().getSimpleName());
        return loadView(viewName, locale);
    }
}  
```

### 11. 日期格式化

```java
public class WebMvcAutoConfiguration {
		
@Bean
		@Override
		public FormattingConversionService mvcConversionService() {
			Format format = this.mvcProperties.getFormat(); //从这里获取日期
			WebConversionService conversionService = new WebConversionService(new DateTimeFormatters()
					.dateFormat(format.getDate()).timeFormat(format.getTime()).dateTimeFormat(format.getDateTime()));
			addFormatters(conversionService);
			return conversionService;
		}
}
```

springMvc中默认的日期格式

```java
	public static class Format {

		/**
		 * Date format to use, for example `dd/MM/yyyy`.
		 */
		private String date;

		/**
		 * Time format to use, for example `HH:mm:ss`.
		 */
		private String time;

		/**
		 * Date-time format to use, for example `yyyy-MM-dd HH:mm:ss`.
		 */
		private String dateTime;
}  
```

自定义日期格式化器的格式

```yaml
  spring
    mvc:
      format:
        date: yyyy/MM/dd  # 设置日期格式化器，默认是dd/MM/yyyy
        date-time: yyyy-MM-dd HH:mm:ss
        time: HH:mm:ss
```

### 12. 请求方法过滤器

```java
public class HiddenHttpMethodFilter extends OncePerRequestFilter {
	
@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		HttpServletRequest requestToUse = request;

		if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {
      //this.methodParam = "_method"
			String paramValue = request.getParameter(this.methodParam);
			if (StringUtils.hasLength(paramValue)) {
				String method = paramValue.toUpperCase(Locale.ENGLISH);
				if (ALLOWED_METHODS.contains(method)) {
					requestToUse = new HttpMethodRequestWrapper(request, method);
				}
			}
		}

		filterChain.doFilter(requestToUse, response);
	}
}
```

### 13. 异常处理器

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class })
// Load before the main WebMvcAutoConfiguration so that the error View is available
@AutoConfigureBefore(WebMvcAutoConfiguration.class)
@EnableConfigurationProperties({ ServerProperties.class, ResourceProperties.class, WebMvcProperties.class })
public class ErrorMvcAutoConfiguration {
  		//帮助我们共享信息
 			@Bean
      @ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
      public DefaultErrorAttributes errorAttributes() {
        return new DefaultErrorAttributes();
      } 
  
  		@Bean
      @ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)
      public BasicErrorController basicErrorController(ErrorAttributes errorAttributes,
          ObjectProvider<ErrorViewResolver> errorViewResolvers) {
        return new BasicErrorController(errorAttributes, this.serverProperties.getError(),
            errorViewResolvers.orderedStream().collect(Collectors.toList()));
      }
  
  
  		//一旦出现4xx,5xx之类的错误，就会生效：定制错误的响应规则
  		@Bean
      public ErrorPageCustomizer errorPageCustomizer(DispatcherServletPath dispatcherServletPath) {
        return new ErrorPageCustomizer(this.serverProperties, dispatcherServletPath);
      }
  
  		@Configuration(proxyBeanMethods = false)
	static class DefaultErrorViewResolverConfiguration {

		private final ApplicationContext applicationContext;

		private final ResourceProperties resourceProperties;

		DefaultErrorViewResolverConfiguration(ApplicationContext applicationContext,
				ResourceProperties resourceProperties) {
			this.applicationContext = applicationContext;
			this.resourceProperties = resourceProperties;
		}
///WebMvcAutoConfiguration自动配置的DefaultErrorViewResolver
          @Bean
          @ConditionalOnBean(DispatcherServlet.class)
          @ConditionalOnMissingBean(ErrorViewResolver.class)
          DefaultErrorViewResolver conventionErrorViewResolver() {
            return new DefaultErrorViewResolver(this.applicationContext, this.resourceProperties);
          }

	}
  
}
```

springboot通过ErrorPageCustomizer对象来处理4xx,5xx错误。系统出现错误以后，发送/error请求（web.xml注册错误页面规则）

```java
	/**
	 * {@link WebServerFactoryCustomizer} that configures the server's error pages.
	 */
	static class ErrorPageCustomizer implements ErrorPageRegistrar, Ordered {

		private final ServerProperties properties;

		private final DispatcherServletPath dispatcherServletPath;

		protected ErrorPageCustomizer(ServerProperties properties, DispatcherServletPath dispatcherServletPath) {
			this.properties = properties;
			this.dispatcherServletPath = dispatcherServletPath;
		}

		@Override
		public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
      //this.properties.getError().getPath())) = "/error"
			ErrorPage errorPage = new ErrorPage(
					this.dispatcherServletPath.getRelativePath(this.properties.getError().getPath()));
			errorPageRegistry.addErrorPages(errorPage);
		}

		@Override
		public int getOrder() {
			return 0;
		}

	}
```

ErrorPageCustomizer捕获到/error请求之后，就会被BasicErrorController处理

```java

//如果yml/properties中server.error.path路径不存在，则获取error.path:/error这个作为拦截路径
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
  
  // 这里@RequestMapping没有添加额外的路径，使用的是类上的@RequestMapping注解路径
	@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)//MediaType.TEXT_HTML_VALUE="text/html"，产生html类型的数据
  //produces属性：当浏览器的请求头中包含accept=text/html时，走的是这个controller
	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
		HttpStatus status = getStatus(request);
    //很复杂,getErrorAttributes()方法获取共享信息,最终调用的事DefaultErrorAttributes对象的信息
		Map<String, Object> model = Collections
				.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
    // 选取那个页面作为错误页面；包含页面内容和页面数据
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
    // 没有自定义的错误页面，则使用springBoot自带的错误页面，并返回，根据"error"作为id，从容器中获取到对应的视图@Bean("error")，WebMvcAutoConfiguration中有配置
		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
	}

	@RequestMapping	//产生json类型的数据，没有produces属性说明，当浏览器请求头包含accept=*/*时，走的是这个controller
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		HttpStatus status = getStatus(request);
		if (status == HttpStatus.NO_CONTENT) {
			return new ResponseEntity<>(status);
		}
		Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
		return new ResponseEntity<>(body, status);
	}
}
```

错误响应页面的解析代码；



```java
public abstract class AbstractErrorController implements ErrorController {
		protected ModelAndView resolveErrorView(HttpServletRequest request, HttpServletResponse response, HttpStatus status,
			Map<String, Object> model) {
      // 通过所有的errorViewResolvers得到ModelAndView
		for (ErrorViewResolver resolver : this.errorViewResolvers) {
      // 解析响应页面。这里的resolver是ErrorViewResolver接口的一个实例
			ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
			if (modelAndView != null) {
				return modelAndView;
			}
		}
		return null;
	}
}
```

响应页面：去那个响应页面是由ErrorViewResolver接口的resolveErrorView()方法解析的

```java
@FunctionalInterface
public interface ErrorViewResolver {

	/**
	 * Resolve an error view for the specified details.
	 * @param request the source request
	 * @param status the http status of the error
	 * @param model the suggested model to be used with the view
	 * @return a resolved {@link ModelAndView} or {@code null}
	 */
	ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model);

}
```

DefaultErrorViewResolver就是一个实现了ErrorViewResolver接口的解析器（WebMvcAutoConfiguration自动配置的DefaultErrorViewResolver）

```java
public class DefaultErrorViewResolver implements ErrorViewResolver, Ordered {

	private static final Map<Series, String> SERIES_VIEWS;

	static {
		Map<Series, String> views = new EnumMap<>(Series.class);
		views.put(Series.CLIENT_ERROR, "4xx");	//优先级较低
		views.put(Series.SERVER_ERROR, "5xx");
		SERIES_VIEWS = Collections.unmodifiableMap(views);
	}
  
  	@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}
  // 上个方法的状态码(例如404)会传到这个方法作为视图名字
	private ModelAndView resolve(String viewName, Map<String, Object> model) {
		// 然后拼接成视图的名字
    String errorViewName = "error/" + viewName;
    //如果模版引擎可以解析这个页面地址，就用模版引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
				this.applicationContext);
		if (provider != null) {
      //模版引擎可用的情况下，转发到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
    //模版引擎不可用
		return resolveResource(errorViewName, model);
	}
  
  	//模版引擎不可用时，调用的方法
  	private ModelAndView resolveResource(String viewName, Map<String, Object> model) {
      //在静态资源文件夹下找errorViewName对应的页面 /static/error/404.html
		for (String location : this.resourceProperties.getStaticLocations()) {
			try {
				Resource resource = this.applicationContext.getResource(location);
				resource = resource.createRelative(viewName + ".html");
				if (resource.exists()) {
          //静态资源文件夹下存在对应的错误页面
					return new ModelAndView(new HtmlResourceView(resource), model);
				}
			}
			catch (Exception ex) {
			}
		}
		return null;
	}
}
```

**总结：**

  1. 如何定制错误页面

       1. 有模版引擎的情况下，将错误页面的文件名改为 “状态吗.html”放在error/文件夹下

          例如：templates/error/404.html

          但是为每一个错误的状态码都定义一个错误页面太费时，可以定义一个4xx.html作为默认的显示4xx错误的页面（有精确错误代码的页面优先显示：同时有4xx.html,404.html，当有404错误的时候，优先跳转404.html）

          例如：templates/error/4xx.html

          （1）可以获取的错误信息，这个对象已经由WebMvcAutoConfiguration对象自动配置（创建），并且被BasicErrorController的errorHtml()方法调用过这个对象的getErrorAttributes()方法，获取错误信息

          ```java
          public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered {
          	// 能拿到所有信息，可以通过potions排除不需要的信息
          @Override
          	public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
              // getErrorAttributes()继续调用下一个方法
          		Map<String, Object> errorAttributes = getErrorAttributes(webRequest, options.isIncluded(Include.STACK_TRACE));
          		if (this.includeException != null) {
          			options = options.including(Include.EXCEPTION);
          		}
          		if (!options.isIncluded(Include.EXCEPTION)) {
          			errorAttributes.remove("exception");
          		}
          		if (!options.isIncluded(Include.STACK_TRACE)) {
          			errorAttributes.remove("trace");
          		}
          		if (!options.isIncluded(Include.MESSAGE) && errorAttributes.get("message") != null) {
          			errorAttributes.put("message", "");
          		}
          		if (!options.isIncluded(Include.BINDING_ERRORS)) {
          			errorAttributes.remove("errors");
          		}
          		return errorAttributes;
          	}
            
            	@Override
          	@Deprecated
          	public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
              // 最终，页面能获取到的信息
          		Map<String, Object> errorAttributes = new LinkedHashMap<>();
          		errorAttributes.put("timestamp", new Date());
              // 这些addxx()方法最终会把所有错误信息都添加进去，如果满足条件的话
          		addStatus(errorAttributes, webRequest);
          		addErrorDetails(errorAttributes, webRequest, includeStackTrace);
          		addPath(errorAttributes, webRequest);
          		return errorAttributes;
          	}
          }
          ```

          **页面最终能获取到的信息**

          timestamp:时间戳

          status: 状态码

          error:错误提示

          exception:异常对象

          message: 异常信息

          errors: JSR303数据校验错误的消息

          ```html
          				<h1>[[${timestamp}]]:时间戳</h1>
          				<h1>[[${status}]]: 状态码</h1>
          				<h1>[[${error}]]:错误提示</h1>
          				<h1>[[${exception}]]:异常对象</h1>
          				<h1>[[${message}]]: 异常信息</h1>
          				<h1>[[${errors}]]: JSR303数据校验错误的消息</h1>
          ```

          2. 没有模版引擎的情况下（模版引擎找不到这些错误页面），回去这个路径下寻找，并返回页面

             

![静态错误页面存放的位置](/Users/luo/Documents/开发笔记/images/静态错误页面存放的位置.png)

   3. 以上都没有，则来到springboot默认的错误导航页面

      ```java
      //WebMvcAutoConfiguration下的这个类返回springBoot默认的错误页面
      	@Configuration(proxyBeanMethods = false)
      	@ConditionalOnProperty(prefix = "server.error.whitelabel", name = "enabled", matchIfMissing = true)
      	@Conditional(ErrorTemplateMissingCondition.class)
      	protected static class WhitelabelErrorViewConfiguration {
      
      		private final StaticView defaultErrorView = new StaticView();
      
      		@Bean(name = "error")
      		@ConditionalOnMissingBean(name = "error")
      		public View defaultErrorView() {
      			return this.defaultErrorView;
      		}
      
      		// If the user adds @EnableWebMvc then the bean name view resolver from
      		// WebMvcAutoConfiguration disappears, so add it back in to avoid disappointment.
      		@Bean
      		@ConditionalOnMissingBean
      		public BeanNameViewResolver beanNameViewResolver() {
      			BeanNameViewResolver resolver = new BeanNameViewResolver();
      			resolver.setOrder(Ordered.LOWEST_PRECEDENCE - 10);
      			return resolver;
      		}
      
      	}
      ```

      

