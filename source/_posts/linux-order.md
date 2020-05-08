---
title: linux常用指令
date: 2018-12-17 18:42:29
categories: linux
tags: 
    - yum
    - locate
---

&emsp;&emsp;linux常用的指令，用到一个记一个！！！

<!-- more -->

### yum

&emsp;&emsp;Yum是一个用来安装软件的软件，使用Yum指令可以安装以`.rpm`结尾的软件包，实际上很多软件包又依赖于其他软件包，众多的软件包之间的依赖关系复杂，使用rpm指令安装略显复杂，所以有了Yum的诞生，使用Yum可以很方便的安装你所需要的软件包，他会自动下载安装目标软件需要的依赖。

#### 查询软件包

- 根据关键字查询可安装的软件包

````bash
yum search mysql
````

![运行结果](/linux-order/linux-yum-search.png)

- 根据关键字检索系统中已安装的软件

```bash
yum list installed mysql-community-client
```

![运行结果](/linux-order/linux-yum-list.png)

- 结合通配符检索系统中已安装的软件

```bash
yum list installed mysql-community*
```

![运行结果](/linux-order/linux-yum-list-i.png)

&emsp;&emsp;很奇怪，我使用`yum list installed mysql*`进行检索就没有找到，多打几个字可以检索到软件，不懂。。

- 显示所有可更新的软件

```bash
yum list updates
```

![运行结果](/linux-order/linux-yum-list-updates.png)

- 显示详细信息

```bash
yum info mysql-community-client*
```

![运行结果](/linux-order/linux-yum-info.png)

#### 安装软件包

&emsp;&emsp;使用yum指令进行软件的安装，可以使用`-y`（安装过程中提示全部选择'y'），`-p`（不显示安装过程）或者`-h`（帮助）。

- 安装

```bash
yum -y install mysql-community-server
```

#### 更新软件

- 更新全部软件

```bash
yum update
```

- 更新指定软件

```bash
yum update mysql-community-client
```

#### 移除软件包

- 移除

```bash
yum remove mysql-community-client
```

#### 清除缓存（/var/cache/yum）

- 清除软件包

```bash
yum clean packages
```

- 清除headers

```bash
yum clean headers
```

- 一步到位直接清除所有

```bash
yum clean all
```

### 查看网卡信息

#### linux

&emsp;&emsp;[查看网卡信息](https://blog.51cto.com/13150617/1963833)

### locate

- 安装

```bash
yum -y install mlocate
```

- 更新索引数据库

```bash
updatedb
```

- 搜索

```bash
locate -i /etc/mysql
```

&emsp;&emsp;搜索etc目录下的包含mysql的目录或者文件，不区分大小写，去掉-i区分大小写。

- mac下更新索引数据库

```bash
sudo /usr/libexec/locate.updatedb
```

&emsp;&emsp;mac下`locate`的updatedb指令在这个位置,所以直接运行这个指令就好了,使用`sudo`命令扩大权限,使得索引数据库更加完善.

&emsp;&emsp;为了方便起见,不至于每次更新数据库都执行这么一长串命令,可以设置别名.

- 设置别名

```bash
echo 'alias updatedb="sudo /usr/libexec/locate.updatedb"' >> ~/.bash_profile
```
&emsp;&emsp;此时直接使用`updatedb`即可执行更新索引数据库的操作

### MAC本地使用ssh连接服务器

&emsp;&emsp;服务器端设置:修改配置文件`/etc/ssh/sshd_config`,将`PubkeyAuthentication`设置为yes，允许使用公密钥登陆，将`PasswordAuthentication`设置为yes，允许使用密码登陆

- 一般指令

```bash
ssh user@ip
```

&emsp;&emsp;端口号默认是22，如果服务器改了端口号，那么需要显式指定端口`ssh -p port user@ip`，会车之后会要求输入密码，ssh认证完成之后如果是第一次连接，那么会弹出提示，大意是第一次连接这个服务器，问你要不要继续连接，当输入`yes`后会将这个服务器的信息保存到`~/.ssh/known_hosts`文件中，下次登陆该服务器就不会有提示了。

&emsp;&emsp;采用别名登陆：本地的`~/.ssh/config`文件(若无则新建文件)中添加以下内容。

```bash
Host ferry
HostName ip
Port 22
User root
IdentityFile    /usr/local/aliyun/SSH-ferry-aliyun-private
```

&emsp;&emsp;`Host`设置服务器的别名，配置好后可以通过别名来进行登陆;`HostName`为IP地址;`Port`为端口号，ssh的端口号默认为22;`User`为要登陆的用户;`IdentityFile`为密钥的绝对路径,如果没有给定密钥路径，连接时会要求输入密码。

- 连接指令

```bash
ssh ferry
```

### 用 Sublime Text 编辑 SSH 远程文件(MAC)

&emsp;&emsp;本地的`~/.ssh/config`文件中添加以下内容

```bash
Host ferry
HostName ip
Port 22
User root
IdentityFile    /usr/local/aliyun/SSH-ferry-aliyun-private
RemoteForward 52698 localhost:52698
```

&emsp;&emsp;登陆远程服务器，然后执行以下命令：

```bash
sudo wget -O /usr/local/bin/subl \
https://raw.github.com/aurora/rmate/master/rmate
sudo chmod a+x /usr/local/bin/subl
sudo mv /usr/local/bin/subl ~bin/
```

&emsp;&emsp;安装`rmate`并将其重命名为subl.最后让subl这个指令全局可用。此时，就可以用ssh连接到服务器，然后用`subl [-w] fileName`在本地用sublime打开文件进行编辑,-w参数会让shell不可操作，必须关闭打开的文件后才可以继续执行命令。

### SSH自动断开连接问题

&emsp;&emsp;使用ssh连接服务器，如果超过一定时间无操作就会断开连接，就很烦。

- 编辑`/etc/ssh/sshd_config`

```bash
ClientAliveInterval 60
ClientAliveCountMax 3
```

&emsp;&emsp;以上两个参数表示每隔60秒服务端向客户端发送一次心跳，若客户端3次无响应则断开连接。

### 用户的创建与删除

- 创建用户(adduser/useradd)

&emsp;&emsp;在CentOs下创建用户使用两个指令没有区别，在home下自动创建目录，没有设置密码，需要使用passwd命令修改密码。而在Ubuntu下useradd与adduser有所不同,adduser在使用该命令创建用户是会在/home下自动创建与用户名同名的用户目录，系统shell版本，会在创建时会提示输入密码，更加友好。useradd在使用该命令创建用户是不会在/home下自动创建与用户名同名的用户目录，而且不会自动选择shell版本，也没有设置密码，那么这个用户是不能登录的，需要使用passwd命令修改密码。

- 删除用户

&emsp;&emsp;userdel只能删除用户，并不会删除相关的目录文件。userdel -r 可以删除用户及相关目录。

### 文件、目录权限

&emsp;&emsp;Linux系统是多用户操作系统，因此其中的每个文件和目录都有访问许可权限，用它来确定谁能通过何种方式对文件和目录进行访问和操作。

- 权限的分类

> `r`读权限：可以打开文件、目录读取查看；
> `w`写权限：对文件、目录可以编写更改；
> `x`执行权限：对文件可执行(可执行文件)、对目录可查找该目录下的内容；
> `-`没有权限。

- 权限所属对象

> `u`拥有者：生成文件或目录时登录的当前人，权限最高；
> `g`同组人：系统管理员分配的同组的一个或几个人；
> `o`其他人：除拥有者，同组人以外的人；
> `a`所有人：包括拥有者、同组人及其他人。

#### chmod

- 使用字母表示权限

```bash
chmod g+rwx,o+r dir
```

![设置权限](/linux-order/linux_file_permissions.png)

- 使用数字表示权限

```bash
chmod 555 1.txt
```

![设置权限](/linux-order/linux_permission_num1.png)

![设置权限](/linux-order/linux_permission_num2.png)

#### chown

- 更改某个文件或目录的属主和属组，可用于授权[- R 递归式地改动指定目录及其下的所有子目录和文件的拥有者。]。

```bash
chown -R ferry:ferry 1.txt
```


### 系统服务管理器指令`systemctl`

&emsp;&emsp;systemctl命令是系统服务管理器指令，它实际上将 service 和 chkconfig 这两个命令组合到一起。

#### 常用指令

- 启动服务

```bash
systemctl start tomcat.service
```

- 重启服务

```bash
systemctl restart tomcat.service
```

- 查看服务状态

```bash
systemctl status tomcat.service
```

- 关闭服务

```bash
systemctl stop tomcat.service
```

- 设置开机自启

```bash
systemctl enable tomcat.service
```

- 禁用开机自启

```bash
systemctl disable tomcat.service
```

- 查看所有已启动的服务

```bash
systemctl list -units --type=service
```

#### 将服务配置成系统服务

- 创建服务的配置文件，以redis为例

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

&emsp;&emsp;具体这个配置文件还可以指定哪些参数可以自行百度。

- 可执行权限

```bash
chmod +x /lib/systemd/system/redisd.service
```

- 刷新配置

```bash
systemctl daemon-reload
```

&emsp;&emsp;注意，每次修改`/lib/systemd/system/*.service`文件都需要刷新配置才可以进行启动。

- 启动服务

```bash
systemctl start redisd.service
```

### wget

&emsp;&emsp;Linux系统中的wget是一个下载文件的工具，它用在命令行下。wget支持HTTP，HTTPS和FTP协议，可以使用HTTP代理。wget 非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性.如果是由于网络的原因下载失败，wget会不断的尝试，直到整个文件下载完毕。如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。这对从那些限定了链接时间的服务器上下载大文件非常有用。

- 命令参数

**启动参数**

-V, –version 显示wget的版本后退出

-h, –help 打印语法帮助

-b, –background 启动后转入后台执行

-e, –execute=COMMAND 执行`.wgetrc’格式的命令，wgetrc格式参见/etc/wgetrc或~/.wgetrc

**记录和输入文件参数**

-o, –output-file=FILE 把记录写到FILE文件中

-a, –append-output=FILE 把记录追加到FILE文件中

-d, –debug 打印调试输出

-q, –quiet 安静模式(没有输出)

-v, –verbose 冗长模式(这是缺省设置)

-nv, –non-verbose 关掉冗长模式，但不是安静模式

-i, –input-file=FILE 下载在FILE文件中出现的URLs

-F, –force-html 把输入文件当作HTML格式文件对待

-B, –base=URL 将URL作为在-F -i参数指定的文件中出现的相对链接的前缀

–sslcertfile=FILE 可选客户端证书

–sslcertkey=KEYFILE 可选客户端证书的KEYFILE

–egd-file=FILE 指定EGD socket的文件名

**下载参数**

–bind-address=ADDRESS 指定本地使用地址(主机名或IP，当本地有多个IP或名字时使用)

-t, –tries=NUMBER 设定最大尝试链接次数(0 表示无限制).

-O –output-document=FILE 把文档写到FILE文件中

-nc, –no-clobber 不要覆盖存在的文件或使用.#前缀

-c, –continue 接着下载没下载完的文件

–progress=TYPE 设定进程条标记

-N, –timestamping 不要重新下载文件除非比本地文件新

-S, –server-response 打印服务器的回应

–spider 不下载任何东西

-T, –timeout=SECONDS 设定响应超时的秒数

-w, –wait=SECONDS 两次尝试之间间隔SECONDS秒

–waitretry=SECONDS 在重新链接之间等待1…SECONDS秒

–random-wait 在下载之间等待0…2*WAIT秒

-Y, –proxy=on/off 打开或关闭代理

-Q, –quota=NUMBER 设置下载的容量限制

–limit-rate=RATE 限定下载输率

**目录参数**

-nd –no-directories 不创建目录

-x, –force-directories 强制创建目录

-nH, –no-host-directories 不创建主机目录

-P, –directory-prefix=PREFIX 将文件保存到目录 PREFIX/…

–cut-dirs=NUMBER 忽略 NUMBER层远程目录

**HTTP 选项参数**

–http-user=USER 设定HTTP用户名为 USER.

–http-passwd=PASS 设定http密码为 PASS

-C, –cache=on/off 允许/不允许服务器端的数据缓存 (一般情况下允许)

-E, –html-extension 将所有text/html文档以.html扩展名保存

–ignore-length 忽略 `Content-Length’头域

–header=STRING 在headers中插入字符串 STRING

–proxy-user=USER 设定代理的用户名为 USER

–proxy-passwd=PASS 设定代理的密码为 PASS

–referer=URL 在HTTP请求中包含 `Referer: URL’头

-s, –save-headers 保存HTTP头到文件

-U, –user-agent=AGENT 设定代理的名称为 AGENT而不是 Wget/VERSION

–no-http-keep-alive 关闭 HTTP活动链接 (永远链接)

–cookies=off 不使用 cookies

–load-cookies=FILE 在开始会话前从文件 FILE中加载cookie

–save-cookies=FILE 在会话结束后将 cookies保存到 FILE文件中

**FTP 选项参数**

-nr, –dont-remove-listing 不移走 `.listing’文件

-g, –glob=on/off 打开或关闭文件名的 globbing机制

–passive-ftp 使用被动传输模式 (缺省值).

–active-ftp 使用主动传输模式

–retr-symlinks 在递归的时候，将链接指向文件(而不是目录)

**递归下载参数**

-r, –recursive 递归下载－－慎用!

-l, –level=NUMBER 最大递归深度 (inf 或 0 代表无穷)

–delete-after 在现在完毕后局部删除文件

-k, –convert-links 转换非相对链接为相对链接

-K, –backup-converted 在转换文件X之前，将之备份为 X.orig

-m, –mirror 等价于 -r -N -l inf -nr

-p, –page-requisites 下载显示HTML文件的所有图片

**递归下载中的包含和不包含(accept/reject)**

-A, –accept=LIST 分号分隔的被接受扩展名的列表

-R, –reject=LIST 分号分隔的不被接受的扩展名的列表

-D, –domains=LIST 分号分隔的被接受域的列表

–exclude-domains=LIST 分号分隔的不被接受的域的列表

–follow-ftp 跟踪HTML文档中的FTP链接

–follow-tags=LIST 分号分隔的被跟踪的HTML标签的列表

-G, –ignore-tags=LIST 分号分隔的被忽略的HTML标签的列表

-H, –span-hosts 当递归时转到外部主机

-L, –relative 仅仅跟踪相对链接

-I, –include-directories=LIST 允许目录的列表

-X, –exclude-directories=LIST 不被包含目录的列表

-np, –no-parent 不要追溯到父目录

wget -S –spider url 不下载只显示过程

### less命令

&emsp;&emsp;使用`less`打开文件查看并快速定位文件内容，常用于查看日志文件。

- 打开文件

```bash
less catalina.out
```

&emsp;&emsp;可以在打开的时候添加参数进行文件内容定位，比如说定位到第100行，最后一行等，但是在我实际应用的过程中基本没有用到，感觉华而不实，故此处不做介绍。

- 全局导航

g: 定位到文件的第一行

G: 定位到文件的最后一行

v: 调用编辑器进入编辑模式

q: 退出

- 行导航

j: 向下移动一行

k: 向上移动一行

=: 显示当前行信息，如行号、字节位置等

- 页导航

b: 向上翻一页

空格: 向下翻一页

u: 向上翻半页

d: 向下翻半页

- 标记导航

ma: 在当前位置标记，标记名为`a`
'a: 跳转到标记名为`a`的标记处

- 搜索

/text: 搜索`text`并定义到下一个匹配的文本，匹配内容高亮显示，在次搜索模式下，n: 定位到下一个匹配内容，N: 定位到上一个匹配内容。

?text: 搜索`text`并定义到上一个匹配的文本，匹配内容高亮显示，在次搜索模式下，n: 定位到下一个匹配内容，N: 定位到上一个匹配内容。

- 实时查看文件内容

F: 实现类似`tail -f *.log`的效果，可以实时查看文件的最新内容。使用`Ctrl C`退出这种状态。

### 查看磁盘空间使用情况

#### du命令

&emsp;&emsp;du命令是对文件和目录磁盘使用的空间的查看，显示每个文件和目录的磁盘使用空间。是通过搜索文件来计算每个文件的大小然后累加，du能看到的文件只是一些当前存在的，没有被删除的。他计算的大小就是当前他认为存在的所有文件大小的累加和。

- 缺省参数

```bash
du
```

&emsp;&emsp;显示当前目录下的子目录和文件的大小及名称，并递归查询并显示子目录下的目录文件的大小，最后一行为当前目录的大小。

- du file1/directory1 file2/directory2 ... 

```bash
du catalina.out
```

&emsp;&emsp;显示指定的文件/目录的大小。如果指定的是目录，还会递归查询并显示指定目录的子目录的大小。

- du -s

```bash
du -s
du -s apache-tomcat-8.5.47/
```

![查看](/linux-order/du-s.png)

&emsp;&emsp;显示指定的文件/目录占用磁盘空间大小，不显示子目录

- du -h

```bash
du -h 
```

![查看](/linux-order/du-h.png)

&emsp;&emsp;优化显示占用磁盘空间大小，会过滤文件，只显示目录。

- du -ah

```bash
du -ah logs/
```

&emsp;&emsp;优化显示指定文件/目录占用磁盘空间大小，递归查询并优化显示指定目录的子目录的占用磁盘空间大小。

- du -c

```bash
du -c README.md  RUNNING.txt
```

![查看](/linux-order/du-c.png)

&emsp;&emsp;查看文件占用磁盘空间大小并显示统计。

- du | sort -nr

```bash
du | sort -nr
```

&emsp;&emsp;对目录文件大小进行顺序排序。

&emsp;&emsp;还有一些其他的参数可以使用，这里不做介绍，以上的参数可以组合使用，完成对目录磁盘空间占用大小的统计查看。

- 查看指定目录下的文件/目录大小并优化显示并排序

```bash
du -ah apache-tomcat-8.5.47 | sort -nr
```

#### df命令

&emsp;&emsp;df命令用于显示目前在Linux系统上的文件系统的磁盘使用情况统计。通过文件系统来快速获取空间大小的信息，当我们删除一个文件的时候，这个文件不是马上就在文件系统当中消失了，而是暂时消失了，当所有程序都不用时，才会根据OS的规则释放掉已经删除的文件， df记录的是通过文件系统获取到的文件的大小，他比du强的地方就是能够看到已经删除的文件，而且计算大小的时候，把这一部分的空间也加上了，更精确了。

&emsp;&emsp;当文件系统也确定删除了该文件后，这时候du与df就一致了。

&emsp;&emsp;这里只记录我最常用的一个命令。优化文件系统的输出显示。

```bash
df -h
```

&emsp;&emsp;使用心得，一般情况下，如果磁盘空间满了，我会先用`df -h`查看文件系统的使用情况，然后使用`du`命令跟踪定位到具体的某个或某些大文件，并根据实际情况清理释放磁盘空间。

### CentOS7防火墙设置

&emsp;&emsp;在旧版本的CentOS中，是使用 iptables 命令来设置防火墙的。但是，从CentOS7开始，默认就没有安装iptables，而是改用firewall来配置防火墙。修改了防火墙的设置，必须要重启防火墙才能生效。

- **启动,停止和重启**

```bash
systemctl start firewalld
systemctl stop firewalld
systemctl restart firewalld
```

- **添加端口**

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

- **更新防火墙规则**

```bash
firewall-cmd --reload
```

- **查看端口是否开放**

```bash
firewall-cmd --zone=public --query-port=80/tcp
```

- **删除**

```bash
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

- **查看防火墙状态**

```bash
firewall-cmd --state
```

- **查看防火墙所有设置**

```bash
firewall-cmd --list-all
```

- **查看所有打开的端口**

```bash
firewall-cmd --zone=public --list-ports
```

- **查看区域信息**

```bash
firewall-cmd --get-active-zones
```

- **查看指定接口所属区域**

```bash
firewall-cmd --get-zone-of-interface=eth0
```

- **拒绝所有包**

```bash
firewall-cmd --panic-on
```

- **取消拒绝状态**

```bash
firewall-cmd --panic-off
```

- **查看是否拒绝**

```bash
firewall-cmd --query-panic
```




