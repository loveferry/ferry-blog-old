---
title: Spring Security（二） 认证配置
date: 2020-06-26 12:08:25
tags:
    - Spring
    - Java
    - Spring Security
categories: Spring Security
---

&emsp;&emsp;继上一篇 [Spring Security（一） 体系结构之认证](Spring-Security-Architecture-Authentication.html) 后，这里研究一下Spring Security是如何配置我们的认证信息的。
<!-- more -->

# AuthenticationConfiguration

&emsp;&emsp;这是认证配置的配置类，该配置类的主要作用是用于生成一个全局的`AuthenticationManagerBuilder`注入到Spring容器中。使用者注入该配置类然后获取AuthenticationManagerBuilder再生成`AuthenticationManager`。


# AuthenticationManagerBuilder

&emsp;&emsp;**这个建造器用于创建一个认证管理器(AuthenticationManager)，能够轻松的建造出一个基于内存的认证，LDAP认证或者是基于jdbc的认证，添加`UserDetailsService`以及添加`AuthenticationProvider`** 。

&emsp;&emsp;需要注意的是，使用AuthenticationManagerBuilder.build()构造出来的AuthenticationManager是`ProviderManager`的实例。看一下它的继承关系。

![继承关系](/Spring-Security-Authentication-Configuration/AuthenticationManagerBuilder-extends.png)

&emsp;&emsp;通过继承关系可以看出，AuthenticationManagerBuilder和它的父类AbstractConfiguredecurityBuilder都没有实现build方法，所以调用的是AbstractSecurityBuilder方法。

```java
public final O build() throws Exception {
    if (this.building.compareAndSet(false, true)) {
        this.object = doBuild();
        return this.object;
    }
    throw new AlreadyBuiltException("This object has already been built");
}
```

&emsp;&emsp;调用了`doBuild()`方法并返回其结果，AuthenticationManagerBuilder没有实现此方法，而父类AbstractConfiguredecurityBuilder实现了。

````java
protected final O doBuild() throws Exception {
    synchronized (configurers) {
        buildState = BuildState.INITIALIZING;
        
        beforeInit();
        init();
        
        buildState = BuildState.CONFIGURING;
        
        beforeConfigure();
        configure();
        
        buildState = BuildState.BUILDING;
        
        O result = performBuild();
        
        buildState = BuildState.BUILT;
        
        return result;
    }
}
````

&emsp;&emsp;可以看到，最终返回的是`performBuild()`方法的返回值，而AuthenticationManagerBuilder实现了此方法。

```java
protected ProviderManager performBuild() throws Exception {
    if (!isConfigured()) {
        logger.debug("No authenticationProviders and no parentAuthenticationManager defined. Returning null.");
        return null;
    }
    ProviderManager providerManager = new ProviderManager(authenticationProviders, parentAuthenticationManager);
    if (eraseCredentials != null) {
        providerManager.setEraseCredentialsAfterAuthentication(eraseCredentials);
    }
    if (eventPublisher != null) {
        providerManager.setAuthenticationEventPublisher(eventPublisher);
    }
    providerManager = postProcess(providerManager);
    return providerManager;
}
```

&emsp;&emsp;显然，使用AuthenticationManagerBuilder的build方法构造AuthenticationManager其实例的类型就是ProviderManager。
