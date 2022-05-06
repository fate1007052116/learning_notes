# Spring Security

## 一、基于内存存储令牌

[概述](https://funtl.com/zh/spring-security-oauth2/基于内存存储令牌.html#概述)

本章节基于 **内存存储令牌** 的模式用于演示最基本的操作，帮助大家快速理解 oAuth2 认证服务器中 "认证"、"授权"、"访问令牌” 的基本概念

**操作流程**

![oauth2-认证流程](images/oauth2-认证流程.png)

* 配置认证服务器

  * 配置客户端信息：

    ```java
    
    ```

    * `inMemory`：内存配置
    * `withClient`：客户端标识
    * `secret`：客户端安全码
    * `authorizedGrantTypes`：客户端授权类型
    * `scopes`：客户端授权范围
    * `redirectUris`：注册回调地址

* 配置 Web 安全

* 通过 GET 请求访问认证服务器获取授权码

  * 端点：`/oauth/authorize`

* 通过 POST 请求利用授权码访问认证服务器获取令牌

  * 端点：`/oauth/token`

**附：默认的端点 URL**

* `/oauth/authorize`：授权端点
* `/oauth/token`：令牌端点
* `/oauth/confirm_access`：用户确认授权提交端点
* `/oauth/error`：授权服务错误信息端点
* `/oauth/check_token`：用于资源服务访问的令牌解析端点
* `/oauth/token_key`：提供公有密匙的端点，如果你使用 JWT 令牌的话

### 1.[配置认证服务器](https://funtl.com/zh/spring-security-oauth2/基于内存存储令牌.html#配置认证服务器)

创建一个类继承 `AuthorizationServerConfigurerAdapter` 并添加相关注解：

* `@Configuration`
* `@EnableAuthorizationServer`

```java
@Configuration
@EnableAuthorizationServer
@Slf4j
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Resource
    PasswordEncoder passwordEncoder;

    /**
     * 对客户端进行认证配置
     * 客户端从这个地址获取授权码，client_id 需要和这个方法中配置的ClientId一致
     * http://localhost:8001/oauth/authorize?client_id=clientId&response_type=code
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        super.configure(clients);

        clients
                // 将客户端的请求信息放在内存中
                .inMemory()
                // 生成客户端id
                .withClient("clientId")
                // 客户端secret
                .secret(passwordEncoder.encode("clientSecret"))
                // 授权模式（4种，这里是授权码模式）
                .authorizedGrantTypes("authorization_code")
                // 授权范围
                .scopes("app")
                /* 拿到授权码之后，跳转的地址，
                 * 授权码会被以?的形式拼接到跳转的地址后面，即 http://127.0.0.1:8002/?code=5nx8EF
                 * 拿到授权码之后，再去认证服务器获得令牌的认证服务器地址
                 */
                .redirectUris("http://127.0.0.1:9001");
        log.info("正在配置认证服务器..");

        /**
         * 最后通过 http://clientId:clientSecret@localhost:8001/oauth/token 来获取 access_token
         * 
         * */
    }
}
```

### 2. [服务器安全配置](https://funtl.com/zh/spring-security-oauth2/基于内存存储令牌.html#服务器安全配置)

创建一个类继承 `WebSecurityConfigurerAdapter` 并添加相关注解：

* `@Configuration`
* `@EnableWebSecurity`
* `@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)`：全局方法拦截

```java

@EnableWebSecurity
/**
 * 全局方法拦截，prePostEnabled 启用注解 securedEnabled 和 注解jsr250Enabled
 * */
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
@Configuration
@Slf4j
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {


    @Resource
    PasswordEncoder passwordEncoder;


    /**
     * 验证用户的账号和密码
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                /*.withUser("admin").password("a1!").roles("Admin")
                    oAuth2中不允许密码通过明文来存储
                 */
                .withUser("admin").password(passwordEncoder.encode("a1!")).roles("Admin")
                .and()
                .withUser("user").password(passwordEncoder.encode("123")).roles("User");

        log.info("正在在内存中创建账号密码..");
    }
}
```

[application.yml](https://funtl.com/zh/spring-security-oauth2/基于内存存储令牌.html#application-yml)

```yaml
spring:
  application:
    name: oauth2-server
server:
  port: 8080
```

### 3. [访问获取授权码](https://funtl.com/zh/spring-security-oauth2/基于内存存储令牌.html#访问获取授权码)

打开浏览器，输入地址：

```shell
# client_id=clientId 中的 clientId 需要和 .withClient("clientId") 中的一致
http://localhost:8001/oauth/authorize?client_id=clientId&response_type=code
```

第一次访问会跳转到登录页面

![img](https://funtl.com/assets1/Lusifer_20190401195014.png)

验证成功后会询问用户是否授权客户端

![img](https://funtl.com/assets1/Lusifer_20190401195129.png)

选择授权后会跳转到我的博客，浏览器地址上还会包含一个授权码（`code=1JuO6V`），浏览器地址栏会显示如下地址：

```text
http://127.0.0.1:8002/?code=5nx8EF
```

有了这个授权码就可以获取访问令牌了

### 4. [通过授权码向服务器申请令牌](https://funtl.com/zh/spring-security-oauth2/基于内存存储令牌.html#)

通过 CURL 或是 Postman 请求

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=authorization_code&code=1JuO6V' "http://client:secret@localhost:8080/oauth/token"
```

![Snip20200908_1](/Users/luo/Documents/开发笔记/images/Snip20200908_1.png)

### 5. 至此获取到了 access_token

**注意：此时无法请求到令牌，访问服务器会报错 `There is no PasswordEncoder mapped for the id “null”`解决方案请移步 [这里](https://funtl.com/zh/spring-security-oauth2/PasswordEncoder.html)**







## 二、基于 JDBC 存储令牌  

![Snip20200908_2](/Users/luo/Documents/开发笔记/images/Snip20200908_2.png)

* 初始化 oAuth2 相关表

* 在数据库中配置客户端

* 配置认证服务器

  * 配置数据源：`DataSource`

  * 配置令牌存储方式：`TokenStore` -> `JdbcTokenStore`

  * 配置客户端读取方式：`ClientDetailsService` -> `JdbcClientDetailsService`

  * 配置服务端点信息：

    ```
    AuthorizationServerEndpointsConfigurer
    ```

    * `tokenStore`：设置令牌存储方式

  * 配置客户端信息：

    ```
    ClientDetailsServiceConfigurer
    ```

    * `withClientDetails`：设置客户端配置读取方式

* 配置 Web 安全

  * 配置密码加密方式：`BCryptPasswordEncoder`
  * 配置认证信息：`AuthenticationManagerBuilder`

* 通过 GET 请求访问认证服务器获取授权码

  * 端点：`/oauth/authorize`

* 通过 POST 请求利用授权码访问认证服务器获取令牌

  * 端点：`/oauth/token`

**附：默认的端点 URL**

* `/oauth/authorize`：授权端点
* `/oauth/token`：令牌端点
* `/oauth/confirm_access`：用户确认授权提交端点
* `/oauth/error`：授权服务错误信息端点
* `/oauth/check_token`：用于资源服务访问的令牌解析端点
* `/oauth/token_key`：提供公有密匙的端点，如果你使用 JWT 令牌的话

### 1. 初始化 oAuth2 相关表

使用官方提供的建表脚本初始化 oAuth2 相关表，地址如下：

```text
https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql
```

由于我们使用的是 MySQL 数据库，默认建表语句中主键为 `VARCHAR(256)`，这超过了最大的主键长度，请手动修改为 `128`，并用 `BLOB` 替换语句中的 `LONGVARBINARY` 类型，修改后的建表脚本如下：

```sql
CREATE TABLE `clientdetails` (
  `appId` varchar(128) NOT NULL,
  `resourceIds` varchar(256) DEFAULT NULL,
  `appSecret` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `grantTypes` varchar(256) DEFAULT NULL,
  `redirectUrl` varchar(256) DEFAULT NULL,
  `authorities` varchar(256) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additionalInformation` varchar(4096) DEFAULT NULL,
  `autoApproveScopes` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`appId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_access_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication_id` varchar(128) NOT NULL,
  `user_name` varchar(256) DEFAULT NULL,
  `client_id` varchar(256) DEFAULT NULL,
  `authentication` blob,
  `refresh_token` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`authentication_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_approvals` (
  `userId` varchar(256) DEFAULT NULL,
  `clientId` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `status` varchar(10) DEFAULT NULL,
  `expiresAt` timestamp NULL DEFAULT NULL,
  `lastModifiedAt` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_client_details` (
  `client_id` varchar(128) NOT NULL,
  `resource_ids` varchar(256) DEFAULT NULL,
  `client_secret` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `authorized_grant_types` varchar(256) DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) DEFAULT NULL,
  `authorities` varchar(256) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additional_information` varchar(4096) DEFAULT NULL,
  `autoapprove` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`client_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_client_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication_id` varchar(128) NOT NULL,
  `user_name` varchar(256) DEFAULT NULL,
  `client_id` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`authentication_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_code` (
  `code` varchar(256) DEFAULT NULL,
  `authentication` blob
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_refresh_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication` blob
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2. [在数据库中配置客户端](https://funtl.com/zh/spring-security-oauth2/基于-JDBC-存储令牌.html#在数据库中配置客户端)

在表 `oauth_client_details` 中增加一条客户端配置记录，需要设置的字段如下：

* `client_id`：客户端标识
* `client_secret`：客户端安全码，**此处不能是明文，需要加密**
* `scope`：客户端授权范围
* `authorized_grant_types`：客户端授权类型
* `web_server_redirect_uri`：服务器回调地址

使用 `BCryptPasswordEncoder` 为客户端安全码加密，代码如下：

```java
    public static void main(String[] args) {
        PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

        String encode = passwordEncoder.encode("123");

        System.out.println(encode);
    }
```

![Snip20200908_6](/Users/luo/Documents/开发笔记/images/Snip20200908_6.png)

### 3. 数据源报错

原来使用Spring Boot 1.x版本，升级2.x之后，报如下异常：

或者像上面那样使用自定义的数据源配置读取`dataSource`的时候

```shell
### Cause: java.lang.IllegalArgumentException: jdbcUrl is required with driverClassName.
    at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:92)
    at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:440)
    at com.sun.proxy.$Proxy68.selectOne(Unknown Source)
```



此异常的主要原因是：在1.x配置数据源配置为：

```properties
spring.datasource.url=

spring.datasource.driverClassName=
```



升级之后，需要配置如下：

```properties
spring.datasource.jdbc-url=

spring.datasource.username=

spring.datasource.password=

spring.datasource.driver-class-name=
```



也就是说url替换为jdbc-url，driverClassName替换为driver-class-name。

### 4. 访问获取授权码

> http://localhost:8001/oauth/authorize?client_id=clientId&response_type=code
>
> 获取到的授权码
>
> http://localhost:9001/?code=HoHvEJ

### 5. 

# 全新开始学习spring-boot-starter-security



## 一、总体架构

### 1.UserDetailService接口

spring security会调用这个接口来查询用户的账号和密码，实现自定义逻辑时，只要实现这个接口即可

### 2.UserNamePasswordAuthticationFilter

用于认证用户名和密码


```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
  
	public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
		if (postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}

		String username = obtainUsername(request);
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();

		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password);

		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);

		return this.getAuthenticationManager().authenticate(authRequest);
	}
}
```



如果认证成功会调用父类的`successful()`方法，失败调用`unsuccessfulAuthtication()`方法

> 对于所有的`Filter`：我们需要继承这个类重写`doFilter()`方法，来实现自定义认证

```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean
		implements ApplicationEventPublisherAware, MessageSourceAware {
		
  	// 记住密码的默认实现类
  	private RememberMeServices rememberMeServices = new NullRememberMeServices();

  
  public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
    
    // 判断当前请求是否需要认证，如果不需要认证就不用认证了
    if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);

			return;
		}
    
    // 省略
    Authentication authResult;

		try {
      // 调用本类的抽象方法进行认证
			authResult = attemptAuthentication(request, response);
			if (authResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				// authentication
				return;
			}
			sessionStrategy.onAuthentication(authResult, request, response);
		}
		catch (InternalAuthenticationServiceException failed) {
			logger.error(
					"An internal error occurred while trying to authenticate the user.",
					failed);
			unsuccessfulAuthentication(request, response, failed);	// 认证失败

			return;
		}
		catch (AuthenticationException failed) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, failed); // 认证失败

			return;
		}
    
    
		// Authentication success
		if (continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}

		successfulAuthentication(request, response, chain, authResult); // 认证成功时调用
  }
  
  	// 认证成功的方法
  	protected void successfulAuthentication(HttpServletRequest request,
			HttpServletResponse response, FilterChain chain, Authentication authResult)
			throws IOException, ServletException {

		if (logger.isDebugEnabled()) {
			logger.debug("Authentication success. Updating SecurityContextHolder to contain: "
					+ authResult);
		}

		SecurityContextHolder.getContext().setAuthentication(authResult);

		rememberMeServices.loginSuccess(request, response, authResult); // 执行记住我的相关操作

		// Fire event
		if (this.eventPublisher != null) {
			eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(
					authResult, this.getClass()));
		}

		successHandler.onAuthenticationSuccess(request, response, authResult);
	}
}
```



### 3.PasswordEncoder接口

加密密码和匹配密码

## 二、配置

### 1.使用配置文件配置用户名和密码

```yml
spring:
  security:
    user:
      name: luo
      password: 123
      roles:
        - admin
```

### 2.基于配置类实现用户名和密码的配置

> 一旦在配置类中配置了用户名和密码，yml中配置的用户信息就失效了

```java
@Configuration
@Slf4j
public class WebSecurityConfigTestUserNameAndPassword extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        /**
         * 一定要注释 super.configure(auth);
         * 父方法的内容是
         * this.disableLocalConfigureAuthenticationBldr = true;
         * */
//        super.configure(auth);

        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

        auth.inMemoryAuthentication().passwordEncoder(passwordEncoder);
        // 这里只是方便测试，手动加密一下密码，实际不是这么配置的
        String encode = passwordEncoder.encode("1234");

        auth.inMemoryAuthentication().withUser("lucy").password(encode).roles("admin");
    }

    /**
     * 必须往容器中配置 PasswordEncoder （注意加密和解密都是要用这个 PasswordEncoder）
     * 否则： There is no PasswordEncoder mapped for the id "null"
     *
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        log.info("正在配置认证的密码加密算法");
        return new BCryptPasswordEncoder();
    }
}
```

### 3.配置spring security 使用 自定义实现的 UserDetailService 实现类来从数据库中获取账号名和密码

## 三、注解的使用

### 1.@Secured

只有具有该角色才能进入`Controller`

> 注意：这里配置字符串需要添加前缀`ROLE_`
>
> 而且要在启动类上使用注解开启
>
> ```java
> @EnableGlobalMethodSecurity(securedEnabled = true)
> ```

```java
    @GetMapping("/deleteDB")
    @Secured(SysConstant.Role.DBA) // 常量 SysConstant.Role.DBA = "ROLE_dba"
    public String deleteDB() {
        return "角色是DBA的人要删除数据库啦";
    }
```



### 2.@PreAuthorize

只有具有该权限的用户才能进入`Controller`

> 在启类上开启该注解的支持
>
> ```java
> @EnableGlobalMethodSecurity(prePostEnabled = true)
> ```

```java
    @GetMapping("/installRouter")
//    @PreAuthorize("hasAnyAuthority('SysConstant.Authority.INSTALL_ROUTER')") //因为涉及到EL表达式，所以不能使用常量
    @PreAuthorize("hasAnyAuthority('installRouter')")
    public String installRouter() {
        return "有权限安装路由器";
    }
```

### 3.@PostAuthorize

在方法执行之后验证用户权限（权限验证时候通过，controller 都会执行）

> 在启类上开启该注解的支持
>
> ```java
> @EnableGlobalMethodSecurity(prePostEnabled = true)
> ```

```java
    @GetMapping("/downloadRouteTable")
    @PostAuthorize("hasAnyAuthority('downloadRouteTable')")
    public String downloadRouteTable() {
        return "您有权下载路由表";
    }
```

### 4.@PostFilter

```java
    /**
     * 对方法返回的结果进行过滤，只有符合条件的结果能返回到前端
     *
     * @return
     */
    @GetMapping("/getAllUserInfo")
    @PostFilter("filterObject.sex.equals('0')") // 这里的 sex 字段类型是 String '0' 代表男人，'1'代表女人
    public List<UserEntity> getAllUserInfo() {

        List<UserEntity> list = userService.list();

        return list;
    }
```

### 5.@PreFilter

```java
    /**
     * 对传入方法的数据进行过滤（未测试）
     * @return
     */
    @PostMapping("insertUser")
    @PreFilter("filterObject.sex.equals('1')")
    public String insertUser(@RequestBody UserEntity userEntity){
        this.userService.save(userEntity);
        return "插入用户成功";
    }
```

## 四、自动登录

### 1.实现自动登录的方式

#### （1）cookie技术

#### （2）安全框架机制实现自动登陆

![Snip20201219_1](/Users/luo/Documents/开发笔记/images/Snip20201219_1.png)

![Snip20201219_2](/Users/luo/Documents/开发笔记/images/Snip20201219_2.png)

### 2.自动登录源码

#### （1）第一次登录

默认使用的自动登录类（没有实现自动登录）

```java
public class NullRememberMeServices implements RememberMeServices {
	// ~ Methods
	// ========================================================================================================

	public Authentication autoLogin(HttpServletRequest request,
			HttpServletResponse response) {
		return null;
	}

	public void loginFail(HttpServletRequest request, HttpServletResponse response) {
	}

	public void loginSuccess(HttpServletRequest request, HttpServletResponse response,
			Authentication successfulAuthentication) {
    // 没有进行记住我的操作
	}
}
```



```java
public abstract class AbstractRememberMeServices implements RememberMeServices,
		InitializingBean, LogoutHandler {
      
  @Override
	public final void loginSuccess(HttpServletRequest request,
			HttpServletResponse response, Authentication successfulAuthentication) {

		if (!rememberMeRequested(request, parameter)) {
			logger.debug("Remember-me login not requested.");
			return;
		}

		onLoginSuccess(request, response, successfulAuthentication); // 调用下面的抽象方法
	}
      
  protected abstract void onLoginSuccess(HttpServletRequest request,
			HttpServletResponse response, Authentication successfulAuthentication); //有两个实现本方法的类

}
```



```java
public class PersistentTokenBasedRememberMeServices extends AbstractRememberMeServices {
  		
  
  	/**
	 * Creates a new persistent login token with a new series number, stores the data in
	 * the persistent token repository and adds the corresponding cookie to the response.
	 *
	 */
	protected void onLoginSuccess(HttpServletRequest request,
			HttpServletResponse response, Authentication successfulAuthentication) {
		String username = successfulAuthentication.getName();

		logger.debug("Creating new persistent login for user " + username);

		PersistentRememberMeToken persistentToken = new PersistentRememberMeToken(
				username, generateSeriesData(), generateTokenData(), new Date());
		try {
			tokenRepository.createNewToken(persistentToken); // 生成一个 token 字符串
			addCookie(persistentToken, request, response);		// 将这个 token 放到 cookie 中去
		}
		catch (Exception e) {
			logger.error("Failed to save persistent token ", e);
		}
	}
}
```

使用持久层存储`token`

> 从它的建表语句可以看出，`spring-security`甚至会在数据库中建表

```java
public class JdbcTokenRepositoryImpl extends JdbcDaoSupport implements PersistentTokenRepository {
		
  	/** Default SQL for creating the database table to store the tokens */
	public static final String CREATE_TABLE_SQL = "create table persistent_logins (username varchar(64) not null, series varchar(64) primary key, "
			+ "token varchar(64) not null, last_used timestamp not null)";
	/** The default SQL used by the <tt>getTokenBySeries</tt> query */
	public static final String DEF_TOKEN_BY_SERIES_SQL = "select username,series,token,last_used from persistent_logins where series = ?";
	/** The default SQL used by <tt>createNewToken</tt> */
	public static final String DEF_INSERT_TOKEN_SQL = "insert into persistent_logins (username, series, token, last_used) values(?,?,?,?)";
	/** The default SQL used by <tt>updateToken</tt> */
	public static final String DEF_UPDATE_TOKEN_SQL = "update persistent_logins set token = ?, last_used = ? where series = ?";
	/** The default SQL used by <tt>removeUserTokens</tt> */
	public static final String DEF_REMOVE_USER_TOKENS_SQL = "delete from persistent_logins where username = ?";
  
  	public void createNewToken(PersistentRememberMeToken token) {
		getJdbcTemplate().update(insertTokenSql, token.getUsername(), token.getSeries(),
				token.getTokenValue(), token.getDate());
	}

	public void updateToken(String series, String tokenValue, Date lastUsed) {
		getJdbcTemplate().update(updateTokenSql, tokenValue, lastUsed, series);
	}

}
```

#### （2）浏览器再次登录时

会被过滤器检测到`cookie`中之前存入的`token`

```java
public class RememberMeAuthenticationFilter extends GenericFilterBean implements
		ApplicationEventPublisherAware {
  
  	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		if (SecurityContextHolder.getContext().getAuthentication() == null) {
			Authentication rememberMeAuth = rememberMeServices.autoLogin(request, // 在这里执行自动登录，跳回之前的 AbstractRememberMeServices
					response);

			if (rememberMeAuth != null) {
				// Attempt authenticaton via AuthenticationManager
				try {
					rememberMeAuth = authenticationManager.authenticate(rememberMeAuth);

					// Store to SecurityContextHolder
					SecurityContextHolder.getContext().setAuthentication(rememberMeAuth);

					onSuccessfulAuthentication(request, response, rememberMeAuth);

					if (logger.isDebugEnabled()) {
						logger.debug("SecurityContextHolder populated with remember-me token: '"
								+ SecurityContextHolder.getContext().getAuthentication()
								+ "'");
					}

					// Fire event
					if (this.eventPublisher != null) {
						eventPublisher
								.publishEvent(new InteractiveAuthenticationSuccessEvent(
										SecurityContextHolder.getContext()
												.getAuthentication(), this.getClass()));
					}

					if (successHandler != null) {
						successHandler.onAuthenticationSuccess(request, response,
								rememberMeAuth);

						return;
					}

				}
				catch (AuthenticationException authenticationException) {
					if (logger.isDebugEnabled()) {
						logger.debug(
								"SecurityContextHolder not populated with remember-me token, as "
										+ "AuthenticationManager rejected Authentication returned by RememberMeServices: '"
										+ rememberMeAuth
										+ "'; invalidating remember-me token",
								authenticationException);
					}

					rememberMeServices.loginFail(request, response);

					onUnsuccessfulAuthentication(request, response,
							authenticationException);
				}
			}

			chain.doFilter(request, response);
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("SecurityContextHolder not populated with remember-me token, as it already contained: '"
						+ SecurityContextHolder.getContext().getAuthentication() + "'");
			}

			chain.doFilter(request, response);
		}
	}
  
}
```

`AbstractRememberMeServices`

```java
public abstract class AbstractRememberMeServices implements RememberMeServices,
		InitializingBean, LogoutHandler {
      
      @Override
	public final Authentication autoLogin(HttpServletRequest request,
			HttpServletResponse response) {
		String rememberMeCookie = extractRememberMeCookie(request); // 从请求中的cookie中解析出token

		if (rememberMeCookie == null) {
			return null;
		}

		logger.debug("Remember-me cookie detected");

		if (rememberMeCookie.length() == 0) {
			logger.debug("Cookie was empty");
			cancelCookie(request, response);
			return null;
		}

		UserDetails user = null;

		try {
			String[] cookieTokens = decodeCookie(rememberMeCookie); // 将 token 解密
			user = processAutoLoginCookie(cookieTokens, request, response);
			userDetailsChecker.check(user);													// 判断浏览器传过来的 token 和数据库中存储的数据是否一致

			logger.debug("Remember-me cookie accepted");

			return createSuccessfulAuthentication(request, user);		// 执行自动登录
		}
		catch (CookieTheftException cte) {
			cancelCookie(request, response);
			throw cte;
		}
		catch (UsernameNotFoundException noUser) {
			logger.debug("Remember-me login was valid but corresponding user not found.",
					noUser);
		}
		catch (InvalidCookieException invalidCookie) {
			logger.debug("Invalid remember-me cookie: " + invalidCookie.getMessage());
		}
		catch (AccountStatusException statusInvalid) {
			logger.debug("Invalid UserDetails: " + statusInvalid.getMessage());
		}
		catch (RememberMeAuthenticationException e) {
			logger.debug(e.getMessage());
		}

		cancelCookie(request, response);
		return null;
	}
      
    }
```

### 3.设置自动登录

配置`jdbcTokenRepository`

```java
@Configuration
@Slf4j
public class RememberMeConfig {


    @Bean
    public PersistentTokenRepository persistentTokenRepository(DataSource dataSource) {

        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();

        jdbcTokenRepository.setDataSource(dataSource);

        // 启动的时候执行创建 user-token 表
        /**
         * 以下为执行的建表语句
         * 	Default SQL for creating the database table to store the tokens
         *
         * public static final String CREATE_TABLE_SQL = "create table persistent_logins (username varchar(64) not null, series varchar(64) primary key, "
         * +"token varchar(64) not null, last_used timestamp not null)";
         * */
        jdbcTokenRepository.setCreateTableOnStartup(true);

        log.info("正在初始化 RememberMe 功能的 jdbcTokenRepository");

        return jdbcTokenRepository;
    }
}
```

配置

```java
@Configuration
@Slf4j
public class WebSecurityConfigWithUserDetailService extends WebSecurityConfigurerAdapter {    

		@Override
    protected void configure(HttpSecurity http) throws Exception {
//        super.configure(http);
        http.
/**
                 * 配置登陆时的记住我
                 * */
                .rememberMe()
                .tokenRepository(persistentTokenRepository) // 操作数据库的对象
                .tokenValiditySeconds(60*60)                   // 设置 token 的过期时间
                .userDetailsService(userDetailsService)     // 设置用 userDetailsService 去操作数据库
                .rememberMeCookieName("rememberMe-suibian") // 设置用于rememberMe 的 前端 cookie 的 token 的名字
                .rememberMeParameter("rememberMe");         // 设置前端 传递过来的记住我的 name  <input type="checkbox" name="rememberMe"/>
    }
  
}
```

## 五、CSRF的理解

### 1.跨域请求伪造

`Cross-Site Request Forgery`，也被称作`one-click attack`或者`session ridding`，通常缩写为`CSRF`或`XSRF`，是一种挟制用户在当前已登陆的`Web`应用程序上执行非本意操作的攻击方式。**跨网站脚本(Xss)**相比，`XSS`利用的是用户对指定网站的信任，`CSRF`利用的是网站对用户浏览器的信任。

### 2.跨站请求攻击

简单来说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个用户自己曾经认证过的网很赞并运行一些操作（如发邮件、发消息、甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作去运行。这**利用了Web中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的**。

从`spring security 4.0`开始，默认情况下会启动`CSRF`保护，防止`CSRF`攻击应用程序，`spring security csrf`会针对`Patch`，`Post`，`put`和`delete`方法进行保护

### 3.spring security 实现 csrf 原理

#### 生成csrfToken保存到HttpSession或者cookie中

```java
public final class CsrfFilter extends OncePerRequestFilter {
  	
  @Override
	protected void doFilterInternal(HttpServletRequest request,
			HttpServletResponse response, FilterChain filterChain)
					throws ServletException, IOException {
		request.setAttribute(HttpServletResponse.class.getName(), response);

		CsrfToken csrfToken = this.tokenRepository.loadToken(request); // 从 request 中获取 token
		final boolean missingToken = csrfToken == null;
		if (missingToken) {
			csrfToken = this.tokenRepository.generateToken(request);
			this.tokenRepository.saveToken(csrfToken, request, response);
		}
		request.setAttribute(CsrfToken.class.getName(), csrfToken);			// 将 _csrf.token 设置到request中，方便thymeleaf获取
		request.setAttribute(csrfToken.getParameterName(), csrfToken); // 将 _csrf.parameterName设置到 request 中，作为提交 _csrf.token的参数名

		if (!this.requireCsrfProtectionMatcher.matches(request)) { // 比较 request 中的 csrf token 和 session 中的token 是否一致
			filterChain.doFilter(request, response);
			return;
		}

		String actualToken = request.getHeader(csrfToken.getHeaderName());
		if (actualToken == null) {
			actualToken = request.getParameter(csrfToken.getParameterName());
		}
		if (!csrfToken.getToken().equals(actualToken)) {
			if (this.logger.isDebugEnabled()) {
				this.logger.debug("Invalid CSRF token found for "
						+ UrlUtils.buildFullRequestUrl(request));
			}
			if (missingToken) {
				this.accessDeniedHandler.handle(request, response,
						new MissingCsrfTokenException(actualToken));
			}
			else {
				this.accessDeniedHandler.handle(request, response,
						new InvalidCsrfTokenException(csrfToken, actualToken));
			}
			return;
		}

		filterChain.doFilter(request, response);
	}
  
  
  	private static final class DefaultRequiresCsrfMatcher implements RequestMatcher {
		private final HashSet<String> allowedMethods = new HashSet<>(
				Arrays.asList("GET", "HEAD", "TRACE", "OPTIONS"));  // 静态内部类中指明了不支持 csrf 保护的方法

		/*
		 * (non-Javadoc)
		 *
		 * @see
		 * org.springframework.security.web.util.matcher.RequestMatcher#matches(javax.
		 * servlet.http.HttpServletRequest)
		 */
		@Override
		public boolean matches(HttpServletRequest request) {
			return !this.allowedMethods.contains(request.getMethod());
		}
	}
}
```

### 4.如何配置

#### （1）开启csrf保护

```java
// 注释掉关闭csrf保护
//                .csrf().disable() //关闭 csrf 环护（当这里注释标示 开启 csrf 保护）
```

#### （2）登陆时携带csrf toekn

1、`thymeleaf`配置方式

```java
    <form action="/auth/login" method="post">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
        <div>
            <label for="user-name-label">用户名</label>
            <input name="username" id="user-name-label" value="luo"/>
        </div>
        <div>
            <label for="password-label" >密码</label>
            <input name="password" id="password-label" value="1"/>
        </div>
        <div>
            <label for="rememberMe-label" >记住我</label>
            <input type="checkbox" name="rememberMe" id="rememberMe-label"/>
        </div>
        <div>
            <button type="submit">登陆</button>
        </div>
    </form>
```

## 六、微服务认证授权

### 1.认证授权过程分析

（1）如果是基于`session`，那么`spring-security`会对`cookie`里的`sessionId`进行解析，找到服务器里存储的`session`信息，然后判断当前用户是否符合请求的要求。

（2）如果是`token`，则解析出`token`，然后将当前请求加入到`spring-security`管理的权限信息中去

![Snip20201220_1](/Users/luo/Documents/开发笔记/images/Snip20201220_1.png)















## 七、读Spring Security部分源码有感

### 1. AuthenticationManager 认证接口

>The [`Authentication`](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authentication-authentication) that is returned is then set on the [SecurityContextHolder](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-authentication-securitycontextholder) by the controller (i.e. [Spring Security’s `Filters`s](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#servlet-security-filters)) that invoked the `AuthenticationManager`
>
>由调用AuthenticationManager的控制器(即Spring Security的filter)在SecurityContextHolder上设置返回的 Authentication。如果你没有集成Spring Security的过滤器，你可以直接设置SecurityContextHolder，而不需要使用AuthenticationManager

```java
package org.springframework.security.authentication;

// 定义了 spring security 的 filter 如何进行认证

public interface AuthenticationManager {
  
  // 返回一个完全填充的Authentication对象
  	Authentication authenticate(Authentication authentication) throws AuthenticationException;
  
}
```



> 多个`AuthenticationProvider`可以背自动注入到`ProviderManager`中 

```java
package org.springframework.security.authentication;

public interface AuthenticationProvider {
  
    // 和 AuthenticationManager.authenticate() 方法功能类似
 
  	Authentication authenticate(Authentication authentication) throws AuthenticationException;
  
  	boolean supports(Class<?> authentication);
  
}
```

### 2.认证管理器的建造者

```java
public class AuthenticationManagerBuilder extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder> implements ProviderManagerBuilder<AuthenticationManagerBuilder> {
  
  // 如果 authenticationProviders 中的都调用完了，还是不能进行认证，那么就调用 parentAuthenticationProvider 来进行认证（如果有的话）
  private AuthenticationManager parentAuthenticationManager;
  // 存储 AuthenticationProvider 列表，当要进行认证的时候，就依次调用 list 中的 AuthenticationProvider，看看谁能进行认证
	private List<AuthenticationProvider> authenticationProviders = new ArrayList<>();
	private UserDetailsService defaultUserDetailsService;
  
  
    // 添加 AuthenticationProvider 到 list 中
  public AuthenticationManagerBuilder authenticationProvider(
			AuthenticationProvider authenticationProvider) {
		this.authenticationProviders.add(authenticationProvider);
		return this;
	}
  
  
  @Override
	protected ProviderManager performBuild() throws Exception {
		if (!isConfigured()) {
			logger.debug("No authenticationProviders and no parentAuthenticationManager defined. Returning null.");
			return null;
		}
    // 4.最终创建了 ProviderManager
		ProviderManager providerManager = new ProviderManager(authenticationProviders,
				parentAuthenticationManager);
		if (eraseCredentials != null) {
			providerManager.setEraseCredentialsAfterAuthentication(eraseCredentials);
		}
		if (eventPublisher != null) {
			providerManager.setAuthenticationEventPublisher(eventPublisher);
		}
		providerManager = postProcess(providerManager);
		return providerManager;
	}
  
  public AuthenticationManagerBuilder parentAuthenticationManager(
			AuthenticationManager authenticationManager) {
		if (authenticationManager instanceof ProviderManager) {
			eraseCredentials(((ProviderManager) authenticationManager)
					.isEraseCredentialsAfterAuthentication());
		}
    //7. 此时暂时只考虑自定义WebSecurityConfigurerAdapter子类的情形
    //子类的配置全部存于 localConfigureAuthenticationBldr ，并且通过 localConfigureAuthenticationBldr 建造了 authenticationManager
    // 此时将 authenticationManager 设置为 WebSecurityConfigurerAdapter 的 authenticationBuilder 的 parentAuthenticationManager
		this.parentAuthenticationManager = authenticationManager;
		return this;
	}
}
```



### 3.配置类

```java
package org.springframework.security.config.annotation.web.configuration;

public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {
  
  // 认证管理器的建造者
  private AuthenticationManagerBuilder authenticationBuilder;
  private AuthenticationManagerBuilder localConfigureAuthenticationBldr;

  // 此 boolean 决定 是否关闭 上面的 localConfigureAuthenticationBldr，默认为 false
  private boolean disableLocalConfigureAuthenticationBldr;


  // 注意 @Autowired 注解，会自动注入 ioc 容器
  @Autowired
	public void setApplicationContext(ApplicationContext context) {
		this.context = context;

		ObjectPostProcessor<Object> objectPostProcessor = context.getBean(ObjectPostProcessor.class);
		LazyPasswordEncoder passwordEncoder = new LazyPasswordEncoder(context);

    // 此时创建 认证管理器的建造者
		authenticationBuilder = new DefaultPasswordEncoderAuthenticationManagerBuilder(objectPostProcessor, passwordEncoder);
    
    // 此时创建临时的 认证管理器的建造者
		localConfigureAuthenticationBldr = new DefaultPasswordEncoderAuthenticationManagerBuilder(objectPostProcessor, passwordEncoder) {
			@Override
			public AuthenticationManagerBuilder eraseCredentials(boolean eraseCredentials) {
				authenticationBuilder.eraseCredentials(eraseCredentials);
				return super.eraseCredentials(eraseCredentials);
			}

		};
	}
  
  // 因为 实现了 SecurityConfigurer 接口，所以有这个方法
  public void init(final WebSecurity web) throws Exception {
		final HttpSecurity http = getHttp();  // 1.调用下一个方法
		// 省略
	}
  
  	protected final HttpSecurity getHttp() throws Exception {
		if (http != null) {
			return http;
		}

		DefaultAuthenticationEventPublisher eventPublisher = objectPostProcessor
				.postProcess(new DefaultAuthenticationEventPublisher());
		localConfigureAuthenticationBldr.authenticationEventPublisher(eventPublisher);

      
		AuthenticationManager authenticationManager = authenticationManager(); // 2.调用下一个方法
      
    // 6. 这里得到的 authenticationManager   ，要么是来源于配置类，要么是来源于用户继承自本类的子类配置
		authenticationBuilder.parentAuthenticationManager(authenticationManager);
		authenticationBuilder.authenticationEventPublisher(eventPublisher);
      // 省略
    }
  
  
  	protected AuthenticationManager authenticationManager() throws Exception {
		if (!authenticationManagerInitialized) {
			configure(localConfigureAuthenticationBldr); // 3.调用下一个方法（注意：此时传入的是临时的（本地的）认证管理器的建造者），如果覆盖了这个方法( configure() )，那么 disableLocalConfigureAuthenticationBldr 就依然为false
      
      
      
			if (disableLocalConfigureAuthenticationBldr) { // 
        
				authenticationManager = authenticationConfiguration
						.getAuthenticationManager();
			}
			else {
				authenticationManager = localConfigureAuthenticationBldr.build(); // 覆盖configure() 之后，就会调用子类对 localConfigureAuthenticationBldr 所做的所有配置来进行构建 AuthenticationManager 。如果没有覆盖 configure() ，则会使用spring boot 的自动配置类
        //此处调用 build() 方法时，会经过 AbstractSecurityBuilder 抽象类的简单调用之后最终执行
        // AuthenticationManagerBuilder.performBuild() 方法，此处就不再粘贴代码了
			}
			authenticationManagerInitialized = true;
		}
		return authenticationManager;
	}
  
  // 这个是最最经常被子配置类覆盖的方法，子类可以对本地的 AuthenticationManagerBuilder 进行配置 AuthenticationProvider 集合
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		this.disableLocalConfigureAuthenticationBldr = true; 
	}
  
  // 8. 子类还必须覆盖此方法，并且调用 父类的本方法，并且标注为 @bean 才能将
  // 子类通过 configure(AuthenticationManagerBuilder auth) 配置好并生成的 ProviderManager 给暴露到 ioc 容器中
  public AuthenticationManager authenticationManagerBean() throws Exception {
		return new AuthenticationManagerDelegator(authenticationBuilder, context);
	}
  
  static final class AuthenticationManagerDelegator implements AuthenticationManager {
    // 9. AuthenticationManagerDelegator 是一个实现了 AuthenticationManager 接口 的内部类
    // 在这里看来其作用只是做了个委派，并且加了个双检索的懒加载
     @Override
    public Authentication authenticate(Authentication authentication)
				throws AuthenticationException {
			if (delegate != null) {
				return delegate.authenticate(authentication);
			}

			synchronized (delegateMonitor) {
				if (delegate == null) {
					delegate = this.delegateBuilder.getObject();
					this.delegateBuilder = null;
				}
			}

			return delegate.authenticate(authentication);
		}
  }
  
}
```

### 4.ProviderManager

![ProviderManager](images/ProviderManager.png)

当有多条`SecurityFilterChain`（过滤器链）时，每个`SecurityFilterChain`中应该都有一个`AuthenticationManager`对象，但是往往他们都有相同的`parentAuthenticationManager`属性，为了让多条过滤器链使用相同的父认证器，即`AuthenticationManager`

![multipleProviderManager](images/multipleProviderManager.png)

```java
package org.springframework.security.authentication;
// 注意 ProviderManager 实现了最开始提到的 AuthenticationManager 接口
public class ProviderManager implements AuthenticationManager, MessageSourceAware,
		InitializingBean {
      
  // providerManager 委派了一系列的 AuthenticationProvider ，每一个 AuthenticationProvider 都有机会来决定认证是否成功，每个 AuthenticationManager 都能对应执行相应的认证（例如：username/password 认证，saml 认证），这样的话就只用暴露一个 ProviderManager 对象，就能进行多种类型的认证    
  private List<AuthenticationProvider> providers = Collections.emptyList();
	private AuthenticationManager parent;
      
  // 5. 调用 ProviderManager 的构造器将 AuthenticationProvider 集合传入    
  public ProviderManager(List<AuthenticationProvider> providers,AuthenticationManager parent) {
    
		Assert.notNull(providers, "providers list cannot be null");
		this.providers = providers;
		this.parent = parent;
		checkState();
	}
      
    // 只不过可以使用 多个 实现了 AuthenticationProvider 接口的实现类来顺序的 对传入的 authentication 对象进行认证
    // 如果 authentication 对象不能认证（没有返回非空 authentication 对象），那么就传递给下一个 AuthenticationProvider 进行认证
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		boolean debug = logger.isDebugEnabled();

		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}

			if (debug) {
				logger.debug("Authentication attempt using "
						+ provider.getClass().getName());
			}

			try {
				result = provider.authenticate(authentication);

				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException e) {
				prepareException(e, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw e;
			} catch (AuthenticationException e) {
				lastException = e;
			}
		}

		if (result == null && parent != null) {
			// Allow the parent to try.
			try {
				result = parentResult = parent.authenticate(authentication);
			}
			catch (ProviderNotFoundException e) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException e) {
				lastException = parentException = e;
			}
		}

		if (result != null) {
			if (eraseCredentialsAfterAuthentication
					&& (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}

			// If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
			if (parentResult == null) {
				eventPublisher.publishAuthenticationSuccess(result);
			}
			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).

		if (lastException == null) {
			lastException = new ProviderNotFoundException(messages.getMessage(
					"ProviderManager.providerNotFound",
					new Object[] { toTest.getName() },
					"No AuthenticationProvider found for {0}"));
		}

		// If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}

		throw lastException;
	}
      
    }
```



> `AuthenticationProvider`的其中一个实现：`DaoAuthenticationProvider`，以模版方法的形式重写了`retrieveUser`方法，在这里调用之前配置的`userDetailsService`来调用数据库进行账号密码认证

```java
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
	

	@Override
	protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		prepareTimingAttackProtection();
		try {
			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
			if (loadedUser == null) {
				throw new InternalAuthenticationServiceException(
						"UserDetailsService returned null, which is an interface contract violation");
			}
			return loadedUser;
		}
		catch (UsernameNotFoundException ex) {
			mitigateAgainstTimingAttack(authentication);
			throw ex;
		}
		catch (InternalAuthenticationServiceException ex) {
			throw ex;
		}
		catch (Exception ex) {
			throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
		}
	}
  
}
```





### 5.WebSecurityConfigurerAdapter  子类的具体实现案例

```java
package com.github.fenixsoft.bookstore.infrastructure.configuration;

/**
 * Spring Security的用户认证服务器配置
 * <p>
 * 借用Spring Security作为认证服务器，告知服务器通过怎样的途径去查询用户、加密密码和验证用户真伪
 * 我们实际上并不使用Spring Security提供的认证表单，而是选择了前端通过OAuth2的密码模式，在授权过程中同时完成认证
 * 由于服务端整套安全机制（方法授权判断、OAuth2密码模式的用户认证、密码的加密算法）仍然是构建在Spring Security基础之上
 * 所以我们的认证服务、用户信息服务仍然继承着Spring Security提供的基类，并在这里注册到Spring Security当中
 *
 * @author icyfenix@gmail.com
 * @date 2020/3/7 19:41
 **/
@Configuration
@EnableWebSecurity
public class AuthenticationServerConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticAccountDetailsService authenticAccountDetailsService;

    @Autowired
    private UsernamePasswordAuthenticationProvider userProvider;

    @Autowired
    private PreAuthenticatedAuthenticationProvider preProvider;

    @Autowired
    private PasswordEncoder encoder;


    /**
     * 需要把AuthenticationManager主动暴漏出来
     * 以便在授权服务器{@link AuthorizationServerConfiguration}中可以使用它来完成用户名、密码的认证
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
     * 配置Spring Security的安全认证服务
     * Spring Security的Web安全设置，将在资源服务器配置{@link ResourceServerConfiguration}中完成
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(authenticAccountDetailsService)
                .passwordEncoder(encoder);
        auth.authenticationProvider(userProvider);
        auth.authenticationProvider(preProvider);
    }
}

```





## 八、spring security 起源



### 1.委派filter代理

> https://docs.spring.io/spring-security/site/docs/current/reference/html5/#samples
>
> 9.2 章节

```java
// 这里使用懒加载的原因是：过滤器必须全都加载好了之后，spring boot 项目才能启动
// 然而，Spring通常使用ContextLoaderListener来加载Spring bean，这在需要注册Filter实例之前不会完成。
public class DelegatingFilterProxy extends GenericFilterBean {

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		// Lazily initialize the delegate if necessary.
		Filter delegateToUse = this.delegate;
    
		if (delegateToUse == null) {
			synchronized (this.delegateMonitor) {
				delegateToUse = this.delegate;
				if (delegateToUse == null) {
					WebApplicationContext wac = findWebApplicationContext();
					if (wac == null) {
						throw new IllegalStateException("No WebApplicationContext found: " +
								"no ContextLoaderListener or DispatcherServlet registered?");
					}
					delegateToUse = initDelegate(wac);
				}
				this.delegate = delegateToUse;
			}
		}

		// Let the delegate perform the actual doFilter operation.
    // 1. 注意：这里的 filterChain 是 tomcat 为我们传递过来的 过滤器链
    // 但是接下来实际是，从ioc中取出来的 delegateToUse 作为根过滤器链，tomcat 传递过来的 filterChain 则只是作为一个参数
		invokeDelegate(delegateToUse, request, response, filterChain);
	}
  
  
  	protected void invokeDelegate(
			Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		// 2. 由此，完成了从 tomcat 过滤器链，到 spring security 的 FilterChainProxy 的切换
		delegate.doFilter(request, response, filterChain);
	}
}
```

### 2.FilterChainProxy

> FilterChainProxy 是spring security 中一种特殊的过滤器
>
> 它可以通过`SecurityFilterChain`来委派许多过滤器实例



总的来说：`FilterChainProxy`持有 `List<SecurityFilterChain>`，其中的每一个`SecurityFilterChain`又持有`List<Filter>`



```java
public interface SecurityFilterChain {

	boolean matches(HttpServletRequest request);

	List<Filter> getFilters();
}
```





```java
public class FilterChainProxy extends GenericFilterBean {

    //  FilterChainProxy 私有的 filterChains 
  // 注意：每一个 filterChains 下面都包含了各自的 filters
  	private List<SecurityFilterChain> filterChains;
  
  	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
    // 是否要清理 此次请求的上下文信息
		boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
		if (clearContext) {
			try {
				request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
				doFilterInternal(request, response, chain);
			}
			finally {
				SecurityContextHolder.clearContext(); // 清理 上下文，防止内存泄漏
				request.removeAttribute(FILTER_APPLIED);
			}
		}
		else {
      // 3. 最终都要挨个的调用 FilterChainProxy 上面的每一个过滤器
			doFilterInternal(request, response, chain);
		}
	}
  
  
  private List<Filter> getFilters(HttpServletRequest request) {
		for (SecurityFilterChain chain : filterChains) {
			if (chain.matches(request)) {
        // 5.返回能够处理 request 的第一个 filterChain 其中包含了一系列的 filter 集合
				return chain.getFilters();
			}
		}

		return null;
	}
  
  
  	private void doFilterInternal(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {

		FirewalledRequest fwRequest = firewall
				.getFirewalledRequest((HttpServletRequest) request);  // 防火墙能保护应用免受特定类型的攻击
		HttpServletResponse fwResponse = firewall
				.getFirewalledResponse((HttpServletResponse) response);

      // 4. 向下调用
		List<Filter> filters = getFilters(fwRequest);

		if (filters == null || filters.size() == 0) {
			if (logger.isDebugEnabled()) {
				logger.debug(UrlUtils.buildRequestUrl(fwRequest)
						+ (filters == null ? " has no matching filters"
								: " has an empty filter list"));
			}

			fwRequest.reset();
			// 6. 如果 FilterChainProxy 中的过滤器链为空，则继续调用 tomcat 的过滤器
			chain.doFilter(fwRequest, fwResponse);

			return;
		}

      // 7. 开始执行 FilterChainProxy 的过滤器链
		VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
		vfc.doFilter(fwRequest, fwResponse);
	}
  
  // 静态内部类
  private static class VirtualFilterChain implements FilterChain {
		private final FilterChain originalChain;
		private final List<Filter> additionalFilters;
		private final FirewalledRequest firewalledRequest;
		private final int size;
		private int currentPosition = 0;

		private VirtualFilterChain(FirewalledRequest firewalledRequest,
				FilterChain chain, List<Filter> additionalFilters) {
			this.originalChain = chain;
			this.additionalFilters = additionalFilters;
			this.size = additionalFilters.size();
			this.firewalledRequest = firewalledRequest;
		}

		@Override
		public void doFilter(ServletRequest request, ServletResponse response)
				throws IOException, ServletException {
			if (currentPosition == size) {
				if (logger.isDebugEnabled()) {
					logger.debug(UrlUtils.buildRequestUrl(firewalledRequest)
							+ " reached end of additional filter chain; proceeding with original chain");
				}

				// Deactivate path stripping as we exit the security filter chain
				this.firewalledRequest.reset();

        // 9. 当 过滤器链中的所有过滤器都调用完成了之后，就调用 tomcat 自带的 filterChain
				originalChain.doFilter(request, response);
			}
			else {
				currentPosition++;

				Filter nextFilter = additionalFilters.get(currentPosition - 1);

				if (logger.isDebugEnabled()) {
					logger.debug(UrlUtils.buildRequestUrl(firewalledRequest)
							+ " at position " + currentPosition + " of " + size
							+ " in additional filter chain; firing Filter: '"
							+ nextFilter.getClass().getSimpleName() + "'");
				}
				// 8. 核心：每一个 filter 处理完成后都会调用 filterChain.doFilter()
        // 因为这里传入的 filterChain 为 this，所以每个 filter 正常通过之后，都会回到本对象
        // 这里也是一个极好的 debug 的开端
				nextFilter.doFilter(request, response, this);
			}
		}
	}

  
}
```

![FilterChainProxy](images/FilterChainProxy.png)



### 3.SecurityFilterChain

`FilterChainProxy`决定哪一条`SecurityFilterChain`应该用来处理此次请求。

`SecurityFilterChain`中的`Security Filters`都是典型的`spring ioc`容器中的对象，但是他们都被注册到了`FilterChainProxy`中，

注意：而不是`DelegatingFilterProxy`中

![SecurityFilterChain](images/SecurityFilterChain.png)

### 4.根据url匹配为每个请求使用不同的过滤器链

![Multiple SecurityFilterChain](images/Multiple SecurityFilterChain.png)

### 5.ExceptionTranslationFilter 专门用于处理认证异常的过滤器

> The [`ExceptionTranslationFilter`](https://docs.spring.io/spring-security/site/docs/5.5.3/api/org/springframework/security/web/access/ExceptionTranslationFilter.html) allows translation of [`AccessDeniedException`](https://docs.spring.io/spring-security/site/docs/5.5.3/api/org/springframework/security/access/AccessDeniedException.html) and [`AuthenticationException`](https://docs.spring.io/spring-security/site/docs/5.5.3/api//org/springframework/security/core/AuthenticationException.html) into HTTP responses.
>
> If the application does not throw an `AccessDeniedException` or an `AuthenticationException`, then `ExceptionTranslationFilter` does not do anything.
>
> 如果其他过滤器不抛出任何的 `AccessDeniedException`或者`AuthenticationException`异常，那么`ExceptionTranslationFilter`将不会做任何处理

![ExceptionTranslationFilter](images/ExceptionTranslationFilter.png)

```java
package org.springframework.security.web.access;


public class ExceptionTranslationFilter extends GenericFilterBean {
  
  private AuthenticationEntryPoint authenticationEntryPoint;

  
  public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		try {
      // 1、首先：ExceptionTranslationFilter 将会调用过滤器链中剩下的过滤器
			chain.doFilter(request, response);

			logger.debug("Chain processed normally");
		}
		catch (IOException ex) {
			throw ex;
		}
		catch (Exception ex) {
			// Try to extract a SpringSecurityException from the stacktrace
			Throwable[] causeChain = throwableAnalyzer.determineCauseChain(ex);
      
      // 2、如果用户没有认证、就能从异常堆栈中找出这个 认证异常
			RuntimeException ase = (AuthenticationException) throwableAnalyzer
					.getFirstThrowableOfType(AuthenticationException.class, causeChain);

			if (ase == null) {
        // 3、如果用户抛出的异常堆栈中含有 AccessDeniedException 异常
        //    然后返回这个 AccessDeniedException 异常
				ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(
						AccessDeniedException.class, causeChain);
			}

			if (ase != null) {
				if (response.isCommitted()) {
					throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", ex);
				}
        // 4、 然后开启认证、拒绝访问相关的处理
				handleSpringSecurityException(request, response, chain, ase);
			}
			else {
				// Rethrow ServletExceptions and RuntimeExceptions as-is
				if (ex instanceof ServletException) {
					throw (ServletException) ex;
				}
				else if (ex instanceof RuntimeException) {
					throw (RuntimeException) ex;
				}

				// Wrap other Exceptions. This shouldn't actually happen
				// as we've already covered all the possibilities for doFilter
				throw new RuntimeException(ex);
			}
		}
	}
  
  
  private void handleSpringSecurityException(HttpServletRequest request,
			HttpServletResponse response, FilterChain chain, RuntimeException exception)
			throws IOException, ServletException {
		if (exception instanceof AuthenticationException) {
      // 5、 认证异常，开始认证
			logger.debug(
					"Authentication exception occurred; redirecting to authentication entry point",
					exception);

			sendStartAuthentication(request, response, chain,
					(AuthenticationException) exception);
		}
		else if (exception instanceof AccessDeniedException) {
			Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
			if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
				logger.debug(
						"Access is denied (user is " + (authenticationTrustResolver.isAnonymous(authentication) ? "anonymous" : "not fully authenticated") + "); redirecting to authentication entry point",
						exception);
				// 5、拒绝访问异常，但是是匿名登陆、记住我登陆，则开始认证
				sendStartAuthentication(
						request,
						response,
						chain,
						new InsufficientAuthenticationException(
							messages.getMessage(
								"ExceptionTranslationFilter.insufficientAuthentication",
								"Full authentication is required to access this resource")));
			}
			else {
				logger.debug(
						"Access is denied (user is not anonymous); delegating to AccessDeniedHandler",
						exception);
				 // 5、拒绝访问异常，抛出异常
				accessDeniedHandler.handle(request, response,
						(AccessDeniedException) exception);
			}
		}
	}
  
  
  	protected void sendStartAuthentication(HttpServletRequest request,
			HttpServletResponse response, FilterChain chain,
			AuthenticationException reason) throws ServletException, IOException {
		// SEC-112: Clear the SecurityContextHolder's Authentication, as the
		// existing Authentication is no longer considered valid
      // 清除安全上下文
		SecurityContextHolder.getContext().setAuthentication(null);
    // 将 HttpServletRequest 保存到 requestCache 中
		requestCache.saveRequest(request, response);
		logger.debug("Calling Authentication entry point.");
      // 6、往下委派调用
		authenticationEntryPoint.commence(request, response, reason);
	}
  
}
```





```java
package org.springframework.security.web;


/**
 * Used by ExceptionTranslationFilter to commence an authentication scheme.
 */
public interface AuthenticationEntryPoint {
	// ~ Methods
	// ========================================================================================================

	/**
	 * Commences an authentication scheme.
	 * <p>
	 * <code>ExceptionTranslationFilter</code> will populate the <code>HttpSession</code>
	 * attribute named
	 * <code>AbstractAuthenticationProcessingFilter.SPRING_SECURITY_SAVED_REQUEST_KEY</code>
	 * with the requested target URL before calling this method.
	 * <p>
	 * Implementations should modify the headers on the <code>ServletResponse</code> as
	 * necessary to commence the authentication process.
	 *
	 * @param request that resulted in an <code>AuthenticationException</code>
	 * @param response so that the user agent can begin authentication
	 * @param authException that caused the invocation
	 *
	 */
	void commence(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException authException) throws IOException, ServletException;
}
```

## 九、认证

![securitycontextholder](images/securitycontextholder.png)

>  Spring Security’s [FilterChainProxy](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filterchainproxy) ensures that the `SecurityContext` is always cleared.
>
> `FilterChainProxy`保证`SecurityContext`总是能够被清理

### 1、避免多线程之间对 `SecurityContext`的竞态条件

```java
SecurityContext context = SecurityContextHolder.createEmptyContext(); 
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER"); 
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context);
```

### 2、Authentication对象

`Authentication`在`spring security`中的两个主要作用：

（1）用于提供在`AuthenticationManager`接口中作为参数传入的用于身份验证的凭据

（2）代表当前已经认证过的用户，可以通过`SecurityContext`中获得

```java
package org.springframework.security.core;

public interface Authentication extends Principal, Serializable {


}
```

```java
package org.springframework.security.authentication;

public abstract class AbstractAuthenticationToken implements Authentication,
		CredentialsContainer {
      
    }
```

其中最常见的一个实现类是：`UsernamePasswordAuthenticationToken`

```java
package org.springframework.security.authentication;

public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {

}
```

### 3、AuthenticationEntryPoint

> 用于响应给浏览器，并要求浏览器发送客户端的身份信息
>
> 如果客户端有未认证的请求，且要访问需要授权的资源，
>
> 该接口的实现类将通过返回一个响应的给客户端，要求客户端发送相应的身份信息
>
> 典型的响应是：
>
> 1、重定向到登陆页
>
> 2、响应` WWW-Authenticate`未授权的http头
>
> 3、如果是前后端分离开发的话，返回一个自定义的响应，前端收到以后，使用`js`跳转到登陆页
>
> ​		其中一个`AuthenticationEntryPoint`的参考实现就是`HttpStatusEntryPoint`

```java
package org.springframework.security.web;

/**
 * Used by {@link ExceptionTranslationFilter} to commence an authentication scheme.
 *
 * @author Ben Alex
 */
public interface AuthenticationEntryPoint {
	// ~ Methods
	// ========================================================================================================

	/**
	 * Commences an authentication scheme.
	 * <p>
	 * <code>ExceptionTranslationFilter</code> will populate the <code>HttpSession</code>
	 * attribute named
	 * <code>AbstractAuthenticationProcessingFilter.SPRING_SECURITY_SAVED_REQUEST_KEY</code>
	 * with the requested target URL before calling this method.
	 * <p>
	 * Implementations should modify the headers on the <code>ServletResponse</code> as
	 * necessary to commence the authentication process.
	 *
	 * @param request that resulted in an <code>AuthenticationException</code>
	 * @param response so that the user agent can begin authentication
	 * @param authException that caused the invocation
	 *
	 */
	void commence(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException authException) throws IOException, ServletException;
}
```

### 4、AbstractAuthenticationProcessingFilter

> `AbstractAuthenticationProcessingFilter`作为一个基本的`Filter`来验证用户的身份信息。
>
> 在执行认证之前，`spring security`一般会通过`AuthenticationEntryPoint`来请求用户的身份信息。
>
> 认证步骤：
>
> （1）执行身份认证的时候，`AbstractAuthenticationProcessingFilter`的子类要从`HttpServletRequest`中提取并封装一个`Authentication`对象用于后续的认证
>
> （2）生成的`Authentication`对象将被`AuthenticationManager`对象用于后续的认证过程
>
> 此过滤器是一切认证的起点。

![abstractauthenticationprocessingfilter](images/abstractauthenticationprocessingfilter.png)

```java
package org.springframework.security.web.authentication;


public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean
		implements ApplicationEventPublisherAware, MessageSourceAware{
  
  
  	private AuthenticationManager authenticationManager;

  	private SessionAuthenticationStrategy sessionStrategy = new NullAuthenticatedSessionStrategy();

	/**
	 * Invokes the
	 * {@link #requiresAuthentication(HttpServletRequest, HttpServletResponse)
	 * requiresAuthentication} method to determine whether the request is for
	 * authentication and should be handled by this filter. If it is an authentication
	 * request, the
	 * {@link #attemptAuthentication(HttpServletRequest, HttpServletResponse)
	 * attemptAuthentication} will be invoked to perform the authentication. There are
	 * then three possible outcomes:
	 * <ol>
	 * <li>An <tt>Authentication</tt> object is returned. The configured
	 * {@link SessionAuthenticationStrategy} will be invoked (to handle any
	 * session-related behaviour such as creating a new session to protect against
	 * session-fixation attacks) followed by the invocation of
	 * {@link #successfulAuthentication(HttpServletRequest, HttpServletResponse, FilterChain, Authentication)}
	 * method</li>
	 * <li>An <tt>AuthenticationException</tt> occurs during authentication. The
	 * {@link #unsuccessfulAuthentication(HttpServletRequest, HttpServletResponse, AuthenticationException)
	 * unsuccessfulAuthentication} method will be invoked</li>
	 * <li>Null is returned, indicating that the authentication process is incomplete. The
	 * method will then return immediately, assuming that the subclass has done any
	 * necessary work (such as redirects) to continue the authentication process. The
	 * assumption is that a later request will be received by this method where the
	 * returned <tt>Authentication</tt> object is not null.
	 * </ol>
	 */
	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);

			return;
		}


		Authentication authResult;

		try {
      // 1、让子类封装 Authentication 对象
			authResult = attemptAuthentication(request, response);
			if (authResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				// authentication
				return;
			}
			sessionStrategy.onAuthentication(authResult, request, response);
		}
		catch (InternalAuthenticationServiceException failed) {
			logger.error(
					"An internal error occurred while trying to authenticate the user.",
					failed);
			unsuccessfulAuthentication(request, response, failed);

			return;
		}
		catch (AuthenticationException failed) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, failed);

			return;
		}

		// Authentication success
		if (continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}
		// 6、登陆成功，这里后续的话是简单的将请求进行重定向到之前配置的登陆成功的页面
		successfulAuthentication(request, response, chain, authResult);
	}

  // 2、 子类必须实现此抽象方法
  public abstract Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException, IOException,
			ServletException;
  
  
  	protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
			Authentication authResult) throws IOException, ServletException {
      
    // 7、设置 Authentication 对象到 SecurityContextHolder 中
    //  之后的 SecurityContextPersistenceFilter 过滤器将会把 SecurityContext 保存在 HttpSession 中
 		SecurityContextHolder.getContext().setAuthentication(authResult);
		if (this.logger.isDebugEnabled()) {
			this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
		}
		this.rememberMeServices.loginSuccess(request, response, authResult);
		if (this.eventPublisher != null) {
			this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
		}
		this.successHandler.onAuthenticationSuccess(request, response, authResult);
	}
  
}
```

#### （1）子类`UsernamePasswordAuthenticationFilter`调用`AuthenticationManager`进行账号密码认证

```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
  
  // 3、子类重写的认证方法
  @Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username : "";
		username = username.trim();
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
    // 4、将请求中的用户名和密码封装位 Authentication 对象
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
    // 5、调用之前提到过的认证管理器进行认证
		return this.getAuthenticationManager().authenticate(authRequest);
	}
  
}
```







```java

package org.springframework.security.web.authentication.session;

/**
 * Allows pluggable support for HttpSession-related behaviour when an authentication
 * occurs.
 * <p>
 * Typical use would be to make sure a session exists or to change the session Id to guard
 * against session-fixation attacks.
 *
 * @author Luke Taylor
 * @since
 */
public interface SessionAuthenticationStrategy {

	/**
	 * Performs Http session-related functionality when a new authentication occurs.
	 *
	 * @throws SessionAuthenticationException if it is decided that the authentication is
	 * not allowed for the session. This will typically be because the user has too many
	 * sessions open at once.
	 */
	void onAuthentication(Authentication authentication, HttpServletRequest request,
			HttpServletResponse response) throws SessionAuthenticationException;

}
```

### 5、用户名密码方式认证

![basicauthenticationentrypoint](images/basicauthenticationentrypoint.png)

```java
package org.springframework.security.web.authentication.www;

public class BasicAuthenticationEntryPoint implements AuthenticationEntryPoint,
		InitializingBean {
      
  public void commence(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException authException) throws IOException {
    // 设置未认证的http响应头
		response.addHeader("WWW-Authenticate", "Basic realm=\"" + realmName + "\"");
		response.sendError(HttpStatus.UNAUTHORIZED.value(), HttpStatus.UNAUTHORIZED.getReasonPhrase());
	}
}
```



> 注意：`ExceptionTranslationFilter`持有一个`AuthenticationEntryPoint`对象，前文已有涉及到，可自行查阅
>
>  `ExceptionTranslationFilter`启动并初始化认证，`AuthenticationEntryPoint` （此处以 `BasicAuthenticationEntryPoint`实现类为例）发回带有`WWW-Authenticate`响应头的http报文



客户端收到该响应之后，应准备相应的用户名与密码，然后开始真正的认证流程

![basicauthenticationfilter](images/basicauthenticationfilter.png)

本图基于`SecurityFilterChain`

#### （1）BasicAuthenticationFilter

> 当用户提交了账号、密码之后，创建了一个`UsernamePasswordAuthenticationToken`

```java
package org.springframework.security.web.authentication.www;

public class BasicAuthenticationFilter extends OncePerRequestFilter {

  	private BasicAuthenticationConverter authenticationConverter = new BasicAuthenticationConverter();


  @Override  // doFilterInternal() 为 OncePerRequestFilter 对 doFilter() 封装之后，流出的模版方法
	protected void doFilterInternal(HttpServletRequest request,
			HttpServletResponse response, FilterChain chain)
					throws IOException, ServletException {
		final boolean debug = this.logger.isDebugEnabled();
		try {
      // 1、创建了一个 Authentication 的实现类 UsernamePasswordAuthenticationToken
      // 其中封装了用户名与密码
			UsernamePasswordAuthenticationToken authRequest = authenticationConverter.convert(request);
			if (authRequest == null) {
				chain.doFilter(request, response);
				return;
			}

			String username = authRequest.getName();



			if (authenticationIsRequired(username)) {
        // 2、UsernamePasswordAuthenticationToken 被交予 authenticationManager 进行相关的认证操作
				Authentication authResult = this.authenticationManager
						.authenticate(authRequest);

				// 3、认证过程没有抛出异常，认证结果被设置到 SecurityContext 中
				SecurityContextHolder.getContext().setAuthentication(authResult);

				this.rememberMeServices.loginSuccess(request, response, authResult);

				onSuccessfulAuthentication(request, response, authResult);
			}

		}
		catch (AuthenticationException failed) {
			SecurityContextHolder.clearContext();

			if (debug) {
				this.logger.debug("Authentication request for failed: " + failed);
			}

			this.rememberMeServices.loginFail(request, response);

			onUnsuccessfulAuthentication(request, response, failed);

			if (this.ignoreFailure) {
				chain.doFilter(request, response);
			}
			else {
				this.authenticationEntryPoint.commence(request, response, failed);
			}

			return;
		}

		chain.doFilter(request, response);
	}
}
```

#### （2）Basic的认证的最小配置

```java
protected void configure(HttpSecurity http) {
	http
		// ...
		.httpBasic(withDefaults());
}
```

![basic_authentication](images/basic_authentication.png)

#### （3）AbstractUserDetailsAuthenticationProvider

![各大认证接口之间的关系](images/各大认证接口之间的关系.png)





![daoauthenticationprovider](images/daoauthenticationprovider.png)

> 步骤：
>
> 1、上一个`Filter`从`HttpServletRequest`中读取了`UsernamePasswordAuthenticationToken`（包含账号名和密码）之后，
>
> 2、最终会通过`ProviderManager`中的其中一个`AuthenticationManager`传递给`DaoAuthenticationProvider`
>
> 3、`DaoAuthenticationProvider`通过`UserDetailsService`查找一个`UserDetails`对象
>
> 4、然后使用`PasswordEncoder`来验证密码





```java
	
public abstract class AbstractUserDetailsAuthenticationProvider implements
		AuthenticationProvider, InitializingBean, MessageSourceAware {

  // 准备将 只含有 username/password 的 authentication 转换为含有 authrities、principal的 authentication 对象
  @Override
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
				() -> messages.getMessage(
						"AbstractUserDetailsAuthenticationProvider.onlySupports",
						"Only UsernamePasswordAuthenticationToken is supported"));

		// Determine username
		String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
				: authentication.getName();

		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);

		if (user == null) {
			cacheWasUsed = false;

			try {
        // 调用本类中的模版方法进行查找用户
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException notFound) {
				logger.debug("User '" + username + "' not found");

				if (hideUserNotFoundExceptions) {
					throw new BadCredentialsException(messages.getMessage(
							"AbstractUserDetailsAuthenticationProvider.badCredentials",
							"Bad credentials"));
				}
				else {
					throw notFound;
				}
			}

			Assert.notNull(user,
					"retrieveUser returned null - a violation of the interface contract");
		}

		try {
			preAuthenticationChecks.check(user);
			additionalAuthenticationChecks(user,
					(UsernamePasswordAuthenticationToken) authentication);
		}
		catch (AuthenticationException exception) {
			if (cacheWasUsed) {
				// There was a problem, so try again after checking
				// we're using latest data (i.e. not from the cache)
				cacheWasUsed = false;
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
				preAuthenticationChecks.check(user);
				additionalAuthenticationChecks(user,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			else {
				throw exception;
			}
		}

		postAuthenticationChecks.check(user);

		if (!cacheWasUsed) {
			this.userCache.putUserInCache(user);
		}

		Object principalToReturn = user;

		if (forcePrincipalAsString) {
			principalToReturn = user.getUsername();
		}

		return createSuccessAuthentication(principalToReturn, authentication, user);
	}
  
  protected abstract UserDetails retrieveUser(String username,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException;
}
```

#### （4）DaoAuthenticationProvider

```java
/**
 * An AuthenticationProvider implementation that retrieves user details from a UserDetailsService.
 * AuthenticationProvider 接口的具体实现类，通过 UserDetailsService 来检索用户详情信息
 */
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
  
  	private UserDetailsService userDetailsService;
  
    protected UserDetailsService getUserDetailsService() {
      return userDetailsService;
    }
  
    @Override
  	protected final UserDetails retrieveUser(String username,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		prepareTimingAttackProtection();
		try {
      // 1、调用 userDetailsService 来加载用户
			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
			if (loadedUser == null) {
				throw new InternalAuthenticationServiceException(
						"UserDetailsService returned null, which is an interface contract violation");
			}
			return loadedUser;
		}
		catch (UsernameNotFoundException ex) {
			mitigateAgainstTimingAttack(authentication);
			throw ex;
		}
		catch (InternalAuthenticationServiceException ex) {
			throw ex;
		}
		catch (Exception ex) {
			throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
		}
	}
}
```

## 十、授权（旧）

#### （1）AccessDecisionManager

> `AccessDecisionManager`决定了这个请求能否调用相应的方法。
>
> 然而这个接口已经不再推荐被使用，官方建议使用`AuthorizationManager`接口
>
> 参考：https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html

```java
package org.springframework.security.access;

public interface AccessDecisionManager {

	/**
	 * Resolves an access control decision for the passed parameters.
	 *
	 * @param authentication the caller invoking the method (not null)
	 * @param object the secured object being called
	 * @param configAttributes the configuration attributes associated with the secured
	 * object being invoked
	 *
	 * @throws AccessDeniedException if access is denied as the authentication does not
	 * hold a required authority or ACL privilege
	 * @throws InsufficientAuthenticationException if access is denied as the
	 * authentication does not provide a sufficient level of trust
	 */
	void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
			InsufficientAuthenticationException;
  
}
```

#### （2）Pre-Invocation Handling

`spring security`使用了`interceptors`来控制对安全对象的访问，例如方法调用或者web请求。`AccessDecisionManager`决定了`Pre-Invocation Handling`的调用是否允许被执行。

#### （3）AbstractSecurityInterceptor

![AbstractSecurityInterceptor](images/AbstractSecurityInterceptor.png)

>步骤：
>
>1、`AbstractSecurityInterceptor`通过调用`AccessDecisionManager`来进行权限控制。
>
>

```java
package org.springframework.security.access.intercept;


public abstract class AbstractSecurityInterceptor implements InitializingBean,
		ApplicationEventPublisherAware, MessageSourceAware {
      
    private AccessDecisionManager accessDecisionManager;
      
      public void afterPropertiesSet() {
        // 一开始就调用 supports() 方法来检查 AccessDecisionManager 是否支持操作特定类型的安全对象（例如：方法安全对象）
      		Assert.isTrue(this.accessDecisionManager.supports(getSecureObjectClass()),
				() -> "AccessDecisionManager does not support secure object class: "
						+ getSecureObjectClass());
    }

    protected InterceptorStatusToken beforeInvocation(Object object) {
      
      // 省略

      Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource()
          .getAttributes(object);

      // 省略

      if (SecurityContextHolder.getContext().getAuthentication() == null) {
        credentialsNotFound(messages.getMessage(
            "AbstractSecurityInterceptor.authenticationNotFound",
            "An Authentication object was not found in the SecurityContext"),
            object, attributes);
      }

      Authentication authenticated = authenticateIfRequired();

      // 1、尝试进行授权。Attempt authorization
      try {
        this.accessDecisionManager.decide(authenticated, object, attributes);
      }
      catch (AccessDeniedException accessDeniedException) {
        publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated,
            accessDeniedException));

        throw accessDeniedException;
      }

      if (debug) {
        logger.debug("Authorization successful");
      }

      if (publishAuthorizationSuccess) {
        publishEvent(new AuthorizedEvent(object, attributes, authenticated));
      }

      // Attempt to run as a different user
      Authentication runAs = this.runAsManager.buildRunAs(authenticated, object,
          attributes);

      if (runAs == null) {
        if (debug) {
          logger.debug("RunAsManager did not change Authentication object");
        }

        // no further work post-invocation
        return new InterceptorStatusToken(SecurityContextHolder.getContext(), false,
            attributes, object);
      }
      else {
        if (debug) {
          logger.debug("Switching to RunAs Authentication: " + runAs);
        }

        SecurityContext origCtx = SecurityContextHolder.getContext();
        SecurityContextHolder.setContext(SecurityContextHolder.createEmptyContext());
        SecurityContextHolder.getContext().setAuthentication(runAs);

        // need to revert to token.Authenticated post-invocation
        return new InterceptorStatusToken(origCtx, true, attributes, object);
      }
	}

}
```

#### （4）AccessDecisionManager

```java
public interface AccessDecisionManager {
	// ~ Methods
	// ========================================================================================================

	/**
	 * Resolves an access control decision for the passed parameters.
	 *
	 * @param authentication the caller invoking the method (not null)
	 * @param object the secured object being called
	          传递安全对象可以检查包含在实际安全对象调用中的参数
	 * @param configAttributes the configuration attributes associated with the secured
	 * object being invoked
	 *
	 * @throws AccessDeniedException if access is denied as the authentication does not
	 * hold a required authority or ACL privilege
	 * @throws InsufficientAuthenticationException if access is denied as the
	 * authentication does not provide a sufficient level of trust
	 */
	void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
			InsufficientAuthenticationException;

	/**
	 * Indicates whether this <code>AccessDecisionManager</code> is able to process
	 * authorization requests presented with the passed <code>ConfigAttribute</code>.
	 * <p>
	 * This allows the <code>AbstractSecurityInterceptor</code> to check every
	 * configuration attribute can be consumed by the configured
	 * <code>AccessDecisionManager</code> and/or <code>RunAsManager</code> and/or
	 * <code>AfterInvocationManager</code>.
	 * </p>
	 *
	 * @param attribute a configuration attribute that has been configured against the
	 * <code>AbstractSecurityInterceptor</code>
	 *
	 * @return true if this <code>AccessDecisionManager</code> can support the passed
	 * configuration attribute
	 */
	boolean supports(ConfigAttribute attribute);

	/**
	 * Indicates whether the <code>AccessDecisionManager</code> implementation is able to
	 * provide access control decisions for the indicated secured object type.
	 *
	 * @param clazz the class that is being queried
	 *
	 * @return <code>true</code> if the implementation can process the indicated class
	 */
	boolean supports(Class<?> clazz);
}
```

`spring security`基于`AccessDecisionManager`的自主实现

![access-decision-voting](images/access-decision-voting.png)

![AffirmativeBased](images/AffirmativeBased.png)

* ConsensusBased：
* AffirmativeBased：只要有一个或以上的`ACCESS_GRANTED`，那么就通过授权
* UnanimousBased：只要有一个或以上的`ACCESS_DENIED`，那么就拒绝访问

#### （5）AccessDecisionVoter

```java
package org.springframework.security.access;

/**
 * Indicates a class is responsible for voting on authorization decisions.
 * <p>
 * The coordination of voting (ie polling {@code AccessDecisionVoter}s, tallying their
 * responses, and making the final authorization decision) is performed by an
 * {@link org.springframework.security.access.AccessDecisionManager}.
 *
 * @author Ben Alex
 */
public interface AccessDecisionVoter<S> {
	// ~ Static fields/initializers
	// =====================================================================================
  // 本接口的 vote() 方法所能返回的结果只能为以下之一
	int ACCESS_GRANTED = 1; // 已授权
	int ACCESS_ABSTAIN = 0; // 未知，交由下一个进行处理
	int ACCESS_DENIED = -1; // 拒绝访问

	// ~ Methods
	// ========================================================================================================

	/**
	 * Indicates whether this {@code AccessDecisionVoter} is able to vote on the passed
	 * {@code ConfigAttribute}.
	 * <p>
	 * This allows the {@code AbstractSecurityInterceptor} to check every configuration
	 * attribute can be consumed by the configured {@code AccessDecisionManager} and/or
	 * {@code RunAsManager} and/or {@code AfterInvocationManager}.
	 *
	 * @param attribute a configuration attribute that has been configured against the
	 * {@code AbstractSecurityInterceptor}
	 *
	 * @return true if this {@code AccessDecisionVoter} can support the passed
	 * configuration attribute
	 */
	boolean supports(ConfigAttribute attribute);

	/**
	 * Indicates whether the {@code AccessDecisionVoter} implementation is able to provide
	 * access control votes for the indicated secured object type.
	 *
	 * @param clazz the class that is being queried
	 *
	 * @return true if the implementation can process the indicated class
	 */
	boolean supports(Class<?> clazz);

	/**
	 * Indicates whether or not access is granted.
	 * <p>
	 * The decision must be affirmative ({@code ACCESS_GRANTED}), negative (
	 * {@code ACCESS_DENIED}) or the {@code AccessDecisionVoter} can abstain (
	 * {@code ACCESS_ABSTAIN}) from voting. Under no circumstances should implementing
	 * classes return any other value. If a weighting of results is desired, this should
	 * be handled in a custom
	 * {@link org.springframework.security.access.AccessDecisionManager} instead.
	 * <p>
	 * Unless an {@code AccessDecisionVoter} is specifically intended to vote on an access
	 * control decision due to a passed method invocation or configuration attribute
	 * parameter, it must return {@code ACCESS_ABSTAIN}. This prevents the coordinating
	 * {@code AccessDecisionManager} from counting votes from those
	 * {@code AccessDecisionVoter}s without a legitimate interest in the access control
	 * decision.
	 * <p>
	 * Whilst the secured object (such as a {@code MethodInvocation}) is passed as a
	 * parameter to maximise flexibility in making access control decisions, implementing
	 * classes should not modify it or cause the represented invocation to take place (for
	 * example, by calling {@code MethodInvocation.proceed()}).
	 *
	 * @param authentication the caller making the invocation
	 * @param object the secured object being invoked
	 * @param attributes the configuration attributes associated with the secured object
	 *
	 * @return either {@link #ACCESS_GRANTED}, {@link #ACCESS_ABSTAIN} or
	 * {@link #ACCESS_DENIED}
	 */
	int vote(Authentication authentication, S object,
			Collection<ConfigAttribute> attributes);
}


```

#### （6）After Invocation Handling

虽然 AccessDecisionManager 在继续安全对象调用之前由 AbstractSecurityInterceptor 调用，但一些应用程序需要一种修改安全对象调用实际返回的对象的方法。 虽然您可以轻松实现自己的 AOP 关注点来实现这一点，但 Spring Security 提供了一个方便的钩子，它具有几个与其 ACL 功能集成的具体实现。

![after-invocation](images/after-invocation.png)

#### （7）FilterSecurityInterceptor

> `FilterSecurityInterceptor`为`HttpServletRequest`提供授权操作，它在`FilterChainProxy`之中

![filtersecurityinterceptor_流程](images/filtersecurityinterceptor_流程.png)



![filtersecurityinterceptor](images/filtersecurityinterceptor.png)

```java
package org.springframework.security.web.access.intercept;

public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements
		Filter {
  
  
  
	/**
	 * Method that is actually called by the filter chain. Simply delegates to the
	 * {@link #invoke(FilterInvocation)} method.
	 *
	 * @param request the servlet request
	 * @param response the servlet response
	 * @param chain the filter chain
	 *
	 * @throws IOException if the filter chain fails
	 * @throws ServletException if the filter chain fails
	 */
  @Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
    // 1、创建 FilterInvocation 对象
		FilterInvocation fi = new FilterInvocation(request, response, chain);
    // 2、委派过滤器链
		invoke(fi);
	}
  
  public void invoke(FilterInvocation fi) throws IOException, ServletException {
    // 3、进入实例方法
		if ((fi.getRequest() != null)
				&& (fi.getRequest().getAttribute(FILTER_APPLIED) != null)
				&& observeOncePerRequest) {
			// filter already applied to this request and user wants us to observe
			// once-per-request handling, so don't re-do security checking
			fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
		}
		else {
			// first time this request being called, so perform security checking
			if (fi.getRequest() != null && observeOncePerRequest) {
				fi.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
			}
      // 4、调用了父类 AbstractSecurityInterceptor（前文有记录） 来 获取 SecurityContext 
      // 5、父类还 获得 ConfigAttribute 并将其传入 AccessDecisionManager
			InterceptorStatusToken token = super.beforeInvocation(fi);

			try {
        // 如果授权成功，则会继续调用过滤器链
				fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
			}
      // 如果授权失败，将调用`ExceptionTranslationFilter`处理`AccessDeniedException`
			finally {
				super.finallyInvocation(token);
			}

			super.afterInvocation(token, null);
		}
	}

  
}
```



```java
package org.springframework.security.web.access.intercept;


public class FilterInvocation {
	// ~ Static fields
	// ==================================================================================================
	static final FilterChain DUMMY_CHAIN = (req, res) -> {
		throw new UnsupportedOperationException("Dummy filter chain");
	};

	// ~ Instance fields
	// ================================================================================================

	private FilterChain chain;
	private HttpServletRequest request;
	private HttpServletResponse response;

	// ~ Constructors
	// ===================================================================================================
  // 2、 进入构造器
	public FilterInvocation(ServletRequest request, ServletResponse response,
			FilterChain chain) {
		if ((request == null) || (response == null) || (chain == null)) {
			throw new IllegalArgumentException("Cannot pass null values to constructor");
		}

		this.request = (HttpServletRequest) request;
		this.response = (HttpServletResponse) response;
		this.chain = chain;
	}
  
}
```

#### （8）默认配置要求每一个请求授权之前都需要认证

```java
protected void configure(HttpSecurity http) throws Exception {
	http
		// ...
		.authorizeHttpRequests(authorize -> authorize
			.anyRequest().authenticated()
		);
}
```

## 十一、授权（新）

### 1、AuthorizationManager 授权管理器

```java
public interface AuthorizationManager<T> {

	/**
	 * Determines if access should be granted for a specific authentication and object.
	 * @param authentication the {@link Supplier} of the {@link Authentication} to check
	 * @param object the {@link T} object to check
	 * @throws AccessDeniedException if access is not granted
	 */
	default void verify(Supplier<Authentication> authentication, T object) {
		AuthorizationDecision decision = check(authentication, object);
		if (decision != null && !decision.isGranted()) {
			throw new AccessDeniedException("Access Denied");
		}
	}

	/**
	 * Determines if access is granted for a specific authentication and object.
	 * @param authentication the {@link Supplier} of the {@link Authentication} to check
	 * @param object the {@link T} object to check
	 * @return an {@link AuthorizationDecision} or null if no decision could be made
	 */
	@Nullable
	AuthorizationDecision check(Supplier<Authentication> authentication, T object);

}
```

### 2、AuthorizationFilter 授权过滤器

> `AuthorizationFilter`调用 `AuthorizationManager`进行授权

```java
public class AuthorizationFilter extends OncePerRequestFilter {

	private final AuthorizationManager<HttpServletRequest> authorizationManager;

	/**
	 * Creates an instance.
	 * @param authorizationManager the {@link AuthorizationManager} to use
	 */
	public AuthorizationFilter(AuthorizationManager<HttpServletRequest> authorizationManager) {
		Assert.notNull(authorizationManager, "authorizationManager cannot be null");
		this.authorizationManager = authorizationManager;
	}

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		this.authorizationManager.verify(this::getAuthentication, request);
		filterChain.doFilter(request, response);
	}

	private Authentication getAuthentication() {
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		if (authentication == null) {
			throw new AuthenticationCredentialsNotFoundException(
					"An Authentication object was not found in the SecurityContext");
		}
		return authentication;
	}

}
```

### 3、RequestMatcherDelegatingAuthorizationManager

> `RequestMatcherDelegatingAuthorizationManager`是`AuthorizationManager`的一个实现。
>
> `spring security`委派`RequestMatcherDelegatingAuthorizationManager`来协调多个`AuthorizationManager`合作进行授权。
>
> `RequestMatcherDelegatingAuthorizationManager`将找出最匹配此次请求的`AuthorizationManager`实现来进行本次授权
>
> 对于方法层面的安全，可以使用`AuthorizationManagerBeforeMethodInterceptor`和`AuthorizationManagerAfterMethodInterceptor`

```java
public final class RequestMatcherDelegatingAuthorizationManager implements AuthorizationManager<HttpServletRequest> {

  
}
```

### 4、el 表达式中的根对象 SecurityExpressionRoot

> 更多用法，请参照官方文档：https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html

```java
    /**
     * spring security el 表达式中 根对象是
     * org.springframework.security.access.expression.SecurityExpressionRoot 
     * 该对象中的方法都可以直接使用 
     * @ref https://docs.spring.io/spring-security/reference/servlet/authorization/expression-based.html
     * @return
     */
    @PreAuthorize("hasAnyAuthority('coding')")
    @GetMapping(PATH_1)
    @ResponseBody
    public String coding(){
        return "有写代码的权利："+atomicInteger.getAndIncrement();
    }
```

el表达式高级应用

```java
import org.springframework.security.access.method.P;

...

@PreAuthorize("#c.name == authentication.name")
public void doSomething(@P("c") Contact contact);
```



```java
import org.springframework.data.repository.query.Param;

...

@PreAuthorize("#n == authentication.name")
Contact findContactByName(@Param("n") String name);
```

```java
@PreAuthorize("#contact.name == authentication.name")
public void doSomething(Contact contact);
```

