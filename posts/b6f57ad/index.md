# 在 Ubuntu 中修改 IP 地址


<!--more-->

**从 ubuntu 18.04 开始，修改使用 netplan 方式，新版本也不例外。**

```shell
编辑网络配置文件（旧系统直接修改/etc/network/interfaces/....）
使用文本编辑器（如vi,vim）打开网络配置文件：
cd /etc/netplan
 
sudo vi 50-cloud-init.yaml
```

要将其修改为静态 IP，配置可能如下（示例 IP 地址为192.168.1.100，子网掩码为24位，网关为192.168.1.1，DNS 服务器为192.168.1.1）：

\#一个标准配置

```shell
network:
    renderer: networkd
    ethernets:
        ens33: # 替换为你的网络接口名称
            dhcp4: false # 关闭 DHCP
            dhcp6: false # 关闭 DHCP
            addresses: [192.168.100.122/24] # 静态 IP 地址和子网掩码
            routes:
              - to: default
                via: 192.168.100.1 # 网关地址
            nameservers:
                addresses: [192.168.100.1] # DNS 服务器地址
                search: []
    version: 2
 
#一个标准配置
# This is the network config written by 'user'
network:
   ethernets:
   enp0s3:
       dhcp4: false
       addresses:
         - 10.0.2.15/24
       routes:
         - to: default
         via: 10.0.2.2
      nameservers:
         addresses: 
         - 114.114.114.114
         - 8.8.8.8
version: 2
```

应用新的网络配置 保存配置文件后，使用以下命令使新配置生效：

```shell
sudo netplan apply
DNSPod的 Public DNS+是目前国内第一家支持ECS的公共DNS，是DNSPod推出的公共域名解析服务，可以为全网用户提供域名的公共递归解析服务！
DNS 服务器 IP 地址：
首选：119.29.29.29
备选：182.254.116.116
阿里公共DNS是阿里巴巴集团推出的DNS递归解析系统，目标是成为国内互联网基础设施的组成部分，面向互联网用户提供“快速”、“稳定”、“智能”的免费DNS递归解析服务。
DNS 服务器 IP 地址：
首选：223.5.5.5
备选：223.6.6.6
```


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/b6f57ad/  

