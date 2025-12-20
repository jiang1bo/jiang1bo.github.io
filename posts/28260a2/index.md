# Linux终端自定义欢迎语，欢迎界面




在我们登入终端后，会弹出一段欢迎信息，上面会显示登录主机的信息。通过修改 Motd 文件，我们可以修改SSH终端欢迎信息。

<!--more-->

![](.\assets\Snipaste_2024-10-31_08-37-15-1024x533.jpg)

## 原理

**在类 Unix 系统中，/etc/motd 是一个包含“今日消息”（Message of the day）的文件。**

比起电子邮件，使用 Motd 给所有用户发送消息显得更有效率。其它操作系统中也可能有这一功能，比如 Multics 中的 Motd 信息分段。

/etc/motd 内容的会在用户成功登录后由 Unix 登录命令显示，整个过程发生在 Shell 登录之前。

较新的类 Unix 系统可以生成动态消息。

**Linux 系统通过 /etc/issue、/etc/http://issue.net、/etc/motd 以及 /run/motd.dynamic 来显示本地或者是远程登录后的提示信息**，**通常用于向用户提示系统版本、硬件信息等，/etc/issue、/etc/http://issue.net 通常在未登录终端前显示，/etc/motd、/run/motd.dynamic 则是在成功登录终端后显示。** 

此外还有 /etc/profile 与 /etc/profile.d/，在登录后会运行其中的脚本。

## 实现方式

**通过[在线生成ASCII艺术字](https://www.lddgo.net/string/text-to-ascii-art)在线网站，生成想要的提示信息，保存名为motd放入/etc/目录下，重连ssh生效。**

**通过Neofetch(查询系统信息)+lolcat（炫酷的颜色） 来实现效果**

使用教程

```shell
[root@localhost ~]# ll /etc/profile.d/
total 88
-rw-r--r--. 1 root root 196 May 16 2022 colorgrep.csh
-rw-r--r--. 1 root root 201 May 16 2022 colorgrep.sh
-rw-r--r--. 1 root root 1586 Apr 21 2024 colorls.csh
-rw-r--r--. 1 root root 1431 Apr 21 2024 colorls.sh
-rw-r--r--. 1 root root 162 Jun 15 2022 colorxzgrep.csh
-rw-r--r--. 1 root root 183 Jun 15 2022 colorxzgrep.sh
-rw-r--r--. 1 root root 216 Oct 30 2022 colorzgrep.csh
-rw-r--r--. 1 root root 220 Oct 30 2022 colorzgrep.sh
-rw-r--r--. 1 root root 80 Apr 8 2024 csh.local
-rw-r--r--. 1 root root 674 Apr 6 2024 debuginfod.csh
-rw-r--r--. 1 root root 596 Apr 6 2024 debuginfod.sh
-rw-r--r--. 1 root root 831 Apr 18 2024 flatpak.sh
-rw-r--r--. 1 root root 1107 Aug 29 2019 gawk.csh
-rw-r--r--. 1 root root 757 Aug 29 2019 gawk.sh
-rw-r--r--. 1 root root 3424 Jun 23 2020 lang.csh
-rw-r--r--. 1 root root 3187 Jun 23 2020 lang.sh
-rw-r--r--. 1 root root 500 May 31 08:18 less.csh
-rw-r--r--. 1 root root 253 May 31 08:18 less.sh
-r-x--x--x 1 root root 51 Oct 25 11:25 neofetch.sh
-rw-r--r--. 1 root root 81 Apr 8 2024 sh.local
-rw-r--r--. 1 root root 120 Oct 26 2023 which2.csh
-rw-r--r--. 1 root root 540 Oct 26 2023 which2.sh
[root@localhost ~]# cat /etc/profile.d/neofetch.sh
#/bin/sh
cat /etc/motdz | lolcat && neofetch --off
[root@localhost ~]# cat /etc/motdz
______ __ __ ______ __
/ \ / | / | / | / |
/$$$$$$ | ______ _______ _______ $$ |____ $$/ _______ $$$$$$/ _____ ____ ______ ______ _______ _$$ |_
$$ | _$$/ / \ / \ / |$$ \ / |/ \ $$ | / \/ \ / \ / \ / |/ $$ |
$$ |/ |/$$$$$$ |$$$$$$$ |/$$$$$$$/ $$$$$$$ |$$ |$$$$$$$ | $$ | $$$$$$ $$$$ |/$$$$$$ | $$$$$$ |/$$$$$$$/ $$$$$$/
$$ |$$$$ |$$ $$ |$$ | $$ |$$ \ $$ | $$ |$$ |$$ | $$ | $$ | $$ | $$ | $$ |$$ | $$ | / $$ |$$ | $$ | __
$$ \__$$ |$$$$$$$$/ $$ | $$ | $$$$$$ |$$ | $$ |$$ |$$ | $$ | _$$ |_ $$ | $$ | $$ |$$ |__$$ |/$$$$$$$ |$$ \_____ $$ |/ |
$$ $$/ $$ |$$ | $$ |/ $$/ $$ | $$ |$$ |$$ | $$ | / $$ |$$ | $$ | $$ |$$ $$/ $$ $$ |$$ | $$ $$/
$$$$$$/ $$$$$$$/ $$/ $$/ $$$$$$$/ $$/ $$/ $$/ $$/ $$/ $$$$$$/ $$/ $$/ $$/ $$$$$$$/ $$$$$$$/ $$$$$$$/ $$$$/
$$ |
$$ |
$$/
[root@localhost ~]#
```

 

[Linux系统之neofetch工具的基本使用-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2331413)

[Linux系统之neofetch工具的基本使用 – 知乎](https://zhuanlan.zhihu.com/p/690584480)

[Linux 自定义 Neofetch 的输出信息-CSDN博客](https://blog.csdn.net/m0_54656561/article/details/130657649)

[Linux 中 9 个鲜为人知的 Neofetch 使用方法 – Linux迷](https://www.linuxmi.com/linux-neofetch.html)

[技术|lolcat ：一个在 Linux 终端中输出彩虹特效的命令行工具](https://linux.cn/article-5798-1.html)


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/28260a2/  

