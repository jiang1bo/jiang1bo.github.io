# Windows与Linux入侵排查


windows入侵排查，Linux入侵排查流程

<!--more-->

# windows入侵排查

## 一、检查系统账号安全

攻击者面对windows系统会先从用户密码入手，首先是通过rdp服务对Administrator、Guest等默认账户的口令爆破，如果爆破没结果的话会固定密码，对用户账号进行爆破，再之后加入还是失败的话就是社工生成账号、密码字典，运气好那么就可以直接登录到管理员账号。在拿到系统权限后，权限维持则是必不可少的一步，创造一个新的管理账号方便后期登录查看就是一个不错的方法，当然为了增加隐蔽性该账号可以是影子账户。根据这几方面，检查看系统账号时可以重点关注弱口令、可疑账号、影子账户。

## 二、检查异常端口、进程

端口作为计算机内部与外部数据交互的窗口，在攻击者眼里也是作为香饽饽的存在，在入侵系统后，攻击者可以在计算机上开启专属的端口来访问被害主机或植入病毒用于挖矿等，熟悉计算机的朋友应该都知道常用的端口也就那么几个，所以通过排查可疑端口能确定主机是否存在后门、是否被植入挖矿病毒等，再根据端口的PID对可疑进程对应的程序排查，确定是否为恶意程序。

### （一）排查可疑端口

检查方法1：

1、使用netstat命令查看当前网络连接，定位可疑的ESTABLISHED连接

```
netstat -ano
```

2、根据PID编号通过tasklist对进程进行定位

```
tasklist ｜ findstr “PID”
```

https://i-blog.csdnimg.cn/blog_migrate/6fb83ac09fbafdaa66a0c8e1c83e3328.png

### （二）排查可疑进程

检查方法1：

1、在桌面打开运行（可使用快捷键 win+R），输入 msinfo32 命令
2、依次点击 “软件环境 — 正在运行任务” 就可以查看到进程的详细信息，比如进程路径、进程ID、文件创建日期以及启动时间等。

 

## 三、检查启动项、计划任务和服务

启动项、计划任务、服务是攻击者维持权限的惯用手段。在入侵windows计算机后，攻击者可以通过修改注册表、替换粘滞键程序在系统启动时就获得权限，也能够在管理员权限下设置计划任务，因为计划任务后门分为管理员权限和普通用户权限两种。管理员权限可以设置更多的计划任务，例如重启后运行等。也可以通过meterpreter创建后门服务。

### （一）排查异常启动项

检查方法1：

单击【开始】>【所有程序】>【启动】，默认情况下此目录在是一个空目录，确认是否有非业务程序在该目录下。

检查方法2：

在桌面打开运行（可使用快捷键 win+R），输入 msconfig，查看是否存在命名异常的启动项目，是则取消勾选命名异常的启动项目，并到命令中显示的路径删除文件。

检查方法3:

在桌面打开运行（可使用快捷键 win+R），输入 regedit，打开注册表，查看开机启动项是否正常，特别注意如下三个注册表项：

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Runonce
```


检查右侧是否有启动异常的项目，如有请删除，并建议安装杀毒软件进行病毒查杀，清除残留病毒或木马。

### （二）排查计划任务

检查方法1：

1、在桌面打开运行（可使用快捷键 win+R），输入 control 打开控制面板
2、在 系统与安全 中查看计划任务属性，便可以发现木马文件的路径。
检查方法2：

1、在桌面打开运行（可使用快捷键 win+R），输入 cmd 打开命令行窗口
2、检查计算机与网络上的其它计算机之间的会话或计划任务，如有，则确认是否为正常连接，其中计划任务在windows7及之前版本的操作系统中使用at命令进行调用，在从windows8版本开始的操作系统中使用schtasks命令调用。

windows server 2016 下执行 

```
schtasks
```



### （三）排查服务自启动

检查方法：

1、在桌面打开运行（可使用快捷键 win+R），输入 services.msc
2、注意服务状态和启动类型，检查是否有异常服务

 

## 四、检查系统相关信息

系统本身如果存在漏洞，那么结果往往是致命的，如果计算机存在**永恒之蓝**漏洞且未采取防护措施。那么攻击者就能直接通过MSF的漏洞利用程序获取目标windows系统的system权限。与此同时，攻击者在进入系统后往往也会留一些蛛丝马迹，如未将上传文件清除、浏览器浏览记录未删除、下载的文件未删除等。在检查系统相关信息时就需要重点关注系统本身存在的漏洞以及攻击者使用过的文件。

### （一）查看系统版本以及补丁信息

检查方法：

1、在桌面打开运行（可使用快捷键 win+R）输入 systeminfo
2、查看系统信息和补丁状态

### （二）查看可疑目录及文件

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

### （三）查看隐藏文件

检查方法1：

1、在桌面打开运行（可使用快捷键 win+R），输入 control，进入控制面板
2、找到文件资源管理器选项，点击 查看 后，取消”隐藏受保护的操作系统文件“勾选，在隐藏文件和文件夹下面的单选选择显示隐藏的文件、文件夹和驱动器

### （四）日志分析

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

总结：

**检查系统账号—->检查异常端口、进程netstat -ano，tasklist ｜ findstr “PID”， msinfo32系统信息—->检查启动项、计划任务和服务,msconfig启动项，services.msc服务—->检查系统相关信息systeminfo系统信息,eventvwr.msc系统日志**

```shell
##检查系统账号，检查异常端口、进程
netstat -ano，tasklist ｜ findstr “PID”

##系统信息—->检查启动项、计划任务和服务
msinfo32
##启动项
msconfig
##服务
services.msc
##检查系统相关信息
systeminfo
##系统日志
eventvwr.msc
```

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
> URL: http://localhost:1313/posts/51fa418/  

