# Windows+linux常用操作命令


<!--more-->

# windows系统

//进入d盘
D:
//进入F盘
F:

cd /? //获取使用帮助

cd \ //跳转到硬盘的根目录

cd C:\WINDOWS //跳转到当前硬盘的其他文件

d: //跳转到其他硬盘

cd /d e:\software //跳转到其他硬盘的其他文件夹，注意此处必须加/d参数。否则无法跳转。

cd.. //跳转到上一层目录

//查看当前目录下的文件，类似于linux下的ls

## dir

copy 路径\文件名 路径\文件名 ：把一个文件拷贝到另一个地方。

move 路径\文件名 路径\文件名 ：把一个文件移动（就是剪切+复制）到另一个地方。

//这个是专门删除文件的，不能删除文件夹
del 文件名

## taskkill

taskkill /? 获取使用帮助
TASKKILL /S system /F /IM notepad.exe /T
TASKKILL /PID 1230 /PID 1241 /PID 1253 /T
TASKKILL /F /IM QQ.exe
TASKKILL /F /IM notepad.exe /IM mspaint.exe

## find /？获取使用帮助

netstat -ano|find “.8” //使用管道符，进行模糊查询

“|”cmd命令中|代表前一个的输出代表后一个的输入

查找特定ip的网络连接及进程号：netstat -ano|find “192.168.1.10”

重定向输出符号> >>
将原本输出到命令窗口的内容，转存到文件中，如jstack 12912 >d:/s.txt 打印线程到指定文件

cmd > 重定向输出并覆盖源文件。

echo hello >c:\1.txt // 1.txt的文件内容先被清空，然后写入hello。
cmd >>重定向输出追加到文件末尾

echo hello >>c:\1.txt // 在1.txt文件末尾加上hello

netstat -ano
netstat -ab
端口，进程，进程号
netstat -ano|findstr “”
查看开放使用的端口号，以及占用进程进程号
tasklist|findsrt “”
通过进程号寻找相关进程

 

dfrg.msc——-磁盘碎片整理程序
chkdsk.exe—–Chkdsk磁盘检查
devmgmt.msc— 设备管理器
winver———检查Windows版本

net user 用户名　密码　/add 建立用户
net user 帐户名 查看帐户的属性
net localgroup administrators 用户名 /add 把“用户”添加到管理员中使其具有管理员权限
net start 查看开启了哪些服务
net start 服务名　开启服务；(如:net start telnet， net start schedule)
net stop 服务名 停止某服务
net time 目标ip 查看对方时间
net time 目标ip /set 设置本地计算机时间与“目标IP”主机的时间同步,加上参数/yes可取消确认信息
net user guest /active:yes 激活guest用户
net user guest 12345 用guest用户登陆后用将密码改为12345
net password 密码 更改系统登陆密码
fc one.txt two.txt > 3st.txt 对比二个文件并把不同之处输出到3st.txt文件中，”> “和”> >” 是重定向命令
at 查看所有的计划任务
route print 显示出IP路由，将主要显示网络地址Network addres，子网掩码Netmask，网关地址Gateway addres，接口地址Interface
arp 查看和处理ARP缓存，ARP是名字解析的意思，负责把一个IP解析成一个物理性的MAC地址。arp -a将显示出全部信息
set 显示当前所有的环境变量
findstr “Hello” aa.txt 在aa.txt文件中寻找字符串hello
find 文件名 查找某文件
title 标题名字 更改CMD窗口标题名字
color 颜色值 设置cmd控制台前景和背景颜色；0=黑、1=蓝、2=绿、3=浅绿、4=红、5=紫、6=黄、7=白、8=灰、9=淡蓝、A=淡绿、B=淡浅绿、C=淡红、D=淡紫、E=淡黄、F=亮白
format 盘符 /FS:类型 格式化磁盘,类型:FAT、FAT32、NTFS ,例：Format D: /FS:NTFS
chkdsk /F D: 检查磁盘D并显示状态报告；加参数/f并修复磁盘上的错误

# 批处理命令与变量

## for命令及变量

基本格式： FOR /参数 %variable IN (set) DO command [command_parameters]

%variable:指定一个单一字母可替换的参数，如：%i ，而指定一个变量则用：%%i ，而调用变量时用：%i% ，变量是区分大小写的（%i 不等于 %I）。
批处理每次能处理的变量从%0—%9共10个，其中%0默认给批处理文件名使用，%1默认为使用此批处理时输入的的第一个值，同理：%2—%9指输入的第2-9个值；例：net use ipipc$ pass /user:user 中ip为%1,pass为%2 ,user为%3
(set):指定一个或一组文件，可使用通配符，如：(D:user.txt)和(1 1 254)(1 -1 254),{ “(1 1 254)”第一个”1″指起始值，第二个”1″指增长量，第三个”254″指结束值，即：从1到254；“(1 -1 254)”说明：即从254到1 }
command：指定对第个文件执行的命令，如：net use命令；如要执行多个命令时，命令这间加：& 来隔开
command_parameters：为特定命令指定参数或命令行开关
IN (set)：指在(set)中取值；DO command ：指执行command
参数：/L 指用增量形式{ (set)为增量形式时 }；/F 指从文件中不断取值，直到取完为止{ (set)为文件时，如(d:pass.txt)时 }。

用法举例：

 

@echo off
echo 用法格式：test.bat *.*.* > test.txt
for /L %%G in (1 1 254) do echo %1.%%G >>test.txt & net use \%1.%%G /user:administrator | find “命令成功完成” >>test.txt
存为test.bat 说明：对指定的一个C类网段的254个IP依次试建立administrator密码为空的IPC$连接，如果成功就把该IP存在test.txt中。

/L指用增量形式（即从1-254或254-1）；
输入的IP前面三位：..*为批处理默认的 %1；
%%G 为变量(ip的最后一位）；
& 用来隔开echo 和net use 这二个命令；
| 指建立了ipc$后，在结果中用find查看是否有”命令成功完成”信息；
%1.%%G 为完整的IP地址；
(1 1 254) 指起始值，增长量，结止值。
@echo off
echo 用法格式：ok.bat ip
FOR /F %%i IN (D:user.dic) DO smb.exe %1 %%i D:pass.dic 200
存为：ok.exe 说明：输入一个IP后，用字典文件d:pass.dic来暴解d:user.dic中的用户密码，直到文件中值取完为止。

%%i为用户名；
%1为输入的IP地址（默认）。
Shutdown.exe
Shutdown IP地址 t:20
20秒后将对方NT自动关闭

 

## fpipe.exe (TCP端口重定向工具)

fpipe -l 80 -s 1029 -r 80
当有人扫锚你的80端口时，他扫到的结果会完全是的主机信息

Fpipe -l 23 -s 88 -r 23 目标IP
把本机向目标IP发送的23端口Telnet请求经端口重定向后，就通过88端口发送到目标IP的23端口。（与目标IP建立Telnet时本机就用的88端口与其相连接）然后：直接Telnet 127.0.0.1（本机IP）就连接到目标IP的23端口了。

# Linux系统

## vim/vi 文本编辑器

1）常用快捷键：(在一般模式下使用)：

a ：在光标后一位开始插入

A 在该行的最后插入

I 在该行的最前面插入

gg 直接跳到文件的首行

G 直接跳到文件的末行

dd 删除一行；3dd 删除3行；yy 复制一行；3yy 复制3行；p 粘贴；u：即undo撤销操作；

v ：进入字符选择模式，选择完成后，按y复制，按p粘贴

ctrl+v 进入块选择模式，选择完成后，按y复制，按p粘贴

shift+v 进入行选择模式，选择完成后，按y复制，按p粘贴

2）查找并替换

1 显示行号 :set nu

2 隐藏行号：:set nonu

3 查找关键字 :/you ## 效果：查找文件中出现的you，并定位到第一个找到的地方，按n可以定位到下一个匹配位置(按N定位到上一个)

4 替换操作 ：s/sad/bbb 查找光标所在行的第一个sad，替换为bbb

:%s/sad/bbb 查找文件中所有sad，替换为bbb

head 显示文件内容头部。
head -10 install.log 查看文件头部的10行

# tail 显示文件内容尾部

tail **-10** install.log 查看文件尾部的10行

tail **+10** install.log 查看文件 10–>末行

tail -f install.log 小f跟踪文件的唯一inode号，就算文件改名后，还是跟踪原来这个inode表示的文件

tail -F install.log 大F按照文件名来跟踪

## 文件压缩及解压缩命令

将许多文件一起保存至一个单独的磁带或磁盘归档，并能从归档中单独还原所需文件。

示例
tar -cf archive.tar foo bar # 从文件 foo 和 bar 创建归档文件
archive.tar。
tar -tvf archive.tar # 详细列举归档文件 archive.tar
中的所有文件。
tar -xf archive.tar # 展开归档文件 archive.tar

tar -cvf archive.tar file1 创建一个非压缩的 tarball
tar -cvf archive.tar file1 file2 dir1 创建一个包含了 ‘file1’, ‘file2’ 以及 ‘dir1’的档案文件
tar -tf archive.tar 显示一个包中的内容
tar -xvf archive.tar 释放一个包
tar -xvf archive.tar -C /tmp 将压缩包释放到 /tmp目录下
tar -cvfj archive.tar.bz2 dir1 创建一个bzip2格式的压缩包
tar -xvfj archive.tar.bz2 解压一个bzip2格式的压缩包
tar -cvfz archive.tar.gz dir1 创建一个gzip格式的压缩包
tar -xvfz archive.tar.gz 解压一个gzip格式的压缩包
解压到/usr/下：tar -zxvf a.tar.gz -C /usr
查看压缩包内容tar -ztvf a.tar.gz
c创建
v显示文件信息
f递归操作
z gzip格式
x 解压

## unzip 解压和zip压缩文件

zip file1.zip file1 创建一个zip格式的压缩包
zip -r file1.zip file1 file2 dir1 将几个文件和目录同时压缩成一个zip格式的压缩包
unzip file1.zip 解压一个zip格式压缩包

## 系统信息查询命令

hostname 显示或者设置当前系统的主机名。
dmesg 显示开机信息，用于诊断系统故障。
uptime 显示系统运行时间以及负载。
uname -m 显示机器的处理器架构(2)
cat /proc/cpuinfo 显示CPU info的信息
cat /proc/meminfo 校验内存使用
cat /proc/version 显示内核的版本
lspci -tv 罗列 PCI 设备
lsusb -tv 显示 USB 设备

top 实时显示系统资源使用情况。
free 查看系统内存
date 显示、设置系统时间
cal 查看日历、时间信息
cal 2021 显示2021年的日历表
date 041217002021.00 设置日期和时间 – 月日时分年.秒
clock -w 将时间修改保存到 BIOS

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

输入以下配置文件

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

使用容器mysql命令行工具

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

容器化安装 Redis 容器
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

Nacos配置
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

# root用户远程ssh登录

 

vim /etc/ssh/sshd_config //进入修改ssh_config文件模式
PermitRootLogin：no–>yes //PermitRootLogin字段值改成yes
systemctl restart sshd //重启sshd

chmod 4755 /bin/su //su root无法登陆root账号，提示认证失败

 

# Nmap简单的扫描方式

全面扫描：nmap -T4 -A ip

主机发现：nmap -T4 -sn ip

端口扫描：nmap -T4 ip

服务扫描：nmap -T4 -sV ip

操作系统扫描：nmap -T4 -O ip

1、SYN扫描
首先可以利用基本的SYN扫描方式探测其端口开放状态。

nmap -sS -T4 www.fujieace.com
扫描输出结果为：

All 997 ports are filtered
PORT STATE SERVICE
80/tcp open http
113/tcp closed auth
507/tcp open crs
我们可以看到SYN方式探测到3个端口开放，而有997个端口被过滤。Nmap默认扫描只扫描1000个最可能开放的端口，如果想扫描全部的端口，命令如下：

nmap -sS -T4-p- www.fujieace.com
2、FIN扫描
然后可以利用FIN扫描方式探测防火墙状态。FIN扫描方式用于识别端口是否关闭，收到RST回复说明该端口关闭，否则说明是open或filtered状态。

nmap -sF -T4 www.fujieace.com
扫描输出结果为：

PORT STATE SERVICE
7/tcp open|filtered echo
9/tcp open|filtered discard
11/tcp open|filtered systat
13/tcp open|filtered daytime
23/tcp open|filtered telnet
25/tcp open|filtered smtp
37/tcp open|filtered time
79/tcp open|filtered finger
80/tcp open|filtered http
更多端口，此处省略…….
3、ACK扫描
然后利用ACK扫描判断端口是否被过滤。针对ACK探测包，未被过滤的端口（无论打开、关闭）会回复RST包。

nmap -sA -T4 www.fujieace.com
扫描输出结果为：

Not shown: 997 unfiltered ports
PORT STATE SERVICE
135/tcp filtered msrpc
1434/tcp filtered ms-sql-m
32777/tcp filtered sometimes-rpc17
从结果可以知道997个端口是未被过滤的（unfiltered），而3个（135/1434/32777）被过滤了。所以，将ACK与FIN扫描 的结果结合分析，我们可以找到很多开放的端口。例如7号端口，FIN中得出的状态是:open或filtered，从ACK中得出的状态是 unfiltered，那么该端口只能是open的。

4、Window扫描
当然也可以利用Window扫描方式，得出一些端口信息，可以与之前扫描分析的结果相互补充。Window扫描方式只对某些TCPIP协议栈才有效。

window扫描原理与ACK类似，发送ACK包探测目标端口，对回复的RST包中的Window size进行解析。在某些TCPIP协议栈实现中，关闭的端口在RST中会将Window size设置为0；而开放的端口将Window size设置成非0的值。

nmap -sW -p- -T4 www.fujieace.com
输出结果：

PORT STATE SERVICE
7/tcp open echo
9/tcp open discard
11/tcp open systat
13/tcp open daytime
更多端口，此处省略……
在采用多种

扫描路由器操作系统
nmap -O -F -n 192.168.1.1
与通用PC扫描方式类似，使用-O选项扫描路由器的操作系统。-F用于快速扫描最可能开放的100个端口，并根据端口扫描结果进一步做OS的指纹分析方式获取出防火墙状态后，可以进一步进行应用程序与版本侦测及OS侦测。

nmap [扫描目标] [选项]
v: 增加输出的详细程度。

A: 启用OS检测、版本检测、脚本扫描和traceroute。

p:指定要扫描的端口范围，例如p 1-100。

# Wireshark使用

ip.addr == 180.101.49.11 and icmp 表示只显示ICPM协议且源主机IP或者目的主机IP为180.101.49.11的数据包。

三次握手原理
先来看一下基本的原理。
第一次握手：建立连接时，客户端发送SYN到服务器，并进入SYN_SENT状态。
第二次握手：服务器收到请求后，回送SYN+ACK信令到客户端，此时服务器进入SYN_RECV状态。
第三次握手：客户端收到SYN+ACK包，向服务器发送确认ACK包，客户端进入ESTABLISHED状态，服务器收到请求后也进入ESTABLISHED状态，完成三次握手，此时TCP连接成功，客户端与服务器开始传送数据。

第一次握手
第一次握手：建立连接时，客户端发送SYN到服务器，并进入SYN_SENT状态。

SYN ：标志位，表示请求建立连接。
Seq = 0 ：初始建立连接值为0，数据包的相对序列号从0开始，表示当前还没有发送数据。
Ack =0：初始建立连接值为0，已经收到包的数量，表示当前没有接收到数据。
WIN = 8192 来自Window size: 8192。
MSS = 1460 来自 Maximum segment size: 1460 byte ，最长报文段，TCP包所能携带的最大数据量，不包含TCP头和Option。一般为MTU值减去IPv4头部(至少20字节)和TCP头部(至少20字节)得到。
WS = 4 来自windows scale : 2 (multiply by 4)： 窗口扩张，放在TCP头之外的Option，向对方声明一个shift count，作为2的指数，再乘以TCP定义的接收窗口，得到真正的TCP窗口。

第二次握手
第二次握手：服务器收到请求后，回送SYN+ACK信令到客户端，此时服务器进入SYN_RECV状态。
Seq = 0 ：初始建立值为0，表示当前还没有发送数据
Ack = 1 : 表示当前端成功接收的数据位数，虽然客户端没有发送任何有效数据，确认号还是被加1，因为包含SYN或FIN标志位。尽管客户端没有发送任何有效数据，确认号还是被加1，这是因为接收的包中包含SYN或FIN标志位（并不会对有效数据的计数产生影响，因为含有SYN或FIN标志位的包并不携带有效数据。

第三次握手
第三次握手：客户端收到SYN+ACK包，向服务器发送确认ACK包，客户端进入ESTABLISHED状态，服务器收到请求后也进入ESTABLISHED状态，完成三次握手，此时TCP连接成功，客户端与服务器开始传送数据。
ACK ：标志位，表示已经收到记录
Seq = 1 ：表示当前已经发送1个数据
Ack = 1 : 表示当前端成功接收的数据位数，虽然客户端没有发送任何有效数据，确认号还是被加1，因为包含SYN或FIN标志位。尽管客户端没有发送任何有效数据，确认号还是被加1，这是因为接收的包中包含SYN或FIN标志位（并不会对有效数据的计数产生影响，因为含有SYN或FIN标志位的包并不携带有效数据)

为什么是三次握手
这个问题的本质是, 信道不可靠, 但是通信双发需要就某个问题达成一致. 而要解决这个问题, 无论你在消息中包含什么信息, 三次通信是理论上的最小值. 所以三次握手不是TCP本身的要求, 而是为了满足”在不可靠信道上可靠地传输信息”这一需求所导致的.

也是为了最小的代价验证会话双方的收发功能正常:

第一次握手成功：说明客户端的数据可以被服务端收到，说明客户端的发功能可用，说明服务端的收功能可用。但客户端自己不知道数据是否被接收。
第二次握手成功：说明服务端的数据可以被客户端收到，说明服务端的发功能可用，说明客户端的收功能可用。同时客户端知道自己的数据已经正确到达服务端，自己的发功能正常。但是服务端自己不知道数据是否被接收。
第三次握手成功：说明服务端知道自己的数据已经正确到达客户端端，自己的发功能正常。至此服务成功建立。

请求数据的过程
客户端和服务端建立连接后，开始传输数据。下图中首先是客户端发起一个GET请求，服务端收到请求后首先返回确认信息。待服务端处理完成后，将数据返回给客户端，客户端对收到的数据进行确认，完成一次数据交互的过程。

交互过程中，SEQ表示发送的数据，LEN表示发送的数据长度，下一次的SEQ就等于当前SEQ加上LEN。ACK表示接收的数据位。客户端和服务端分别计算自己的增长值。对应上图最后一次服务端返回数据时SEQ是2737，LEN是450。客户端对接收数据做了两次返回确认，第一次ACK是2737，表示还没有完成数据接收。第二次ACK是3187，等于服务端SEQ＋LEN（2737+450）表示已经完成了全部数据的接收。

第一次挥手：客户端 发送一个[FIN+ACK]，表示自己没有数据要发送了，想断开连接，并进入FIN_WAIT_1状态（不能再发送数据到服务端，但能够发送控制信息ACK到服务端）。
第二次挥手：服务端收到FIN后，知道不会再有数据从客户端传来，发送ACK进行确认，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），服务端进入CLOSE_WAIT状态。
第三次挥手：服务端发送FIN给对方，表示自己没有数据要发送了，服务端进入LAST_ACK状态，然后直接断开TCP会话的连接，释放相应的资源。
第四次挥手：客户端收到了服务端对FIN的ACK后，进入FIN_WAIT2状态（等待服务端完成资源释放的一系列工作：然后释放你为创建这个连接所分配的资源，并通知我你关闭了）; 客户端收到了服务端的FIN信令后，进入TIMED_WAIT状态，并发送ACK确认消息。客户端在TIMED_WAIT状态下，等待2MSL一段时间，没有数据到来的，就认为对面已经收到了自己发送的ACK并正确关闭了进入CLOSE状态，自己也断开了到服务端的TCP连接，释放所有资源。当服务端收到客户端的ACK回应后，会进入CLOSE状态，并关闭本端的会话接口，释放相应资源。TIME_WAIT状态持续2MSL(MSL是数据分节在网络中存活的最长时间）。
网络上比较主流的文章都说关闭TCP会话是四次挥手，但是实际上为了提高效率通常合并第二、三次的挥手，即三次挥手。

关闭连接需要四次握手。
客户端向服务端发送FIN为1的报文，服务端返回确认，关闭客户端与服务端通信的部分。
服务端向客户端发送FIN为1的报文，客户端返回确认，关闭服务端与客户端通信的部分。

端口最大65535，这里用十六进制，248高八位，162低八位，即真正的端口号为248*256+162=63650
因此在过滤器处对63650端口进行过滤
tcp:port == 63650

就看到列的目录的完整信息，里面有两个文件，意义不大，继续ftp过滤追踪
看到1323有一个下载的操作

可以看到下载的时候又开放了一个端口，就去追踪这个
248*256+165=63653

http && (ip.src==202.1.1.2 || ip.dst == 192.168.1.99)

黑客上传的第一个文件名称是什么
这里使用过滤器
http && (ip.src==202.1.1.2 || ip.dst == 192.168.1.99) && http.request.method==POST

黑客需要以攻克的1.99来作为跳板来访问2.88的ftp服务器，设置过滤器
ftp && ip.src == 192.168.1.99

RETR ftp 下载标识

服务器2.88中用户名为admin_zz的web后台管理员的密码
既然这里给出用户名，直接设置过滤器

ip.addr == 192.168.1.99 && http contains “admin_zz”

服务器2.88的mysql账号密码(格式：账号/密码)
这里也应该是查看黑客下载的敏感文件中的内容，设置过滤器ip.addr == 192.168.1.99 && ftp-data

服务器2.88的mysql服务中有和admin有关的三个表，请按照黑客的查询顺序作答，使用空格分隔

ip.addr == 192.168.1.99 && mysql contains “met_”

请列出黑客设置的genreal log的绝对路径(将路径复制出来，区分大小写)
mysql日志–>设置过滤器

ip.addr == 192.168.1.99 && mysql

路由器的品牌、型号、版本(请直接复制粘贴)
涉及到路由器，设置就过滤器

ip.addr == 192.168.0.1 && http

在路由器的端口监控中，监控端口和被监控端口分别是多少，例，1号端口监控2/3/4号端口:1–>2,3,4
这里提到监控端口,还是设置过滤器慢慢找

ip.addr == 192.168.0.1 && http &&contains “port”

路由器的系统路由表中一共有几条?第三条的子网掩码是多少。例: 255 255.255.0则为24 (格式:用英文逗号分隔)
ip.addr == 192.168.0.1 && http contains “gateway”

路由器的5Gwif名称是什么，信道是多少(格式:名称信道)
设置过滤器
ip.addr == 192.168.0.1 && http contains “5G”

# windows服务器只允许特定IP访问

1. 打开Windows Defender防火墙高级安全

打开“控制面板” > “系统和安全” > “Windows Defender 防火墙”。
在左侧菜单选择“高级设置”。

2. 创建允许规则

在“Windows Defender 防火墙高级安全”窗口的左侧，点击“入站规则”，然后在右侧的“操作”栏点击“新建规则”。

选择“自定义规则”，点击“下一步”。

对于“协议和端口”，根据需要保护的服务选择协议（TCP或UDP）和端口（特定端口或范围），然后点击“下一步”。

在“作用域”页面：

“本地IP地址”保持为“任何IP地址”。
在“远程IP地址”下选择“这些IP地址”，然后输入允许访问的特定IP地址或地址范围。点击“下一步”。
在“操作”页面，选择“允许连接”，点击“下一步”。

在“配置文件”页面，根据服务器的网络环境选择相应的配置文件（通常是全部勾选），点击“下一步”。

给规则命名（例如，“允许特定IP访问HTTP服务”），并提供描述（可选），点击“完成”。

3. 创建拒绝规则

为了确保除了指定的IP地址外，其他所有IP地址都无法访问，你需要创建一个拒绝规则。

重复上述步骤创建新规则，但在“操作”步骤中选择“阻止连接”。
在“远程IP地址”下选择“任何IP地址”。
确保规则名称和描述能清晰表明这是一个拒绝规则，如“拒绝所有其他IP访问HTTP服务”。

4. 确认规则顺序

确保允许规则位于拒绝规则之前，因为在处理规则时，Windows防火墙会按照从上到下的顺序应用规则，最先匹配的规则生效。

5. 测试规则

从允许的IP地址尝试访问服务器，确认可以成功连接。
从一个不在允许列表中的IP地址尝试访问，确保被拒绝。
通过上述步骤，你就成功设置了Windows服务器，只允许特定IP地址访问，而阻止了所有其他IP地址的访问。请根据实际需要调整端口、协议以及IP地址范围。

# **windows：wmic命令**

C:\Users\Administrator>wmic
wmic:root\cli>baseboard get manufacturer,product
Manufacturer Product
ASUSTeK COMPUTER INC. H510M-D3H/M.2 R2.0

wmic:root\cli>

显示进程的详细信息
输入 process where name=”chrome.exe” list full

显示出BIOS信息
wmic bios list full

停止、暂停和运行服务功能
启动服务startservice,
停止服务stopservice,
暂停服务pauseservice,
Service where caption=”windows time” call stopservice ——停止服务
Service where caption=”windows time” call startservice ——启动服务
Service where name=”w32time” call stopservice ——停止服务

注意name和caption的区别:caption 显示服务名,name：服务名称.
如： telnet服务的显示名称是telnet ,服务名称.是tlntsvr。
还有Windows Time服务的名称是w32time 显示名称是”Windows Time”要用引号引起来，主要是有一个空格。

好了具体看一下：输入Service where caption=”windows time” call startservice后有一个确认输入y就可以了，返回ReturnValue = 0;表示成功

大家可能注意到了上面命令行中还有两个参数list和full。list决定显示的信息格式与范围，它有Brief、Full、Instance、 Status、System、Writeable等多个参数，full只是它的一个参数，也是list的缺省参数，表示显示所有的信息。其他几个参数顾名思义，如Brief表示只显示摘要信息，Instance表示只显示对象实例，Status表示显示对象状态，Writeable表示只显示该对象的可写入的属性信息等。

停止进程的操作
例如，执行下面的命令将关闭正在运行的QQ.exe：
例1、wmic process where name=’QQ.exe’ call terminate
例2、wmic process where name=”qq.exe” delete

列出所有的进程 : wmic process.

连接远程电脑
连接远程的电脑，不过好象对要开一些相应的服务

wmic /node:”192.168.203.131″ /password:”” /user:”administrator”

BIOS – 基本输入/输出服务 (BIOS) 管理
查看bios版本型号
wmic bios get Manufacturer,Name

C:\Users\Administrator>wmic baseboard get manufacturer,name,product
Manufacturer Name Product
ASUSTeK COMPUTER INC. 基板 H510M-D3H/M.2 R2.0

WMIC设置IP地址
配置或更新IP地址：
wmic nicconfig where index=0 call enablestatic(“192.168.1.5”), (“255.255.255.0”) ；index=0说明是配置网络接口1。
配置网关（默认路由）：
wmic nicconfig where index=0 call setgateways(“192.168.1.1″),(1)

CPU – CPU 管理
查看cpu型号
wmic cpu get name

C:\Users\Administrator>wmic cpu get name
Name
11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz

DISKDRIVE – 物理磁盘驱动器管理

获取物理磁盘型号大小等
wmic DISKDRIVE get Caption,size,InterfaceType

C:\Users\Administrator>wmic diskdrive get caption,size,interfacetype
Caption InterfaceType Size
SSK SCSI Disk Device SCSI 512105932800
Kingston DataTraveler 3.0 USB Device USB 123724661760
Great Wall GW3300 256GB SCSI 256052966400

LOGICALDISK – 本地储存设备管理
获取硬盘系统格式、总大小、可用空间等
wmic LOGICALDISK get name,Description,filesystem,size,freespace

远程计算机的远程桌面连接
WMIC命令开启远程计算机的远程桌面连接
执行wmic /node:192.168.1.2 /USER:administrator PATH win32_terminalservicesetting WHERE (__Class!=””) CALL SetAllowTSConnections 1
具体格式：

wmic /node:”[full machine name]” /USER:”[domain]\[username]”
PATH win32_terminalservicesetting WHERE (__Class!=””) CALL SetAllowTSConnections 1

实例summary
wmic 获取进程名称以及可执行路径:
wmic process get name,executablepath

wmic 删除指定进程(根据进程名称):
wmic process where name=”qq.exe” call terminate
或者用
wmic process where name=”qq.exe” delete

wmic 删除指定进程(根据进程PID):
wmic process where pid=”123″ delete

wmic 创建新进程
wmic process call create “C:\Program Files\Tencent\QQ\QQ.exe”

在远程机器上创建新进程：
wmic /node:192.168.201.131 /user:administrator /password:123456 process call create cmd.exe

关闭本地计算机
wmic process call create shutdown.exe

重启远程计算机
wmic /node:192.168.1.10/user:administrator /password:123456 process call create “shutdown.exe -r -f -m”

更改计算机名称
wmic computersystem where “caption=’%ComputerName%'” call rename newcomputername

更改帐户名
wmic USERACCOUNT where “name=’%UserName%'” call rename newUserName

wmic 结束可疑进程（根据进程的启动路径）

wmic process where “name=’explorer.exe’ and executablepath<>’%SystemDrive%\\windows\\explorer.exe'” delete

wmic 获取物理内存
wmic memlogical get TotalPhysicalMemory|find /i /v “t”

wmic 获取文件的创建、访问、修改时间

@echo off
for /f “skip=1 tokens=1,3,5 delims=. ” %%a in (‘wmic datafile where name^=”c:\\windows\\system32\\notepad.exe” get CreationDate^,LastAccessed^,LastModified’) do (
set a=%%a
set b=%%b
set c=%%c
echo 文件: c:\windows\system32\notepad.exe
echo.
echo 创建时间: %a:~0,4% 年 %a:~4,2% 月 %a:~6,2% 日 %a:~8,2% 时 %a:~10,2% 分 %a:~12,2% 秒
echo 最后访问: %b:~0,4% 年 %b:~4,2% 月 %b:~6,2% 日 %b:~8,2% 时 %b:~10,2% 分 %b:~12,2% 秒
echo 最后修改: %c:~0,4% 年 %c:~4,2% 月 %c:~6,2% 日 %c:~8,2% 时 %c:~10,2% 分 %c:~12,2% 秒
)
echo.
pause

wmic 全盘搜索某文件并获取该文件所在目录
for /f “skip=1 tokens=1*” %i in (‘wmic datafile where “FileName=’qq’ and extension=’exe'” get drive^,path’) do (set “qPath=%i%j”&@echo %qPath:~0,-3%)

获取屏幕分辨率 wmic DESKTOPMONITOR where Status=’ok’ get ScreenHeight,ScreenWidth

wmic PageFileSet set InitialSize=”512″,MaximumSize=”512″

设置虚拟内存到E盘，并删除C盘下的页面文件,重启计算机后生效

wmic PageFileSet create name=”E:\\pagefile.sys”,InitialSize=”1024″,MaximumSize=”1024″
wmic PageFileSet where “name=’C:\\pagefile.sys'” delete

获得进程当前占用的内存和最大占用内存的大小：

wmic process where caption=’filename.exe’ get WorkingSetSize,PeakWorkingSetSize

以KB为单位显示

@echo off
for /f “skip=1 tokens=1-2 delims= ” %%a in (‘wmic process where caption^=”conime.exe” get WorkingSetSize^,PeakWorkingSetSize’) do (
set /a m=%%a/1024
set /a mm=%%b/1024
echo 进程conime.exe现在占用内存：%m%K；最高占用内存：%mm%K
)
pause

远程打开计算机远程桌面

wmic /node:%pcname% /USER:%pcaccount% PATH win32_terminalservicesetting WHERE (__Class!=””) CALL SetAllowTSConnections 1

# nmap漏洞扫描

通过下面三个强大的Nmap脚本选项, 可以将Nmap变成一个全面的漏洞扫描工具:

 

Vuln：–script vuln选项执行所有NSE脚本, 这些脚本专注于识别正在扫描目标系统是否有安全漏洞, 这是发现已知漏洞的常用选项。
Vulners：–script vulners选项使用vulners.com在线数据库，利用Nmap扫描目标以查找最新漏洞。该数据库定期更新，可能会找到其它扫描可能遗漏的信息。
Vulscan: –script vulscan选项是一个Nmap漏洞扫描脚本, 它使用多个离线漏洞数据库来测试目标系统漏洞, 它使用了NVD、CVE和OVAL漏洞数据库来扫描本地网络中的漏洞, 该模式支持本地离线和在线扫描。

Nmap 的漏洞扫描将针对目标运行该工具的默认漏洞扫描脚本。这些对于查找可利用的常见漏洞非常有用。要运行漏洞扫描，使用以下命令语法：

复制
nmap –script vuln <target>

vulners漏洞扫描能够使用在线数据库vulners.com扫描目标是否有最新的漏洞。要执行此扫描，使用以下命令语法：

复制

nmap -sV –script vulners <target>





## 信息搜索：

国家企业信用信息公示系统（http://www.gsxt.gov.cn/index.html）

该网站主要提供市场主体的注册登记、许可审批、年度报告、行政处罚、抽查结果、经营异常状态等信息。（不含港、澳、台地区企业信息）

中国裁判文书网（http://wenshu.court.gov.cn/）

中国裁判文书网是统一公布各级人民法院的生效裁判文书的平台。

中国执行信息公开网（http://zxgk.court.gov.cn/）

中国执行信息公开网，查公司法人信息

国家知识产权局官方网站（https://www.sipo.gov.cn）

国家知识产权局商标局 中国商标网（http://sbj.cnipa.gov.cn/）

1. 使用全国企业信用信息公示系统查到公司法人、经营状态、社保缴纳人数、企业失信、行政处罚等信息
2. 使用中国裁判文书网查找公司是否有劳务诉讼（欠薪、非法裁员、或其他信息）
3. 使用中国执行信息公开网查询公司法人是否为“老赖”，使用搜索引擎查找负面新闻，可以使用“【公司名】 【关键词】”

在脉脉和BOSS直聘上搜这家公司的人，基本可以还原这家公司的人员组成。

搜索某家公司的流量信息，可以用alexa网站排名

搜索某家公司APP的信息，可以用七麦数据

http://www.itjuzi.com

IT桔子是国内专注于提供互联网公司及产品数据服务的公司，其官网和 App 可以按照行业分类来筛选项目，虽然基于数据聚合的技术存在部分公司或产品信息不全的情况，但不管是对投资人还是创业者来说，IT桔子是目前国内公司信息查询的首选平台。

如果出现了劳动仲裁，要小心。

公司频繁更换法人也需要注意。



## windows，linux常用命令

(1)ping 127.0.0.1

检查本地TCP/IP协议的安装或则运行是否存在某些最基本的问题。这是回路测试ip，ping回路测试ip时，命令被发送到本地的计算机的ip协议软件。

(2)ping localhost

localhost是网络保留名，代表计算机本身。是127.0.0.1的别名，每台计算机都能将该名字转换成该ip地址。因此，如果使用该命令没有收到应答，表示主机文件（/windows/host）存在问题；

(3)ping 本机ip

用于检测本地主机网卡和调制解调器（猫）是否有问题。

IP地址（Internet Protocol Address 互联网国际地址）地址划分，Excel使用

(4)ping 局域网内其他ip

检测同网段主机间网络线路是否出现故障。该命令会经过网卡和网线发送到目的计算机，再返回。收到回送应答表明本地网络中的网卡和载体运行正常，如果收到0个回送应答，那么表示子网掩码不正确或者网卡配置错误或者网线有问题；若网络中还包含有路由器，则应先Ping路由器在本网段端口的IP，不通则此段线路有问题，通则再PING路由器在目标计算机所在网段的端口IP，不通则是路由出现故障，通则再Ping目的机IP地址。

要查看同网段正在使用的ip可以在cmd命令窗口输入for /L %i IN (1,1,254) DO ping -w 2 -n 1 10.34.0.%i

电脑突然连接不上网络诊断顺序

1.ping 127.0.0.1 先检查TCP/IP协议栈是否正常。

2.ping 本地ip 检查你网卡是否工作正常。

3.ping 网关地址 检查和网关连接性。

4.ping 远程网站 检查远程连接

## ping命令返回信息分析

1.Request timed out(请求超时)

（1）对方已关机或者网络上没有这个地址

（2）对方与自己不在同一个网段内，通过路由也无法找到对方

（3）对方存在但是设置了ICMP数据包过滤屏蔽（如防火墙设置）

怎么知道对方是存在还是不存在呢？可以用ping -a探测对方，如果能得到对方的NetBIOS名称，说明对方存在，但是有防火墙设置。如果得不到，多半是对方关机或者不存在，或不在同一个网段。

（4）网卡IP地址设置错误，如网卡手动设置出现问题。

2.Destination host Unreachable（目的主机不可达）

（1）对方和自己不在同一个网段内，而自己又没有设置默认的路由。

（2）网线出现了故障。

这里要分辨出“Destination host Unreachable”和“time out”的区别，所经过的路由器的路由表中具有到达目标的路由，而目标因为其他原因不可到达，这时候会出现“time out”，如果路由表中连到达目标的路由都没有，就会出现“Destination host Unreachable”

3.Bad IP address

表示可能没有连接到DNS服务器，所有没有办法解析这个ip地址，也可能是ip地址不存在。

4.Source quench received

表示对方或者中途的服务器繁忙无法回应。

5.Unknown host（不知名主机）

该远程主机名字不能被DNS服务器转换为IP地址。故障原因可能是域名服务器有问题或者其名字不正确，或者网络管理员的系统与远程主机之间的同信线路有故障。

6.No answer（无响应）

说明本地系统有一条通向中心主机的路由，但是却接受不到它发给中心主机的任何信息。故障原因可能是：中心主机没有工作；本地或中心主机网络配置不正确；本地或者中心的路由器没有工作；通信线路故障；中心主机存在路由选择问题。

7.TTL

可以通过ping返回的TTL值来判断数据报已经通过了多少个路由器。TTL的初值通常是对方系统的缺省值，每个操作系统可能不一样（但可以更改），是一个2的乘方数。那么源地点TTL的初始值减去返回时的TTL的值就是中间经过路由器的跳数。由于每个路由器至少要把TTL值减一，因此TTL通常表示包在被丢弃之前最多能经过的路由器个数。当计数到0时，路由器决定丢弃该包，并发送一个ICMP报文给最初的发送者。

例如，TTL返回值为51，那么我们可以推算数据报离开源地址的TTL初始值为64（2的6次方），所以从源地点到目的地点要通过13个路由网段（64—54）

另外TTL值还能帮我们分辨操作系统类型。例如

Unix及类Unix操作系统ICMP应答的TTL值一般为255；

Windows 2003/XP操作系统ICMP应答的TTL值一般为128；

Windows 7/8操作系统ICMP应答的TTL值一般为64；

Windows 10操作系统ICMP应答的TTL值一般为128；

Linux操作系统ICMP应答的TTL值一般为64；

MacOS/MacTCP操作系统ICMP应答的TTL值一般为60；

网络攻击者可以通过这种方式来判断别人的操作系统，以便于采用不同的策略来攻击。前提是这个初始值没有被更改过。

一个IPv6的地址使用冒号十六进制表示方法：128位的地址每16位分成一段，每个16位的段用十六进制表示并用冒号分隔开，例如：

一个普通公网IPv6地址：2001:0D12:0000:0000:02AA:0987:FE29:9871

IPv6地址支持压缩前导零的表示方法，例如上面的地址可以压缩表示为：

2001 12:0:0:2AA:987:FE29:9871

为了进一步精简IPv6地址，当冒号十六进制格式中出现连续几段数值0的位段时，这些段可以压缩为双冒号的表示，例如上面的地址还可以进一步精简表示为：

2001 12::2AA:987:FE29:9871

又例如IPv6的地址FF80:0:0:0:FF:3BA:891:67C2可以进一步精简表示为：

FE80::FF:3BA:891:67C2

这里值得注意的是：双冒号只能出现一次。

## 永恒之蓝测试

在CMD窗口下，输入如下命令：netstat –ano | findstr “445”，找出相关进程号，其中SYN_SENT状态，很显然，该电脑被感染永恒之蓝病毒了

找出相关进程号对应的进程，在cmd下输入tasklist | findstr 20164，看看是哪个进程产生该数据包

打开任务管理器，找到相应的进程，先打开文件位置，然后结束进程

## 开放端口有什么风险？

严格意义上讲，开放端口并不一定存在风险，任何一个特定的端口也并不一定比其他端口更安全或者更危险。

端口本身是中立的。开放端口是否存在风险，由使用端口的服务、管理服务的人来决定。当使用端口的服务存在漏洞、没有及时打补丁，或者服务配置错误、安全策略不合理时，开放端口是危险的。

网络攻击通常从端口扫描开始。攻击者将从开放端口的服务中寻找安全漏洞，并通过成功的漏洞利用非法访问未授权的资源。例如：SMB协议用于组织内部共享文件夹、打印机和串行端口。由于协议本身设计存在缺陷，SMB协议被称为永恒之蓝的零日漏洞利用，导致WannaCry勒索病毒大爆发。在互联网上公开此类服务，必将存在安全风险。除了查找漏洞，攻击者还经常采用暴力破解。例如，针对RDP的暴力破解经常被用于勒索软件攻击，是最常见的入侵手法之一。

如果说开放端口有风险的话，那么这个风险就是，开放端口容易受到攻击。开放任何端口都会增加攻击面，并增加由于漏洞、配置错误和管理不足而受到威胁的可能性。

端口是为通信而存在的。组织应实施必要的审核程序，以确定是否开放端口。如果有运行某个服务的需求，开放相应的端口是有意义的。这时，应从合法渠道获取应用软件、及时检查安全漏洞并实施安全加固措施，然后再正式开放端口。

如前所述，开放任何端口都会增加攻击面，并增加受到威胁的可能性。如果没有合理的通信需求，不要开放端口。

组织内部网络中有大量计算机，每台计算机上都可能开放了非必要的端口。你可以逐个计算机去关闭端口，也可以把这项工作交给防火墙。防火墙通常部署在网络的出口，在防火墙上可以阻断此类端口与外部网络的通信。想象你有一个四合院（网络），你可以关闭每个房间（计算机）的门窗（端口），你也可以依赖院墙，然后守好四合院的大门。

检查开放的端口。使用开源的端口扫描工具（如Nmap）或者部署华为漏洞扫描产品VSCAN，可以发现网络中开放的端口。

评估开放端口的必要性。安全专家建议，仅在特定设备上开放必要的端口，非必要的端口应立即关闭。如果扫描结果中出现了未主动开放的端口，请检查主机是否被植入了木马程序。

评估开放端口的安全性。了解每个端口上承载的服务，了解端口可能的风险。端口的安全风险可以从三个方面来评估。

## tcp连接全过程各种状态详解

LISTEN：侦听来自远方的TCP端口的连接请求

SYN-SENT：再发送连接请求后等待匹配的连接请求（客户端）

SYN-RECEIVED：再收到和发送一个连接请求后等待对方对连接请求的确认（服务器）

ESTABLISHED：代表一个打开的连接

FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认

FIN-WAIT-2：从远程TCP等待连接中断请求

CLOSE-WAIT：等待从本地用户发来的连接中断请求

CLOSING：等待远程TCP对连接中断的确认

LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认

TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认

CLOSED：没有任何连接状态

tcp连接建立（tcp三握手）

TCP连接的建立采用客户-服务器模式：主动发起连接建立的应用进程叫做客户，被动等待连接建立的应用进程叫做服务器。

连接建立阶段：

第一次握手：客户端的应用进程主动打开，并向服务端发出请求报文段。其首部中：SYN=1,seq=x。

第二次握手：服务器应用进程被动打开。若同意客户端的请求，则发回确认报文，其首部中：SYN=1,ACK=1,ack=x+1,seq=y。

第三次握手：客户端收到确认报文之后，通知上层应用进程连接已建立，并向服务器发出确认报文，其首部：ACK=1,ack=y+1。当服务器收到客户端的确认报文之后，也通知其上层应用进程连接已建立。

在这个过程中，通信双方的状态如下图，其中CLOSED：关闭状态、LISTEN：收听状态、SYN-SENT：同步已发送、SYN-RCVD：同步收到、ESTAB-LISHED：连接已建立

至此，TCP连接就建立了，客户端和服务器可以愉快地玩耍了。只要通信双方没有一方发出连接释放的请求，连接就将一直保持。

tcp连接终止协议*（tcp四挥手）

由于TCP连接是全双工的，因此每个方向都必须单独进行关闭。这原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个 FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

（1） TCP客户端发送一个FIN，用来关闭客户到服务器的数据传送。

（2） 服务器收到这个FIN，它发回一个ACK，确认序号为收到的序号加1。和SYN一样，一个FIN将占用一个序号。

（3） 服务器关闭客户端的连接，发送一个FIN给客户端。

（4） 客户端发回ACK报文确认，并将确认序号设置为收到序号加1。



## 热备冷备

windows备份

dism++(支持热备)

驱动天空ns5备份功能（可封装系统母盘）

ghost备份

linux

rsync定时传输文件到设定位置

ghost备份

nas(ABB套机)实时备份linux+windows

SyncBack

SyncBackPro中文正式版现在可以同时上传和下载多个文件，这大大提高了性能，我们已经在努力将此功能添加到FTP / SFTP和SyncBack Touch，由于文件现在可以并行压缩，因此多zip文件复制速度也更快，对于虚拟机和数据库等大型文件，您可能只想复制和存储更改而不是整个文件来节省存储空间，SyncBackPro现在通过现有的版本控制功能支持Delta-Copy，我们还将Delta-Copy添加到SyncBack Touch，其中只有更改通过网络传输，整个文件在另一端重建。两个新的FTP / FTPS引擎和一个新的SFTP引擎。当您需要使用远程FTP或SFTP服务器时，这为您提供了最佳的兼容性。为了提高文件扫描性能，我们与CompleteFTP的创建者合作，以便能够在一次FTP调用中扫描整个文件夹树。

备用

使用SyncBack备份数据非常简单，选择要备份的内容和位置，创建计划，设置它并忘记它，快速备份，线程文件复制和并行文件传输将确保备份尽快运行。

同步

智能同步为您提供最佳的可配置性，您可以决定SyncBack在每种情况下应该做什么，例如，如果文件已更改，或者已删除，或者已创建新文件等，应该执行的操作等。

云存储服务

SyncBackPro支持所有主要的云存储服务 Amazon S3™，Google Storage™，Google Drive™，Microsoft Azure™，Microsoft OneDrive™，OneDrive for Business（Office 365）SharePoint™（Office 365），Dropbox™，Box，SugarSync™ ，Amazon Drive™，OpenStack，Backblaze™B2，OVH™，Egnyte™，hubiC™和WebDAV。支持并行和线程文件传输。

FTP

SyncBackPro支持FTP，FTPS和SFTP，为了获得最大兼容性，可以选择三种不同的FTP引擎，支持所有主要的FTP命令以及自定义的命令，如CompleteFTP中引入的新的更快的文件扫描方法，自动备份您的网站从未如此简单。

SyncBack Touch

SyncBack Touch是一种跨平台服务（Windows，macOS，Linux和Android），可让SyncBack远程访问设备的文件系统，以执行备份/恢复和同步操作。SyncBackPro和SyncBackSE提供两个与SyncBack Touch的免费连接。

SyncBack管理服务

SyncBack管理系统（SBMS）通过一个简单易用的解决方案提高安全性，问责制和报告。它为系统管理员提供了宝贵的工具，可以监控和管理多个SyncBackPro安装。

复制

您现在可以只复制和存储对文件的更改，备份VHD或数据库文件并节省大量存储空间，SyncBack Touch（适用于Windows）也得到了改进，允许通过网络进行Delta-Copy，从而减少网络使用。

版本

永远不要丢失旧的文件副本，使用SyncBack，您可以指定要保留的文件版本数量以及保留时间，所有云存储服务，FTP，NAS驱动器，SyncBack Touch等也支持版本控制。

安全

SyncBack支持256位AES文件加密以及PKZip SecureZip（SyncBackPro）勒索软件检测可以阻止备份文件损坏。

官方网站 https://www.2brightsparks.com/

食用说明

1、双击正常安装主程序安装包 SyncBackPro 专业版 ；

2、运行Crack文件夹的SyncBackPro KeyGen解锁钥匙 ；

3、SyncBackPro 专业版河蟹成功尽情的使用吧；

傲梅

系统备份与还原

像一键Ghost系统那样，您只需要点几次鼠标就能将系统盘备份到指定的位置，如备份系统到移动硬盘。以后，如果系统出现问题就可以快速地还原系统，从而避免了重装系统的麻烦。

文件备份与还原

如果只想备份电脑少数的几个目录或文件，怎么办？例如：只需要备份工作目录，只想备份D盘中的Word文档。文件备份与还原功能能只处理特定的文件，让备份功能变得更灵活。

磁盘分区备份与还原

它也称作磁盘分区镜像，能直接备份多个硬盘和多个分区。当然，硬盘和分区中包含的所有数据都将被备份。还原功能除了能直接还原硬盘和分区外，还能选择性地还原特定的文件。

定时备份

对于某些重要的文件，我们可能想定期做一次备份。定时备份功能支持按每天，每周，每月，或某个周期自动做您指定数据的备份。

增量与差异备份

增量与差异备份只会备份中相比上一次改变过的数据，此功能能极大地提高备份速度和降低镜像文件所占用的空间。定时备份默认将使用增量备份的方式来优化备份的效率。

备份后Email通知

自动备份完成后，是不是想知道备份的结果怎么样了，Email通知功能允许您设置是否将备份日志报告发到您的邮箱。于此，让您更安心！

磁盘与分区克隆

克隆一个硬盘到另一个硬盘

硬盘1 -> 磁盘2

克隆硬盘上的一个分区到另一个硬盘D: -> F:

系统克隆与系统迁移

克隆系统或迁移系统到固态硬盘或其它普通硬盘

支持系统如下

Windows 10/7/8/8.1、Windows XP/Vista、Windows Server 2003/2008/2012/2016、Windows SBS 2003, 2008, 2011

支持存储如下

各种接口的硬盘如：IDE、SATA、SCSI、IEEE1394、USB1.1/2.0/3.0、各种类型的硬盘如：机械硬盘、固态硬盘(SSD)、U盘、移动硬盘、RAID盘、2TB+硬盘、各种格式的磁盘如：MBR磁盘、GPT磁盘、动态卷、NAS盘、备份数据到各种形式的光盘，如：CD-R/RW、DVD-R/RW、DVD+R/RW。

如果注册反弹请把以下加入到Windows\System32\drivers\etc\hosts文件里面以屏蔽联网验证；

127.0.0.1 www.aomeitech.com

无人值守静默安装参数：

/S =绝对路径默认C盘程序文件夹

/D E:\MyProgram ＝自定义路径

如果有较多文件夹不便移动，可以将文件夹路径添加至排除列表中 (菜单栏 – 详细设置 – 排除列表设置)，如 iCloud File、Tencent Files、Wechat Files。关于 ExclusionList 的高级用法可参照 Microsoft – Dism Configuration，此设置仅在 Dism 的映像备份中可用。

在文件保存对话框中，需要选择一个非系统盘的位置来保存备份映像；映像名称按照一定格式命名（系统备份_日期_系统版本），如「D:\Sys_Backup\Sys_Full_Backup-200804-Win10.wim」；压缩率按照需要选择，映像大小与制作所需时间、压缩率成反比，压缩率越高，制作所需的时间越长，映像文件越小，这里一般选择高压缩（Wim 格式映像）即可。

在 D 盘的 Sys_Backup 文件夹中找到备份的映像文件，然后将其拷贝至移动硬盘等安全的位置，需要时能够快速调用。后期如果需要修改备份映像也十分简单，在保存路径中选择先前的备份映像，Dism++ 会自动执行增量备份。

 wmic 常用命令

```shell
 ## wmic 常用命令
 
 C:\Users\Administrator   wmic 
 wmic:root\cli   baseboard get manufacturer,product 
 Manufacturer           Product 
 ASUSTeK COMPUTER INC.  H510M-D3H/M.2 R2.0 
  
 wmic:root\cli    
  
 列出所有的进程 : wmic process. 
  
 显示进程的详细信息 
 输入 process where name=   chrome.exe    list full 
  
 停止进程的操作 
 例如，执行下面的命令将关闭正在运行的QQ.exe： 
 例1、wmic process where name=   QQ.exe    call terminate 
 例2、wmic process where name=   qq.exe    delete 
  
 显示出BIOS信息 
 wmic bios list full 
  
 C:\Users\Administrator   wmic baseboard get manufacturer,name,product 
 Manufacturer           Name  Product 
 ASUSTeK COMPUTER INC.  基板  H510M-D3H/M.2 R2.0 
  
 CPU    CPU 管理 
 查看cpu型号 
 wmic cpu get name 
  
 C:\Users\Administrator   wmic cpu get name 
 Name 
 11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz 
  
 DISKDRIVE  物理磁盘驱动器管理 
  
 获取物理磁盘型号大小等 
 wmic DISKDRIVE get Caption,size,InterfaceType 
  
 C:\Users\Administrator   wmic diskdrive get caption,size,interfacetype 
 Caption                               InterfaceType  Size 
 SSK  SCSI Disk Device                 SCSI           512105932800 
 Kingston DataTraveler 3.0 USB Device  USB            123724661760 
 Great Wall GW3300 256GB               SCSI           256052966400 
  
 LOGICALDISK    本地储存设备管理 
 获取硬盘系统格式、总大小、可用空间等 
 wmic LOGICALDISK get name,Description,filesystem,size,freespace 
  
 C:\Users\Administrator   wmic logicaldisk get name,description,filesystem,size,freespace 
 Description   FileSystem  FreeSpace     Name  Size 
 本地固定磁盘  NTFS        79900905472   C:    161063366656 
 本地固定磁盘  NTFS        88761200640   D:    94663675904 
 本地固定磁盘  NTFS        138470047744  E:    512108785664 
 可移动磁盘    exFAT       70374952960   F:    122979606528 
 可移动磁盘    FAT         108486656     G:    311771136 

```

## Oracle基本操作

以具有足够权限的用户身份登录操作系统：通常需要具有管理员权限或oracle用户权限。

停止监听器（如果需要）:

打开命令提示符或终端，输入以下命令来停止监听器服务：

lsnrctl stop

通过SQL*Plus登录数据库:

执行如下命令以系统管理员身份登录SQL*Plus：

sqlplus / as sysdba

关闭数据库:

在SQL*Plus中，执行以下命令以正常方式关闭数据库（这可能需要等待所有活动会话结束）：

SHUTDOWN IMMEDIATE;

如果需要快速关闭而不等待事务完成，可以使用SHUTDOWN ABORT，但这可能会导致未提交的事务丢失。

启动数据库:

数据库关闭后，再次在SQL*Plus中执行以下命令以启动数据库：

STARTUP;

这个命令会启动数据库实例并打开数据库。

启动监听器（如果之前已停止）:

回到命令提示符或终端，输入以下命令启动监听器：

lsnrctl start

验证数据库和监听器状态:

oracle基本命令

DDL (Data Definition Language) 常用命令

创建表:

Sql

CREATE TABLE table_name (

 column1 datatype,

 column2 datatype,

 …

);

修改表结构:

Sql

ALTER TABLE table_name

ADD column_name datatype;

ALTER TABLE table_name

MODIFY column_name datatype;

ALTER TABLE table_name

DROP COLUMN column_name;

删除表:

Sql

DROP TABLE table_name CASCADE CONSTRAINTS;

创建用户:

Sql

CREATE USER username IDENTIFIED BY password DEFAULT TABLESPACE users QUOTA unlimited ON users;

授予权限:

Sql

GRANT privilege TO username;

DML (Data Manipulation Language) 常用命令

插入数据:

Sql

INSERT INTO table_name (column1, column2, …) VALUES (value1, value2, …);

更新数据:

Sql

UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;

删除数据:

Sql

DELETE FROM table_name WHERE condition;

查询数据:

Sql

SELECT column1, column2, … FROM table_name [WHERE condition] [ORDER BY column ASC|DESC];

DCL (Data Control Language) 常用命令

授予权限 (已在DDL部分提及)

回收权限:

Sql

REVOKE privilege FROM username;

其他常用命令

连接数据库:

Sql

sqlplus username/password@database

显示当前用户:

Sql

SELECT user FROM dual;

查看表空间信息:

Sql

SELECT * FROM dba_tablespaces;

启动/关闭数据库 (需要以SYSDBA身份):

启动:

Sql

STARTUP;

关闭:

Sql

SHUTDOWN IMMEDIATE;

启动/停止监听器:

启动监听器:

Cmd

lsnrctl start

停止监听器:

Cmd

lsnrctl stop

备份恢复命令

备份方案与操作步骤：

配置RMAN:

确保RMAN已正确配置并与数据库相连。这通常涉及到设置RMAN的配置参数，如备份保留策略、备份目的地等。

创建备份脚本（可选）:

为了自动化备份过程，可以创建RMAN脚本文件，包含备份命令。

执行全备:

打开命令行工具，如CMD或PowerShell，使用如下命令执行全库备份：

Cmd

rman target /

RMAN> RUN {

 BACKUP AS COMPRESSED BACKUPSET DATABASE INCLUDE CURRENT CONTROLFILE FORMAT ‘%d_DB_%U’;

 }

这里%d和%U是格式化字符串，代表日期和唯一标识符，帮助组织备份文件。

执行增量备份（可选）:

如果需要，可以执行增量备份来节省存储空间。例如，执行级别1的增量备份：

Cmd

RMAN> RUN {

 BACKUP INCREMENTAL LEVEL 1 DATABASE;

 }

归档日志备份:

为了完全恢复，也需要定期备份归档日志：

Cmd

RMAN> BACKUP ARCHIVELOG ALL;

恢复方案与操作步骤：

模拟恢复测试（推荐）:

在真正需要恢复前，先进行一次模拟恢复测试，确保恢复流程无误。

启动RMAN并连接到数据库:

在命令行中，连接到数据库：

Cmd

rman target /

关闭数据库:

如果数据库还开着，需要先关闭：

Sql

RMAN> SHUTDOWN IMMEDIATE;

启动数据库到挂载状态:

接下来，启动数据库到挂载状态，以便进行恢复操作：

Sql

RMAN> STARTUP MOUNT;

恢复数据库:

使用备份文件恢复数据库：

Sql

RMAN> RESTORE DATABASE;

应用归档日志:

应用自上次备份以来的所有归档日志，确保数据库的一致性：

Sql

RMAN> RECOVER DATABASE;

打开数据库:

恢复完成后，打开数据库：

Sql

RMAN> ALTER DATABASE OPEN RESETLOGS;

虽然Oracle 9i数据库版本较旧，但基本的备份与恢复原理与后续版本类似，主要也是依赖于RMAN（Recovery Manager）工具。下面是针对Windows环境下Oracle 9i数据库的备份与恢复方案以及操作步骤概述：

备份方案与操作步骤：

配置RMAN环境

确保RMAN已安装并配置：首先确认RMAN客户端已安装，并且环境变量如ORACLE_HOME和ORACLE_SID已正确设置。

创建RMAN脚本（可选）：为了自动化备份流程，你可以创建一个RMAN脚本来执行定期备份任务。

执行备份

启动RMAN：打开命令提示符（CMD），输入rman target /以以sysdba身份连接到数据库。

执行全备份：

Cmd

RMAN> BACKUP AS COMPRESSED BACKUPSET DATABASE PLUS ARCHIVELOG DELETE INPUT;

此命令将创建一个压缩的数据库备份，并包括所有归档日志，完成后删除已备份的归档日志文件。

配置备份策略（如增量备份）：

如果需要，可以配置增量备份策略以减少备份所需的存储空间。

Cmd

RMAN> CONFIGURE CONTROLFILE AUTOBACKUP ON;

RMAN> CONFIGURE RETENTION POLICY TO REDUNDANCY 2; — 根据需要设置保留策略

RMAN> BACKUP INCREMENTAL LEVEL 1 DATABASE;

恢复方案与操作步骤：

准备恢复环境

停止所有与数据库相关的服务：包括数据库服务和监听器服务，确保没有活动连接。

启动RMAN：如同备份步骤一样，打开命令提示符并连接到数据库。

执行恢复

关闭数据库（如果尚未关闭）：

Sql

RMAN> SHUTDOWN IMMEDIATE;

启动到nomount状态：

Sql

RMAN> STARTUP NOMOUNT;

恢复控制文件（如有必要）：

如果控制文件丢失，需要先从备份恢复控制文件。

加载数据库：

Sql

RMAN> STARTUP MOUNT;

执行恢复操作：

如果是全库恢复：

Sql

RMAN> RESTORE DATABASE;

RMAN> RECOVER DATABASE;

如果是特定文件恢复，需要指定文件：

Sql

RMAN> RESTORE DATAFILE <file_number>;

RMAN> RECOVER DATAFILE <file_number>;

打开数据库：

Sql

RMAN> ALTER DATABASE OPEN RESETLOGS;

注意事项

在进行任何恢复操作之前，请确保有足够的备份，并了解所执行命令的影响。

如果数据库配置了归档模式，恢复过程中可能需要应用归档日志以保证数据的一致性

登录，搜索

查找密码修改密码

## 瑞星防火墙配置

0.设置只允许某ip段可以访问端口

一、设置白名单,配置通过ip,同时端口设置所有端口范围禁止tcp,udp，方向本地禁止，

二、ip规则配置，（清除默认规则)

顶部配置：所有ip包，远程允许ip范围，本地任意，协议类型all,放行，弹窗提示

顶部之下配置：所有ip包，本地任意，本地任意，协议类型all，阻止

1.启用端口隐身，高频阻拦，查看扫描ip,添加黑名单

2.封禁所有不再使用的端口，只给需要的端口打补丁

需要在设置中确定方向，可以检测到端口，状态为filtered

3.一直开启端口隐身，使用时打开，不用关闭

## windows系统数据备份恢复

snap

原生

ghost

ventoy制作启动优盘，iventoy批量装机windows+linux

网刻启动盘

支持pxe,批量安装系统

批量（重）装系统：iventoy，网刻启动盘，支持pxe,批量安装系统

iso传统镜像文件

## 绝区零账号

```shell
</div>
<div>
<div>zhaojiangtest@163.com</div>
<div></div>
<div>bbaeeww9839592@163.com</div>
<div></div>
<div>liangzhui765duy@163.com</div>
<div></div>
<div>pmvhmr0509@163.com</div>
<div></div>
<div>zhaoj5811@gmail.com</div>
<div></div>
<div>ktamdz@163.com</div>
<div></div>
<div></div>
<div>testzjb@163.com</div>
<div></div>
<div></div>
<div>3044483124@qq.com</div>
<div></div>
<div>15346361841</div>
<div></div>
<div>15737092152</div>
<div>mm</div>
<div>12345678z.</div>

```

## 李金良

123456zxc

联系人：李金良

账号：13790312651

密码：Li131288

手机号|：同账号

gy13790312651

Li131288@

学习资料

https://blog.kejilion.pro/kejilion-sh-commands/

VPS优惠链接

https://kejilion.pro/index.php/topVPS/

科技lion博客

https://blog.kejilion.pro/

科技lion官网

https://kejilion.pro/

Centos 静态ip设置，无法ping通外网，排查思路

https://blog.csdn.net/qq_41523340/article/details/123365282

## vps科学上网+个人网站+图床

去公安机关做笔录前需要了解的7个问题

（希望大家不会用到）

1.核实身份

确认询问/讯问人员的身份，一方面确保他们是正规的执法官员，有权进行询问/讯问，另一方面了解到是哪个机关在具体办案，有助于后续的案件办理。办案人员的姓名和工作单位在笔录开头部分会注明，要留心查看。

2.了解刑事案件程序期限

被传唤者要了解一个最重要的数字–48小时。48小时其实是由2个24小时组成的。

第一个24小时：

按照规定，传唤、拘传持续的时间不得超过十二小时；案情特别重大复杂，需要采取拘留、逮捕措施的，传唤、拘传持续的时间不得超过二十四小时。也就是说，如果超过24小时公安机关还没放人，那一般就是被刑事拘留了。宣布拘留时，公安机关会让你签《拘留证》。被拘留时，你可能会立即知道，但你的家属不一定知道，因为《拘留通知书》大多都是以挂号信的形式邮寄，等家属收到时，你可能早已经不在被传唤的派出所了。

第二个24小时：

在拘留后，办案机关需在24小时内将人送看守所。送到看守所后，就一定程度上脱离了讯问人员的掌控，在看守所有相对固定的作息时间。

上述2个24小时，合计48小时，为什么要牢记48小时？因为传唤时间超过24小时便是违法，第一个24小时之内，要么放你走，要么把你拘留。而拘留后24小时，要么放你走，要么送到看守所。超过规定时限取得的讯问材料，可能作为后续申请非法证据排除的理由。在这48小时之内，你完全处于办案机关的控制之下，所以常被办案人员称之为“黄金48小时”。这48小时内，你的口供在很大程度上会决定案件的走向，通过密集的讯问，公安机关会对你有罪还是无罪、构成此罪还是彼罪有了预判。办案人员想的是如何突破你的口供，而你应该想的是如何不被冤枉。熬过这48小时，一般就会被送到看守所里面了。

此阶段注意：犯罪嫌疑人在这48小时内需要签署《传唤证》、《拘留证》，到案的时间和结束讯问的时间都要在上面载明，签署时精确到X年X月X日X时，这两证会随案移送起诉机关和审判机关，务必如实填写，切勿因为配合侦查机关不如实签署时间。

3.了解讯问权利

了解您的法律权利这非常重要，清楚自己的权利才能在应对讯问时有充足的底气。以下是讯问中几项重要的权利：

第一， 按照法律规定，“不得强迫任何人证实自己有罪”。

第二，拒绝回答与案件无关问题的权利。

第三，你有核对、补充、修改笔录的权利。

一定要核对笔录，一定要核对笔录，一定要核对笔录。重要的事情说三遍！

4.不做猜测性回答

首先，注意询问/讯问者的问题是否清晰明确，如有不理解，应要求对方解释清楚后再回答。

其次，对于自己记不清的、自己从别人那听说的不确定事项，不要轻易回答，避免被办案机关加工成自己的明知。对于不清楚的事情可以直接回答“我不清楚”。

5.关注有无同步录音录像

注意讯问室内同步录音录像设备是否开启。是否进行同步录音录像，这在某些案件中可能是法定要求，有助于确保笔录的真实性和完整性。同步录音录像作用是两方面的，一方面有利于犯罪嫌疑人，防止发生刑讯逼供、确保笔录被准确记录，一方面有利于侦查人员，防止被讯问人突发疾病发生意外时被追究责任。

6.休息与饮食的权力

长时间询问时，有权要求休息和获得饮食的权利。《中华人民共和国刑事诉讼法》第一百一十九条规定：“传唤、拘传犯罪嫌疑人，应当保证犯罪嫌疑人的饮食和必要的休息时间。”并不是“不老实交待”就不能休息和吃饭。超过规定时间不让休息吃饭，相关笔录可能会作为非法证据被排除。

7.最好事先咨询律师

在前往公安机关之前，如果条件允许，最好能够事先咨询法律专业人士，以便更好地准备和保护自己的权益。

 【来源：https://ruanjianku.cloud/gongan/，转载请注明】

一台可科学上网的 VPS 服务器

ok

一个域名

域名托管到 Cloudflare 并做好服务器的 DNS 解析

AutoIt是一个强大的脚本语言，主要用于Windows操作系统的自动化。它被设计来自动化Windows GUI（图形用户界面）和一般的脚本任务，如自动点击、填写表单、处理文件和目录等。由于它的简单性和强大的功能，AutoIt广泛应用于软件测试、批量操作和日常重复任务的自动化。

Sysinternals 工具涵盖了 Windows 系统管理的诸多关键方面，包括进程管理、内存控制、网络监控、磁盘管理、故障排查以及安全性。这些工具不仅仅是简单的软件集合，更是专为 IT 专业人员和开发人员设计，以提高系统性能、解决问题并增强系统的安全性。

Boot Time: <Boot Time>

User Name: <User Name>

Host Name: <Host Name>

IP Address: <IP Address>

CPU: <CPU>

Default Gateway: <Default Gateway>

DNS Server: <DNS Server>

MAC Address: <MAC Address>

Memory: <Memory>

Network Speed:<Network Speed>

OS Version: <OS Version>

Snapshot Time: <Snapshot Time>

System Type: <System Type>

Volumes: <Volumes>

Space: <Free Space>

Terminus

ssh终端，不好用，好看

Zabbix 由 Alexei Vladishev 创建，目前由 Zabbix SIA 主导开发和支持。

Zabbix 是一个企业级的开源分布式监控解决方案。

Zabbix 是一款监控网络的众多参数以及服务器、虚拟机、应用程序、服务、数据库、网站、云等的健康和完整性的软件。Zabbix 使用灵活的通知机制，允许用户为几乎任何事件配置基于电子邮件的告警，以实现对服务器问题做出快速反应。Zabbix 基于存储的数据提供出色的报告和数据可视化功能。这使得 Zabbix 成为容量规划的理想选择。

Zabbix 支持轮询和 trapping。所有 Zabbix 报告和统计数据以及配置参数都可以通过基于 Web 的前端访问。基于 Web 的前端确保可以从任何位置评估您的网络状态和服务器的健康状况。如果配置得当，不管对于拥有少量服务器的小型组织还是拥有大量服务器的大公司来讲，Zabbix 都可以在监控 IT 基础设施方面发挥重要作用。

Zabbix 是免费的。Zabbix 是在 GPL 通用公共许可证第 2 版下编写和分发的。这意味着它的源代码是免费分发的，可供公众使用。

nagios

Nagios是一款开源的免费网络监视工具，能有效监控Windows、Linux和Unix的主机状态，交换机路由器等网络设备，打印机等。 在系统或服务状态异常时发出邮件或短信报警第一时间通知网站运维人员，在状态恢复后发出正常的邮件或短信通知。

observium

Observium 可以满足你的需求。这是一个免费的监控系统，它可以帮助你远程监控你的服务器。它是一个由PHP编写的基于自动发现 SNMP 的网络监控平台，支持非常广泛的网络硬件和操作系统，包括 Cisco、Windows、Linux、HP、NetApp 等等。

netxms

NetXMS是一款开源的网络管理系统监控工具，功能强大，拥有丰富工具集，遵守GPL2开源许可。它可用于监测整个IT基础设施，从支持SNMP的硬件（如交换机和路由器）到服务器及应用程序。 支持Windows 2003+、Linux、Solaris、AIX、HP-UX以及FreeBSD等多套系统平台。





**linux命令行下面执行某个命令的时候，首先保证该命令是否存在，若存在，但输入命令的时候若仍提示：command not found**

## **这个时候就的查看PATH环境变量的设置了，当前命令是否存在于PATH环境变量中**

```bash
#查看PATH：
echo $PATH
举例说，命令 composr 在/usr/loca/bin
```

但执行的时候提示：

-bash: composr: command not found
这个时候，通过echo $PATH，，发现composer并未在PATH环境变量中有设置，这个时候就需要把composer所在路径添加到PATH中

所以需要修改PATH环境变量，具体如下：

```bash
方法一：
 
export PATH=/usr/local/bin:$PATH
#配置完后可以通过echo $PATH查看配置结果。
#生效方法：立即生效
#有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢#复原有的path配置
#用户局限：仅对当前用户
方法二：
 
#通过修改.bashrc文件:
vim ~/.bashrc 
#在最后一行添上：
export PATH=/usr/local/bin:$PATH
#生效方法：（有以下两种）
#1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
#2、输入“source ~/.bashrc”命令，立即生效
#有效期限：永久有效
#用户局限：仅对当前用户
方法三：
 
#通过修改profile文件:
vim /etc/profile
export PATH=/usr/local/bin:$PATH
#生效方法：系统重启
#有效期限：永久有效
#用户局限：对所有用户
方法四：
 
#通过修改environment文件:
vim /etc/environment
在PATH="/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin"中加入 
":/usr/local/bin"
#生效方法：系统重启
#有效期限：永久有效
#用户局限：对所有用户
```

注意事项：
系统在环境变量中寻找程序路径时，是由前往后（win7,8），有上往下(win10)的方式寻找的，找到第一个后九会直接使用它而不会再往后往下寻找了。
所以在将程序路径复制进入到path或者其他环境变量名中时，需要如果原先已经配置过这个程序的路径，那么系统会使用先找到的那个路径，所有为了保证系统使用的是我们最新配置的路径，一般我们直接把新配置的路径写在最上面或者最前面。



## linux杂谈：一些常用命令

```csharp
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

**网卡无法启动**

```css
用ifconfig查看，发现网卡不见了。用ifconfig -a查看，发现ens33网卡是在的，只是状态是down的。
临时的解决方法：
 
使用ifconfig命令，直接启动ens33网卡
 
[root@centos7 ~]# ifconfig ens33 up
 
然后，临时配置一个IP地址
 
[root@centos7 ~]# ifconfig ens33 192.168.100.1 netmask 255.255.255.0
 
注意：此方法再虚拟机重启后失效。
```

**RPM 包的常用操作命令**

```go
1. 安装
rpm -ivh package.rpm：安装指定的 RPM 包，其中-i表示安装，-v表示显示详细信息，-h表示以哈希符号显示安装进度。
 
2. 升级
rpm -Uvh package.rpm：升级指定的 RPM 包，如果系统中未安装该包，则会进行安装。-U选项用于升级操作。
 
3. 卸载
rpm -e package：卸载指定的已安装软件包，-e表示卸载操作。
 
4. 查询
rpm -qa：查询系统中已安装的所有 RPM 包。可以通过管道符|与grep命令结合使用，来查找特定的软件包。例如，rpm -qa | grep openssh可以查找系统中已安装的 OpenSSH 相关的 RPM 包。
rpm -qi package：查询指定软件包的详细信息，包括软件的描述、版本、依赖关系等。
rpm -ql package：查询指定软件包安装的文件列表，显示该软件包在系统中安装的所有文件及其路径。
```

**SSH时出现WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!**

翻译过来就是 警告：远程主机标识已更改！ssh服务是通过公钥和私钥来进行连接的，它会把每个曾经访问过计算机或服务器的公钥（public key），记录在~/.ssh/known_hosts 中，当下次访问曾经访问过的计算机或服务器时，ssh就会核对公钥，如果和上次记录的不同，OpenSSH会发出警告。 使用命令清除所连接的IP ssh-keygen -R XX.XX.XX.XX

**ubuntu拓荒指南**

**1.ubuntu上报E: Unable to locate package update错误的解决方法**

```sql
如果第一次使用系统，软件库缓存没有更新会包找不到软件包的错误。使用下面命令更新
 
sudo apt update
 
注意，上面的命令不是apt install，因为使用apt install，apt 包管理器会搜索缓存以获取包和版本信息，然后通过网络从其存储库下载它。如果软件包不在此缓存中，系统将无法安装软件包。
 
新装系统，缓存是空的。首先应该运行 apt update 命令。这样就不会出现E: Unable to locate package 错误。
```

[ubuntu上报E: Unable to locate package update错误的解决方法 – 知乎](https://zhuanlan.zhihu.com/p/484067483#:~:text=ubuntu上报E%3A Unable to locate package update错误的解决方法 1 1.检查软件包名名称：,4. 检查 Ubuntu 版本的活跃程度 Ubuntu 有两种主要的版本。 常规版本支持九个月，而长期支持版本支持五年。 )

**ubuntu使用ping命令未发现**

```csharp
Sudo apt-get install iputils-ping
```





**在Linux系统使用中，我们经常需要下载文件、对文件进行压缩和解压操作。**

**本文将介绍一些常用的Linux命令，帮助你轻松完成这些操作。**

## 1.文件下载

### 1.1 wget命令

**wget命令是Linux中最常用的下载命令之一。它支持HTTP、HTTPS、FTP等多种协议，并可以设置断点续传等功能。**

基本用法:

wget [选项] URL地址
常用选项:

**-c: 断点重连，继续下载未完成的文件。**
**-O: 指定下载文件的保存名称。**
**-P: 指定下载文件的保存路径。**
**-q: 静默模式，不显示下载进度。**

```shell
 
# 下载文件并保存为默认名称
wget https://www.example.com/file.txt
 
# 下载文件并保存为默认名称，-c为断点重连，如果断网后，下次会接续
wget -c https://www.example.com/file.txt
 
# 下载文件并指定保存名称
wget -O myfile.txt https://www.example.com/file.txt
 
# 下载文件并指定保存路径
wget -P /tmp https://www.example.com/file.txt
 
# 静默模式下载文件
wget -q https://www.example.com/file.txt
wget还有许多其他的选项，可以通过man wget或wget help查看。

```

### 1.2curl命令

**curl命令也是一个常用的下载命令，它支持的功能与wget类似。**

**基本用法:**

**curl [选项] URL地址**
**常用选项:**

**-o: 指定下载文件的保存名称。**
**-L: 跟踪重定向。**
**-s: 静默模式，不显示下载进度。**

```shell
# 下载文件并保存为默认名称
curl https://www.example.com/file.txt
 
# 下载文件并指定保存名称
curl -o myfile.txt https://www.example.com/file.txt
 
# 跟踪重定向并下载文件
curl -L https://www.example.com/redirect.html
 
# 静默模式下载文件
curl -s https://www.example.com/file.txt
curl还有许多其他的选项，可以通过man curl 或则 curl help查看

```

## 2.文件压缩

### 2.1 gzip命令

**gzip命令是Linux中最常用的压缩命令之一，它可以对文件进行 gzip 压缩格式的压缩。**

**基本用法:**

**gzip [选项] 文件名**
**常用选项:**

**-c: 压缩后不删除源文件。**
**-d: 解压缩文件。**
**-f: 强制压缩文件，即使文件已存在。**
**-9: 使用最高压缩级别。**

```shell
# 压缩文件
gzip file.txt

# 解压缩文件
gzip -d file.txt.gz

# 强制压缩文件
gzip -f file.txt

# 使用最高压缩级别压缩文件
gzip -9 file.txt
```

### 2.2 bzip2命令

**bzip2命令也是一个常用的压缩命令，它可以对文件进行 bzip2 压缩格式的压缩。**

**基本用法:**

**bzip2 [选项] 文件名**
**常用选项:**

**-c: 压缩后不删除源文件。**
**-d: 解压缩文件。**
**-f: 强制压缩文件，即使文件已存在。**
**-9: 使用最高压缩级别。**

```shell
# 压缩文件
bzip2 file.txt

# 解压缩文件
bzip2 -d file.txt.bz2

# 强制压缩文件
bzip2 -f file.txt

# 使用最高压缩级别压缩文件
bzip2 -9 file.txt
```

## 3.文件解压

### 3.1 tar命令

**tar命令是Linux中常用的解压命令，它可以解压 tar 格式的压缩包。**

**基本用法:**

**tar [选项] 文件名**
**常用选项:**

**-c: 创建压缩包。**
**-x: 解压缩文件。**
**-f: 指定压缩包文件名。**
**-v: 显示详细信息。**

```
# 解压缩文件
tar -xvf file.tar.gz

# 解压缩文件并显示详细信息
tar -xvzf file.tar.gz

# 创建压缩包
tar -cf myfile.tar file1 file2
```

### 3.2 unzip命令

**unzip命令是Linux中常用的解压命令，它可以解压 zip 格式的压缩包。**

**基本用法:**

**unzip [选项] 文件名**
**常用选项:**

**-d: 指定解压文件的保存路径。**
**-l: 查看压缩包内容。**
**-q: 静默模式，不显示解压进度。**

```shell
# 解压缩文件
unzip file.zip
```

本文介绍了Linux中三个常用的命令：wget、gzip和tar，分别用于文件的下载、压缩和解压。这些命令都有很多的选项，可以根据不同的需求进行调整。通过掌握这些命令，我们可以更方便地管理和处理文件。希望本文对你有所帮助。

# windows系统

//进入d盘
D:
//进入F盘
F:

cd /? //获取使用帮助

cd \ //跳转到硬盘的根目录

cd C:\WINDOWS //跳转到当前硬盘的其他文件

d: //跳转到其他硬盘

cd /d e:\software //跳转到其他硬盘的其他文件夹，注意此处必须加/d参数。否则无法跳转。

cd.. //跳转到上一层目录

//查看当前目录下的文件，类似于linux下的ls

## dir

copy 路径\文件名 路径\文件名 ：把一个文件拷贝到另一个地方。

move 路径\文件名 路径\文件名 ：把一个文件移动（就是剪切+复制）到另一个地方。

//这个是专门删除文件的，不能删除文件夹
del 文件名

## taskkill

taskkill /? 获取使用帮助
TASKKILL /S system /F /IM notepad.exe /T
TASKKILL /PID 1230 /PID 1241 /PID 1253 /T
TASKKILL /F /IM QQ.exe
TASKKILL /F /IM notepad.exe /IM mspaint.exe

## find /？获取使用帮助

netstat -ano|find “.8” //使用管道符，进行模糊查询

“|”cmd命令中|代表前一个的输出代表后一个的输入

查找特定ip的网络连接及进程号：netstat -ano|find “192.168.1.10”

重定向输出符号> >>
将原本输出到命令窗口的内容，转存到文件中，如jstack 12912 >d:/s.txt 打印线程到指定文件

cmd > 重定向输出并覆盖源文件。

echo hello >c:\1.txt // 1.txt的文件内容先被清空，然后写入hello。
cmd >>重定向输出追加到文件末尾

echo hello >>c:\1.txt // 在1.txt文件末尾加上hello

netstat -ano
netstat -ab
端口，进程，进程号
netstat -ano|findstr “”
查看开放使用的端口号，以及占用进程进程号
tasklist|findsrt “”
通过进程号寻找相关进程

 

dfrg.msc——-磁盘碎片整理程序
chkdsk.exe—–Chkdsk磁盘检查
devmgmt.msc— 设备管理器
winver———检查Windows版本

net user 用户名　密码　/add 建立用户
net user 帐户名 查看帐户的属性
net localgroup administrators 用户名 /add 把“用户”添加到管理员中使其具有管理员权限
net start 查看开启了哪些服务
net start 服务名　开启服务；(如:net start telnet， net start schedule)
net stop 服务名 停止某服务
net time 目标ip 查看对方时间
net time 目标ip /set 设置本地计算机时间与“目标IP”主机的时间同步,加上参数/yes可取消确认信息
net user guest /active:yes 激活guest用户
net user guest 12345 用guest用户登陆后用将密码改为12345
net password 密码 更改系统登陆密码
fc one.txt two.txt > 3st.txt 对比二个文件并把不同之处输出到3st.txt文件中，”> “和”> >” 是重定向命令
at 查看所有的计划任务
route print 显示出IP路由，将主要显示网络地址Network addres，子网掩码Netmask，网关地址Gateway addres，接口地址Interface
arp 查看和处理ARP缓存，ARP是名字解析的意思，负责把一个IP解析成一个物理性的MAC地址。arp -a将显示出全部信息
set 显示当前所有的环境变量
findstr “Hello” aa.txt 在aa.txt文件中寻找字符串hello
find 文件名 查找某文件
title 标题名字 更改CMD窗口标题名字
color 颜色值 设置cmd控制台前景和背景颜色；0=黑、1=蓝、2=绿、3=浅绿、4=红、5=紫、6=黄、7=白、8=灰、9=淡蓝、A=淡绿、B=淡浅绿、C=淡红、D=淡紫、E=淡黄、F=亮白
format 盘符 /FS:类型 格式化磁盘,类型:FAT、FAT32、NTFS ,例：Format D: /FS:NTFS
chkdsk /F D: 检查磁盘D并显示状态报告；加参数/f并修复磁盘上的错误

# 批处理命令与变量

## for命令及变量

基本格式： FOR /参数 %variable IN (set) DO command [command_parameters]

%variable:指定一个单一字母可替换的参数，如：%i ，而指定一个变量则用：%%i ，而调用变量时用：%i% ，变量是区分大小写的（%i 不等于 %I）。
批处理每次能处理的变量从%0—%9共10个，其中%0默认给批处理文件名使用，%1默认为使用此批处理时输入的的第一个值，同理：%2—%9指输入的第2-9个值；例：net use ipipc$ pass /user:user 中ip为%1,pass为%2 ,user为%3
(set):指定一个或一组文件，可使用通配符，如：(D:user.txt)和(1 1 254)(1 -1 254),{ “(1 1 254)”第一个”1″指起始值，第二个”1″指增长量，第三个”254″指结束值，即：从1到254；“(1 -1 254)”说明：即从254到1 }
command：指定对第个文件执行的命令，如：net use命令；如要执行多个命令时，命令这间加：& 来隔开
command_parameters：为特定命令指定参数或命令行开关
IN (set)：指在(set)中取值；DO command ：指执行command
参数：/L 指用增量形式{ (set)为增量形式时 }；/F 指从文件中不断取值，直到取完为止{ (set)为文件时，如(d:pass.txt)时 }。

用法举例：

 

@echo off
echo 用法格式：test.bat *.*.* > test.txt
for /L %%G in (1 1 254) do echo %1.%%G >>test.txt & net use \%1.%%G /user:administrator | find “命令成功完成” >>test.txt
存为test.bat 说明：对指定的一个C类网段的254个IP依次试建立administrator密码为空的IPC$连接，如果成功就把该IP存在test.txt中。

/L指用增量形式（即从1-254或254-1）；
输入的IP前面三位：..*为批处理默认的 %1；
%%G 为变量(ip的最后一位）；
& 用来隔开echo 和net use 这二个命令；
| 指建立了ipc$后，在结果中用find查看是否有”命令成功完成”信息；
%1.%%G 为完整的IP地址；
(1 1 254) 指起始值，增长量，结止值。
@echo off
echo 用法格式：ok.bat ip
FOR /F %%i IN (D:user.dic) DO smb.exe %1 %%i D:pass.dic 200
存为：ok.exe 说明：输入一个IP后，用字典文件d:pass.dic来暴解d:user.dic中的用户密码，直到文件中值取完为止。

%%i为用户名；
%1为输入的IP地址（默认）。
Shutdown.exe
Shutdown IP地址 t:20
20秒后将对方NT自动关闭

 

## fpipe.exe (TCP端口重定向工具)

fpipe -l 80 -s 1029 -r 80
当有人扫锚你的80端口时，他扫到的结果会完全是的主机信息

Fpipe -l 23 -s 88 -r 23 目标IP
把本机向目标IP发送的23端口Telnet请求经端口重定向后，就通过88端口发送到目标IP的23端口。（与目标IP建立Telnet时本机就用的88端口与其相连接）然后：直接Telnet 127.0.0.1（本机IP）就连接到目标IP的23端口了。

# Linux系统

## vim/vi 文本编辑器

1）常用快捷键：(在一般模式下使用)：

a ：在光标后一位开始插入

A 在该行的最后插入

I 在该行的最前面插入

gg 直接跳到文件的首行

G 直接跳到文件的末行

dd 删除一行；3dd 删除3行；yy 复制一行；3yy 复制3行；p 粘贴；u：即undo撤销操作；

v ：进入字符选择模式，选择完成后，按y复制，按p粘贴

ctrl+v 进入块选择模式，选择完成后，按y复制，按p粘贴

shift+v 进入行选择模式，选择完成后，按y复制，按p粘贴

2）查找并替换

1 显示行号 :set nu

2 隐藏行号：:set nonu

3 查找关键字 :/you ## 效果：查找文件中出现的you，并定位到第一个找到的地方，按n可以定位到下一个匹配位置(按N定位到上一个)

4 替换操作 ：s/sad/bbb 查找光标所在行的第一个sad，替换为bbb

:%s/sad/bbb 查找文件中所有sad，替换为bbb

head 显示文件内容头部。
head -10 install.log 查看文件头部的10行

# tail 显示文件内容尾部

tail **-10** install.log 查看文件尾部的10行

tail **+10** install.log 查看文件 10–>末行

tail -f install.log 小f跟踪文件的唯一inode号，就算文件改名后，还是跟踪原来这个inode表示的文件

tail -F install.log 大F按照文件名来跟踪

## 文件压缩及解压缩命令

将许多文件一起保存至一个单独的磁带或磁盘归档，并能从归档中单独还原所需文件。

示例
tar -cf archive.tar foo bar # 从文件 foo 和 bar 创建归档文件
archive.tar。
tar -tvf archive.tar # 详细列举归档文件 archive.tar
中的所有文件。
tar -xf archive.tar # 展开归档文件 archive.tar

tar -cvf archive.tar file1 创建一个非压缩的 tarball
tar -cvf archive.tar file1 file2 dir1 创建一个包含了 ‘file1’, ‘file2’ 以及 ‘dir1’的档案文件
tar -tf archive.tar 显示一个包中的内容
tar -xvf archive.tar 释放一个包
tar -xvf archive.tar -C /tmp 将压缩包释放到 /tmp目录下
tar -cvfj archive.tar.bz2 dir1 创建一个bzip2格式的压缩包
tar -xvfj archive.tar.bz2 解压一个bzip2格式的压缩包
tar -cvfz archive.tar.gz dir1 创建一个gzip格式的压缩包
tar -xvfz archive.tar.gz 解压一个gzip格式的压缩包
解压到/usr/下：tar -zxvf a.tar.gz -C /usr
查看压缩包内容tar -ztvf a.tar.gz
c创建
v显示文件信息
f递归操作
z gzip格式
x 解压

## unzip 解压和zip压缩文件

zip file1.zip file1 创建一个zip格式的压缩包
zip -r file1.zip file1 file2 dir1 将几个文件和目录同时压缩成一个zip格式的压缩包
unzip file1.zip 解压一个zip格式压缩包

## 系统信息查询命令

hostname 显示或者设置当前系统的主机名。
dmesg 显示开机信息，用于诊断系统故障。
uptime 显示系统运行时间以及负载。
uname -m 显示机器的处理器架构(2)
cat /proc/cpuinfo 显示CPU info的信息
cat /proc/meminfo 校验内存使用
cat /proc/version 显示内核的版本
lspci -tv 罗列 PCI 设备
lsusb -tv 显示 USB 设备

top 实时显示系统资源使用情况。
free 查看系统内存
date 显示、设置系统时间
cal 查看日历、时间信息
cal 2021 显示2021年的日历表
date 041217002021.00 设置日期和时间 – 月日时分年.秒
clock -w 将时间修改保存到 BIOS

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

输入以下配置文件

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

使用容器mysql命令行工具

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

容器化安装 Redis 容器
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

Nacos配置
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

# root用户远程ssh登录

 

vim /etc/ssh/sshd_config //进入修改ssh_config文件模式
PermitRootLogin：no–>yes //PermitRootLogin字段值改成yes
systemctl restart sshd //重启sshd

chmod 4755 /bin/su //su root无法登陆root账号，提示认证失败

 

# Nmap简单的扫描方式

全面扫描：nmap -T4 -A ip

主机发现：nmap -T4 -sn ip

端口扫描：nmap -T4 ip

服务扫描：nmap -T4 -sV ip

操作系统扫描：nmap -T4 -O ip

1、SYN扫描
首先可以利用基本的SYN扫描方式探测其端口开放状态。

nmap -sS -T4 www.fujieace.com
扫描输出结果为：

All 997 ports are filtered
PORT STATE SERVICE
80/tcp open http
113/tcp closed auth
507/tcp open crs
我们可以看到SYN方式探测到3个端口开放，而有997个端口被过滤。Nmap默认扫描只扫描1000个最可能开放的端口，如果想扫描全部的端口，命令如下：

nmap -sS -T4-p- www.fujieace.com
2、FIN扫描
然后可以利用FIN扫描方式探测防火墙状态。FIN扫描方式用于识别端口是否关闭，收到RST回复说明该端口关闭，否则说明是open或filtered状态。

nmap -sF -T4 www.fujieace.com
扫描输出结果为：

PORT STATE SERVICE
7/tcp open|filtered echo
9/tcp open|filtered discard
11/tcp open|filtered systat
13/tcp open|filtered daytime
23/tcp open|filtered telnet
25/tcp open|filtered smtp
37/tcp open|filtered time
79/tcp open|filtered finger
80/tcp open|filtered http
更多端口，此处省略…….
3、ACK扫描
然后利用ACK扫描判断端口是否被过滤。针对ACK探测包，未被过滤的端口（无论打开、关闭）会回复RST包。

nmap -sA -T4 www.fujieace.com
扫描输出结果为：

Not shown: 997 unfiltered ports
PORT STATE SERVICE
135/tcp filtered msrpc
1434/tcp filtered ms-sql-m
32777/tcp filtered sometimes-rpc17
从结果可以知道997个端口是未被过滤的（unfiltered），而3个（135/1434/32777）被过滤了。所以，将ACK与FIN扫描 的结果结合分析，我们可以找到很多开放的端口。例如7号端口，FIN中得出的状态是:open或filtered，从ACK中得出的状态是 unfiltered，那么该端口只能是open的。

4、Window扫描
当然也可以利用Window扫描方式，得出一些端口信息，可以与之前扫描分析的结果相互补充。Window扫描方式只对某些TCPIP协议栈才有效。

window扫描原理与ACK类似，发送ACK包探测目标端口，对回复的RST包中的Window size进行解析。在某些TCPIP协议栈实现中，关闭的端口在RST中会将Window size设置为0；而开放的端口将Window size设置成非0的值。

nmap -sW -p- -T4 www.fujieace.com
输出结果：

PORT STATE SERVICE
7/tcp open echo
9/tcp open discard
11/tcp open systat
13/tcp open daytime
更多端口，此处省略……
在采用多种

扫描路由器操作系统
nmap -O -F -n 192.168.1.1
与通用PC扫描方式类似，使用-O选项扫描路由器的操作系统。-F用于快速扫描最可能开放的100个端口，并根据端口扫描结果进一步做OS的指纹分析方式获取出防火墙状态后，可以进一步进行应用程序与版本侦测及OS侦测。

nmap [扫描目标] [选项]
v: 增加输出的详细程度。

A: 启用OS检测、版本检测、脚本扫描和traceroute。

p:指定要扫描的端口范围，例如p 1-100。

# Wireshark使用

ip.addr == 180.101.49.11 and icmp 表示只显示ICPM协议且源主机IP或者目的主机IP为180.101.49.11的数据包。

三次握手原理
先来看一下基本的原理。
第一次握手：建立连接时，客户端发送SYN到服务器，并进入SYN_SENT状态。
第二次握手：服务器收到请求后，回送SYN+ACK信令到客户端，此时服务器进入SYN_RECV状态。
第三次握手：客户端收到SYN+ACK包，向服务器发送确认ACK包，客户端进入ESTABLISHED状态，服务器收到请求后也进入ESTABLISHED状态，完成三次握手，此时TCP连接成功，客户端与服务器开始传送数据。

第一次握手
第一次握手：建立连接时，客户端发送SYN到服务器，并进入SYN_SENT状态。

SYN ：标志位，表示请求建立连接。
Seq = 0 ：初始建立连接值为0，数据包的相对序列号从0开始，表示当前还没有发送数据。
Ack =0：初始建立连接值为0，已经收到包的数量，表示当前没有接收到数据。
WIN = 8192 来自Window size: 8192。
MSS = 1460 来自 Maximum segment size: 1460 byte ，最长报文段，TCP包所能携带的最大数据量，不包含TCP头和Option。一般为MTU值减去IPv4头部(至少20字节)和TCP头部(至少20字节)得到。
WS = 4 来自windows scale : 2 (multiply by 4)： 窗口扩张，放在TCP头之外的Option，向对方声明一个shift count，作为2的指数，再乘以TCP定义的接收窗口，得到真正的TCP窗口。

第二次握手
第二次握手：服务器收到请求后，回送SYN+ACK信令到客户端，此时服务器进入SYN_RECV状态。
Seq = 0 ：初始建立值为0，表示当前还没有发送数据
Ack = 1 : 表示当前端成功接收的数据位数，虽然客户端没有发送任何有效数据，确认号还是被加1，因为包含SYN或FIN标志位。尽管客户端没有发送任何有效数据，确认号还是被加1，这是因为接收的包中包含SYN或FIN标志位（并不会对有效数据的计数产生影响，因为含有SYN或FIN标志位的包并不携带有效数据。

第三次握手
第三次握手：客户端收到SYN+ACK包，向服务器发送确认ACK包，客户端进入ESTABLISHED状态，服务器收到请求后也进入ESTABLISHED状态，完成三次握手，此时TCP连接成功，客户端与服务器开始传送数据。
ACK ：标志位，表示已经收到记录
Seq = 1 ：表示当前已经发送1个数据
Ack = 1 : 表示当前端成功接收的数据位数，虽然客户端没有发送任何有效数据，确认号还是被加1，因为包含SYN或FIN标志位。尽管客户端没有发送任何有效数据，确认号还是被加1，这是因为接收的包中包含SYN或FIN标志位（并不会对有效数据的计数产生影响，因为含有SYN或FIN标志位的包并不携带有效数据)

为什么是三次握手
这个问题的本质是, 信道不可靠, 但是通信双发需要就某个问题达成一致. 而要解决这个问题, 无论你在消息中包含什么信息, 三次通信是理论上的最小值. 所以三次握手不是TCP本身的要求, 而是为了满足”在不可靠信道上可靠地传输信息”这一需求所导致的.

也是为了最小的代价验证会话双方的收发功能正常:

第一次握手成功：说明客户端的数据可以被服务端收到，说明客户端的发功能可用，说明服务端的收功能可用。但客户端自己不知道数据是否被接收。
第二次握手成功：说明服务端的数据可以被客户端收到，说明服务端的发功能可用，说明客户端的收功能可用。同时客户端知道自己的数据已经正确到达服务端，自己的发功能正常。但是服务端自己不知道数据是否被接收。
第三次握手成功：说明服务端知道自己的数据已经正确到达客户端端，自己的发功能正常。至此服务成功建立。

请求数据的过程
客户端和服务端建立连接后，开始传输数据。下图中首先是客户端发起一个GET请求，服务端收到请求后首先返回确认信息。待服务端处理完成后，将数据返回给客户端，客户端对收到的数据进行确认，完成一次数据交互的过程。

交互过程中，SEQ表示发送的数据，LEN表示发送的数据长度，下一次的SEQ就等于当前SEQ加上LEN。ACK表示接收的数据位。客户端和服务端分别计算自己的增长值。对应上图最后一次服务端返回数据时SEQ是2737，LEN是450。客户端对接收数据做了两次返回确认，第一次ACK是2737，表示还没有完成数据接收。第二次ACK是3187，等于服务端SEQ＋LEN（2737+450）表示已经完成了全部数据的接收。

第一次挥手：客户端 发送一个[FIN+ACK]，表示自己没有数据要发送了，想断开连接，并进入FIN_WAIT_1状态（不能再发送数据到服务端，但能够发送控制信息ACK到服务端）。
第二次挥手：服务端收到FIN后，知道不会再有数据从客户端传来，发送ACK进行确认，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），服务端进入CLOSE_WAIT状态。
第三次挥手：服务端发送FIN给对方，表示自己没有数据要发送了，服务端进入LAST_ACK状态，然后直接断开TCP会话的连接，释放相应的资源。
第四次挥手：客户端收到了服务端对FIN的ACK后，进入FIN_WAIT2状态（等待服务端完成资源释放的一系列工作：然后释放你为创建这个连接所分配的资源，并通知我你关闭了）; 客户端收到了服务端的FIN信令后，进入TIMED_WAIT状态，并发送ACK确认消息。客户端在TIMED_WAIT状态下，等待2MSL一段时间，没有数据到来的，就认为对面已经收到了自己发送的ACK并正确关闭了进入CLOSE状态，自己也断开了到服务端的TCP连接，释放所有资源。当服务端收到客户端的ACK回应后，会进入CLOSE状态，并关闭本端的会话接口，释放相应资源。TIME_WAIT状态持续2MSL(MSL是数据分节在网络中存活的最长时间）。
网络上比较主流的文章都说关闭TCP会话是四次挥手，但是实际上为了提高效率通常合并第二、三次的挥手，即三次挥手。

关闭连接需要四次握手。
客户端向服务端发送FIN为1的报文，服务端返回确认，关闭客户端与服务端通信的部分。
服务端向客户端发送FIN为1的报文，客户端返回确认，关闭服务端与客户端通信的部分。

端口最大65535，这里用十六进制，248高八位，162低八位，即真正的端口号为248*256+162=63650
因此在过滤器处对63650端口进行过滤
tcp:port == 63650

就看到列的目录的完整信息，里面有两个文件，意义不大，继续ftp过滤追踪
看到1323有一个下载的操作

可以看到下载的时候又开放了一个端口，就去追踪这个
248*256+165=63653

http && (ip.src==202.1.1.2 || ip.dst == 192.168.1.99)

黑客上传的第一个文件名称是什么
这里使用过滤器
http && (ip.src==202.1.1.2 || ip.dst == 192.168.1.99) && http.request.method==POST

黑客需要以攻克的1.99来作为跳板来访问2.88的ftp服务器，设置过滤器
ftp && ip.src == 192.168.1.99

RETR ftp 下载标识

服务器2.88中用户名为admin_zz的web后台管理员的密码
既然这里给出用户名，直接设置过滤器

ip.addr == 192.168.1.99 && http contains “admin_zz”

服务器2.88的mysql账号密码(格式：账号/密码)
这里也应该是查看黑客下载的敏感文件中的内容，设置过滤器ip.addr == 192.168.1.99 && ftp-data

服务器2.88的mysql服务中有和admin有关的三个表，请按照黑客的查询顺序作答，使用空格分隔

ip.addr == 192.168.1.99 && mysql contains “met_”

请列出黑客设置的genreal log的绝对路径(将路径复制出来，区分大小写)
mysql日志–>设置过滤器

ip.addr == 192.168.1.99 && mysql

路由器的品牌、型号、版本(请直接复制粘贴)
涉及到路由器，设置就过滤器

ip.addr == 192.168.0.1 && http

在路由器的端口监控中，监控端口和被监控端口分别是多少，例，1号端口监控2/3/4号端口:1–>2,3,4
这里提到监控端口,还是设置过滤器慢慢找

ip.addr == 192.168.0.1 && http &&contains “port”

路由器的系统路由表中一共有几条?第三条的子网掩码是多少。例: 255 255.255.0则为24 (格式:用英文逗号分隔)
ip.addr == 192.168.0.1 && http contains “gateway”

路由器的5Gwif名称是什么，信道是多少(格式:名称信道)
设置过滤器
ip.addr == 192.168.0.1 && http contains “5G”

# windows服务器只允许特定IP访问

1. 打开Windows Defender防火墙高级安全

打开“控制面板” > “系统和安全” > “Windows Defender 防火墙”。
在左侧菜单选择“高级设置”。

2. 创建允许规则

在“Windows Defender 防火墙高级安全”窗口的左侧，点击“入站规则”，然后在右侧的“操作”栏点击“新建规则”。

选择“自定义规则”，点击“下一步”。

对于“协议和端口”，根据需要保护的服务选择协议（TCP或UDP）和端口（特定端口或范围），然后点击“下一步”。

在“作用域”页面：

“本地IP地址”保持为“任何IP地址”。
在“远程IP地址”下选择“这些IP地址”，然后输入允许访问的特定IP地址或地址范围。点击“下一步”。
在“操作”页面，选择“允许连接”，点击“下一步”。

在“配置文件”页面，根据服务器的网络环境选择相应的配置文件（通常是全部勾选），点击“下一步”。

给规则命名（例如，“允许特定IP访问HTTP服务”），并提供描述（可选），点击“完成”。

3. 创建拒绝规则

为了确保除了指定的IP地址外，其他所有IP地址都无法访问，你需要创建一个拒绝规则。

重复上述步骤创建新规则，但在“操作”步骤中选择“阻止连接”。
在“远程IP地址”下选择“任何IP地址”。
确保规则名称和描述能清晰表明这是一个拒绝规则，如“拒绝所有其他IP访问HTTP服务”。

4. 确认规则顺序

确保允许规则位于拒绝规则之前，因为在处理规则时，Windows防火墙会按照从上到下的顺序应用规则，最先匹配的规则生效。

5. 测试规则

从允许的IP地址尝试访问服务器，确认可以成功连接。
从一个不在允许列表中的IP地址尝试访问，确保被拒绝。
通过上述步骤，你就成功设置了Windows服务器，只允许特定IP地址访问，而阻止了所有其他IP地址的访问。请根据实际需要调整端口、协议以及IP地址范围。

# **windows：wmic命令**

C:\Users\Administrator>wmic
wmic:root\cli>baseboard get manufacturer,product
Manufacturer Product
ASUSTeK COMPUTER INC. H510M-D3H/M.2 R2.0

wmic:root\cli>

显示进程的详细信息
输入 process where name=”chrome.exe” list full

显示出BIOS信息
wmic bios list full

停止、暂停和运行服务功能
启动服务startservice,
停止服务stopservice,
暂停服务pauseservice,
Service where caption=”windows time” call stopservice ——停止服务
Service where caption=”windows time” call startservice ——启动服务
Service where name=”w32time” call stopservice ——停止服务

注意name和caption的区别:caption 显示服务名,name：服务名称.
如： telnet服务的显示名称是telnet ,服务名称.是tlntsvr。
还有Windows Time服务的名称是w32time 显示名称是”Windows Time”要用引号引起来，主要是有一个空格。

好了具体看一下：输入Service where caption=”windows time” call startservice后有一个确认输入y就可以了，返回ReturnValue = 0;表示成功

大家可能注意到了上面命令行中还有两个参数list和full。list决定显示的信息格式与范围，它有Brief、Full、Instance、 Status、System、Writeable等多个参数，full只是它的一个参数，也是list的缺省参数，表示显示所有的信息。其他几个参数顾名思义，如Brief表示只显示摘要信息，Instance表示只显示对象实例，Status表示显示对象状态，Writeable表示只显示该对象的可写入的属性信息等。

停止进程的操作
例如，执行下面的命令将关闭正在运行的QQ.exe：
例1、wmic process where name=’QQ.exe’ call terminate
例2、wmic process where name=”qq.exe” delete

列出所有的进程 : wmic process.

连接远程电脑
连接远程的电脑，不过好象对要开一些相应的服务

wmic /node:”192.168.203.131″ /password:”” /user:”administrator”

BIOS – 基本输入/输出服务 (BIOS) 管理
查看bios版本型号
wmic bios get Manufacturer,Name

C:\Users\Administrator>wmic baseboard get manufacturer,name,product
Manufacturer Name Product
ASUSTeK COMPUTER INC. 基板 H510M-D3H/M.2 R2.0

WMIC设置IP地址
配置或更新IP地址：
wmic nicconfig where index=0 call enablestatic(“192.168.1.5”), (“255.255.255.0”) ；index=0说明是配置网络接口1。
配置网关（默认路由）：
wmic nicconfig where index=0 call setgateways(“192.168.1.1″),(1)

CPU – CPU 管理
查看cpu型号
wmic cpu get name

C:\Users\Administrator>wmic cpu get name
Name
11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz

DISKDRIVE – 物理磁盘驱动器管理

获取物理磁盘型号大小等
wmic DISKDRIVE get Caption,size,InterfaceType

C:\Users\Administrator>wmic diskdrive get caption,size,interfacetype
Caption InterfaceType Size
SSK SCSI Disk Device SCSI 512105932800
Kingston DataTraveler 3.0 USB Device USB 123724661760
Great Wall GW3300 256GB SCSI 256052966400

LOGICALDISK – 本地储存设备管理
获取硬盘系统格式、总大小、可用空间等
wmic LOGICALDISK get name,Description,filesystem,size,freespace

远程计算机的远程桌面连接
WMIC命令开启远程计算机的远程桌面连接
执行wmic /node:192.168.1.2 /USER:administrator PATH win32_terminalservicesetting WHERE (__Class!=””) CALL SetAllowTSConnections 1
具体格式：

wmic /node:”[full machine name]” /USER:”[domain]\[username]”
PATH win32_terminalservicesetting WHERE (__Class!=””) CALL SetAllowTSConnections 1

实例summary
wmic 获取进程名称以及可执行路径:
wmic process get name,executablepath

wmic 删除指定进程(根据进程名称):
wmic process where name=”qq.exe” call terminate
或者用
wmic process where name=”qq.exe” delete

wmic 删除指定进程(根据进程PID):
wmic process where pid=”123″ delete

wmic 创建新进程
wmic process call create “C:\Program Files\Tencent\QQ\QQ.exe”

在远程机器上创建新进程：
wmic /node:192.168.201.131 /user:administrator /password:123456 process call create cmd.exe

关闭本地计算机
wmic process call create shutdown.exe

重启远程计算机
wmic /node:192.168.1.10/user:administrator /password:123456 process call create “shutdown.exe -r -f -m”

更改计算机名称
wmic computersystem where “caption=’%ComputerName%'” call rename newcomputername

更改帐户名
wmic USERACCOUNT where “name=’%UserName%'” call rename newUserName

wmic 结束可疑进程（根据进程的启动路径）

wmic process where “name=’explorer.exe’ and executablepath<>’%SystemDrive%\\windows\\explorer.exe'” delete

wmic 获取物理内存
wmic memlogical get TotalPhysicalMemory|find /i /v “t”

wmic 获取文件的创建、访问、修改时间

@echo off
for /f “skip=1 tokens=1,3,5 delims=. ” %%a in (‘wmic datafile where name^=”c:\\windows\\system32\\notepad.exe” get CreationDate^,LastAccessed^,LastModified’) do (
set a=%%a
set b=%%b
set c=%%c
echo 文件: c:\windows\system32\notepad.exe
echo.
echo 创建时间: %a:~0,4% 年 %a:~4,2% 月 %a:~6,2% 日 %a:~8,2% 时 %a:~10,2% 分 %a:~12,2% 秒
echo 最后访问: %b:~0,4% 年 %b:~4,2% 月 %b:~6,2% 日 %b:~8,2% 时 %b:~10,2% 分 %b:~12,2% 秒
echo 最后修改: %c:~0,4% 年 %c:~4,2% 月 %c:~6,2% 日 %c:~8,2% 时 %c:~10,2% 分 %c:~12,2% 秒
)
echo.
pause

wmic 全盘搜索某文件并获取该文件所在目录
for /f “skip=1 tokens=1*” %i in (‘wmic datafile where “FileName=’qq’ and extension=’exe'” get drive^,path’) do (set “qPath=%i%j”&@echo %qPath:~0,-3%)

获取屏幕分辨率 wmic DESKTOPMONITOR where Status=’ok’ get ScreenHeight,ScreenWidth

wmic PageFileSet set InitialSize=”512″,MaximumSize=”512″

设置虚拟内存到E盘，并删除C盘下的页面文件,重启计算机后生效

wmic PageFileSet create name=”E:\\pagefile.sys”,InitialSize=”1024″,MaximumSize=”1024″
wmic PageFileSet where “name=’C:\\pagefile.sys'” delete

获得进程当前占用的内存和最大占用内存的大小：

wmic process where caption=’filename.exe’ get WorkingSetSize,PeakWorkingSetSize

以KB为单位显示

@echo off
for /f “skip=1 tokens=1-2 delims= ” %%a in (‘wmic process where caption^=”conime.exe” get WorkingSetSize^,PeakWorkingSetSize’) do (
set /a m=%%a/1024
set /a mm=%%b/1024
echo 进程conime.exe现在占用内存：%m%K；最高占用内存：%mm%K
)
pause

远程打开计算机远程桌面

wmic /node:%pcname% /USER:%pcaccount% PATH win32_terminalservicesetting WHERE (__Class!=””) CALL SetAllowTSConnections 1

# nmap漏洞扫描

通过下面三个强大的Nmap脚本选项, 可以将Nmap变成一个全面的漏洞扫描工具:

 

Vuln：–script vuln选项执行所有NSE脚本, 这些脚本专注于识别正在扫描目标系统是否有安全漏洞, 这是发现已知漏洞的常用选项。
Vulners：–script vulners选项使用vulners.com在线数据库，利用Nmap扫描目标以查找最新漏洞。该数据库定期更新，可能会找到其它扫描可能遗漏的信息。
Vulscan: –script vulscan选项是一个Nmap漏洞扫描脚本, 它使用多个离线漏洞数据库来测试目标系统漏洞, 它使用了NVD、CVE和OVAL漏洞数据库来扫描本地网络中的漏洞, 该模式支持本地离线和在线扫描。

Nmap 的漏洞扫描将针对目标运行该工具的默认漏洞扫描脚本。这些对于查找可利用的常见漏洞非常有用。要运行漏洞扫描，使用以下命令语法：

复制
nmap –script vuln <target>

vulners漏洞扫描能够使用在线数据库vulners.com扫描目标是否有最新的漏洞。要执行此扫描，使用以下命令语法：

复制

nmap -sV –script vulners <target>

# windows入侵排查

## 一、检查系统账号安全

攻击者面对windows系统会先从用户密码入手，首先是通过rdp服务对Administrator、Guest等默认账户的口令爆破，如果爆破没结果的话会固定密码，对用户账号进行爆破，再之后加入还是失败的话就是社工生成账号、密码字典，运气好那么就可以直接登录到管理员账号。在拿到系统权限后，权限维持则是必不可少的一步，创造一个新的管理账号方便后期登录查看就是一个不错的方法，当然为了增加隐蔽性该账号可以是影子账户。根据这几方面，检查看系统账号时可以重点关注弱口令、可疑账号、影子账户。

## 二、检查异常端口、进程

端口作为计算机内部与外部数据交互的窗口，在攻击者眼里也是作为香饽饽的存在，在入侵系统后，攻击者可以在计算机上开启专属的端口来访问被害主机或植入病毒用于挖矿等，熟悉计算机的朋友应该都知道常用的端口也就那么几个，所以通过排查可疑端口能确定主机是否存在后门、是否被植入挖矿病毒等，再根据端口的PID对可疑进程对应的程序排查，确定是否为恶意程序。

（一）排查可疑端口

检查方法1：

1、使用netstat命令查看当前网络连接，定位可疑的ESTABLISHED连接

netstat -ano
2、根据PID编号通过tasklist对进程进行定位

tasklist ｜ findstr “PID”

https://i-blog.csdnimg.cn/blog_migrate/6fb83ac09fbafdaa66a0c8e1c83e3328.png

二）排查可疑进程

检查方法1：

1、在桌面打开运行（可使用快捷键 win+R），输入 msinfo32 命令
2、依次点击 “软件环境 — 正在运行任务” 就可以查看到进程的详细信息，比如进程路径、进程ID、文件创建日期以及启动时间等。

 

## 三、检查启动项、计划任务和服务

启动项、计划任务、服务是攻击者维持权限的惯用手段。在入侵windows计算机后，攻击者可以通过修改注册表、替换粘滞键程序在系统启动时就获得权限，也能够在管理员权限下设置计划任务，因为计划任务后门分为管理员权限和普通用户权限两种。管理员权限可以设置更多的计划任务，例如重启后运行等。也可以通过meterpreter创建后门服务。

（一）排查异常启动项

检查方法1：

单击【开始】>【所有程序】>【启动】，默认情况下此目录在是一个空目录，确认是否有非业务程序在该目录下。

检查方法2：

在桌面打开运行（可使用快捷键 win+R），输入 msconfig，查看是否存在命名异常的启动项目，是则取消勾选命名异常的启动项目，并到命令中显示的路径删除文件。

检查方法3:

在桌面打开运行（可使用快捷键 win+R），输入 regedit，打开注册表，查看开机启动项是否正常，特别注意如下三个注册表项：

HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Runonce
检查右侧是否有启动异常的项目，如有请删除，并建议安装杀毒软件进行病毒查杀，清除残留病毒或木马。
（二）排查计划任务

检查方法1：

1、在桌面打开运行（可使用快捷键 win+R），输入 control 打开控制面板
2、在 系统与安全 中查看计划任务属性，便可以发现木马文件的路径。
检查方法2：

1、在桌面打开运行（可使用快捷键 win+R），输入 cmd 打开命令行窗口
2、检查计算机与网络上的其它计算机之间的会话或计划任务，如有，则确认是否为正常连接，其中计划任务在windows7及之前版本的操作系统中使用at命令进行调用，在从windows8版本开始的操作系统中使用schtasks命令调用。

windows server 2016 下执行 schschtasks

（三）排查服务自启动

检查方法：

1、在桌面打开运行（可使用快捷键 win+R），输入 services.msc
2、注意服务状态和启动类型，检查是否有异常服务

 

## 四、检查系统相关信息

系统本身如果存在漏洞，那么结果往往是致命的，如果计算机存在永恒之蓝漏洞且未采取防护措施。那么攻击者就能直接通过MSF的漏洞利用程序获取目标windows系统的system权限。与此同时，攻击者在进入系统后往往也会留一些蛛丝马迹，如未将上传文件清除、浏览器浏览记录未删除、下载的文件未删除等。在检查系统相关信息时就需要重点关注系统本身存在的漏洞以及攻击者使用过的文件。

（一）查看系统版本以及补丁信息

检查方法：

1、在桌面打开运行（可使用快捷键 win+R）输入 systeminfo
2、查看系统信息和补丁状态
（二）查看可疑目录及文件

检查方法1：

查看用户目录，是否存在新建用户目录

Window 2003版本 : C:\Documents and Settings
Window 2003以后版本 : C:\Users\

检查方法2：

1、在桌面打开运行（可使用快捷键 win+R）输入 %UserProfile%\Recent
2、分析最近打开的可疑文件

检查方法3：

1、点击文件资源管理器，查找服务器内中的各个文件夹
2、将文件夹文件按时间进行排序，查找可疑文件，其中修改时间在创建时间之前的为可疑文件，也可以在搜索中搜索某一时间修改的文件。重点关注windows\system32的sethc.exe是否被替换为cmd程序

检查方法4：

针对回收站、浏览器下载目录以及历史记录进行排查
（三）查看隐藏文件

检查方法1：

1、在桌面打开运行（可使用快捷键 win+R），输入 control，进入控制面板
2、找到文件资源管理器选项，点击 查看 后，取消”隐藏受保护的操作系统文件“勾选，在隐藏文件和文件夹下面的单选选择显示隐藏的文件、文件夹和驱动器

五、日志分析
主要查看系统日志和web日志，通过日志可以帮助我们验证对入侵过程的判断和发现其他入侵行为。但它的前提则是日志记录已开启的情况下才能获取。这块具体会在之后的日志分析篇提到

（一）系统日志

分析方法：

1、在桌面打开运行（可使用快捷键 win+R），输入 eventvwr.msc
2、找到事件查看器，查看windows日志（包括应用程序、安全、Setup、系统、事件）
（二）web日志

分析方法：

1、找到中间件、应用、WAF的日志（包括但不限于IIS、Nginx、宝塔、网站等）
2、打包至本地进行分析，在编辑器中对关键字进行搜索

**使用日志分析工具进行分析异常**

总结：检查系统账号—->检查异常端口、进程netstat -ano，tasklist ｜ findstr “PID”， msinfo32系统信息—->检查启动项、计划任务和服务,msconfig启动项，services.msc服务—->检查系统相关信息systeminfo系统信息,eventvwr.msc系统日志

[端口状态 LISTENING、ESTABLISHED、TIME_WAIT及CLOSE_WAIT详解，以及三次握手，滑动窗口_established状态-CSDN博客](https://blog.csdn.net/zzhongcy/article/details/38851271)

[终于搞懂了 TCP 的 11 种状态 ，太不容易了… – 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/news/646106)

# Linux入侵排查流程

**查看用户信息文件,是否存在新增用户**
**cat /etc/passwd**

**查看 /home目录是否有新目录(如果存在可疑目录,可能被入侵)**

 

**查看是否还有其他特权用户(默认只有一个特权用户 root 即uid=0)**

```shell
awk -F: '$3==0{print $1}' /etc/passwd
 

```

**查看当前登录用户**

```
who # tty：本地登录 pts：远程登录
```

**查看系统信息**

```
w  #可以知道某一时刻用户的行为
```

**查看的用户的多少，用户上线了多久，以及负载**

```
uptime
```

**删除用户密码**

```
passwd -d username
```

**查看用户上一次密码修改时间**

```
stat /etc/passwd    #如果近期修改过，可能存在问题
```

**查看不可登录之外的用户**

```
cat /etc/passwd | grep -v nologin  #有无新增用户，有则可能存在问题
```

**查看具有root权限的用户**

```
cat /etc/passwd | grep x:0  # 有新增的就可能存在问题
```

**查看使用shell的用户**

```
cat /etc/passwd | grep /bin/bash
```

**查看具有sudo权限的用户**

```
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"
```

**查看登录失败的次数**

```perl
grep -o "Failed password" /var/log/secure|uniq -c
```

**查看登录失败的用户**

```undefined
lastb
```

**查看登录使用的用户名 （可以看到出现的次数）**

```swift
grep "Failed password" /var/log/secure|perl -e 'while($_=<>){ /for(.*?) from/; print "$1\n";}'|uniq -c|sort -nr
 
```

**查看成功登录的ip**

```perl
grep "Accepted" /var/log/secure | awk '{print $11}' | sort | uniq -c |sort -nr | more
```

**禁用账号**

```bash
usermod -L user  #/etc/shadow 第二栏开头为！则表示以禁用
```

### 历史命令

**用户执行操作的历史记录存放在文件.bash_history，登录root可查看其他用户的历史记录信息**。

```bash
cat /home/XXX/.bash_history  #XXX是要查看的用户
```

**查看当前用户操作命令历史**

```bash
history
```

### 检查异常端口

```diff
 
netstat –antp #显示直接ip所有建立相关链接程序名的tcp的端口情况
-a：显示所有选项
-t:仅显示tcp相关选项 -u:仅显示udp相关选项 
-n：直接使用ip地址，而不通过域名服务器
-l：仅列出有在 Listen (监听) 的服務状态
-p：显示建立相关链接的程序名
-r：显示路由信息，路由表
-e：显示扩展信息，例如uid等
-s：显示网络工作信息统计表（按各个协议进行统计）
-c：每隔一个固定时间，执行该netstat命令
```

### 文件查找

**查找指定的文件（find）
查找最近一天修改的文件**

```undefined
find / -mtime -1 > /XXX.txt  #将修改过的文件保存到根目录下的XXX.txt中
```

**查找大文件**

```undefined
find / -size +10000k -print  #查找大于10000k的文件
```

日志分析
默认日志位置

var/log

定位有多少IP在爆破主机的root帐号

```perl
grep “Failed password for root” /var/log/secure | awk ‘{print $11}’ | sort | uniq -c | sort -nr | more
```

**查看日志配置情况**

```undefined
more /etc/rsyslog.conf
```

[应急响应之Windows/Linux(入侵排查篇)_主机入侵排查应急响应-CSDN博客](https://blog.csdn.net/weixin_46944519/article/details/121466490)

[应急响应之windows&linux（基础篇） – 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/637750067)


---

> 作者: <no value>  
> URL: http://localhost:1313/posts/30d96c1/  

