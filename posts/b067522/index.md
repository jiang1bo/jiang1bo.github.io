# Windows常用操作


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

# windows系统

```
windows常用命令
```



好的，这里为您整理了一份非常全面和实用的Windows常用命令指南，涵盖系统信息、硬件信息、系统维护和防火墙设置等核心类别。这些命令主要在 **命令提示符（CMD）** 或 **PowerShell**（功能更强大）中运行。

**重要提示**：部分命令需要**以管理员身份运行**命令行工具才能执行。

---

### 一、系统信息查询命令

这些命令用于查看操作系统版本、配置、激活状态等。

1.  **`systeminfo`**
    *   **作用**：显示详细的计算机及其操作系统的配置信息，包括版本、安装日期、启动时间、硬件抽象层、网卡信息等。
    *   **示例**：`systeminfo`
    *   **进阶**：`systeminfo | findstr /B /C:“OS 名称” /C：“OS 版本”` (只显示特定信息)

2.  **`winver`**
    *   **作用**：弹出一个简洁的对话框，显示Windows版本和内部版本号。
    *   **示例**：在“运行”（Win+R）中直接输入 `winver`。

3.  **`slmgr.vbs`** (软件许可管理脚本)
    *   **作用**：查看和管理Windows激活状态。
    *   **常用参数**：
        *   `slmgr.vbs -xpr`：查看是否永久激活。
        *   `slmgr.vbs -dlv`：显示详细的许可证信息。
        *   `slmgr.vbs -ipk <产品密钥>`：安装产品密钥（替换 `< >` 内内容）。

4.  **`hostname`**
    *   **作用**：显示当前计算机的名称。
    *   **示例**：`hostname`

5.  **`whoami`**
    *   **作用**：显示当前登录用户的域名和用户名。
    *   **示例**：`whoami`
    *   **进阶**：`whoami /priv` (显示当前用户的所有特权)。

---

### 二、硬件信息查询命令

这些命令用于查看CPU、内存、磁盘、设备等硬件详情。

1.  **`wmic`** (Windows Management Instrumentation Command-line)
    *   功能极其强大的硬件信息查询工具。下面是一些常用查询：
    *   **CPU**：`wmic cpu get name,NumberOfCores,NumberOfLogicalProcessors`
    *   **内存**：`wmic memorychip get capacity, speed, manufacturer`
    *   **磁盘**：`wmic diskdrive get size, model`
    *   **BIOS**：`wmic bios get serialnumber` (获取BIOS序列号)
    *   **查看全部**：直接输入 `wmic` 进入交互模式。

2.  **`dxdiag`**
    *   **作用**：打开“DirectX 诊断工具”，这是一个图形化界面，可以查看系统、显示、声音、输入设备等详细信息，常用于游戏问题排查。
    *   **示例**：在“运行”（Win+R）中直接输入 `dxdiag`。

3.  **`msinfo32`**
    *   **作用**：打开“系统信息”图形化工具，提供比 `systeminfo` 更详细、分类更清晰的软硬件信息树状图。
    *   **示例**：在“运行”（Win+R）中直接输入 `msinfo32`。

4.  **`driverquery`**
    *   **作用**：显示所有已安装的设备驱动程序列表及其属性。
    *   **示例**：`driverquery`
    *   **进阶**：`driverquery /v` (显示详细信息) 或 `driverquery /fo csv > drivers.csv` (导出到CSV文件)。

---

### 三、常用系统维护命令

这些命令用于磁盘管理、网络修复、进程控制等日常维护。

1.  **磁盘检查与修复**
    *   **`chkdsk`**：检查磁盘状态并显示状态报告。修复错误需要管理员权限。
        *   **示例**：`chkdsk C: /f` (`/f` 表示修复错误，重启后可能执行)。

2.  **磁盘清理**
    *   **`cleanmgr`**：打开图形化的“磁盘清理”工具。
        *   **示例**：在“运行”（Win+R）中直接输入 `cleanmgr`。

3.  **网络相关**
    *   **`ipconfig`**：显示所有网络适配器的IP地址、子网掩码、默认网关等。
        *   **常用参数**：
            *   `ipconfig /all`：显示完整信息。
            *   `ipconfig /release` 和 `ipconfig /renew`：释放和更新DHCP租约。
            *   `ipconfig /flushdns`：**清除本地DNS缓存**，解决某些网站无法访问的问题。
    *   **`ping`**：测试与目标主机的网络连接。
        *   **示例**：`ping www.baidu.com -t` (`-t` 表示持续ping，按Ctrl+C停止)。
    *   **`tracert`**：跟踪数据包到达目标主机所经过的路由。
        *   **示例**：`tracert www.google.com`
    *   **`netsh`**：功能强大的网络配置工具。
        *   **重置网络**：`netsh winsock reset` 然后 `netsh int ip reset` (执行后需重启)。

4.  **进程与服务管理**
    *   **`tasklist`**：显示当前正在运行的所有进程列表，类似任务管理器。
        *   **示例**：`tasklist | findstr “chrome”` (查找包含“chrome”的进程)。
    *   **`taskkill`**：终止进程。
        *   **示例**：`taskkill /im notepad.exe` (按映像名称终止) 或 `taskkill /pid 1234` (按PID终止)。
    *   **`sc`** (服务控制)：用于管理Windows服务。
        *   **示例**：`sc query` (列出所有服务)，`sc stop wuauserv` (停止Windows Update服务)。

5.  **文件系统操作**
    *   **`sfc /scannow`**：**系统文件检查器**，扫描并修复受保护的Windows系统文件。**（需管理员权限）**
    *   **`DISM`**：部署映像服务和管理工具，用于修复Windows映像，比SFC更底层。
        *   **常用命令**：`DISM /Online /Cleanup-Image /RestoreHealth` (**需管理员权限**，通常在执行SFC修复无效后使用)。

---

### 四、防火墙设置命令

Windows防火墙主要通过 `netsh advfirewall` 上下文进行配置。

1.  **查看防火墙状态**
    *   **`netsh advfirewall show allprofiles`**：显示所有配置文件（域、专用、公用）的防火墙状态和设置。

2.  **启用/禁用防火墙**
    *   **启用**：`netsh advfirewall set allprofiles state on`
    *   **禁用**：`netsh advfirewall set allprofiles state off`

3.  **添加/删除防火墙规则**
    *   **允许程序入站**（例如，允许MyApp通过80端口TCP）：
        ```
        netsh advfirewall firewall add rule name="MyApp Port 80" dir=in action=allow protocol=TCP localport=80
        ```
    *   **阻止IP地址**：
        ```
        netsh advfirewall firewall add rule name="Block Bad IP" dir=in action=block remoteip=192.168.1.100
        ```
    *   **删除规则**：
        `netsh advfirewall firewall delete rule name="MyApp Port 80"`

4.  **备份与还原防火墙规则**
    *   **备份**：`netsh advfirewall export “C:\firewall_backup.pol”`
    *   **还原**：`netsh advfirewall import “C:\firewall_backup.pol”`



好的，我们继续深入，为您介绍更多强大且实用的Windows维护与诊断命令。这些命令是排查网络问题、分析系统稳定性和性能的利器。

---

### 五、系统稳定与性能监控命令

1.  **`perfmon /rel`**
    *   **作用**：**一键打开“可靠性监视器”**。这是一个非常重要的图形化工具，按时间轴显示系统稳定性事件（如应用崩溃、Windows故障、驱动程序安装/卸载、Windows更新），并生成一个“稳定性指数”。是排查系统蓝屏、频繁崩溃原因的**首选工具**。
    *   **用法**：在“运行”（Win+R）中直接输入 `perfmon /rel`。

2.  **`perfmon` 或 `perfmon /sys`**
    *   **作用**：打开“性能监视器”。您可以创建数据收集器集，实时监控或记录CPU、内存、磁盘、网络等成百上千个性能计数器的数据，用于进行深入的性能瓶颈分析。

3.  **`resmon`**
    *   **作用**：打开“资源监视器”。这是任务管理器的增强版，提供关于CPU、内存、磁盘和网络活动的**实时、更详细的视图**。可以查看哪个进程在读写硬盘、监听哪个网络端口等，非常直观。
    *   **用法**：在“运行”中直接输入 `resmon`，或在任务管理器的“性能”选项卡中点击“打开资源监视器”。

---

### 六、网络诊断高级命令

这些是网络管理员和IT支持人员的核心工具集。

1.  **`netstat` （网络统计）**
    *   **作用**：显示活动的网络连接、监听端口、路由表等网络统计信息。**用于查看本机开放了哪些端口，哪些程序在连接外部地址，是排查木马、网络问题的关键命令。**
    *   **最常用参数组合**：
        *   `netstat -ano`：这是**最强大、最常用的组合**。
            *   `-a`：显示所有连接和监听端口。
            *   `-n`：以数字形式显示地址和端口号（更快，不进行DNS解析）。
            *   `-o`：显示拥有该连接的**进程PID**。这是关键！
        *   `netstat -abn`：`-b` 参数会尝试显示**创建每个连接或监听端口的可执行程序名**（需管理员权限），有时比 `-o` 更直接。
    *   **典型使用场景**：
        1.  怀疑有木马后门：运行 `netstat -ano`，查看 `LISTENING`（监听）状态下有无可疑的陌生端口。
        2.  程序无法连接服务器：运行 `netstat -ano | findstr “:80”`，查看本地80端口的占用情况（PID）。
        3.  找到PID后，去任务管理器（详细信息选项卡）中根据PID找到对应进程，或使用命令 `tasklist | findstr “1234”`（假设PID是1234）。

2.  **`nslookup` （名称服务器查找）**
    *   **作用**：查询DNS记录，用于诊断域名解析问题。检查某个域名是否能正确解析为IP地址，或者查询特定的DNS记录类型（如MX邮件记录）。
    *   **用法**：
        *   **交互模式**：直接输入 `nslookup`，然后依次输入要查询的域名（如 `www.baidu.com`）和指定的DNS服务器（如 `8.8.8.8`），按 Ctrl+C 退出。
        *   **直接模式**：`nslookup www.baidu.com` 或 `nslookup www.baidu.com 8.8.8.8`（使用Google DNS查询）。
        *   **查询特定记录**：`nslookup -type=mx gmail.com` （查询Gmail的邮件交换器记录）。

3.  **`pathping`**
    *   **作用**：结合了 `ping` 和 `tracert` 的功能。它先像 `tracert` 一样跟踪路径，然后对路径上的每个节点进行一段时间的统计计算，**显示每个跃点的丢包率**。对于判断网络中断或延迟发生在哪个路由器段特别有用。
    *   **示例**：`pathping www.qq.com`

4.  **`telnet` (测试端口连通性)**
    *   **注意**：Win10/11默认未安装。需要在“启用或关闭Windows功能”中启用“Telnet客户端”。
    *   **作用**：用于**测试TCP端口的连通性**，而不仅仅是ICMP（ping）。例如，想知道服务器的80端口（HTTP）或3389端口（远程桌面）是否真的开放。
    *   **用法**：`telnet <主机名或IP> <端口号>`
        *   示例：`telnet 192.168.1.1 80`。如果窗口变黑或显示连接成功字样，则端口开放；如果提示“无法打开到主机的连接”，则端口可能关闭或被防火墙阻止。

---

### 七、系统配置与启动管理命令

1.  **`msconfig`**
    *   **作用**：打开“系统配置”工具。可以管理**启动选项**（如安全启动）、**服务**、**启动项**（旧版Windows，Win10以后在任务管理器中管理）等。常用于**禁用不必要的启动程序和服务以加速启动**，或进行干净启动以排除软件冲突。
    *   **用法**：在“运行”中直接输入 `msconfig`。

2.  **`taskmgr`**
    *   **作用**：打开“任务管理器”。除了管理进程，其“启动”选项卡是管理开机启动程序的**现代标准位置**，“性能”选项卡是查看资源占用的快速入口。
    *   **用法**：在“运行”中直接输入 `taskmgr`，或按 `Ctrl+Shift+Esc` 快捷键。

3.  **`services.msc`**
    *   **作用**：打开“服务”管理控制台。可以启动、停止、禁用Windows服务，并设置其启动类型（自动/手动/禁用）。
    *   **用法**：在“运行”中直接输入 `services.msc`。

---

### 八、电源与计划任务命令

1.  **`powercfg` （电源配置）**
    *   **作用**：管理和诊断电源设置。
    *   **常用参数**：
        *   `powercfg /energy`：生成一份系统能量效率和电池寿命报告（**需管理员权限**，运行60秒后生成 `energy-report.html`）。可以找到驱动程序和设备导致的电源问题。
        *   `powercfg /batteryreport`：生成详细的电池使用历史报告（HTML格式），对于笔记本用户查看电池健康度非常有用。
        *   `powercfg /lastwake`：报告是什么设备最近将计算机从睡眠状态唤醒。

2.  **`schtasks`**
    *   **作用**：比图形界面的“任务计划程序”更强大的命令行工具，可以创建、查询、更改、删除计划任务。适合编写脚本或进行批量操作。
    *   **示例**：`schtasks /query /fo LIST /v` （以详细列表格式查询所有计划任务）。

---

### **总结与建议**

*   **快速上手**：遇到系统问题，可以依次尝试 `ipconfig /flushdns`、`sfc /scannow`、`DISM /Online /Cleanup-Image /RestoreHealth`。
*   **信息查询**：快速看系统用 `winver`，看详细软硬件用 `msinfo32`，看网络配置用 `ipconfig /all`。
*   **网络调试**：`ping` 测试连通性，`tracert` 跟踪路由。
*   **权限是关键**：记住，涉及系统底层修改（修复、防火墙规则）的命令，**务必以管理员身份运行CMD或PowerShell**。
*   **拥抱 PowerShell**：对于更复杂的任务，建议学习PowerShell（命令如 `Get-Process`， `Get-Service`， `Get-NetFirewallRule`），它比CMD更强大、更一致。
*   **基础连通性**：`ping 目标域名或IP` （检查是否能到达）。
*   **域名解析**：`nslookup 目标域名` （检查DNS是否正确解析）。
*   **路由追踪**：`pathping 目标域名` （检查网络路径和中间节点丢包）。
*   **本地端口占用**：如果服务在自己机器上启动失败，用 `netstat -ano | findstr :端口号` 检查端口是否被占用。
*   **远程端口测试**：用 `telnet 目标IP 端口号` 测试远端服务端口是否开放。
*   **查看关联进程**：根据 `netstat` 找到的PID，用 `tasklist | findstr PID` 定位是哪个程序在连接或监听。
*   **系统层面检查**：如果怀疑系统不稳定，用 `perfmon /rel` 查看近期有无系统错误事件。













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

# windows服务器只允许特定IP访问

1. ### 打开Windows Defender防火墙高级安全

打开“控制面板” > “系统和安全” > “Windows Defender 防火墙”。
在左侧菜单选择“高级设置”。

2. ### 创建允许规则

在“Windows Defender 防火墙高级安全”窗口的左侧，点击“入站规则”，然后在右侧的“操作”栏点击“新建规则”。

选择“自定义规则”，点击“下一步”。

对于“协议和端口”，根据需要保护的服务选择协议（TCP或UDP）和端口（特定端口或范围），然后点击“下一步”。

在“作用域”页面：

“本地IP地址”保持为“任何IP地址”。
在“远程IP地址”下选择“这些IP地址”，然后输入允许访问的特定IP地址或地址范围。点击“下一步”。
在“操作”页面，选择“允许连接”，点击“下一步”。

在“配置文件”页面，根据服务器的网络环境选择相应的配置文件（通常是全部勾选），点击“下一步”。

给规则命名（例如，“允许特定IP访问HTTP服务”），并提供描述（可选），点击“完成”。

3. ### 创建拒绝规则

为了确保除了指定的IP地址外，其他所有IP地址都无法访问，你需要创建一个拒绝规则。

重复上述步骤创建新规则，但在“操作”步骤中选择“阻止连接”。
在“远程IP地址”下选择“任何IP地址”。
确保规则名称和描述能清晰表明这是一个拒绝规则，如“拒绝所有其他IP访问HTTP服务”。

4. ### 确认规则顺序

确保允许规则位于拒绝规则之前，因为在处理规则时，Windows防火墙会按照从上到下的顺序应用规则，最先匹配的规则生效。

5. ### 测试规则

从允许的IP地址尝试访问服务器，确认可以成功连接。
从一个不在允许列表中的IP地址尝试访问，确保被拒绝。
通过上述步骤，你就成功设置了Windows服务器，只允许特定IP地址访问，而阻止了所有其他IP地址的访问。请根据实际需要调整端口、协议以及IP地址范围。

# **windows：wmic命令**

C:\Users\Administrator>wmic
wmic:root\cli>baseboard get manufacturer,product
Manufacturer Product
ASUSTeK COMPUTER INC. H510M-D3H/M.2 R2.0

wmic:root\cli>

### 显示进程的详细信息

输入 process where name=”chrome.exe” list full

### 显示出BIOS信息

wmic bios list full

### 停止、暂停和运行服务功能

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

### 连接远程的电脑，不过好象对要开一些相应的服务

wmic /node:”192.168.203.131″ /password:”” /user:”administrator”

### BIOS – 基本输入/输出服务 (BIOS) 管理

查看bios版本型号
wmic bios get Manufacturer,Name

C:\Users\Administrator>wmic baseboard get manufacturer,name,product
Manufacturer Name Product
ASUSTeK COMPUTER INC. 基板 H510M-D3H/M.2 R2.0

### WMIC设置IP地址

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

# windows永恒之蓝测试

在CMD窗口下，输入如下命令：netstat –ano | findstr “445”，找出相关进程号，其中SYN_SENT状态，很显然，该电脑被感染永恒之蓝病毒了

找出相关进程号对应的进程，在cmd下输入tasklist | findstr 20164，看看是哪个进程产生该数据包

打开任务管理器，找到相应的进程，先打开文件位置，然后结束进程

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

# 瑞星防火墙配置(低版本window)

**0.设置只允许某ip段可以访问端口**

一、设置白名单,配置通过ip,同时端口设置所有端口范围禁止tcp,udp，方向本地禁止，

二、ip规则配置，（清除默认规则)

顶部配置：所有ip包，远程允许ip范围，本地任意，协议类型all,放行，弹窗提示

顶部之下配置：所有ip包，本地任意，本地任意，协议类型all，阻止

**1.启用端口隐身，高频阻拦，查看扫描ip,添加黑名单**

**2.封禁所有不再使用的端口，只给需要的端口打补丁,需要在设置中确定方向，可以检测到端口，状态为filtered**

**3.一直开启端口隐身，使用时打开，不用关闭**


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/b067522/  

