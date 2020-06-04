---
title: mysql
date: 2018-12-17 18:43:35
categories: 数据库
tags:
	- mysql
	- limit
	- case
	- 创建用户
	- 分配权限
	- 安装mysql5.7
---

&emsp;&emsp;记录一下mysql中常用的语法吧，也算是通用的sql语法。

<!-- more -->

### 建库建表

* 创建一个新的数据库

```sql
CREATE DATABASE DEMO DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

* 建表语句

```sql
CREATE TABLE user (
  id bigint(20) NOT NULL AUTO_INCREMENT COMMENT "自增长，不为空",
  number varchar(50) NOT NULL COMMENT "编号",
  name varchar(50) NOT NULL COMMENT "可以超过255个字符，存储变长，节省存储空间",
  age tinyint COMMENT "-128~127",
  money float(10,1) COMMENT "单精度浮点型，m总个数，d小数位，容易造成精度丢失",
  height double(10,1) COMMENT "双精度浮点型，m总个数，d小数位",
  weight double COMMENT "可以不设置m,d",
  phone decimal(10,1) COMMENT "高精度的数据类型，常用来存储交易相关的数据，DECIMAL(M,N).M代表总精度，N代表小数点右侧的位数。1 < M < 254, 0 < N < 60;存储空间变长",
  born_date date COMMENT "精确到年月日",
  register_date datetime COMMENT "精确到年月日时分秒",
  description text COMMENT "总大小为65535字节，约为64KB",
  constraint pk_user_id_and_number primary key (id,number),
  constraint fk_user_height_and_weight foreign key(height,weight) references other_table_name(major_key_one,major_key_two),
  constraint uk_user_name unique key(name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT "表注释";
```

* 显示建表语句

```sql
show create table user;
```

* 查询表结构

```sql
desc user;
```

* 修改表名

```sql
alter table user rename new_user;
```

* 如果存在则删除表

```sql 
drop table if exists user;
```

### 常用关键字的用法

#### limit的用法

```sql
select * from user limit 5;
select * from user limit 2,3;
```

&emsp;&emsp;当`limit`后面只有一个参数的时候，表示返回最大的记录行数。当`limit`后面有两个参数的时候，第一个参数表示偏移量，第二个参数表示返回记录行的最大的数目

#### distinct的用法

* 查询id不同的记录有多少条

```sql
select count(distinct id) from user;
```

* 查询id和name同时不相同的记录

```sql
select distinct id,name from user;
```

#### case的用法

* 第一种用法

```sql
select id, name,
	case 
		when sex = 'male' then '男'
		when sex = 'female' then '女'
		else '人妖' 
	end as sex
from user;
```

&emsp;&emsp;这种用法还可以在`when`中使用`and`、`or`等关键字拼接多个条件对进行筛选

* 第二种用法

```sql
select id, name,
	case sex
		when 'male' then '男'
		when 'female' then '女'
		else '人妖' 
	end as sex
from user;
```

&emsp;&emsp;值得一提的是，`CASE`函数只返回第一个符合条件的结果，所以在使用的时候需要注意`WHEN`条件的返回。

### 用户操作

#### 创建用户

* 创建用户

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

* 修改密码

```sql
alter user 'root'@'localhost' identified by 'ferry';
flush privileges;
```

&emsp;&emsp;`host`是IP地址，可以指向某一个ip地址，也可以使用`%`来表示所有的ip地址都可以通过该用户名密码连接该数据库。密码可以为空，如果密码给空字符串，则登录该用户不需要密码（不推荐）。

#### 授权

&emsp;&emsp;为用户授权的语法：grant 权限 on 数据库.表名 to '用户名'@'登录主机' IDENTIFIED BY '密码';

* 赋予用户某个数据库全部权限并刷新系统权限

```sql
grant all privileges on demo.* to 'ferry'@'%' IDENTIFIED BY 'ferry';
flush privileges;
```

&emsp;&emsp;`all privileges`是所有权限的意思，这条语句的意思是将`demo`数据库的所有表的所有权限赋予密码为`ferry`，用户名为`ferry`的用户，可以通过任意IP地址通过该用户连接数据库。

* 赋予用户部分权限并刷新系统权限

```sql
grant select,delete,update,create,drop on demo.user to 'ferry'@'localhost' IDENTIFIED BY 'ferry';
flush privileges;
```

&emsp;&emsp;这条语句的意思是将`demo`数据库的`user`的crud权限赋予密码为`ferry`，用户名为`ferry`的用户，该用户只能在本地连接数据库。

* 超级权限

```sql
grant all privileges on demo.* to 'ferry'@'%' IDENTIFIED BY 'ferry' WITH GRANT OPTION;
flush privileges;
```

&emsp;&emsp;`ferry`用户可以赋权给其他用户。

#### 删除用户

* 删除用户及其权限

```sql
drop user 'ferry'@'%';
```

### 基于centOS安装mysql5.7

#### yum安装

- 更新yum

```bash
yum update
```

- 安装wget（已安装忽略）

```bash
yum -y install wget
```

- 查看Linux发行版本（其实没必要，看一下自己的系统版本而已。。）

```bash
cat /etc/redhat-release
```

- 获取MySQL官方的Yum Repository

```bash
wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```

&emsp;&emsp;这里要bb一句，下载链接是从mysql官网找到粘贴过来的，用点心，注意官网的yum repository分类下面找下载链接

- 查看文件

```bash
ls
```

![查看结果](/mysql/mysql-ls-result.png)

- 安装mysql的Yum Repository

```bash
yum -y install mysql*
```

&emsp;&emsp;因为该目录下只有一个以mysql开口的文件名，所以在这里我直接使用`*`模糊查询。

- 安装MySQL的服务器版本

```bash
yum -y install mysql-community-server
```

- 启动mysql服务

```bash
systemctl start mysqld.service
```

- 查看服务状态

```bash
systemctl status mysqld.service
```

![查看状态](/mysql/mysql-install-service-status.png)

&emsp;&emsp;很显然，服务已经正常的启动了，接下来就是登陆mysql了，使用yum安装mysql并启动服务后，MySQL进程会自动在进程日志中打印root用户的初始密码。

- 获取mysql初始密码

```bash
grep "password" /var/log/mysqld.log
```

![获取初始密码](/mysql/mysql-init-password.png)

- 登陆并修改密码

```sql
alter user 'root'@'localhost' identified by 'ferry';
```

&emsp;&emsp;直接这样运行，那么肯定报错了，当你使用日志中的密码登陆时会要求你修改密码，在你修改密码之前，你所能做的操作真的少得可怜。同时这个密码的强度默认是中等的，具体这个中等强度有那些要求我不太清楚，反正密码我没设置成功过。想设置简单一点的密码就是降低密码强度。还需要注意的是因为root用户的ip地址是`localhost`，所以只能指向localhost。

- 降低密码强度并设置密码最低位数

```sql
set global validate_password_policy=0;
set global validate_password_length=4;
```

- 修改密码并刷新权限

```sql
alter user 'root'@'localhost' identified by 'ferry';
flush privileges;
```

&emsp;&emsp;如果报错，请使用：

```sql
set password for ferry@localhost=password('ferry');
```

#### 二进制文件安装

##### 环境准备

&emsp;&emsp;在[mysql官网](www.mysql.com)点击`MySQL Community Server`-`Looking for previous GA versions?`选择指定的版本的数据库。如下选择的是GA版5.7.28的linux通用版64位的进行下载。

![选择二进制文件](/mysql/mysql-install-select.png)

&emsp;&emsp;下文安装选择mysql8.0进行安装。

- MD5校验，确保软件包可用

```bash
md5sum mysql-8.0.18-linux-glibc2.12-x86_64.tar.xz
```

- SELinux和系统防火墙关闭

```bash
systemctl stop firewalld.service
vi /etc/sysconfig/selinux
SELINUX=disabled
```

&emsp;&emsp;修改SELinux需要重启系统生效。安全增强型 Linux（Security-Enhanced Linux）简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。SELinux 主要作用就是最大限度地减小系统中服务进程可访问的资源（最小权限原则）。

- I/O调度系统使用deadline模式


1. Linus Elevator: 在2.4 内核中它是第一种I/O调度器。它的主要作用是为每个设备维护一个查询请求，当内核收到一个新请求时，如果能合并就合并。如果不能合并，就会尝试排序。如果既不能合并，也没有合适的位置插入，就放到请求队列的最后。

2. Anticipatory: Anticipatory的中文含义是"预料的，预想的"，顾名思义有个I/O发生的时候，如果又有进程请求I/O操作，则将产生一个默认的6毫秒猜测时间，猜测下一个进程请求I/O是要干什么的。这个I/O调度器对读操作优化服务时间，在提供一个I/O的时候进行短时间等待，使进程能够提交到另外的I/O。Anticipatory算法从Linux 2.6.33版本后被删除了，因为使用CFQ通过配置也能达到Anticipatory的效果。

3. DeadLine: Deadline翻译成中文是截止时间调度器，是对Linus Elevator的一种改进，它避免有些请求太长时间不能被处理。另外可以区分对待读操作和写操作。DEADLINE额外分别为读I/O和写I/O提供了FIFO队列。

4. CFQ: CFQ全称Completely Fair Scheduler ，中文名称完全公平调度器，它是现在许多 Linux 发行版的默认调度器，CFQ是内核默认选择的I/O调度器。它将由进程提交的同步请求放到多个进程队列中，然后为每个队列分配时间片以访问磁盘。对于通用的服务器是最好的选择,CFQ均匀地分布对I/O带宽的访问。CFQ为每个进程和线程,单独创建一个队列来管理该进程所产生的请求,以此来保证每个进程都能被很好的分配到I/O带宽，I/O调度器每次执行一个进程的4次请求。该算法的特点是按照I/O请求的地址进行排序，而不是按照先来后到的顺序来进行响应。简单来说就是给所有同步进程分配时间片，然后才排队访问磁盘

5. NOOP: NOOP全称No Operation,中文名称电梯式调度器，该算法实现了最简单的FIFO队列，所有I/O请求大致按照先来后到的顺序进行操作。NOOP实现了一个简单的FIFO队列,它像电梯的工作主法一样对I/O请求进行组织。它是基于先入先出（FIFO）队列概念的 Linux 内核里最简单的I/O 调度器。此调度程序最适合于固态硬盘。

&emsp;&emsp;I/O调度器的选择: 目前主流Linux发行版本使用三种I/O调度器：DeadLine、CFQ、NOOP，通常来说Deadline适用于大多数环境,特别是写入较多的文件服务器，从原理上看，DeadLine是一种以提高机械硬盘吞吐量为思考出发点的调度算法，尽量保证在有I/O请求达到最终期限的时候进行调度，非常适合业务比较单一并且I/O压力比较重的业务，比如Web服务器，数据库应用等。CFQ 为所有进程分配等量的带宽,适用于有大量进程的多用户系统，CFQ是一种比较通用的调度算法，它是一种以进程为出发点考虑的调度算法，保证大家尽量公平,为所有进程分配等量的带宽,适合于桌面多任务及多媒体应用。NOOP 对于闪存设备和嵌入式系统是最好的选择。对于固态硬盘来说使用NOOP是最好的，DeadLine次之，而CFQ效率最低。

- 查看当前系统支持的I/O调度器

```bash
dmesg | grep -i scheduler
```

![显示](/mysql/mysql_install_io_show.png)

&emsp;&emsp;当前I/O调度器是`deadline`。

- 查看某块硬盘的IO调度算法I/O调度器

```bash
cat /sys/block/sda/queue/scheduler
```

&emsp;&emsp;如果sda目录不存在，就找vda目录。


- swap分区的设置

&emsp;&emsp;不建议设置，或者设置4G。

- 文件系统的选择

&emsp;&emsp;建议使用xfs文件系统，相比ext4，它更加方便管理，支持动态扩容，删除文件也很方便。

- 操作系统的限制

```bash
ulimit -a
```

![查看参数](/mysql/mysql_os_ulimit.png)

&emsp;&emsp;open files如果设置不合理，那么当服务器连接过多或者表过多时就可能访问不了或者打不开表；max user processes参数的用途是，有时候我们可能会跑多实例，但是创建不了新的连接，报出"resource temporarily unavailable"的错误，表示没有足够的资源。

- 修改系统限制，编辑`/etc/security/limits.conf`(重启系统)

```bash
*   soft    nproc   65535
*   hard    nproc   65535
*   soft    nofile  65535
*   hard    nofile  65535   
```

- 关闭numa

&emsp;&emsp;关闭numa功能，能够更好的分配内存，不需要采用swap的方式获取内存。

```bash
numa --interleave=all /usr/local/mysql/bin/mysqld_safe -defaults-file=/etc/my.cnf &
```

##### 用户，目录设置

- 创建用户，用户组

```bash
groupadd mysql
useradd -g mysql mysql -s /sbin/nologin
```

- 解压软件包

```bash
tar -zxvf mysql-8.0.18-linux-glibc2.12-x86_64.tar.gz
```

&emsp;&emsp;如果解压出错，但是之前md5校验没问题，那么大概率是我们的系统缺少什么依赖没有安装，按照shell提示的错误信息解决即可。

- 软连接，方便日后升级

```bash
ln -s mysql-8.0.18-linux-glibc2.12-x86_64 mysql
```

- 授权

```bash
chown mysql:mysql -R mysql
```

- 数据目录

```bash
mkdir -p /data/mysql
chown mysql:mysql -R /data/mysql
```

- 配置mysql环境变量

```bash
cd /etc/profile.d
vi mysql.sh
```

```bash
export PATH=$PATH:/usr/local/mysql/bin
```

```bash
chown mysql:mysql mysql.sh
```

```bash
. /etc/profile.d/mysql.sh
```



- 配置文件`vi /etc/my.cnf`

```bash

```



##### 启动数据库


- 初始化数据库

```bash
cd /usr/local/mysql/bin
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql/ --user=mysql --initialize
```

&emsp;&emsp;`defaults-file`指定mysql的配置文件,`basedir`指定mysql的安装目录,`datadir`指定mysql的数据目录,`user`指定用户,`initialize`会在数据目录下生成一个文件log-error,记录零时的数据库初始化密码，如果是`initialize-insecure`代表无密码进入。

- 启动数据库

```bash
cd /usr/local/mysql/bin/
./mysqld_safe --defaults-file=/etc/my.cnf &
```

- 登陆

```bash
mysql -u root -p -P 3306 -h localhost
```

### 修改mysql配置文件

- 查看mysql服务默认读取的的my.cnf文件

```bash
mysql --help|grep 'my.cnf'
```

#### 查询缓存

&emsp;&emsp;查询缓存只能缓存静态的数据，数据仓库之类可能考虑使用查询缓存，生产建议关闭。Mysql5.6之前查询缓存默认开启，之后默认关闭，8.0版本之后删除了查询缓存功能。

- 查看查询缓存参数

```mysql
show variables like "%query_cache%";
```

![查询结果](/mysql/mysql_query_cache_variables.png)

- 关闭查询缓存(vi /etc/my.cnf)重启服务

```bash
[mysqld]
query_cache_type=0
query_cache_size=0
```

#### 设置时区

&emsp;&emsp;mysql中默认使用的SYSTEM时区，即EST时区，EST时区要比北京时间（东八区）慢13个小时。我们一般使用北京时间也就是东八区。

- 查询时区

```sql
show variables like '%time_zone%';
```

- 设置时区(vi /etc/my.cnf)重启服务

```bash
[mysqld]
default-time_zone = '+8:00'
```

&emsp;&emsp;未完待续。。。。。
