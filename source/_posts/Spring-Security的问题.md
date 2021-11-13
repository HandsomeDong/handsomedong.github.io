---
title: Spring Security CSRF防御
date: 2019-09-08 22:02:57
tags:
categories: 
    - 工作中遇到的问题
---
最近在公司开发一个用Spring Boot框架的项目，用POST上传文件的时候死活上传不了，一直都是返回403，忙活了一个上午才发现是因为Spring Sercurity开启了CSRF防御，所以在POST请求中必须还要包含增加一个字段及数据才能请求成功，不然的话都会被Spring Sercurity给拦截掉并且认为是非法请求，比如我要提交表单数据，必须要增加一个隐藏的字段：
```
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
```
详情可以在官方文档中看到，https://docs.spring.io/autorepo/docs/spring-security/3.2.0.CI-SNAPSHOT/reference/html/csrf.html


### 什么是CSRF攻击
看了一下官方文档才了解到这CSRF到底是什么，就像文档所说的，假如银行网站向用户提供转账功能，转账请求的代码是这样的：

```
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid; Domain=bank.example.com; Secure; HttpOnly
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876
```

但是假如你登录到了银行网站，然后在还没注销的情况下又访问了一个恶意网站，这时你看到恶意网站上有个【领取奖品】按钮，你美滋滋地点击了这个按钮，却不知道这个按钮会提交这样的表单信息：

```
<form action="https://bank.example.com/transfer" method="post">
  <input type="hidden"
      name="amount"
      value="100.00"/>
  <input type="hidden"
      name="routingNumber"
      value="evilsRoutingNumber"/>
  <input type="hidden"
      name="account"
      value="evilsAccountNumber"/>
  <input type="submit"
      value="Win Money!"/>
</form>
```

最后你会收到一条信息，您向银行卡号XXX转账100元……当场流下悔恨的泪水，大喊三声“为什么”？！
因为恶意网站虽然没办法获取你的cookie信息，但是它向银行后端请求的时候，浏览器还是会将cookie信息一起发送的，银行当然不知道这个请求是恶意网站向它发送的，它只知道cookie是你的！

**像这种伪装成当前已登录认证用户去请求正在访问的网站后端的攻击方式，就被称为CSRF。**

所以解决办法就是增加一些表单信息，而这些信息是恶意网站所无法获取的！例如图片验证码什么的。但是除了登录注册的时候使用验证码外，其它时候也使用验证码的话，用户岂不是会觉得很烦，所以Spring Sercurity通过在WEB应用中增加前端过滤器，验证请求是否包含CSRF的token信息，不包含的自然就是非法请求咯，因为攻击网站是无法获取到token信息的，只要是跨域提交的信息，都是无法通过这个过滤器的校验的。

### 源码解析
下面是CsrfFilter中的doFilterInternal源码。
```
@Override
	protected void doFilterInternal(HttpServletRequest request,
			HttpServletResponse response, FilterChain filterChain)
					throws ServletException, IOException {
		request.setAttribute(HttpServletResponse.class.getName(), response);
		// 先从tokenRepository中加载token
		CsrfToken csrfToken = this.tokenRepository.loadToken(request);
		final boolean missingToken = csrfToken == null;
		  // 如果为空，则tokenRepository生成新的token，并保存到tokenRepository中
		if (missingToken) {
			csrfToken = this.tokenRepository.generateToken(request);
			this.tokenRepository.saveToken(csrfToken, request, response);
		}
        // 将token写入request的attribute中，方便页面上使用
		request.setAttribute(CsrfToken.class.getName(), csrfToken);
		request.setAttribute(csrfToken.getParameterName(), csrfToken);
		//这个macher就是我们在Spring配置文件中自定义的过滤器，也就是GET，HEAD, TRACE, OPTIONS和我们的rest都不处理
 
        // 这个macher就是我们在Spring配置文件中自定义的过滤器，
		//如果不需要csrf验证的请求，则直接下传请求（requireCsrfProtectionMatcher是默认的对象，对符合^(GET|HEAD|TRACE|OPTIONS)$的请求和我们自定义的请求不验证）
		if (!this.requireCsrfProtectionMatcher.matches(request)) {
			filterChain.doFilter(request, response);
			return;
		}
        // 从用户请求中获取token信息
		String actualToken = request.getHeader(csrfToken.getHeaderName());
		if (actualToken == null) {
			actualToken = request.getParameter(csrfToken.getParameterName());
		}
        // 验证，如果相同，则下传请求，如果不同，则抛出异常
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
```
唉，有空还是得多到Spring官网看看文档了解了解框架啊，不然一个简单的问题就得搞半天！


