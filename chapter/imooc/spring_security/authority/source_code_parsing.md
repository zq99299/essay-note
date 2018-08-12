# Spring Security源码解析
![](/assets/image/imooc/spring_secunity/snipaste_20180812_173639.png)

spring security的基本原理之前讲解过了。这章主要看后面两个：

* FilterSecurityInterceptor

  决定该用户是否有权限访问指定的资源
* ExceptionTranslationFilter

  异常处理，如果不能访问，则处理FilterSecurityInterceptor中抛出的异常

## AnonymousAuthenticationFilter
AnonymousAuthenticationFilter ： 匿名过滤器，位置固定，前面所有的都走完之后，会经过该过滤器

该过滤器之前自己也是备受折磨，特别是在调试oath2的时候，在没有正确开始basic登录的时候，
就一直在抱怨，为什么会走这个呢？

看看最主要的源码：
```java
public AnonymousAuthenticationFilter(String key) {
  // 固定了用户信息和角色
  this(key, "anonymousUser", AuthorityUtils.createAuthorityList("ROLE_ANONYMOUS"));
}

public AnonymousAuthenticationFilter(String key, Object principal,
    List<GrantedAuthority> authorities) {
  Assert.hasLength(key, "key cannot be null or empty");
  Assert.notNull(principal, "Anonymous authentication principal must be set");
  Assert.notNull(authorities, "Anonymous authorities must be set");
  this.key = key;
  this.principal = principal;  // 用户信息不是一个对象，是一个字符串
  this.authorities = authorities;
}

public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
    throws IOException, ServletException {

  // 如果之前没有身份认证
  // 则创建一个 AnonymousAuthenticationToken
  // 也就是说到达 FilterSecurityInterceptor 中的时候一定有一个身份信息
  if (SecurityContextHolder.getContext().getAuthentication() == null) {
    SecurityContextHolder.getContext().setAuthentication(
        createAuthentication((HttpServletRequest) req));

    if (logger.isDebugEnabled()) {
      logger.debug("Populated SecurityContextHolder with anonymous token: '"
          + SecurityContextHolder.getContext().getAuthentication() + "'");
    }
  }
  else {
    if (logger.isDebugEnabled()) {
      logger.debug("SecurityContextHolder not populated with anonymous token, as it already contained: '"
          + SecurityContextHolder.getContext().getAuthentication() + "'");
    }
  }

  chain.doFilter(req, res);
}

protected Authentication createAuthentication(HttpServletRequest request) {
  AnonymousAuthenticationToken auth = new AnonymousAuthenticationToken(key,
      principal, authorities);
  auth.setDetails(authenticationDetailsSource.buildDetails(request));

  return auth;
}
```


## FilterSecurityInterceptor
> 看源码技巧：找准一个关键入口，然后分析调用关系和类图

![](/assets/image/imooc/spring_secunity/snipaste_20180812_175032.png)

* FilterSecurityInterceptor
* AccessDecisionManager 管理什么呢？AccessDecisionVoter ：投票者

  就是用来一系列的判定，是否可以进行访问什么的
  * AffirmativeBased 只要有一票否决则不能访问 （默认使用）

      在未登录的时候用的是只要有一个拒绝则不能访问，在登录后，只要有一个允许则放行。。有点懵逼
  * ConsensusBased  通过/不通过 哪一个票数多就遵循哪一个
  * UnanimousBased  只要有一票否决则不允许访问
* AccessDecisionVoter 投票者 spring3-,由一堆实现类来解决
  * WebExpressionVoter spring3+ 由该类全部包办，它投过就过，不过就不过
* -------- 以上是三个核心的类---------
* SecurityConfig   所有的配置信息
* ConfigAttribute  对应了每一个url的配置信息
* SecurityContextHolder
* Authentication 用户身份信息，用户拥有的权限信息

主流程：拿到 配置信息，用户身份信息，请求信息 然后给投票者进行投票；最后根据策略进行决定是否放行

从两个流程进行分析：

1. 未登录访问被拦截
2. 登录后访问

### 未登录访问被拦截
访问： http://localhost:8080/user/1

```java
public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements
		Filter {

  public void doFilter(ServletRequest request, ServletResponse response,
    FilterChain chain) throws IOException, ServletException {
    // 拿到请求和响应封装成一个 invocation(调用)
    FilterInvocation fi = new FilterInvocation(request, response, chain);
    invoke(fi);
  }
  public void invoke(FilterInvocation fi) throws IOException, ServletException {
    // 保证每次请求只被检查一次，依据就是 request中的 FILTER_APPLIED
		if ((fi.getRequest() != null)
				&& (fi.getRequest().getAttribute(FILTER_APPLIED) != null)
				&& observeOncePerRequest) {
			// filter already applied to this request and user wants us to observe
			// once-per-request handling, so don't re-do security checking
			fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
		}
		else {
      // 如果是第一次经过该过滤器，则设置标记
			// first time this request being called, so perform security checking
			if (fi.getRequest() != null && observeOncePerRequest) {
				fi.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
			}
      // 权限的校验
			InterceptorStatusToken token = super.beforeInvocation(fi);

      // 如果校验通过则是执行真正的服务了
			try {
				fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
			}
			finally {
				super.finallyInvocation(token);
			}

			super.afterInvocation(token, null);
		}
	}
}

// 抽象父类
public abstract class AbstractSecurityInterceptor implements InitializingBean,
		ApplicationEventPublisherAware, MessageSourceAware {
      protected InterceptorStatusToken beforeInvocation(Object object) {
    		Assert.notNull(object, "Object was null");
    		final boolean debug = logger.isDebugEnabled();

    		if (!getSecureObjectClass().isAssignableFrom(object.getClass())) {
    			throw new IllegalArgumentException(
    					"Security invocation attempted for object "
    							+ object.getClass().getName()
    							+ " but AbstractSecurityInterceptor only configured to support secure objects of type: "
    							+ getSecureObjectClass());
    		}

        // 获取配置信息中对访问的url匹配的权限
        // 这里会返回 hasRole('ROLE_ADMIN') ，也就是之前配置的admin角色
    		Collection<ConfigAttribute> attributes = this.obtainSecurityMetadataSource()
    				.getAttributes(object);

    		if (attributes == null || attributes.isEmpty()) {
    			if (rejectPublicInvocations) {
    				throw new IllegalArgumentException(
    						"Secure object invocation "
    								+ object
    								+ " was denied as public invocations are not allowed via this interceptor. "
    								+ "This indicates a configuration error because the "
    								+ "rejectPublicInvocations property is set to 'true'");
    			}

    			if (debug) {
    				logger.debug("Public object - authentication not attempted");
    			}

    			publishEvent(new PublicInvocationEvent(object));

    			return null; // no further work post-invocation
    		}

    		if (debug) {
    			logger.debug("Secure object: " + object + "; Attributes: " + attributes);
    		}

    		if (SecurityContextHolder.getContext().getAuthentication() == null) {
    			credentialsNotFound(messages.getMessage(
    					"AbstractSecurityInterceptor.authenticationNotFound",
    					"An Authentication object was not found in the SecurityContext"),
    					object, attributes);
    		}

        // 获取身份信息
    		Authentication authenticated = authenticateIfRequired();

    		// Attempt authorization 尝试授权，也就是投票
    		try {
          //  accessDecisionManager 的实现是 AffirmativeBased，里面只有一个 WebExpressionVoter
          // 如果没有通过就会抛出异常
    			this.accessDecisionManager.decide(authenticated, object, attributes);
    		}
    		catch (AccessDeniedException accessDeniedException) {
    			publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated,
    					accessDeniedException));
          // 在这里异常会被 org.springframework.security.web.access.ExceptionTranslationFilter#doFilter 捕获到
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


// 默认投票策略
public class AffirmativeBased extends AbstractAccessDecisionManager {

	public AffirmativeBased(List<AccessDecisionVoter<? extends Object>> decisionVoters) {
		super(decisionVoters);
	}

	public void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
		int deny = 0;

		for (AccessDecisionVoter voter : getDecisionVoters()) {
      // WebExpressionVoter 去投票
			int result = voter.vote(authentication, object, configAttributes);

			if (logger.isDebugEnabled()) {
				logger.debug("Voter: " + voter + ", returned: " + result);
			}

			switch (result) {
        // 如果 是允许访问，则立即返回，也就是只要有一个同意则放行
  			case AccessDecisionVoter.ACCESS_GRANTED:
  				return;
        // 如果返回的是拒绝则计数
  			case AccessDecisionVoter.ACCESS_DENIED:
  				deny++;

  				break;

  			default:
  				break;
  			}
		}
    // 如果没有允许访问，则判定是否是被拒绝了。
    // 被拒绝则异常给调用者
		if (deny > 0) {
			throw new AccessDeniedException(messages.getMessage(
					"AbstractAccessDecisionManager.accessDenied", "Access is denied"));
		}

		// To get this far, every AccessDecisionVoter abstained
		checkAllowIfAllAbstainDecisions();
	}
}


public class ExceptionTranslationFilter extends GenericFilterBean {

  	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
  			throws IOException, ServletException {
  		HttpServletRequest request = (HttpServletRequest) req;
  		HttpServletResponse response = (HttpServletResponse) res;

  		try {
  			chain.doFilter(request, response);

  			logger.debug("Chain processed normally");
  		}
  		catch (IOException ex) {
  			throw ex;
  		}
  		catch (Exception ex) {
  			// Try to extract a SpringSecurityException from the stacktrace
  			Throwable[] causeChain = throwableAnalyzer.determineCauseChain(ex);
        // 获取到异常链中，第一次抛出的是否是身份异常信息
  			RuntimeException ase = (AuthenticationException) throwableAnalyzer
  					.getFirstThrowableOfType(AuthenticationException.class, causeChain);

  			if (ase == null) {
          // 这一次流程是未登录访问
          // 所以是被拒绝访问的异常
  				ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(
  						AccessDeniedException.class, causeChain);
  			}

  			if (ase != null) {
          // 检测响应是否已经提交
          // 提交过得响应已经写了 http状态码和响应头
  				if (response.isCommitted()) {
  					throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", ex);
  				}
          // 处理安全异常
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
          // 如果是 Anonymous 身份信息，又是拒绝访问异常
          // 则发送，并开始授权
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

  				accessDeniedHandler.handle(request, response,
  						(AccessDeniedException) exception);
  			}
  		}
  	}

    protected void sendStartAuthentication(HttpServletRequest request,
      HttpServletResponse response, FilterChain chain,
      AuthenticationException reason) throws ServletException, IOException {
      SecurityContextHolder.getContext().setAuthentication(null);
      requestCache.saveRequest(request, response);
      logger.debug("Calling Authentication entry point.");
      // org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint#commence
      // 该类是我们之前配置的  loginFormUrl = /authentication/require 的处理器端点
      // 该端点会获取到 之前配置的地址然后使用
      // redirectStrategy.sendRedirect(request, response, redirectUrl); 进行跳转
      // 该访问之后就完成了拒绝并引导授权的功能
      authenticationEntryPoint.commence(request, response, reason);
  }
}
```
### 登录后访问

该源码就不记录了。篇幅太多了；
