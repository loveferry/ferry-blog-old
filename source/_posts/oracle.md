---
title: oracle
date: 2018-12-17 18:44:30
categories: 数据库
tags:
    - docker中安装oracle
    - oracle数据迁移
---

&emsp;&emsp;Oracle数据库是目前世界上使用最为广泛的数据库管理系统，作为一个通用的数据库系统，它具有完整的数据管理功能；作为一个关系数据库，它是一个完备关系的产品；作为分布式数据库它实现了分布式处理功能。

<!-- more -->

### 安装oracle

#### 基于centOS 7 安装oracle12c

##### 依赖安装，环境准备

&emsp;&emsp;**以下操作请登录 root 用户执行**。

- 安装依赖

```bash
yum -y install binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33.i686 gcc gcc-c++ glibc glibc.i686 glibc-devel glibc-devel.i686 ksh libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel libstdc++-devel.i686 libaio libaio.i686 libaio-devel libaio-devel.i686 ibXext ibXext.i686 libX11 libX11.i686 libxcb libxcb.i686 libXi libXi.i686 make sysstat
```

- 修改系统内核参数

```bash
vi /etc/sysctl.conf
```

```bash
# 在文件末尾加以下几行
kernel.shmmni=4096
kernel.sem=250 32000 100 128
fs.file-max=6815744
fs.aio-max-nr=1048576
net.ipv4.ip_local_port_range=9000 65500
net.core.rmem_default=262144
net.core.rmem_max=4194304
net.core.wmem_default=262144
net.core.wmem_max=1048576
```

- 使修改的内核参数生效

```bash
sysctl -p
```

- 创建用户和组

```bash
groupadd oinstall  
groupadd dba  
groupadd oper  
useradd -g oinstall -G dba,oper oracle
```

- 设置 oracle 用户密码

```bash
passwd oracle
```

- 限制资源参数

```bash
vi /etc/pam.d/login
```

```bash
# 在文件末尾加以下几行
session    required     pam_selinux.so open
session    required     pam_namespace.so

session    required     pam_limits.so

session    optional     pam_keyinit.so force revoke
session    include      system-auth
session   optional     pam_ck_connector.so
```

- 修改用户限制

```bash
vi /etc/security/limits.conf
```

```bash
# 在文件末尾加以下几行
oracle  soft  nproc   2047
oracle  hard  nproc   16384
oracle  soft  nofile  1024
oracle  hard  nofile  65536
oracle  soft  stack   10240
oracle  hard  stack   32768
```

- 创建交换空间

&emsp;&emsp;若系统已有交换空间且大于8G，则这一步不用执行，oracle 大小大概在7.*G,若交换空间小于所需空间，安装时会安装失败。

```bash
dd if=/dev/zero of=/var/swap bs=1024 count=8388608
mkswap /var/swap
chmod 0600 /var/swap
swapon /var/swap
```

- 系统启动时自动启用交换分区

```bash
vi /etc/fstab
```

```bash
/var/swap swap swap default 0 0
```

- 创建目录

```bash
mkdir -p /u01/app
```

- 修改目录属主和属组

```bash
chown -R oracle:oinstall /u01/app
```

&emsp;&emsp;**以下操作请登录 oracle 用户执行**。

- 配置环境

```bash
vi ~/.bash_profile
```

```bash
umask 022
export ORACLE_BASE=/u01/app/oracle
```

- 配置立即生效

```bash
source ~/.bash_profile
```

- 下载安装文件

```bash
wget -b -c -O linuxx64_12201_database.zip https://download.oracle.com/otn/linux/oracle12c/122010/linuxx64_12201_database.zip?AuthParam=1581938083_d20f5a2be1ce563b28554044efc3c94d
```

- 解压

```bash
unzip linuxx64_12201_database.zip
```

&emsp;&emsp;这里选择 oracle12c 的linux 64位的版本。注意，官网下载需要登录，先登录，然后点击下载，在浏览器中复制下载链接使用wget进行下载，`-b`参数表明wget进行后台下载，`-c`参数表明断点续传，`-O`参数为下载的文件命名。`AuthParam`参数有时效性，是oracle官网登录后点击下载然后生成的，所以这个下载链接需要自己去生成。

![查找](/oracle/oracle-install-linux.png)

- 解压

```bash
unzip linuxx64_12201_database.zip
```

##### 通过X11实现linux图形化界面本地显示-MAC

&emsp;&emsp;Windows环境使用xshell连接centos显示图形化界面可自行百度。mac下通过 Xquartz 协助完成图形化界面在本地显示的需求。

- mac下载Xquartz

&emsp;&emsp;去[官网](https://www.xquartz.org/)下载安装包并按照提示安装。

- 服务端设置

```bash
vi /etc/ssh/sshd_config
```

```bash
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost no
```

```bash
systemctl restart sshd.service
```

- mac设置

```bash
vi /private/etc/ssh/sshd_config
```

```bash
X11Forwarding yes
X11UseLocalhost no
```

&emsp;&emsp;重启mac。

##### 安装数据库软件

- 登录oracle用户

```bash
ssh -X oracle@ip
```

&emsp;&emsp;使用 `-X` 即使用 XQuartz 。

- 尝试安装

```bash
cd tmp/
./database/runInstaller
```

&emsp;&emsp;若出现下面错误，则登录root用户检查`/usr/bin/xdpyinfo`是否存在。

![查找](/oracle/oracle-install-test.png)

- 下载xdpyinfo并设置xhost

```bash
yum -y install xdpyinfo
export DISPLAY=:1.0
xhost +
```

&emsp;&emsp;若提示xhost找不到，则安装所以依赖。查找缺失依赖`yum whatprovides "*/xhost"`,安装依赖`yum -y install ***`,安装完成再执行`xhost +`。若报错`unable to open display ""`，则先安装vnc服务，再设置xhost。

- vnc服务安装

&emsp;&emsp;终端输入`vncserver`测试是否安装了vnc服务，若未安装则安装vnc。

```bash
yum -y install tigervnc-server
```

```bash
vncserver
```

&emsp;&emsp;按提示键入用户名密码。

- 设置 xhost

```bash
export DISPLAY=localhost:1
xhost +
```

&emsp;&emsp;设置好xhost后退出root用户登录oracle用户`ssh -X oracle@ip`。

- 设置 DISPLAY

```bash
export DISPLAY=localhost:1.0
```

- 尝试安装

```bash
cd tmp/
./database/runInstaller
```

&emsp;&emsp;出现下图，那基本上就是可以，稍等一会，安装界面就会在本地显示了。若弹出的界面中文乱码了，最简单的方式就是设置零时环境变量，将语言设置成英文`export LANG=en_US.UTF-8`。也有可能会出现一切正常但是安装画面并未显示的情况，此时我们退出重新登录oracle用户，查看DISPLAY变量是否有值，若有值可以先不设置临时值，直接运行脚本看看能否显示安装界面。

![查找](/oracle/oracle-install-run.png)

&emsp;&emsp;下面就是正常的安装了。

![查找](/oracle/oracle-install-1.png)

![查找](/oracle/oracle-install-2.png)

![查找](/oracle/oracle-install-3.png)

![查找](/oracle/oracle-install-4.png)

![查找](/oracle/oracle-install-5.png)

![查找](/oracle/oracle-install-6.png)

![查找](/oracle/oracle-install-7.png)

![查找](/oracle/oracle-install-8.png)

![查找](/oracle/oracle-install-9.png)

&emsp;&emsp;这一步报错了，那么我们根据他的错误提示解决就好了。这里是说系统缺少一个工具，那么我们登录root用户安装一下`yum -y install smartmontools`,然后点击`Check Again`。

![查找](/oracle/oracle-install-10.png)

![查找](/oracle/oracle-install-11.png)

![查找](/oracle/oracle-install-12.png)

&emsp;&emsp;这一步按照他的提示去root用户下执行这两个文件。

```bash
sh /u01/app/oraInventory/orainstRoot.sh
```

```bash
sh /u01/app/oracle/product/12.2.0/dbhome_1/root.sh
```

&emsp;&emsp;这里会让你输入服务器的bin目录，默认是`/usr/local/bin`，然后会让你确认是否安装Oracle跟踪文件分析器，这里输入yes。两个脚本执行完成后，在安装界面点击OK，继续下一步。

![查找](/oracle/oracle-install-13.png)

&emsp;&emsp;到这里，数据库软件就安装完成了。

- 配置环境变量

```bash
vi ~/.bash_profile
```

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1

export PATH=$PATH:$ORACLE_HOME/bin

export ORACLE_SID=orcl
```

```bash
source ~/.bash_profile
```

- 创建监听

&emsp;&emsp;退出当前登录并重新登录oracle用户。

```bash
export LANG=en_US.UTF-8
netca
```

![查找](/oracle/oracle-install-14.png)

![查找](/oracle/oracle-install-15.png)

![查找](/oracle/oracle-install-16.png)

![查找](/oracle/oracle-install-17.png)

![查找](/oracle/oracle-install-18.png)

![查找](/oracle/oracle-install-19.png)

![查找](/oracle/oracle-install-20.png)

&emsp;&emsp;此时，我们的监听就创建完成了，在弹出的页面点击finish就可以退出了。

- 创建数据库

&emsp;&emsp;退出当前登录并重新登录oracle用户。

```bash
export LANG=en_US.UTF-8
dbca
```

![查找](/oracle/oracle-install-21.png)

![查找](/oracle/oracle-install-22.png)

&emsp;&emsp;这里填入管理员，例如sys/system用户的密码，勾选创建容器服务器，并设置容器名称。

&emsp;&emsp;接下来一路默认就好了。


&emsp;&emsp;登录oracle用户，启动监听和容器。


- 启动监听


```bash
lsnrctl start
```

- 启动容器

```bash
sqlplus sys as sysdba
```

```sql
startup
```

#### docker中安装oracle

##### 查找并拉取镜像

&emsp;&emsp;我们去 [hub.docker.com](hub.docker.com) 上搜索 [oracle](https://hub.docker.com/_/oracle-database-enterprise-edition) ，选择类型为database，如下图，这个镜像是oracle官方提供的，我们就选择这个好了。

![查找](/oracle/oracle-docker-hub.png)

&emsp;&emsp;不要说话，照着做就完事了，官方镜像，逼格高点，需填写一些东西。要先登录docker hub，没账号的自己建一个。

![查找](/oracle/oracle-show-proceed.png)

![查找](/oracle/oracle-docker-info.png)

&emsp;&emsp;复制去下载镜像，terminal中可能还需要docker login一下，得用你docker hub中login的账号。镜像下载速度看几个方面，一个就是你docker的源用的哪里的，一个就是你的网速，网的稳定性。

&emsp;&emsp;左边的是官网友情提供的一些说明，比如连接的一些基本参数，如何指定一个外部配置文件，怎么连接这个容器。一个很重要的信息是sys管理员的密码是`Oradoc_db1`

![查找](/oracle/oracle-docker-show-detail.png)

##### 运行镜像

- 查看我们刚才拉取的镜像

```bash
docker images
```

![运行结果](/oracle/docker-image.png)

- 运行

```bash
docker run -d -it --name oracle12.2.0.1 -p 1521:1521 12a359cd0528
```

> 参数解释
> run:run命令就是用来运行镜像文件的呀...通常使用docker run 镜像名称或者镜像的IMAGE ID 组成来运行镜像
> -d:后台运行
> --name:重命名运行镜像生成的容器
> -p:端口映射,如上面的,将容器的1521端口映射到宿主机的1521端口,将容器的8080端口映射到宿主机的8081端口
> -v:可以将容器的指定目录映射到宿主机的指定目录,这个操作很骚气,很多人喜欢,但是我觉得没有必要啊,防止误删容器数据丢失??你别瞎几把删除容器不就好了...
> -it:-i以交互模式运行容器,-t为容器重新分配一个伪输入终端.这两个指令通常是搭配在一起使用.使用这个命令就是在生成容器之后模拟出来一个终端让你可以对这个容器进行操作,当然,像oracle,使用-it一般也需要在后面加上启动参数,选择以什么用户进入容器

&emsp;&emsp;如下图所示，容器启动需要时间,等到状态变成healthy时就启动好了。

![运行结果](/oracle/docker-start-image.png)

- 连接

```bash
docker exec -it oracle12.2.0.1 bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
```

![运行结果](/oracle/oracle-docker-conn.png)

&emsp;&emsp;用户名`sys`，密码`Oradoc_db1`。

```bash
docker exec -it oracle12.2.0.1 bash
```

&emsp;&emsp;这是直接访问容器的目录，导入导出，查看日志等操作可能用到。

### 数据库操作

#### 启动和停止监听与容器

- 启动监听

```bash
lsnrctl start
```

- 关闭监听

```bash
lsnrctl stop
```

- 查看状态

```bash
lsnrctl status
```

- 启动容器

```bash
sqlplus sys as sysdba
```

```sql
startup
```

- 关闭容器

```bash
sqlplus sys as sysdba
```

```sql
shutdown
```

#### 连接数据库

- 在oracle用户下执行：

```base
sqlplus sys/oracle as sysdba
```

#### 创建表空间，用户

&emsp;&emsp;以下操作登录sys/system用户执行。

- 创建表空间

```sql
CREATE TABLESPACE ferry DATAFILE '/u01/app/oracle/admin/ORCL/dpdump/ferry.dbf' SIZE 50M AUTOEXTEND ON;
```

&emsp;&emsp;on后面可以指定每次自增长空间大小,路径为绝对路径，不存在则报错。

- 创建用户并指定表空间

```sql
create user ferry identified by ferry default tablespace ferry;
```

- 用户授权

```sql
grant dba to ferry;
grant connect to ferry;
grant alter session to ferry;
grant create any context to ferry;
grant create procedure to ferry;
grant create sequence to ferry;
grant create session to ferry;
grant create synonym to ferry;
grant create table to ferry;
grant create type to ferry;
grant create user to ferry;
grant create view to ferry;
grant create any table to ferry;
grant DEBUG CONNECT SESSION to ferry;
grant query rewrite to ferry;
grant select any dictionary to ferry;
grant unlimited tablespace to ferry;
grant read,write on directory DATA_PUMP_DIR to ferry;
```

#### 删除用户，表空间

&emsp;&emsp;以下操作登录sys/system用户执行。

- 删除用户

```sql
drop user ferry cascade;
```

&emsp;&emsp;加上cascade是将用户的数据库数据一并删除，并没有删除相应的表空间。

- 删除表空间

```sql
DROP TABLESPACE ferry INCLUDING CONTENTS AND DATAFILES;
```

&emsp;&emsp;若删除的表空间是默认的表空间，则会提示不能删除默认表空间，需要先修改默认表空间然后才能删除。

- 查看并修改默认表空间

```sql
-- 查看默认表空间
select * from database_properties where property_name='DEFAULT_PERMANENT_TABLESPACE';
-- 修改默认表空间
alter database default tablespace USERS;
```

#### wm_concat

&emsp;&emsp;在oracle11g和12c上已经摒弃了wm_concat函数，但是我们的程序中可能依旧使用了这个函数，为了数据库的迁移省事，可以选择自己创建一个wm_concat函数。但是注意，即使创建了该函数，在使用的过程中，也需要用to_char(wm_concat())方式，才能完全替代之前的应用。

##### 方法一

&emsp;&emsp;以下操作登录sys/system用户执行。

- 解锁WMSYS用户

```sql
alter user wmsys account unlock;
```

- 授权WMSYS用户

```sql
grant dba to wmsys;
grant connect to wmsys;
grant alter session to wmsys;
grant create any context to wmsys;
grant create procedure to wmsys;
grant create sequence to wmsys;
grant create session to wmsys;
grant create synonym to wmsys;
grant create table to wmsys;
grant create type to wmsys;
grant create user to wmsys;
grant create view to wmsys;
grant create any table to wmsys;
grant DEBUG CONNECT SESSION to wmsys;
grant query rewrite to wmsys;
grant select any dictionary to wmsys;
grant unlimited tablespace to wmsys;
grant read,write on directory DATA_PUMP_DIR to wmsys;
```

&emsp;&emsp;以WMSYS用户连接数据库并执行下面的操作.

- 创建包，包体和函数

```sql
CREATE OR REPLACE TYPE WM_CONCAT_IMPL AS OBJECT
-- AUTHID CURRENT_USER AS OBJECT
(
CURR_STR VARCHAR2(32767), 
STATIC FUNCTION ODCIAGGREGATEINITIALIZE(SCTX IN OUT WM_CONCAT_IMPL) RETURN NUMBER,
MEMBER FUNCTION ODCIAGGREGATEITERATE(SELF IN OUT WM_CONCAT_IMPL,
P1 IN VARCHAR2) RETURN NUMBER,
MEMBER FUNCTION ODCIAGGREGATETERMINATE(SELF IN WM_CONCAT_IMPL,
RETURNVALUE OUT VARCHAR2,
FLAGS IN NUMBER)
RETURN NUMBER,
MEMBER FUNCTION ODCIAGGREGATEMERGE(SELF IN OUT WM_CONCAT_IMPL,
SCTX2 IN WM_CONCAT_IMPL) RETURN NUMBER
);
/
 
-- 定义类型body:
CREATE OR REPLACE TYPE BODY WM_CONCAT_IMPL
IS
STATIC FUNCTION ODCIAGGREGATEINITIALIZE(SCTX IN OUT WM_CONCAT_IMPL)
RETURN NUMBER
IS
BEGIN
SCTX := WM_CONCAT_IMPL(NULL) ;
RETURN ODCICONST.SUCCESS;
END;
MEMBER FUNCTION ODCIAGGREGATEITERATE(SELF IN OUT WM_CONCAT_IMPL,
P1 IN VARCHAR2)
RETURN NUMBER
IS
BEGIN
IF(CURR_STR IS NOT NULL) THEN
CURR_STR := CURR_STR || ',' || P1;
ELSE
CURR_STR := P1;
END IF;
RETURN ODCICONST.SUCCESS;
END;
MEMBER FUNCTION ODCIAGGREGATETERMINATE(SELF IN WM_CONCAT_IMPL,
RETURNVALUE OUT VARCHAR2,
FLAGS IN NUMBER)
RETURN NUMBER
IS
BEGIN
RETURNVALUE := CURR_STR ;
RETURN ODCICONST.SUCCESS;
END;
MEMBER FUNCTION ODCIAGGREGATEMERGE(SELF IN OUT WM_CONCAT_IMPL,
SCTX2 IN WM_CONCAT_IMPL)
RETURN NUMBER
IS
BEGIN
IF(SCTX2.CURR_STR IS NOT NULL) THEN
SELF.CURR_STR := SELF.CURR_STR || ',' || SCTX2.CURR_STR ;
END IF;
RETURN ODCICONST.SUCCESS;
END;
END;
/
--自定义行变列函数:
CREATE OR REPLACE FUNCTION wm_concat(P1 VARCHAR2)
RETURN VARCHAR2 AGGREGATE USING WM_CONCAT_IMPL ;
/
```

- 创建同义词并授权

```sql
create public synonym WM_CONCAT_IMPL for wmsys.WM_CONCAT_IMPL
/
create public synonym wm_concat for wmsys.wm_concat
/
 
grant execute on WM_CONCAT_IMPL to public
/
grant execute on wm_concat to public
/
```


##### 方法二

&emsp;&emsp;若容器中没有用户 wmsys，可以随便选择一个用户创建此函数，例如，这里使用ferry用户创建该函数。

&emsp;&emsp;以 ferry 用户连接数据库并执行下面的操作.

- 创建包，包体和函数

```sql
CREATE OR REPLACE TYPE WM_CONCAT_IMPL AS OBJECT
-- AUTHID CURRENT_USER AS OBJECT
(
CURR_STR VARCHAR2(32767), 
STATIC FUNCTION ODCIAGGREGATEINITIALIZE(SCTX IN OUT WM_CONCAT_IMPL) RETURN NUMBER,
MEMBER FUNCTION ODCIAGGREGATEITERATE(SELF IN OUT WM_CONCAT_IMPL,
P1 IN VARCHAR2) RETURN NUMBER,
MEMBER FUNCTION ODCIAGGREGATETERMINATE(SELF IN WM_CONCAT_IMPL,
RETURNVALUE OUT VARCHAR2,
FLAGS IN NUMBER)
RETURN NUMBER,
MEMBER FUNCTION ODCIAGGREGATEMERGE(SELF IN OUT WM_CONCAT_IMPL,
SCTX2 IN WM_CONCAT_IMPL) RETURN NUMBER
);
/
 
-- 定义类型body:
CREATE OR REPLACE TYPE BODY WM_CONCAT_IMPL
IS
STATIC FUNCTION ODCIAGGREGATEINITIALIZE(SCTX IN OUT WM_CONCAT_IMPL)
RETURN NUMBER
IS
BEGIN
SCTX := WM_CONCAT_IMPL(NULL) ;
RETURN ODCICONST.SUCCESS;
END;
MEMBER FUNCTION ODCIAGGREGATEITERATE(SELF IN OUT WM_CONCAT_IMPL,
P1 IN VARCHAR2)
RETURN NUMBER
IS
BEGIN
IF(CURR_STR IS NOT NULL) THEN
CURR_STR := CURR_STR || ',' || P1;
ELSE
CURR_STR := P1;
END IF;
RETURN ODCICONST.SUCCESS;
END;
MEMBER FUNCTION ODCIAGGREGATETERMINATE(SELF IN WM_CONCAT_IMPL,
RETURNVALUE OUT VARCHAR2,
FLAGS IN NUMBER)
RETURN NUMBER
IS
BEGIN
RETURNVALUE := CURR_STR ;
RETURN ODCICONST.SUCCESS;
END;
MEMBER FUNCTION ODCIAGGREGATEMERGE(SELF IN OUT WM_CONCAT_IMPL,
SCTX2 IN WM_CONCAT_IMPL)
RETURN NUMBER
IS
BEGIN
IF(SCTX2.CURR_STR IS NOT NULL) THEN
SELF.CURR_STR := SELF.CURR_STR || ',' || SCTX2.CURR_STR ;
END IF;
RETURN ODCICONST.SUCCESS;
END;
END;
/
--自定义行变列函数:
CREATE OR REPLACE FUNCTION wm_concat(P1 VARCHAR2)
RETURN VARCHAR2 AGGREGATE USING WM_CONCAT_IMPL ;
/
```

- 创建同义词并授权

```sql
create public synonym WM_CONCAT_IMPL for ferry.WM_CONCAT_IMPL
/
create public synonym wm_concat for ferry.wm_concat
/
 
grant execute on WM_CONCAT_IMPL to public
/
grant execute on wm_concat to public
/
```

#### 数据库导入导出

&emsp;&emsp;这里使用的是 impdp/expdp 命令进行导入导出的,需要在oracle用户下执行命令。

- 数据导出

```bash
expdp user_name/password@127.0.0.1:1521/ORCL directory=DATA_PUMP_DIR dumpfile=ferry.dmp [version=11.2.0.2.0]
```

&emsp;&emsp;用户名和密码就是你要导出的数据库所属的用户,IP地址和端口号是服务器的,服务名是你这个用户登录所使用的服务名,当然,我觉得这里如果你是用SID的,那么替换成SID我觉得也没问题,目录指定你要到处的dump的存放位置,这个我们一般使用oracle自带的那么目录,如下图,指定文件名就没什么好说的了,后缀别搞错就行了,最后面的版本号是可选的,当你将高版本的数据迁移到低版本时,你需要指定导出dump的版本,这样你在低版本的数据库中执行导入才不会有问题,但是,高版本导出低版本数据也是有可能发生错误的,比如你高版本oracle服务器端使用了高版本的特性,但是你指定的导出版本没有这个特性,那么就可能会报错啊(事实是,我指定低版本号执行导出确实有过报错,大概是数据类型过长还是什么的我忘了,以上导出版本冲突是我的猜想…).

![目录](/oracle/oracle-directories-1.png)

![目录](/oracle/oracle-directories-2.png)

&emsp;&emsp;上图1是通过客户端`oracle sql developer`登录你要导出的用户查看当前用户的目录访问权限并查看指定目录的系统路径,在导出成功后可以在此系统路径下找到导出的日志文件和dump.

&emsp;&emsp;上图2是通过客户端`PL/SQL`登录你要导出的用户查看当前用户的目录访问权限,在你想查看的目录上鼠标右键-edit查看该目录的系统路径.

&emsp;&emsp;你以为导出就到此结束了么,或者你以为导出成功拿到dump就结束了么??不!!!你得拿到你导出的这个dump的schema和tablespace信息,这个信息没拿到,你怎么执行导入操作呢?

- 获取导出数据的schema和表空间

&emsp;&emsp;schema一般情况下都是和用户一一对应的,所以我们这直接取导出数据的用户名就好了,表空间就需要在服务器上使用sqlplus连接该用户,执行`select default_tablespace from user_users;`查看当前用户的表空间,或者可以使用客户端连接登录该用户执行sql,多样化嘛,怎么方便怎么来.

- 数据导入

```bash
impdp user_name/password@127.0.0.1:1521/ORCL remap_schema=from_schema:to_schema remap_tablespace=from_tablespace:to_tablespace directory=DATA_PUMP_DIR dumpfile=ferry.dmp
```

#### 事务锁查询与杀死

&emsp;&emsp;查询与杀死操作在同一个session中处理。

- 查询

```sql
select sess.sid,
sess.serial#,
lo.oracle_username,
lo.os_user_name,
ao.object_name,
lo.locked_mode
from v$locked_object lo,
dba_objects ao,
v$session sess
where ao.object_id = lo.object_id and lo.session_id = sess.sid;
```

- 杀死

```sql
ALTER SYSTEM KILL SESSION 'SID,SERIR#';
```

#### PDB容器的操作

- **查看PDB容器**

```sql
show pdbs
```

- **启动PDB容器**

```sql
alter pluggable database pdborcl1 open;
```

- **关闭PDB容器**

```sql
alter pluggable database pdborcl1 close;
```

- **切换会话**

```sql
alter session set container=odborcl1;
```
