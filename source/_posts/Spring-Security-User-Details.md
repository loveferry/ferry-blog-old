---
title: Spring Security（三） 用户信息配置
date: 2020-07-21 09:57:25
tags:
    - Spring
    - Java
    - Spring Security
categories: Spring Security
---

&emsp;&emsp;继上一篇 [Spring Security（二） 认证配置类](Spring-Security-Authentication-Configuration.html) 后，这里展开说明一下其中点到的一个接口`UserDetailsService`，这个接口的作用在于加载用户数据。这里加载的用户数据就是我们合法的用户信息，前端登录传递的用户信息必须是这里加载的用户信息中的一员才可以通过校验。我们可以通过实现这个接口来完成自定义加载用户信息的实现。

<!-- more -->

