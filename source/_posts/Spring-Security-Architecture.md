---
title: Spring Security（一） 体系结构
date: 2020-06-26 12:08:25
tags:
    - Spring
    - Java
    - Spring Security
categories: Spring Security
---

&emsp;&emsp;`Spring Security`是Spring的安全框架，它能够帮助我们完成用户的认证和访问控制，具体的来说呢就是帮助我们判断当前请求是否需要认证，如果需要认证那么提供的认证信息是否合法有效，提供的认证信息是否具有足够的权限访问当前请求需要的资源。 

<!-- more -->

# 认证（Authentication）

&emsp;&emsp;**认证主要完成对提供的用户信息进行识别，判断其是否合法有效。** 在Spring Security中，为认证设计的接口是`AuthenticationManager`。

```java
package org.springframework.security.authentication;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;

public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;
}
```

&emsp;&emsp;这个接口只有一个方法，所有的的用户认证都得调用这个接口去完成信息认证，参数`authentication`其类型也是一个接口，这个接口是对认证信息的封装，Spring Security通过调用该接口的方法获取用户信息完成认证并且后期对用户信息鉴权也需要从此接口获取信息。在调用认证方法后，如果认证顺利，那么就会返回一个认证信息，此认证信息因为已经完成认证，此时调用`isAuthenticated()`方法应该返回`true`。注意，返回的这个认证信息一般情况下是创建了一个新的对象，而不是修改参数对象并返回，所以这是一个访问器方法。在实际开发中，我们可以自定义`Authentication`接口的实现类，在其中封装更多的项目需要的参数，比如说我们用户信息需要携带一个邮箱地址，电话号码等，Spring Security本身提供了一些其实现类供我们使用，例如`UsernamePasswordAuthenticationToken`用于用户名密码认证的场景，`AnonymousAuthenticationToken`用于匿名认证的场景。

&emsp;&emsp;如果认证的过程并不顺利，比如说用户不存在，密码错误，用户失效等常见情况，此时应当抛出认证异常`AuthenticationException`。认证异常是一个抽象类。其有诸多的子类，在不同的情况下应当抛出不同子类异常，例如当账户失效时应当抛出的`AccountExpiredException`，当认证的用户名密码错误时应当抛出的`BadCredentialsException`和用户信息被锁定时应当抛出的`DisabledException`等。当抛出AuthenticationException异常时，服务应当响应给客户端401状态码，告诉客户端认证失败了，此时客户端应当自行处理最终将其渲染并呈现给用户。

&emsp;&emsp;AuthenticationManager的一个重要实现就是`ProviderManager`，一般情况下都是通过ProviderManager完成认证，接下来看看这个类能做些什么吧。

## ProviderManager

&emsp;&emsp;通过阅读源码上的注释可以了解到，它是将认证信息Authentication依次尝试AuthenticationProviders列表，如果响应结果不为空表示当前提供者有权决定认证请求，并且不再尝试其他提供者。如果后续提供者成功验证了请求，则将忽略较早的验证异常，并将使用成功的验证。如果没有后续提供者提供非空响应或new AuthenticationException，AuthenticationException则将使用最后收到的提供者。如果没有提供者返回非null的响应，或者表明它甚至可以处理an Authentication，ProviderManager则将抛出 ProviderNotFoundException。还可以设置一个parent的AuthenticationManager，如果没有配置的认证程序可以执行身份验证，也可以尝试使用此方法。但是通常不应该使用此功能。

&emsp;&emsp;完整的看一下这个类。

```java
package org.springframework.security.authentication;

import java.util.Collections;
import java.util.List;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.MessageSource;
import org.springframework.context.MessageSourceAware;
import org.springframework.context.support.MessageSourceAccessor;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.CredentialsContainer;
import org.springframework.security.core.SpringSecurityMessageSource;
import org.springframework.util.Assert;

public class ProviderManager implements AuthenticationManager, MessageSourceAware,
		InitializingBean {

    /**
    * 日志对象
    */
	private static final Log logger = LogFactory.getLog(ProviderManager.class);
	
    /**
    * 认证事件发布器，认证成功或者失败的通知机制
    */
	private AuthenticationEventPublisher eventPublisher = new NullEventPublisher();
	
	/**
	* 认证提供者列表，实际完成认证工作的列表
    */
	private List<AuthenticationProvider> providers = Collections.emptyList();
	
	/**
	* 消息国际化的实现类，这里不做扩展
    */
	protected MessageSourceAccessor messages = SpringSecurityMessageSource.getAccessor();
	
	/**
	* 父级认证管理器，相当于一个默认的认证程序，如果上面的认证提供者列表中没有一个完成认证工作，那么就会尝试使用这个认证管理器进行认证
    */
	private AuthenticationManager parent;
	
	/**
	* 认证成功后是否擦除凭证信息
    */
	private boolean eraseCredentialsAfterAuthentication = true;

	public ProviderManager(List<AuthenticationProvider> providers) {
		this(providers, null);
	}

    /**
    * 初始化对象的时候必须提供一个认证提供者的列表，父级的认证管理器可以为空
    */
	public ProviderManager(List<AuthenticationProvider> providers,
			AuthenticationManager parent) {
		Assert.notNull(providers, "providers list cannot be null");
		this.providers = providers;
		this.parent = parent;
		checkState();
	}

    /**
    * 属性设置完成后校验，父级认证管理器和认证提供者列表二者至少有一个不能为空，因为若都为空则无法通过此对象完成认证工作 
    */
	public void afterPropertiesSet() throws Exception {
		checkState();
	}

	private void checkState() {
		if (parent == null && providers.isEmpty()) {
			throw new IllegalArgumentException(
					"A parent AuthenticationManager or a list "
							+ "of AuthenticationProviders is required");
		}
	}

    /**
    * 认证过程
    */
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
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
			catch (AccountStatusException e) {
				prepareException(e, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw e;
			}
			catch (InternalAuthenticationServiceException e) {
				prepareException(e, authentication);
				throw e;
			}
			catch (AuthenticationException e) {
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
				lastException = e;
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

		prepareException(lastException, authentication);

		throw lastException;
	}

    /**
    * 将异常通过认证事件发布器发布出去 
    */
	@SuppressWarnings("deprecation")
	private void prepareException(AuthenticationException ex, Authentication auth) {
		eventPublisher.publishAuthenticationFailure(ex, auth);
	}

    /**
    * 将需要认证的认证信息赋值到新的认证信息中
    */
	private void copyDetails(Authentication source, Authentication dest) {
		if ((dest instanceof AbstractAuthenticationToken) && (dest.getDetails() == null)) {
			AbstractAuthenticationToken token = (AbstractAuthenticationToken) dest;

			token.setDetails(source.getDetails());
		}
	}

    /**
    * 获取认证提供者
    */
	public List<AuthenticationProvider> getProviders() {
		return providers;
	}
    
	/**
	* 设置消息国际化 
    */
	public void setMessageSource(MessageSource messageSource) {
		this.messages = new MessageSourceAccessor(messageSource);
	}

    /**
    * 设置认证事件发布器
    */
	public void setAuthenticationEventPublisher(
			AuthenticationEventPublisher eventPublisher) {
		Assert.notNull(eventPublisher, "AuthenticationEventPublisher cannot be null");
		this.eventPublisher = eventPublisher;
	}

    /**
    * 设置是否在认证成功后将凭证信息擦除
    */
	public void setEraseCredentialsAfterAuthentication(boolean eraseSecretData) {
		this.eraseCredentialsAfterAuthentication = eraseSecretData;
	}

    /**
    * 是否在认证成功后擦除凭证信息
    */
	public boolean isEraseCredentialsAfterAuthentication() {
		return eraseCredentialsAfterAuthentication;
	}

    /**
    * 空的认证事件发布器
    */
	private static final class NullEventPublisher implements AuthenticationEventPublisher {
		public void publishAuthenticationFailure(AuthenticationException exception,
				Authentication authentication) {
		}

		public void publishAuthenticationSuccess(Authentication authentication) {
		}
	}
}
```



