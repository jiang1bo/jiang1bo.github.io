# RedmiNote7+Termux服务器改造方案


RedmiNote7+Termux服务器改造方案并部署博客

<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

## 生命的意义在于折腾

**在一个无聊的下午，翻出了早已退休的redminote7pro。秉承着老同志继续发光发热的想法，在对deekseek进行几天友好互动后，于是有了这篇文章。**



## 为手机配置 Linux环境

在安卓手机上运行完整 Linux 有几种主流方案，核心区别在于**是否需要高版本系统、Root权限以及对性能的追求**。下面这个表格能帮你快速了解并选择：

| 方案                             | 核心特点                                   | 是否需要 Root | 适合谁                                                |
| :------------------------------- | :----------------------------------------- | :------------ | :---------------------------------------------------- |
| **🔗 利用 Android 16 原生功能**   | 系统级支持，性能好，设置简单               | **否**        | 手机已升级到 **Android 16**，追求稳定和较好性能的用户 |
| **🐧 Termux + 发行版**            | 最通用、资源多，可安装完整发行版和图形桌面 | **否**        | **绝大多数用户**的首选，想灵活尝试不同Linux发行版     |
| **🖥️ LocalDesktop 等一键工具**    | 安装配置简化，侧重桌面体验                 | **否**        | 希望快速获得Linux桌面环境，不想手动敲命令的用户       |
| **⚙️ 高级方案 (如 Linux Deploy)** | 功能最完整，性能潜力高                     | **是**        | 拥有已Root设备，追求极致性能和完整控制的进阶用户      |





### 使用 Termux 安装linux环境（最通用、最灵活）



**经过几天的折腾发现还是termux方便，虽然不能运行完整的linux环境，但是能满足90%的需要**



这是目前最受欢迎的方式，无需Root，适用于绝大多数安卓手机（系统要求通常为 **Android 7.0 以上**）。

**Termux 使用 apt 和 dpkg 进行包管理，类似于 Ubuntu 或 Debian。**（网页翻译）

**Termux是一个在Android系统中内置了Linux环境的软件**


Termux官网：https://termux.dev/

Termux下载地址(GitHub)：https://github.com/termux/termux-app/releases

Termux下载地址(F-Droid)：https://f-droid.org/en/packages/com.termux/

### 局限性

Termux 不支持使用 Debian、Ubuntu 及其他 Linux 发行版仓库的软件包。因此，不要尝试将这些文件放入你的sources.list或手动安装.deb文件。请参见[“与Linux的区别](https://wiki.termux.com/wiki/Differences_from_Linux)”以了解原因。

额外限制：

- 目前只支持单一架构。你不能同时安装64位和32位包。
- root地址下的apt使用被限制，以防止Android /data分区的所有权和SELinux标签出错。
- 不支持降级。为了回收磁盘空间，我们不保存包版本的历史记录。

### 使用包管理器

强烈建议使用公用事业，而不是直接使用。它是一个包装程序，执行多种任务：`pkg``apt`

- 提供命令快捷键。用“pkg in”代替“pkg install”或“apt install”。
- 如果需要安装包，会自动运行“apt update”。
- 通过定期自动切换镜像，实现客户端仓库的负载均衡。*这对于防止我们达到托管配额上限非常重要。*安装新软件包：

```
pkg install package-name
```

强烈建议在安装新软件包前先升级现有软件包。您可以通过执行以下命令安装更新：

```
pkg upgrade
```

此外，我们建议至少每周查看一次更新。否则在安装软件包或升级过程中，存在一定风险会遇到问题。

移除已安装的软件包：

```
pkg uninstall package-name
```

这会移除包，但修改后的配置文件会保留。如果你想去除它们，就用它代替。`apt purge`

以下为更多支持的命令：

|           指挥            |              描述              |
| :-----------------------: | :----------------------------: |
|      `pkg autoclean`      | 从缓存中移除过时的 .deb 文件。 |
|        `pkg clean`        |   从缓存中移除所有.deb文件。   |
|   `pkg files <package>`   |    列出按指定包安装的文件。    |
|      `pkg list-all`       |      列出所有可用的套餐。      |
|   `pkg list-installed`    |     列出当前安装的软件包。     |
| `pkg reinstall <package>` |       重新安装指定的包。       |
|   `pkg search <query>`    |        按查询搜索包裹。        |
|   `pkg show <package>`    |      显示具体包裹的信息。      |

### 官方资料库

主 Termux 仓库可通过 https://packages.termux.org/apt/termux-main/ 访问。

我们有一些可选的仓库，提供特定主题的内容，可以通过安装名称以 . 结尾的包来启用。`-repo`

当前可选仓库列表：

|                            存储库                            |        命令订阅仓库        |
| :----------------------------------------------------------: | :------------------------: |
|      [游戏包](https://github.com/termux/game-packages)       |  `pkg install game-repo`   |
|     [科学包](https://github.com/termux/science-packages)     | `pkg install science-repo` |
| [termux-root-packages](https://github.com/termux/termux-root-packages) |  `pkg install root-repo`   |
| [x11 套件](https://github.com/termux/x11-packages)（仅限 Android 7+） |   `pkg install x11-repo`   |

我们官方仓库的软件包由位于 [github.com/termux/](https://github.com/termux) 的脚本构建，并由 [Termux 开发团队](https://github.com/orgs/termux/people)成员维护和签署。用于验证签名的公钥包含在包中。关于仓库如何签名的更多信息，请参见 [termux-keyring](https://wiki.termux.com/wiki/Termux-keyring)。`termux-keyring`

Termux apt 仓库的镜像是可用的。在 [Github](https://github.com/termux/termux-packages/wiki/Mirrors) 上查看最新的相关信息。

你可以用效用来选镜像。`termux-change-repo`

### 其他包管理器

有些编程语言有自己的包管理器。由于在构建环境中交叉编译时存在问题，我们通常不会用这些软件打包可安装内容。

- [Node.js 包管理（npm）](https://wiki.termux.com/wiki/Node.js)
- [Perl 包管理（CPAN）](https://wiki.termux.com/wiki/Perl)
- [Python 包管理（pip）](https://wiki.termux.com/wiki/Python)
- [Ruby 包管理（宝石）](https://wiki.termux.com/wiki/Ruby)
- [TeX 实时包管理（tlmgr）](https://wiki.termux.com/wiki/TeX_Live)
- [Rust 包裹管理器（货物）](https://wiki.termux.com/wiki/Development_Environments#Rust)

## Termux基础配置

### 首先，打开Termux更新pkg

```
输入 pkg update   输入 Y
```

后面就一路回车就行

### 换源

我们在使用 pkg update 的时候会发现pkg官方源比较慢，所以我们可以更换国内的其他源


例如：这里更换了清华源


输入：

```shell
#清华源
sed -i 's@^\(deb.*stable main\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/termux-packages-24 stable main@' $PREFIX/etc/apt/sources.list
sed -i 's@^\(deb.*games stable\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/game-packages-24 games stable@' $PREFIX/etc/apt/sources.list.d/game.list
sed -i 's@^\(deb.*science stable\)$@#\1\ndeb https://mirrors.tuna.tsinghua.edu.cn/termux/science-packages-24 science stable@' $PREFIX/etc/apt/sources.list.d/science.list

apt update && apt upgrade

```

**到这时候，你已经完成了linux运行环境的搭建**

## 网站环境配置（wordpress,typecho）

### 安装数据库（mariadb）

MariaDB 是由社区开发、商业支持的 MySQL 关系数据库管理系统的分支。

#### 安装包

安装包“mariadb”：

```
pkg install mariadb
```

安装应该不会出错。如果发生了错误，你可能需要在 [Github](https://github.com/termux/termux-packages/issues) 上报告问题。

上述命令还会初始化数据库，使用**两个全权限账户**（可能在MariaDB 10.4.x中引入）。第一个是“root”，无法访问，第二个是你的 Termux 用户名（用命令或 Check）。`id -un``whoami`

要启用root账户访问权限，你需要用你的Termux用户名登录

```
mysql -u $(whoami)
```

并手动更改root密码

```
use mysql;
set password for 'root'@'localhost' = password('YOUR_ROOT_PASSWORD_HERE');
flush privileges;
quit;
```

确认你能用 Root登录。你需要提供前一步设置的密码。` mysql -u root -p`

#### 使用 package

每当你想通过命令行或某个程序（网页应用）手动访问MySQL数据库时，你需要启动MySQL服务器：

```
mysqld_safe
```

然后你应该能够连接到数据库，例如用 。`mysql -u root -p`



```shell


#装一个unzip
pkg install unzip

#装一个mariadb
pkg install mariadb

#启动数据库
nohup mysqld &

```

### 安装配置nginx

```shell
#安装
pkg install nginx

#检查配置文件是否正常
nginx -t

#启动
nginx

#Termux 在 Nginx 上默认运行的端口号是 8080
#可以使用pgrep查看nginx的进程pid号:
pgrep nginx

#直接打开浏览器访问：
http://127.0.0.1:8080

#重启nginx
nginx -s reload

#停止nginx
#以nginx提供的原生方法：

nginx -s stop #直接停止

nginx -s quit #完成已经接受的请求，然后退出。

#杀死进程
kill -9 `pgrep nginx`

# 查询 nginx 进程相关的 PID 号
pgrep nginx

# 杀掉 查询出的 PID号进程
kill -9 PID

```

### 配置nginx

```shell
#配置nginx
vim $PREFIX/etc/nginx/nginx.conf



```

1.添加 index.php 到默认首页的规则里面

![image-20260122034903645](.\assets\image-20260122034903645.png)

2.取消 location ~ \.php$ 这些注释，改成图片上面的样子：

![image-20260122035000525](.\assets\image-20260122035000525.png)

Termux 里面的 Nginx 默认网站的根目为：`/data/data/com.termux/files/usr/share/nginx/html`

如果想要修改默认路径的话 只需要在上图配置文件中 替换2处出现的这个路径即可。

### 安装php-fpm

```shell
pkg install php

pkg install php-fpm

#编辑 php-fpm 的配置文件www.conf
vim $PREFIX/etc/php-fpm.d/www.conf
#定位搜索 listen = 找到
listen = /data/data/com.termux/files/usr/var/run/php-fpm.sock
#改为：
listen = 127.0.0.1:9000

#测试php解析
#需要先完成nginx和php-fpm的安装和配置
#在这个网站根目录下：
/data/data/com.termux/files/usr/share/nginx/html

#新建 info.php 内容为：
<?php phpinfo(); ?>

```

**先启动php-fpm,然后启动nginx,如果你的 Nginx 已经启动了的话，使用 nginx -s reload 重启 Nginx.**

```
http://[ip地址]:8080/info.php
```

### **正式开始搭建wordpress**

```
#新建数据库

mysql -uroot -p[密码] -e"create database wordpress;show databases;"

#  wget 下载
wget https://cn.wordpress.org/wordpress-5.4-zh_CN.zip

# unzip 解压 没有安装unzip请自行安装
unzip wordpress-4.9.4-zh_CN.zip

# 将解压的文件夹移动到 nginx 网站根目录下
mv wordpress/ $PREFIX/share/nginx/html

```

### 安装wordpress

浏览器访问: http://127/.0.0.1/wordpress/进行 WordPress 的安装





**一个开机自启动脚本（检测服务状态是否启动，如果没有启动则重新启动）**

```shell
~ $ cat .bashrc
# ===== 服务自启动与状态检查脚本 (增强版) =====
# 等待系统初始化完成
sleep 6
echo "=== [$(date '+%Y-%m-%d %H:%M:%S')] RedmiNote 7 Pro已启动，正在检测wordpress与halo相关服务 ==="

# 1. 启动 SSH 服务 (端口 8022)
if pgrep -f "sshd.*8022" > /dev/null; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] SSH 服务已在运行。"
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] 启动 SSH 服务..."
    sshd -p 8022 > /dev/null 2>&1 &
fi

# 2. 启动 MariaDB 数据库
if pgrep mariadbd > /dev/null; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] MariaDB 已在运行。"
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] 启动 MariaDB..."
    nohup mysqld_safe --datadir=/data/data/com.termux/files/home/mysql_data --socket=/data/data/com.termux/files/usr/tmp/mysql.sock > /de                                                                     v/null 2>&1 &
    sleep 3 # 给数据库更长的启动时间
fi

# 3. 启动 PHP-FPM (检查master进程)
if pgrep -f "php-fpm: master" > /dev/null; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] PHP-FPM 已在运行。"
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] 启动 PHP-FPM..."
    php-fpm > /dev/null 2>&1 &
    sleep 3
fi

# 4. 启动 Halo 博客系统
if pgrep -f "halo.jar" > /dev/null; then
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Halo 已在运行。"
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] 启动 Halo..."
    cd ~/halo && nohup java -Dfile.encoding=UTF-8 -jar halo.jar --spring.config.additional-location=optional:file:"$HOME"/.halo2/ > halo.                                                                     log 2>&1 &
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Halo 启动命令已执行，日志输出至 ~/halo/halo.log"
    sleep 5 # 给Java应用充分的初始化时间
fi

# 5. 启动 Nginx (修正版 - 使用模糊匹配)
if pgrep "nginx" > /dev/null; then
    NGINX_PID=$(pgrep "nginx" | head -1)
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Nginx 已在运行 (主进程 PID: $NGINX_PID)。"
else
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Nginx 未运行，正在启动..."
    nginx 2>/dev/null

    # 等待并检查启动结果
    sleep 3
    if pgrep "nginx" > /dev/null; then
        NGINX_PID=$(pgrep "nginx" | head -1)
        NGINX_COUNT=$(pgrep "nginx" | wc -l)
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] Nginx 启动成功 (主进程 PID: $NGINX_PID，共 $NGINX_COUNT 个进程)。"
    else
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] Nginx 启动失败，请检查配置或端口冲突。"
    fi
fi

# 6. 最终状态检查
echo
echo "=== [$(date '+%Y-%m-%d %H:%M:%S')] 服务运行状态检查 ==="
# 检查SSH
if pgrep -f "sshd.*8022" > /dev/null; then
    echo "[ ✓ ] SSH (端口8022) 正在运行。"
else
    echo "[ ✗ ] SSH 未运行！"
fi
# 检查MariaDB
if pgrep "mariadbd" > /dev/null; then
    echo "[ ✓ ] MariaDB 正在运行。"
else
    echo "[ ✗ ] MariaDB 未运行！"
fi
# 检查PHP-FPM
if pgrep -f "php-fpm: master" > /dev/null; then
    echo "[ ✓ ] PHP-FPM 正在运行。"
else
    echo "[ ✗ ] PHP-FPM 未运行！"
fi
# 检查Halo
if pgrep -f "halo.jar" > /dev/null; then
    HALO_PID=$(pgrep -f "halo.jar")
    echo "[ ✓ ] Halo (PID: $HALO_PID) 正在运行。"
else
    echo "[ ✗ ] Halo 未运行！"
fi
# 检查Nginx
if pgrep "nginx" > /dev/null; then
    NGINX_PID=$(pgrep  "nginx")
    echo "[ ✓ ] Nginx (PID: $NGINX_PID) 正在运行。"
else
    echo "[ ✗ ] Nginx 未运行！"
fi
echo "====================================="
# ===== 脚本结束 =====
~ $
~ $

```



---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/118444f/  

