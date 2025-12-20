# Linux手动升级OpenSSH版本(rocky9.4环境)


<!--more-->

# Linux手动升级OpenSSH版本(rocky9.4环境)

## 1.备份ssh配置数据

```shell
mkdir -p /backup/ssh_backup/pam.d
cp /etc/pam.d/sshd /backup/ssh_backup/pam.d/
cp -r /etc/ssh /backup/ssh_backup/
```

## 2.下载telnet远程连接工具,telnet常见操作命令（以防万一）

```shell
[root@localhost ~]# yum install telnet telnet-server -y
Last metadata expiration check: 0:00:46 ago on Wed Oct 23 14:22:22 2024.
Dependencies resolved.
================================================================================
Package Architecture Version Repository Size
================================================================================
Installing:
telnet x86_64 1:0.17-85.el9 appstream 63 k
telnet-server x86_64 1:0.17-85.el9 appstream 38 k
 
Transaction Summary
================================================================================
Install 2 Packages
 
Total download size: 101 k
Installed size: 179 k
Downloading Packages:
(1/2): telnet-server-0.17-85.el9.x86_64.rpm 153 kB/s | 38 kB 00:00
(2/2): telnet-0.17-85.el9.x86_64.rpm 256 kB/s | 63 kB 00:00
--------------------------------------------------------------------------------
Total 106 kB/s | 101 kB 00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
Preparing : 1/1
Installing : telnet-1:0.17-85.el9.x86_64 1/2
Installing : telnet-server-1:0.17-85.el9.x86_64 2/2
Running scriptlet: telnet-server-1:0.17-85.el9.x86_64 2/2
Verifying : telnet-server-1:0.17-85.el9.x86_64 1/2
Verifying : telnet-1:0.17-85.el9.x86_64 2/2
 
Installed:
telnet-1:0.17-85.el9.x86_64 telnet-server-1:0.17-85.el9.x86_64
 
Complete!
[root@localhost ~]# systemctl start telnet.socket
[root@localhost ~]# systemctl enable telnet.socket
Created symlink /etc/systemd/system/sockets.target.wants/telnet.socket → /usr/li b/systemd/system/telnet.socket.
[root@localhost ~]# firewall-cmd --permanent --add-port=23/tcp
Warning: ALREADY_ENABLED: 23:tcp
success
[root@localhost ~]# firewall-cmd --reload
success

#3.升级ssh
#下载源码，搭建编译环境，编译rpm包
#阿里云ssh源链接： https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/ 我们只需要下载#XXXX.tar.gz文件就好了 openssh-9.9p1.tar.gz 搭建编译环境
#编译rpm包
yum install wget  tree openssl-devel libXt-devel gtk2-devel perl krb5-devel rpm-build rpmdevtools tar pam-devel zlib-devel gcc-c++ -y
#注：本页面操作在/root/下运行
 
 
# 生成目录结构
[root@localhost ~]# rpmdev-setuptree
# 源码及spec文件放入指定目录
mv openssh-9.9p1.tar.gz rpmbuild/SOURCES/

tar zxvf rpmbuild/SOURCES/openssh-9.9p1.tar.gz openssh-9.9p1/contrib/redhat/openssh.spec

mv openssh-9.9p1/contrib/redhat/openssh.spec rpmbuild/SPECS/
 
# 下载 ssh-askpass包
wget -c 断点续传，-P 保存到指定的目录
x11-ssh-askpass - Fedora Packages
##x11-ssh-askpass是用于OpenSSH或其他开放SSH变体的轻量级密码短语对话框。特别是，x11-ssh-askpass对于Damien Miller等人开发的OpenSSH的Unix端口非常有用，并且Damien将其包含在他的OpenSSH RPM包中。X11 -ssh-askpass仅使用常用的X11库（libX11、libXt）作为其用户界面。
#这减少了对外部库（如GNOME或Perl/Tk）的依赖。有关更多信息，请参阅README。

wget https://src.fedoraproject.org/repo/pkgs/openssh/x11-ssh-askpass-1.2.4.1.tar.gz/8f2e41f3f7eaa8543a2440454637f3c3/x11-ssh-askpass-1.2.4.1.tar.gz -c -P rpmbuild/SOURCES/
 
# 完成后目录结构
[root@localhost ~]# tree rpmbuild/
rpmbuild/
├── BUILD
├── RPMS
├── SOURCES
│ ├── openssh-9.9p1.tar.gz
│ └── x11-ssh-askpass-1.2.4.1.tar.gz
├── SPECS
│ └── openssh.spec
└── SRPMS

# 编译并生成rpm
cd ~/rpmbuild/SPECS
rpmbuild -ba openssh.spec
 
# 编译中会报错缺少imake依赖,但是yum,dnf都下载不了
“Imake是一个已弃用的源代码配置和构建系统，传统上由X11R6和以前版本中的x窗口系统提供并用于构建该系统。
在X窗口系统X11R7发行版中，X窗口系统已经切换到使用GNU自动工具作为主要构建系统，而Imake系统现在已被弃用，不应该在新的软件项目中使用。
鼓励软件开发人员将软件迁移到GNU自动工具系统。”
 
可以到这里下载（Imake Download (APK, EOPKG, PKG, RPM, TGZ, TXZ, XBPS, XZ, ZST)）imake rpm包
wget https://dl.rockylinux.org/pub/rocky/8/PowerTools/x86_64/os/Packages/i/imake-1.0.7-11.el8.x86_64.rpm

#安装imake
rpm -ivh imake-1.0.7-11.el8.x86_64.rpm

# 编译完成后的rpm包生成在rpmbuild/RPMS目录中
[root@localhost ~]# ls rpmbuild/RPMS/x86_64/
openssh-9.9p1-1.el9.x86_64.rpm openssh-clients-debuginfo-9.9p1-1.el9.x86_64.rpm
openssh-askpass-9.9p1-1.el9.x86_64.rpm openssh-debuginfo-9.9p1-1.el9.x86_64.rpm
openssh-askpass-debuginfo-9.9p1-1.el9.x86_64.rpm openssh-debugsource-9.9p1-1.el9.x86_64.rpm
openssh-askpass-gnome-9.9p1-1.el9.x86_64.rpm openssh-server-9.9p1-1.el9.x86_64.rpm
openssh-askpass-gnome-debuginfo-9.9p1-1.el9.x86_64.rpm openssh-server-debuginfo-9.9p1-1.el9.x86_64.rpm
openssh-clients-9.9p1-1.el9.x86_64.rpm
```



### 安装编译完成后的rpm包

```shell
[root@localhost x86_64]# cat /etc/os-release
NAME="Rocky Linux"
VERSION="9.4 (Blue Onyx)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="9.4"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Rocky Linux 9.4 (Blue Onyx)"
ANSI_COLOR="0;32"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:rocky:rocky:9::baseos"
HOME_URL="https://rockylinux.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
SUPPORT_END="2032-05-31"
ROCKY_SUPPORT_PRODUCT="Rocky-Linux-9"
ROCKY_SUPPORT_PRODUCT_VERSION="9.4"
REDHAT_SUPPORT_PRODUCT="Rocky Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.4"
[root@localhost x86_64]# ssh -V
OpenSSH_8.7p1, OpenSSL 3.0.7 1 Nov 2022
[root@localhost x86_64]# ll
total 6196
-rw-r--r--. 1 root root 539499 Oct 18 23:24 openssh-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 32593 Oct 18 23:24 openssh-askpass-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 43293 Oct 18 23:24 openssh-askpass-debuginfo-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 15374 Oct 18 23:24 openssh-askpass-gnome-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 27174 Oct 18 23:24 openssh-askpass-gnome-debuginfo-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 647279 Oct 18 23:24 openssh-clients-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 1571293 Oct 18 23:24 openssh-clients-debuginfo-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 1046694 Oct 18 23:24 openssh-debuginfo-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 732579 Oct 18 23:24 openssh-debugsource-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 529008 Oct 18 23:24 openssh-server-9.9p1-1.el9.x86_64.rpm
-rw-r--r--. 1 root root 1140563 Oct 18 23:24 openssh-server-debuginfo-9.9p1-1.el9.x86_64.rpm
[root@localhost x86_64]# yum localinstall openssh-*.rpm
Last metadata expiration check: 0:26:43 ago on Fri Oct 18 23:10:44 2024.
Dependencies resolved.
==============================================================================================================================================================================================================
Package Architecture Version Repository Size
==============================================================================================================================================================================================================
Installing:
openssh-askpass x86_64 9.9p1-1.el9 @commandline 32 k
openssh-askpass-debuginfo x86_64 9.9p1-1.el9 @commandline 42 k
openssh-askpass-gnome x86_64 9.9p1-1.el9 @commandline 15 k
openssh-askpass-gnome-debuginfo x86_64 9.9p1-1.el9 @commandline 27 k
openssh-clients-debuginfo x86_64 9.9p1-1.el9 @commandline 1.5 M
openssh-debuginfo x86_64 9.9p1-1.el9 @commandline 1.0 M
openssh-debugsource x86_64 9.9p1-1.el9 @commandline 715 k
openssh-server-debuginfo x86_64 9.9p1-1.el9 @commandline 1.1 M
Upgrading:
openssh x86_64 9.9p1-1.el9 @commandline 527 k
openssh-clients x86_64 9.9p1-1.el9 @commandline 632 k
openssh-server x86_64 9.9p1-1.el9 @commandline 517 k
Installing dependencies:
chkconfig x86_64 1.24-1.el9 baseos 161 k
initscripts x86_64 10.11.6-1.el9 baseos 208 k
 
Transaction Summary
==============================================================================================================================================================================================================
Install 10 Packages
Upgrade 3 Packages
 
Total size: 6.4 M
Total download size: 369 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): initscripts-10.11.6-1.el9.x86_64.rpm 484 kB/s | 208 kB 00:00
(2/2): chkconfig-1.24-1.el9.x86_64.rpm 180 kB/s | 161 kB 00:00
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total 238 kB/s | 369 kB 00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
Preparing : 1/1
Installing : openssh-debugsource-9.9p1-1.el9.x86_64 1/16
Installing : openssh-debuginfo-9.9p1-1.el9.x86_64 2/16
Installing : chkconfig-1.24-1.el9.x86_64 3/16
Installing : initscripts-10.11.6-1.el9.x86_64 4/16
Running scriptlet: initscripts-10.11.6-1.el9.x86_64 4/16
Created symlink /etc/systemd/system/sysinit.target.wants/import-state.service → /usr/lib/systemd/system/import-state.service.
Created symlink /etc/systemd/system/sysinit.target.wants/loadmodules.service → /usr/lib/systemd/system/loadmodules.service.
 
Upgrading : openssh-9.9p1-1.el9.x86_64 5/16
Installing : openssh-askpass-9.9p1-1.el9.x86_64 6/16
Installing : openssh-askpass-gnome-9.9p1-1.el9.x86_64 7/16
Upgrading : openssh-clients-9.9p1-1.el9.x86_64 8/16
Running scriptlet: openssh-server-9.9p1-1.el9.x86_64 9/16
Upgrading : openssh-server-9.9p1-1.el9.x86_64 9/16
Running scriptlet: openssh-server-9.9p1-1.el9.x86_64 9/16
Installing : openssh-askpass-debuginfo-9.9p1-1.el9.x86_64 10/16
Installing : openssh-askpass-gnome-debuginfo-9.9p1-1.el9.x86_64 11/16
Installing : openssh-clients-debuginfo-9.9p1-1.el9.x86_64 12/16
Installing : openssh-server-debuginfo-9.9p1-1.el9.x86_64 13/16
Running scriptlet: openssh-server-8.7p1-38.el9_4.4.x86_64 14/16
Cleanup : openssh-server-8.7p1-38.el9_4.4.x86_64 14/16
Running scriptlet: openssh-server-8.7p1-38.el9_4.4.x86_64 14/16
Running scriptlet: openssh-clients-8.7p1-38.el9_4.4.x86_64 15/16
Cleanup : openssh-clients-8.7p1-38.el9_4.4.x86_64 15/16
Cleanup : openssh-8.7p1-38.el9_4.4.x86_64 16/16
Running scriptlet: openssh-8.7p1-38.el9_4.4.x86_64 16/16
Job for sshd.service failed because the control process exited with error code.
See "systemctl status sshd.service" and "journalctl -xeu sshd.service" for details.
 
Verifying : chkconfig-1.24-1.el9.x86_64 1/16
Verifying : initscripts-10.11.6-1.el9.x86_64 2/16
Verifying : openssh-askpass-9.9p1-1.el9.x86_64 3/16
Verifying : openssh-askpass-debuginfo-9.9p1-1.el9.x86_64 4/16
Verifying : openssh-askpass-gnome-9.9p1-1.el9.x86_64 5/16
Verifying : openssh-askpass-gnome-debuginfo-9.9p1-1.el9.x86_64 6/16
Verifying : openssh-clients-debuginfo-9.9p1-1.el9.x86_64 7/16
Verifying : openssh-debuginfo-9.9p1-1.el9.x86_64 8/16
Verifying : openssh-debugsource-9.9p1-1.el9.x86_64 9/16
Verifying : openssh-server-debuginfo-9.9p1-1.el9.x86_64 10/16
Verifying : openssh-9.9p1-1.el9.x86_64 11/16
Verifying : openssh-8.7p1-38.el9_4.4.x86_64 12/16
Verifying : openssh-clients-9.9p1-1.el9.x86_64 13/16
Verifying : openssh-clients-8.7p1-38.el9_4.4.x86_64 14/16
Verifying : openssh-server-9.9p1-1.el9.x86_64 15/16
Verifying : openssh-server-8.7p1-38.el9_4.4.x86_64 16/16
 
Upgraded:
openssh-9.9p1-1.el9.x86_64 openssh-clients-9.9p1-1.el9.x86_64 openssh-server-9.9p1-1.el9.x86_64
Installed:
chkconfig-1.24-1.el9.x86_64 initscripts-10.11.6-1.el9.x86_64 openssh-askpass-9.9p1-1.el9.x86_64 openssh-askpass-debuginfo-9.9p1-1.el9.x86_64
openssh-askpass-gnome-9.9p1-1.el9.x86_64 openssh-askpass-gnome-debuginfo-9.9p1-1.el9.x86_64 openssh-clients-debuginfo-9.9p1-1.el9.x86_64 openssh-debuginfo-9.9p1-1.el9.x86_64
openssh-debugsource-9.9p1-1.el9.x86_64 openssh-server-debuginfo-9.9p1-1.el9.x86_64
 
Complete!
[root@localhost x86_64]# ssh -V
OpenSSH_9.9p1, OpenSSL 3.0.7 1 Nov 2022
[root@localhost x86_64]#
```

### 安装openssh

```shell
yum localinstall openssh-*.rpm
```

### 还原配置文件

```shell
cp /backup/ssh_backup/pam.d/sshd /etc/pam.d/
chmod 400 /etc/ssh/ssh_host_*
#（按需修改）
# 修改配置文件sftp内容
sed -i 's?/usr/libexec/openssh/sftp-server?/usr/libexec/openssh/sftp-server -l INFO -f AUTH?g' /etc/ssh/sshd_config
 
# 修改配置文件 UsePAM
sed -i 's?#UsePAM no?UsePAM yes?g' /etc/ssh/sshd_config
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config    
#直接用root登录终端（此处根据自身情况考虑）
#设置是否允许X11转发
echo 'X11Forwarding yes' >> /etc/ssh/sshd_config   
#是否允许密码验证 
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config  

systemctl restart sshd 
 
## !!!记得先不要关闭当前页面，另外新连接一个服务器，看是否可以正常连接，如果正常连接，说明升级成功
```

# **升级openssh使用systemctl命令无法管理服务问题解决**

[centos7源码编译升级openssh使用systemctl命令无法管理服务问题解决-云社区-华为云](https://bbs.huaweicloud.com/blogs/420442)

**准备依赖包的离线文件**

```perl
# 基于上述rpm包下载需要的依赖离线文件放在对应文件
 
yum install -y --downloadonly --downloaddir=/root/install_pckg_rpm/dpndncy_pckg openssh-9.4p1/*.rpm
```

 

参考文档：[CVE-2024-6387&Open SSH漏洞彻底解决举措（含踩坑内容）_openssh 竞争条件问题漏洞(cve-2024-6387)-CSDN博客](https://blog.csdn.net/weixin_43372364/article/details/140314046)

[解决：Job for sshd.service failed because the control process exited with error code.See “systemctI sta-CSDN博客](https://blog.csdn.net/qq_42055933/article/details/138202448)

[SELinux服务开启后导致SSH连接异常的解决办法_seliunx开启后sshd断了-CSDN博客](https://blog.csdn.net/weixin_43837883/article/details/106414689)

[解决 SELinux 导致 SSH 无法登录的问题 – 0xCAFEBABE – 博客园](https://www.cnblogs.com/0xcafebabe/p/16791345.html)

[CentOS7 通过编译RPM包升级OpenSSH-8.8 – 白日醒梦 – 博客园](https://www.cnblogs.com/Likfees/p/16963189.html)

[Linux环境下OpenSSH升级到 OpenSSH_9.8p1（内置保姆级教程并包含openssl升级过程）-CSDN博客](https://blog.csdn.net/weixin_50902636/article/details/141183849)

[记一次手动将OpenSSH从7.4升级到9.3的过程 – IT老大哥 – 博客园](https://www.cnblogs.com/jianzhan/p/ssh-update.html)

[CentOS 7 SSH协议升级至9.8p1以修复CVE-2024-6387漏洞的专业指南](https://vmlib.com/manual/centos-7-upgrade-ssh-9-8p1/#google_vignette)


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/e7e6a08/  

