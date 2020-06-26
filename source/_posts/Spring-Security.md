---
title: Spring Security 学习历程
date: 2020-06-26 12:08:25
tags:
    - Spring
    - Java
categories: Spring
---


&emsp;&emsp;Spring Security是一个灵活和强大的身份验证和访问控制框架，以确保基于Spring的Java Web应用程序的安全。Spring Security是一个轻量级的安全框架，它确保基于Spring的应用程序提供身份验证和授权支持。它与Spring MVC有很好地集成，并配备了流行的安全算法实现捆绑在一起。

<!-- more -->


# 集成 Spring Security

&emsp;&emsp;这一部分内容记录从零开始搭建一个`Spring Security`项目，使用Spring默认的表单登录。

&emsp;&emsp;`Spring Security`的三大模块简单介绍，三大模块都实现了`SecurityBuilder`接口：

> `AuthenticationManagerBuilder`：用于创建认证管理器；
> `WebSecurity`：web安全控制，过滤器链配置等；
> `HttpSecurity`：请求认证权限配置等。

## HttpSecurity

&emsp;&emsp;

## 项目搭建，基本配置

### 环境准备

&emsp;&emsp;如下图，创建一个基于`Spring Boot`的项目，。

![创建](/Spring-Security/create_project.png)

&emsp;&emsp;在IDE中打开项目，使用maven打包，为了方便我们阅读源码，建议使用maven下载源码。

![下载源码](/Spring-Security/download_source_code.png)

&emsp;&emsp;引入插件`actuator`，方便后续web端访问。

- 依赖

```bash
<!--actuator依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 配置

```bash
# actuator 配置参数
management:
  endpoint:
    shutdown:
      enabled: false   # 禁用接口关闭 Spring Boot
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: "*"        # 打开所有的监控点
        exclude: shutdown   # 禁用关机服务
      base-path: /actuator  # 定制监控点路径，默认为 /actuator
# 定制 actuator 信息
info:
  app:
    name: ferry
    author: ferry
    desciption: 人生而自由，却无往不在枷锁之中
```

### Security 配置

&emsp;&emsp;`WebSecurityConfigurerAdapter`是一个安全配置适配器，我们一般通过继承这个适配器完成对Spring Security的配置。这个适配器有三个重要方法，`configure(AuthenticationManagerBuilder auth)`,`configure(WebSecurity web)`,`configure(HttpSecurity http)`，通过子类对这三个方法的覆盖可以完成对三大模块的个性化配置。

&emsp;&emsp;我们现在的目的是使用`Spring Security`完成权限认证，访问actuator的时候需要先登录再访问，基于这样的需求，也就是说我们需要三大模块中的`HttpSecurity`帮助我们。查看并覆盖适配器中的`configure(HttpSecurity http)`方法，我们可以发现适配器中的这个方法很简单，就是说所有的请求都需要进行权限认证，开启表单登录功能。

![适配器中的方法](/Spring-Security/http_security_configurer.png)

&emsp;&emsp;我们覆盖这个方法，然后调用父类的这个方法，看效果怎么样。

```java
package cn.org.ferry.security.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * <p>安全配置类
 *
 * @author ferry ferry_sy@163.com
 * created by 2020/06/26 12:28
 */

@EnableWebSecurity
@Slf4j
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }
}
```

&emsp;&emsp;访问`localhost:8080/actuator`会跳转到登录页面，Spring Security 默认登录页面是/login，默认的用户名是user，密码是随机生成的，输出在日志中。输入用户名密码认证成功跳转到/actuator。到这里，最简单的Spring Security就配置好了。

### 表单登录历程分析

&emsp;&emsp;这一小节看一下访问swagger跳转登录，登录完成和认证具体那些组件发挥了作用。

&emsp;&emsp;首先看一下network，访问swagger怎么就跳转到登录页面了。

![登录跳转](/Spring-Security/actuator_redirect_login.png)

&emsp;&emsp;可以看到，actuator被重定向到了login。那么接下来分析一下DEBUG日志。

```bash
2020-06-26 14:43:59.108 DEBUG 63277 --- [8080-Acceptor-0] o.apache.tomcat.util.threads.LimitLatch  : Counting up[http-nio-8080-Acceptor-0] latch=1
2020-06-26 14:43:59.109 DEBUG 63277 --- [8080-Acceptor-0] o.apache.tomcat.util.threads.LimitLatch  : Counting up[http-nio-8080-Acceptor-0] latch=2
2020-06-26 14:43:59.109 DEBUG 63277 --- [nio-8080-exec-3] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read from buffer: [0]
2020-06-26 14:43:59.110 DEBUG 63277 --- [nio-8080-exec-3] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read direct from socket: [639]
2020-06-26 14:43:59.110 DEBUG 63277 --- [nio-8080-exec-3] o.a.coyote.http11.Http11InputBuffer      : Received [GET /actuator HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: JSESSIONID=CF8FA325ADBABCB906FEFBF2624055F6

]
2020-06-26 14:43:59.110 DEBUG 63277 --- [nio-8080-exec-3] o.a.t.util.http.Rfc6265CookieProcessor   : Cookies: Parsing b[]: JSESSIONID=CF8FA325ADBABCB906FEFBF2624055F6
2020-06-26 14:43:59.110 DEBUG 63277 --- [nio-8080-exec-3] o.a.catalina.connector.CoyoteAdapter     :  Requested cookie session id is CF8FA325ADBABCB906FEFBF2624055F6
2020-06-26 14:43:59.113 DEBUG 63277 --- [nio-8080-exec-3] o.a.c.authenticator.AuthenticatorBase    : Security checking request GET /actuator
2020-06-26 14:43:59.113 DEBUG 63277 --- [nio-8080-exec-3] org.apache.catalina.realm.RealmBase      :   No applicable constraints defined
2020-06-26 14:43:59.113 DEBUG 63277 --- [nio-8080-exec-3] o.a.c.authenticator.AuthenticatorBase    :  Not subject to any constraint
2020-06-26 14:43:59.113 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 1 of 15 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 2 of 15 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] w.c.HttpSessionSecurityContextRepository : HttpSession returned null object for SPRING_SECURITY_CONTEXT
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] w.c.HttpSessionSecurityContextRepository : No SecurityContext was available from the HttpSession: org.apache.catalina.session.StandardSessionFacade@218ce5da. A new one will be created.
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 3 of 15 in additional filter chain; firing Filter: 'HeaderWriterFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 4 of 15 in additional filter chain; firing Filter: 'CsrfFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 5 of 15 in additional filter chain; firing Filter: 'LogoutFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request 'GET /actuator' doesn't match 'POST /logout'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 6 of 15 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request 'GET /actuator' doesn't match 'POST /login'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 7 of 15 in additional filter chain; firing Filter: 'DefaultLoginPageGeneratingFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 8 of 15 in additional filter chain; firing Filter: 'DefaultLogoutPageGeneratingFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.AntPathRequestMatcher  : Checking match of request : '/actuator'; against '/logout'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 9 of 15 in additional filter chain; firing Filter: 'BasicAuthenticationFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 10 of 15 in additional filter chain; firing Filter: 'RequestCacheAwareFilter'
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : pathInfo: both null (property equals)
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : queryString: both null (property equals)
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : requestURI: arg1=/actuator; arg2=/actuator (property equals)
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : serverPort: arg1=8080; arg2=8080 (property equals)
2020-06-26 14:43:59.114 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : requestURL: arg1=http://localhost:8080/actuator; arg2=http://localhost:8080/actuator (property equals)
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : scheme: arg1=http; arg2=http (property equals)
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : serverName: arg1=localhost; arg2=localhost (property equals)
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : contextPath: arg1=; arg2= (property equals)
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.DefaultSavedRequest            : servletPath: arg1=/actuator; arg2=/actuator (property equals)
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.HttpSessionRequestCache        : Removing DefaultSavedRequest from session if present
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 11 of 15 in additional filter chain; firing Filter: 'SecurityContextHolderAwareRequestFilter'
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 12 of 15 in additional filter chain; firing Filter: 'AnonymousAuthenticationFilter'
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.a.AnonymousAuthenticationFilter  : Populated SecurityContextHolder with anonymous token: 'org.springframework.security.authentication.AnonymousAuthenticationToken@8be58c07: Principal: anonymousUser; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffc7f0c: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: CF8FA325ADBABCB906FEFBF2624055F6; Granted Authorities: ROLE_ANONYMOUS'
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 13 of 15 in additional filter chain; firing Filter: 'SessionManagementFilter'
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 14 of 15 in additional filter chain; firing Filter: 'ExceptionTranslationFilter'
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.security.web.FilterChainProxy        : /actuator at position 15 of 15 in additional filter chain; firing Filter: 'FilterSecurityInterceptor'
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.a.i.FilterSecurityInterceptor    : Secure object: FilterInvocation: URL: /actuator; Attributes: [authenticated]
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.a.i.FilterSecurityInterceptor    : Previously Authenticated: org.springframework.security.authentication.AnonymousAuthenticationToken@8be58c07: Principal: anonymousUser; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffc7f0c: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: CF8FA325ADBABCB906FEFBF2624055F6; Granted Authorities: ROLE_ANONYMOUS
2020-06-26 14:43:59.115 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.access.vote.AffirmativeBased       : Voter: org.springframework.security.web.access.expression.WebExpressionVoter@35470409, returned: -1
2020-06-26 14:43:59.116 DEBUG 63277 --- [nio-8080-exec-3] o.s.b.a.audit.listener.AuditListener     : AuditEvent [timestamp=2020-06-26T06:43:59.115Z, principal=anonymousUser, type=AUTHORIZATION_FAILURE, data={details=org.springframework.security.web.authentication.WebAuthenticationDetails@fffc7f0c: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: CF8FA325ADBABCB906FEFBF2624055F6, type=org.springframework.security.access.AccessDeniedException, message=Access is denied}]
2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.a.ExceptionTranslationFilter     : Access is denied (user is anonymous); redirecting to authentication entry point

org.springframework.security.access.AccessDeniedException: Access is denied
	at org.springframework.security.access.vote.AffirmativeBased.decide(AffirmativeBased.java:84) ~[spring-security-core-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.access.intercept.AbstractSecurityInterceptor.beforeInvocation(AbstractSecurityInterceptor.java:233) ~[spring-security-core-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityInterceptor.java:124) ~[spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityInterceptor.java:91) ~[spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:119) ~[spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:137) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthenticationFilter.java:111) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContextHolderAwareRequestFilter.java:170) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.java:63) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.authentication.www.BasicAuthenticationFilter.doFilterInternal(BasicAuthenticationFilter.java:158) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter.doFilterInternal(DefaultLogoutPageGeneratingFilter.java:52) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter.doFilter(DefaultLoginPageGeneratingFilter.java:206) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:200) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:116) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.csrf.CsrfFilter.doFilterInternal(CsrfFilter.java:100) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:74) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:105) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(WebAsyncManagerIntegrationFilter.java:56) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:215) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:178) [spring-security-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:357) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:270) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:92) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:117) [spring-boot-actuator-2.1.1.RELEASE.jar:2.1.1.RELEASE]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:106) [spring-boot-actuator-2.1.1.RELEASE.jar:2.1.1.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) [spring-web-5.1.3.RELEASE.jar:5.1.3.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:199) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.StandardContextValve.__invoke(StandardContextValve.java:96) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:40002) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:490) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:408) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:791) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1417) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_181]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_181]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.13.jar:9.0.13]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_181]

2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : Trying to match using Ant [pattern='/**', GET]
2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request '/actuator' matched by universal pattern '/**'
2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : Trying to match using NegatedRequestMatcher [requestMatcher=Ant [pattern='/**/favicon.*']]
2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.AntPathRequestMatcher  : Checking match of request : '/actuator'; against '/**/favicon.*'
2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.NegatedRequestMatcher  : matches = true
2020-06-26 14:43:59.117 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : Trying to match using NegatedRequestMatcher [requestMatcher=MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@79ae6b56, matchingMediaTypes=[application/json], useEquals=false, ignoredMediaTypes=[*/*]]]
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : httpRequestMediaTypes=[text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8]
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing text/html
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/json .isCompatibleWith text/html = false
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing application/xhtml+xml
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/json .isCompatibleWith application/xhtml+xml = false
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing image/webp
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/json .isCompatibleWith image/webp = false
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing image/apng
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/json .isCompatibleWith image/apng = false
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing application/xml;q=0.9
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/json .isCompatibleWith application/xml;q=0.9 = false
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing application/signed-exchange;v=b3;q=0.9
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/json .isCompatibleWith application/signed-exchange;v=b3;q=0.9 = false
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing */*;q=0.8
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Ignoring
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Did not match any media types
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.NegatedRequestMatcher  : matches = true
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : Trying to match using NegatedRequestMatcher [requestMatcher=RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]]
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.NegatedRequestMatcher  : matches = true
2020-06-26 14:43:59.118 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : All requestMatchers returned true
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] org.apache.tomcat.util.http.Parameters   : Set encoding to UTF-8
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.s.HttpSessionRequestCache        : DefaultSavedRequest added to Session: DefaultSavedRequest[http://localhost:8080/actuator]
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.a.ExceptionTranslationFilter     : Calling Authentication entry point.
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] s.w.a.DelegatingAuthenticationEntryPoint : Trying to match using AndRequestMatcher [requestMatchers=[NegatedRequestMatcher [requestMatcher=RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]], MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@79ae6b56, matchingMediaTypes=[application/xhtml+xml, image/*, text/html, text/plain], useEquals=false, ignoredMediaTypes=[*/*]]]]
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : Trying to match using NegatedRequestMatcher [requestMatcher=RequestHeaderRequestMatcher [expectedHeaderName=X-Requested-With, expectedHeaderValue=XMLHttpRequest]]
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.matcher.NegatedRequestMatcher  : matches = true
2020-06-26 14:43:59.119 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : Trying to match using MediaTypeRequestMatcher [contentNegotiationStrategy=org.springframework.web.accept.ContentNegotiationManager@79ae6b56, matchingMediaTypes=[application/xhtml+xml, image/*, text/html, text/plain], useEquals=false, ignoredMediaTypes=[*/*]]
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : httpRequestMediaTypes=[text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8]
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : Processing text/html
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : application/xhtml+xml .isCompatibleWith text/html = false
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : image/* .isCompatibleWith text/html = false
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.u.m.MediaTypeRequestMatcher      : text/html .isCompatibleWith text/html = true
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.util.matcher.AndRequestMatcher   : All requestMatchers returned true
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] s.w.a.DelegatingAuthenticationEntryPoint : Match found! Executing org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint@606d6d2c
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.web.DefaultRedirectStrategy        : Redirecting to 'http://localhost:8080/login'
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] o.s.s.w.header.writers.HstsHeaderWriter  : Not injecting HSTS header since it did not match the requestMatcher org.springframework.security.web.header.writers.HstsHeaderWriter$SecureRequestMatcher@66afd03c
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] w.c.HttpSessionSecurityContextRepository : SecurityContext is empty or contents are anonymous - context will not be stored in HttpSession.
2020-06-26 14:43:59.120 DEBUG 63277 --- [nio-8080-exec-3] s.s.w.c.SecurityContextPersistenceFilter : SecurityContextHolder now cleared, as request processing completed
2020-06-26 14:43:59.124 DEBUG 63277 --- [nio-8080-exec-3] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read from buffer: [0]
2020-06-26 14:43:59.124 DEBUG 63277 --- [nio-8080-exec-3] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read direct from socket: [0]
2020-06-26 14:43:59.124 DEBUG 63277 --- [nio-8080-exec-3] o.apache.coyote.http11.Http11Processor   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Status in: [OPEN_READ], State out: [OPEN]
2020-06-26 14:43:59.127 DEBUG 63277 --- [nio-8080-exec-4] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read from buffer: [0]
2020-06-26 14:43:59.127 DEBUG 63277 --- [nio-8080-exec-4] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read direct from socket: [636]
2020-06-26 14:43:59.127 DEBUG 63277 --- [nio-8080-exec-4] o.a.coyote.http11.Http11InputBuffer      : Received [GET /login HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: JSESSIONID=CF8FA325ADBABCB906FEFBF2624055F6

]
2020-06-26 14:43:59.127 DEBUG 63277 --- [nio-8080-exec-4] o.a.t.util.http.Rfc6265CookieProcessor   : Cookies: Parsing b[]: JSESSIONID=CF8FA325ADBABCB906FEFBF2624055F6
2020-06-26 14:43:59.127 DEBUG 63277 --- [nio-8080-exec-4] o.a.catalina.connector.CoyoteAdapter     :  Requested cookie session id is CF8FA325ADBABCB906FEFBF2624055F6
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] o.a.c.authenticator.AuthenticatorBase    : Security checking request GET /login
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] org.apache.catalina.realm.RealmBase      :   No applicable constraints defined
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] o.a.c.authenticator.AuthenticatorBase    :  Not subject to any constraint
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 1 of 15 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 2 of 15 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] w.c.HttpSessionSecurityContextRepository : HttpSession returned null object for SPRING_SECURITY_CONTEXT
2020-06-26 14:43:59.128 DEBUG 63277 --- [nio-8080-exec-4] w.c.HttpSessionSecurityContextRepository : No SecurityContext was available from the HttpSession: org.apache.catalina.session.StandardSessionFacade@218ce5da. A new one will be created.
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 3 of 15 in additional filter chain; firing Filter: 'HeaderWriterFilter'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 4 of 15 in additional filter chain; firing Filter: 'CsrfFilter'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 5 of 15 in additional filter chain; firing Filter: 'LogoutFilter'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request 'GET /login' doesn't match 'POST /logout'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 6 of 15 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request 'GET /login' doesn't match 'POST /login'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 7 of 15 in additional filter chain; firing Filter: 'DefaultLoginPageGeneratingFilter'
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] o.s.s.w.header.writers.HstsHeaderWriter  : Not injecting HSTS header since it did not match the requestMatcher org.springframework.security.web.header.writers.HstsHeaderWriter$SecureRequestMatcher@66afd03c
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] w.c.HttpSessionSecurityContextRepository : SecurityContext is empty or contents are anonymous - context will not be stored in HttpSession.
2020-06-26 14:43:59.129 DEBUG 63277 --- [nio-8080-exec-4] s.s.w.c.SecurityContextPersistenceFilter : SecurityContextHolder now cleared, as request processing completed
2020-06-26 14:43:59.130 DEBUG 63277 --- [nio-8080-exec-4] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read from buffer: [0]
2020-06-26 14:43:59.130 DEBUG 63277 --- [nio-8080-exec-4] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Read direct from socket: [0]
2020-06-26 14:43:59.130 DEBUG 63277 --- [nio-8080-exec-4] o.apache.coyote.http11.Http11Processor   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@2dfe7cd0:org.apache.tomcat.util.net.NioChannel@364c81d6:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:61047]], Status in: [OPEN_READ], State out: [OPEN]
2020-06-26 14:44:09.867 DEBUG 63277 --- [Engine[Tomcat]]] org.apache.catalina.session.ManagerBase  : Start expire sessions StandardManager at 1593153849867 sessioncount 1
2020-06-26 14:44:09.867 DEBUG 63277 --- [Engine[Tomcat]]] org.apache.catalina.session.ManagerBase  : End expire sessions StandardManager processingTime 0 expired sessions: 0

```

&emsp;&emsp;请求/actuator进来，经过Spring Security的默认的15个过滤器走一遍，发现没有权限，此时抛出403异常。异常被过滤器链中的`ExceptionTranslationFilter`捕获，这个过滤器是专门处理Spring Security抛出的异常。看他的核心方法：

```java
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
        RuntimeException ase = (AuthenticationException) throwableAnalyzer
                .getFirstThrowableOfType(AuthenticationException.class, causeChain);

        if (ase == null) {
            ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(
                    AccessDeniedException.class, causeChain);
        }

        if (ase != null) {
            if (response.isCommitted()) {
                throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", ex);
            }
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
    // SEC-112: Clear the SecurityContextHolder's Authentication, as the
    // existing Authentication is no longer considered valid
    SecurityContextHolder.getContext().setAuthentication(null);
    requestCache.saveRequest(request, response);
    logger.debug("Calling Authentication entry point.");
    authenticationEntryPoint.commence(request, response, reason);
}
```

&emsp;&emsp;从doFilter方法中不难发现，如果过滤器链在执行过程中抛出IO异常那么直接抛出，如果是认证异常或者鉴权异常那么就走`handleSpringSecurityException`方法。在`handleSpringSecurityException`方法中调用了`sendStartAuthentication`方法，最终走`AuthenticationEntryPoint`处理异常。在这里，为了跟踪actuator重定向到login问题，暂且告诉大家这里的`AuthenticationEntryPoint`默认用的是`DelegatingAuthenticationEntryPoint`实现的。这是一个委托类，核心方法是：

```java
public void commence(HttpServletRequest request, HttpServletResponse response,
        AuthenticationException authException) throws IOException, ServletException {

    for (RequestMatcher requestMatcher : entryPoints.keySet()) {
        if (logger.isDebugEnabled()) {
            logger.debug("Trying to match using " + requestMatcher);
        }
        if (requestMatcher.matches(request)) {
            AuthenticationEntryPoint entryPoint = entryPoints.get(requestMatcher);
            if (logger.isDebugEnabled()) {
                logger.debug("Match found! Executing " + entryPoint);
            }
            entryPoint.commence(request, response, authException);
            return;
        }
    }

    if (logger.isDebugEnabled()) {
        logger.debug("No match found. Using default entry point " + defaultEntryPoint);
    }

    // No EntryPoint matched, use defaultEntryPoint
    defaultEntryPoint.commence(request, response, authException);
}
```

&emsp;&emsp;这里暂且不管这个组件里面的属性具体有什么，从这个方法来看，它是匹配请求，如果请求匹配到了，那么就执行实际的`AuthenticationEntryPoint.commence`方法，否则走默认的`AuthenticationEntryPoint.commence`方法。从日志"All requestMatchers returned true.Match found! Executing org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint@606d6d2c"可以发现，请求匹配到了，走了`LoginUrlAuthenticationEntryPoint`类，这个类就是负责进行登录重定向的。


&emsp;&emsp;此时已经重定向到了login页面，浏览器发起get请求，此时针对login请求又要走一遍过滤器链。看日志，走到了第7个过滤器`DefaultLoginPageGeneratingFilter`就没有下一个了，那么打开这个类看一下他做了什么。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;

    boolean loginError = isErrorPage(request);
    boolean logoutSuccess = isLogoutSuccess(request);
    if (isLoginUrlRequest(request) || loginError || logoutSuccess) {
        String loginPageHtml = generateLoginPageHtml(request, loginError,
                logoutSuccess);
        response.setContentType("text/html;charset=UTF-8");
        response.setContentLength(loginPageHtml.getBytes(StandardCharsets.UTF_8).length);
        response.getWriter().write(loginPageHtml);

        return;
    }

    chain.doFilter(request, response);
}
```

&emsp;&emsp;通过它的doFilter方法发现，这个过滤器其实就是进行登录操作相关页面（登录页面，登录成功页面，登录失败页面）渲染输出的。显然，在这个过滤器中这个login请求和登录uri(/login)匹配上了，所以直接通过response响应给客户端了，也就没后面的过滤器啥事了。


&emsp;&emsp;登录成功又是怎么跳转到actuator的呢？看debug日志

```java
2020-06-26 15:45:59.055 DEBUG 64997 --- [nio-8080-exec-3] o.apache.coyote.http11.Http11Processor   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@6a2a0d95:org.apache.tomcat.util.net.NioChannel@64edff78:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52502]], Status in: [OPEN_READ], State out: [CLOSED]
2020-06-26 15:45:59.055 DEBUG 64997 --- [nio-8080-exec-4] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read from buffer: [0]
2020-06-26 15:45:59.056 DEBUG 64997 --- [nio-8080-exec-4] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read direct from socket: [885]
2020-06-26 15:45:59.056 DEBUG 64997 --- [nio-8080-exec-4] o.a.coyote.http11.Http11InputBuffer      : Received [POST /login HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 102
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
Origin: http://localhost:8080
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost:8080/login
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: JSESSIONID=A3FF3ECA223F77D88488B770D5C9DF2A

username=user&password=c760af91-087d-447c-ba5e-0fcb6f354547&_csrf=08b4d22f-4ce8-44fe-896b-39fab4a2542d]
2020-06-26 15:45:59.056 DEBUG 64997 --- [nio-8080-exec-3] o.apache.tomcat.util.threads.LimitLatch  : Counting down[http-nio-8080-exec-3] latch=3
2020-06-26 15:45:59.057 DEBUG 64997 --- [nio-8080-exec-3] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioChannel@64edff78:java.nio.channels.SocketChannel[closed]] closed
2020-06-26 15:45:59.089 DEBUG 64997 --- [nio-8080-exec-4] o.a.t.util.http.Rfc6265CookieProcessor   : Cookies: Parsing b[]: JSESSIONID=A3FF3ECA223F77D88488B770D5C9DF2A
2020-06-26 15:45:59.089 DEBUG 64997 --- [nio-8080-exec-4] o.a.catalina.connector.CoyoteAdapter     :  Requested cookie session id is A3FF3ECA223F77D88488B770D5C9DF2A
2020-06-26 15:45:59.091 DEBUG 64997 --- [nio-8080-exec-4] o.a.c.authenticator.AuthenticatorBase    : Security checking request POST /login
2020-06-26 15:45:59.091 DEBUG 64997 --- [nio-8080-exec-4] org.apache.catalina.realm.RealmBase      :   No applicable constraints defined
2020-06-26 15:45:59.091 DEBUG 64997 --- [nio-8080-exec-4] o.a.c.authenticator.AuthenticatorBase    :  Not subject to any constraint
2020-06-26 15:45:59.091 DEBUG 64997 --- [nio-8080-exec-4] org.apache.tomcat.util.http.Parameters   : Set encoding to UTF-8
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] org.apache.tomcat.util.http.Parameters   : Start processing with input [username=user&password=c760af91-087d-447c-ba5e-0fcb6f354547&_csrf=08b4d22f-4ce8-44fe-896b-39fab4a2542d]
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 1 of 15 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 2 of 15 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] w.c.HttpSessionSecurityContextRepository : HttpSession returned null object for SPRING_SECURITY_CONTEXT
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] w.c.HttpSessionSecurityContextRepository : No SecurityContext was available from the HttpSession: org.apache.catalina.session.StandardSessionFacade@90c6235. A new one will be created.
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 3 of 15 in additional filter chain; firing Filter: 'HeaderWriterFilter'
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 4 of 15 in additional filter chain; firing Filter: 'CsrfFilter'
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 5 of 15 in additional filter chain; firing Filter: 'LogoutFilter'
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.s.w.u.matcher.AntPathRequestMatcher  : Checking match of request : '/login'; against '/logout'
2020-06-26 15:45:59.092 DEBUG 64997 --- [nio-8080-exec-4] o.s.security.web.FilterChainProxy        : /login at position 6 of 15 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
2020-06-26 15:45:59.093 DEBUG 64997 --- [nio-8080-exec-4] o.s.s.w.u.matcher.AntPathRequestMatcher  : Checking match of request : '/login'; against '/login'
2020-06-26 15:45:59.093 DEBUG 64997 --- [nio-8080-exec-4] w.a.UsernamePasswordAuthenticationFilter : Request is to process authentication
2020-06-26 15:45:59.093 DEBUG 64997 --- [nio-8080-exec-4] o.s.s.authentication.ProviderManager     : Authentication attempt using org.springframework.security.authentication.dao.DaoAuthenticationProvider
2020-06-26 15:45:59.250 DEBUG 64997 --- [nio-8080-exec-4] o.s.b.a.audit.listener.AuditListener     : AuditEvent [timestamp=2020-06-26T07:45:59.250Z, principal=user, type=AUTHENTICATION_SUCCESS, data={details=org.springframework.security.web.authentication.WebAuthenticationDetails@fffd3270: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: A3FF3ECA223F77D88488B770D5C9DF2A}]
2020-06-26 15:45:59.250 DEBUG 64997 --- [nio-8080-exec-4] s.CompositeSessionAuthenticationStrategy : Delegating to org.springframework.security.web.authentication.session.ChangeSessionIdAuthenticationStrategy@7956f93a
2020-06-26 15:45:59.251 DEBUG 64997 --- [nio-8080-exec-4] s.CompositeSessionAuthenticationStrategy : Delegating to org.springframework.security.web.csrf.CsrfAuthenticationStrategy@45a3d5ee
2020-06-26 15:45:59.251 DEBUG 64997 --- [nio-8080-exec-4] w.a.UsernamePasswordAuthenticationFilter : Authentication success. Updating SecurityContextHolder to contain: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@34267f: Principal: org.springframework.security.core.userdetails.User@36ebcb: Username: user; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Not granted any authorities; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffd3270: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: A3FF3ECA223F77D88488B770D5C9DF2A; Not granted any authorities
2020-06-26 15:45:59.252 DEBUG 64997 --- [nio-8080-exec-4] RequestAwareAuthenticationSuccessHandler : Redirecting to DefaultSavedRequest Url: http://localhost:8080/actuator
2020-06-26 15:45:59.252 DEBUG 64997 --- [nio-8080-exec-4] o.s.s.web.DefaultRedirectStrategy        : Redirecting to 'http://localhost:8080/actuator'
2020-06-26 15:45:59.252 DEBUG 64997 --- [nio-8080-exec-4] o.s.s.w.header.writers.HstsHeaderWriter  : Not injecting HSTS header since it did not match the requestMatcher org.springframework.security.web.header.writers.HstsHeaderWriter$SecureRequestMatcher@5120d812
2020-06-26 15:45:59.252 DEBUG 64997 --- [nio-8080-exec-4] w.c.HttpSessionSecurityContextRepository : SecurityContext 'org.springframework.security.core.context.SecurityContextImpl@34267f: Authentication: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@34267f: Principal: org.springframework.security.core.userdetails.User@36ebcb: Username: user; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Not granted any authorities; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffd3270: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: A3FF3ECA223F77D88488B770D5C9DF2A; Not granted any authorities' stored to HttpSession: 'org.apache.catalina.session.StandardSessionFacade@90c6235
2020-06-26 15:45:59.252 DEBUG 64997 --- [nio-8080-exec-4] s.s.w.c.SecurityContextPersistenceFilter : SecurityContextHolder now cleared, as request processing completed
2020-06-26 15:45:59.253 DEBUG 64997 --- [nio-8080-exec-4] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read from buffer: [0]
2020-06-26 15:45:59.253 DEBUG 64997 --- [nio-8080-exec-4] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read direct from socket: [0]
2020-06-26 15:45:59.253 DEBUG 64997 --- [nio-8080-exec-4] o.apache.coyote.http11.Http11Processor   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Status in: [OPEN_READ], State out: [OPEN]
2020-06-26 15:45:59.255 DEBUG 64997 --- [nio-8080-exec-5] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read from buffer: [0]
2020-06-26 15:45:59.255 DEBUG 64997 --- [nio-8080-exec-5] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read direct from socket: [684]
2020-06-26 15:45:59.255 DEBUG 64997 --- [nio-8080-exec-5] o.a.coyote.http11.Http11InputBuffer      : Received [GET /actuator HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost:8080/login
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: JSESSIONID=148448546D54BBBDA932481E1EE5FBD5

]
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] o.a.t.util.http.Rfc6265CookieProcessor   : Cookies: Parsing b[]: JSESSIONID=148448546D54BBBDA932481E1EE5FBD5
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] o.a.catalina.connector.CoyoteAdapter     :  Requested cookie session id is 148448546D54BBBDA932481E1EE5FBD5
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] o.a.c.authenticator.AuthenticatorBase    : Security checking request GET /actuator
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] org.apache.catalina.realm.RealmBase      :   No applicable constraints defined
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] o.a.c.authenticator.AuthenticatorBase    :  Not subject to any constraint
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 1 of 15 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 2 of 15 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
2020-06-26 15:45:59.256 DEBUG 64997 --- [nio-8080-exec-5] w.c.HttpSessionSecurityContextRepository : Obtained a valid SecurityContext from SPRING_SECURITY_CONTEXT: 'org.springframework.security.core.context.SecurityContextImpl@34267f: Authentication: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@34267f: Principal: org.springframework.security.core.userdetails.User@36ebcb: Username: user; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Not granted any authorities; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffd3270: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: A3FF3ECA223F77D88488B770D5C9DF2A; Not granted any authorities'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 3 of 15 in additional filter chain; firing Filter: 'HeaderWriterFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 4 of 15 in additional filter chain; firing Filter: 'CsrfFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 5 of 15 in additional filter chain; firing Filter: 'LogoutFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request 'GET /actuator' doesn't match 'POST /logout'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 6 of 15 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.u.matcher.AntPathRequestMatcher  : Request 'GET /actuator' doesn't match 'POST /login'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 7 of 15 in additional filter chain; firing Filter: 'DefaultLoginPageGeneratingFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 8 of 15 in additional filter chain; firing Filter: 'DefaultLogoutPageGeneratingFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.u.matcher.AntPathRequestMatcher  : Checking match of request : '/actuator'; against '/logout'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 9 of 15 in additional filter chain; firing Filter: 'BasicAuthenticationFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 10 of 15 in additional filter chain; firing Filter: 'RequestCacheAwareFilter'
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : pathInfo: both null (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : queryString: both null (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : requestURI: arg1=/actuator; arg2=/actuator (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : serverPort: arg1=8080; arg2=8080 (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : requestURL: arg1=http://localhost:8080/actuator; arg2=http://localhost:8080/actuator (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : scheme: arg1=http; arg2=http (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : serverName: arg1=localhost; arg2=localhost (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : contextPath: arg1=; arg2= (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.DefaultSavedRequest            : servletPath: arg1=/actuator; arg2=/actuator (property equals)
2020-06-26 15:45:59.257 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.s.HttpSessionRequestCache        : Removing DefaultSavedRequest from session if present
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 11 of 15 in additional filter chain; firing Filter: 'SecurityContextHolderAwareRequestFilter'
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 12 of 15 in additional filter chain; firing Filter: 'AnonymousAuthenticationFilter'
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.a.AnonymousAuthenticationFilter  : SecurityContextHolder not populated with anonymous token, as it already contained: 'org.springframework.security.authentication.UsernamePasswordAuthenticationToken@34267f: Principal: org.springframework.security.core.userdetails.User@36ebcb: Username: user; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Not granted any authorities; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffd3270: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: A3FF3ECA223F77D88488B770D5C9DF2A; Not granted any authorities'
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 13 of 15 in additional filter chain; firing Filter: 'SessionManagementFilter'
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 14 of 15 in additional filter chain; firing Filter: 'ExceptionTranslationFilter'
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator at position 15 of 15 in additional filter chain; firing Filter: 'FilterSecurityInterceptor'
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.a.i.FilterSecurityInterceptor    : Secure object: FilterInvocation: URL: /actuator; Attributes: [authenticated]
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.a.i.FilterSecurityInterceptor    : Previously Authenticated: org.springframework.security.authentication.UsernamePasswordAuthenticationToken@34267f: Principal: org.springframework.security.core.userdetails.User@36ebcb: Username: user; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Not granted any authorities; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffd3270: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: A3FF3ECA223F77D88488B770D5C9DF2A; Not granted any authorities
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.access.vote.AffirmativeBased       : Voter: org.springframework.security.web.access.expression.WebExpressionVoter@13c20521, returned: 1
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.a.i.FilterSecurityInterceptor    : Authorization successful
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.a.i.FilterSecurityInterceptor    : RunAsManager did not change Authentication object
2020-06-26 15:45:59.258 DEBUG 64997 --- [nio-8080-exec-5] o.s.security.web.FilterChainProxy        : /actuator reached end of additional filter chain; proceeding with original chain
2020-06-26 15:45:59.263 DEBUG 64997 --- [nio-8080-exec-5] org.apache.tomcat.util.http.Parameters   : Set encoding to UTF-8
2020-06-26 15:45:59.263 DEBUG 64997 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : GET "/actuator", parameters={}
2020-06-26 15:45:59.268 DEBUG 64997 --- [nio-8080-exec-5] s.b.a.e.w.s.WebMvcEndpointHandlerMapping : Mapped to Actuator root web endpoint
2020-06-26 15:45:59.293 DEBUG 64997 --- [nio-8080-exec-5] m.m.a.RequestResponseBodyMethodProcessor : Using 'application/vnd.spring-boot.actuator.v2+json;q=0.8', given [text/html, application/xhtml+xml, image/webp, image/apng, application/xml;q=0.9, application/signed-exchange;v=b3;q=0.9, */*;q=0.8] and supported [application/vnd.spring-boot.actuator.v2+json, application/json]
2020-06-26 15:45:59.297 DEBUG 64997 --- [nio-8080-exec-5] m.m.a.RequestResponseBodyMethodProcessor : Writing [{_links={self=[Link@14e0a8bf href = 'http://localhost:8080/actuator'], configprops=[Link@2c464248 href = 'http://localhost:8080/actuator/configprops'], env=[Link@4da1881f href = 'http://localhost:8080/actuator/env'], env-toMatch=[Link@604e6f14 href = 'http://localhost:8080/actuator/env/{toMatch}'], auditevents=[Link@71fa96f2 href = 'http://localhost:8080/actuator/auditevents'], beans=[Link@6f30ce87 href = 'http://localhost:8080/actuator/beans'], info=[Link@2e90420e href = 'http://localhost:8080/actuator/info'], conditions=[Link@3db5ae07 href = 'http://localhost:8080/actuator/conditions'], health=[Link@185df245 href = 'http://localhost:8080/actuator/health'], health-component=[Link@7170433f href = 'http://localhost:8080/actuator/health/{component}'], health-component-instance=[Link@5ddcff2d href = 'http://localhost:8080/actuator/health/{component}/{instance}'], httptrace=[Link@4d09a707 href = 'http://localhost:8080/actuator/httptrace'], threaddump=[Link@3a31bb39 href = 'http://localhost:8080/actuator/threaddump'], loggers=[Link@49ab7efb href = 'http://localhost:8080/actuator/loggers'], loggers-name=[Link@7334c574 href = 'http://localhost:8080/actuator/loggers/{name}'], metrics=[Link@7305758a href = 'http://localhost:8080/actuator/metrics'], metrics-requiredMetricName=[Link@44daf0a0 href = 'http://localhost:8080/actuator/metrics/{requiredMetricName}'], caches-cache=[Link@56941266 href = 'http://localhost:8080/actuator/caches/{cache}'], caches=[Link@3de72a5 href = 'http://localhost:8080/actuator/caches'], heapdump=[Link@5bafee71 href = 'http://localhost:8080/actuator/heapdump'], scheduledtasks=[Link@48e34a54 href = 'http://localhost:8080/actuator/scheduledtasks'], mappings=[Link@3e92660c href = 'http://localhost:8080/actuator/mappings']}}]
2020-06-26 15:45:59.313 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.header.writers.HstsHeaderWriter  : Not injecting HSTS header since it did not match the requestMatcher org.springframework.security.web.header.writers.HstsHeaderWriter$SecureRequestMatcher@5120d812
2020-06-26 15:45:59.314 DEBUG 64997 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
2020-06-26 15:45:59.324 DEBUG 64997 --- [nio-8080-exec-5] o.s.s.w.a.ExceptionTranslationFilter     : Chain processed normally
2020-06-26 15:45:59.324 DEBUG 64997 --- [nio-8080-exec-5] s.s.w.c.SecurityContextPersistenceFilter : SecurityContextHolder now cleared, as request processing completed
2020-06-26 15:45:59.325 DEBUG 64997 --- [nio-8080-exec-5] o.a.tomcat.util.net.SocketWrapperBase    : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read from buffer: [0]
2020-06-26 15:45:59.325 DEBUG 64997 --- [nio-8080-exec-5] org.apache.tomcat.util.net.NioEndpoint   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Read direct from socket: [0]
2020-06-26 15:45:59.325 DEBUG 64997 --- [nio-8080-exec-5] o.apache.coyote.http11.Http11Processor   : Socket: [org.apache.tomcat.util.net.NioEndpoint$NioSocketWrapper@3330cf23:org.apache.tomcat.util.net.NioChannel@55f6c228:java.nio.channels.SocketChannel[connected local=/0:0:0:0:0:0:0:1:8080 remote=/0:0:0:0:0:0:0:1:52501]], Status in: [OPEN_READ], State out: [OPEN]
```

&emsp;&emsp;客户端发来POST请求/login，请求体携带参数username和password，这个login请求就是登录请求，登录请求走Spring Security的过滤器链，走到了第6个过滤器`UsernamePasswordAuthenticationFilter`停下了，没有后续了，那么我们看一下这个过滤器。

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
        throws IOException, ServletException {

    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;

    if (!requiresAuthentication(request, response)) {
        chain.doFilter(request, response);

        return;
    }

    if (logger.isDebugEnabled()) {
        logger.debug("Request is to process authentication");
    }

    Authentication authResult;

    try {
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

    successfulAuthentication(request, response, chain, authResult);
}
```

&emsp;&emsp;这个用户名密码认证过滤器没有实现自己的doFilter方法，所以会走父类`AbstractAuthenticationProcessingFilter`的doFilter方法。在这个方法中，首先判断当前请求是不是登录请求，如果不是，那么走下一个过滤器，如果是，走`attemptAuthentication`方法进行登录校验，UsernamePasswordAuthenticationFilter实现了父类的抽象方法attemptAuthentication，如下：

```java
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
```

&emsp;&emsp;获取请求上面的用户名密码参数，然后通过`UsernamePasswordAuthenticationToken`认证最后用认证管理器管理认证信息。认证完成后继续走doFilter方法后续逻辑，比如说从请求缓存中拿到之前请求的请求对象，重定向到该请求路径(/actuator)上。

&emsp;&emsp;客户端重定向到/actuator后再次发起/actuator请求，走Spring Security的过滤器链，通过请求携带的session信息顺利完成验证，最终服务端将数据返回给客户端。

### 小结

&emsp;&emsp;大致的流程就是客户端发起一个请求，在没有登录的情况下抛出403错误，此错误被Spring Security异常过滤器捕获并通过判断让认证异常类`DelegatingAuthenticationEntryPoint`去处理了，这个类是一个委托类，它通过请求判断走具体哪一个`AuthenticationEntryPoint`实现类，此时显然走了`LoginUrlAuthenticationEntryPoint`将请求重定向到了登录请求上，客户端重新发起登录请求走了`DefaultLoginPageGeneratingFilter`将渲染好的页面响应给客户端，客户端输入用户名密码再次发起登录请求，此时走了`UsernamePasswordAuthenticationFilter`过滤器完成登录操作，将登录信息记录在认证管理器并重定向到登录前的请求上。客户端再次请求的时候由于请求携带的session信息已经完成认证，稍做比对后就完成认证放行了。


