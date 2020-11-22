---
title: 搭建maven私服
date: 2020-11-14 10:55:01
categories: 开发工具
tags:
    - maven
---

&emsp;&emsp;私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的Maven用户使用。当Maven需要下载构件的时候，它从私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。

<!-- more -->

### Nexus3

&emsp;&emsp;目前市面上有三种maven仓库管理器可以搭建私服，分别是[apache的archiva](http://archiva.apache.org)、[artifactory](https://www.jfrogchina.com)、[Sonatype的Nexus](https://www.sonatype.com)。目前使用较多的是Nexus，它简化了本地内部仓库的维护和外部仓库的访问；Nexus在代理远程仓库的同时维护本地仓库，以降低中央仓库的负荷,节省外网带宽和时间；Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据；Nexus使用ExtJS来开发界面，利用Restlet来提供完整的REST APIs；Nexus支持WebDAV与LDAP安全身份认证;Nexus还提供了强大的仓库管理功能，构件搜索功能，它基于REST，友好的UI是一个extjs的REST客户端，它占用较少的内存，基于简单文件系统而非数据库。

&emsp;&emsp;这里选择下载Nexus的docker镜像安装，我们可以在`docker hub`上选择[Nexus的官方镜像](https://hub.docker.com/r/sonatype/nexus3) 。

```bash
docker pull sonatype/nexus3
```

&emsp;&emsp;运行容器

```bash
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

&emsp;&emsp;测试

```bash
curl http://localhost:8081
```

### 配置nexus

#### 初始化设置

&emsp;&emsp;登录

![登录](/maven-private-server/nexus-home.png)

&emsp;&emsp;用户名是`admin`，可以通过`docker exec -it nexus /bin/bash`进入容器，然后查看`/nexus-data/admin.password`文件获取admin用户的初始密码。

&emsp;&emsp;登录后会要求修改初始密码,然后需要设置匿名用户的访问权限，这里禁止匿名用户访问。

![匿名访问控制](/maven-private-server/nexus-anonymous-permission.png)

#### 仓库配置

&emsp;&emsp;nexus上的maven仓库分三种类型：

> proxy：代理仓库，代理远程的中央仓库，当用户向该类型仓库请求时，先在本地寻找，未找到的情况下去代理的中央仓库下载并存入本地然后返回给用户
> hosted：宿主仓库，部署自己的构建到该仓库中
> group：仓库组，用来组合多个宿主仓库和代理仓库的，当项目引用多个仓库时，可以使用仓库组将多个仓库组合起来，项目直接使用仓库组即可。

&emsp;&emsp;替换代理仓库地址，删除nexus自带的仓库。这里我们选择[阿里云的中央仓库](https://maven.aliyun.com/repository/public),这里给出我使用的仓库地址 `https://maven.aliyun.com/repository/public` 。

&emsp;&emsp;删除所有nexus自带的仓库，创建以下几个仓库：一个代理仓库，用来代理中央仓库；至少一个宿主仓库（每个项目创建一个对应的宿主仓库）；一个仓库组（可选，若A项目依赖于B项目，那么可以创建一个仓库组将代理仓库和B项目的仓库组合在一起，这样可以避免A项目使用多个仓库）。

&emsp;&emsp;创建代理仓库

![创建代理仓库](/maven-private-server/nexus-create-repository.png)

![创建代理仓库](/maven-private-server/nexus-create-proxy.png)

![创建代理仓库](/maven-private-server/nexus-create-proxy2.png)

&emsp;&emsp;创建宿主仓库

![创建宿主仓库](/maven-private-server/nexus-create-repository.png)

![创建宿主仓库](/maven-private-server/nexus-create-hosted.png)

![创建宿主仓库](/maven-private-server/nexus-create-hosted2.png)

&emsp;&emsp;创建仓库组

![创建仓库组](/maven-private-server/nexus-create-repository.png)

![创建仓库组](/maven-private-server/nexus-create-group.png)

![创建仓库组](/maven-private-server/nexus-create-group2.png)

#### 用户角色配置

&emsp;&emsp;创建角色

![创建角色](/maven-private-server/nexus-create-role.png)

![创建角色](/maven-private-server/nexus-create-role2.png)

&emsp;&emsp;创建用户

![创建用户](/maven-private-server/nexus-create-user.png)

![创建用户](/maven-private-server/nexus-create-user2.png)

&emsp;&emsp;到这里私服就算是配置好了，下面就是说明一下如何使用私服。

### 使用私服

#### maven从私服获取依赖

&emsp;&emsp;我们在上一步创建了角色和用户，并为角色分配了权限，为用户分配了角色，这里我们使用创建的用户来完成私服的使用。

&emsp;&emsp;首先我们需要直到maven获取依赖的搜索顺序：

> 本地仓库：本地仓库的地址在maven的配置文件中指定，默认是`~/.m2/repository/`;
> 远程仓库：项目的配置文件(pom.xml)中的 repositories 配置的优先级高于配置文件(settings.xml)中的 profiles 中配置的 repositories；
> 中央仓库：默认的一个远程仓库，其id为`central`

&emsp;&emsp;在配置中需要注意**若远程仓库需要用户名密码，则需要配置settings.xml配置文件中的server，其id标签和repository的id标签需要保持一致**。

&emsp;&emsp;镜像仓库的理解：若A仓库的所有内容都可以在B仓库寻找到，那么B仓库就是A仓库的镜像。基于此，我们使用nexus创建一个代理仓库，可以使用这个代理仓库作为中央仓库的镜像，每个项目可以配置其自己的仓库。总结来说，每个项目可以从自己的仓库获取依赖文件，而镜像就是所有项目的最后一道防线，如果所需的依赖从镜像仓库中都不能获取，那么maven就会报错找不到依赖文件。

```bash
<mirrors>
    <mirror>
        <id>nexus-proxy-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>ferry私服代理仓库</name>
        <url>http://ferry.org.cn:8081/repository/proxy-aliyun/</url>
    </mirror>
</mirrors>
```

&emsp;&emsp;上面的就是镜像的配置，该配置是写在settings.xml配置文件中的，我们可以配置多个仓库的镜像，这里举例配置了中央仓库的镜像，如上配置，当访问central仓库的时候url会被替换成该镜像配置中的url，如果访问需要用户名密码校验，那么他会去servers标签下寻找与此镜像id相同的server来完成用户认证。**注意：`mirrorOf`中不能写`*`,`*`表示对所有仓库镜像**。

&emsp;&emsp;基于上面的理解，我们可以为项目配置一个项目级的仓库，然后配置一个中央仓库的镜像仓库。

##### 配置项目级仓库

&emsp;&emsp;settings配置文件修改,在servers标签下添加

```xml
<server>
    <id>group-ferry</id>
    <username>developer</username>
    <password>***********</password>
</server>
```

&emsp;&emsp;项目下的配置文件pom.xml修改,在repositories标签下添加

```xml
<repository>
    <id>group-ferry</id>
    <url>http://ferry.org.cn:8081/repository/group-ferry/</url>
</repository>
```

&emsp;&emsp;注意，两个id要一致，这是用户认证的关键。

##### 配置镜像

&emsp;&emsp;settings配置文件修改,在servers标签下添加

```xml
<server>
    <id>nexus-proxy-aliyun</id>
    <username>developer</username>
    <password>***********</password>
</server>
```

&emsp;&emsp;settings配置文件修改,在mirrors标签下添加

```xml
<mirror>
    <id>nexus-proxy-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>ferry私服代理仓库</name>
    <url>http://ferry.org.cn:8081/repository/proxy-aliyun/</url>
</mirror>
```

&emsp;&emsp;同样的，两个id要一致，这是用户认证的关键。这里`mirrorOf`值为`central`，表示是`central`仓库的镜像。

&emsp;&emsp;配置全局仓库的话就是先添加用户密码信息（settings.xml中添加一个server标签），然后在settings.xml的`profiles`中定义，这里不举例了。

#### 构建服务到私服

&emsp;&emsp;实际开发中，我们需要将开发好的公共服务发布到私服供其他服务使用，这是我们需要进行一定的配置。

&emsp;&emsp;这里回顾以下我们之前在nexus上创建的宿主仓库（如下图），我们可以选择版本为稳定发布板（releases）还是开发版（Snapshot）亦或者修复版本（Mixed），还有一个重要的选择，就是`Deployment policy`需要选择`Allow redeploy`,否则无法上传我们的构建。

![创建宿主仓库](/maven-private-server/nexus-create-hosted2.png)

&emsp;&emsp;settings配置文件修改,在servers标签下添加

```xml
<server>
    <id>ferry-release</id>
    <username>developer</username>
    <password>***********</password>
</server>
```

&emsp;&emsp;项目下的配置文件pom.xml修改,在distributionManagement标签下添加

```xml
<repository>
    <id>ferry-release</id>
    <url>http://ferry.org.cn:8081/repository/ferry-releases/</url>
</repository>
```

&emsp;&emsp;两个id要一致，这是用户认证的关键。

&emsp;&emsp;发布构建的时候在项目根目录下执行`mvn clean deploy`即可。

### 浏览nexus仓库内容

&emsp;&emsp;nexus提供了友好的UI界面供我们访问仓库内容

![browse](/maven-private-server/nexus-browse.png)