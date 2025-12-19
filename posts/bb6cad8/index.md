# Wifi密码破解分析的几种常见方式


<!--more-->

## 一，跑字典，暴力穷举

***\*aircrack-ng破解wifi密码\****

[IYATT-yx/WiFi-Password-Dictionary: WiFi 密码字典 (github.com)](https://github.com/IYATT-yx/WiFi-Password-Dictionary?tab=readme-ov-file)

[aircrack-ng破解wifi密码【黑客渗透测试零基础入门必知必会】-CSDN博客](https://blog.csdn.net/qq_41314882/article/details/142614793)

 

***\*wifi-crack-tool WiFi密码暴破工具 wifi破解\****

[wifi-crack-tool WiFi密码暴破工具 wifi破解 – 🔰雨苁ℒ🔰 (ddosi.org)](https://jiangbos.top/index.php/2024/10/07/wifi密码破解分析的几种常见方式/106/#google_vignette)

 

### 爆破方案

1.连接尝试：https://blog.iyatt.com/?p=10615
这种方案速度较慢，相当于一个一个去连接尝试

2.抓包匹配：https://blog.iyatt.com/?p=10677
锁定目标 WiFi 扫描，攻击干扰 WiFi 让处于连接状态的设备断开，设备尝试重新连接时会发送包含加密密码的握手包，抓取这个握手包，再用字典中的密码尝试按照相同方式加密去匹配，速度完虐方案 1。

## 二，钓鱼

### **主要原理：**

***\*创建一个伪AP来“狸猫换太子”，然后撤销用户AP的授权，\****

***\*通知用户需要进行“固件升级”，需要重新验证密码。你的假AP由于具有相同的SSID，用户便会“交代”密码。\****

***\*这样你就能得到用户的密码，并且让用户采用你的伪AP做为自己的接入点。而对方一无所知。\****

[四步教你破解隔壁老王的Wi-Fi密码，蹭网没商量！_分分钟解开邻居家的wifi-CSDN博客](https://blog.csdn.net/u012889638/article/details/49174965)

[渗透测试之地基服务篇：无线攻防之钓鱼无线攻击（上） – FreeBuf网络安全行业门户](https://www.freebuf.com/articles/wireless/328243.html)

 **Lindset**的高级社会工程攻击演变而来，**Fluxion**的操作简单只需要选择你需要钓的鱼其他的你基本就不用管了。

工作原理：

1、扫描网络

2、捕获握手（不能没有有效的握手使用，需要验证密码）

3、使用WEB界面

4、启动 FakeAP 实例来模拟原始接入点

5、生成一个 MDK3 进程，该进程使所有已经连接到目标网络的用户连接失效，然后引诱他们连接到 FakeAP 并输入 WPA 的密码

6、启动一个虚假的 DNS 服务器，抓取所有的 DNS 请求并将其重定向到运行该脚本的主机

7、启动强制访问的网站页面，并提示用户输入他们的 WPA 密码

8、每个提交的密码都通过之前捕获的握手进行验证

9、只要提交了正确的密码，攻击就会自动终止

 

[Wifi钓鱼工具——fluxion – freeliver – 博客园 (cnblogs.com)](https://www.cnblogs.com/cmt110/p/15125418.html)

（Kali Linux和两个无线适配器，其中一个必须能支持数据包注入。）


---

> 作者: <no value>  
> URL: http://localhost:1313/posts/bb6cad8/  

