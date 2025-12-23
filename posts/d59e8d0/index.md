# Nmap常用操作


# Nmap常用操作及参数

<!--more-->

```shell
nmap -O -sV 192.168.1.1
#扫描系统版本信息和服务版本信息，
#要扫描单个服务信息（如web）使用nmap -sV -p 80 192.168.1.1
#扩展 以下是常用参数：
-O：激活操作探测；
-P0：只进行扫描，不ping主机；
-PT：同TCP的ping；
-sV：探测服务版本信息；
-sP：ping扫描，仅发现目标主机是否存活；
-ps：发送同步（SYN）报文；
-PU：发送udp ping；
-PE：强制执行直接的ICMPping；
-PB：默认模式，可以使用ICMPping和TCPping；
-6：使用IPv6地址；
-v：得到更多选项信息；
-d：增加调试信息的输出；
-oN：以人们可阅读的格式输出；
-oX：以xml格式向指定文件输出信息；
-oM：以机器可阅读的格式输出；
-A：使用所有高级扫描选项；
 
-P：指定要扫描的端口，可以是一个单独的端口，用逗号隔开多个端口，使用“-”表示端口范围；
-e：在多网络接口Linux系统中，指定扫描使用的网络接口；
-g：将指定的端口作为源端口进行扫描；
 

```

–resume：继续上次执行完的扫描；

–ttl：指定发送的扫描报文的生存期；

–packet-trace：显示扫描过程中收发报文统计；

–scanflags：设置在扫描报文中的TCP标志。



## Nmap简单的扫描方式

```shell
#全面扫描：nmap -T4 -A ip

#主机发现：nmap -T4 -sn ip

#端口扫描：nmap -T4 ip

#服务扫描：nmap -T4 -sV ip

#操作系统扫描：nmap -T4 -O ip
```



### 1、SYN扫描

首先可以利用基本的SYN扫描方式探测其端口开放状态。



```
nmap -sS -T4 www.fujieace.com
#扫描输出结果为：

All 997 ports are filtered
PORT STATE SERVICE
80/tcp open http
113/tcp closed auth
507/tcp open crs
```


我们可以看到SYN方式探测到3个端口开放，而有997个端口被过滤。Nmap默认扫描只扫描1000个最可能开放的端口，如果想扫描全部的端口，命令如下：

nmap -sS -T4-p- www.fujieace.com

### 2、FIN扫描

然后可以利用FIN扫描方式探测防火墙状态。FIN扫描方式用于识别端口是否关闭，收到RST回复说明该端口关闭，否则说明是open或filtered状态。



```
nmap -sF -T4 www.fujieace.com
#扫描输出结果为：

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
```


更多端口，此处省略…….

### 3、ACK扫描

然后利用ACK扫描判断端口是否被过滤。针对ACK探测包，未被过滤的端口（无论打开、关闭）会回复RST包。

```
nmap -sA -T4 www.fujieace.com
#扫描输出结果为：

Not shown: 997 unfiltered ports
PORT STATE SERVICE
135/tcp filtered msrpc
1434/tcp filtered ms-sql-m
32777/tcp filtered sometimes-rpc17
```


从结果可以知道997个端口是未被过滤的（unfiltered），而3个（135/1434/32777）被过滤了。所以，将ACK与FIN扫描 的结果结合分析，我们可以找到很多开放的端口。例如7号端口，FIN中得出的状态是:open或filtered，从ACK中得出的状态是 unfiltered，那么该端口只能是open的。

### 4、Window扫描

当然也可以利用Window扫描方式，得出一些端口信息，可以与之前扫描分析的结果相互补充。Window扫描方式只对某些TCPIP协议栈才有效。

window扫描原理与ACK类似，发送ACK包探测目标端口，对回复的RST包中的Window size进行解析。在某些TCPIP协议栈实现中，关闭的端口在RST中会将Window size设置为0；而开放的端口将Window size设置成非0的值。

```
nmap -sW -p- -T4 www.fujieace.com
输出结果：

PORT STATE SERVICE
7/tcp open echo
9/tcp open discard
11/tcp open systat
13/tcp open daytime
```


更多端口，此处省略……
在采用多种

扫描路由器操作系统
nmap -O -F -n 192.168.1.1
与通用PC扫描方式类似，使用-O选项扫描路由器的操作系统。-F用于快速扫描最可能开放的100个端口，并根据端口扫描结果进一步做OS的指纹分析方式获取出防火墙状态后，可以进一步进行应用程序与版本侦测及OS侦测。

nmap [扫描目标] [选项]
v: 增加输出的详细程度。

A: 启用OS检测、版本检测、脚本扫描和traceroute。

p:指定要扫描的端口范围，例如p 1-100。

## Nmap实战

### 1.扫描局域网中存活的主机

举个栗子假设我自己的网络是在192.168.88.0网段，就可以使用以下命令进行ping探测。

```powershell
nmap -sP 192.168.88.* 
#或者
nmap -sP 192.168.88.0/24
```

nmap -sP 192.168.88.* 或者nmap -sP 192.168.88.0/24

### 2.怎么看我的网段

按住win键+R键打开运行窗口

在运行窗口输入cmd回车，打开cmd命令行

输入ipconfig就可以查看到自己的ip地址了哦

### 3.Nmap扫描目标开放的端口和识别服务版本

举个栗子上面扫描时发现好几个ip地址，先扫描个192.168.88.14看看。

```powershell
nmap -sV 192.168.88.1
```

 

## Nmap高级篇

### 1.探查目标服务详细信息并使用相关脚本进行扫描，展示更多的信息。

```shell
nmap -A 192.168.88.14
```

### 2.nmap脚本引擎

**nmap**的脚本默认目录为：/usr/share/nmap/scripts/
类别 解释
**auth** 负责处理鉴权证书（绕开鉴权）的脚本
**broadcast** 在局域网内探查更多服务开启状况，如dhcp/dns/sqlserver等服务
**brute** 提供暴力破解方式，针对常见的应用如http/snmp等
**default** 使用-sC或-A选项扫描时候默认的脚本，提供基本脚本扫描能力
**discovery** 对网络进行更多的信息，如SMB枚举、SNMP查询等
**dos** 用于进行拒绝服务攻击
**exploit** 利用已知的漏洞入侵系统
**external** 利用第三方的数据库或资源，例如进行whois解析
**fuzzer** 模糊测试的脚本，发送异常的包到目标机，探测出潜在漏洞 intrusive:入侵性的脚本，此类脚本可能引发对方的IDS/IPS的记录或屏蔽
**malware** 探测目标机是否感染了病毒、开启了后门等信息
**safe** 此类与intrusive相反，属于安全性脚本
**version** 负责增强服务与版本扫描（Version Detection）功能的脚本
**vuln** 负责检查目标机是否有常见的漏洞（Vulnerability），如是否有MS08_067
举个栗子使用vuln模块进行常见漏洞检测（会有点慢）：

```powershell
nmap –script vuln 192.168.88.14
```



### 3.防火墙逃逸，绕过防火墙扫描。

**报文分段法：**

```powershell
#报文分段法：
nmap -f -v 192.168.88.14
```


在Nmap使用-f选项时会将TCP头分段在几个包中，使得包过滤器、IDS以及其他工具的检测更加困难。Nmap在IP头后会将包分为8个字节或更小。
一些主机会禁止相应ICMP请求，对于这种情况就可以使用报文分段的方法来逃避目标防火墙的规则。

**指定报文大小：**

```powershell
#指定报文大小
nmap –mtu 16 192.168.88.14
```


此值设定TCP/IP协议传输数据报时的最大传输单元。使用指定MTU可以达到逃逸防火墙/IDS的目的，需要注意的是偏移量必须是8的倍数。

**指定发包长度:**

```powershell
#指定发包长度
nmap –data-length 30 192.168.88.14
```


通常情况下，TCP包是40个字节，ICMP Echo 有28个字节，所以在原来的报文基础上附加随机数据达到规避防火墙的效果。

### 4.探测目标防火墙规则

```powershell
nmap –script=firewalk –traceroute 192.168.88.14
```

nmap –script=firewalk –traceroute 192.168.88.14
在Nmap的firewalk脚本通过发送一个请求并分析TTL值，可以探测防火墙的规则。

### 5.VMWare认证破解

```powershell
nmap -p 902 –script vmauthd-brute 192.168.88.14
```


Nmap中的vmauthd-brute脚本可以破解安装虚拟机系统的用户名与密码。

转载自：[渗透武器库—nmap实战总结-CSDN博客](https://blog.csdn.net/tangshuangsss/article/details/111354476)


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/d59e8d0/  

