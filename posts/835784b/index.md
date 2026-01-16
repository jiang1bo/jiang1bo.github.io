# Linux杂谈：一些常用命令


# linux杂谈：一些常用命令

<!--more-->

```shell
##linux杂谈：一些常用命令
#查看内核版本 
rpm -p kernel
 
#删除....内核版本
dnf -y remove .......
 
#删除....内核版本
dnf remove --oldinstallonly --setopt installonly_limit=2 kernel
 
#查看文本文件所在行
grep root /etc/passwd
 
#执行如下命令可以设置一个新的系统时间：
date -s "20190712 18:30:50" #执行如下命令将系统时间同步到硬件，防止系统重启后时间被还原。 hwclock --systohc

#重启一下我们修改过的网卡，nmtui修改ip
[root@7-2 ~]# nmcli connection up ens3 ens33  ens34  ens35
```

## **网卡无法启动**

```shell
#用ifconfig查看，发现网卡不见了。用ifconfig -a查看，发现ens33网卡是在的，只是状态是down的。
#临时的解决方法：
#使用ifconfig命令，直接启动ens33网卡
[root@centos7 ~]# ifconfig ens33 up
#然后，临时配置一个IP地址
[root@centos7 ~]# ifconfig ens33 192.168.100.1 netmask 255.255.255.0
 
注意：此方法再虚拟机重启后失效。
```

## **RPM 包的常用操作命令**

```shell
#1. 安装
rpm -ivh package.rpm：安装指定的 RPM 包，其中-i表示安装，-v表示显示详细信息，-h表示以哈希符号显示安装进度。
 
#2. 升级
rpm -Uvh package.rpm：升级指定的 RPM 包，如果系统中未安装该包，则会进行安装。-U选项用于升级操作。
 
#3. 卸载
rpm -e package：卸载指定的已安装软件包，-e表示卸载操作。
 
#4. 查询
rpm -qa：查询系统中已安装的所有 RPM 包。可以通过管道符|与grep命令结合使用，来查找特定的软件包。例如，rpm -qa | grep openssh可以查找系统中已安装的 OpenSSH 相关的 RPM 包。
rpm -qi package：查询指定软件包的详细信息，包括软件的描述、版本、依赖关系等。
rpm -ql package：查询指定软件包安装的文件列表，显示该软件包在系统中安装的所有文件及其路径。
```

## **SSH时出现WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!**

**翻译过来就是 警告：远程主机标识已更改！**ssh服务是通过公钥和私钥来进行连接的，它会把每个曾经访问过计算机或服务器的公钥（public key），记录在~/.ssh/known_hosts 中，当下次访问曾经访问过的计算机或服务器时，ssh就会核对公钥，如果和上次记录的不同，OpenSSH会发出警告。 使用命令清除所连接的IP ssh-keygen -R XX.XX.XX.XX

## vim/vi 文本编辑器

### 1）常用快捷键：

(在一般模式下使用)：

```shell
#在光标后一位开始插入
a 

#在该行的最后插入
A 

#在该行的最前面插入
I 

#直接跳到文件的首行
gg 

#直接跳到文件的末行
G 

#删除一行；3dd 删除3行；yy 复制一行；3yy 复制3行；p 粘贴；u：即undo撤销操作；
dd 

#进入字符选择模式，选择完成后，按y复制，按p粘贴
v 

#进入块选择模式，选择完成后，按y复制，按p粘贴
ctrl+v 

#进入行选择模式，选择完成后，按y复制，按p粘贴
shift+v 
```

### 2）查找并替换

1 显示行号 :set nu

2 隐藏行号：:set nonu

3 查找关键字 :/you ## 效果：查找文件中出现的you，并定位到第一个找到的地方，按n可以定位到下一个匹配位置(按N定位到上一个)

4 替换操作 ：s/sad/bbb 查找光标所在行的第一个sad，替换为bbb

```shell
:%s/sad/bbb 查找文件中所有sad，替换为bbb
```

### **head 显示文件内容头部**

```shell
#查看文件头部的10行
head -10 install.log 
```

## tail 显示文件内容尾部

```shell
tail -10 install.log 查看文件尾部的10行

tail +10 install.log 查看文件 10–>末行

tail -f install.log 小f跟踪文件的唯一inode号，就算文件改名后，还是跟踪原来这个inode表示的文件

tail -F install.log 大F按照文件名来跟踪
```



## windows永恒之蓝测试

在CMD窗口下，输入如下命令：netstat –ano | findstr “445”，找出相关进程号，其中SYN_SENT状态，很显然，该电脑被感染永恒之蓝病毒了

找出相关进程号对应的进程，在cmd下输入tasklist | findstr 20164，看看是哪个进程产生该数据包

打开任务管理器，找到相应的进程，先打开文件位置，然后结束进程



## Cockpit 是什么具体解释：

```shell
systemctl enable --now cockpit.socket
```

这条命令的作用是**启用并立即启动 Cockpit Web 控制台的按需监听服务**。

### 1. **Cockpit 是什么？**
   - Cockpit 是一个**基于 Web 的 Linux 服务器管理界面**
   - 可通过浏览器（端口 9090）管理服务器：监控系统状态、管理服务、配置网络、查看日志等

### 2. **cockpit.socket 的特殊性**
   - 这是 **Socket 激活（Socket Activation）** 方式
   - 与传统服务不同：
     - 普通服务：`cockpit.service` 会一直运行
     - Socket 服务：`cockpit.socket` 只**监听端口**，当有连接请求时才启动服务

### 3. **命令执行的具体效果：**
   ```bash
   systemctl enable --now cockpit.socket
   ```
   - `enable`：设置开机自动启动
   - `--now`：立即启动（无需重启系统）
   - 结果：
     1. 系统开始监听 **9090 端口**
     2. 当用户通过浏览器访问时，才启动 cockpit 服务
     3. 空闲一段时间后，服务会自动停止（节省资源）

### 4. **与传统启动方式的对比：**
   ```bash
   # Socket 方式（按需启动 - 推荐）
   systemctl enable --now cockpit.socket
   ```

### 5. **验证命令：**
   ```bash
   # 查看 socket 状态
   systemctl status cockpit.socket
   
   # 查看监听端口
   ss -ltn | grep 9090
   
   # 如果服务未按需启动，可检查
   systemctl status cockpit.service
   ```

### 6. **访问方式：**
   - 浏览器访问：`https://你的服务器IP:9090`
   - 使用系统账户密码登录

### 7. **关闭/禁用方法：**
   ```bash
   # 停止并禁用
   systemctl disable --now cockpit.socket
   
   # 如果 cockpit.service 已被激活运行，也需要停止
   systemctl stop cockpit.service
   ```

**优点**：Socket 激活方式更节省资源，特别适合不经常使用的管理界面。

## **ubuntu拓荒指南**

### **1.ubuntu上报E: Unable to locate package update错误的解决方法**

```sql
#如果第一次使用系统，软件库缓存没有更新会包找不到软件包的错误。使用下面命令更新
 
sudo apt update
 
#注意，上面的命令不是apt install，因为使用apt install，apt 包管理器会搜索缓存以获取包和版本信息，然后通##过网络从其存储库下载它。如果软件包不在此缓存中，系统将无法安装软件包。
 
新装系统，缓存是空的。首先应该运行 apt update 命令。这样就不会出现E: Unable to locate package 错误。
```

[ubuntu上报E: Unable to locate package update错误的解决方法 – 知乎](https://zhuanlan.zhihu.com/p/484067483#:~:text=ubuntu上报E%3A Unable to locate package update错误的解决方法 1 1.检查软件包名名称：,4. 检查 Ubuntu 版本的活跃程度 Ubuntu 有两种主要的版本。 常规版本支持九个月，而长期支持版本支持五年。)

### **2.ubuntu使用ping命令未发现**

```csharp
Sudo apt-get install iputils-ping
```



## 用户管理及权限、用户授权，登陆相关命令

useradd 添加用户。
添加一个tom用户，设置它属于users组，并添加注释信息：useradd -g users -c “hr tom” tom
useradd -c “Name Surname ” -g admin -d /home/user1 -s /bin/bash user1 创建一个属于 “admin” 用户组的用户

usermod 修改系统已经存在用户属性。
修改tom用户的登陆名为tomcat：usermod -l tomcat tom

将tomcat添加到sys和root组（-G，-g）中：usermod -G sys,root tomcat
查看tomcat的组信息：groups tomcat
usermod -c “User FTP” -g system -d /ftp/user1 -s /bin/nologin user1 修改用户属性

userdel 删除用户。
userdel -r spark 加一个-r就表示把用户及用户的主目录都删除

groupadd 添加用户群组+groupmod组修改
passwd 修改用户密码。
chage 修改用户密码有效期限。
id 查看用户的uid/gid及归属的用户群组

groupmod -n new_group_name old_group_name 重命名一个用户组

## chmod 改变文件、目录权限

chmod -R 770 aaa/ ##如果要将一个文件夹的所有内容权限统一修改，则可以-R参数
chmod ugo+rwx directory1 ##设置目录的所有人(u)、群组(g)以及其他人(o)以读（r ）、写(w)和执行(x)的权限
chmod go-rwx directory1 ##删除群组(g)与其他人(o)对目录的读写执行权限

find / -perm -u+s 罗列一个系统中所有使用了SUID控制的文件
chmod u+s /bin/file1 设置一个二进制文件的 SUID 位 – 运行该文件的用户也被赋予和所有者同样的权限
chmod u-s /bin/file1 禁用一个二进制文件的 SUID位
chmod g+s /home/public 设置一个目录的SGID 位 – 类似SUID ，不过这是针对目录的
chmod g-s /home/public 禁用一个目录的 SGID 位
chmod o+t /home/public 设置一个文件的 STIKY 位 – 只允许合法所有人删除文件
chmod o-t /home/public 禁用一个目录的 STIKY 位
chgrp 更改文件用户群组。

chown 改变文件、目录的属主和属组
chown :angela aaa ## 改变所属组
chown angela:angela aaa/ ## 同时修改所属用户和所属组
chown -R user1 directory1 ##改变一个目录的所有人属性并同时改变改目录下所有文件的属性

## umask 显示、设置权限掩码。

whoami 显示当前有效用户名称，相当于执行id -un命令。
who 显示目前已登录系统的用户信息。
w 显示已登陆系统的用户列表，并显示用户正在执行的指令。
last 显示已登入系统的用户。
lastlog 显示系统中所有用户最近一次的登录信息。
users 显示当前登录系统的所有用户的用户列表。
finger 查找、显示用户信息。

## gpasswd

将tomcat用户从root组和sys组删除：gpasswd -d tomcat root&&gpasswd -d tomcat sys
将america组名修改为am：groupmod -n am america

## Docker

启动 docker
\#启动docker
sudo systemctl start docker

\#查看docker服务状态 running 就是启动成功
sudo systemctl status docker

设置 docker 开机自启
sudo systemctl enable docker

\#镜像命令
docker images：列出所有镜像
docker search [image]：搜索 Docker 镜像
docker pull [image]：拉取指定镜像
docker rmi [image]：删除指定镜像
\#容器命令
docker ps：列出当前所有正在运行的容器
docker ps -a：列出所有容器，包括已经停止的容器
docker create [image]：创建一个新的容器，但不启动它
docker start [container]：启动一个容器
docker stop [container]：停止一个容器
docker rm [container]：删除一个容器
docker exec -it [container] [command]：在运行中的容器中执行命令
\#其他命令
docker info：显示 Docker 系统信息
docker version：显示 Docker 版本信息
docker logs [container]：查看容器的日志
docker network ls：列出 Docker 网络
docker network create [network]：创建一个新的 Docker 网络
docker network connect [network] [container]：将容器连接到指定的 Docker 网络
docker network disconnect [network] [container]：将容器从指定的 Docker 网络中断开连接

创建挂载的目录
vi /mydata/mysql/conf/my.cnf

### 输入以下mysql配置文件

[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect=’SET collation_connection = utf8_unicode_ci’
init_connect=’SET NAMES utf8′
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

docker pull mysql:5.7

docker run -p 3306:3306 –name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7

-p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口
-v /mydata/mysql/conf:/etc/mysql/conf.d：将配置文件夹挂载到主机
-v /mydata/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
-v /mydata/mysql/data:/var/lib/mysql：将数据文件夹挂载到主机
-e MYSQL_ROOT_PASSWORD=root：初始化 root 用户的密码

### 使用容器mysql命令行工具

docker exec -it mysql mysql -uroot -proot

设置 root 远程访问

//设置远程root密码登录，与本地root密码登录不同
grant all privileges on *.* to ‘root’@’%’ identified by ‘root’ with grant option;
// 如需修改
//alter user ‘root’@’%’ identified with mysql_native_password by ‘123456’;
flush privileges;

进入容器内部，查看配置文件是否挂载成功

docker exec -it mysql /bin/bash

cd /etc/mysql/conf.d
cat my.cnf

### 容器化安装 Redis 容器

创建挂载的目录
vi /mydata/redis/conf/redis.conf
1
输入以下配置文件

\# 暂无
\#password= 123

1、下载镜像文件，创建实例并启动
不多废话，直接上代码

docker pull redis

docker run -p 6379:6379 –name redis \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf

参数说明

-p 6379:6379：将容器的 3306 端口映射到主机的 3306 端口
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf：将配置文件夹挂载到主机
-v /mydata/redis/data:/data ：将数据文件夹挂载到主机
2、进入容器
\#重启redis
docker restart redis
\#启动redis客户端
docker exec -it redis redis-cli

Docker 容器化安装 Nacos容器
mysql创建nacos的数据库
create database nacos_config

SQL 文件：db/nacos-db.sql · yeye/gmall – 码云 – 开源中国 (gitee.com)

### Nacos配置

创建挂载的目录
mkdir -p /mydata/nacos/conf/ #新建nacos的logs目录
mkdir -p /mydata/nacos/logs/
vim /mydata/nacos/conf/application.properties #新建并修改nacos的配置文件

server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848

spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://xx.xx.xx.x:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=root

nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false

management.metrics.export.elastic.enabled=false

management.metrics.export.influx.enabled=false

server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t “%r” %s %b %D %{User-Agent}i

nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true

注意： 这里的配置需要把数据库链接配置一下。

db.num=1

db.url.0=jdbc:mysql://xx.xx.xx.x:3306/nacos_devtest_prod?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true

db.user=user

db.password=pass

修改为自己的数据库地址和账号密码

下载镜像文件，创建实例并启动
不多废话，直接上代码

docker pull nacos/nacos-server:1.1.4
docker run -d \
–name nacos \
-p 8848:8848 \
–restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-v /mydata/nacos/logs:/home/nacos/logs \
-v /mydata/nacos/conf/application.properties:/home/nacos/conf/application.properties \
nacos/nacos-server

参数说明

-p 6379:6379：将容器的 3306 端口映射到主机的 3306 端口
/mydata/nacos/conf/application.properties:/home/nacos/conf/application.properties：将配置文件夹挂载到主机
-v /mydata/nacos/logs:/home/nacos/logs：将日志文件夹挂载到主机
进入容器
\#重启redis
docker restart nacos
\#进入容器
docker exec -it nacos bash

访问 Nacos
访问地址：http://x.x.x.x:8848/nacos (这里是服务器的ip地址)
账号密码：nacos / nacos

进去了就算成功
“\”这是啥意思.好像是空格，删掉直接把下一行的内容跟上一行放一起就行

tcpdump -i ens33 -w ./xxx.cap //指定网口抓包
tcpdump -i ens33 src net 10.193.17.4 -w ./package.cap //指定网口+SIP抓包并保存./package.cap

date //查看系统时间
hwclock //查看硬件时间
date –s “2017-08-10 10:21:00” //系统时间校准
hwclock –w //同步时间给硬件服务器

systemctl status firewalld //查看防火墙状态
systemctl stop firewalld //临时关闭防火墙
systemctl disable firewalld //永久防火墙开机自关闭
systemctl start firewalld //临时打开防火墙
systemctl enable firewalld //防火墙开机启动

systemctl stop network.service
systemctl start network.service
service network restart //重启网卡
cd /etc/sysconfig/network-scripts/ //查看网卡信息的路径
ethtool xxx(ens33) //查看网卡xxx状态，包括速率、Link状态，传输类型等；


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/835784b/  

