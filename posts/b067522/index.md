# Windows常用操作


一份非常全面和实用的Windows常用命令指南，涵盖系统信息、硬件信息、系统维护和防火墙设置、系统稳定与性能监控命令、网络诊断高级命令、系统配置与启动管理命令、等核心类别。

<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

**重要提示**：涉及系统底层修改（修复、防火墙规则、进程终止）的命令，**务必以管理员身份运行** CMD 或 PowerShell。
命令主要在 **命令提示符（CMD）** 或 **PowerShell** 中运行，功能更复杂的任务推荐 PowerShell。

---

# 版本适用性速览

| 命令                                            | 适用性                                                       | 新系统替代方案                                            |
| ----------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| `wmic`                                          | ⚠️ Windows 10 早期可用；**Windows 11 起默认移除/弃用**（官方宣布弃用） | PowerShell `Get-CimInstance`（见第十三章对照表）          |
| `at`                                            | ❌ Win8 起弃用，仅兼容性保留                                  | `schtasks`                                                |
| `telnet`                                        | ⚠️ 默认未安装，需手动启用"Telnet 客户端"功能                  | PowerShell `Test-NetConnection`                           |
| `dfrg.msc`                                      | ❌ Windows XP 时代工具                                        | Win7+：`dfrgui`；Win8+：`Optimize-Volume`（PowerShell）   |
| `netsh advfirewall`                             | ✅ 仍可用，微软推荐逐步迁移                                   | PowerShell `NetSecurity` 模块（`New-NetFirewallRule` 等） |
| `ipconfig` / `ping` / `netstat` / `tasklist` 等 | ✅ 全版本通用                                                 | —                                                         |

# 一、系统信息查询

| 命令             | 作用                                                     | 注释                                                       |
| ---------------- | -------------------------------------------------------- | ---------------------------------------------------------- |
| `systeminfo`     | 显示系统版本、安装日期、启动时间、补丁、网卡等完整配置   | 加 `\\| findstr /B /C:"OS 名称" /C:"OS 版本"` 可过滤指定行 |
| `winver`         | 弹窗显示 Windows 版本和内部版本号                        | 也可在"运行"(Win+R)中直接输入                              |
| `slmgr.vbs -xpr` | 查看是否**永久激活**                                     | `-dlv` 显示详细许可证信息；`-ipk <密钥>` 安装产品密钥      |
| `hostname`       | 显示计算机名                                             | 常用于脚本中取本机标识                                     |
| `whoami`         | 显示当前用户（域\用户名）                                | 加 `/priv` 显示当前用户所有特权                            |
| `msinfo32`       | 图形化"系统信息"，软硬件树状图，比 systeminfo 更详细分类 | 排查驱动冲突常用                                           |
| `dxdiag`         | DirectX 诊断工具，查看显示/声音/输入设备                 | 游戏、显卡问题排查首选                                     |

**PowerShell 替代（推荐）：**

```powershell
Get-ComputerInfo                     # 相当于 systeminfo 的现代化版本，信息更结构化
Get-CimInstance Win32_OperatingSystem  # 获取系统版本、安装日期、启动时间等
```

---

# 二、硬件信息查询

| 命令                                                        | 作用                                 | 注释                                            |
| ----------------------------------------------------------- | ------------------------------------ | ----------------------------------------------- |
| `wmic cpu get name,NumberOfCores,NumberOfLogicalProcessors` | CPU 型号/物理核/逻辑核               | ⚠️ 见弃用说明，推荐下方 PowerShell 替代          |
| `wmic memorychip get capacity,speed,manufacturer`           | 内存容量/频率/厂商                   | 同上                                            |
| `wmic diskdrive get size,model`                             | 物理磁盘型号与容量                   | 同上                                            |
| `wmic bios get serialnumber`                                | 查 BIOS 序列号（报修、资产登记常用） | 同上                                            |
| `driverquery`                                               | 列出所有设备驱动程序                 | `/v` 详细信息；`/fo csv > drivers.csv` 导出 CSV |

**PowerShell 替代（wmic 已弃用，新系统推荐）：**

```powershell
Get-CimInstance Win32_Processor              # CPU 信息（对应 wmic cpu）
Get-CimInstance Win32_PhysicalMemory         # 内存信息（对应 wmic memorychip）
Get-CimInstance Win32_DiskDrive              # 物理磁盘（对应 wmic diskdrive）
Get-CimInstance Win32_BIOS                   # BIOS 序列号（对应 wmic bios）
Get-CimInstance Win32_LogicalDisk            # 逻辑磁盘容量/剩余空间（对应 wmic logicaldisk）
Get-CimInstance Win32_BaseBoard              # 主板信息（对应 wmic baseboard）
Get-CimInstance Win32_DesktopMonitor         # 显示器/分辨率（对应 wmic desktopmonitor）
Get-PnpDevice                                # 设备管理器的命令行版（比 driverquery 更现代）
```

---

# 三、系统维护

## 3.1 磁盘

| 命令           | 作用               | 注释                                                       |
| -------------- | ------------------ | ---------------------------------------------------------- |
| `chkdsk C: /f` | 检查并修复磁盘错误 | `/f` 修复错误，`/r` 同时查找坏扇区；修复系统盘需重启后执行 |
| `cleanmgr`     | 图形化磁盘清理     | 也可运行 `cleanmgr /sageset:1` 做无人值守清理配置          |

## 3.2 网络修复（三连招，遇到断网/网页打不开依次执行）

```cmd
ipconfig /flushdns        # 清除本地DNS缓存，解决"域名能ping通IP但打不开网页"
netsh winsock reset       # 重置网络套接字（解决软件冲突导致的网络异常）
netsh int ip reset        # 重置TCP/IP协议栈，执行后需重启
```

`ipconfig` 常用参数：`/all` 完整信息；`/release` + `/renew` 释放/更新 DHCP 租约。

## 3.3 连通性测试

| 命令                     | 作用                        | 注释                                                         |
| ------------------------ | --------------------------- | ------------------------------------------------------------ |
| `ping www.baidu.com -t`  | 持续测试连通性，Ctrl+C 停止 | 丢包、延迟排查第一步                                         |
| `tracert www.baidu.com`  | 追踪数据包经过的路由        | 判断在哪一跳断掉                                             |
| `pathping www.baidu.com` | tracert + 逐跳丢包率统计    | 比 tracert 更能定位中间节点问题，耗时较长                    |
| `telnet 192.168.1.1 80`  | 测试**TCP端口**是否开放     | ⚠️ 默认未安装；新系统推荐 `Test-NetConnection 192.168.1.1 -Port 80` |

**PowerShell 替代：**

```powershell
Test-NetConnection www.baidu.com -InformationLevel Detailed   # 集成 ping+路由+端口测试
Test-NetConnection 192.168.1.1 -Port 3389 -InformationLevel Quiet   # 只输出 True/False，适合脚本
Resolve-DnsName www.baidu.com                                  # 对应 nslookup
```

## 3.4 进程与服务

| 命令                            | 作用                                   | 注释                                         |
| ------------------------------- | -------------------------------------- | -------------------------------------------- |
| `tasklist \\| findstr "chrome"` | 查找指定进程                           | 可加 `/svc` 查看进程关联的服务               |
| `taskkill /im notepad.exe /f`   | 按映像名强制结束进程                   | `/pid 1234` 按 PID 结束；`/t` 同时结束子进程 |
| `sc query`                      | 列出所有服务状态                       | `sc stop wuauserv` 停止 Windows Update 服务  |
| `taskmgr`                       | 任务管理器（Ctrl+Shift+Esc）           | "启动"选项卡管理开机自启                     |
| `services.msc`                  | 服务管理控制台                         | 设置启动类型：自动/手动/禁用                 |
| `msconfig`                      | 系统配置（启动项、安全引导、干净启动） | 排查软件冲突常用                             |

**PowerShell 替代：**

```powershell
Get-Process chrome                # 对应 tasklist
Stop-Process -Name notepad -Force # 对应 taskkill
Get-Service / Stop-Service wuauserv  # 对应 sc query / sc stop
```

## 3.5 系统文件修复（按顺序执行，逐级深入）

```cmd
sfc /scannow                              # 第1步：扫描并修复受保护的系统文件（需管理员）
DISM /Online /Cleanup-Image /RestoreHealth  # 第2步：修复系统映像（SFC无效时使用，需联网，需管理员）
sfc /scannow                              # 第3步：DISM 修复后再跑一次 SFC 收尾
```

---

# 四、防火墙配置

## 4.1 查看与开关

```cmd
netsh advfirewall show allprofiles                    # 查看所有配置文件（域/专用/公用）状态
netsh advfirewall set allprofiles state on            # 启用防火墙
netsh advfirewall set allprofiles state off           # 禁用防火墙（不推荐）
```

## 4.2 规则管理

```cmd
:: 允许程序入站（例：放行TCP 80端口）
netsh advfirewall firewall add rule name="MyApp Port 80" dir=in action=allow protocol=TCP localport=80

:: 阻止指定IP访问
netsh advfirewall firewall add rule name="Block Bad IP" dir=in action=block remoteip=192.168.1.100

:: 删除规则
netsh advfirewall firewall delete rule name="MyApp Port 80"

:: 备份与还原
netsh advfirewall export "C:\firewall_backup.pol"
netsh advfirewall import "C:\firewall_backup.pol"
```

**PowerShell 替代（推荐）：**

```powershell
Get-NetFirewallProfile                              # 查看防火墙状态
Set-NetFirewallProfile -All -Enabled True           # 启用防火墙
New-NetFirewallRule -DisplayName "MyApp Port 80" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 80
New-NetFirewallRule -DisplayName "Block Bad IP" -Direction Inbound -Action Block -RemoteAddress 192.168.1.100
Remove-NetFirewallRule -DisplayName "MyApp Port 80"
```

## 4.3 实战：只允许特定 IP 访问本机服务（白名单模式）

图形界面操作步骤（Windows Defender 防火墙高级安全）：

1. 控制面板 → 系统和安全 → Windows Defender 防火墙 → 左侧"高级设置"。
2. **新建允许规则**：入站规则 → 新建规则 → 自定义 → 选协议和端口 → 作用域页"远程IP地址"选"这些IP地址"填入允许 IP → 操作选"允许连接" → 命名（如"允许特定IP访问HTTP"）。
3. **新建拒绝规则**：重复上述步骤，远程IP选"任何IP地址"，操作选"阻止连接"。
4. **确认顺序**：允许规则必须排在拒绝规则**之前**（防火墙自上而下匹配，先匹配先生效）。
5. **测试**：允许 IP 可访问，其他 IP 被拒即配置成功。

---

# 五、性能监控与稳定性诊断

| 命令                       | 作用                                                         | 注释                                 |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------ |
| `perfmon /rel`             | **可靠性监视器**：按时间轴显示崩溃/蓝屏/驱动安装/更新事件及稳定性指数 | 排查系统不稳定**首选工具**           |
| `perfmon` / `perfmon /sys` | 性能监视器：实时监控 CPU/内存/磁盘/网络计数器                | 深度性能瓶颈分析                     |
| `resmon`                   | 资源监视器：任务管理器增强版，实时查看进程级磁盘IO/网络连接  | 查看哪个进程在读写硬盘、监听哪个端口 |

**PowerShell 替代：**

```powershell
Get-WinEvent -LogName System -MaxEvents 20    # 查看系统事件日志（对应事件查看器命令行方式）
Get-Counter '\Processor(_Total)\% Processor Time'  # 性能计数器采样
```

---

# 六、网络诊断（管理员/IT 核心工具）

## 6.1 端口排查流程

```cmd
netstat -ano                        # 查看所有连接/监听端口及对应PID（-a所有 -n数字 -o进程号）
netstat -ano | findstr ":80"        # 查80端口被谁占用
tasklist | findstr "1234"           # 根据上一步的PID定位进程（假设PID=1234）
```

> `netstat -abn` 可直接显示进程名（需管理员权限）。
> 怀疑木马：重点看 `LISTENING` 状态下的陌生高位端口。

**PowerShell 替代：**

```powershell
Get-NetTCPConnection -State Listen              # 查看所有监听端口（比 netstat 更直观，直接带进程名）
Get-NetTCPConnection -LocalPort 80 | Select-Object OwningProcess
Get-Process -Id (Get-NetTCPConnection -LocalPort 80).OwningProcess
```

## 6.2 DNS 排查

| 命令                          | 作用                                             | 注释                                           |
| ----------------------------- | ------------------------------------------------ | ---------------------------------------------- |
| `nslookup www.baidu.com`      | 查询域名解析结果                                 | 可加指定 DNS：`nslookup www.baidu.com 8.8.8.8` |
| `nslookup -type=mx gmail.com` | 查询 MX 等特定记录                               | type 还可为 A/AAAA/CNAME/TXT/SOA               |
| `ipconfig /flushdns`          | 清除本地 DNS 缓存                                | 解析异常时先执行它                             |
| `nslookup`（单独输入）        | 进入交互模式，依次输域名和DNS服务器，Ctrl+C 退出 | —                                              |

**PowerShell 替代**：`Resolve-DnsName www.baidu.com -Type MX`

---

# 七、目录与文件操作

```cmd
D:                    # 切换到D盘（盘符即命令）
cd \                  # 跳到当前盘根目录
cd C:\WINDOWS         # 进入当前盘指定目录
cd /d e:\software     # 跨盘切换目录，/d 参数必须加，否则无法跳转
cd ..                 # 返回上一级目录（注意 cd.. 是旧写法，标准写法有空格）
dir                   # 列出当前目录文件，相当于 Linux 的 ls
copy 源路径\文件名 目标路径\文件名    # 复制文件
move 源路径\文件名 目标路径\文件名    # 移动文件（剪切+粘贴）
del 文件名             # 删除文件（不能删目录；删目录用 rd /s 目录名）
```

其他实用命令：

| 命令                            | 作用                                        | 注释                                                         |
| ------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| `type 文件名`                   | 查看文本文件内容                            | 相当于 Linux 的 cat                                          |
| `fc one.txt two.txt > diff.txt` | 对比两个文件差异并输出到文件                | `>` 覆盖写入；`>>` 追加写入                                  |
| `findstr "hello" aa.txt`        | 在文件中搜索字符串                          | 支持正则，比 find 更强大                                     |
| `echo hello > c:\1.txt`         | 重定向：先清空再写入                        | `>>` 则为追加到末尾                                          |
| `jstack 12912 > d:/s.txt`       | 把命令输出转存到文件（Java 线程 dump 示例） | 重定向的典型用法                                             |
| `color 4`                       | 设置 CMD 颜色                               | 0黑1蓝2绿3浅绿4红5紫6黄7白8灰9淡蓝A淡绿B淡浅绿C淡红D淡紫E淡黄F亮白 |
| `title 新标题`                  | 修改 CMD 窗口标题                           | 多窗口操作时区分窗口用                                       |
| `set`                           | 显示所有环境变量                            | `set PATH` 查看指定变量                                      |
| `arp -a`                        | 显示 ARP 缓存（IP↔MAC 映射）                | 排查局域网 IP 冲突                                           |
| `route print`                   | 显示路由表（网络地址/掩码/网关/接口）       | 排查多网卡路由问题                                           |

---

# 八、计划任务与用户管理

| 命令                                        | 作用                                                        | 版本说明与注释                                        |
| ------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------- |
| `schtasks /query /fo LIST /v`               | 查询所有计划任务（详细列表）                                | `at` 命令已在 Win8 起弃用，统一用 schtasks            |
| `net user 用户名 密码 /add`                 | 新建用户                                                    | 需管理员权限                                          |
| `net user 帐户名`                           | 查看帐户属性                                                | —                                                     |
| `net localgroup administrators 用户名 /add` | 将用户加入管理员组                                          | —                                                     |
| `net start` / `net stop 服务名`             | 查看/开启/停止服务                                          | 如 `net stop wuauserv` 停止 Windows Update            |
| `net time \\目标IP`                         | 查看目标主机时间                                            | `/set /yes` 与目标机时间同步                          |
| `shutdown /s /t 60`                         | 60秒后关机                                                  | `/r` 重启；`/a` 取消；`/f` 强制关闭程序               |
| `format D: /FS:NTFS`                        | 格式化磁盘                                                  | ⚠️ 危险操作；新系统推荐图形界面或 `diskpart`（需谨慎） |
| `powercfg /energy`                          | 生成电源效率报告（60秒后输出 energy-report.html，需管理员） | 排查耗电/唤醒问题                                     |
| `powercfg /batteryreport`                   | 生成笔记本电池健康报告                                      | 查看电池损耗程度                                      |
| `powercfg /lastwake`                        | 查看最近是什么设备唤醒了电脑                                | 解决睡眠后被莫名唤醒                                  |

**PowerShell 替代：**

```powershell
Get-ScheduledTask / Register-ScheduledTask     # 计划任务管理
Get-LocalUser / New-LocalUser                  # 本地用户管理
Stop-Computer -ComputerName 目标IP -Force      # 远程关机
Get-WinEvent -LogName Security -MaxEvents 10   # 安全事件审计
```

---

# 九、系统配置与启动管理

| 命令           | 作用                                                   |
| -------------- | ------------------------------------------------------ |
| `msconfig`     | 系统配置：安全引导、服务筛选、干净启动（排查软件冲突） |
| `taskmgr`      | 任务管理器，"启动"选项卡是现代管理开机自启的标准位置   |
| `services.msc` | 服务控制台：启动/停止/禁用服务                         |
| `devmgmt.msc`  | 设备管理器（图形化）                                   |
| `dfrgui`       | 磁盘碎片整理（Win7+；XP时代为 dfrg.msc，已淘汰）       |

**PowerShell 替代（Win8+）：** `Optimize-Volume -DriveLetter C`（自动判断 SSD/HDD 采用合适策略）

---

# 十、批处理基础

## 10.1 for 命令与变量

基本格式：`FOR /参数 %variable IN (set) DO command [command_parameters]`

| 元素        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| `%variable` | 可替换参数，如 `%i`；批处理内部用 `%%i`（双百分号）；调用变量用 `%i%`；**区分大小写**（`%i` ≠ `%I`） |
| `%0 ~ %9`   | 批处理内置变量：`%0`=脚本名本身，`%1`~`%9`=命令行第1~9个参数。如 `net use ipipc$ pass /user:user` 中 ip=`%1`、pass=`%2`、user=`%3` |
| `(set)`     | 文件集合或数字序列，支持通配符。`(1 1 254)` = 起始值1、步长1、结束值254（即1→254）；`(1 -1 254)` 表示 254→1 倒序 |
| `command`   | 对每个元素执行的命令；多条命令用 `&` 隔开                    |
| `/L`        | 增量遍历模式（数字序列）                                     |
| `/F`        | 从文件中逐行取值，直到取完                                   |

## 10.2 实例1：批量扫描网段建立 IPC$ 连接

```bat
@echo off
echo 用法格式：test.bat *.*.* > test.txt
for /L %%G in (1 1 254) do echo %1.%%G >>test.txt & net use \\%1.%%G /user:administrator | find "命令成功完成" >>test.txt
```

说明：对 C 类网段 254 个 IP 依次尝试建立 administrator 空密码 IPC$ 连接，成功则记录到 test.txt。

- `/L`：增量循环 1→254
- `%1`：命令行第一个参数（IP 前三段，如 `192.168.1`）
- `%%G`：循环变量（IP 最后一段）
- `&`：连接多条命令
- `|`：把 net use 输出交给 find 过滤

## 10.3 实例2：配合字典文件批量尝试

```bat
@echo off
echo 用法格式：ok.bat ip
FOR /F %%i IN (D:\user.dic) DO smb.exe %1 %%i D:\pass.dic 200
```

说明：输入目标 IP（`%1`）后，用 `d:\pass.dic` 字典暴力破解 `d:\user.dic` 中的用户密码，直到取完为止。

> ⚠️ 以上两个实例属于渗透测试/安全评估场景，**仅限授权环境使用**，未授权扫描和爆破涉嫌违法。

## 10.4 其他批处理要点

- `pause`：暂停等待按键；`@echo off`：关闭命令回显。
- 管道 `|`：前一个命令的输出作为后一个命令的输入，如 `netstat -ano | find "192.168.1.10"`。
- 重定向：`>` 覆盖写入文件，`>>` 追加写入文件。

---

# 十一、WMIC 命令（⚠️ 已弃用）

> **版本说明**：`wmic` 是功能强大的 WMI 命令行工具，但微软已宣布弃用，**Windows 11 新版本默认不再预装**。新系统请使用 PowerShell 的 `Get-CimInstance` 替代（对照见第一章末表格），功能完全覆盖且更规范。

## 11.1 交互模式

```cmd
wmic                          # 进入交互模式（提示符变为 wmic:root\cli>）
baseboard get manufacturer,product   # 例：查主板厂商和型号

:: 常用查询
wmic process                                # 列出所有进程
wmic process where name="chrome.exe" list full   # 显示指定进程详细信息
wmic process where name="qq.exe" call terminate  # 按名称结束进程（方式1）
wmic process where name="qq.exe" delete          # 按名称结束进程（方式2）
wmic process where pid="123" delete              # 按PID结束进程
wmic bios list full                       # 显示完整BIOS信息
wmic cpu get name                         # CPU型号
wmic diskdrive get caption,size,interfacetype    # 物理磁盘型号/容量/接口
wmic logicaldisk get name,description,filesystem,size,freespace   # 逻辑磁盘格式/容量/剩余空间
wmic memlogical get TotalPhysicalMemory   # 物理内存总量
wmic DESKTOPMONITOR where Status='ok' get ScreenHeight,ScreenWidth  # 屏幕分辨率
wmic process get name,executablepath      # 进程名与可执行路径（排查可疑进程）
```

## 11.2 服务管理（wmic）

```cmd
Service where caption="windows time" call stopservice     # 停止服务（显示名）
Service where caption="windows time" call startservice    # 启动服务
Service where name="w32time" call stopservice             # 按服务名停止
```

> `caption` 是显示名称（如 "Windows Time"），`name` 是服务名（如 `w32time`），带空格的值要用引号。
> 执行 call 命令后会有确认提示，输入 `y` 确认；`ReturnValue = 0` 表示成功。
> `list` 参数控制输出范围：`Brief`（摘要）、`Full`（全部，默认）、`Instance`、`Status`、`Writeable`。

## 11.3 网络配置（wmic）

```cmd
:: 设置静态IP（index=0 为第一块网卡）
wmic nicconfig where index=0 call enablestatic("192.168.1.5"),("255.255.255.0")

:: 设置默认网关，(1) 为跃点数
wmic nicconfig where index=0 call setgateways("192.168.1.1"),(1)
```

**新系统替代（推荐）：**

```powershell
Get-NetAdapter                              # 查看网卡及索引
New-NetIPAddress -InterfaceIndex 12 -IPAddress 192.168.1.5 -PrefixLength 24
Set-DnsClientServerAddress -InterfaceIndex 12 -ServerAddresses 192.168.1.1
```

## 11.4 远程管理（wmic，需目标机开启相应服务与权限）

```cmd
wmic /node:"192.168.203.131" /password:"" /user:"administrator"   # 连接远程主机进入交互模式

:: 远程执行命令（如打开cmd）
wmic /node:192.168.201.131 /user:administrator /password:123456 process call create cmd.exe

:: 远程重启
wmic /node:192.168.1.10 /user:administrator /password:123456 process call create "shutdown.exe -r -f -m"

:: 远程开启远程桌面
wmic /node:192.168.1.2 /USER:administrator PATH win32_terminalservicesetting WHERE (__Class!="") CALL SetAllowTSConnections 1
```

## 11.5 系统与进程管理实例

```cmd
wmic process call create "C:\Program Files\Tencent\QQ\QQ.exe"     # 创建新进程
wmic process call create shutdown.exe                            # 关闭本地计算机

:: 结束可疑进程（路径不在系统目录的 explorer.exe 视为可疑）
wmic process where "name='explorer.exe' and executablepath<>'%SystemDrive%\\windows\\explorer.exe'" delete

:: 更改计算机名 / 帐户名
wmic computersystem where "caption='%ComputerName%'" call rename newcomputername
wmic USERACCOUNT where "name='%UserName%'" call rename newUserName

:: 虚拟内存管理
wmic PageFileSet set InitialSize="512",MaximumSize="512"
wmic PageFileSet create name="E:\\pagefile.sys",InitialSize="1024",MaximumSize="1024"   # 页面文件移到E盘
wmic PageFileSet where "name='C:\\pagefile.sys'" delete                                  # 删除C盘页面文件（重启生效）
```

## 11.6 批处理实例（wmic 获取文件时间）

```bat
@echo off
for /f "skip=1 tokens=1,3,5 delims=. " %%a in ('wmic datafile where name^="c:\\windows\\system32\\notepad.exe" get CreationDate^,LastAccessed^,LastModified') do (
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
```

```cmd
:: 全盘搜索 qq.exe 并输出所在目录
for /f "skip=1 tokens=1*" %i in ('wmic datafile where "FileName='qq' and extension='exe'" get drive^,path') do (set "qPath=%i%j"&@echo %qPath:~0,-3%)
```

```bat
:: 查看进程内存占用（以KB显示）
@echo off
for /f "skip=1 tokens=1-2 delims= " %%a in ('wmic process where caption^="conime.exe" get WorkingSetSize^,PeakWorkingSetSize') do (
set /a m=%%a/1024
set /a mm=%%b/1024
echo 进程conime.exe现在占用内存：%m%K；最高占用内存：%mm%K
)
pause
```

> 以上批处理中 `^` 为转义符，用于在 for 命令的引号字符串内转义 `=`、`,`、`( )` 等特殊字符。

---

# 十二、安全排查场景

## 12.1 永恒之蓝（445端口）感染排查

```cmd
netstat –ano | findstr "445"      # 查看445端口相关连接与进程号
```

发现 `SYN_SENT` 状态的445连接 → 疑似感染永恒之蓝病毒。

```cmd
tasklist | findstr 20164          # 根据进程号（假设20164）定位进程名
```

然后在任务管理器中找到该进程 → 右键"打开文件位置" → 结束进程，再做全盘杀毒。

## 12.2 端口重定向工具 fpipe（第三方工具）

```cmd
fpipe -l 80 -s 1029 -r 80         # 本机1029端口转发到80，扫描者看到的主机信息会被混淆
Fpipe -l 23 -s 88 -r 23 目标IP    # 将发往目标IP 23端口的Telnet请求经88端口重定向
```

> `fpipe.exe` 为第三方安全工具，非 Windows 自带；仅限授权测试环境。

## 12.3 瑞星防火墙配置（低版本 Windows）

1. **设置只允许某 IP 段访问端口**：

- 设白名单，配置放行 IP，同时端口规则设为所有端口范围禁止 TCP/UDP，方向本地禁止。
- IP 规则配置（清除默认规则后）：
  - 顶部：所有 IP 包，远程=允许 IP 范围，本地=任意，协议 all，放行，弹窗提示。
  - 顶部之下：所有 IP 包，本地任意，协议 all，阻止。

2. **启用端口隐身、高频阻拦**，查看扫描 IP 并添加黑名单。
3. **封禁所有不再使用的端口**，只给需要的端口打补丁（设置中确定方向，检测到端口状态为 `filtered`）。
4. **一直开启端口隐身**，使用时打开，不用时关闭。

> ⚠️ 瑞星防火墙为早期产品，现代 Windows 直接使用系统自带 Defender 防火墙（见第四章）即可实现同等防护。

---

# 十三、新系统（PowerShell）替代命令对照总表

| 旧命令 (CMD)                         | 新系统推荐 (PowerShell)                             | 说明            |
| ------------------------------------ | --------------------------------------------------- | --------------- |
| `systeminfo`                         | `Get-ComputerInfo`                                  | 系统信息        |
| `wmic cpu get name`                  | `Get-CimInstance Win32_Processor`                   | CPU             |
| `wmic memorychip get capacity`       | `Get-CimInstance Win32_PhysicalMemory`              | 内存            |
| `wmic diskdrive get size,model`      | `Get-CimInstance Win32_DiskDrive`                   | 物理磁盘        |
| `wmic logicaldisk get freespace`     | `Get-CimInstance Win32_LogicalDisk` 或 `Get-Volume` | 磁盘空间        |
| `wmic bios get serialnumber`         | `Get-CimInstance Win32_BIOS`                        | BIOS序列号      |
| `wmic process` / `tasklist`          | `Get-Process`                                       | 进程            |
| `wmic process where name="x" delete` | `Stop-Process -Name x -Force`                       | 结束进程        |
| `taskkill /pid 1234`                 | `Stop-Process -Id 1234 -Force`                      | 按PID结束       |
| `sc query` / `sc stop`               | `Get-Service` / `Stop-Service`                      | 服务管理        |
| `netstat -ano`                       | `Get-NetTCPConnection`                              | 端口/连接       |
| `netstat -ano \\| findstr :80`       | `Get-NetTCPConnection -LocalPort 80`                | 端口占用        |
| `nslookup`                           | `Resolve-DnsName`                                   | DNS解析         |
| `ping` / `telnet ip port`            | `Test-NetConnection ip -Port port`                  | 连通性/端口测试 |
| `ipconfig /flushdns`                 | `Clear-DnsClientCache`                              | 清DNS缓存       |
| `netsh advfirewall`                  | `Get/New/Set-NetFirewallRule`                       | 防火墙          |
| `netsh int ip reset`                 | `Get-NetAdapter` + `Restart-NetAdapter`             | 网络重置        |
| `at`                                 | `schtasks` / `Get-ScheduledTask`                    | 计划任务        |
| `shutdown /s /t 60`                  | `Stop-Computer` / `Restart-Computer`                | 关机/重启       |
| `driverquery`                        | `Get-PnpDevice`                                     | 驱动/设备       |
| `dfrg.msc` / `dfrgui`                | `Optimize-Volume`                                   | 磁盘优化        |
| `wmic /node:... process call create` | `Invoke-Command -ComputerName ... { }`              | 远程执行        |

---

# 总结与建议

- **快速上手**：遇到系统问题，依次尝试 `ipconfig /flushdns` → `sfc /scannow` → `DISM /Online /Cleanup-Image /RestoreHealth`。
- **信息查询**：快速看版本用 `winver`，详细软硬件用 `msinfo32`，网络配置用 `ipconfig /all`。
- **网络调试四板斧**：`ping` 测连通 → `nslookup` 查解析 → `tracert`/`pathping` 追路由 → `netstat -ano` 查端口。
- **端口占用**：`netstat -ano | findstr :端口号` 查 PID → `tasklist | findstr PID` 定位进程（PowerShell 用 `Get-NetTCPConnection` 一步到位）。
- **系统不稳定**：首选 `perfmon /rel` 可靠性监视器查看近期错误事件。
- **权限是关键**：涉及系统底层修改的命令务必以管理员身份运行。
- **新系统迁移**：`wmic`、`at` 已弃用；复杂任务直接上 PowerShell，配合 `Get-CimInstance`、`Test-NetConnection`、`Get-NetTCPConnection` 等现代化命令。



---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/b067522/  

