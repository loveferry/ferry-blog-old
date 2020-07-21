---
title: Spring Security（三） 用户信息配置
date: 2020-07-21 12:08:25
tags:
    - Spring
    - Java
    - Spring Security
categories: Spring Security
---

&emsp;&emsp;继上一篇 [Spring Security（二） 认证配置类](Spring-Security-Authentication-Configuration.html) 后，这里展开说明一下其中点到的一个接口`UserDetailsService`，这个接口的作用在于加载用户数据。这里加载的用户数据就是我们合法的用户信息，前端登录传递的用户信息必须是这里加载的用户信息中的一员才可以通过校验。我们可以通过实现这个接口来完成自定义加载用户信息的实现。

<!-- more -->

# UserDetailsService

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

&emsp;&emsp;加载用户信息的核心接口。它作为用户DAO在整个框架中使用，在`DaoAuthenticationProvider`中就是使用的此接口获取用户数据。这个接口只有一个访问器方法，简化了新的数据访问策略的支持。

&emsp;&emsp;关注一下这个访问器方法，他是基于用户名加载用户数据的，当找不到用户时应当抛出`UsernameNotFoundException`异常，在我们项目开发中，可能不同的实现类实现方式不同，例如有的区分大小写，有的不区分大小写，在这种情况下我们返回的用户信息和前端传过来的就不一致了，此时我们应当在认证通过后将有凭证的用户信息存储在`SecurityContext`中。

# 用户信息组织结构

![组织结构图](/Spring-Security-User-Details/UserDetailsService.png)

-  `UserDetailsManager`：对用户信息接口进行扩展，使之可以对用户信息进行更新；
-  `JdbcDaoImpl`：封装了默认的查询语句，用于从指定的表获取用户信息；
-  `CachingUserDetailsService`：用户数据缓存实现，将加载的数据缓存起来，在使用时先从缓存中获取，若为空再从委托的UserDetailsService接口加载用户并缓存。其中的缓存接口`UserCache`既可以使用其实现类`EhCacheBasedUserCache`也可以使用`SpringCacheBasedUserCache`传入一个`Cache`从而使用Spring的缓存。
-  `UserDetailsServiceDelegator`：是一个内部类，他的作用是获取全局或者局部的`AuthenticationManagerBuilder`的默认UserDetailsService对象，从而委托给其完成加载数据的操作。

