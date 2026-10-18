# Debian/Ubuntu 与 Fedora/RHEL 双发行版开箱指南


需要在 Debian 系（Debian / Ubuntu / Linux Mint / Pop!_OS）与 Red Hat 系（Fedora / RHEL / CentOS Stream / Rocky / AlmaLinux）之间切换或并行运维的开发者与系统管理员。

<!--more-->

# Linux 双发行版家族运维与实战速查

> 面向 Debian/Ubuntu/Kali 与 Fedora/RHEL/Rocky/AlmaLinux 用户。目标是先识别发行版，再按“任务”查两边的等价操作；不要把某一家族的包管理器、防火墙或安全策略命令直接套到另一家族。
>
> 命令默认以 Bash 为例。`<占位符>` 需替换成实际值；`sudo` 表示以管理员权限执行。涉及网络、防火墙、升级、删除、权限递归修改的操作，先确认目标并准备控制台/备份。
>
> **风险标识**：`[只读]` 只查询信息；`[会修改]` 会改变配置、服务或文件；`[高风险]` 可能造成数据删除、远程失联、权限扩大或系统不可启动。未标识的查询命令默认为只读；编辑/执行类命令按注释和上下文判断。高风险操作统一按“记录当前状态 → 核对目标并预览 → 备份配置/数据 → 小范围执行 → 验证结果 → 保留回滚或控制台访问”处理。防火墙变更应先确认 SSH 放行规则，递归权限变更先列出目标文件，覆盖/删除先使用 dry-run 或清单确认，服务变更先检查配置语法并保留现有连接。



**高频家族差异**

| 任务         | Debian / Ubuntu                 | Fedora / RHEL                 | 先确认什么                            |
| ------------ | ------------------------------- | ----------------------------- | ------------------------------------- |
| 包管理       | `apt` / `.deb`                  | `dnf` / `.rpm`                | `/etc/os-release`、仓库是否启用       |
| 管理员组     | `sudo`                          | `wheel`                       | `id <用户>`、`sudo -l`                |
| SSH 服务名   | `ssh`                           | `sshd`                        | `systemctl status` 中的实际单元名     |
| 防火墙       | 常见 `ufw`                      | 常见 `firewalld`              | 选定唯一管理入口，查看现有规则        |
| 时间同步     | `systemd-timesyncd` 或 `chrony` | 常见 `chronyd`（chrony）      | 不要同时运行多个校时服务              |
| MAC 安全     | 常见 AppArmor                   | 常见 SELinux                  | 查拒绝日志，定点修复策略/标签         |
| 传统系统日志 | `/var/log/syslog`、`auth.log`   | `/var/log/messages`、`secure` | 首选 `journalctl`；文件日志可能未启用 |

<a id="sec-identify"></a>

## 1. 先识别发行版和版本

`/etc/os-release` 是两大家族通用的标准入口，优先使用；不要只根据桌面、主机名或某条命令是否存在来判断。

```bash
cat /etc/os-release                  # 显示发行版 ID、版本号和家族标识
uname -r                             # 显示当前正在运行的 Linux 内核版本
uname -m                             # 显示 CPU 架构，如 x86_64 或 aarch64
```

| 常见发行版                                    | 家族       | 包格式 / 管理工具     | 常见差异                         |
| --------------------------------------------- | ---------- | --------------------- | -------------------------------- |
| Debian、Ubuntu、Linux Mint、Pop!_OS、Kali     | Debian 系  | `.deb`；`apt`、`dpkg` | Ubuntu 常用 UFW；AppArmor 较常见 |
| Fedora、RHEL、CentOS Stream、Rocky、AlmaLinux | Red Hat 系 | `.rpm`；`dnf`、`rpm`  | firewalld、SELinux 较常见        |

**家族定位**：Debian/Ubuntu 操作通常高度兼容；Fedora 是 RHEL 上游，软件更新较快，RHEL 兼容发行版更强调稳定与长期支持。两边普遍使用 systemd、`ip`、`ss`、`journalctl`、`ssh` 等通用工具。

<a id="sec-equivalents"></a>

## 2. 命令不互通时：按任务查等价命令

这是跨发行版时优先查阅的表。若两侧行为略有差异，以“说明”为准，不要只做字符串替换。

| 任务                      | Debian / Ubuntu                                     | Fedora / RHEL                                    | 说明                                                         |
| ------------------------- | --------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 刷新包索引 / 检查更新     | `sudo apt update`                                   | `sudo dnf check-update`                          | APT 明确刷新索引；DNF 通常在执行安装/升级时自动刷新元数据    |
| 升级已安装软件            | `sudo apt upgrade`                                  | `sudo dnf upgrade`                               | DNF 的 `update` 是 `upgrade` 的兼容别名                      |
| 允许升级时移除冲突包      | `sudo apt full-upgrade`                             | `sudo dnf distro-sync`                           | 不完全等价；先阅读将安装、升级、降级或删除的包清单           |
| 安装 / 卸载               | `sudo apt install <包>` / `sudo apt remove <包>`    | `sudo dnf install <包>` / `sudo dnf remove <包>` | 优先使用高层包管理器以自动处理依赖                           |
| 卸载并清除配置            | `sudo apt purge <包>`                               | `sudo dnf remove <包>`，必要时手动检查 `/etc`    | RPM 通常卸载包自有配置文件；用户修改过的配置可能保留         |
| 搜索 / 查看包详情         | `apt search <词>` / `apt show <包>`                 | `dnf search <词>` / `dnf info <包>`              | 搜索结果受已启用仓库影响                                     |
| 列出已安装包              | `dpkg -l` / `apt list --installed`                  | `rpm -qa` / `dnf list installed`                 | RPM 查询使用包名而非 `.rpm` 文件路径                         |
| 查询文件属于哪个包        | `dpkg -S <路径>`                                    | `rpm -qf <路径>`                                 | 查询本机已安装包所拥有的文件                                 |
| 查询包安装了哪些文件      | `dpkg -L <包>`                                      | `rpm -ql <包>`                                   | 查看已安装包的文件清单                                       |
| 安装本地软件包并解依赖    | `sudo apt install ./<文件>.deb`                     | `sudo dnf install ./<文件>.rpm`                  | 本地路径建议带 `./`；不要优先直接用 `dpkg -i` / `rpm -i`     |
| 修复本地包依赖            | `sudo apt --fix-broken install`                     | `sudo dnf distro-sync`（先检查变更）             | RPM 依赖问题可先运行原安装命令查看 DNF 报错                  |
| 清孤立依赖 / 清下载缓存   | `sudo apt autoremove` / `sudo apt clean`            | `sudo dnf autoremove` / `sudo dnf clean all`     | 清理前确认不会移除仍需要的软件                               |
| 查仓库配置                | `/etc/apt/sources.list.d/`、`/etc/apt/sources.list` | `/etc/yum.repos.d/*.repo`                        | Ubuntu 24.04 等系统可能主要使用 DEB822 `.sources` 文件       |
| 设置管理员组              | `sudo usermod -aG sudo <用户>`                      | `sudo usermod -aG wheel <用户>`                  | 修改组后需重新登录；`-aG` 的 `-a` 不可漏                     |
| 用户/组、sudo、权限和属主 | `useradd`、`usermod`、`visudo`、`chmod`、`chown`    | 同左；管理员组常用 `wheel`                       | 命令大多通用；完整用户、sudo、ACL 和权限操作集中在 3.3       |
| 时间同步                  | `timedatectl`、`systemd-timesyncd` 或 `chrony`      | `timedatectl`、`chronyd`（chrony）               | 同一时间只启用一个校时服务；配置和排障见 3.2                 |
| 重启 SSH 服务             | `sudo systemctl restart ssh`                        | `sudo systemctl restart sshd`                    | 个别系统提供别名，脚本中用本机真实单元名                     |
| 查看传统系统日志          | `/var/log/syslog`、`/var/log/auth.log`              | `/var/log/messages`、`/var/log/secure`           | 并非所有安装都启用传统日志文件，优先试 `journalctl`          |
| 防火墙管理                | `ufw` 或 `nft`                                      | `firewall-cmd` 或 `nft`                          | 选定一个主管理工具；不要让多个工具互相覆盖规则               |
| 强制访问控制排障          | `aa-status`、`aa-complain`                          | `getenforce`、`ausearch`、`semanage`             | 不要为图省事永久关闭 AppArmor/SELinux                        |
| 静态 IP（NetworkManager） | `nmcli`                                             | `nmcli`                                          | `nmcli` 两边通用；Ubuntu 也可能由 Netplan 管理配置           |
| 编译工具链                | `sudo apt install build-essential`                  | `sudo dnf group install "Development Tools"`     | 组名和组安装语法在版本间可能变化；先用 `dnf group list` 确认 |
| Java 版本切换             | `sudo update-alternatives --config java`            | `sudo alternatives --config java`                | `alternatives` 也可能提供 `update-alternatives` 兼容命令     |
| Fedora 大版本升级         | `sudo do-release-upgrade`（Ubuntu）                 | Fedora 用 `dnf system-upgrade`；RHEL 用 `leapp`  | 发行版升级方法不是跨家族通用命令                             |

<a id="sec-setup"></a>

## 3. 首次开箱配置

### 3.1 系统更新与安装软件

```bash
sudo apt update                      # 更新 Debian 系软件包索引，不会升级已安装软件
sudo apt upgrade                     # 升级可在不移除其他包的情况下更新的软件
sudo apt full-upgrade                # 处理依赖变化并允许移除冲突包；执行前检查变更计划
sudo apt autoremove                  # 移除 APT 判断为不再需要的自动依赖
sudo apt clean                       # 清除已下载的 APT 软件包缓存

sudo dnf check-update                # 检查 DNF 仓库中的可用更新；返回码 100 可能表示有更新
sudo dnf upgrade                     # 升级已安装软件，通常自动刷新仓库元数据
sudo dnf distro-sync                 # 使已安装包与当前启用仓库版本对齐，可能升级、降级或移除
sudo dnf autoremove                  # 移除 DNF 判断为不再需要的依赖
sudo dnf clean all                   # 清除 DNF 元数据和软件包缓存

sudo apt install <包名>              # 从已配置的仓库安装 Debian 系软件
sudo dnf install <包名>              # 从已启用的仓库安装 Red Hat 系软件
apt search <关键词>                  # 在 Debian 系仓库中搜索软件包
dnf search <关键词>                  # 在 Red Hat 系仓库中搜索软件包
apt show <包名>                      # 显示 Debian 系包的版本、依赖和描述
dnf info <包名>                      # 显示 Red Hat 系包的信息
```

APT 的推荐日常更新次序是 `apt update` 再 `apt upgrade`。RHEL 若未注册或仓库未配置，DNF 可能找不到 BaseOS/AppStream 软件源；先解决订阅和仓库配置，不能靠换一个包名解决。

### 3.2 主机名、时区和时间同步

```bash
sudo hostnamectl set-hostname <主机名>  # 设置系统主机名
sudo timedatectl set-timezone Asia/Shanghai # 设置系统时区为上海；只改显示时区，不改变 UTC 时间点
sudo timedatectl set-ntp true           # 请求 systemd 启用系统配置的网络时间同步服务
timedatectl status                      # 查看时区、系统时钟、RTC 和 NTP 同步状态
timedatectl timesync-status             # 查看 systemd-timesyncd 的同步状态；服务未使用时可能不可用
date                                    # 显示本地时间及当前时区
date -u                                 # 以 UTC 时区显示当前系统时间
sudo hwclock --show                     # 读取并显示硬件实时时钟（RTC）；不修改时钟
```

#### 时间同步工具选择

Linux 系统时钟通常由一个网络时间服务校准。`systemd-timesyncd` 是轻量级 SNTP 客户端，适合常见客户端/单机；`chrony`（服务端进程名 `chronyd`）支持较复杂网络、虚拟机和服务器场景，在 Fedora/RHEL 中常用。不要同时启用多个服务去校准系统时钟；先查清楚当前使用哪个服务。

| 发行版家族      | 常见工具                                   | 包名                           | 服务单元                       |
| --------------- | ------------------------------------------ | ------------------------------ | ------------------------------ |
| Debian / Ubuntu | systemd-timesyncd（若发行版提供）或 chrony | `systemd-timesyncd` / `chrony` | `systemd-timesyncd` / `chrony` |
| Fedora / RHEL   | chrony                                     | `chrony`                       | `chronyd`                      |

```bash
systemctl status systemd-timesyncd # 查看轻量级 systemd 时间同步服务状态
systemctl status chrony            # Debian 系常见 chrony 服务单元
systemctl status chronyd           # Fedora/RHEL 常见 chrony 服务单元
```

#### 使用 chrony 同步时间

```bash
sudo apt install chrony             # Debian/Ubuntu：从已配置仓库安装 chrony
sudo dnf install chrony             # Fedora/RHEL：从已启用仓库安装 chrony
sudo systemctl enable --now chrony  # Debian 系：设置 chrony 开机启动并立即启动
sudo systemctl enable --now chronyd # Fedora/RHEL：设置 chronyd 开机启动并立即启动
chronyc tracking                    # 查看 chronyd 的时钟偏差、同步状态和参考源
chronyc sources -v                  # -v 详细显示 NTP 源、可达性、偏差和选源状态
sudo journalctl -u chrony -b --no-pager -n 50
# Debian 系：查看本次启动 chrony 最近 50 条日志；-u 过滤单元，-b 限定本次启动
# --no-pager 禁用分页器，-n 50 限制记录条数
sudo journalctl -u chronyd -b --no-pager -n 50
# Fedora/RHEL：查看 chronyd 服务日志，参数含义同上
```

`chrony` 配置文件常见于 Debian/Ubuntu 的 `/etc/chrony/chrony.conf` 和 Fedora/RHEL 的 `/etc/chrony.conf`。可在其中配置受信任的 `server` 或 `pool` 时间源；修改后按对应单元名重启服务。网络防火墙需允许到时间源的 UDP 123 出站通信；若主机作为 NTP 服务器对外提供服务，还需按网络策略配置入站规则。

`systemd-timesyncd` 常见配置文件为 `/etc/systemd/timesyncd.conf`，可在 `[Time]` 段配置 `NTP=` 和 `FallbackNTP=` 时间源，保存后重启服务。传统 `ntpd`/`ntpdate` 在旧系统或特定环境中仍可能遇到，但新部署通常优先选 `chrony` 或 `systemd-timesyncd`；不要让多个守护进程同时校准同一个系统时钟。

```bash
sudo systemctl restart systemd-timesyncd # 重启 systemd-timesyncd 以读取配置更改
timedatectl show-timesync --all          # 显示 systemd-timesyncd 的详细属性；--all 包含更多可用字段
sudo systemctl restart chrony     # Debian 系：重启 chrony 以加载配置更改
sudo systemctl restart chronyd    # Fedora/RHEL：重启 chronyd 以加载配置更改
sudo chronyc makestep             # 立即阶跃修正较大时间偏差；可能影响依赖单调时间/墙上时钟的应用
sudo hwclock --systohc            # 用系统时钟写入硬件 RTC；通常由系统服务自动维护，手动前确认 RTC 模式
sudo timedatectl set-local-rtc 0  # 将硬件 RTC 配置为 UTC；双启动 Windows 环境先确认双方 RTC 策略
```

`chronyc makestep` 会让系统墙上时钟跳变，可能影响数据库、日志排序、定时任务和分布式服务；仅在确认偏差并评估影响后使用。通常不需要手动用 `date -s` 改时间。

#### 时间不准或未同步的排查

1. 先区分“时区显示错误”和“系统时钟本身偏差”：若 UTC 正确而本地时间差固定，修正时区，不要手动拨动时钟。
2. 检查同步服务是否运行、当前是否已同步以及时间源是否可达；检查 DNS 和 UDP 123 出站连通性。
3. 查看对应服务日志、配置文件的时间源和系统时间；修复错误服务、DNS、防火墙或时间源后再观察同步状态。
4. 虚拟机、容器和双启动系统还需检查宿主机时钟、容器权限以及硬件 RTC 使用 UTC/本地时间的策略。

```bash
timedatectl status                     # 查看时区、系统时间、RTC 和网络时间同步总状态
timedatectl timesync-status            # 查看 systemd-timesyncd 当前时间源及最近同步信息
chronyc tracking                       # 查看 chrony 时钟偏差、层级和是否已同步
chronyc sources -v                     # 查看 chrony 时间源、可达性及选源状态；-v 显示详细列
systemctl --failed                     # 列出启动失败的单元，检查时间同步服务是否失败
sudo timedatectl set-ntp true          # 请求 systemd 启用系统配置的网络同步服务
sudo systemctl restart systemd-timesyncd # 重启轻量级时间同步服务以重新连接时间源
sudo systemctl restart chrony          # Debian 系：重启 chrony 时间同步服务
sudo systemctl restart chronyd         # Fedora/RHEL：重启 chronyd 时间同步服务
sudo journalctl -u chronyd -b --no-pager -n 50
# Fedora/RHEL：查看本次启动 chronyd 最近 50 条日志；-u 单元、-b 本次启动、--no-pager 禁止分页、-n 限制条数
```

只运行与你选定的校时服务相对应的检查和重启命令。若时间源 DNS 无法解析，先修复 DNS；若服务运行但没有可选时间源，检查配置中的 `server`/`pool`、主机路由和出站 UDP 123 策略。

### 3.3 用户、组、sudo、权限与属主

账户、授权和文件访问控制往往要联合排查，因此集中在本节。先确认文件属主、所属组、权限位和 ACL，再按最小权限原则修复；不要用 `chmod 777` 作为通用解决方法。

#### 用户和组管理

```bash
sudo useradd -m -s /bin/bash <用户>        # 创建用户；-m 创建家目录；-s 指定登录 shell
sudo passwd <用户>                         # 设置或修改该用户密码
sudo usermod -aG sudo <用户>               # Debian 系：把用户追加到 sudo 组
sudo usermod -aG wheel <用户>              # Red Hat 系：把用户追加到 wheel 管理组
sudo usermod -aG <组1>,<组2> <用户>        # 将用户追加到多个附加组；-a 避免覆盖现有附加组
sudo usermod -G <组1>,<组2> <用户>         # 用指定附加组列表替换现有列表；高风险，可能移除 sudo/wheel 权限
sudo userdel -r <用户>                     # 删除用户；-r 同时删除家目录和邮件目录，操作不可逆
sudo usermod -s /bin/zsh <用户>            # 修改用户默认 shell
sudo usermod -l <新用户名> <旧用户名>      # 修改用户名；家目录名不会自动保证同步修改
sudo usermod -d /home/<新路径> -m <用户>   # 修改家目录路径；-d 指定路径，-m 将旧家目录内容迁移过去
sudo usermod -L <用户>                     # 锁定用户密码登录；不一定会终止已有会话或禁用 SSH 密钥登录
sudo usermod -U <用户>                     # 解锁用户密码；仅在账户状态确认安全后执行
sudo groupadd <组名>                        # 创建组
sudo groupdel <组名>                        # 删除组；先确认没有重要用户或文件依赖该组
sudo gpasswd -a <用户> <组名>              # 将用户追加到组；重新登录后新组通常才进入会话
sudo gpasswd -d <用户> <组名>              # 将用户从指定附加组移除
id <用户>                                   # 查看 UID、主 GID 和附加组
groups <用户>                               # 查看用户所属组
getent passwd <用户>                        # 从系统账户数据库（含 NSS/LDAP 配置）查询用户记录
getent group <组名>                         # 从系统组数据库查询组记录和成员
sudo chage -l <用户>                       # 查看密码过期、修改期限和账户有效期
sudo chage -M <天数> <用户>                # 设置密码最大有效天数；-M 后要求定期更新密码
sudo chage -d 0 <用户>                     # 将上次密码修改日期设为 0，要求用户下次登录时修改密码
sudo chage -E <YYYY-MM-DD> <用户>          # 设置账户过期日期；-E 后指定日期，需确认不会意外锁定账户
sudo passwd -S <用户>                      # 查看用户密码状态；不同发行版的状态标记格式略有差异
who                                       # 查看当前登录用户及其终端和登录时间
w                                         # 查看当前登录用户和正在运行的进程概况
last -n 10                                # 查看最近 10 条登录记录；记录受 wtmp 日志可用性影响
```

**高危错误**：`usermod -G` 不带 `-a` 会替换附加组列表，可能移除 `sudo`/`wheel` 等管理权限。更改组后通常要重新登录才生效。不要把普通用户直接加入 `docker` 组，Docker 守护进程权限通常等同 root。

#### sudo 授权和安全编辑

Debian 系通常把管理员授权给 `sudo` 组，Fedora/RHEL 通常使用 `wheel` 组；具体仍以 `/etc/sudoers` 和 `/etc/sudoers.d/` 的策略为准。始终使用 `visudo` 校验语法，不要直接用普通编辑器覆盖 sudoers 配置，否则语法错误可能导致无法提权。

```bash
sudo -v                                  # 验证当前用户 sudo 凭据并刷新凭据有效期
sudo -l                                  # 列出当前用户可执行的 sudo 命令及限制
sudo -u <用户> <命令>                    # 以指定目标用户身份执行单条命令
sudo -i                                  # 启动 root 的登录 shell；退出后返回原用户
sudo visudo                              # 安全编辑主 sudoers 文件并在保存时检查语法
sudo visudo -f /etc/sudoers.d/<文件名>   # 安全创建/编辑独立授权文件并检查语法
```

sudoers 授权应按需限定用户/组、主机、命令和参数。`NOPASSWD` 会免除对应命令的密码确认，不要为了省事授权任意命令。添加组授权前先核实目标发行版默认策略。

#### 权限、属主、ACL 和默认权限

传统权限分为属主（user）、属组（group）、其他用户（other），各有读 `r`、写 `w`、执行/目录遍历 `x` 权限。数字权限中 `4=r`、`2=w`、`1=x`，例如 `750` 表示属主 `rwx`、属组 `r-x`、其他无权限。目录的 `x` 表示可遍历，不代表文件内容可执行。

```bash
ls -l <路径>                            # 查看权限位、属主、属组、大小和修改时间
stat <路径>                              # 查看详细元数据；可确认 UID、GID、权限和时间
namei -l <路径>                          # 逐级显示路径目录权限，定位缺少目录遍历权限的父目录
chmod 755 <文件>                         # 设置属主 rwx、组和其他 r-x
chmod 640 <文件>                         # 设置属主 rw-、组 r--、其他无权限
chmod u+x <脚本>                         # 给属主增加执行权限；u 指属主，+x 指增加执行位
chmod g-w <文件>                         # 移除所属组的写权限；g 指组，-w 指移除写位
chmod -R <模式> <目录>                   # [高风险] 递归修改目录树权限；先检查文件/目录目标清单，不能盲目套同一权限
chmod u+s <程序>                         # 设置 SUID 位；执行时有效 UID 可能采用文件属主
chmod g+s <目录>                         # 设置 SGID 位；目录内新建项通常继承所属组
chmod +t <目录>                          # 设置 sticky bit；常用于限制共享目录互相删除文件
umask                                   # 查看当前 shell 的默认权限掩码
umask 027                               # 设置当前 shell 新建项权限掩码；常见文件权限结果为 640、目录为 750
sudo chown <用户>:<组> <路径>            # 同时修改属主和属组
sudo chown <用户> <文件>                 # 只修改属主；不指定冒号和组名
sudo chown :<组> <文件>                  # 只修改所属组
sudo chown -R <用户>:<组> <目录>         # [高风险] 递归修改属主和属组；先核对目标路径及软链接影响
sudo chown --reference=<参考文件> <目标文件>
# 将目标的属主和属组设置为参考文件的属主和属组；--reference 从参考文件读取属性
sudo chown -h <用户> <符号链接>          # 修改符号链接本身；-h 不跟随链接目标
sudo chgrp <组> <路径>                   # 修改文件/目录所属组
sudo chgrp -R <组> <目录>                # 递归修改目录树所属组；先核对目标路径
getfacl <路径>                           # 查看传统权限位之外的 POSIX ACL 访问控制列表
sudo setfacl -m u:<用户>:rwx <目录>      # 给指定用户增加 ACL；-m 修改，rwx 表示读写和遍历目录
sudo setfacl -m g:<组>:rx <目录>         # 给指定组增加 ACL 只读和遍历权限
sudo setfacl -m d:u:<用户>:rwx <目录>    # 设置目录默认 ACL，使后续新建项继承用户权限规则
sudo setfacl -x u:<用户> <路径>           # 删除指定用户的 ACL 条目；-x 删除单条 ACL
sudo setfacl -b <路径>                   # 清除此路径 ACL 扩展条目；会移除已有 ACL 授权，先备份/确认
sudo chmod 700 /home/<用户>/.ssh         # 将 SSH 私钥目录限制为用户可访问；700 为属主 rwx、其他无权限
sudo chmod 600 /home/<用户>/.ssh/authorized_keys
# 将公钥授权文件限制为属主可读写；600 为 rw-------，其他用户无权限
sudo chown -R <用户>:<组> /home/<用户>/.ssh
# [高风险] 将 SSH 配置目录及内容归属修正为目标用户/组；-R 递归，先核实路径和组名
```

`chmod -R` 会把目录和普通文件一起改成同一模式；例如递归设为 `644` 会让目录失去遍历权限。批量修复时要分别处理目录和文件，且先用 `find ... -print` 检查目标。`chown -R` 不要用于 `/` 或未经确认的变量路径。

**常见权限故障修复流程**：先用 `namei -l <路径>` 找到路径上哪一级缺少 `x` 遍历权限，再用 `stat`/`getfacl` 确认文件权限及 ACL；用 `chown`、`chmod` 或 `setfacl` 只补足所需权限。Web 服务运行用户因发行版而异：Debian/Ubuntu 常见 `www-data`，Fedora/RHEL 的 Apache 常见 `apache`、Nginx 常见 `nginx`；先查实际服务账户，不要猜测。

### 3.4 SSH 安全

两边常见服务端配置文件均为 `/etc/ssh/sshd_config`。改配置前保留当前 SSH 会话，并确认云控制台或带外访问可用。

```bash
sudoedit /etc/ssh/sshd_config            # 以管理员身份安全编辑 SSH 服务端配置
sudo sshd -t                             # 检查服务端配置语法；无输出通常代表语法通过
sudo systemctl restart ssh               # Debian/Ubuntu 常见服务单元名为 ssh
sudo systemctl restart sshd              # Fedora/RHEL 常见服务单元名为 sshd
```

| 配置项                   | 示例值 | 含义与注意事项                                           |
| ------------------------ | ------ | -------------------------------------------------------- |
| `Port`                   | `2222` | 修改监听端口；需同步调整防火墙、云安全组和客户端连接参数 |
| `PermitRootLogin`        | `no`   | 禁止 root 直接 SSH 登录                                  |
| `PasswordAuthentication` | `no`   | 禁用密码认证；确认密钥登录成功后再关闭                   |
| `PubkeyAuthentication`   | `yes`  | 启用公钥认证                                             |
| `MaxAuthTries`           | `3`    | 限制每次连接的认证尝试次数                               |

密钥登录需先把公钥放入目标用户 `~/.ssh/authorized_keys`，并确认目录/文件权限正确。`ssh -p <端口> -i <私钥> <用户>@<主机>` 中 `-p` 指定 SSH 端口，`-i` 指定私钥；`ssh <用户>@<主机> "<远程命令>"` 可远程执行命令。

<a id="sec-packages"></a>

## 4. 软件包、文件与软件源

### 4.1 包管理器常用查询

| 功能           | Debian 系命令及参数                                          | Red Hat 系命令及参数                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 安装           | `apt install <包>`：安装并解依赖                             | `dnf install <包>`：安装并解依赖                             |
| 卸载 / 清配置  | `apt remove <包>`：卸载但保留配置；`apt purge <包>`：同时清理包配置 | `dnf remove <包>`：卸载；必要时检查包留下的配置文件          |
| 已安装列表     | `dpkg -l`：查询 dpkg 包状态；`apt list --installed`：列出已安装包 | `rpm -qa`：查询所有已安装 RPM；`dnf list installed`：DNF 格式列出 |
| 文件归属       | `dpkg -S <路径>`：查询路径归属包                             | `rpm -qf <路径>`：查询路径归属包                             |
| 包文件清单     | `dpkg -L <包>`：列出已安装包文件                             | `rpm -ql <包>`：列出已安装包文件                             |
| 下载但不安装   | `apt download <包>`：下载当前目录中的 `.deb`                 | `dnf download <包>`：由 `dnf-plugins-core` 等提供，先确认命令可用 |
| 重装           | `apt install --reinstall <包>`：重装包                       | `dnf reinstall <包>`：重装包                                 |
| 本地包文件详情 | `dpkg -I <文件>.deb`：查看 deb 元数据；`dpkg -c <文件>.deb`：列出包内文件 | `rpm -qpi <文件>.rpm`：查看 rpm 元数据；`rpm -qpl <文件>.rpm`：列出包内文件 |
| 直接安装本地包 | `dpkg -i <文件>.deb`：安装但不自动拉依赖                     | `rpm -ivh <文件>.rpm`：`-i` 安装、`-v` 详细输出、`-h` 进度条；不自动解决依赖 |
| 推荐本地包安装 | `apt install ./<文件>.deb`：自动解仓库依赖                   | `dnf install ./<文件>.rpm`：自动解仓库依赖                   |
| 修复未完成安装 | `dpkg --configure -a`：配置已解包但未配置的包；`apt --fix-broken install`：修依赖 | DNF 事务按报错处理；`rpm --rebuilddb` 仅用于 RPM 数据库损坏，重建前先备份数据库 |

`.deb` 和 `.rpm` 是不同包格式，不可互换。不要下载一个不匹配发行版版本、架构或依赖的包后强制安装。

### 4.2 软件源与签名

**Debian / Ubuntu**：主配置常见于 `/etc/apt/sources.list`；单独源放入 `/etc/apt/sources.list.d/`。较新的 Ubuntu 版本常用 DEB822 格式 `.sources` 文件。Ubuntu PPA 仅适用于 Ubuntu 生态，不能默认用于 Debian/Kali。

第三方 APT 源应使用项目提供的 GPG 密钥并用 `signed-by` 将密钥限制到单一仓库；不建议使用已弃用且信任范围过大的 `apt-key`。

```bash
sudo install -m 0755 -d /etc/apt/keyrings
# install：安装文件/目录；-m 0755：设目录权限为属主可读写执行、其他人可读执行；-d：创建目录
curl -fsSL <密钥URL> | sudo gpg --dearmor -o /etc/apt/keyrings/<仓库名>.gpg
# curl：获取密钥；-f：HTTP 错误返回失败；-s：静默；-S：失败时显示错误；-L：跟随重定向
# gpg --dearmor：把 ASCII-armored 密钥转为二进制 keyring；-o：指定输出文件
echo "deb [signed-by=/etc/apt/keyrings/<仓库名>.gpg] <仓库URL> <发行版代号> <组件>" |
  sudo tee /etc/apt/sources.list.d/<仓库名>.list
# echo：生成源定义；管道传给 tee；tee 以管理员权限写文件并同时输出内容
sudo apt update
# 刷新 APT 索引，使新仓库可被查询和安装
```

**Fedora / RHEL**：`.repo` 文件放在 `/etc/yum.repos.d/`，常用字段包括仓库 ID、`name`、`baseurl`、`enabled`、`gpgcheck` 和 `gpgkey`。Fedora 常见第三方仓库为 RPM Fusion；RHEL 生态常用 EPEL（需确认与当前发行版版本兼容）。

```bash
dnf repolist                         # 列出已启用仓库
dnf repolist all                     # 列出启用和禁用的仓库
sudo dnf config-manager --set-enabled <仓库ID>  # 启用仓库；config-manager 可能需安装 dnf-plugins-core
sudo dnf config-manager --set-disabled <仓库ID> # 禁用仓库
sudo rpm --import <密钥URL或文件>     # 导入 RPM 仓库签名公钥；先核实密钥来源和指纹
```

只添加可信来源的仓库，开启 GPG 校验；切勿为绕过签名错误而全局关闭校验。

<a id="sec-network-services"></a>

## 5. 网络与服务管理

### 5.1 查看网络与配置静态 IP

```bash
ip addr                               # 查看网络接口、IP 地址和链路状态
ip route                              # 查看 IPv4 路由表和默认网关
resolvectl status                     # 查看 systemd-resolved DNS 状态；若未运行则查看本机 DNS 管理工具
nmcli connection show                 # 列出 NetworkManager 连接配置及名称
nmcli device status                   # 列出网卡设备状态及托管情况
```

NetworkManager 两家族均可用时，可用 `nmcli` 配静态 IPv4；将连接名称、地址、网关和 DNS 替换为实际值。远程配置可能中断当前连接，使用前准备回滚路径。

```bash
sudo nmcli connection modify "<连接名>" \
  ipv4.method manual \
  ipv4.addresses "<地址>/<前缀长度>" \
  ipv4.gateway "<网关>" \
  ipv4.dns "<DNS1>,<DNS2>"
# connection modify：修改连接配置；ipv4.method manual：使用静态 IPv4
# ipv4.addresses：地址及 CIDR 前缀；ipv4.gateway：默认网关；ipv4.dns：逗号分隔 DNS 地址
sudo nmcli connection up "<连接名>"
# 重新激活该连接并应用设置；远程操作需防止连接断开
```

Ubuntu 服务器可能使用 Netplan（`/etc/netplan/*.yaml`），旧 Debian 可能使用 `/etc/network/interfaces`；编辑后按实际网络栈应用配置。Netplan YAML 必须使用空格缩进，可先 `sudo netplan try` 进行带自动回滚的测试，再 `sudo netplan apply` 应用。

### 5.2 systemd 服务

```bash
sudo systemctl start <服务>            # 立即启动服务，不设置开机自启
sudo systemctl stop <服务>             # 立即停止服务
sudo systemctl restart <服务>          # 停止后重新启动
sudo systemctl reload <服务>           # 让服务重载配置而尽量不中断；并非所有服务支持
sudo systemctl enable <服务>           # 设置开机自动启动，不立即启动
sudo systemctl disable <服务>          # 取消开机自动启动，不立即停止
sudo systemctl enable --now <服务>     # enable 设置开机自启；--now 同时立即启动
systemctl status <服务>                # 查看服务状态及最近日志摘要
systemctl is-enabled <服务>            # 查询开机自启状态
systemctl --failed                     # 列出启动失败的 systemd 单元
systemctl get-default                  # 查看默认启动 target
sudo systemctl set-default multi-user.target # 默认进入命令行多用户 target，常用于服务器
sudo systemctl set-default graphical.target # 默认进入图形桌面 target
systemctl list-unit-files --type=service --state=enabled # 列出设为 enabled 的服务单元文件
systemd-analyze                        # 查看本次启动总耗时
systemd-analyze blame                  # 按启动耗时列出单元；数值不等同于关键路径耗时
systemd-analyze critical-chain         # 查看关键启动路径
```

服务名可能有家族差异，例如 SSH 的 `ssh` / `sshd`。服务是否支持 `reload`、配置文件位置和默认启用状态也可能不同。

### 5.3 防火墙：UFW、firewalld 与 nftables

先确认现有管理工具和规则。**不要同时把 UFW、firewalld、手写 nftables、Docker/Kubernetes 规则当作互不相关的配置层反复覆盖。** 远程操作前放行 SSH 实际端口并准备控制台。

#### UFW（常见于 Ubuntu；Debian 可安装）

```bash
sudo apt install ufw                      # 在 Debian 系安装 UFW
sudo ufw status verbose                   # 查看启用状态、默认策略和规则
sudo ufw status numbered                  # 带规则编号显示；删除编号前先确认列表
sudo ufw default deny incoming            # [会修改] 默认拒绝主动进入的连接；远程操作前确认 SSH 放行
sudo ufw default allow outgoing           # [会修改] 默认允许本机发出的连接
sudo ufw allow 22/tcp                     # [会修改] 允许 TCP 22；若 SSH 改端口应替换为实际端口
sudo ufw allow 80/tcp                     # 允许 HTTP 的 TCP 80 端口
sudo ufw allow 443/tcp                    # 允许 HTTPS 的 TCP 443 端口
sudo ufw allow from 192.168.1.0/24 to any port 5432 proto tcp
# 仅允许指定 IPv4 网段访问 TCP 5432；将网段和端口替换为实际值
sudo ufw deny 23/tcp                      # 拒绝 TCP 23（Telnet）
sudo ufw delete allow 80/tcp              # 按规则内容删除先前添加的 HTTP 放行规则
sudo ufw delete <编号>                    # 按编号删除规则；编号来自 status numbered
sudo ufw enable                           # [高风险] 启用防火墙；远程操作前必须先确认 SSH 放行并准备控制台
sudo ufw disable                          # [会修改] 关闭 UFW 管理的防火墙规则，可能扩大暴露面
sudo ufw reset                            # [高风险] 重置 UFW 规则并停用；会清除现有配置，先备份规则
```

#### firewalld（常见于 Fedora / RHEL）

firewalld 将规则按 zone 管理，并区分运行时规则和永久规则。不带 `--permanent` 的规则通常重启或 reload 后消失；加 `--permanent` 后通常还需 `--reload` 让运行时规则更新。

```bash
firewall-cmd --state                     # 查看 firewalld 是否正在运行
firewall-cmd --get-default-zone          # 查看默认 zone
firewall-cmd --get-services              # 列出可用的服务名
firewall-cmd --list-all                  # 查看当前默认 zone 的运行时规则
sudo firewall-cmd --add-port=80/tcp      # 临时允许 TCP 80；reload/重启后可能失效
sudo firewall-cmd --permanent --add-port=80/tcp
# [会修改] 将 TCP 80 规则写入永久配置；--permanent 修改持久配置，变更后需 reload
sudo firewall-cmd --permanent --add-service=https
# 永久允许内置 https 服务定义中的端口/协议
sudo firewall-cmd --reload               # [会修改] 将永久配置重新载入运行时；检查远程管理端口不会被阻断
sudo firewall-cmd --permanent --remove-port=80/tcp # [会修改] 从永久配置删除 TCP 80 规则
sudo firewall-cmd --permanent --remove-service=https # [会修改] 从永久配置删除 https 服务规则
sudo firewall-cmd --set-default-zone=public # 设置默认 zone 为 public；先确认接口所属 zone
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="5432" accept'
# 永久允许指定 IPv4 网段访问 TCP 5432；修改后需 reload
```

#### nftables（底层规则框架）

不同发行版可由 UFW/firewalld 管理 nftables，也可直接使用 `nft`。手写规则前检查现有规则及其他管理组件。下面只给出理解规则结构的基础模板；默认拒绝入站会影响远程访问，必须先核实 SSH 端口并准备恢复手段。

```nft
#!/usr/sbin/nft -f
# 指定 nft 命令解释此配置文件

table inet filter {
    # 创建同时覆盖 IPv4/IPv6 的 inet 表；表名为 filter
    chain input {
        # input 链处理发往本机的数据包
        type filter hook input priority 0; policy drop;
        # 在 input hook 的优先级 0 过滤；未匹配规则默认丢弃
        iifname "lo" accept
        # 允许 loopback 回环流量
        ct state established,related accept
        # 允许已建立连接及相关连接的数据包返回
        ct state invalid drop
        # 丢弃连接跟踪状态无效的数据包
        ip protocol icmp accept
        # 允许 IPv4 ICMP 控制报文
        meta l4proto ipv6-icmp accept
        # 允许 IPv6 ICMP；IPv6 邻居发现等功能依赖它
        tcp dport 22 accept
        # 允许 TCP 22；若 SSH 使用其他端口，必须替换
        tcp dport { 80, 443 } accept
        # 允许 HTTP/HTTPS；不提供网站服务时删除此条
    }
    chain forward {
        # forward 链处理经本机转发的数据包
        type filter hook forward priority 0; policy drop;
        # 默认不转发；路由器、容器主机需按实际网络另配
    }
    chain output {
        # output 链处理本机发出的数据包
        type filter hook output priority 0; policy accept;
        # 默认允许本机发出的流量
    }
}
```

```bash
sudo nft list ruleset                   # 显示内核当前加载的全部 nftables 规则
sudo nft list tables                    # 列出所有 nftables 表
sudo nft -a list chain inet filter input # 列出 input 链并显示 handle 编号
sudo nft -c -f /etc/nftables.conf        # -c 仅检查配置语法，不加载规则
sudo nft -f /etc/nftables.conf           # 从文件加载规则，可能改变当前防火墙行为
sudo systemctl enable --now nftables    # 设置 nftables 服务开机启动并立即启动
sudo nft delete rule inet filter input handle <编号>
# 按规则 handle 删除单条规则；编号须从 nft -a 输出确认
```

`drop` 静默丢弃数据包；`reject` 拒绝并返回错误响应。不要盲目清空规则集或删除整张表；容器网络规则可能由其他组件维护。

### 5.4 AppArmor 与 SELinux

| 项目        | AppArmor（常见于 Ubuntu）                                    | SELinux（常见于 Fedora/RHEL）                                |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 控制方式    | 依据程序路径及 profile 限制访问                              | 依据进程、文件、端口的安全上下文和策略                       |
| 状态 / 诊断 | `sudo aa-status` 查看 profile 状态                           | `getenforce` 查看 Enforcing/Permissive/Disabled              |
| 临时排障    | `sudo aa-complain <程序路径>` 记录但不强制；`sudo aa-enforce <程序路径>` 恢复强制 | `sudo setenforce 0` 临时切到 Permissive；`sudo setenforce 1` 恢复 Enforcing |
| 重要工具    | `aa-status`、`aa-complain`、`aa-enforce`                     | `ausearch`、`semanage`、`restorecon`、`setsebool`            |

优先修复策略或标签，不要长期关闭强制访问控制。AppArmor 配置常见于 `/etc/apparmor.d/`，本地覆盖常放在 `/etc/apparmor.d/local/`。SELinux 永久模式由 `/etc/selinux/config` 中的 `SELINUX=` 控制；从 Disabled 恢复可能需要完整重新标记，按发行版文档操作。

```bash
sudo aa-status                         # 查看 AppArmor 状态及加载 profile
sudo aa-status | grep nginx            # 过滤查看与 nginx 相关的 profile 信息
sudo aa-complain /usr/sbin/nginx       # 将该程序 profile 切到 complain，仅记录违规
sudo aa-enforce /usr/sbin/nginx        # 将该程序 profile 恢复为 enforce 强制模式

getenforce                             # 输出当前 SELinux 模式
sudo setenforce 0                      # 临时切为 Permissive，仅用于短时排障
sudo setenforce 1                      # 临时恢复 Enforcing
ls -Z /var/www/html/                   # 查看文件 SELinux 安全上下文
ps -eZ | grep nginx                    # 查看进程安全上下文并筛选 nginx
sudo ausearch -m AVC -ts recent        # 查询近期 SELinux AVC 拒绝事件
sudo semanage port -a -t http_port_t -p tcp 8080
# 将 TCP 8080 标记为 http_port_t；-a 添加、-t 指定类型、-p 指定协议
sudo semanage fcontext -a -t httpd_sys_content_t '/srv/myweb(/.*)?'
# 为目录及其子路径添加持久文件上下文规则；-a 添加、-t 指定类型
sudo restorecon -Rv /srv/myweb         # 按策略恢复目录标签；-R 递归、-v 输出变更
getsebool -a | grep httpd              # 列出并筛选 SELinux 布尔值
sudo setsebool -P httpd_can_sendmail on # 永久启用指定布尔值；-P 写入持久策略
```

若服务启动失败，查 AVC 拒绝记录并修复端口类型、文件上下文或布尔值。不要把“关闭 SELinux 后能启动”当作修复完成。

<a id="sec-files-shell"></a>

## 6. 文件、查找和系统信息

### 6.1 磁盘空间与挂载

```bash
df -hT                                 # 以易读单位显示文件系统容量，并显示文件系统类型
df -i                                  # 查看 inode 使用量；空间充足但 inode 耗尽也会无法新建文件
du -sh <路径>                          # -s 汇总路径占用；-h 用易读单位显示
du -h --max-depth=1 <目录>             # 统计目录下一层占用；max-depth 限制递归显示深度
du -x -h --max-depth=1 /               # -x 不跨文件系统；查看根文件系统一层目录空间分布
```

`/etc/fstab` 每行包含六个字段：设备/UUID、挂载点、文件系统类型、挂载选项、dump 备份标志、fsck 检查顺序。修改前备份；保存后用 `sudo findmnt --verify` 检查（如版本支持），并谨慎用 `sudo mount -a` 验证，避免开机挂载失败。

### 6.2 find、locate 与文本查看

| 命令                          | 常用参数 / 作用                                              |
| ----------------------------- | ------------------------------------------------------------ |
| `find <起点> -name '<模式>'`  | 实时递归按名称查找；`-iname` 忽略大小写；`-type f/d` 限制普通文件/目录 |
| `find ... -size +500M`        | 按大小查找；`+` 大于、`-` 小于，`M/G/k` 为大小单位           |
| `find ... -mtime +30`         | 按修改时间查找；`+30` 表示超过约 30 天                       |
| `find ... -user <用户>`       | 按属主筛选；`-perm -o+w` 查其他用户可写项                    |
| `find ... -exec <命令> {} \;` | 对每个结果执行一次命令；`{}` 是结果路径，`\;` 结束执行表达式 |
| `find ... -exec <命令> {} +`  | 批量把结果传给命令，通常比逐个执行效率高                     |
| `find ... -delete`            | 删除匹配结果；先去掉 `-delete` 检查列表，慎用                |
| `locate <词>`                 | 查询预建数据库，速度快但结果可能过期                         |
| `locate -i <词>`              | 忽略大小写查找                                               |
| `locate -c <词>`              | 只输出匹配数量                                               |
| `locate -l 10 <词>`           | 最多输出 10 条                                               |
| `locate -r '<正则>'`          | 用正则表达式匹配路径                                         |
| `locate -b <词>`              | 只按路径基名匹配                                             |
| `locate -e <词>`              | 只显示当前仍存在的路径                                       |
| `locate -w <路径>`            | 匹配完整路径                                                 |
| `locate -A <词1> <词2>`       | 要求结果同时包含多个搜索词                                   |
| `sudo updatedb`               | 更新 locate 数据库；新文件在数据库更新前可能查不到           |
| `less <文件>`                 | 分页查看大文件；空格下一页、`b` 上一页、`/词` 搜索、`q` 退出 |
| `head -n 20 <文件>`           | 查看前 20 行；`-c 100` 表示查看前 100 字节                   |
| `tail -n 50 <文件>`           | 查看最后 50 行；`-f` 跟踪追加内容，`-F` 可在日志轮转后重开文件 |
| `grep -i -n '<词>' <文件>`    | `-i` 忽略大小写、`-n` 显示行号                               |
| `grep -r '<词>' <目录>`       | 递归搜索目录；`-v` 排除匹配行；`-E` 启用扩展正则             |

```bash
find /var/log -type f -name '*.log' -mtime +30 -print
# 实时查找 /var/log 下普通文件；先打印 30 天前修改的 .log 文件，不删除
find / -type f -size +500M 2>/dev/null
# 从根目录查找大于 500 MB 的普通文件；2>/dev/null 丢弃权限错误等标准错误
find . -type f -name '*.sh' -exec chmod +x {} +
# 查当前目录树中所有 .sh 普通文件并批量添加执行权限
find . -path './node_modules' -prune -o -name '*.js' -print
# 遇到 node_modules 目录即剪枝不进入；否则打印名称匹配 .js 的路径
locate -i nginx
# 在 locate 数据库中忽略大小写查找含 nginx 的路径；结果不保证实时
grep -E -i -n 'error|fail' app.log
# 搜索 error 或 fail；-E 使用扩展正则，-i 忽略大小写，-n 输出行号
```

### 6.3 系统、CPU、内存与进程

```bash
uptime                                 # 显示运行时长、登录用户数和系统负载
free -h                                # 以易读单位显示内存和 swap；重点关注 available
free -m                                # 以内存 MB 单位显示
free -s 5                              # 每 5 秒刷新一次内存统计
lscpu                                  # 显示 CPU 架构和处理器信息
nproc                                  # 显示当前可用处理器数量
lsblk                                  # 列出块设备、分区及挂载点
ps aux                                # BSD 风格列出进程；a/x 扩展显示终端/无终端进程，u 显示用户格式
ps -ef                                # POSIX 风格列出完整进程信息
ps -ef --forest                       # 以树状显示进程父子关系
top                                   # 实时查看进程；交互键 P 按 CPU、M 按内存、k 发信号、q 退出
top -p <PID>                          # 仅监控指定 PID
top -u <用户>                         # 仅显示指定用户进程
pgrep -a <进程名>                     # 按名称查进程并显示命令行
kill <PID>                            # 默认发送 SIGTERM，请求进程优雅退出
kill -15 <PID>                        # 显式发送 SIGTERM
kill -9 <PID>                          # 发送 SIGKILL，无法捕获；仅在优雅终止无效时使用
pkill <进程名>                         # 按进程名称匹配并发送默认 SIGTERM
killall <进程名>                       # 向匹配名称的进程发送信号，注意匹配范围
vmstat 1                              # 每 1 秒输出虚拟内存、进程和 CPU 统计
iostat -xz 1                          # 每 1 秒输出扩展设备 I/O 统计；通常需安装 sysstat
sar -u 1 5                            # 每 1 秒采样 CPU 共 5 次；通常需安装 sysstat
ss -tlnp                              # 查看监听 TCP 端口；-t TCP、-l 监听、-n 数字、不解析名称、-p 显示进程
ss -tunap                             # 查看 TCP/UDP 网络套接字；-a 全部、-u UDP、-p 进程
sudo lsof -i :<端口>                  # 查哪个进程打开指定网络端口
```

常见可选工具安装：Debian 系用 `sudo apt install htop sysstat lsof tcpdump net-tools`；Fedora/RHEL 用 `sudo dnf install htop sysstat lsof tcpdump net-tools`。`netstat`/`ifconfig` 属于较旧的 net-tools；优先用 `ss`/`ip`。

POSIX ACL 命令 `getfacl` 和 `setfacl` 通常由 `acl` 软件包提供；若命令缺失，Debian 系安装 `sudo apt install acl`，Fedora/RHEL 安装 `sudo dnf install acl`。`namei` 通常由 util-linux 提供。

### 6.4 `/proc` 虚拟文件系统

`/proc` 由内核动态生成，不是普通磁盘目录。`/proc/<PID>/cmdline` 是进程启动参数的 NUL 分隔快照，不记录该进程运行期间接收到的所有输入。

| 路径                                                 | 内容                                         |
| ---------------------------------------------------- | -------------------------------------------- |
| `/proc/<PID>/status`、`cmdline`、`environ`           | 进程状态、启动命令行、环境变量（含敏感信息） |
| `/proc/<PID>/fd/`、`maps`、`io`、`limits`            | 文件描述符、内存映射、I/O 统计、资源限制     |
| `/proc/<PID>/ns/`、`cgroup`                          | 命名空间和 cgroup 归属                       |
| `/proc/cpuinfo`、`meminfo`、`loadavg`、`uptime`      | CPU、内存、负载和运行时间信息                |
| `/proc/mounts`、`filesystems`、`swaps`、`partitions` | 挂载、支持的文件系统、swap 和分区信息        |
| `/proc/stat`、`vmstat`、`interrupts`、`diskstats`    | CPU、虚拟内存、中断和磁盘统计                |
| `/proc/net/`                                         | 网络协议栈信息                               |
| `/proc/sys/`                                         | 可调内核参数；优先使用 `sysctl` 管理         |

```bash
cat /proc/$$/cmdline | tr '\0' ' '
# 查看当前 shell 的参数；$$ 是当前 shell PID；cmdline 用 NUL 分隔，tr 转成空格
cat /proc/$$/environ | tr '\0' '\n'
# 查看当前 shell 环境变量；环境中可能有令牌或密码，不要公开输出
readlink /proc/$$/exe                 # 查看当前 shell 对应的可执行文件路径
readlink /proc/$$/cwd                 # 查看当前 shell 工作目录
ls -l /proc/$$/fd                      # 查看当前 shell 打开的文件描述符及链接目标
sysctl vm.swappiness                  # 查看指定内核参数
sudo sysctl -w vm.swappiness=10        # 临时设置参数；重启后可能恢复
sudo sysctl -p                         # 加载 /etc/sysctl.conf 中的参数
```

要跨重启持久设置内核参数，应在 `/etc/sysctl.d/*.conf` 建立受控配置并用 `sysctl --system` 加载；不建议直接写 `/proc/sys` 文件。审计运行时输入需结合 `strace`、auditd、应用日志或网络监控，不能仅依赖 `/proc/<PID>/cmdline`。

### 6.5 Bash Shell 常用实战与脚本

本节以 Bash 为例，适用于两大发行版家族中安装了 Bash 的系统。交互命令可直接输入；多步操作和重复任务建议保存成 `.sh` 文件，用 `bash <脚本>` 执行或设置执行权限后运行。`<占位符>` 必须替换为实际路径、用户名或参数。

#### 命令组合、变量和重定向

| 写法                        | 功能                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `命令1 \| 命令2`            | 管道：将命令 1 的标准输出送给命令 2 的标准输入；默认不传递 stderr |
| `命令 > 文件` / `>> 文件`   | 标准输出写入文件并覆盖 / 追加到文件                          |
| `命令 2> 文件` / `2>> 文件` | 标准错误写入文件并覆盖 / 追加                                |
| `命令 > 文件 2>&1`          | 标准输出写入文件，再把标准错误也重定向到同一处；顺序有意义   |
| `命令 &> 文件`              | Bash 写法，同时重定向 stdout 和 stderr                       |
| `命令1 && 命令2`            | 仅当命令 1 退出状态为 0（成功）时执行命令 2                  |
| `命令1 \|\| 命令2`          | 仅当命令 1 退出状态非 0（失败）时执行命令 2                  |
| `命令1 ; 命令2`             | 无论命令 1 成功与否都执行命令 2                              |
| `$(命令)`                   | 命令替换：将命令输出作为字符串嵌入当前命令                   |
| `"$变量"`                   | 双引号保留变量内容中的空格为单个参数，并进行变量/命令替换    |
| `'$变量'`                   | 单引号保留字面文本，不展开变量或命令替换                     |
| `"$@"`                      | 在脚本中将每个传入参数分别保留为独立参数；比 `$*` 更适合转发参数 |
| `"$?"`                      | 最近一条命令的退出码；0 通常代表成功，非 0 通常代表失败      |
| `"$#"`                      | 当前脚本或函数收到的位置参数数量                             |
| `"$1"`、`"$2"`              | 第一个、第二个位置参数                                       |
| `"$@"`                      | 所有位置参数，每个参数保持单独边界；需加引号使用             |
| `"$!"`                      | 最近一个后台任务的进程 ID                                    |
| `$$`                        | 当前 shell 进程 ID                                           |

```bash
printf '%s\n' "$HOME"                  # 用 printf 按一行输出 HOME 变量；双引号保护路径中的空格
date '+%F %T'                          # 输出 YYYY-MM-DD HH:MM:SS 格式的本地时间
STAMP=$(date '+%Y%m%d-%H%M%S')         # 用命令替换保存格式化时间；等号两边不能有空格
printf 'backup-%s.tar.gz\n' "$STAMP"   # 将变量作为字符串参数传给 printf，避免未引用展开
grep -i 'error' app.log | tail -n 20  # 先忽略大小写筛选 error 行，再显示最后 20 行
command > output.log 2>&1              # 覆盖日志文件并将 stdout、stderr 都写入该文件
command >> output.log 2>&1             # 追加 stdout、stderr 到日志文件，不覆盖已有内容
command | tee output.log                # stdout 同时显示在终端和文件中；tee 默认覆盖文件
command 2>&1 | tee -a output.log        # 先合并 stderr 到 stdout，再由 tee -a 追加日志
command_a && command_b                  # 仅 command_a 成功时运行 command_b
command_a || printf '失败，退出码=%s\n' "$?" # command_a 失败时输出错误提示及最近命令的退出码
```

Shell 会对未加引号的变量做分词和通配符展开。传递路径、文件名、用户输入时，优先使用 `"$变量"`；删除或批量处理前先打印待处理清单。环境变量仅在当前 shell 及其子进程中有效，使用 `export` 才会传给子进程。

#### 文件清单、文本处理与批量操作

```bash
printf '%s\n' *.log                     # 每行输出当前目录匹配的日志文件名；没有匹配时 Bash 默认保留字面模式
printf '%s\n' "$PWD"                     # 输出当前工作目录变量；双引号避免空格拆分
read -r -p '请输入服务名: ' service      # 显示提示并读入一行；-r 禁止反斜杠转义
printf '%s\n' "$service"                 # 安全显示读取到的文本
cut -d: -f1 /etc/passwd                  # -d 指定冒号为字段分隔符；-f1 输出第一列用户名
awk -F: '{print $1, $3}' /etc/passwd     # -F 指定冒号分隔；输出第一列用户名和第三列 UID
sort -k2,2n usage.txt                    # 按第二列数值升序排序；-k2,2 指定排序字段，n 表示数值比较
sort access.log | uniq -c | sort -nr     # 排序相同行、统计重复数，再按计数数值降序排列
wc -l app.log                            # 统计文件行数；-l 只显示行数
sed -n '1,20p' app.log                   # 只打印第 1 至 20 行；-n 禁止默认输出，p 显示匹配范围
grep -RIn --exclude-dir=.git 'TODO' .   # 递归搜索；-R 递归、-I 忽略二进制、-n 行号、排除 .git 目录
find . -type f -name '*.tmp' -print     # 查找临时文件并只打印，不执行删除
find . -type f -name '*.tmp' -print0 | xargs -0 -r ls -l
# -print0 和 -0 用 NUL 分隔以安全处理空格文件名；-r 在没有输入时不运行 ls
```

`sort | uniq` 通常需要先排序，因为 `uniq` 只合并相邻的重复行。`xargs` 批量执行外部命令时应配合 NUL 分隔，避免文件名中空格、引号或换行导致参数错位。

#### grep、awk、sed 三剑客实战

三者常在管道中组合使用：`grep` 筛选行，`awk` 按字段分析和汇总，`sed` 流式替换或编辑。先对原始数据只读预览，确认输出后再写文件；不要直接对系统配置或生产日志进行未经备份的原地修改。

**grep 常用参数**

| 参数                                    | 功能                                            |
| --------------------------------------- | ----------------------------------------------- |
| `-i`                                    | 忽略大小写                                      |
| `-n`                                    | 显示匹配行号                                    |
| `-v`                                    | 反选，显示不匹配的行                            |
| `-E`                                    | 扩展正则；`a|b` 匹配 a 或 b，`+` 匹配一个或多个 |
| `-F`                                    | 固定字符串匹配，不把正则元字符当作特殊语法      |
| `-r` / `-R`                             | 递归搜索；`-R` 也跟随目录符号链接，慎用         |
| `-l` / `-c`                             | 只显示有匹配的文件名 / 只显示匹配行数           |
| `-w`                                    | 匹配完整单词                                    |
| `-A <N>` / `-B <N>` / `-C <N>`          | 输出匹配行之后 / 之前 / 前后各 N 行上下文       |
| `--include=<模式>` / `--exclude=<模式>` | 递归时限定包含 / 排除的文件名模式               |

```bash
grep -n 'ERROR' app.log
# 搜索区分大小写的 ERROR；-n 同时显示行号
grep -iE 'error|failed|timeout' app.log
# -i 忽略大小写；-E 启用扩展正则；竖线表示匹配任一关键词
grep -F 'a.b[0]' app.log
# -F 把点号和方括号当作普通字符，而不是正则元字符
grep -v '^#' app.conf
# -v 显示不匹配行；^# 表示以 # 开头，可用于排除注释
grep -nC 2 'listen' nginx.conf
# -n 显示行号；-C 2 同时显示匹配行前后各 2 行
grep -RIn --include='*.conf' --exclude-dir=.git 'PermitRootLogin' /etc/ssh
# -R 递归并跟随目录链接；-I 忽略二进制；-n 显示行号
# --include 限定 .conf 文件；--exclude-dir 排除 .git 目录
grep -c ' 500 ' access.log
# 统计包含 HTTP 状态码 500 的匹配行数；-c 统计行而非总匹配次数
```

**awk 常用参数和语法**

| 写法 / 参数                     | 功能                                |
| ------------------------------- | ----------------------------------- |
| `-F '<分隔符>'`                 | 指定输入字段分隔符；默认按空白分列  |
| `$1`、`$2`、`$NF`               | 第一列、第二列、最后一列            |
| `NR` / `FNR`                    | 当前记录总行号 / 当前输入文件内行号 |
| `NF`                            | 当前记录字段数量                    |
| `-v name=value`                 | 将 shell 值作为 awk 变量传入        |
| `BEGIN { ... }` / `END { ... }` | 输入前 / 全部输入处理结束后执行     |
| `条件 { 动作 }`                 | 仅对满足条件的记录执行动作          |

```bash
awk -F: '{print $1, $3}' /etc/passwd
# -F: 指定冒号分隔；输出用户名字段 $1 和 UID 字段 $3
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd
# 只输出第三列 UID 大于等于 1000 的记录
awk 'NF >= 5 {print NR, $0}' data.txt
# NF >= 5 筛选至少五列的行；NR 是行号；$0 是整行
awk -F, 'NR == 1 {print; next} {sum += $3} END {print "合计:", sum}' data.csv
# data.csv 按逗号分隔；先输出表头并用 next 跳过本轮；累加第三列并在 END 输出总和
awk -F: -v wanted='alice' '$1 == wanted {print $0}' /etc/passwd
# -v 将值传入变量 wanted；筛选第一列精确等于 alice 的记录
```

**sed 常用参数和表达式**

| 参数 / 表达式     | 功能                                              |
| ----------------- | ------------------------------------------------- |
| `-n`              | 关闭默认逐行输出，通常与 `p` 打印命令搭配         |
| `-E`              | 使用扩展正则表达式                                |
| `-e '<表达式>'`   | 添加一条编辑表达式；可重复指定                    |
| `-i`              | 原地修改文件；先预览并备份                        |
| `-i.bak`          | GNU sed 原地修改前将原文件备份为 `.bak`           |
| `s/正则/替换/`    | 每行只替换首个匹配；末尾加 `g` 替换每行全部匹配   |
| `地址p` / `地址d` | 打印 / 从输出流删除指定行；不加 `-i` 不修改源文件 |

```bash
sed -n '1,20p' app.log
# -n 禁止默认输出；1,20p 只打印第 1 至 20 行
sed -n '/ERROR/p' app.log
# 只打印匹配 ERROR 的行，原文件不变
sed '/^[[:space:]]*#/d' app.conf
# 从输出中删除以空白（可选）后跟 # 开头的注释行
sed 's/旧值/新值/g' app.conf
# 在输出中把每行所有旧值替换为新值；不写回原文件
sed -i.bak 's/旧值/新值/g' app.conf
# [会修改] 原地全局替换并先备份为 app.conf.bak；执行前先运行上一条预览
```

不同 sed 实现的 `-i` 参数语法略有不同；本文以 GNU sed 为主。重要配置优先输出到临时文件，检查 `diff -u 原文件 临时文件`，确认无误并通过配置语法检查后再替换。

**与 journalctl、find、sort、uniq、cut 等组合**

```bash
journalctl -u sshd -b --no-pager | grep -iE 'failed|invalid|error' | tail -n 30
# journalctl 按服务和本次启动筛选；grep 匹配错误关键词；tail 只显示最后 30 行
cut -d: -f1,3 /etc/passwd | awk -F: '$2 >= 1000 {print $1, $2}'
# cut 用冒号提取第 1、3 列；awk 再筛选 UID >= 1000 的账户
awk '$9 == 500 {count++} END {print "HTTP 500:", count+0}' access.log
# 对常见空白分隔访问日志计数；$9 状态码字段依日志格式而异，先确认格式
grep -E ' 5[0-9]{2} ' access.log | awk '{count[$9]++} END {for (code in count) print code, count[code]}'
# grep 筛选 5xx 行；awk 按状态码字段计数；字段位置依访问日志格式调整
sort access.log | uniq -c | sort -nr | head -n 10
# sort 先让相同行相邻；uniq -c 计数；sort -nr 按数值逆序；head 取前十项
find /var/log -type f -name '*.log' -print0 |
  xargs -0 -r grep -nH -E 'error|failed'
# find -print0 与 xargs -0 用 NUL 安全传递含空格的路径；-r 无输入时不执行
# grep -nH 输出行号和文件名；-E 用扩展正则筛选错误文本
jq -r '.user // empty' app.jsonl | sort | uniq -c | sort -nr
# jq 提取 JSON 的 user 字段，// empty 忽略缺失或 null；再统计并按次数倒序排列
```

`awk` 默认按空白切列，不是完整 CSV 解析器；复杂 CSV 用专用解析工具。结构化 JSON 优先使用 `jq`。所有写文件或原地修改操作先用只读命令预览。

#### 条件、循环、函数与作业控制

```bash
test -f <文件> && printf '普通文件存在\n'
# test -f 检查路径是否为普通文件；&& 仅在检查成功时打印
test -d <目录> && printf '目录存在\n'
# test -d 检查路径是否为目录
if [ -r <文件> ]; then printf '可读\n'; else printf '不可读\n'; fi
# [ -r ] 检查当前用户是否有读取权限；if 根据退出状态选择分支
for file in ./*.log; do
  [ -e "$file" ] || continue                 # 若通配符没有匹配项则跳过，避免把字面 *.log 当文件名
  printf '日志文件: %s\n' "$file"            # 输出当前匹配项；引用变量保留文件名中的空格
done
for ((i=1; i<=5; i++)); do printf '%s\n' "$i"; done
# Bash 算术循环；i=1 初始化、i<=5 为继续条件、i++ 每次递增 1
function greet() { printf '你好，%s\n' "$1"; }
# 定义函数 greet；$1 是调用时传入的第一个参数
greet 'Linux 用户'
# 调用 greet 函数；单引号使参数作为一个完整字符串传递
sleep 60 &
# 将 sleep 进程放入后台运行；& 使 shell 立即返回提示符
jobs -l
# 查看当前 shell 的后台/暂停作业；-l 同时显示进程 ID
fg %1
# 将作业编号 1 放回前台；编号先用 jobs 确认
```

`[ ... ]` 是 `test` 的兼容写法；比较字符串和整数时注意 `=`/`-eq` 等运算符不同。条件表达式中引用变量可以避免空值、空格造成的语法问题。

#### 脚本编写与运行

脚本建议声明解释器、检查参数、启用安全选项、使用绝对或明确的路径、记录错误并返回非零退出码。`set -euo pipefail` 会使脚本在未处理错误、未定义变量或管道前段失败时退出；涉及预期失败的命令应显式放入条件判断。

```bash
#!/usr/bin/env bash
# 通过 env 在当前 PATH 中定位 Bash，避免写死不同系统上的 Bash 安装路径
set -euo pipefail
# -e：未被条件结构处理的命令失败时退出；-u：未定义变量使用时报错退出
# -o pipefail：管道中任一命令失败时，整条管道按失败处理

usage() {
  printf '用法: %s <用户名>\n' "$0" >&2
  # $0 是脚本名；>&2 将用法信息写到标准错误
}

if [ "$#" -ne 1 ]; then
  # $# 是参数个数；-ne 表示数值不等于；要求恰好一个位置参数
  usage
  exit 2
  # 以 2 表示命令行参数错误退出；0 通常表示成功
fi

username=$1
# 把第一个参数赋给变量；变量赋值的等号两侧不能加空格
printf '正在处理用户: %s\n' "$username"
# 使用格式字符串和带引号的参数，安全显示用户输入
```

将示例保存为 `example.sh` 后：

```bash
bash -n example.sh                     # -n 只检查 Bash 语法，不实际执行脚本
chmod u+x example.sh                   # 给当前文件属主增加执行权限；u 表示属主，+x 表示增加执行位
./example.sh alice                     # 从当前目录执行脚本并传入 alice 作为第一个参数
shellcheck example.sh                  # 若已安装 ShellCheck，执行静态检查并报告常见脚本问题
```

#### 实战脚本一：带保留期的目录归档备份

脚本要求显式传入源目录、备份目录和保留天数；创建 gzip tar 归档后验证文件，并仅清理备份目录顶层、名称符合本脚本格式的旧归档。先在测试目录验证；备份目录不要放在源目录之内。

```bash
#!/usr/bin/env bash
# 由当前环境 PATH 定位 Bash 解释器
set -euo pipefail
# -e 命令失败退出；-u 未定义变量时报错；pipefail 使管道前段失败也返回失败

usage() {
  printf '用法: %s <源目录> <备份目录> <保留天数>\n' "$0" >&2
  # 把用法写到标准错误；$0 表示脚本路径
}

if [ "$#" -ne 3 ]; then
  # $# 为参数个数；-ne 数值不等于；脚本需要三个参数
  usage
  exit 2
  # 参数错误时用非零状态退出
fi

source_dir=$(realpath -e -- "$1")
# realpath -e 要求源路径存在；-- 结束选项解析，避免以连字符开头的路径被误认为选项
backup_dir=$2
# 保存备份目录参数；引号在后续引用处保护其中的空格
retention_days=$3
# 保存保留天数文本，下一步验证为正整数后才能用于 find

if [ ! -d "$source_dir" ]; then
  # [ ! -d ] 检查规范化后的源路径不是目录
  printf '错误：源路径不是目录: %s\n' "$source_dir" >&2
  # 输出错误到 stderr
  exit 2
  # 参数/路径无效，以非零状态退出
fi

if [ "$source_dir" = / ]; then
  # 拒绝归档整个根文件系统；这也避免目标目录意外位于根目录树中
  printf '错误：不允许将根目录 / 作为备份源\n' >&2
  # 把拒绝原因写到标准错误
  exit 2
  # 不安全的源路径直接终止
fi

case "$retention_days" in
  ''|*[!0-9]*)
    # 拒绝空值和任何非数字字符，避免无效或危险的 find 时间参数
    printf '错误：保留天数必须是非负整数\n' >&2
    # 输出参数校验错误到 stderr
    exit 2
    # 参数不合法时退出
    ;;
esac

if [ "$retention_days" -lt 1 ]; then
  # -lt 表示数值小于；至少保留 1 天，避免零天配置造成误删
  printf '错误：保留天数必须大于 0\n' >&2
  # 将错误说明输出到 stderr
  exit 2
  # 参数不符合要求时退出
fi

mkdir -p -- "$backup_dir"
# -p 创建不存在的父目录且目录已存在时不报错；-- 结束选项解析
backup_dir=$(realpath -e -- "$backup_dir")
# 将备份目录转为规范绝对路径，便于校验源/目标是否嵌套

case "$backup_dir/" in
  "$source_dir/"*)
    # 若备份目录等于源目录或位于源目录内部，则归档可能递归包含自身
    printf '错误：备份目录不能等于源目录或位于源目录内部\n' >&2
    # 将路径冲突原因输出到 stderr
    exit 2
    # 检测到源/目标路径冲突时退出
    ;;
esac

timestamp=$(date '+%Y%m%d-%H%M%S')
# 生成文件名用时间戳；%Y 年、%m 月、%d 日、%H 小时、%M 分钟、%S 秒
archive="$backup_dir/backup-$timestamp.tar.gz"
# 拼接归档绝对路径；双引号保留路径中的空格

tar -czf "$archive" -C "$(dirname -- "$source_dir")" "$(basename -- "$source_dir")"
# -c 创建归档；-z gzip 压缩；-f 指定输出文件
# -C 在归档前切到源目录的父目录；basename 只归档源目录本身，避免记录绝对路径
gzip -t -- "$archive"
# -t 校验 gzip 数据流完整性；若校验失败，set -e 会令脚本停止
printf '备份完成: %s\n' "$archive"
# 输出成功归档路径

find "$backup_dir" -maxdepth 1 -type f -name 'backup-*.tar.gz' -mtime "+$retention_days" -print
# -maxdepth 1 只查备份目录顶层；-type f 仅普通文件；-name 限定本脚本归档命名
# -mtime +N 匹配修改时间超过 N 个完整的 24 小时周期的文件；-print 先显示待删除清单
if [ "${CONFIRM_DELETE:-0}" = 1 ]; then
  # 仅当调用者显式设置 CONFIRM_DELETE=1 时才进入旧归档清理；默认只备份、不删除
  find "$backup_dir" -maxdepth 1 -type f -name 'backup-*.tar.gz' -mtime "+$retention_days" -delete
  # [高风险] -maxdepth 1 限制只查顶层；-type f 只匹配普通文件；-name 限定归档命名
  # -mtime 按完整 24 小时周期筛选超期归档；-delete 永久删除，必须先核对上面的预览清单
else
  printf '未清理旧归档；检查清单确认无误后，可用 CONFIRM_DELETE=1 再运行清理。\n'
  # 默认保护旧备份不被删除；确认预览清单后由操作者显式选择启用清理
fi
```

默认运行脚本只创建备份并预览超期归档，不删除文件；确认预览清单无误后，可通过 `CONFIRM_DELETE=1 bash backup.sh <源目录> <备份目录> <保留天数>` 显式启用清理。`find -mtime +N` 按完整的 24 小时周期取整，并不等于日历日期的精确边界；如需严格按日期保留，应采用经过验证的时间戳比较规则。高价值数据还应定期实际执行恢复测试。

**恢复、隔离测试与静态检查**

恢复前先列出归档内容并验证压缩流；确认归档可信后，将文件恢复到新的隔离目录，检查文件属主、权限和内容，再按应用恢复流程迁移到正式位置。不要直接将归档解压覆盖生产目录。

```bash
gzip -t -- /backup/backup-20250101-020000.tar.gz
# [只读] 检查 gzip 数据流完整性；文件损坏时不继续恢复
tar -tzf /backup/backup-20250101-020000.tar.gz
# [只读] -t 列出归档成员，-z 解压 gzip 流，-f 指定归档文件
restore_dir=$(mktemp -d)
# [会修改] 创建新的临时隔离目录并把路径保存到变量，避免覆盖正式数据
tar -xzf /backup/backup-20250101-020000.tar.gz -C "$restore_dir"
# [会修改] -x 解包、-z gzip、-f 指定归档、-C 指定隔离目录；解包前确认列表可信
find "$restore_dir" -maxdepth 3 -type f -print
# [只读] 检查恢复后的文件清单；按需调整 maxdepth 查看深层目录
diff -qr /data/source "$restore_dir/source"
# [只读] 递归比较测试恢复目录与源目录；-q 只报告差异文件
```

用临时数据验证备份与恢复，不要以生产数据作为首次测试对象：

```bash
test_root=$(mktemp -d)
# [会修改] 创建唯一测试目录；确认后续操作只针对这个临时路径
mkdir -p "$test_root/source/subdir" "$test_root/archive"
# [会修改] 创建隔离的源目录和备份目录
printf 'backup test\n' > "$test_root/source/subdir/sample.txt"
# [会修改] 写入测试文件；> 会覆盖同名文件，但此处为刚创建的隔离目录
bash backup.sh "$test_root/source" "$test_root/archive" 30
# 执行默认预览模式；创建归档并显示将被清理的旧档，不删除任何旧档
tar -tzf "$test_root"/archive/backup-*.tar.gz
# [只读] 校验并列出测试归档内文件；确认归档结构包含 source/subdir/sample.txt
test_restore="$test_root/restore"
# 设置独立的恢复测试目录，避免与源目录及归档目录混淆
mkdir -p -- "$test_restore"
# [会修改] 创建隔离的恢复目标目录
tar -xzf "$test_root"/archive/backup-*.tar.gz -C "$test_restore"
# [会修改] 将测试归档解到隔离目录，不覆盖源目录或生产路径
diff -qr "$test_root/source" "$test_restore/source"
# [只读] 递归比较源数据和恢复结果；-q 只报告不同文件，退出码 0 表示相同
shellcheck backup.sh
# 可选静态检查；需预先安装 ShellCheck，检查其报告并修复脚本问题
rm -r -- "$test_root"
# [高风险] 递归删除本次临时测试树；执行前打印并确认 test_root 仍是 mktemp 创建的路径
```

测试后仅在确认变量仍指向本次生成的临时目录时清理测试数据；`rm -r` 会递归删除且无法恢复。ShellCheck 未安装时可跳过静态检查，但仍应使用 `bash -n backup.sh` 检查 Bash 语法。生产清理仍需单独预览名单并明确授权，不要在恢复测试中启用 `CONFIRM_DELETE=1`。

#### 实战脚本二：检查服务状态并写入带时间戳的日志

可通过环境变量 `LOG_FILE` 指定日志文件；默认写入当前目录的 `service-status.log`。执行用户必须对日志目录有写权限，日志文件应由适当的轮转策略管理。

```bash
#!/usr/bin/env bash
# 使用 PATH 中可找到的 Bash 解释器执行脚本
set -u
# 对未定义变量报错并启用管道失败传播；业务检查失败需逐项处理，不启用 -e
set -o pipefail
# 管道任一命令失败均返回失败，避免 tee 写日志失败时伪报成功
log_file=${LOG_FILE:-./service-status.log}
# LOG_FILE 已设置则使用其值，否则默认写当前目录；${变量:-默认值} 是 Bash 参数展开

log() {
  if ! printf '[%s] %s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')" "$*" | tee -a "$log_file"; then
    # 检查写日志管道结果；磁盘满、权限不足等错误必须显式报告
    printf '错误：无法写入巡检日志: %s\n' "$log_file" >&2
    # 错误信息输出到标准错误，方便 cron/systemd 捕获
    exit 1
    # 日志不可写时终止，避免脚本以成功状态结束但没有持久记录
  fi
  # date 生成含时区的时间戳；tee 同时写终端和日志；-a 追加，不覆盖既有日志
}

if [ "$#" -lt 1 ]; then
  # $# 是传入参数数；-lt 表示小于；至少要提供一个服务名
  printf '用法: %s <systemd服务名> [服务名 ...]\n' "$0" >&2
  # 将用法提示发送到标准错误
  exit 2
  # 参数错误时以非零状态退出
fi

status=0
# 初始化总退出状态；只要任一服务失败，最后返回 1

for unit in "$@"; do
  # 遍历每一个参数；"$@" 保持每个参数边界
  if systemctl is-active --quiet "$unit"; then
    # is-active 查询服务是否处于 active；--quiet 不打印状态，只用退出码表示结果
    log "$unit: active"
    # 输出当前服务状态并将带时间戳的记录追加到日志文件
  else
    log "$unit: inactive 或检查失败"
    # systemctl 的非零状态可能表示 inactive、failed 或找不到单元
    details=$(systemctl status --no-pager --full "$unit" 2>&1) || true
    # --no-pager 禁止分页，--full 不截断行；2>&1 捕获标准错误；|| true 避免查询失败中止
    while IFS= read -r line; do
      # -r 保留反斜杠；IFS= 不裁掉行首/行尾空白，逐行读取状态详情
      log "  $line"
      # 每一行详细状态均追加时间戳后写入终端和日志文件
    done <<< "$details"
    # here-string 将捕获的多行状态详情传给 while 循环
    status=1
    # 记录至少一个服务异常；脚本结束时返回非零状态
  fi
done

log "检查结束，退出状态: $status"
# 记录本次巡检结果；非零状态可供 cron 或监控系统识别
exit "$status"
# 将累计状态作为脚本退出码返回，供 cron/监控程序识别成功或失败
```

该脚本只读取服务状态，不自动重启服务，避免因配置错误形成反复重启循环。示例：`LOG_FILE=/var/log/service-check.log ./service-check.sh sshd cron`；日志目录应提前创建并授予执行用户最小写权限。脚本通过 `tee -a` 同时输出到终端与日志文件；若输出中含敏感信息，应限制日志权限。

<a id="sec-logs"></a>

## 7. 日志：journalctl 与传统日志

两大家族使用 systemd 时，`journalctl` 基本通用。不同版本可能不支持所有参数；`journalctl --help` 和 `man journalctl` 可确认本机能力。

| 参数                               | 功能                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `-u <单元>`                        | 按 systemd 单元过滤；可重复指定多个 `-u`                     |
| `-f`                               | 实时跟踪新日志                                               |
| `-n <数量>`                        | 显示最近指定数量的记录                                       |
| `-e`                               | 跳到日志末尾                                                 |
| `-r`                               | 反向输出，最新日志在前                                       |
| `-b [偏移]`                        | 当前启动日志；`-b -1` 上次启动，`-b -2` 上上次启动           |
| `-k`                               | 只看内核日志                                                 |
| `-p <优先级>`                      | 过滤优先级；`err` 表示 err 及更严重级别                      |
| `--since <时间>`、`--until <时间>` | 指定起止时间，如 `today`、`1 hour ago`                       |
| `--no-pager`                       | 不进入分页器，适合脚本和管道                                 |
| `-x`                               | 附加 systemd 提供的解释性提示                                |
| `-g <正则>` / `--grep=<正则>`      | 按消息内容搜索；是否可用取决于 systemd 版本                  |
| `-o <格式>`                        | 选择输出格式，如 `short-iso`、`json-pretty`、`verbose`、`cat` |
| `--disk-usage`                     | 查看 journal 磁盘占用                                        |
| `--vacuum-time=<时长>`             | 删除超出保留时长的归档日志                                   |
| `--vacuum-size=<大小>`             | 清理归档日志以限制占用大小                                   |
| `--vacuum-files=<数量>`            | 限制归档日志文件数量                                         |
| `--list-boots`                     | 列出 journal 中可用的启动记录                                |
| `--verify`                         | 检查 journal 文件完整性                                      |

优先级从高到低：`emerg`(0)、`alert`(1)、`crit`(2)、`err`(3)、`warning`(4)、`notice`(5)、`info`(6)、`debug`(7)。`-p err` 会显示 err 及更高严重级别。

```bash
journalctl -u nginx -b --no-pager -n 100
# 查看本次启动 nginx 单元最近 100 条记录；不分页，适合排障复制
journalctl -u nginx --since '1 hour ago' -p err
# 查看最近一小时 nginx 的错误及更严重日志
journalctl -f -u docker
# 实时跟踪 Docker 单元日志
journalctl -k -b -1
# 查看上一次启动的内核日志
journalctl --since today --until now -o short-iso
# 查看今天截至现在的日志，使用 ISO 时间格式
journalctl -g 'timeout|failed|error' --since '1 hour ago'
# 用正则筛选最近一小时的超时、失败或错误消息；旧 systemd 可改用 grep
journalctl -u nginx -o json-pretty --no-pager > nginx.json
# 以美化 JSON 格式导出 nginx 日志；> 会覆盖目标文件
journalctl --disk-usage
# 查看日志磁盘占用
sudo journalctl --vacuum-time=7d
# 清理超过 7 天的归档日志；执行前确认审计保留要求
sudo journalctl --vacuum-size=500M
# 将归档日志占用收缩到约 500 MB；不是精确的总空间硬上限
sudo mkdir -p /var/log/journal
# 创建持久 journal 目录；-p 表示父目录已存在时不报错
sudo systemd-tmpfiles --create --prefix=/var/log/journal
# 按 systemd tmpfiles 规则创建持久日志目录所需结构
sudo systemctl restart systemd-journald
# 重启日志服务以应用配置
```

传统日志路径可能是 Debian/Ubuntu 的 `/var/log/syslog`、`/var/log/auth.log`，或 Fedora/RHEL 的 `/var/log/messages`、`/var/log/secure`。要持久保存 journal，可在 `/etc/systemd/journald.conf` 设置 `Storage=persistent`、`SystemMaxUse=`、`MaxRetentionSec=`，再重启 journald。

### 7.1 logrotate 日志轮转

应用日志通常由 `logrotate` 按周期归档、压缩并清理旧文件；systemd journal 使用 `journald.conf` 的大小/保留设置，不由普通 logrotate 规则管理。发行版常见全局配置为 `/etc/logrotate.conf`，应用规则放在 `/etc/logrotate.d/`。

示例规则（按实际日志路径、服务用户和信号方式调整）：

```conf
/var/log/myapp/*.log {
    weekly
    # 每周轮转一次；可按写入量改为 daily 或 monthly
    rotate 8
    # 保留 8 个已轮转周期；实际保留时长受运行周期影响
    compress
    # 压缩旧日志，通常使用 gzip
    delaycompress
    # 延迟到下一次轮转再压缩最新归档，便于服务短期继续写旧文件时读取
    missingok
    # 日志文件不存在时不报错
    notifempty
    # 空日志文件不轮转
    create 0640 myapp adm
    # 轮转后创建新日志；模式、属主和属组需按系统账户/服务访问需求调整
    su myapp adm
    # 以指定用户和组执行目录内的轮转操作，避免权限警告；需匹配目录属主
}
```

先检查生效配置和模拟结果，再安排实际轮转：

```bash
sudo logrotate -d /etc/logrotate.conf
# [只读] -d 调试/模拟配置，不实际修改日志或状态文件
sudo logrotate -v /etc/logrotate.conf
# -v 显示处理详情；正常情况下仅轮转符合周期条件的规则
sudo logrotate -f /etc/logrotate.d/myapp
# [高风险] -f 强制轮转指定规则；可能立即重命名/压缩日志，先检查规则、保留要求和服务行为
```

应用持续写入日志时，轮转后可能需要通知服务重新打开日志文件。优先使用应用支持的 reopen/reload 信号和 `postrotate`；`copytruncate` 会复制后截断原文件，轮转瞬间可能丢失少量日志，且不适合所有应用。修改规则后检查 `/var/lib/logrotate/status` 和服务日志；不要直接删除仍被进程打开的日志。

<a id="sec-transfer"></a>

## 8. 远程访问、传输与归档

### 8.1 SSH / SCP

| 命令 / 参数                  | 含义                                 |
| ---------------------------- | ------------------------------------ |
| `ssh <用户>@<主机>`          | 登录远程主机                         |
| `ssh -p <端口>`              | 指定远程 SSH 端口；小写 `-p`         |
| `ssh -i <私钥>`              | 指定身份私钥                         |
| `ssh <用户>@<主机> '<命令>'` | 在远端执行命令                       |
| `scp -r`                     | 递归复制目录                         |
| `scp -P <端口>`              | 指定 SCP 远程端口；大写 `-P`         |
| `scp -i <私钥>`              | 指定身份私钥                         |
| `scp -p`                     | 保留修改/访问时间和权限              |
| `scp -C`                     | 启用传输压缩                         |
| `scp -q` / `-v`              | 安静模式 / 详细调试模式              |
| `scp -4` / `-6`              | 只使用 IPv4 / IPv6                   |
| `scp -B`                     | 批处理模式，不交互提示输入密码或口令 |
| `scp -c <算法>`              | 指定 SSH 加密算法                    |
| `scp -F <配置>`              | 指定 SSH 客户端配置文件              |
| `scp -J <跳板机>`            | 通过 SSH 跳板机连接                  |
| `scp -3`                     | 两个远程端之间通过本地中转           |
| `scp -l <Kbit/s>`            | 限制传输速率，单位 Kbit/s            |
| `scp -o <SSH选项>`           | 传递 SSH 客户端选项                  |
| `scp -S <程序>`              | 指定用于连接的 SSH 程序              |

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519 alice@host
# 用指定私钥和端口登录远程主机
ssh alice@host 'df -h'
# 在远程主机执行 df -h；单引号保护命令不被本地 shell 展开
scp file.txt alice@host:/tmp/
# 将本地文件复制到远程目录
scp -r project/ alice@host:/var/www/
# 递归复制本地目录及内容到远程
scp -P 2222 -i ~/.ssh/id_ed25519 file.txt alice@host:/tmp/
# 使用大写 -P 指定 SCP 端口，并用 -i 指定私钥
scp -J jumpuser@jumphost alice@target:/var/log/app.log ./
# 通过跳板机从目标主机拉取日志到当前目录
scp -3 alice@host1:/tmp/file bob@host2:/tmp/
# 两端之间复制并通过本机中转
```

一次性、小规模传输可用 SCP；大目录、增量同步和备份优先考虑 rsync。

### 8.2 curl（HTTP/API/下载）

`curl [选项] <URL>`。下列参数说明同时适用于常见 Linux 发行版，细节以本机 `curl --help all` 为准。

| 参数                                    | 功能                                                    |
| --------------------------------------- | ------------------------------------------------------- |
| `-X, --request <方法>`                  | 指定 HTTP 方法；多数场景 `-d` 已隐含 POST，不必重复指定 |
| `-H, --header '<头: 值>'`               | 添加 HTTP 请求头                                        |
| `-d, --data <数据>`                     | 发送请求数据，默认表单编码；常用于 POST                 |
| `--data-urlencode <数据>`               | 对提交数据进行 URL 编码                                 |
| `-F, --form <字段=值>`                  | 发送 multipart/form-data；`字段=@文件` 表示上传文件     |
| `-I, --head`                            | 只请求响应头（HEAD 请求）                               |
| `-i, --include`                         | 输出中包含响应头和响应体                                |
| `-o <文件>`                             | 将响应保存到指定文件                                    |
| `-O, --remote-name`                     | 用 URL 远程文件名保存响应                               |
| `-L, --location`                        | 跟随 HTTP 重定向                                        |
| `-f, --fail`                            | HTTP 错误状态时以失败退出，避免把错误页当成功内容       |
| `-s, --silent`                          | 隐藏进度和常规错误输出                                  |
| `-S, --show-error`                      | 与 `-s` 搭配时仍显示错误信息                            |
| `-v, --verbose`                         | 输出请求/响应调试细节，可能包含敏感头信息               |
| `-w, --write-out '<格式>'`              | 请求结束后输出状态码、耗时等变量                        |
| `-D <文件>`                             | 保存响应头到文件                                        |
| `-u <用户:密码>`                        | HTTP Basic 等认证信息；避免在 shell 历史中写真实密码    |
| `-b <Cookie>` / `-c <文件>`             | 发送 Cookie / 将 Cookie 保存到文件                      |
| `-x, --proxy <地址>`                    | 指定代理服务器                                          |
| `--socks5 <主机:端口>`                  | 指定 SOCKS5 代理                                        |
| `--connect-timeout <秒>`                | 限制连接阶段超时                                        |
| `--max-time <秒>`                       | 限制整个请求总时长                                      |
| `-k, --insecure`                        | 不校验证书；仅临时排障，不能作为生产安全设置            |
| `--cacert <文件>`                       | 指定受信任 CA 证书文件                                  |
| `--cert <文件>` / `--key <文件>`        | 指定客户端证书和私钥                                    |
| `--resolve <主机:端口:IP>`              | 对单次请求指定主机解析地址，仍使用原主机名进行 TLS/HTTP |
| `-e, --referer <URL>`                   | 设置 Referer 请求头                                     |
| `-A, --user-agent <值>`                 | 设置 User-Agent 请求头                                  |
| `-C -`                                  | 从已存在文件大小处继续下载                              |
| `-r, --range <范围>`                    | 请求指定字节范围                                        |
| `--limit-rate <速率>`                   | 限速，如 `500k`                                         |
| `-T <文件>`                             | 上传文件                                                |
| `--retry <次数>` / `--retry-delay <秒>` | 设置失败重试次数 / 重试间隔                             |
| `--compressed`                          | 声明可接受压缩响应并自动解压                            |
| `--http1.1` / `--http2`                 | 指定 HTTP 协议版本（受构建能力影响）                    |
| `--interface <接口>`                    | 指定出站网络接口                                        |

```bash
curl -fSL -o file.zip 'https://example.com/file.zip'
# -f HTTP 错误返回失败；-s 静默；-S 显示错误；-L 跟随重定向；-o 指定保存文件
curl -sS -I https://example.com
# 安静显示错误并请求响应头；-I 发起 HEAD 请求
curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' https://example.com
# 丢弃响应体到 /dev/null，并输出 HTTP 状态码和总耗时
curl -X POST https://api.example.com/users -H 'Content-Type: application/json' -d '{"name":"alice"}'
# 发送 JSON POST；-H 设置内容类型，-d 提供请求体
curl -F 'file=@/path/to/file' -F 'name=test' https://api.example.com/upload
# 用 multipart 表单上传文件及普通字段
curl -C - -O https://example.com/big.iso
# 从本地已有文件大小处续传；-O 使用远端文件名
curl --retry 3 --retry-delay 2 --max-time 30 https://api.example.com
# 最多重试 3 次、重试间隔 2 秒、整个请求最多 30 秒
```

### 8.3 tar 归档压缩

| 参数               | 含义                             |
| ------------------ | -------------------------------- |
| `-c` / `-x` / `-t` | 创建归档 / 解包 / 列出归档内容   |
| `-z` / `-j` / `-J` | gzip / bzip2 / xz 压缩格式       |
| `-v`               | 显示处理的文件名                 |
| `-f <归档文件>`    | 指定归档文件名；通常放在选项末尾 |
| `-C <目录>`        | 执行后续文件路径前切换目录       |

```bash
tar -czvf backup.tar.gz <目录>
# 创建 gzip 压缩 tar；-c 创建、-z gzip、-v 输出文件、-f 指定归档名
tar -xzvf backup.tar.gz -C <目标目录>
# 解压 gzip tar 到目标目录；-x 解包、-z gzip、-v 显示文件、-C 切换目标目录
tar -tzf backup.tar.gz
# 列出 gzip tar 内容但不解压；-t 列表、-z gzip、-f 指定文件
tar -cJvf backup.tar.xz <目录>
# 创建 xz 压缩 tar；-J 代表 xz
```

<a id="sec-rsync"></a>

## 9. rsync 增量同步与备份

语法：`rsync [选项] 源 目标`。远程路径可写为 `user@host:/path`；SSH 端口可通过 `-e 'ssh -p <端口>'` 指定。

| 参数                                                   | 功能                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `-a, --archive`                                        | 归档模式，等同常见的 `-rlptgoD`：递归、保留链接/权限/时间/属主/属组/设备文件；非 root 无法完整保留部分属主信息 |
| `-r` / `-l` / `-p` / `-t`                              | 递归目录 / 保留符号链接 / 保留权限 / 保留修改时间            |
| `-g` / `-o` / `-D`                                     | 保留属组 / 属主 / 设备和特殊文件                             |
| `-v` / `-q`                                            | 详细输出 / 安静模式                                          |
| `-z`                                                   | 传输压缩；CPU 紧张或高速 LAN 可评估是否需要                  |
| `-P`                                                   | 等同 `--partial --progress`：保留部分文件并显示进度          |
| `--partial` / `--progress`                             | 保留未完成文件 / 显示传输进度                                |
| `-h, --human-readable`                                 | 使用易读单位显示大小                                         |
| `-n, --dry-run`                                        | 只模拟，不实际更改目标                                       |
| `--bwlimit=<KB/s>`                                     | 限制带宽，单位通常为 KB/s                                    |
| `-e '<远程 shell>'`                                    | 指定传输 shell；例如 `-e 'ssh -p 2222'`                      |
| `--delete`                                             | 删除目标中源端不存在的文件；有误删风险                       |
| `--delete-excluded`                                    | 也删除目标中被排除规则排除的文件                             |
| `--exclude=<模式>` / `--include=<模式>`                | 排除 / 包含匹配路径；规则次序会影响匹配                      |
| `--exclude-from=<文件>`                                | 从文件读取排除规则                                           |
| `--existing` / `--ignore-existing`                     | 只更新目标已存在文件 / 跳过所有目标已存在文件                |
| `-u, --update`                                         | 跳过目标端比源端新的文件                                     |
| `--backup` / `--backup-dir=<目录>` / `--suffix=<后缀>` | 覆盖前备份 / 指定旧版本存放目录 / 指定备份文件后缀           |
| `--remove-source-files`                                | 传输成功后删除源文件；不等同于安全的双向同步                 |
| `--size-only` / `-c, --checksum`                       | 仅按大小判断差异 / 按校验和判断（更准确但耗时）              |
| `-L, --copy-links`                                     | 跟随符号链接并复制其指向内容                                 |
| `--copy-unsafe-links`                                  | 只复制指向源目录树外的符号链接目标                           |
| `-K, --keep-dirlinks`                                  | 将目标端目录符号链接按目录处理                               |
| `--hard-links` / `-A` / `-X`                           | 保留硬链接 / ACL / 扩展属性                                  |
| `--numeric-ids`                                        | 以数字 UID/GID 保留属主属组，不按用户名映射                  |
| `--chown=<用户>:<组>` / `--chmod=<模式>`               | 设置目标端属主属组 / 权限模式                                |
| `--max-size=<大小>` / `--min-size=<大小>`              | 限定传输文件最大 / 最小大小                                  |
| `--files-from=<文件>`                                  | 从清单读取待同步路径                                         |
| `--log-file=<文件>` / `--stats`                        | 写同步日志 / 输出统计信息                                    |

```bash
rsync -avh --dry-run --delete /data/source/ /backup/target/
# [只读] -a 归档、-v 详细、-h 易读；--dry-run 仅模拟；--delete 会标记目标多余文件作为待删项
rsync -avh --delete /data/source/ /backup/target/
# [高风险] 执行镜像同步并删除目标多余文件；先核实源/目标、规则和 dry-run 输出，保留目标备份
rsync -avhz --progress -e 'ssh -p 2222' --exclude='.git' /var/www/ alice@host:/var/www/
# 通过 SSH 2222 端口同步；-z 压缩、--progress 显示进度、--exclude 排除 .git
rsync -avhP --bwlimit=5000 /data/big.iso alice@host:/backup/
# 传输大文件并显示进度、保留部分文件；带宽限制约 5000 KB/s
rsync -avhu --ignore-existing /data/source/ /backup/target/
# -u 跳过目标比源新的文件；--ignore-existing 则跳过目标已有文件
rsync -avh --backup --backup-dir=/backup/old --suffix=".$(date +%Y%m%d)" /data/ /backup/current/
# 覆盖前把旧文件备份到指定目录，并用日期作为后缀
rsync -avhAX --numeric-ids /data/ /backup/
# 除归档模式外保留 ACL、扩展属性，并按数字 UID/GID 保留属主关系
rsync -avh --stats /data/ /backup/
# 执行同步并输出文件数、传输量等统计
```

**路径末尾斜杠**：`/source/` 表示复制目录内容；`/source` 表示把目录本身放到目标下。包含 `--delete` 的命令务必先 `--dry-run`，确认源、目标和排除规则正确。

<a id="sec-scheduling"></a>

## 10. cron 定时任务

cron 五字段格式：`分 时 日 月 星期 命令`。分钟 `0–59`，小时 `0–23`，日 `1–31`，月 `1–12`，星期 `0–7`（多数 cron 中 0 和 7 都代表周日）。

| 写法                                                      | 含义                                       |
| --------------------------------------------------------- | ------------------------------------------ |
| `*`                                                       | 任意合法值                                 |
| `,`                                                       | 多个离散值，如 `1,15`                      |
| `-`                                                       | 连续范围，如 `1-5`                         |
| `/`                                                       | 步长，如 `*/5` 表示每 5 个单位             |
| `@reboot`                                                 | cron 启动时运行                            |
| `@hourly` / `@daily` / `@weekly` / `@monthly` / `@yearly` | 每小时 / 每天 / 每周 / 每月 / 每年快捷写法 |

```bash
crontab -l                           # 列出当前用户的 cron 任务
crontab -e                           # 编辑当前用户的 cron 任务
crontab -r                           # 删除当前用户全部 cron 任务；执行前备份
sudo crontab -u <用户> -l             # 以管理员权限列出指定用户的任务
sudo crontab -u <用户> -e             # 以管理员权限编辑指定用户的任务
crontab -l > my-crontab.txt           # 将当前用户任务导出到文件；> 会覆盖文件
crontab my-crontab.txt                # 从文件安装当前用户 crontab
```

`/etc/crontab` 和 `/etc/cron.d/` 是系统级任务；系统级 crontab 通常比用户 crontab 多一个“运行用户”字段。`/etc/cron.daily/`、`hourly/`、`weekly/`、`monthly/` 是按周期运行的脚本目录，具体调度取决于发行版。

cron 环境不同于交互式 shell：设置绝对路径和环境变量；脚本有 shebang 和执行权限；用 `>>日志 2>&1` 同时追加标准输出及错误。crontab 中 `%` 有特殊意义，写入日期格式时通常要转义为 `\%`。

```cron
0 2 * * * /usr/bin/rsync -avh /data/ /backup/data/ >> /var/log/rsync.log 2>&1
# 每天 02:00 同步；分=0、时=2、其余日期字段为任意；日志追加 stdout 和 stderr
*/5 * * * 1-5 /home/alice/sync.sh >> /var/log/sync.log 2>&1
# 周一至周五每 5 分钟执行；*/5 表示分钟步长；1-5 表示周一到周五
@reboot /home/alice/start-services.sh >> /home/alice/startup.log 2>&1
# cron 启动时执行脚本，并把输出追加到日志
0 4 * * 0 /usr/bin/backup-db.sh
# 每周日凌晨 04:00 执行数据库备份脚本
```

复杂任务应放到脚本中，增加日志、错误处理和锁，防止任务重叠；涉及密码时使用受保护的凭据机制，不要把明文密码写进 crontab。服务器升级或备份任务要验证产物可恢复。

```bash
sudo systemctl status cron              # Debian/Ubuntu 常见 cron 服务名
sudo systemctl status crond             # Fedora/RHEL 常见 cron 服务名
sudo journalctl -u cron --since today   # 查看 Debian/Ubuntu cron 单元日志
sudo journalctl -u crond --since today  # 查看 Fedora/RHEL crond 单元日志
sudo flock -n /run/lock/myjob.lock /path/to/job.sh
# flock -n：锁被占用时不等待，直接退出；避免同一任务并行重入
```

### systemd timer 与 cron 如何选择

| 维度         | systemd timer                                                | cron                                             |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| 适用场景     | systemd 主机上的服务维护、开机补跑、依赖服务/用户/资源限制的任务 | 简单周期任务、跨多种 Unix 环境的传统脚本         |
| 日志与状态   | 任务作为 service 运行，可用 `systemctl` 和 `journalctl` 查看状态/日志 | 通常需自行重定向日志；任务状态需由脚本或监控记录 |
| 关机错过执行 | `Persistent=true` 可在系统恢复后补跑错过的日历任务（不适用于所有 timer 类型） | 通常不会自动补跑，取决于发行版及额外工具         |
| 配置复杂度   | 需维护匹配名称的 `.service` 和 `.timer` 两个单元             | crontab 一行即可                                 |

systemd timer 示例：将以下两个文件保存到 `/etc/systemd/system/`。服务单元是任务本身，timer 单元负责调度；单元名需对应为 `my-backup.service` 和 `my-backup.timer`。

```ini
# /etc/systemd/system/my-backup.service
[Unit]
Description=Run application backup
# 描述该一次性任务，显示于 systemctl 状态中

[Service]
Type=oneshot
# oneshot 表示执行完命令后服务即结束，不作为常驻进程
User=backup
# 以专用低权限账户运行；该账户须有源数据读取和目标目录写入权限
ExecStart=/usr/local/sbin/backup.sh
# 任务执行程序；使用绝对路径，脚本应有正确属主和执行权限
```

```ini
# /etc/systemd/system/my-backup.timer
[Unit]
Description=Daily application backup timer
# 定时器的可读说明

[Timer]
OnCalendar=*-*-* 02:00:00
# 每天本地时间 02:00 触发；可用 systemd-analyze calendar 验证表达式
Persistent=true
# 主机关闭时错过触发点，则在 timer 恢复后补执行一次
RandomizedDelaySec=10m
# 在 0 到 10 分钟范围内加入随机延迟，分散多台主机的同时任务
Unit=my-backup.service
# 指定触发的服务单元；未设置时通常使用同名 .service

[Install]
WantedBy=timers.target
# enable 时将定时器挂接到系统 timer 目标，使其开机启动
```

```bash
systemd-analyze calendar '*-*-* 02:00:00'
# [只读] 检查 OnCalendar 表达式并显示下次触发时间
sudo systemd-analyze verify /etc/systemd/system/my-backup.service /etc/systemd/system/my-backup.timer
# [只读] 检查两个单元文件语法和依赖问题
sudo systemctl daemon-reload
# [会修改] 让 systemd 重新读取新增或变更的单元文件
sudo systemctl start my-backup.service
# [会修改] 立即手动运行备份服务，先验证脚本结果和权限
sudo systemctl enable --now my-backup.timer
# [会修改] 设置 timer 开机启用并立即启动调度
systemctl list-timers --all
# [只读] 列出 timer 单元、上次及下次触发时间
systemctl status my-backup.timer my-backup.service
# [只读] 查看定时器和最近一次服务执行结果
journalctl -u my-backup.service --since today --no-pager
# [只读] 查看今天该任务的 systemd 日志
sudo systemctl disable --now my-backup.timer
# [会修改] 停止并取消定时器开机启用；service 文件仍保留
```

生产中停用 timer 通常比删除单元文件更安全；修改后先手动启动一次验证。`OnCalendar` 按日历时间运行；`OnBootSec`、`OnUnitActiveSec` 更适合开机后/距离上次运行间隔调度。选 cron 的环境应额外设置绝对路径、日志、锁和失败告警。

<a id="sec-arp-scan"></a>

## 11. arp-scan 局域网排查

仅在自己管理或获授权的局域网中使用。`arp-scan` 通过 ARP 发现本地二层网络中的主机，不是跨路由网段扫描工具；网卡名和目标网段需替换为实际值。

```bash
sudo arp-scan -l                      # 扫描本机所连接的本地网络
sudo arp-scan --localnet              # 与 -l 类似，按接口配置扫描本地网络
sudo arp-scan 192.168.1.0/24          # 扫描指定 IPv4 CIDR 网段
sudo arp-scan 192.168.1.1-192.168.1.100 # 扫描指定 IPv4 地址范围
sudo arp-scan -I eth0 -l              # -I 指定网卡接口后扫描本地网络
sudo arp-scan -l -q                   # 安静输出，减少厂商解析信息
sudo arp-scan -l --plain              # 去掉页眉/页脚，便于脚本解析
sudo arp-scan -l --ignoredups         # 忽略重复 ARP 响应
sudo arp-scan -l -v                   # 输出更详细的扫描过程
sudo arp-scan --retry=3 --timeout=500 192.168.1.0/24
# 每目标重试 3 次；单次等待超时 500 毫秒；按需调整避免漏报
sudo arp-scan --localnet --plain --format='${ip}\t${mac}'
# 输出制表符分隔的 IP 和 MAC；${ip}/${mac} 是 arp-scan 的格式字段
```

`--srcaddr=MAC` 可指定源 MAC，`--file=<文件>` 可从文件读取目标列表；这些选项应仅用于授权的网络诊断和兼容性测试，不能绕过网络访问控制。

<a id="sec-upgrades"></a>

## 12. 版本升级、开发工具与桌面软件

### 12.1 大版本升级

**升级前必须备份数据和配置，阅读目标版本说明，更新到当前版本最新补丁，并在测试环境验证。** 大版本迁移不是普通软件更新。

```bash
sudo do-release-upgrade                 # Ubuntu 的交互式发行版升级向导
sudo dnf install dnf-plugin-system-upgrade # 安装 Fedora 系统升级插件（版本可能已变化）
sudo dnf system-upgrade download --releasever=<目标版本>
# 下载并准备 Fedora 目标版本升级包；--releasever 指定目标版本
sudo dnf system-upgrade reboot          # 重启进入 Fedora 离线升级流程
sudo leapp preupgrade --target <目标版本>
# RHEL 原地升级预检查；先解决报告中的 inhibitor
sudo leapp upgrade --target <目标版本>
# RHEL 原地升级；仅在完成预检、备份和官方兼容性确认后执行
```

Debian 大版本升级需按官方升级说明修改仓库发行版代号，再分阶段更新；不要用未经核实的全局文本替换脚本直接改生产系统仓库。

### 12.2 常用开发工具与容器

| 需求              | Debian / Ubuntu                                     | Fedora / RHEL                                                |
| ----------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| C/C++ 工具链      | `sudo apt install build-essential`                  | `sudo dnf group install "Development Tools"`                 |
| 当前内核头文件    | `sudo apt install linux-headers-$(uname -r)`        | `sudo dnf install kernel-devel kernel-headers`               |
| Git               | `sudo apt install git`                              | `sudo dnf install git`                                       |
| Python 与 venv    | `sudo apt install python3 python3-pip python3-venv` | `sudo dnf install python3 python3-pip`                       |
| OpenJDK 17 开发包 | `sudo apt install openjdk-17-jdk`                   | `sudo dnf install java-17-openjdk-devel`                     |
| PHP               | `sudo apt install php-cli php-fpm php-mysql`        | `sudo dnf install php-cli php-fpm php-mysqlnd`               |
| Docker / Podman   | 按官方仓库安装，注意守护进程权限                    | RHEL 常推荐 Podman；Fedora/RHEL 上的 Docker 支持和仓库需核对官方说明 |

Node.js、Rust、Go 等若需特定版本，优先采用项目维护的版本管理器或官方安装说明；系统仓库版本可能较旧。不要把 `docker` 用户组当作低权限组，它通常可获得近似 root 的主机访问权限。

跨发行版桌面应用可考虑 Flatpak（若应用和发行版支持）；Snap、AppImage 和语言生态包管理器也可用，但它们与系统包管理器的更新、沙箱及权限模型不同。

### 12.3 Docker / Podman 基础操作

Docker 官方仓库和安装步骤会随发行版版本变化。生产环境按 Docker 官方针对当前系统的安装说明配置仓库；以下展示常见仓库包或 RHEL 推荐的 Podman 用法。

```bash
sudo apt install docker.io                 # 从 Debian 系配置的软件源安装 Docker Engine
sudo systemctl enable --now docker         # 设置 Docker 开机启动并立即启动守护进程
sudo dnf install podman                    # 在 Fedora/RHEL 安装无守护进程的 Podman
podman run --rm hello-world                # 运行测试容器；--rm 在退出后删除该容器
docker run --rm hello-world                # Docker 环境下运行相同的测试容器
docker compose version                     # 查看 Docker Compose 插件是否可用及版本
```

Docker 与 Podman 的常用容器命令大多兼容，但网络、卷、守护进程和 Compose 行为仍需按工具文档确认。将用户加入 `docker` 组可绕过 sudo，但该组通常具有近似 root 的主机控制权；只在理解风险并确有需要时执行 `sudo usermod -aG docker <用户>`（`-aG` 追加附加组），然后重新登录。

<a id="sec-troubleshooting"></a>

## 13. 常见故障快速定位

故障处理按“确认现象 → 查状态/日志 → 定位到配置或资源 → 做最小修复 → 复测”进行。先收集错误信息和当前配置；不要为了让错误暂时消失而直接关闭防火墙、SELinux/AppArmor 或修改全盘权限。

### 磁盘满、inode 耗尽或日志占用过大

```bash
df -hT                                  # 查看各文件系统容量和类型，定位 Use% 接近 100% 的挂载点
df -ih                                  # 查看 inode 使用率；IUse% 满时即使尚有空间也无法创建新文件
sudo du -xhd1 /var                      # 汇总 /var 下一层目录大小；-x 不跨文件系统，-h 易读，-d1 限一层
sudo journalctl --disk-usage            # 查看 systemd journal 当前占用量
sudo journalctl --vacuum-time=14d       # 清除超过 14 天的归档 journal；先确认审计/保留要求
sudo apt clean                          # Debian 系清除已下载软件包缓存，不卸载已安装软件
sudo dnf clean packages                 # Fedora/RHEL 清除已下载 RPM 包缓存，不卸载已安装软件
```

先查出占用最大的挂载点和目录，再按保留策略清理缓存、过期日志或已确认可删除的文件；不要直接清空 `/var/log` 或删除仍被进程打开的文件。inode 耗尽时，重点检查包含大量小文件的缓存、会话、临时目录，并先确认应用是否仍在使用。

### 磁盘、文件系统和挂载失败

先判断问题属于设备未发现、文件系统未挂载、`/etc/fstab` 配置错误、空间/inode 耗尽，还是块设备 I/O/文件系统损坏。不要对已挂载且正在使用的文件系统直接运行修复型 `fsck`；先安排维护窗口、卸载文件系统或从救援环境启动。

```bash
lsblk -f
# [只读] 列出块设备、文件系统类型、UUID、标签和挂载点
blkid
# [只读] 查询块设备的文件系统 UUID、类型和标签；比猜测 /dev/sdX 稳妥
findmnt --real
# [只读] 显示当前真实挂载点；不支持此参数时用 findmnt -a 或 findmnt <路径>
findmnt --verify --verbose
# [只读] 检查 /etc/fstab 配置；若当前 util-linux 版本不支持相关选项，查 findmnt --help
sudo mount -av
# [会修改] 按 fstab 尝试挂载尚未挂载的条目；-a 全部、-v 输出详情
# 执行前先备份并检查 fstab；远程根盘/关键挂载配置错误可能影响启动
sudo journalctl -k -b --no-pager | grep -iE 'I/O error|filesystem error|EXT4-fs|XFS|nvme|ata'
# [只读] 搜索本次启动内核日志中的设备 I/O 和常见文件系统错误
sudo smartctl -H /dev/sdX
# [只读] 检查 SATA/SCSI 设备 SMART 健康概况；替换为 lsblk 确认的实际设备
sudo nvme smart-log /dev/nvme0
# [只读] 查询 NVMe 健康日志；设备路径依实际硬件而定
```

设备或挂载点未出现时，检查虚拟机/云盘是否已附加、内核是否识别，以及 `dmesg`/journal 的设备错误。`/etc/fstab` 启动失败时，从控制台或救援模式检查 UUID（以 `blkid` 输出为准）、挂载点目录和文件系统类型，修正后先执行 `findmnt --verify`，再谨慎用 `mount -av` 验证。若根文件系统因 fstab 错误进入 emergency mode，可先以可写方式挂载根分区、备份并修复 fstab；避免在不清楚启动环境时盲目重启。

如果需要修复文件系统，先备份/制作块设备镜像（可行时），确认设备路径，并确保目标文件系统已卸载；根分区通常需救援介质或 initramfs 环境。按文件系统类型使用对应工具（如 ext 系列 `fsck`、XFS `xfs_repair`），遵循该工具的专用恢复流程；不要对整块磁盘分区表设备或错误分区执行修复。

### DNS、网络或服务端口不可达

```bash
ip addr                                # 检查网卡是否启用以及是否获得预期 IP 地址
ip route                               # 检查默认路由及目标网段路由是否存在
getent hosts <域名>                    # 通过系统 NSS/DNS 配置查询域名；无结果时继续查 DNS 设置
resolvectl status                      # 查看 systemd-resolved 使用的 DNS 服务器和链路设置
ping -c 4 -W 2 <IP或主机名>             # 发送 4 个 ICMP 请求；-W 2 设置单次等待约 2 秒，目标可禁 ping
curl -v --connect-timeout 5 --max-time 15 https://<主机名>/
# -v 显示连接细节；连接阶段最多 5 秒；整个 HTTP 请求最多 15 秒；用于区分 DNS/TCP/TLS/HTTP 故障
ss -tlnp                              # 查看本机监听 TCP 端口及进程；-t TCP、-l 监听、-n 数字、-p 进程
sudo firewall-cmd --list-all           # Fedora/RHEL：查看当前 firewalld zone 的运行时放行规则
sudo ufw status numbered               # Debian/Ubuntu：查看 UFW 状态及带编号规则
```

逐层区分“路由、DNS、TCP 端口、TLS 证书和应用响应”，不要只依赖 ping：

```bash
ip route get <目标IP>
# [只读] 查询到目标 IP 实际会使用的路由、网关和出接口
dig +time=2 +tries=1 <域名> A
# [只读] 查询 IPv4 A 记录；+time 限制单次等待 2 秒，+tries 限制查询 1 次
getent ahosts <域名>
# [只读] 通过系统 NSS 配置查询地址，结果更接近应用程序实际解析路径
nc -vz -w 3 <主机> <端口>
# [只读] -v 显示详细结果、-z 仅探测不发送应用数据、-w 3 设置超时 3 秒
# 不同 netcat 实现的参数可能不同；先用 nc -h 检查本机版本
curl -sS -o /dev/null --connect-timeout 5 --max-time 15 \
  -w 'HTTP=%{http_code} DNS=%{time_namelookup}s TCP=%{time_connect}s TLS=%{time_appconnect}s\n' \
  https://<主机名>/
# [只读] 不显示响应体但显示错误；-w 输出 HTTP 状态、DNS/TCP/TLS 阶段耗时
# 连接超时 5 秒、整个请求最多 15 秒；TLS 阶段耗时为 0 时检查握手/协议情况
tracepath -n <目标IP>
# [只读] 探测路径 MTU 和经过的路由节点；-n 不做反向 DNS 查询
sudo tcpdump -ni <接口> 'host <目标IP> and port <端口>'
# [只读但敏感] -n 不解析名称/端口、-i 选接口；抓包可能含敏感数据，只在授权下短时使用
```

若 `dig` 可解析而 `getent` 不行，检查 NSS/systemd-resolved 配置；DNS 正常但 `nc` 失败则检查路由、目标端口监听和防火墙；TCP 成功而 `curl` 报 TLS 错误则检查证书链、系统时间和主机名；HTTP 返回 4xx/5xx 通常已到达应用层，应继续看服务端日志。安装工具时，Debian/Ubuntu 常用 `dnsutils`、`netcat-openbsd`、`iputils-tracepath`、`tcpdump`；Fedora/RHEL 常用 `bind-utils`、`nmap-ncat`、`iputils`、`tcpdump`。

先确认本机接口/IP/路由，再用 `getent` 判断 DNS；用 `curl` 或对应客户端区分“名字解析失败”“TCP 连接失败”“TLS 证书错误”和“应用返回 HTTP 错误”。修正错误的 DNS、网关、服务监听地址或防火墙规则后复测；不要因 ping 无响应就认定主机不可达，也不要在不了解现有规则时清空防火墙。

若确认 NetworkManager 连接拿到了错误的 DHCP DNS，可在替换连接名和 DNS 地址后指定静态 DNS；这会忽略 DHCP 下发的 DNS，应用时可能短暂断网：

```bash
sudo nmcli connection modify "<连接名>" ipv4.ignore-auto-dns yes ipv4.dns "<DNS1>,<DNS2>"
# 修改连接配置；ignore-auto-dns yes 忽略 DHCP DNS；ipv4.dns 设置逗号分隔的 DNS 地址
sudo nmcli connection up "<连接名>"
# 重新激活连接应用 DNS 配置；远程设备执行前准备回滚/控制台访问
getent hosts <域名>
# 重新查询域名以验证系统解析链路是否恢复
```

### 内存不足、进程异常或系统变慢

```bash
free -h                                # 查看总内存、已用、缓存和 available；优先关注 available
ps -eo pid,user,comm,%mem,%cpu --sort=-%mem | head -n 15
# -e 选全部进程；-o 指定列；--sort=-%mem 按内存占用降序；head -n 15 只看前 15 行
ps -eo pid,user,comm,%mem,%cpu --sort=-%cpu | head -n 15
# 按 CPU 占用降序列出前 15 行，定位持续占用 CPU 的进程
vmstat 1 5                             # 每秒采样一次共 5 次，查看运行队列、交换和 CPU 统计
sudo journalctl -k -b --no-pager | grep -i -E 'out of memory|oom|killed process'
# 查看本次启动内核日志中 OOM 相关记录；-k 内核、-b 本次启动、-E 扩展正则
```

确认进程和日志后，优先修复应用泄漏、并发量或资源限制；只有确定进程可安全终止时才发信号。`kill -15 <PID>` 请求程序正常退出；只有无响应且已评估数据风险时才使用 `kill -9 <PID>` 强制终止。

### SSH 登录失败

```bash
sudo sshd -t                           # 检查 SSH 服务端配置语法；失败时根据报错先修配置
sudo systemctl status ssh              # Debian/Ubuntu：查看 ssh 服务是否运行及最近状态
sudo systemctl status sshd             # Fedora/RHEL：查看 sshd 服务是否运行及最近状态
sudo journalctl -u ssh -b --no-pager -n 100
# Debian/Ubuntu：查看本次启动 ssh 最近 100 条日志，不进入分页器
sudo journalctl -u sshd -b --no-pager -n 100
# Fedora/RHEL：查看本次启动 sshd 最近 100 条日志，不进入分页器
sudo ss -tlnp | grep ':22'             # 检查 TCP 22 是否监听；若 SSH 改端口则换成实际端口
sudo namei -l /home/<用户>/.ssh/authorized_keys
# 逐级检查家目录、.ssh 和 authorized_keys 的属主及遍历权限
```

对照客户端实际端口/用户名/密钥，检查 `sshd_config`、服务日志、监听地址和防火墙；公钥登录通常要求用户家目录及 `.ssh/authorized_keys` 不允许不可信用户写入。按目标发行版恢复正确的属主和最小权限，不要把密钥文件设为全员可读写。改配置后先 `sshd -t` 验证，再重载/重启对应单元，并保留现有会话直到新连接成功。

### sudo 提示用户不在 sudoers 或文件 Permission denied

```bash
id <用户>                                # 查看用户当前 UID、主组和附加组
getent group sudo                       # Debian 系：查询 sudo 组是否存在及其成员
getent group wheel                      # Fedora/RHEL：查询 wheel 组是否存在及其成员
sudo -l                                 # 在有 sudo 权限的账号下检查当前用户允许执行的命令
sudo visudo -c                          # 检查 sudoers 主文件及 include 文件语法，不进入编辑器
sudo usermod -aG sudo <用户>            # Debian 系：由 root/管理员把用户追加到 sudo 组
sudo usermod -aG wheel <用户>           # Fedora/RHEL：由 root/管理员把用户追加到 wheel 组
namei -l <目标路径>                     # 逐级检查路径每一层的属主和权限，定位缺少遍历权限的目录
stat <目标文件>                          # 查看目标文件的属主、组、权限位和元数据
getfacl <目标文件>                       # 检查 ACL 是否额外拒绝或覆盖传统权限判断
sudo chown <用户>:<组> <目标文件>       # 仅在确认属主/属组错误时修正单个文件
sudo chmod <最小必要权限> <目标文件>     # 仅在确认权限位错误时修正；将占位符替换为合适权限如 640
```

用户组变更后必须注销并重新登录，当前 shell 不会自动获得新组；若当前用户已失去 sudo 权限，需由 root、仍有管理员权限的账号或主机控制台修复。不要通过给文件 `777` 或授予所有用户 sudo 来绕过错误。

### 服务启动失败

```bash
sudo systemctl status <服务>                 # 查看单元状态、退出码和日志摘要
sudo journalctl -u <服务> -b --no-pager -n 100
# 查看本次启动该服务最近 100 行日志；不分页
sudo ss -tlnp | grep ':<端口>'               # 检查 TCP 监听端口及进程；-t TCP、-l 监听、-n 数字、-p 进程
sudo ufw status                              # Debian 系：查看 UFW 状态和规则
sudo firewall-cmd --list-all                 # Fedora/RHEL：查看 firewalld 当前 zone 和运行时规则
sudo ausearch -m AVC -ts recent              # 查询近期 SELinux AVC 拒绝；-m 按消息类型、-ts 指定起始时间
sudo aa-status                               # 查看 AppArmor profile 是否加载及其强制/抱怨模式
```

按顺序排查：服务状态和日志 → 配置语法（`nginx -t`：测试 Nginx 配置并退出；`apachectl configtest`：检查 Apache 配置语法）→ 端口占用 → 防火墙 → AppArmor/SELinux → 文件属主及权限。若配置语法错误，按报错修正配置后重新验证，再启动服务。只在隔离、短时诊断时切换安全策略模式，验证后立即恢复，并修复具体策略。

### 包管理故障

| 现象                | Debian / Ubuntu                                              | Fedora / RHEL                                                |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 锁被占用            | 先确认是否有 apt/dpkg 进程正在工作；不要直接删除锁文件       | 等待 DNF 进程结束并确认无其他包操作                          |
| 依赖损坏            | `sudo dpkg --configure -a` 后按提示执行 `sudo apt --fix-broken install` | 查看 DNF 事务报错；确认仓库后尝试 `sudo dnf distro-sync`，先审查删除/降级计划 |
| 仓库 404 / 包找不到 | 检查仓库 URL、发行版代号和架构，再 `apt update`              | 检查 `.repo` 的 `$releasever`、仓库启用状态和订阅            |
| 签名错误            | 确认仓库密钥来源、有效期及 `signed-by` 配置；不要关闭签名验证 | 确认 RPM GPG key 和 `gpgcheck` 配置；不要跳过签名校验        |

只有明确确认包管理器已崩溃且没有任何相关进程后，才进一步处理锁文件或数据库；直接删除锁文件可能造成数据库损坏。

```bash
pgrep -a apt                           # 查找正在运行的 apt 进程；-a 同时显示命令行
pgrep -a dpkg                          # 查找正在运行的 dpkg 进程；不要在事务运行时清理锁或中断进程
pgrep -a dnf                           # 查找正在运行的 DNF 进程；若有更新/安装事务，等待其正常完成
sudo dpkg --configure -a               # Debian 系：完成已解包但尚未配置的包；先确保没有其他 dpkg 事务
sudo apt-get -s -f install             # 模拟修复依赖计划；-s 只模拟，-f 请求修复 broken dependencies
sudo apt --fix-broken install          # Debian 系：按 APT 计划修复未满足的依赖；执行前审查将安装/移除的包
sudo dnf check                         # Fedora/RHEL：检查已安装 RPM 数据库中的依赖问题
sudo dnf distro-sync --assumeno        # 预览与仓库版本对齐的变更；--assumeno 对确认提示默认回答否
sudo dnf distro-sync                   # 执行版本对齐；可能升级、降级或移除包，先检查仓库及事务计划
```

不要为了“解锁”而直接删除 `/var/lib/dpkg/lock*`、DNF PID 文件或包管理数据库文件。若确认系统崩溃中断且没有相关进程，先按发行版官方恢复流程检查数据库和文件系统，再继续配置包事务。

<a id="sec-checklist"></a>

## 14. 安全和运维检查清单

- [ ] 更新安全补丁；启用自动安全更新时确认维护窗口和重启策略。
- [ ] 使用普通管理员账户，避免日常 root 登录；SSH 关闭密码认证前先验证密钥登录。
- [ ] 防火墙仅开放必要端口；确认云安全组与主机防火墙一致。
- [ ] 保持 AppArmor/SELinux 启用；对拒绝记录采取定点修复。
- [ ] 禁用不需要的服务；用 `systemctl --failed` 复核服务异常。
- [ ] 启用时间同步和 journal 持久化；对日志设置大小/保留期限和轮转。
- [ ] 配置磁盘、内存、CPU、证书到期等监控告警。
- [ ] 使用可信软件源并验证 GPG 签名；避免直接运行来历不明的安装脚本。
- [ ] 备份要定期验证可恢复；带 `rsync --delete` 的任务先 dry-run。
- [ ] 递归 `chmod`、`chown`、删除、升级、防火墙变更前检查路径和影响范围。

Debian/Ubuntu 可用 `unattended-upgrades` 自动安装安全更新；Fedora/RHEL 可用 `dnf-automatic`。安装后检查对应配置文件：在 `/etc/dnf/automatic.conf` 中按需设定 `upgrade_type = security` 和 `apply_updates = yes`，并确认是否允许自动重启。

```bash
sudo apt install unattended-upgrades
# 安装 Debian 系自动更新工具
sudo dpkg-reconfigure -plow unattended-upgrades
# 交互式配置 unattended-upgrades；-p low 设置较低 debconf 提示优先级
sudo dnf install dnf-automatic
# 安装 DNF 自动更新定时器及配置
sudo systemctl enable --now dnf-automatic.timer
# 启用并立即启动自动更新 timer；更新类型由 /etc/dnf/automatic.conf 控制
sudo apt install fail2ban
# 在 Debian 系安装登录失败防护服务
sudo dnf install fail2ban
# 在 Fedora/RHEL 安装登录失败防护服务
sudo systemctl enable --now fail2ban
# 设置 fail2ban 开机启动并立即启动
sudo fail2ban-client status
# 查看 fail2ban 服务状态和已启用 jail
sudo fail2ban-client status sshd
# 查看 sshd jail 的封禁状态；需先在 jail 配置中启用该 jail
```

Fail2ban 本地覆盖配置应放在 `/etc/fail2ban/jail.local` 或 `jail.d/`，不要直接修改发行版提供的 `jail.conf`。设置 `bantime`（封禁时长）、`findtime`（统计窗口）和 `maxretry`（触发封禁的失败次数）时，需按实际 SSH 端口、日志后端和运维恢复能力评估。



<a id="sec-summary"></a>

## 15. 一句话排错法

遇到“不认识命令”时，先看任务属于哪一层：**包管理**查 `apt` ↔ `dnf`，**本地包格式**查 `.deb` ↔ `.rpm`，**网络**优先查通用 `ip`/`ss`/`nmcli`，**服务和日志**查 `systemctl`/`journalctl`，**防火墙**确认是 UFW、firewalld 还是 nftables，**权限拒绝**确认 AppArmor 或 SELinux。再用 `cat /etc/os-release` 确认系统家族，用 `command -v <命令>` 检查工具是否安装。

```bash
command -v apt dnf rpm dpkg ufw firewall-cmd nft systemctl
# 对每个命令查询当前 PATH 中是否存在可执行文件；不存在时不代表系统不支持该功能
```



<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/7777167/  

