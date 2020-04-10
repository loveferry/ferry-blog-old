---
title: 常用软件安装
date: 2018-12-17 18:45:37
tags:
    - jdk的安装
categories: 软件安装
---

&emsp;&emsp;记录一下常用的软件的安装以及环境变量的配置，省的以后百度浪费时间。

<!-- more -->


### JDK的安装

&emsp;&emsp;作为Java语言的SDK(软件开发工具包，Software development kit)，普通用户并不需要安装JDK来运行Java程序，而只需要安装JRE（Java Runtime Environment）。而程序开发者必须安装JDK(Java Development Kit)来编译、调试程序。

#### linux系统安装JDK

&emsp;&emsp;在linux上安装JDK有多种方式，你可以使用`wget`从官网下载压缩文件然后解压，然后配置环境变量；也可以使用`yum`直接安装JDK。

##### 使用`yum`安装 openjdk

- 查看JDK版本

```bash
yum list java-1.8*
```

![运行结果](/software-install/install-jdk-linux.png)


- 安装JDK

```bash
yum -y install java-1.8.0-openjdk-devel.x86_64
```

&emsp;&emsp;这里需要注意，我们安装JDK需要选择`-devel`的下载安装，没有这个的是JRE，`x86_64`是64位的，请根据机器的版本进行选择。`-y`参数是在安装过程中如有出现选择是与否时全部默认选择是。

- 查看版本信息

```bash
java -version
```

- 查看JDK安装目录

```bash
which java
```

&emsp;&emsp;发现是一个引用，再查看引用的目录，最终找到实际的安装目录

- 配置java环境变量

```bash
vi /etc/profile
```

```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
export PATH=${PATH}:${JAVA_HOME}/bin
```

```bash
source /etc/profile
```

![jdk安装目录](/software-install/jdk_install_dir.png)

##### 使用 rpm 安装 oracle jdk

- 移除系统自带的open jdk

```bash
yum remove -y *openjdk*
```

&emsp;&emsp;从[官网下载](https://www.oracle.com/technetwork/java/javase/downloads/index.html)对应版本的jdk，注意，是rpm后缀的，可以直接使用wget下载到服务器，但是我每次这样都会说验证失败，所以我的操作是先下载到本地，然后通过scp或者ftp上传到服务器。

- 安装

```bash
rpm -ivh /usr/local/jdk-8u231-linux-i586.rpm
```

&emsp;&emsp;安装完成后rpm包就可以删除了。使用`java -version`可以查看jdk是否安装成功。配置java环境变量可以参考【使用`yum`安装 openjdk】节的步奏。

### maven安装

![maven](/software-install/install-maven.png)

&emsp;&emsp;在官网上有不同的maven下载链接供大家选择，这里简单介绍一下。`-bin`结尾的是编译好的class文件，可以直接使用，我们一般也选择这种的进行下载，`-src是java原文件，可以查看源码。`.tar.gz`是linux系统用的压缩格式，`.zip`是windows系统用的压缩格式，所以根据自己的硬件配置选择对应的文件下载就好了。

&emsp;&emsp;友情提示，maven是用java语言实现的，所以安装maven必须得先安装jdk。

#### linux安装maven

- 下载

```bash
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.tar.gz
```

- 解压

```bash
tar zxvf apache-maven-3.6.0-bin.tar.gz
```

- 配置环境变量

```bash
vi /etc/profile
```

&emsp;&emsp;在文件最后面添加如下语句

```bash
export MAVEN_HOME=/usr/local/apache-maven-3.6.0
export PATH=${PATH}:${MAVEN_HOME}/bin
```

&emsp;&emsp;按`ESC`,输入`:`然后输入`wq`然后回车，这一步是保存文件的操作。

- 使配置立即生效

```bash
source /etc/profile
```

- 查看maven版本信息

```bash
mvn --version
```

- 修改镜像为阿里云

```bash
vi $MAVEN_HOME/conf/settings.xml
```

&emsp;&emsp;将mirrors下替换为以下内容。

```bash
<mirror>
  <id>alimaven</id>
  <name>aliyun maven</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
  <mirrorOf>central</mirrorOf>
</mirror>
```

### tomcat安装

&emsp;&emsp;tomcat的运行是需要JDK的支持的哦，所以，请先安装JDK。

![tomcat](/software-install/install-tomcat-linux.png)

#### linux安装tomcat

&emsp;&emsp;在tomcat官网上，在左边选择我们需要的tomcat版本，在右边选择`core`下的`tar.gz`格式的文件进行下载。

&emsp;&emsp;有时候我们会在网上看到安装了tomcat后还需要配置一个环境变量叫做`CATALINA_HOME`,关于这个环境变量的配置可以参考这篇博客，个人觉得说的很详细了。[tomcat启动分析](https://www.cnblogs.com/heshan664754022/archive/2013/03/27/2984357.html)。

- 下载

```bash
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.35/bin/apache-tomcat-8.5.35.tar.gz
```

- 解压

```bash
tar zxvf apache-tomcat-8.5.35.tar.gz
```

&emsp;&emsp;到这里就已经安装好了。

### git安装

&emps;&emsp;git，非常先进的分布式版本控制系统。

#### linux安装git

&emsp;&emsp;在linux环境下可以使用`yum`指令安装git，但是版本未必符合我们的要求，为此，这里介绍从github下载源码进行编译安装的方式。

- 下载

```bash
wget https://github.com/git/git/archive/master.zip
```

- 解压

```bash
unzip master.zip
```

&emsp;&emsp;如果这里提示`-bash: unzip: 未找到命令`,那么请使用`yum -y install unzip`安装，然后执行上面命令。

- 重命名

```bash
mv git-master git
```

&emsp;&emsp;将解压的文件夹重新命名，这一步可有可无，我有强迫症，喜欢用这个名字命名git文件夹。

- 安装插件

```bash
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```

- 进入git根目录，编译

```bash
make prefix=/usr/local/git all
```

&emsp;&emsp;其中的`/usr/local/git`是git的安装目录，这个就是配置git安装目录并且编译

- 安装

```bash
make install
```

- 查看版本信息

```bash
git --version
```

&emsp;&emsp;这一步如果抱git命令未找到错误，那么就需要你去配置一下环境变量了。

- 配置环境变量

```bash
vi /etc/profile
```

```bash
export PATH=$PATH:/usr/local/git/bin
```

&emsp;&emsp;保存退出后执行`source /etc/profile`然后查看版本信息。

### redis安装

&emsp;&emsp;介绍安装redis服务并且注册为系统服务开机自启。

#### windows安装redis

&emsp;&emsp;现在redis官网没有windows版本，只有linux版本可供下载，那么先从github上[下载 redis](https://github.com/MicrosoftArchive/redis/releases) 。

&emsp;&emsp;下载安装版的就按照步骤next就好了，解压版的就解压在你指定的地方就可以了。

* 在redis安装目录下打开终端，按住shift点击鼠标右键打开命令窗口输入以下指令启动服务

``` bash
redis-server redis.windows.conf
```

&emsp;&emsp;服务成功启动如图所示：

![启动redis](/software-install/redis_start.png)

&emsp;&emsp;通过上述方式启动redis虽然可以正常使用，但是只要关闭该命令窗口服务就会停止，而不关闭该窗口任务栏占用一个位置就很烦，每次开机都要打开安装目录执行一遍启动命令也是很烦的。所以可以考虑将redis设置成系统服务。

* 在没有启动redis的前提下，在redis的安装目录打开命令窗口，执行

``` bash
redis-server --service-install redis.windows-service.conf --loglevel verbose
```

&emsp;&emsp;如果没有报错那么说明成功了，此时打开任务管理器，单击服务，可以看到redis服务已经存在了。

![注册服务](/software-install/redis_server.png)

* 刚刚设置的服务可能没有启动，此时执行下面的语句启动服务

``` bash
redis-server --service-start
```

* 刷新任务管理器，查看服务是否已经启动，当有一天你可能不再使用redis了，可以使用以下命令停止redis服务

``` bash
redis-server --service-stop
```

* 卸载redis服务

``` bash
redis-server --service-uninstall
```

#### linux安装redis

##### 方法一

- 安装redis

```bash
yum install -y redis
```

- 启动redis服务

```bash
systemctl start redis.service
```

- 开机自启

```bash
systemctl enable redis.service
```

##### 方法二

###### 安装

- 下载安装包

```bash
cd /usr/local/
wget http://download.redis.io/releases/redis-4.0.14.tar.gz
```

&emsp;&emsp;选择Stable版本下载。

- 解压

```bash
tar -zxvf redis-4.0.14.tar.gz
```

- 编译

```bash
cd /usr/local/redis-4.0.14/
make
```

- 启动

```bash
cd /usr/local/redis-4.0.14/src/
./redis-server ../redis.conf
```

- 连接

```bash
cd /usr/local/redis-4.0.14/src/
./redis-cli
```

###### 配置开机自启服务

- 复制启动脚本

```bash
# 复制redis的脚本文件
cd /usr/local/redis-4.0.14/src/
cp redis-server redis-cli /usr/local/bin/
```

- 修改redis的配置文件

```bash
vi /usr/local/bin/usr/local/redis-4.0.14/redis.conf
```

```bash
supervised systemd
daemonize yes
```

- 配置服务

```bash
vi /lib/systemd/system/redisd.service
```

```bash
# 基础信息
[Unit]
# 描述
Description=Redis
# 在某个服务启动后启动
After=network.target

# 服务信息
[Service]
Type=forking
# 启动服务的命令
ExecStart=/usr/local/bin/redis-server /usr/local/bin/usr/local/redis-4.0.14/redis.conf
# 终止服务的命令
ExecStop=/usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 shutdown

# 安装相关信息
[Install]
# WantedBy 是以哪种方式启动：multi-user.target表明当系统以多用户方式（默认的运行级别）启动时，这个服务需要被自动运行。
WantedBy=multi-user.target
```

- 可执行权限

```bash
chmod +x /lib/systemd/system/redisd.service
```

- 刷新配置

```bash
systemctl daemon-reload
```

- 启动服务

```bash
systemctl start redisd.service
```

- 查看状态

```bash
systemctl status redisd.service
```

- 关闭服务

```bash
systemctl stop redisd.service
```

- 开机自启

```bash
systemctl enable redisd.service
```

- 禁止开机自启

```bash
systemctl disable redisd.service
```

### linux安装jenkins

- 安装jenkins

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install -y jenkins
```

&emsp;&emsp;目录介绍：

>配置文件目录：`/etc/sysconfig/jenkins`;
>默认的JENKINS_HOME目录：`/var/lib/jenkins/`；
>jenkins安装目录：/usr/lib/jenkins/；
>jenkins日志文件：`/var/log/jenkins/jenkins.log`。

&emsp;&emsp;具体的路径啊，端口啊都可以通过配置文件修改的，参数里面找一下，一眼就认出来，改好了保存重启jenkins就好了。

- 启动

```bash
systemctl start jenkins
```

- 查看jenkins进程信息

```bash
ps -aux |grep jenkins|grep -v grep
```

&emsp;&emsp;jenkins默认监听的是8080端口，需要防火墙开放8080端口或者自定义的端口。

- 第一次登陆

![login](/software-install/jenkins_login.png)

&emsp;&emsp;根据提示去对应的文件查看密码,然后将密码输入，点击继续。

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

- 插件的安装

![plugin](/software-install/jenkins_plugin.png)

&emsp;&emsp;一般情况下我们点击左边的"安装推荐的插件"就好了。

- 创建管理员用户

&emsp;&emsp;按照提示创建就好了。创建好了之后后面的一路默认就好了。

- 额外的插件安装

&emsp;&emsp;系统管理-插件管理-可选插件：下面是我另外安装的插件列表:

> Localization: Chinese (Simplified)：Jenkins 及其插件的简体中文语言包。
> Git Parameter：参数化构建git项目
> Maven Integration：maven项目

- JDK和MAVEN的路径配置

&emsp;&emsp;系统管理-全局工具配置:点击新增JDK，取消勾选"自动安装"，将JAVA_HOME的路径copy到此处；点击新增MAVEN，取消勾选"自动安装"，将M2_HOME的路径copy到此处。

- 错误解决

```bash
HTTP ERROR 403 Problem accessing /view/all/createItem. 
Reason: No valid crumb was included in the request 
Powered by Jetty:// 9.4.z-SNAPSHOT
```

&emsp;&emsp;系统管理-全局安全设置-跨站请求伪造保护:取消勾选"防止跨站点请求伪造"。

#### 创建spring boot项目的任务

- 新建任务

![task](/software-install/jenkins_task.png)

- 配置

1. 描述：随便写一点。

2. 参数化构建过程：勾选，选择`Git Parameter`，`Parameter Type`可以选择branch或者tag，看具体情况，然后给定一个默认值，

![General](/software-install/jenkins_general.png)

3. 源码管理：选择git,添加项目的git地址，然后添加用户名密码或者公私钥。`Branch Specifier (blank for 'any')`使用$符号后面加上前面参数化构建过程中定义的名字，在这里就是`$branch`。

4. Pre Steps：add一个"执行shell"。

```bash
today=$(date +%Y%m%d) ;

server_name=ferry ;

target_dir=/u01/$server_name ;
start_log_dir=$target_dir/start_log/$today ;
start_log=$start_log_dir/start.log ;
back_up_dir=$target_dir/back_up/$today ;
log_dir=$target_dir/log/$today ;

if test ! -d $target_dir ; then mkdir -p $target_dir ; fi;
if test ! -d $start_log_dir ; then mkdir -p $start_log_dir ; fi;
if test ! -d $back_up_dir ; then mkdir -p $back_up_dir ; fi;
if test ! -d $log_dir ; then mkdir -p $log_dir ; fi;

java_pid=`ps -aux|grep java|grep -v grep|grep -v kill|grep -v jenkins|awk '{print $2}'` ;

echo -------------------BEGIN KILL SERVER----------------------- >> $start_log ;
if [ $java_pid ] ; 
then kill -s 9 $java_pid ; 
echo kill java server success >> $start_log ; 
else echo not exists server is running >> $start_log ; 
fi ;
echo -------------------END   KILL SERVER----------------------- >> $start_log ;

echo ----------------BEGIN BACKUP OLD SERVER-------------------- >> $start_log ;
if test -f $WORKSPACE/core/target/ferry.jar ;
then mv $WORKSPACE/core/target/ferry.jar $back_up_dir/ ; 
echo move jar to $back_up_dir dir >> $start_log ;
fi ;
echo -----------------END BACKUP OLD SERVER--------------------- >> $start_log ;
```

5. Build：Goals and options中填写`clean package -Dmaven.test.skip=true -P prod`

6. Post Steps：add一个"执行shell"。

```bash
BUILD_ID=DONTKILLME
today=$(date +%Y%m%d) ;
server_name=ferry ;

target_dir=/u01/$server_name ;
start_log_dir=$target_dir/start_log/$today ;
start_log=$start_log_dir/start.log ;
back_up_dir=$target_dir/back_up/$today ;
log_dir=$target_dir/log/$today ;

echo ----------------BEGIN RUN SERVER-------------------- >> $start_log ;
cd $WORKSPACE/core/target/ ;
nohup java -jar ferry.jar > $log_dir/log.out & 
echo SERVER RUNNING SUCCESS >> $start_log ;
echo ------------------END RUN SERVER-------------------- >> $start_log ;
```

- 执行任务

&emsp;&emsp;点击"Build with Parameters"->选择分支->点击"开始构建"

![build](/software-install/jenkins_build.png)

### sysbench安装

&emsp;&emsp;[sysbench](https://dev.mysql.com/downloads/benchmarks.html)是一个开源的、模块化的、跨平台的多线程性能测试工具，可以用来进行CPU、内存、磁盘I/O、线程、数据库的性能测试。

```bash
cd /usr/local/
tar -zxvf sysbench-0.4.12.14.tar.gz
cd sysbench-0.4.12.14/
./configure --with-mysql-includes=/usr/local/mysql/include --with-mysql-libs=/usr/local/mysql/lib
make && make install
```

### nginx安装

#### centos源代码编译安装nginx

##### 基本安装

- 下载源文件并解压

```bash
cd /usr/local/
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
```

- 编译选项的配置参数说明

> `--prefix=path`:它定义了存放服务器文件的路径。configure及nginx.conf配置文件中的所有路径都是以该目录作为一个相对路径（库源文件路径除外）。默认情况下，其会被设置为/usr/local/nginx目录。
> `--sbin-path=path`: 设置nginx可执行文件的名字。该名字只在安装过程中会被用到。默认情况下该文件会被命名为prefix/sbin/nginx
> `--conf-path=path`: 设置nginx.conf配置文件的名字。假如需要，nginx可以通过命令行参数-c file 来指定一个不同的配置文件。默认情况下，该文件会被设置为prefix/conf/nginx.conf。
> `--pid-path=path`: 设置nginx.pid文件的名字，该文件会用于存放主进程的进程ID。在安装之后，该文件的名字可以通过nginx.conf文件中的pid指令进行修改。默认情况下该文件会被命名为prefix/logs/nginx.pid
> `--error-log-path`=path: 设置主要的error,warnings及diagnostic文件的名字。在安装之后，该文件的名字也可以通过nginx.conf文件中的error_log指令进行修改。默认情况下，该文件会被命名为prefix/logs/error.log
> `--http-log-path`=path: 发送到HTTP Server的请求的日志文件名称。在安装之后，该文件的名称可以通过nginx.conf文件中的access_log指令进行修改。默认情况下，该文件会被命名为prefix/logs/access.log
> `--build=name`: 设置nginx构建出来后的名字（可选项）
> `--user=name`: 设置nginx工作进程的所属用户。在安装之后，该名字也可以通过nginx.conf文件中的user指令进行修改。默认的用户名为nobody
> `--group=name`: 设置nginx工作进程的所属组。在安装之后，该名字也可以通过nginx.conf文件中的user指令进行修改。默认情况下会被设置为非特权用户的用户名称
> `--with-select_module/without-select_module`: 使能/禁止一个模块使用select()方法。这个模块会被自动的编译构建假如该平台并不支持一些更合适的方法，例如kqueue，epoll，/dev/poll
> `--with-poll_module/without-poll_module`: 使能/禁止一个模块使用poll()方法。该模块会被自动的编译构建假如该平台并不支持一些更合适的方法，例如kqueue，epoll,/dev/poll。
> `--without-http_gzip_module`: 禁止构建压缩响应的模块。要想构建及运行此模块，必须依赖与zlib库。
> `--without-http_rewrite_module`: 禁止构建允许HTTP Server进行请求重定向及改变请求URI的模块。要构建此模块的话需要PCRE库的支持。
> `--without-http_proxy_module`: 禁止构建HTTP Server proxying模块
> `--without-http_ssl_module`: 构建HTTP Server对https协议的支持。默认情况下该模块并不会被构建。构建和运行本模块需要OpenSSL的支持
> `--with-pcre=path`: 设置PCRE库原文件的路径。该库的发布版本(version 4.4 – 8.40)需要从PCRE官方网站下载然后解压。剩余的操作则由nginx的./configure和make完成。在location指令中的表达式匹配及ngx_http_rewrite_module模块需要依赖该库。
> `--with-pcre-jit`: 构建支持“just-in-time compilation”特性的PCRE库。
> `--with-zlib=path`: 设置zlib库源文件的路径。该库的发布版本(version 1.1.3 - 1.2.11)需要从zlib官方网站下载然后解压。剩余的操作是由nginx的./configure及make来完成。ngx_http_gzip_module模块需要依赖于该库。
> `--with-cc-opt=parameters`: 设置一些额外的参数，这些参数会被添加到CFLAGS变量后。当在FreeBSD系统下使用系统PCRE库的时候，--with-cc-opt=”-I /usr/local/include” 应该被指定。假如需要指定select()函数支持的文件句柄数也可以通过这样指定：--with-cc-opt=”-D FD_SETSIZE=2048”。
> `--with-ld-opt=parameters`: 设置链接时候的一些额外的参数。当在FreeBSD系统下使用PCRE库时，应该指定--with-ld-opt=”-L /usr/local/lib”。
> `--with-debug`: 将Nginx需要打印debug调试级别日志的代码编译进Nginx。这样可以在Nginx运行时通过修改配置文件来使其打印调试日志，这对于研究、定位Nginx问题非常有帮助

- 配置

```bash
./configure --prefix=/usr/local/nginx-1.16.1 --sbin-path=/usr/local/nginx-1.16.1/sbin/nginx --conf-path=/usr/local/nginx-1.16.1/conf/nginx.conf --pid-path=/usr/local/nginx-1.16.1/nginx.pid --error-log-path=/usr/local/nginx-1.16.1/logs/error.log --http-log-path=/usr/local/nginx-1.16.1/logs/access.log --with-http_ssl_module --with-debug
```

- 编译

```bash
make
make install
```

- 启动

```bash
/usr/local/nginx-1.16.1/sbin/nginx -c /usr/local/nginx-1.16.1/conf/nginx.conf
```

- 停止

```bash
/usr/local/nginx-1.16.1/sbin/nginx -s quit
```

- 设置环境变量，nginx全局可用

```bash
vi /etc/profile
```

```bash
export NGINX_HOME=/usr/local/nginx-1.16.1
export PATH=${PATH}:${NGINX_HOME}/sbin
```

```bash
source /etc/profile
```

##### 将nginx配置成服务(基于centos7)

- 创建service文件

```bash
vi /lib/systemd/system/nginx.service
```

```bash
# 基础信息
[Unit]
# 描述
Description=Nginx
# 在某个服务启动后启动
After=network.target

# 服务信息
[Service]
Type=forking
# 启动服务的命令
ExecStart=/usr/local/nginx-1.16.1/sbin/nginx -c /usr/local/nginx-1.16.1/conf/nginx.conf
# 终止服务的命令
ExecStop=/usr/local/nginx-1.16.1/sbin/nginx -s quit

# 安装相关信息
[Install]
# WantedBy 是以哪种方式启动：multi-user.target表明当系统以多用户方式（默认的运行级别）启动时，这个服务需要被自动运行。
WantedBy=multi-user.target
```

- 设置可执行权限

```bash
chmod +x /lib/systemd/system/nginx.service
```

- 刷行配置

```bash
systemctl daemon-reload
```

- 启动服务

```bash
systemctl start nginx.service
```

- 查看服务

```bash
systemctl status nginx.service
```
