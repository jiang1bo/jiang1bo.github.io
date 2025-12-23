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





---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/835784b/  

