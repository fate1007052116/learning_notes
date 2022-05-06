# Spring Security

## 一、本质

`spring security`本质上是一个过滤器链

### 1.`FilterSecurityInterceptor`

方法级权限过滤器，基本位于过滤器的最底部

```java
public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements Filter {
 
  	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		FilterInvocation fi = new FilterInvocation(request, response, chain);
		invoke(fi);
	}
  
  	public void invoke(FilterInvocation fi) throws IOException, ServletException {
      			// 省略
						// 调用本过滤器之前的过滤器      
      			InterceptorStatusToken token = super.beforeInvocation(fi);

      			try {
              fi.getChain().doFilter(fi.getRequest(), fi.getResponse()); // 表真正的调用后台服务
            }
            finally {
              super.finallyInvocation(token);
            }

            super.afterInvocation(token, null);
    }
}
```

### 2.`ExceptionTranslationFilter`

> 异常过滤器：处理在认证和授权中抛出的异常

```java
public class ExceptionTranslationFilter extends GenericFilterBean {
  
}
```

### 3.`UsernamePasswordAuthenticationFilter`

> 对`/login`的`post`请求做拦截，校验表单中的用户名，密码

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

## 二、过滤器加载原理

### 1.使用spring security配置过滤器

`DelegatingFilterProxy`

```java
public class DelegatingFilterProxy extends GenericFilterBean {
  
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
					delegateToUse = initDelegate(wac); // 初始化
				}
				this.delegate = delegateToUse;
			}
		}

		// Let the delegate perform the actual doFilter operation.
		invokeDelegate(delegateToUse, request, response, filterChain);
	}
  
  
  protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
		String targetBeanName = getTargetBeanName(); // Debug 时，targetBeanName = springSecurityFilterChain
		Assert.state(targetBeanName != null, "No target bean name set");
		Filter delegate = wac.getBean(targetBeanName, Filter.class);
		if (isTargetFilterLifecycle()) {
			delegate.init(getFilterConfig());
		}
		return delegate;
	}
}
```

### 2.

```java
public class FilterChainProxy extends GenericFilterBean {
 	
  	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
		if (clearContext) {
			try {
				request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
				doFilterInternal(request, response, chain);
			}
			finally {
				SecurityContextHolder.clearContext();
				request.removeAttribute(FILTER_APPLIED);
			}
		}
		else {
			doFilterInternal(request, response, chain);
		}
	}
  
  
  	private void doFilterInternal(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {

		FirewalledRequest fwRequest = firewall
				.getFirewalledRequest((HttpServletRequest) request);
		HttpServletResponse fwResponse = firewall
				.getFirewalledResponse((HttpServletResponse) response);

		List<Filter> filters = getFilters(fwRequest); // 调用下面的方法获取所有的过滤器并加载到过滤链中

		if (filters == null || filters.size() == 0) {
			if (logger.isDebugEnabled()) {
				logger.debug(UrlUtils.buildRequestUrl(fwRequest)
						+ (filters == null ? " has no matching filters"
								: " has an empty filter list"));
			}

			fwRequest.reset();

			chain.doFilter(fwRequest, fwResponse);

			return;
		}

		VirtualFilterChain vfc = new VirtualFilterChain(fwRequest, chain, filters);
		vfc.doFilter(fwRequest, fwResponse);
	}
  
  
  	/**
	 * Returns the first filter chain matching the supplied URL.
	 *
	 * @param request the request to match
	 * @return an ordered array of Filters defining the filter chain
	 */
	private List<Filter> getFilters(HttpServletRequest request) {
		for (SecurityFilterChain chain : filterChains) {
			if (chain.matches(request)) {
				return chain.getFilters();
			}
		}

		return null;
	}
  
}
```

