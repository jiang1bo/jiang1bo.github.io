# Openeuler系统管理员指南


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

# 基础配置 

## 设置语言环境 

您可以通过localectl修改系统的语言环境，对应的参数设置保存在/etc/locale.conf文件中。这些参数会在系统启动过程中被systemd的守护进程读取。

### 显示当前语言环境状态 

显示当前语言环境，命令如下：

```
localectl status
```

例如显示系统当前的设置，命令和输出如下：

```
# localectl status
   System Locale: LANG=zh_CN.UTF-8
       VC Keymap: cn
      X11 Layout: cn
```

### 列出可用的语言环境

显示当前可用的语言环境，命令如下：

```
# localectl list-locales
```

例如显示当前系统中所有可用的中文环境，命令和输出如下：

```
# localectl list-locales | grep zh
zh_CN.UTF-8
```

### 设置语言环境 

要设置语言环境，在root权限下执行如下命令，其中 *locale* 是您要设置的语言类型，取值范围可通过**localectl list-locales**获取，请根据实际情况修改。

```
# localectl set-locale LANG=locale
```

例如设置为简体中文语言环境，在root权限下执行如下命令：

```
# localectl set-locale LANG=zh_CN.UTF-8
```

说明：

修改后需要重新登录或者在root权限下执行`source /etc/locale.conf`命令刷新配置文件，使修改生效。

## 设置键盘 

您可以通过localectl修改系统的键盘设置，对应的参数设置保存在/etc/locale.conf文件中。这些参数，会在系统启动的早期被systemd的守护进程读取。

### 显示当前设置 

显示当前键盘设置，命令如下：

```
# localectl status
```

例如显示系统当前的设置，命令和输出如下：

```
# localectl status
   System Locale: LANG=zh_CN.UTF-8
       VC Keymap: cn
      X11 Layout: cn
```

### 列出可用的键盘布局 

显示当前可用的键盘布局，命令如下：

```
# localectl list-keymaps
```

例如显示系统当前的中文键盘布局，命令和输出如下：

```
# localectl list-keymaps | grep cn
cn
```

### 设置键盘布局 

设置键盘布局，在root权限下执行如下命令，其中 *map* 是您想要设置的键盘类型，取值范围可通过**localectl list-keymaps**获取，请根据实际情况修改：



```
# localectl set-keymap map
```

此时设置的键盘布局同样也会应用到图形界面中。

设置完成后，查看当前状态：



```
# localectl status
   System Locale: LANG=zh_CN.UTF-8
       VC Keymap: cn
      X11 Layout: us
```

## 设置日期和时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-设置日期和时间)

本节介绍如何通过timedatectl、date、hwclock命令来设置系统的日期、时间和时区等。

### 使用timedatectl命令设置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-使用timedatectl命令设置)

#### 显示日期和时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-显示日期和时间)

显示当前的日期和时间，命令如下：



```
# timedatectl
```

例如显示系统当前的日期和时间，命令和输出如下：



```
# timedatectl
               Local time: Mon 2019-09-30 04:05:00 EDT
           Universal time: Mon 2019-09-30 08:05:00 UTC
                 RTC time: Mon 2019-09-30 08:05:00
                Time zone: China Standard Time (CST), UTC +8
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

#### 通过远程服务器进行时间同步 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-通过远程服务器进行时间同步)

您可以启用NTP远程服务器进行系统时钟的自动同步。是否启用NTP，可在root权限下执行如下命令进行设置。其中 *boolean* 可取值yes和no，分别表示启用和不启用NTP进行系统时钟自动同步，请根据实际情况修改。

说明：

若启用了NTP远程服务器进行系统时钟自动同步，则不能手动修改日期和时间。若需要手动修改日期或时间，则需确保已经关闭NTP系统时钟自动同步。可执行`timedatectl set-ntp no`命令进行关闭。

例如开启自动远程时间同步，命令如下：



```
# timedatectl set-ntp yes
```

#### 修改日期 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-修改日期)

说明：

修改日期前，请确保已经关闭NTP系统时钟自动同步。

修改当前的日期，在root权限下执行如下命令，其中 *YYYY* 代表年份，*MM* 代表月份，*DD* 代表某天，请根据实际情况修改：



```
# timedatectl set-time YYYY-MM-DD
```

例如修改当前的日期为2019年8月14号，命令如下：



```
# timedatectl set-time '2019-08-14'
```

#### 修改时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-修改时间)

说明：

修改时间前，请确保已经关闭NTP系统时钟自动同步。

修改当前的时间，在root权限下执行如下命令，其中 *HH* 代表小时，*MM* 代表分钟，*SS* 代表秒，请根据实际情况修改：



```
# timedatectl set-time HH:MM:SS
```

例如修改当前的时间为15点57分24秒，命令如下：



```
# timedatectl set-time 15:57:24
```

#### 修改时区 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-修改时区)

显示当前可用时区，命令如下：



```
timedatectl list-timezones
```

要修改当前的时区，在root权限下执行如下命令，其中 *time_zone* 是您想要设置的时区，请根据实际情况修改：



```
# timedatectl set-timezone time_zone
```

例如修改当前的时区，首先查询所在地域的可用时区，此处以Asia为例：



```
# timedatectl list-timezones | grep Asia
Asia/Aden
Asia/Almaty
Asia/Amman
Asia/Anadyr
Asia/Aqtau
Asia/Aqtobe
Asia/Ashgabat
Asia/Baghdad
Asia/Bahrain
……

Asia/Seoul
Asia/Shanghai
Asia/Singapore
Asia/Srednekolymsk
Asia/Taipei
Asia/Tashkent
Asia/Tbilisi
Asia/Tehran
Asia/Thimphu
Asia/Tokyo
```

然后修改当前的时区为“Asia/Shanghai”，命令如下：



```
# timedatectl set-timezone Asia/Shanghai
```

### 使用date命令设置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-使用date命令设置)

#### 显示当前的日期和时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-显示当前的日期和时间)

显示当前的日期和时间，命令如下：



```
# date
```

默认情况下，date命令显示本地时间。要显示UTC时间，添加--utc或-u参数：



```
# date --utc
```

要自定义对应的输出信息格式，添加 +"format" 参数：



```
# date +"format"
```

**表 1** 参数说明



| 格式参数 | 说明                                                       |
| :------- | :--------------------------------------------------------- |
| %H       | 小时以HH格式（例如 17）。                                  |
| %M       | 分钟以MM格式（例如 37）。                                  |
| %S       | 秒以SS格式（例如 25）。                                    |
| %d       | 日期以DD格式（例如 15）。                                  |
| %m       | 月份以MM格式（例如 07）。                                  |
| %Y       | 年份以YYYY格式（例如 2019）。                              |
| %Z       | 时区缩写（例如CEST）。                                     |
| %F       | 日期整体格式为YYYY-MM-DD（例如 2019-7-15），等同%Y-%m-%d。 |
| %T       | 时间整体格式为HH:MM:SS（例如 18:30:25），等同%H:%M:%S。    |

实际使用示例如下：

- 显示当前的日期和本地时间。

  

  ```
  # date 
  2019年 08月 17日 星期六 17:26:34 CST
  ```

- 显示当前的日期和UTC时间。

  

  ```
  # date --utc
  2019年 08月 17日 星期六 09:26:18 UTC
  ```

- 自定义date命令的输出。

  

  ```
  # date +"%Y-%m-%d %H:%M"
  2019-08-17 17:24
  ```

#### 修改时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-修改时间-1)

要修改当前的时间，添加--set或者-s参数。在root权限下执行如下命令，其中 *HH* 代表小时，*MM* 代表分钟，*SS* 代表秒，请根据实际情况修改：



```
# date --set HH:MM:SS
```

默认情况下， date命令设置本地时间。要设置UTC时间，添加--utc或-u参数：



```
# date --set HH:MM:SS --utc
```

例如修改当前的时间为23点26分00秒，在root权限下执行如下命令：



```
# date --set 23:26:00
```

#### 修改日期 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-修改日期-1)

修改当前的日期，添加--set或者-s参数。在root权限下执行如下命令，其中 *YYYY* 代表年份，*MM* 代表月份，*DD* 代表某天，请根据实际情况修改：



```
# date --set YYYY-MM-DD
```

例如修改当前的日期为2019年11月2日，命令如下：



```
# date --set 2019-11-02
```

### 使用hwclock命令设置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-使用hwclock命令设置)

可以使用 hwclock 命令设置硬件时钟RTC (Real Time Clock) 。

#### 硬件时钟和系统时钟 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-硬件时钟和系统时钟)

Linux 将时钟分为：

- 系统时钟 (System Clock) ：当前Linux Kernel中的时钟。
- 硬件时钟 RTC：主板上由电池供电的主板硬件时钟，该时钟可以在BIOS的 "Standard BIOS Feature" 项中进行设置。

当Linux启动时，会读取硬件时钟，并根据硬件时间来设置系统时间。

#### 显示日期和时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-显示日期和时间-1)

显示当前硬件的日期和时间，在root权限下执行如下命令：



```
# hwclock
```

例如显示当前硬件的日期和时间，命令和输出如下：



```
# hwclock
2019-08-26 10:18:42.528948+08:00
```

#### 设置日期和时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-设置日期和时间-1)

修改当前硬件的日期和时间，在root权限下执行如下命令，其中 *dd* 表示日，*mm* 表示月份，*yyyy* 表示年份，*HH* 表示小时，*MM* 表示分钟，请根据实际情况修改：



```
# hwclock --set --date "yyyy-mm-dd HH:MM"
```

例如修改当前的时间为2019年10月21日21点17分，命令如下：



```
# hwclock --set --date "2019-10-21 21:17"
```

## 设置kdump [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-设置kdump)

本节介绍如何设置kdump预留内存及修改kdump配置文件参数。

### 设置kdump预留内存 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-设置kdump预留内存)

#### 预留内存参数格式 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-预留内存参数格式)

kdump预留内存参数必须添加到内核启动参数中，配置文件为/boot/efi/EFI/openEuler/grub.cfg（UEFI引导模式）或/boot/grub2/grub.cfg（legacy引导模式），openEuler发布版本中默认已经添加，可以根据实际使用情况调整。添加和修改启动参数后，重启系统生效。kdump预留内存参数格式如下：

| 内核启动参数                         | 描述                                                         | 缺省值                      | 备注                                                         |
| :----------------------------------- | :----------------------------------------------------------- | :-------------------------- | :----------------------------------------------------------- |
| crashkernel=x                        | 在4G以下的物理内存预留x大小的内存给kdump使用。               | x86版本默认配置512M         | 该配置方法只在4G以下内存预留，必须保证4G以下连续可用内存足够预留。 |
| crashkernel=x@y                      | 在y起始地址预留x大小的内存给kdump使用。                      | 未使用                      | 需要确保y起始地址的x大小的内存未被其他模块预留。             |
| crashkernel=x,high                   | 在4G以下的物理内存中预留256M内存，在4G以上预留x大小内存给kdump使用。 | arm64版本默认配置1024M,high | 需要确保4G以下有256M连续可用的物理内存，4G以上有连续的x大小的连续物理内存。实际预留内存大小为256M+x。 |
| crashkernel=x,low crashkernel=y,high | 在4G以下的物理内存中预留x大小，在4G以上预留y大小内存给kdump使用。 | 未使用                      | 需要确保4G以下有连续的x大小物理内存，4G以上有连续的y大小物理内存。 |

### 预留内存推荐值 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-预留内存推荐值)

| 推荐方案 | 预留参数               | 参数说明                                                     |
| :------- | :--------------------- | :----------------------------------------------------------- |
| 通用方案 | crashkernel=2048M,high | 4G以下预留256M，4G以上预留2048M内存给kdump使用。共256+2048M。 |
| 经济方案 | crashkernel=1024M,high | 4G以下预留256M，4G以上预留1024M内存给kdump使用。共256+1024M。 推荐系统512M内存以内的场景，并不使用网络转储kdump文件。对于虚拟机场景，可以适当减少内存预留值，推荐虚拟机设置为crashkernel=512M或者crashkernel=256M,high |

说明：

不通过网络转储kdump文件时，需要设置kdump文件系统不打包网络相关驱动。网络驱动加载需要申请较大内存，可能导致预留内存不足，kdump失败。因此建议禁用网络相关驱动。

### 禁用网络相关驱动 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-禁用网络相关驱动)

kdump配置文件（/etc/kdump.conf）中，dracut参数可以设置裁剪的驱动模块，可以将网络驱动配置到裁剪驱动列表中，让kdump文件系统中不加载该驱动，修改配置文件后，重启kdump服务生效。dracut参数配置如下所示：

```
dracut_args --omit-drivers "mdio-gpi usb_8dev et1011c rt2x00usb bcm-phy-lib mac80211_hwsim rtl8723be rndis_host hns3_cae amd vrf rtl8192cu mt76x02-lib int51x1 ppp_deflate team_mode_loadbalance smsc911x aweth bonding mwifiex_usb hnae dnet rt2x00pci vaser_pci hdlc_ppp marvell rtl8xxxu mlxsw_i2c ath9k_htc rtl8150 smc91x cortina at803x rockchip cxgb4 spi_ks8995 mt76x2u smsc9420 mdio-cavium bnxt_en ch9200 dummy macsec ice mt7601u rtl8188ee ixgbevf net1080 liquidio_vf be2net mlxsw_switchx2 gl620a xilinx_gmii2rgmii ppp_generic rtl8192de sja1000_platform ath10k_core cc770_platform realte igb c_can_platform c_can ethoc dm9601 smsc95xx lg-vl600 ifb enic ath9 mdio-octeon ppp_mppe ath10k_pci cc770 team_mode_activebackup marvell10g hinic rt2x00lib mlx4_en iavf broadcom igc c_can_pci alx rtl8192se rtl8723ae microchip lan78xx atl1c rtl8192c-common almia ax88179_178a qed netxen_nic brcmsmac rt2800usb e1000 qla3xxx mdio-bitbang qsemi mdio-mscc-miim plx_pci ipvlan r8152 cx82310_eth slhc mt76x02-usb ems_pci xen-netfront usbnet pppoe mlxsw_minimal mlxsw_spectrum cdc_ncm rt2800lib rtl_usb hnae3 ath9k_common ath9k_hw catc mt76 hns_enet_drv ppp_async huawei_cdc_ncm i40e rtl8192ce dl2 qmi_wwan mii peak_usb plusb can-dev slcan amd-xgbe team_mode_roundrobin ste10Xp thunder_xcv pptp thunder_bgx ixgbe davicom icplus tap tun smsc75xx smsc dlci hns_dsaf mlxsw_core rt2800mmi softing uPD60620 vaser_usb dp83867 brcmfmac mwifiex_pcie mlx4_core micrel team macvlan bnx2 virtio_net rtl_pci zaurus hns_mdi libcxgb hv_netvsc nicvf mt76x0u teranetics mlxfw cdc_eem qcom-emac pppox mt76-usb sierra_net i40evf bcm87xx mwifiex pegasus rt2x00mmi sja1000 ena hclgevf cnic cxgb4vf ppp_synctty iwlmvm team_mode_broadcast vxlan vsockmon hdlc_cisc rtl8723-common bsd_comp fakelb dp83822 dp83tc811 cicada fm10 8139t sfc hs geneve hclge xgene-enet-v2 cdc_mbim hdlc asix netdevsim rt2800pci team_mode_random lxt ems_usb mlxsw_pci sr9700 mdio-thunder mlxsw_switchib macvtap atlantic cdc_ether mcs7830 nicpf mdi peak_pci atl1e cdc_subset ipvtap btcoexist mt76x0-common veth slip iwldvm bcm7xxx vitesse netconsole epic100 myri10ge r8169 qede microchip_t1 liquidi bnx2x brcmutil mwifiex_sdi mlx5_core rtlwifi vmxnet3 nlmon hns3 hdlc_raw esd_usb2 atl2 mt76x2-common iwlwifi mdio-bcm-unimac national ath rtwpci rtw88 nfp rtl8821ae fjes thunderbolt-net 8139cp atl1 mscc vcan dp83848 dp83640 hdlc_fr e1000e ipheth net_failover aquantia rtl8192ee igbvf rocker intel-xway tg3" --omit "ramdisk network ifcfg qemu-net" --install "chmod" --nofscks
```

## 设置磁盘调度算法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-设置磁盘调度算法)

本节介绍如何设置磁盘调度算法。

### 临时修改调度策略 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-临时修改调度策略)

例如将所有IO调度算法修改为mq-deadline，此修改重启后会失效。



```
# echo mq-deadline > /sys/block/sd*/queue/scheduler
```

### 永久设置调度策略 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-永久设置调度策略)

可以通过在内核启动配置文件grub.cfg中的kernel行追加：elevator=mq-deadline，重启后生效。



```
linux   /vmlinuz-4.19.90-2003.4.0.0036.oe1.x86_64 root=/dev/mapper/openeuler-root ro resume=/dev/mapper/openeuler-swap rd.lvm.lv=openeuler/root rd.lvm.lv=openeuler/swap quiet crashkernel=512M elevator=mq-deadline
```

## 设置NMI watchdog [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-设置nmi-watchdog)

本节介绍openEuler在arm64架构上NMI watchdog方案的差异以及配置。

### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-概述)

NMI watchdog（Hard lockup detector）是一种用来检测系统是否出现Hard lockup（硬死锁）的机制。一般的watchdog依赖时钟中断进行挂死检测，当系统在原子上下文（中断，或者中断关闭的上下文中，etc）中出现挂死时，时钟中断处理，检测失效。NMI watchdog一般通过PMC（或者PMU）的NMI中断进行检测，NMI中断可以在原子上下文中产生并处理，因此可以用来检测原子上下文中挂死的场景。

NMI watchdog主线已经支持，当硬件满足以下条件时可以使能NMI watchdog：

1. 支持NMI中断
2. 支持PMC（PMU）

在arm64上，openEuler基于arm64的SDEI功能实现了SDEI watchdog作为NMI watchdog。因此openEuler在arm64上存在2种NMI watchdog方案：

1. SDEI watchdog（默认方式）
2. 基于PMC（PMU）中断的NMI watchdog

### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-注意事项)

对于arm64机器，需要注意以下事项：

- 默认情况下使用SDEI watchdog。当SDEI watchdog使能失败时，不会切换到NMI watchdog
- 需要使用NMI watchdog时，需要显式的在启动参数中禁用SDEI watchdog：disable_sdei_nmi_watchdog
- 当需要使用NMI watchdog时，需要保证硬件支持NMI中断：
  - 当硬件支持NMI中断时，不需要额外处理
  - 当硬件不支持NMI中断，但是支持伪NMI中断时，需要显式的在启动参数中使能伪NMI中断：irqchip.gicv3_pseudo_nmi=1

以上事项不影响非arm64平台。

### 操作步骤 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-操作步骤)

针对arm64架构配置NMI watchdog的操作步骤如下：

1. 在OS的引导配置文件grub.cfg中添加如下参数：irqchip.gicv3_pseudo_nmi=1（仅通过Pseudo-NMI实现NMI watchdog时添加） disable_sdei_nmi_watchdog
2. 检查NMI watchdog是否加载成功，如果加载成功，内核dmesg日志打印类似如下内容



```
[   11.361889][  T129] NMI watchdog: Enabled. Permanently consumes one hw-PMU counter.
```

### 关闭NMI watchdog [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-关闭nmi-watchdog)

将NMI watchdog临时关闭，此修改重启后会失效；默认nmi_watchdog=1。



```
#  echo 0 > /proc/sys/kernel/nmi_watchdog
```

在OS启动时，可以通过配置内核参数nmi_watchdog=0关闭NMI watchdog。

### 修改NMI watchdog阈值 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/basic_configuration.html#user-content-修改nmi-watchdog阈值)

修改NMI watchdog阈值，此修改重启后会失效；默认watchdog_thresh=10。



```
#  echo 10 > /proc/sys/kernel/watchdog_thresh
```

在OS启动时，可以通过配置内核参数watchdog_thresh=[0-60]修改阈值。

# 管理用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-管理用户)

在Linux中，每个普通用户都有一个帐户，包括用户名、密码和主目录等信息。除此之外，还有一些系统本身创建的特殊用户，它们具有特殊的意义，其中最重要的是管理员帐户，默认用户名是root。同时Linux也提供了用户组，使每一个用户至少属于一个组，从而便于权限管理。

用户和用户组管理是系统安全管理的重要组成部分，本章主要介绍openEuler提供的用户管理和组管理命令，以及为普通用户分配特权的方法。

## 管理用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-管理用户-1)

### 增加用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-增加用户)

#### useradd命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-useradd命令)

在root权限下，通过useradd命令可以为系统添加新用户信息，其中 *options* 为相关参数， *username* 为用户名称。



```
useradd [options] username
```

#### 用户信息文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-用户信息文件)

与用户帐号信息有关的文件如下：

- /etc/passwd：用户帐号信息文件。
- /etc/shadow：用户帐号信息加密文件。
- /etc/group：组信息文件。
- /etc/default/useradd：定义默认设置文件。
- /etc/login.defs：系统广义设置文件。
- /etc/skel：默认的初始配置文件目录。

#### 创建用户实例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-创建用户实例)

例如新建一个用户名为userexample的用户，在root权限下执行如下命令：



```
# useradd userexample
```

说明：

没有任何提示，表明用户建立成功。这时并没有设置用户的口令，请使用passwd命令修改用户的密码，没有设置密码的新帐号不能登录系统。

使用id命令查看新建的用户信息，命令如下：



```
# id userexample
uid=1001(userexample)    gid=1001(userexample)    groups=1001(userexample)
```

修改用户userexample的密码：



```
# passwd userexample
```

建议在修改用户密码时满足密码复杂度要求，密码的复杂度的要求如下：

1. 口令长度至少8个字符。

2. 口令至少包含大写字母、小写字母、数字和特殊字符中的任意3种。

3. 口令不能和帐号一样。

4. 口令不能使用字典词汇。

   - 查询字典 在已装好的openEuler环境中，可以通过如下命令导出字典库文件dictionary.txt，用户可以查询密码是否在该字典中。

     

     ```
     cracklib-unpacker /usr/share/cracklib/pw_dict > dictionary.txt
     ```

   - 修改字典

     1. 修改上面导出的字典文件，执行如下命令更新系统字典库。

        

        ```
        # create-cracklib-dict dictionary.txt
        ```

     2. 在原字典库基础上新增其他字典内容custom.txt。

        

        ```
        # create-cracklib-dict dictionary.txt custom.txt
        ```

根据提示两次输入新用户的密码，完成密码更改。过程如下：



```
# passwd userexample
Changing password for user userexample.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

说明：

若打印信息中出现“BAD PASSWORD: The password fails the dictionary check - it is too simplistic/systematic”，表示设置的密码过于简单，建议设置复杂度较高的密码。

### 修改用户帐号 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改用户帐号)

#### 修改密码 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改密码)

普通用户可以用passwd修改自己的密码，只有管理员才能用passwd username为其他用户修改密码。

#### 修改用户shell设置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改用户shell设置)

使用chsh命令可以修改自己的shell，只有管理员才能用chsh username为其他用户修改shell设置。

用户也可以使用usermod命令修改shell信息，在root权限下执行如下命令，其中 *new_shell_path* 为目标shell路径，*username* 为要修改用户的用户名，请根据实际情况修改：



```
usermod -s new_shell_path username
```

例如，将用户userexample的shell改为csh，命令如下：



```
# usermod -s /bin/csh userexample
```

#### 修改主目录 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改主目录)

- 修改主目录，可以在root权限下执行如下命令，其中 *new_home_directory* 为已创建的目标主目录的路径，*username* 为要修改用户的用户名，请根据实际情况修改：

  

  ```
  usermod -d new_home_directory username
  ```

- 如果想将现有主目录的内容转移到新的目录，应该使用-m选项，命令如下：

  

  ```
  usermod -d new_home_directory -m username
  ```

#### 修改UID [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改uid)

修改用户ID，在root权限下执行如下命令，其中 *UID* 代表目标用户ID，*username* 代表用户名，请根据实际情况修改：



```
usermod -u UID username
```

该用户主目录中所拥有的文件和目录都将自动修改UID设置。但是，对于主目录外所拥有的文件，只能使用chown命令手动修改所有权。

#### 修改帐号的有效期 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改帐号的有效期)

如果使用了影子口令，则可以在root权限下，执行如下命令来修改一个帐号的有效期，其中 *MM* 代表月份，*DD* 代表某天，*YY* 代表年份，*username* 代表用户名，请根据实际情况修改：



```
usermod -e MM/DD/YY username
```

### 删除用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-删除用户)

在root权限下，使用userdel命令可删除现有用户。

例如，删除用户Test，命令如下：



```
# userdel Test
```

如果想同时删除该用户的主目录以及其中所有内容，要使用-r参数递归删除。

说明：

不建议直接删除已经进入系统的用户，如果需要强制删除，请使用 userdel -f *Test* 命令。

### 管理员帐户授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-管理员帐户授权)

使用sudo命令可以允许普通用户执行管理员帐户才能执行的命令。

sudo命令允许已经在/etc/sudoers文件中指定的用户运行管理员帐户命令。例如，一个已经获得许可的普通用户可以运行如下命令：



```
sudo /usr/sbin/useradd newuserl
```

实际上，sudo的配置完全可以指定某个已经列入/etc/sudoers文件的普通用户可以做什么，不可以做什么。

/etc/sudoers的配置行如下所示。

- 空行或注释行（以#字符打头）：无具体功能的行。

- 可选的主机别名行：用来创建主机列表的简称。必须以Host_Alias关键词开头，列表中的主机必须用逗号隔开，如：

  

  ```
  Host_Alias  linux=ted1,ted2
  ```

  其中ted1和ted2是两个主机名，可使用linux（别名）称呼它们。

- 可选的用户别名行：用来创建用户列表的简称。用户别名行必须以User_Alias关键词开头，列表中的用户名必须以逗号隔开。其格式同主机别名行。

- 可选的命令别名行：用来创建命令列表的简称。必须以Cmnd_Alias开头，列表中的命令必须用逗号隔开。

- 可选的运行方式别名行：用来创建用户列表的简称。不同的是，使用这样的别名可以告诉sudo程序以列表中某一用户的身份来运行程序。

- 必要的用户访问说明行。

  用户访问的说明语法如下：

  

  ```
  user host = [ run as user ] command list
  ```

  在user处指定一个真正的用户名或定义过的别名，host也可以是一个真正的主机名或者定义过的主机别名。默认情况下，sudo执行的所有命令都是以root身份执行。如果您想使用其他身份可以指定。command list可以是以逗号分隔的命令列表，也可以是一个已经定义过的别名，如：

  

  ```
  ted1   ted2=/sbin/shutdown
  ```

  这一句说明ted1可以在ted2主机上运行关机命令。

  

  ```
  newuser1 ted1=(root) /usr/sbin/useradd,/usr/sbin/userdel
  ```

  这一句说明ted1主机上的newuser1具有以root用户权限执行useradd，userdel命令的功能。

  说明：

  

  - 可以在一行定义多个别名，中间用冒号 (:) 隔开。
  - 可在命令或命令别名之前加上感叹号 (!)，使该命令或命令别名无效。
  - 有两个关键词：ALL和NOPASSWD。ALL意味着“所有”（所有文件、所有主机或所有命令），NOPASSWD意味着不用密码。
  - 通过修改用户访问，将普通用户的访问权限修改为同root一样，则可以给普通用户分配特权。

下面是一个sudoers文件的例子：



```
#sudoers files
#User alias specification
User_Alias ADMIN=ted1:POWERUSER=globus,ted2
#user privilege specification
ADMIN ALL=ALL
POWERUSER ALL=ALL,!/bin/su
```

其中：

- User_Alias ADMIN=ted1:POWERUSER=globus,ted2

  定义了两个别名ADMIN和POWERUSER

- ADMIN ALL=ALL

  说明在所有主机上，ADMIN用户都可以以root身份执行所有命令

- POWERUSER ALL=ALL,!/bin/su

  给POWERUSER用户除了运行su命令外等同ADMIN的权限

## 管理用户组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-管理用户组)

### 增加用户组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-增加用户组)

#### groupadd命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-groupadd命令)

在root权限下，通过groupadd命令可以为系统添加新用户组信息，其中 *options* 为相关参数， *groupname* 为用户组名称。



```
groupadd [options] groupname
```

#### 用户组信息文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-用户组信息文件)

与用户组信息有关的文件如下：

- /etc/gshadow：用户组信息加密文件。
- /etc/group：组信息文件。
- /etc/login.defs：系统广义设置文件。

#### 创建用户组实例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-创建用户组实例)

例如新建一个用户组名为groupexample的用户，在root权限下执行如下命令：



```
# groupadd groupexample
```

### 修改用户组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改用户组)

#### 修改GID [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改gid)

修改用户组ID，在root权限下执行如下命令，其中 *GID* 代表目标用户组ID， *groupname* 代表用户组，请根据实际情况修改：



```
# groupmod -g GID groupname
```

#### 修改用户组名 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-修改用户组名)

修改用户组名，在root权限下执行如下命令，其中 *newgroupname* 代表新用户组名， *oldgroupname* 代表已经存在的待修改的用户组名，请根据实际情况修改：



```
# groupmod -n newgroupname oldgroupname
```

### 删除用户组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-删除用户组)

在root权限下，使用groupdel命令可删除用户组。

例如，删除用户组Test，命令如下：



```
# groupdel Test
```

说明：

groupdel不能直接删除用户的主组，如果需要强制删除用户主组，请使用 groupdel -f *Test* 命令。

### 将用户加入用户组或从用户组中移除 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-将用户加入用户组或从用户组中移除)

在root权限下，使用gpasswd命令将用户加入用户组或从用户组中移除。

例如，将用户 *userexample* 加入用户组 *Test* ，命令如下：



```
# gpasswd -a userexample Test
```

例如，将用户 *userexample* 从 *Test* 用户组中移除，命令如下：



```
# gpasswd -d userexample Test
```

### 切换用户组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/user_and_user_group_management.html#user-content-切换用户组)

一个用户同时属于多个用户组时，则在用户登录后，使用newgrp命令可以切换到其他用户组，以便具有其他用户组的权限。

例如，将用户 *userexample* 切换到 *Test* 用户组，命令如下：



```
newgrp Test
```

# 使用DNF管理软件包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-使用dnf管理软件包)

DNF是一款Linux软件包管理工具，用于管理RPM软件包。DNF可以查询软件包信息，从指定软件库获取软件包，自动处理依赖关系以安装或卸载软件包，以及更新系统到最新可用版本。

说明：



- DNF与YUM完全兼容，提供了YUM兼容的命令行以及为扩展和插件提供的API。
- 使用DNF需要管理员权限，本章所有命令需要在管理员权限下执行。

## 配置DNF [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-配置dnf)

### DNF配置文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-dnf配置文件)

DNF 的主要配置文件是 /etc/dnf/dnf.conf，该文件包含两部分：

- “main”部分保存着DNF的全局设置。
- “repository”部分保存着软件源的设置，可以有零个或多个“repository”。

另外，在/etc/yum.repos.d 目录中保存着零个或多个repo源相关文件，它们也可以定义不同的“repository”。

所以openEuler软件源的配置一般有两种方式，一种是直接配置/etc/dnf/dnf.conf文件中的“repository”部分，另外一种是在/etc/yum.repos.d目录下增加.repo文件。

#### 配置main部分 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-配置main部分)

/etc/dnf/dnf.conf 文件包含的“main”部分，配置示例如下：



```
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
```

常用选项说明：

**表 1** main参数说明



| 参数                         | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| cachedir                     | 缓存目录，该目录用于存储RPM包和数据库文件。                  |
| keepcache                    | 可选值是1和0，表示是否要缓存已安装成功的那些RPM包及头文件，缺省值为0，即不缓存。 |
| debuglevel                   | 设置dnf生成的debug信息。取值范围：[0-10]，数值越大会输出越详细的debug信息。缺省值为2，设置为0表示不输出debug信息。 |
| clean_requirements_on_remove | 删除在dnf remove期间不再使用的依赖项，如果软件包是通过DNF安装的，而不是通过显式用户请求安装的，则只能通过clean_requirements_on_remove删除软件包，即它是作为依赖项引入的。 缺省值为True。 |
| best                         | 升级包时，总是尝试安装其最高版本，如果最高版本无法安装，则提示无法安装的原因并停止安装。缺省值为True。 |
| obsoletes                    | 可选值1和0，设置是否允许更新陈旧的RPM包。缺省值为1，表示允许更新。 |
| gpgcheck                     | 可选值1和0，设置是否进行gpg校验。缺省值为1，表示需要进行校验。 |
| plugins                      | 可选值1和0，表示启用或禁用dnf插件。缺省值为1，表示启用dnf插件。 |
| installonly_limit            | 设置可以同时安装“installonlypkgs”指令列出包的数量。缺省值为3，不建议降低此值。 |

#### 配置repository部分 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-配置repository部分)

repository部分允许您定义定制化的openEuler软件源仓库，各个仓库的名称不能相同，否则会引起冲突。配置repository部分有两种方式，一种是直接配置/etc/dnf/dnf.conf文件中的“repository”部分，另外一种是配置/etc/yum.repos.d目录下的.repo文件。

- 直接配置/etc/dnf/dnf.conf文件中的“repository”部分

  下面是[repository]部分的一个最小配置示例：

  

  ```
  [repository]
  name=repository_name
  baseurl=repository_url
  ```

  说明：

  openEuler提供在线的镜像源，地址：https://repo.openeuler.org/。以 openEuler 23.09的aarch64版本为例，baseurl可配置为https://repo.openeuler.org/openEuler-23.09/OS/aarch64/。

  选项说明：

  **表 2** repository参数说明

  | 参数                   | 说明                                                         |
  | :--------------------- | :----------------------------------------------------------- |
  | name=repository_name   | 软件仓库（repository ）描述的字符串。                        |
  | baseurl=repository_url | 软件仓库（repository ）的地址。使用http协议的网络位置：例如 http://path/to/repo使用ftp协议的网络位置：例如 ftp://path/to/repo本地位置：例如 file:///path/to/local/repo |

- 配置/etc/yum.repos.d目录下的.repo文件

  openEuler提供了多种repo源供用户在线使用，各repo源含义可参考[系统安装](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/releasenotes/releasenotes/os_installation.html)。使用root权限添加openEuler repo源，示例如下：

  

  ```
  # vi /etc/yum.repos.d/openEuler.repo
  
  [OS]
  name=openEuler-$releasever-OS
  baseurl=https://repo.openeuler.org/openEuler-{version}/OS/$basearch/
  enabled=1
  gpgcheck=1
  gpgkey=https://repo.openeuler.org/openEuler-{version}/OS/$basearch/RPM-GPG-KEY-openEuler
  ```

  说明：

  

  - enabled为是否启用该软件源仓库，可选值为1和0。缺省值为1，表示启用该软件源仓库。
  - gpgkey为验证签名用的公钥。

#### 显示当前配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-显示当前配置)

- 显示当前的配置信息：

  

  ```
  # dnf config-manager --dump
  ```

- 显示相应软件源的配置，首先查询repo id：

  

  ```
  # dnf repolist
  ```

  然后执行如下命令，显示对应id的软件源配置，其中 *repository* 为查询得到的repo id：

  

  ```
  # dnf config-manager --dump repository
  ```

- 您也可以使用一个全局正则表达式，来显示所有匹配部分的配置：

  

  ```
  # dnf config-manager --dump glob_expression
  ```

### 创建本地软件源仓库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-创建本地软件源仓库)

要建立一个本地软件源仓库，请按照下列步骤操作。

1. 安装createrepo软件包。

   

   ```
   # dnf install createrepo
   ```

2. 将需要的软件包复制到一个目录下，如/mnt/local_repo/ 。

3. 创建软件源。

   

   ```
   # createrepo /mnt/local_repo
   ```

### 添加、启用和禁用软件源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-添加启用和禁用软件源)

本节将介绍如何通过“dnf config-manager”命令添加、启用和禁用软件源仓库。

#### 添加软件源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-添加软件源)

要定义一个新的软件源仓库，您可以在 /etc/dnf/dnf.conf 文件中添加“repository”部分，或者在/etc/yum.repos.d/目录下添加“.repo”文件进行说明。建议您通过添加“.repo”的方式，每个软件源都有自己对应的“.repo”文件，以下介绍该方式的操作方法。

要在您的系统中添加一个这样的源，请在root权限下执行如下命令，执行完成之后会在/etc/yum.repos.d/目录下生成对应的repo文件。其中 *repository_url* 为repo源地址，详情请参见[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#zh-cn_topic_0151921080_t7c83ace02ab94e9986c0684f417e3436)。



```
# dnf config-manager --add-repo repository_url
```

#### 启用软件源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-启用软件源)

要启用软件源，请在root权限下执行如下命令，其中 *repository* 为新增.repo文件中的repo id（可通过dnf repolist查询）：



```
# dnf config-manager --set-enable repository
```

您也可以使用一个全局正则表达式，来启用所有匹配的软件源。其中 *glob_expression* 为对应的正则表达式，用于同时匹配多个repo id：



```
# dnf config-manager --set-enable glob_expression
```

#### 禁用软件源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-禁用软件源)

要禁用软件源，请在root权限下执行如下命令：



```
# dnf config-manager --set-disable repository
```

同样的，您也可以使用一个全局正则表达式来禁用所有匹配的软件源：



```
# dnf config-manager --set-disable glob_expression
```

## 管理软件包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-管理软件包)

使用dnf能够让您方便的进行查询、安装、删除软件包等操作。

### 搜索软件包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-搜索软件包)

您可以使用rpm包名称、缩写或者描述搜索需要的RPM包，使用命令如下：



```
# dnf search httpd (以httpd为例)
```

### 列出软件包清单 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-列出软件包清单)

要列出系统中所有已安装的以及可用的RPM包信息，使用命令如下：



```
# dnf list all
```

要列出系统中特定的RPM包信息，使用命令如下：



```
# dnf list httpd  (以httpd为例)
```

### 显示RPM包信息 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-显示rpm包信息)

要显示一个或者多个RPM包信息，多个包之间以空格分隔，使用命令如下：



```
# dnf info httpd zip  (以httpd，zip两个包为例)
```

### 安装RPM包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-安装rpm包)

要安装一个软件包及其所有未安装的依赖，请在root权限下执行如下命令：



```
# dnf install package_name
```

您也可以通过添加软件包名字同时安装多个软件包。配置文件/etc/dnf/dnf.conf添加参数strict=False，运行dnf命令参数添加--setopt=strict=0。请在root权限下执行如下命令：



```
# dnf install package_name package_name... --setopt=strict=0
```

示例如下：



```
# dnf install httpd
```

说明：

安装RPM包过程中，若出现安装失败，可参考[问题5：安装时出现软件包冲突、文件冲突或缺少软件包导致安装失败](https://docs.openeuler.openatom.cn/zh/docs/common/faq/server/administration_faqs.html#问题5-安装时出现软件包冲突、文件冲突或缺少软件包导致安装失败)。

### 下载软件包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-下载软件包)

使用dnf下载软件包，请在root权限下输入如下命令：



```
# dnf download package_name
```

如果需要同时下载未安装的依赖，则加上--resolve，使用命令如下：



```
# dnf download --resolve package_name
```

示例如下：



```
# dnf download --resolve httpd
```

### 删除软件包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-删除软件包)

要卸载软件包以及相关的依赖软件包，请在root权限下执行如下命令：



```
# dnf remove package_name...
```

示例如下：



```
# dnf remove totem
```

## 管理软件包组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-管理软件包组)

软件包集合是服务于一个共同的目的一组软件包，例如系统工具集等。使用dnf可以对软件包组进行安装/删除等操作，使相关操作更高效。

### 列出软件包组清单 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-列出软件包组清单)

使用summary参数，可以列出系统中所有已安装软件包组、可用的组，可用的环境组的数量，命令如下：



```
# dnf groups summary
```

要列出所有软件包组和它们的组ID ，命令如下：



```
# dnf group list
```

### 显示软件包组信息 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-显示软件包组信息)

要列出包含在一个软件包组中必须安装的包和可选包，使用命令如下：



```
# dnf group info glob_expression...
```

例如显示Development Tools信息，示例如下：



```
# dnf group info "Development Tools"
```

### 安装软件包组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-安装软件包组)

每一个软件包组都有自己的名称以及相应的ID（groupid），您可以使用软件包组名称或它的ID进行安装。

要安装一个软件包组，请在root权限下执行如下命令：



```
# dnf group install group_name
# dnf group install groupid
```

例如安装Development Tools相应的软件包组，命令如下：



```
# dnf group install "Development Tools"
# dnf group install development
```

### 删除软件包组 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-删除软件包组)

要卸载软件包组，您可以使用软件包组名称或它的ID，在root权限下执行如下命令：



```
# dnf group remove group_name
# dnf group remove groupid
```

例如删除Development Tools相应的软件包组，命令如下：



```
# dnf group remove "Development Tools"
# dnf group remove development
```

## 检查并更新 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-检查并更新)

dnf可以检查您的系统中是否有软件包需要更新。您可以通过dnf列出需要更新的软件包，并可以选择一次性全部更新或者只对指定包进行更新。

### 检查更新 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-检查更新)

如果您需要显示当前系统可用的更新，使用命令如下：



```
# dnf check-update
```

### 升级 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-升级)

如果您需要升级单个软件包，在root权限下执行如下命令：



```
# dnf update package_name
```

例如升级rpm包，示例如下：



```
# dnf update anaconda-gui.aarch64 （以anaconda-gui包为例）
```

类似的，如果您需要升级软件包组，在root权限下执行如下命令：



```
# dnf group update group_name
```

### 更新所有的包和它们的依赖 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/using_dnf_to_manage_software_packages.html#user-content-更新所有的包和它们的依赖)

要更新所有的包和它们的依赖，在root权限下执行如下命令：



```
# dnf update
```

# 管理服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-管理服务)

本章介绍如何使用systemd进行系统和服务管理。

## 简介 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-简介)

systemd是在Linux下，与SysV和LSB初始化脚本兼容的系统和服务管理器。systemd使用socket和D-Bus来开启服务，提供基于守护进程的按需启动策略，支持快照和系统状态恢复，维护挂载和自挂载点，实现了各服务间基于从属关系的一个更为精细的逻辑控制，拥有更高的并行性能。

### 概念介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-概念介绍)

systemd开启和监督整个系统是基于unit的概念。unit是由一个与配置文件对应的名字和类型组成的（例如：avahi.service unit有一个具有相同名字的配置文件，是守护进程Avahi的一个封装单元）。unit有多种类型，如[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#zh-cn_topic_0151921012_t2dcb6d973cc249ed9ccd56729751ca6b)所示。

**表 1** unit说明



| unit名称       | 后缀名     | 描述                                  |
| :------------- | :--------- | :------------------------------------ |
| Service unit   | .service   | 系统服务。                            |
| Target unit    | .target    | 一组systemd units。                   |
| Automount unit | .automount | 文件系统挂载点。                      |
| Device unit    | .device    | 内核识别的设备文件。                  |
| Mount unit     | .mount     | 文件系统挂载点。                      |
| Path unit      | .path      | 在一个文件系统中的文件或目录。        |
| Scope unit     | .scope     | 外部创建的进程。                      |
| Slice unit     | .slice     | 一组用于管理系统进程分层组织的units。 |
| Socket unit    | .socket    | 一个进程间通信的Socket。              |
| Swap unit      | .swap      | swap设备或者swap文件。                |
| Timer unit     | .timer     | systemd计时器。                       |

所有的可用systemd unit类型，可在如[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#zh-cn_topic_0151921012_t2523a0a9a0c54f9b849e52d1efa0160c)所示的路径下查看。

**表 2** 可用systemd unit类型



| 路径                     | 描述                                    |
| :----------------------- | :-------------------------------------- |
| /usr/lib/systemd/system/ | 随安装的RPM产生的systemd units。        |
| /run/systemd/system/     | 在运行时创建systemd units。             |
| /etc/systemd/system/     | 由系统管理员创建和管理的systemd units。 |

## 特性说明 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-特性说明)

### 更快的启动速度 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-更快的启动速度)

systemd提供了比UpStart更激进的并行启动能力，采用了socket/D-Bus activation等技术启动服务，带来了更快的启动速度。

为了减少系统启动时间，systemd的目标是：

- 尽可能启动更少的进程。
- 尽可能将更多进程并行启动。

### 提供按需启动能力 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-提供按需启动能力)

当sysvinit系统初始化的时候，它会将所有可能用到的后台服务进程全部启动运行。并且系统必须等待所有的服务都启动就绪之后，才允许用户登录。这种做法有两个缺点：首先是启动时间过长；其次是系统资源浪费。

某些服务很可能在很长一段时间内，甚至整个服务器运行期间都没有被使用过。比如CUPS，打印服务在多数服务器上很少被真正使用到。您可能没有想到，在很多服务器上SSHD也是很少被真正访问到的。花费在启动这些服务上的时间是不必要的；同样，花费在这些服务上的系统资源也是一种浪费。

systemd可以提供按需启动的能力，只有在某个服务被真正请求的时候才启动它。当该服务结束，systemd可以关闭它，等待下次需要时再次启动它。

### 采用cgroup特性跟踪和管理进程的生命周期 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-采用cgroup特性跟踪和管理进程的生命周期)

init系统的一个重要职责就是负责跟踪和管理服务进程的生命周期。它不仅可以启动一个服务，也能够停止服务。这看上去没有什么特别的，然而在真正用代码实现的时候，您或许会发现停止服务比一开始想的要困难。

服务进程一般都会作为守护进程（daemon）在后台运行，为此服务程序有时候会派生（fork）两次。在UpStart中，需要在配置文件中正确地配置expect小节。这样UpStart通过对fork系统调用进行计数，从而获知真正的运行进程的PID号。

cgroup已经出现了很久，它主要用来实现系统资源配额管理。cgroup提供了类似文件系统的接口，使用方便。当进程创建子进程时，子进程会继承父进程的cgroup。因此无论服务如何启动新的子进程，所有的这些相关进程都会属于同一个cgroup，systemd只需要简单地遍历指定的cgroup即可正确地找到所有的相关进程，将它们逐一停止即可。

### 启动挂载点和自动挂载的管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-启动挂载点和自动挂载的管理)

传统的Linux系统中，用户可以用/etc/fstab文件来维护固定的文件系统挂载点。这些挂载点在系统启动过程中被自动挂载，一旦启动过程结束，这些挂载点就会确保存在。这些挂载点都是对系统运行至关重要的文件系统，比如HOME目录。和sysvinit一样，systemd管理这些挂载点，以便能够在系统启动时自动挂载它们。systemd还兼容/etc/fstab文件，您可以继续使用该文件管理挂载点。

有时候用户还需要动态挂载点，比如打算访问DVD内容时，才临时执行挂载以便访问其中的内容，而不访问光盘时该挂载点被取消（umount），以便节约资源。传统地，人们依赖autofs服务来实现这种功能。

systemd内建了自动挂载服务，无需另外安装autofs服务，可以直接使用systemd提供的自动挂载管理能力来实现autofs的功能。

### 实现事务性依赖关系管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-实现事务性依赖关系管理)

系统启动过程是由很多的独立工作共同组成的，这些工作之间可能存在依赖关系，比如挂载一个NFS文件系统必须依赖网络能够正常工作。systemd虽然能够最大限度地并发执行很多有依赖关系的工作，但是类似“挂载NFS”和“启动网络”这样的工作还是存在天生的先后依赖关系，无法并发执行。对于这些任务，systemd维护一个“事务一致性”的概念，保证所有相关的服务都可以正常启动而不会出现互相依赖，以至于死锁的情况。

### 与SysV初始化脚本兼容 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-与sysv初始化脚本兼容)

和UpStart一样，systemd引入了新的配置方式，对应用程序的开发也有一些新的要求。如果systemd想替代目前正在运行的初始化系统，就必须和现有程序兼容。任何一个Linux发行版都很难为了采用systemd而在短时间内将所有的服务代码都修改一遍。

systemd提供了和sysvinit以及LSB initscripts兼容的特性。系统中已经存在的服务和进程无需修改。这降低了系统向systemd迁移的成本，使得systemd替换现有初始化系统成为可能。

### 能够对系统进行快照和恢复 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-能够对系统进行快照和恢复)

systemd支持按需启动，因此系统的运行状态是动态变化的，人们无法准确地知道系统当前运行了哪些服务。systemd快照提供了一种将当前系统运行状态保存并恢复的能力。

比如系统当前正运行服务A和B，可以用systemd命令行对当前系统运行状况创建快照。然后将进程A停止，或者做其他的任意的对系统的改变，比如启动新的进程C。在这些改变之后，运行systemd的快照恢复命令，就可立即将系统恢复到快照时刻的状态，即只有服务A和B在运行。一个可能的应用场景是调试：比如服务器出现一些异常，为了调试用户将当前状态保存为快照，然后可以进行任意的操作，比如停止服务等等。等调试结束，恢复快照即可。

## 管理系统服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-管理系统服务)

systemd提供systemctl命令来运行、关闭、重启、显示、启用/禁用系统服务。

### sysvinit命令和systemd命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-sysvinit命令和systemd命令)

systemd提供systemctl命令与sysvinit命令的功能类似。当前版本中依然兼容service和chkconfig命令，相关说明如[表3](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#zh-cn_topic_0151920917_ta7039963b0c74b909b72c22cbc9f2e28)，但建议用systemctl进行系统服务管理。

**表 3** sysvinit命令和systemd命令的对照表



| **sysvinit命令**              | **systemd命令**                                  | **备注**                                           |
| :---------------------------- | :----------------------------------------------- | :------------------------------------------------- |
| service *network* start       | systemctl start *network*.service                | 用来启动一个服务 (并不会重启现有的)。              |
| service *network* stop        | systemctl stop *network*.service                 | 用来停止一个服务 (并不会重启现有的)。              |
| service *network* restart     | systemctl restart *network*.service              | 用来停止并启动一个服务。                           |
| service *network* reload      | systemctl reload *network*.service               | 当支持时，重新装载配置文件而不中断等待操作。       |
| service *network* condrestart | systemctl condrestart *network*.service          | 如果服务正在运行那么重启它。                       |
| service *network* status      | systemctl status *network*.service               | 检查服务的运行状态。                               |
| chkconfig *network* on        | systemctl enable *network*.service               | 在下次启动时或满足其他触发条件时设置服务为启用。   |
| chkconfig *network* off       | systemctl disable *network*.service              | 在下次启动时或满足其他触发条件时设置服务为禁用。   |
| chkconfig *network*           | systemctl is-enabled *network*.service           | 用来检查一个服务在当前环境下被配置为启用还是禁用。 |
| chkconfig --list              | systemctl list-unit-files --type=service         | 输出在各个运行级别下服务的启用和禁用情况。         |
| chkconfig *network* --list    | ls /etc/systemd/system/*.wants/*network*.service | 用来列出该服务在哪些运行级别下启用和禁用。         |
| chkconfig *network* --add     | systemctl daemon-reload                          | 当您创建新服务文件或者变更设置时使用。             |

### 显示所有当前服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-显示所有当前服务)

如果您需要显示当前正在运行的服务，使用命令如下：



```
systemctl list-units --type service
```

如果您需要显示所有的服务（包括未运行的服务），需要添加-all参数，使用命令如下：



```
# systemctl list-units --type service --all
```

例如显示当前正在运行的服务，命令如下：



```
# systemctl list-units --type service
UNIT                        LOAD   ACTIVE     SUB           DESCRIPTION  
atd.service                 loaded active     running       Deferred execution scheduler  
auditd.service              loaded active     running       Security Auditing Service  
avahi-daemon.service        loaded active     running       Avahi mDNS/DNS-SD Stack  
chronyd.service             loaded active     running       NTP client/server  
crond.service               loaded active     running       Command Scheduler  
dbus.service                loaded active     running       D-Bus System Message Bus  
dracut-shutdown.service     loaded active     exited        Restore /run/initramfs on shutdown  
firewalld.service           loaded active     running       firewalld - dynamic firewall daemon  
getty@tty1.service          loaded active     running       Getty on tty1  
gssproxy.service            loaded active     running       GSSAPI Proxy Daemon  
......
```

### 显示服务状态 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-显示服务状态)

如果您需要显示某个服务的状态，可执行如下命令：



```
systemctl status name.service
```

相关状态显示参数说明如[表4](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#zh-cn_topic_0151920917_t36cd267d69244ed39ae06bb117ed8e62)所示。

**表 4** 状态参数说明



| **参数** | **描述**                                                   |
| :------- | :--------------------------------------------------------- |
| Loaded   | 说明服务是否被加载，并显示服务对应的绝对路径以及是否启用。 |
| Active   | 说明服务是否正在运行，并显示时间节点。                     |
| Main PID | 相应的系统服务的PID值。                                    |
| CGroup   | 相关控制组（CGroup）的其他信息。                           |

如果您需要鉴别某个服务是否运行，可执行如下命令：



```
systemctl status name.service
```

is-active命令的返回结果如下：

**表 5** is-active命令的返回结果



| 状态            | 含义                                                         |
| :-------------- | :----------------------------------------------------------- |
| active(running) | 有一个或多个程序正在系统中执行。                             |
| active(exited)  | 仅执行一次就正常结束的服务，目前并没有任何程序在系统中执行。 举例来说，开机或者 是挂载时才会进行一次的 quotaon 功能。 |
| active(waiting) | 正在执行当中，不过要等待其他的事件才能继续处理。例如：打印的队列相关服务 就是这种状态，虽然正在启动中，不过也需要真的有队列进来 (打印作业) 这样他才会继续唤醒打印机 服务来进行下一步打印的功能 |
| inactive        | 这个服务没有运行                                             |

同样，如果您需要判断某个服务是否被启用，可执行如下命令：



```
systemctl is-enabled name.service
```

is-enabled命令的返回结果如下：

**表 6** is-enabled命令的返回结果



| 状态              | 含义                                                         |
| :---------------- | :----------------------------------------------------------- |
| "enabled"         | 已经通过 /etc/systemd/system/ 目录下的 Alias= 别名、 .wants/ 或 .requires/ 软链接被永久启用。 |
| "enabled-runtime" | 已经通过 /run/systemd/system/ 目录下的 Alias= 别名、 .wants/ 或 .requires/ 软链接被临时启用。 |
| "linked"          | 虽然单元文件本身不在标准单元目录中，但是指向此单元文件的一个或多个软链接已经存在于 /etc/systemd/system/ 永久目录中。 |
| "linked-runtime"  | 虽然单元文件本身不在标准单元目录中，但是指向此单元文件的一个或多个软链接已经存在于 /run/systemd/system/ 临时目录中。 |
| "masked"          | 已经被 /etc/systemd/system/ 目录永久屏蔽(软链接指向 /dev/null 文件)，因此 **start** 操作会失败。 |
| "masked-runtime"  | 已经被 /run/systemd/systemd/ 目录临时屏蔽(软链接指向 /dev/null 文件)，因此 **start** 操作会失败。 |
| "static"          | 尚未被启用，并且单元文件的 "[Install]" 小节中没有可用于 **enable** 命令的选项。 |
| "indirect"        | 尚未被启用，但是单元文件的 "[Install]" 小节中 Also= 选项的值列表非空(也就是列表中的某些单元可能已被启用)、或者它拥有一个不在 Also= 列表中的其他名称的别名软链接。对于模版单元来说，表示已经启用了一个不同于 DefaultInstance= 的实例。 |
| "disabled"        | 尚未被启用，但是单元文件的 "[Install]" 小节中存在可用于 **enable** 命令的选项 |
| "generated"       | 单元文件是被单元生成器动态生成的。被生成的单元文件可能并未被直接启用，而是被单元生成器隐含的启用了。 |
| "transient"       | 单元文件是被运行时API动态临时生成的。该临时单元可能并未被启用。 |
| "bad"             | 单元文件不正确或者出现其他错误。 **is-enabled** 不会返回此状态，而是会显示一条出错信息。 **list-unit-files** 命令有可能会显示此单元。 |

例如查看gdm.service服务状态，命令如下：



```
# systemctl status gdm.service
gdm.service - GNOME Display Manager   Loaded: loaded (/usr/lib/systemd/system/gdm.service; enabled)   Active: active (running) since Thu 2013-10-17 17:31:23 CEST; 5min ago
 Main PID: 1029 (gdm)
   CGroup: /system.slice/gdm.service
           ├─1029 /usr/sbin/gdm
           ├─1037 /usr/libexec/gdm-simple-slave --display-id /org/gno...           
           └─1047 /usr/bin/Xorg :0 -background none -verbose -auth /r...Oct 17 17:31:23 localhost systemd[1]: Started GNOME Display Manager.
```

### 运行服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-运行服务)

如果您需要运行某个服务，请在root权限下执行如下命令：



```
systemctl start name.service
```

例如运行httpd服务，命令如下：



```
# systemctl start httpd.service
```

### 关闭服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-关闭服务)

如果您需要关闭某个服务，请在root权限下执行如下命令：



```
systemctl stop name.service
```

例如关闭蓝牙服务，命令如下：



```
# systemctl stop bluetooth.service
```

### 重启服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-重启服务)

如果您需要重启某个服务，请在root权限下执行如下命令：



```
systemctl restart name.service
```

执行命令后，当前服务会被关闭，但马上重新启动。如果您指定的服务，当前处于关闭状态，执行命令后，服务也会被启动。

例如重启蓝牙服务，命令如下：



```
# systemctl restart bluetooth.service
```

### 启用服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-启用服务)

如果您需要在开机时启用某个服务，请在root权限下执行如下命令：



```
systemctl enable name.service
```

例如设置httpd服务开机时启动，命令如下：



```
# systemctl enable httpd.service
ln -s '/usr/lib/systemd/system/httpd.service' '/etc/systemd/system/multi-user.target.wants/httpd.service'
```

### 禁用服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-禁用服务)

如果您需要在开机时禁用某个服务，请在root权限下执行如下命令：



```
systemctl disable name.service
```

例如在开机时禁用蓝牙服务启动，命令如下：



```
# systemctl disable bluetooth.service
Removed /etc/systemd/system/bluetooth.target.wants/bluetooth.service.
Removed /etc/systemd/system/dbus-org.bluez.service.
```

## 改变运行级别 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-改变运行级别)

### Target和运行级别 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-target和运行级别)

systemd用目标（target）替代了运行级别的概念，提供了更大的灵活性，如您可以继承一个已有的目标，并添加其他服务，来创建自己的目标。[表7](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#zh-cn_topic_0151920939_t9af92c282ad240ea9a79fb08d26e8181)列举了systemd下的目标和常见runlevel的对应关系。

**表 7** 运行级别和systemd目标



| 运行级别     | systemd目标（target）                                 | 描述                                                      |
| :----------- | :---------------------------------------------------- | :-------------------------------------------------------- |
| 0            | runlevel0.target，poweroff.target                     | 关闭系统。                                                |
| 1, s, single | runlevel1.target，rescue.target                       | 单用户模式。                                              |
| 2, 4         | runlevel2.target，runlevel4.target，multi-user.target | 用户定义/域特定运行级别。默认等同于3。                    |
| 3            | runlevel3.target，multi-user.target                   | 多用户，非图形化。用户可以通过多个控制台或网络登录。      |
| 5            | runlevel5.target，graphical.target                    | 多用户，图形化。通常为所有运行级别3的服务外加图形化登录。 |
| 6            | runlevel6.target，reboot.target                       | 重启系统。                                                |
| emergency    | emergency.target                                      | 紧急Shell。                                               |

### 查看系统默认启动目标 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-查看系统默认启动目标)

查看当前系统默认的启动目标，命令如下：



```
systemctl get-default
```

### 查看当前系统所有的启动目标 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-查看当前系统所有的启动目标)

查看当前系统所有的启动目标，命令如下：



```
systemctl list-units --type=target
```

### 改变默认目标 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-改变默认目标)

改变系统默认的目标，在root权限下执行如下命令：



```
systemctl set-default name.target
```

### 改变当前目标 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-改变当前目标)

改变当前系统的目标，在root权限下执行如下命令：



```
systemctl isolate name.target
```

### 切换到救援模式 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-切换到救援模式)

改变当前系统为救援模式，在root权限下执行如下命令：



```
systemctl rescue
```

这条命令和“systemctl isolate rescue.target”类似。命令执行后会在串口有如下打印信息：



```
You are in rescue mode. After logging in, type "journalctl -xb" to viewsystem logs, "systemctl reboot" to reboot, "systemctl default" or "exit"to boot into default mode.
Give root password for maintenance
(or press Control-D to continue):
```

说明：

从救援模式进入正常模式，用户需要重启系统。

### 切换到紧急模式 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-切换到紧急模式)

改变当前系统为紧急模式，在root权限下执行如下命令：



```
systemctl emergency
```

这条命令和“systemctl isolate emergency.target”类似。命令执行后会在串口有如下打印信息：



```
You are in emergency mode. After logging in, type "journalctl -xb" to viewsystem logs, "systemctl reboot" to reboot, "systemctl default" or "exit"to boot into default mode.
Give root password for maintenance
(or press Control-D to continue):
```

说明：

从紧急模式进入正常模式，用户需要重启系统。

## 关闭、暂停和休眠系统 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-关闭暂停和休眠系统)

### systemctl命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-systemctl命令)

systemd通过systemctl命令可以对系统进行关机、重启、休眠等一系列操作。当前仍兼容部分Linux常用管理命令，对应关系如[表8](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#zh-cn_topic_0151920964_t3daaaba6a03b4c36be9668efcdb61f3b)。建议用户使用systemctl命令进行操作。

**表 8** 命令对应关系



| Linux常用管理命令 | systemctl命令      | 描述     |
| :---------------- | :----------------- | :------- |
| halt              | systemctl halt     | 关闭系统 |
| poweroff          | systemctl poweroff | 关闭电源 |
| reboot            | systemctl reboot   | 重启     |

### 关闭系统 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-关闭系统)

关闭系统并下电，在root权限下执行如下命令：



```
systemctl poweroff
```

关闭系统但不下电机器，在root权限下执行如下命令：



```
systemctl halt
```

执行上述命令会给当前所有的登录用户发送一条提示消息。如果不想让systemd发送该消息，您可以添加“--no-wall”参数。具体命令如下：



```
systemctl --no-wall poweroff
```

### 重启系统 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-重启系统)

重启系统，在root权限下执行如下命令：



```
systemctl reboot
```

执行上述命令会给当前所有的登录用户发送一条提示消息。如果不想让systemd发送该消息，您可以添加“--no-wall”参数。具体命令如下：



```
systemctl --no-wall reboot
```

### 使系统待机 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-使系统待机)

使系统待机，在root权限下执行如下命令：



```
systemctl suspend
```

### 使系统休眠 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html#user-content-使系统休眠)

使系统休眠，在root权限下执行如下命令：



```
systemctl hibernate
```

使系统待机且处于休眠状态，在root权限下执行如下命令：



```
systemctl hybrid-sleep
```

# 管理进程 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-管理进程)

操作系统管理多个用户的请求和多个任务。大多数系统都只有一个CPU和一个主要存储，但一个系统可能有多个二级存储磁盘和多个输入/输出设备。操作系统管理这些资源并在多个用户间共享资源，当用户提出一个请求时，造成好像系统被用户独占的假象。实际上操作系统监控着一个等待执行的任务队列，这些任务包括用户任务、操作系统任务、邮件和打印任务等。本章节将从用户的角度讲述如何控制进程。

## 查看进程 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-查看进程)

Linux是一个多任务系统，经常需要对这些进程进行一些调配和管理。要进行管理，首先就要知道现在的进程情况：有哪些进程、进程的状态如何等。Linux提供了多种命令来了解进程的状况。

### who命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-who命令)

who命令主要用于查看当前系统中的用户情况。如果用户想和其他用户建立即时通讯，比如使用talk命令，那么首先要确定的就是该用户确实在线上，不然talk进程就无法建立起来。又如，系统管理员希望监视每个登录的用户此时此刻的所作所为，也要使用who命令。who命令应用起来非常简单，可以比较准确地掌握用户的情况，所以使用非常广泛。

例如查看系统中的用户及其状态。使用如下：



```
# who
admin     tty1         Jul 28 15:55
admin     pts/0        Aug  5 15:46 (192.168.0.110)
admin     pts/2        Jul 29 19:52 (192.168.0.110)
root     pts/3        Jul 30 12:07 (192.168.0.110)
root     pts/4        Jul 31 10:29 (192.168.0.144)
root     pts/5        Jul 31 14:52 (192.168.0.11)
root     pts/6        Aug  6 10:12 (192.168.0.234)
root     pts/8        Aug  6 11:34 (192.168.0.234)
```

### ps命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-ps命令)

ps命令是最基本又非常强大的进程查看命令。使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵尸、哪些进程占用了过多的资源等，大部分进程信息都是可以通过执行该命令得到的。

ps命令最常用的还是用来监控后台进程的工作情况，因为后台进程是不与屏幕、键盘这些标准输入/输出设备进行通信的，所以如果需要检测其状况，就可使用ps命令。ps命令的常见选项如[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#zh-cn_topic_0151921029_t34619d964a3d41ad8694189ec383359c)所示。

**表 1** 选项说明



| 选项 | 描述                                       |
| :--- | :----------------------------------------- |
| -e   | 显示所有进程。                             |
| -f   | 全格式。                                   |
| -h   | 不显示标题。                               |
| -l   | 使用长格式。                               |
| -w   | 宽行输出。                                 |
| -a   | 显示终端上的所有进程，包括其他用户的进程。 |
| -r   | 只显示正在运行的进程。                     |
| -x   | 显示没有控制终端的进程。                   |

例如显示系统中终端上的所有进程。命令如下：



```
# ps -a
  PID TTY          TIME CMD
12175 pts/6    00:00:00 bash
24526 pts/0    00:00:00 vsftpd
29478 pts/5    00:00:00 ps
32461 pts/0    1-01:58:33 sh
```

### top命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-top命令)

top命令和ps命令的基本作用是相同的，显示系统当前的进程和其他状况，但是top是一个动态显示过程，即可以通过用户按键来不断刷新进程的当前状态，如果在前台执行该命令，它将独占前台，直到用户终止该程序为止。其实top命令提供了实时的对系统处理器的状态监视。它将显示系统中CPU的任务列表。该命令可以按CPU使用、内存使用和执行时间对任务进行排序，而且该命令的很多特性都可以通过交互式命令或者在定制文件中进行设定。

top命令输出的示例如[图1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#zh-cn_topic_0151921029_f289234fcdbac453796200d80e9889cd1)所示：

**图 1** top显示
![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/top_display.Plm0kpAj.png)

### kill命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-kill命令)

当需要中断一个前台进程的时候，通常足使用“Ctrl+c”组合键，而对于后台进程不能用组合键来终止，这时就可以使用kill命令。该命令可以终止前台和后台进程。终止后台进程的原因包括：该进程占用CPU的时间过多、该进程已经死锁等。

kill命令是通过向进程发送指定的信号来结束进程的。如果没有指定发送的信号，那么缺省值为TERM信号。TERM信号将终止所有不能捕获该信号的进程。至于那些可以捕获该信号的进程可能就需要使用KILL信号（它的编号为9），而该信号不能被捕捉。

kill命令的浯法格式有以下两种方式：



```
kill [-s 信号 | -p] [-a] 进程号…
kill -l [信号]
```

其中进程号可以通过ps命令的输出得到。-s选项是给程序发送指定的信号，详细的信号可以用“kill -l”命令查看；-p选项只显示指定进程的ID号。

杀死pid为1409的进程，在root权限下执行如下命令：



```
# kill -9 1409
```

显示所有的信号及其编号对应关系，示例如下：



```
# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

## 调度启动进程 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-调度启动进程)

有时候需要对系统进行一些比较费时而且占用资源的维护工作，这些工作适合在深夜进行，这时候用户就可以事先进行调度安排，指定任务运行的时间或者场合，到时候系统会自动完成这些任务。要使用自动启动进程的功能，就需要掌握以下几个启动命令。

### 定时运行一批程序（at） [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-定时运行一批程序at)

#### at命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-at命令)

用户使用at命令在指定时刻执行指定的命令序列。该命令至少需要指定一个命令和一个执行时间。at命令可以只指定时间，也可以时间和日期一起指定。

at命令的语法格式如下：



```
 at [-V] [-q 队列] [-f 文件名] [-mldbv] 时间
 at -c 作业 [作业…]
```

#### 设置时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-设置时间)

at允许使用一套相当复杂的时间指定方法，比如：

- 接受在当天的hh:mm（小时：分钟）式的时间指定。如果该时间已经过去，那么就放在第二天执行。
- 使用midnight（深夜）、noon（中午）、teatime（饮茶时间，一般是下午4点）等比较模糊的词语来指定时间。
- 采用12小时计时制，即在时间后面加上AM（上午）或者PM（下午）来说明是上午还是下午。
- 指定命令执行的具体日期，指定格式为month day（月日）或者mm/dd/yy（月/日/年）或者dd.mm.yy（日.月.年）。指定的日期必须跟在指定时间的后面。

上面介绍的都是绝对计时法，其实还可以使用相对计时法，这对于安排不久就要执行的命令是很有好处的。指定格式为now+count time-units，now就是当前时间，time-units是时间单位，这里可以是minutes（分钟）、hours（小时）、days（天）、weeks（星期）。count是时间的数量，究竟是几天，还是几小时等。还有一种计时方法就是直接使用today（今天）、tomorrow（明天）来指定完成命令的时间。下面通过一些例子来说明具体用法。

例如指定在今天下午4:30执行某个命令。假设现在时间是中午12:30，2019年6月7日，可用命令格式如下：



```
 at 4:30pm
 at 16:30
 at 16:30 today
 at now+4 hours
 at now+ 240 minutes
 at 16:30 7.6.19
 at 16:30 6/7/19
 at 16:30 Jun 7
```

以上这些命令表达的意义是完全一样的，所以在安排时间的时候完全可以根据个人喜好和具体情况自由选择。一般采用绝对时间的24小时计时法可以避免由于用户自己的疏忽造成计时错误，例如上例可以写成：at 16:30 6/7/19。

#### 执行权限 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-执行权限)

对于at命令来说，需要定时执行的命令是从标准输入或者使用-f选项指定的文件中读取并执行的。如果at命令是从一个使用su命令切换到用户shell中执行的，那么当前用户被认为是执行用户，所有的错误和输出结果都会送给这个用户。但是如果有邮件送出的话，收到邮件的将是原来的用户，也就是登录时shell的所有者。

例如在6月8日上午10点执行slocate -u命令。在root权限下执行命令如下：



```
# at  10:00  6/8/19
at> slocate -u
at>
[1]+   Stopped    at  10:00  6/8/19
```

上面的结果中，输入at命令之后，会出现提示符at>，提示用户输入命令，在此输入了slocate -u，然后按回车键。还可以输入多条命令，当所有要执行的命令输入结束后，按“Ctrl+d”键结束at命令。

在任何情况下，管理员帐户都可以使用这个命令。对于其他用户来说，是否可以使用就取决于/etc/at.allow和/etc/at.deny文件。

### 周期性运行一批程序（cron） [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-周期性运行一批程序cron)

前面介绍at命令都会在一定时间内完成一定任务，但是它只能执行一次。也就是说，当指定了运行命令后，系统在指定时间完成任务，以后就不再执行了。但是在很多情况下需要周期性重复执行一些命令，这时候就需要使用cron命令来完成任务。

#### 运行机制 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-运行机制)

首先cron命令会搜索/var/spool/cron目录，寻找以/etc/passwd文件中的用户名命名的crontab文件，被找到的这种文件将装入内存。比如一个用户名为userexample的用户，对应的crontab文件应该是/var/spool/cron/userexample，即以该用户命名的crontab文件存放在/var/spool/cron目录下面。

cron命令还将搜索/etc/crontab文件，这个文件是用不同的格式写成的。cron启动以后，它将首先检查是否有用户设置了crontab文件，如果没有就转入睡眠状态，释放系统资源。所以该后台进程占用资源极少，它每分钟被唤醒一次，查看当前是否有需要运行的命令。

命令执行结束后，任何输出都将作为邮件发送给crontab的所有者，或者是/etc/crontab文件中MAILTO环境变量中指定的用户。这是cron的工作原理，但是cron命令的执行不需要用户干涉，用户只需要修改crontab中要执行的命令。

#### crontab命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-crontab命令)

crontab命令用于安装、删除或者显示用于驱动cron后台进程的表格。用户把需要执行的命令序列放到crontab文件中以获得执行，而且每个用户都可以有自己的crontab文件。

crontab命令的常用方法如下：

- crontab -u //设置某个用户的cron服务，root用户在执行crontab时需要此参数。
- crontab -l //列出某个用户cron服务的详细内容。
- crontab -r //删除某个用户的cron服务。
- crontab -e //编辑某个用户的cron服务。

例如root查看自己的cron设置。命令如下：



```
# crontab -u root -l
```

#### crontab文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-crontab文件)

在crontab文件中输入需要执行的命令和时间。该文件中每行都包括6个域，其中前5个域是指定命令被执行的时间，最后一个域是要被执行的命令。每个域之间使用空格或者制表符分隔。格式如下：



```
minute hour day-of-month month-of-year day-of-week commands
```

对于每一项的说明如[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#zh-cn_topic_0151921016_t7d97d1204fe249d7ae0a87b4cf9a9353)所示。

**表 2** 参数说明



| 参数          | 描述                               |
| :------------ | :--------------------------------- |
| minute        | 分钟（0~59）。                     |
| hour          | 小时（0~23）。                     |
| day-of-month  | 一个月的第几天（1~31）。           |
| month-of-year | 一年的第几个月（1~12）。           |
| day-of-week   | 一周的星期几（0~6），0代表星期天。 |
| commands      | 需要执行的命令。                   |

这些项都不能为空，必须指定值。除了数字还有几个特殊的符号“*”、“/”和“-”、“，”。其中，*代表所有的取值范围内的数字，/代表每的意思，“*/5”表示每5个单位，“-”代表从某个数字到某个数字，“，”用于分开几个离散数字。对于要执行的命令，调用的时候需要写出命令的完整路径。

例如晚上18点到22点之间每两个小时，在/tmp/test.txt文件中加入sleepy文本。在crontab文件中对应的行如下：



```
* 18-22/2 * * * echo "sleepy" >> /tmp/test.txt
```

每次编辑完某个用户的cron设置后，cron自动在/var/spool/cron下生成一个与此用户同名的文件。此用户的cron信息都记录在这个文件中，这个文件是不可以直接编辑的，只可以用crontab -e来编辑。用户也可以另外建立一个文件，使用“cron文件名”命令导入cron设置。

假设有个用户名为userexample，它需要为自己创建一个crontab文件。步骤如下：

1. 首先可以使用任何文本编辑器建立一个新文件，并向该文件加入需要运行的命令和要定期执行的时间，假设该文件为 ~/userexample.cron。

2. 然后在root权限下使用crontab命令安装这个文件，使之成为该用户的crontab文件。命令如下：

   

   ```
   # crontab -u userexample ~/userexample.cron
   ```

这样crontab文件就建立好了，可以转到/var/spool/cron目录下面查看，发现多了一个userexample文件。这个文件就是所需的crontab文件。

说明：

cron启动后，每过一分钟读一次crontab文件，检查是否要执行里面的命令。因此该文件被修改后不需要重新启动cron服务。

#### 编辑配置文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-编辑配置文件)

cron服务每分钟不仅要读一次/var/spool/cron内的所有文件，还需要读一次/etc/crontab，因此通过配置这个文件也能得到cron的服务。用crontab配置是针对某个用户的，而编辑/etc/crontab是针对系统的任务。此文件的文件格式如下：



```
SHELL=/bin/sh
PATH=/usr/bin:/usr/sbin:/sbin:/bin:/usr/lib/news/bin
MAILTO=root  //如果出现错误，或者有数据输出，将发邮件给这个帐号
HOME=/
#  run-parts
01  * * * *   root run-parts /etc/cron.hourly     //每个小时执行一次/etc/cron.hourly里的脚本
02 4 * * *   root run-parts /etc/cron.daily    //每天执行一次/etc/cron.daily里的脚本
22 4 * * 0  root run-parts /etc/cron.weekly     //每周执行一次/etc/cron.weekly里的脚本
42 4 1  * *  root run-parts /etc/cron.monthly     //每月执行一次/etc/cron.monthly里的脚本
```

说明：

如果去掉run-parts参数，其后面就是运行的某个脚本名，而不是目录名。

## 挂起/恢复进程 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/process_management.html#user-content-挂起恢复进程)

作业控制允许进程挂起并可以在需要时恢复进程的运行，被挂起的作业恢复后将从中止处开始继续运行。只要在键盘上按“Ctrl+Z”键，即可挂起当前的前台作业。在键盘上按“Ctrl+Z”键后，将挂起当前执行的命令cat。使用jobs命令可以显示shell的作业清单，包括具体的作业、作业号以及作业当前所处的状态。

恢复进程执行时，有两种选择：用fg命令将挂起的作业放回到前台执行；用bg命令将挂起的作业放到后台执行。灵活使用上述命令，将给自己带来很大的方便。



# 搭建repo服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-搭建repo服务器)

说明：

openEuler提供了多种repo源供用户在线使用，各repo源含义可参考[系统安装](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/releasenotes/releasenotes/os_installation.html)。若用户无法在线获取openEuler repo源，则可使用openEuler提供的ISO发布包创建为本地openEuler repo源。本章节中以openEuler-{version}-aarch64-dvd.iso发布包为例，请根据实际需要的ISO发布包进行修改。

## 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-概述)

将openEuler提供的ISO发布包openEuler-{version}-aarch64-dvd.iso创建为repo源，如下以使用nginx进行repo源部署，提供http服务为例进行说明。

## 创建/更新本地repo源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-创建更新本地repo源)

使用mount挂载，将openEuler的ISO发布包openEuler-{version}-aarch64-dvd.iso创建为repo源，并能够对repo源进行更新。

### 获取ISO发布包 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-获取iso发布包)

请从如下网址获取openEuler的ISO发布包：

https://repo.openeuler.org/openEuler-{version}/ISO/

### 挂载ISO创建repo源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-挂载iso创建repo源)

在root权限下使用mount命令挂载ISO发布包。

示例如下：



```
# mount /home/openEuler/openEuler-{version}-aarch64-dvd.iso /mnt/
```

挂载好的mnt目录如下：



```
.
│── boot.catalog
│── docs
│── EFI
│── images
│── Packages
│── repodata
│── TRANS.TBL
└── RPM-GPG-KEY-openEuler
```

其中，Packages为rpm包所在的目录，repodata为repo源元数据所在的目录，RPM-GPG-KEY-openEuler为openEuler的签名公钥。

### 创建本地repo源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-创建本地repo源)

可以拷贝ISO发布包中相关文件至本地目录以创建本地repo源，示例如下：



```
# mount /home/openEuler/openEuler-{version}-aarch64-dvd.iso /mnt/
# mkdir -p /home/openEuler/srv/repo/
# cp -r /mnt/Packages /home/openEuler/srv/repo/
# cp -r /mnt/repodata /home/openEuler/srv/repo/
# cp -r /mnt/RPM-GPG-KEY-openEuler /home/openEuler/srv/repo/
```

从而本地repo目录如下：



```
.
│── Packages
│── repodata
└── RPM-GPG-KEY-openEuler
```

Packages为rpm包所在的目录，repodata为repo源元数据所在的目录，RPM-GPG-KEY-openEuler为openEuler的签名公钥。

### 更新repo源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-更新repo源)

更新repo源有两种方式：

- 通过新版本的ISO更新已有的repo源，与创建repo源的方式相同，即挂载ISO发布包或重新拷贝ISO发布包至本地目录。

- 在repo源的Packages目录下添加rpm包，然后通过createrepo命令更新repo源。

  

  ```
  # createrepo --update --workers=10 ~/srv/repo
  ```

  其中，--update表示更新，--workers表示线程数，可自定义。

  说明：

  若命令打印信息为“createrepo：未找到命令”，则表示未安装createrepo软件，可在root权限下执行**dnf install createrepo**进行安装。

## 部署远端repo源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-部署远端repo源)

安装openEuler操作系统，在openEuler上通过nginx部署repo源。

### nginx安装与配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-nginx安装与配置)

1. 请自行下载nginx工具并在root权限下安装nginx。

2. 安装nginx之后，在root权限下配置/etc/nginx/nginx.conf。

   说明：

   文档中的配置内容仅供参考，请用户根据实际情况（例如安全加固需要）进行配置。

   

   ```
   user  nginx;
   worker_processes  auto;                          # 建议设置为core-1
   error_log  /var/log/nginx/error.log  warn;       # log存放位置
   pid        /var/run/nginx.pid;
   
   events {
       worker_connections  1024;
   }
   
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
   
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
   
       access_log  /var/log/nginx/access.log  main;
       sendfile        on;
       keepalive_timeout  65;
   
       server {
           listen       80;
           server_name  localhost;                 # 服务器名（url）
           client_max_body_size 4G;
           root         /usr/share/nginx/repo;                 # 服务默认目录
   
           location / {
               autoindex            on;            # 开启访问目录下层文件
               autoindex_exact_size on;
               autoindex_localtime  on; 
           }
   
       }
   
   }
   ```

### 启动nginx服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-启动nginx服务)

1. 在root权限下通过systemctl命令启动nginx服务：

   

   ```
   # systemctl enable nginx
   # systemctl start nginx
   ```

2. nginx是否启动成功可通过下面命令查看：

   

   ```
   # systemctl status nginx
   ```

   - [图1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#zh-cn_topic_0151920971_fd25e3f1d664b4087ae26631719990a71)表示nginx服务启动成功

     **图 1** nginx服务启动成功
     ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/nginx_start_success.D3ADraGT.png)

   - 若nginx服务启动失败，查看错误信息：

   

   ```
   # systemctl status nginx.service --full
   ```

   **图 2** nginx服务启动失败
   ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/nginx_start_failed.Q03mytTi.png)

   如[图2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#zh-cn_topic_0151920971_f1f9f3d086e454b9cba29a7cae96a4c54)所示nginx服务创建失败，是由于目录/var/spool/nginx/tmp/client_body创建失败，在root权限下手动进行创建，类似的问题也这样处理：

   

   ```
   # mkdir -p /var/spool/nginx/tmp/client_body
   # mkdir -p /var/spool/nginx/tmp/proxy
   # mkdir -p /var/spool/nginx/tmp/fastcgi
   # mkdir -p /usr/share/nginx/uwsgi_temp
   # mkdir -p /usr/share/nginx/scgi_temp
   ```

### repo源部署 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-repo源部署)

1. 在root权限下创建nginx配置文件/etc/nginx/nginx.conf中指定的目录/usr/share/nginx/repo：

   

   ```
   # mkdir -p /usr/share/nginx/repo
   ```

2. 在root权限下修改目录/usr/share/nginx/repo的权限：

   

   ```
   # chmod -R 755 /usr/share/nginx/repo
   ```

3. 设置防火墙规则，开启nginx设置的端口（此处为80端口），在root权限下通过firewall设置端口开启：

   

   ```
   # firewall-cmd --add-port=80/tcp --permanent
   # firewall-cmd --reload
   ```

   在root权限下查询80端口是否开启成功，输出为yes则表示80端口开启成功：

   

   ```
   # firewall-cmd --query-port=80/tcp
   ```

   也可在root权限下通过iptables来设置80端口开启：

   

   ```
   # iptables -I INPUT -p tcp --dport 80 -j ACCEPT
   ```

4. nginx服务设置好之后，即可通过ip直接访问网页，如[图3](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#zh-cn_topic_0151921017_fig1880404110396)：

   **图 3** nginx部署成功
   ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/nginx_deployed_success.DON1yG5K.png)

5. 通过下面几种方式将repo源放入到/usr/share/nginx/repo下：

   - 在root权限下拷贝镜像中相关文件至/usr/share/nginx/repo下，并修改目录权限。

     

     ```
     # mount /home/openEuler/openEuler-{version}-aarch64-dvd.iso  /mnt/
     # cp -r /mnt/Packages /usr/share/nginx/repo
     # cp -r /mnt/repodata /usr/share/nginx/repo
     # cp -r /mnt/RPM-GPG-KEY-openEuler /usr/share/nginx/repo
     # chmod -R 755 /usr/share/nginx/repo
     ```

     openEuler-{version}-aarch64-dvd.iso存放在/home/openEuler目录下。

   - 使用root在/usr/share/nginx/repo下创建repo源的软链接。

     

     ```
     # ln -s /mnt /usr/share/nginx/repo/os
     ```

     /mnt为已经创建好的repo源，/usr/share/nginx/repo/os将指向/mnt。

## 使用repo源 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-使用repo源)

repo可配置为yum源，yum（全称为 Yellow dog Updater, Modified）是一个Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载和安装。

### repo配置为yum源（软件源） [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-repo配置为yum源软件源)

构建好的repo可以配置为yum源使用，在/etc/yum.repos.d/目录下使用root权限创建***.repo的配置文件（必须以.repo为扩展名），分为本地和http服务器配置yum源两种方式：

- 配置本地yum源

  在/etc/yum.repos.d目录下创建openEuler.repo文件，使用构建的本地repo源作为yum源，openEuler.repo的内容如下：

  

  ```
  [base]
  name=base
  baseurl=file:///home/openEuler/srv/repo
  enabled=1
  gpgcheck=1
  gpgkey=file:///home/openEuler/srv/repo/RPM-GPG-KEY-openEuler
  ```

  说明：

  

  - [*repoid*]中的repoid为软件仓库（repository）的ID号，所有.repo配置文件中的各repoid不能重复，必须唯一。示例中repoid设置为**base**。
  - name为软件仓库描述的字符串。
  - baseurl为软件仓库的地址。
  - enabled为是否启用该软件源仓库，可选值为1和0。缺省值为1，表示启用该软件源仓库。
  - gpgcheck可设置为1或0，1表示进行gpg（GNU Private Guard）校验，0表示不进行gpg校验，gpgcheck可以确定rpm包的来源是有效和安全的。
  - gpgkey为验证签名用的公钥。

- 配置http服务器yum源

  在/etc/yum.repos.d目录下创建openEuler.repo文件。

  - 若使用用户部署的http服务端的repo源作为yum源，openEuler.repo的内容如下：

    

    ```
    [base]
    name=base
    baseurl=http://192.168.139.209/
    enabled=1
    gpgcheck=1
    gpgkey=http://192.168.139.209/RPM-GPG-KEY-openEuler
    ```

    说明：

    “192.168.139.209”为示例地址，请用户根据实际情况进行配置。

  - 若使用openEuler提供的openEuler repo源作为yum源，以AArch64架构的OS repo源为例，openEuler.repo的内容如下：

    

    ```
    [base]
    name=base
    baseurl=http://repo.openeuler.org/openEuler-{version}/OS/aarch64/
    enabled=1
    gpgcheck=1
    gpgkey=http://repo.openeuler.org/openEuler-{version}/OS/aarch64/RPM-GPG-KEY-openEuler
    ```

### repo优先级 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-repo优先级)

当有多个repo源时，可通过在.repo文件的priority参数设置repo的优先级（如果不设置，默认优先级是99，当相同优先级的源中存在相同rpm包时，会安装最新的版本）。其中，1为最高优先级，99为最低优先级，如给openEuler.repo配置优先级为2：



```
[base]
name=base
baseurl=http://192.168.139.209/
enabled=1
priority=2
gpgcheck=1
gpgkey=http://192.168.139.209/RPM-GPG-KEY-openEuler
```

### dnf相关命令 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html#user-content-dnf相关命令)

dnf命令在安装升级时能够自动解析包的依赖关系，一般的使用方式如下：



```
dnf <command> <packages name>
```

常用的命令如下：

- 安装，需要在root权限下执行。

  

  ```
  # dnf install <packages name>
  ```

- 升级，需要在root权限下执行。

  

  ```
  # dnf update <packages name>
  ```

- 回退，需要在root权限下执行。

  

  ```
  # dnf downgrade <packages name>
  ```

- 检查更新

  

  ```
  # dnf check-update
  ```

- 卸载，需要在root权限下执行。

  

  ```
  # dnf remove <packages name>
  ```

- 查询

  

  ```
  # dnf search <packages name>
  ```

- 本地安装，需要在root权限下执行。

  

  ```
  # dnf localinstall <absolute path to package name>
  ```

- 查看历史记录

  

  ```
  # dnf history
  ```

- 清除缓存目录

  

  ```
  # dnf clean all
  ```

- 更新缓存

  

  ```
  # dnf makecache
  ```

# 搭建FTP服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-搭建ftp服务器)

## 总体介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-总体介绍)

### FTP简介 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-ftp简介)

FTP（File Transfer Protocol）即文件传输协议，是互联网最早的传输协议之一，其最主要的功能是服务器和客户端之间的文件传输。FTP使用户可以通过一套标准的命令访问远程系统上的文件，而不需要直接登录远程系统。另外，FTP服务器还提供了如下主要功能：

- 用户分类

  默认情况下，FTP服务器依据登录情况，将用户分为实体用户（real user）、访客（guest）、匿名用户（anonymous）三类。三类用户对系统的访问权限差异较大，实体用户具有较完整的访问权限，匿名用户仅有下载资源的权限。

- 命令记录和日志文件记录

  FTP可以利用系统的syslogd记录数据，这些数据包括用户历史使用命令与用户传输数据（传输时间、文件大小等），用户可以在/var/log/中获得各项日志信息。

- 限制用户的访问范围

  FTP可以将用户的工作范围限定在用户主目录。用户通过FTP登录后系统显示的根目录就是用户主目录，这种环境被称为change root，简称chroot。这种方式可以限制用户只能访问主目录，而不允许访问/etc、/home、/usr/local等系统的重要目录，从而保护系统，使系统更安全。

### FTP使用到的端口 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-ftp使用到的端口)

FTP的正常工作需要使用到多个网络端口，服务器端会使用到的端口主要有：

- 命令通道，默认端口为21
- 数据通道，默认端口为20

两者的连接发起端不同，端口21主要接收来自客户端的连接，端口20则是FTP服务器主动连接至客户端。

### vsftpd简介 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-vsftpd简介)

由于FTP历史悠久，它采用未加密的传输方式，所以被认为是一种不安全的协议。为了更安全地使用FTP，这里介绍FTP较为安全的守护进程vsftpd（Very Secure FTP Daemon）。

之所以说vsftpd安全，是因为它最初的发展理念就是构建一个以安全为中心的FTP服务器。它具有如下特点：

- vsftpd服务的启动身份为一般用户，具有较低的系统权限。此外，vsftpd使用chroot改变根目录，不会误用系统工具。
- 任何需要较高执行权限的vsftpd命令均由一个特殊的上层程序控制，该上层程序的权限较低，以不影响系统本身为准。
- vsftpd整合了大部分FTP会使用到的额外命令（例如dir、ls、cd等），一般不需要系统提供额外命令，对系统来说比较安全。

## 使用vsftpd [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-使用vsftpd)

### 安装vsftpd [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-安装vsftpd)

使用vsftpd需要安装vsftpd软件，在已经配置yum源的情况下，通过root权限执行如下命令，即可完成vsftpd的安装。



```
# dnf install vsftpd
```

### 管理vsftpd服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-管理vsftpd服务)

启动、停止和重启vsftpd服务，请在root权限下执行对应命令。

- 启动vsftpd服务

  

  ```
  # systemctl start vsftpd
  ```

  可以通过netstat命令查看通信端口21是否开启，如下显示说明vsftpd已经启动。

  

  ```
  # netstat -tulnp | grep 21
  tcp6       0      0 :::21                   :::*                    LISTEN      19716/vsftpd
  ```

  说明：

  如果没有**netstat**命令，可以执行**dnf install net-tools**命令安装后再使用**netstat**命令。

- 停止vsftpd服务

  

  ```
  # systemctl stop vsftpd
  ```

- 重启vsftpd服务

  

  ```
  # systemctl restart vsftpd
  ```

## 配置vsftpd [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-配置vsftpd)

### vsftpd配置文件介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-vsftpd配置文件介绍)

用户可以通过修改vsftpd的配置文件，控制用户权限等。vsftpd的主要配置文件和含义如[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#table1541615718372)所示，用户可以根据需求修改配置文件的内容。更多的配置参数含义可以通过man查看。

**表 1** vsftpd配置文件介绍



| 配置文件                | 含义                                                         |
| :---------------------- | :----------------------------------------------------------- |
| /etc/vsftpd/vsftpd.conf | vsftpd进程的主配置文件，配置内容格式为“参数=参数值”，且参数和参数值不能为空。vsftpd.conf 的详细介绍可以使用如下命令查看：man 5 vsftpd.conf |
| /etc/pam.d/vsftpd       | PAM（Pluggable Authentication Modules）认证文件，主要用于身份认证和限制一些用户的操作。 |
| /etc/vsftpd/ftpusers    | 禁止使用vsftpd的用户列表文件。默认情况下，系统帐号也在该文件中，因此系统帐号默认无法使用vsftpd。 |
| /etc/vsftpd/user_list   | 禁止或允许登录vsftpd服务器的用户列表文件。该文件是否生效，取决于主配置文件vsftpd.conf中的如下参数：userlist_enable：是否启用userlist机制，YES为启用，此时userlist_deny配置有效，NO为禁用。userlist_deny：是否禁止user_list中的用户登录，YES为禁止名单中的用户登录，NO为允许命令中的用户登录。例如userlist_enable=YES，userlist_deny=YES，则user_list中的用户都无法登录。 |
| /etc/vsftpd/chroot_list | 是否限制在主目录下的用户列表。该文件默认不存在，需要手动建立。它是主配置文件vsftpd.conf中参数chroot_list_file的参数值。其作用是限制还是允许，取决于主配置文件vsftpd.conf中的如下参数：chroot_local_user：是否将所有用户限制在主目录，YES为启用，NO禁用。chroot_list_enable：是否启用限制用户的名单，YES为启用，NO禁用。例如chroot_local_user=YES，chroot_list_enable=YES，且指定chroot_list_file=/etc/vsftpd/chroot_list时，表示所有用户被限制在其主目录下，而chroot_list中的用户不受限制。 |
| /usr/sbin/vsftpd        | vsftpd的唯一执行文件。                                       |
| /var/ftp/               | 匿名用户登录的默认根目录，与ftp帐户的用户主目录有关。        |

### 默认配置说明 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-默认配置说明)

说明：

文档中的配置内容仅供参考，请用户根据实际情况（例如安全加固需要）进行修改。

openEuler系统中 ，vsftpd默认不开放匿名用户，使用vim命令查看主配置文件，其内容如下：



```
# vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
```

其中各参数含义如[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#table18185162512499)所示。

**表 2** 参数说明



| 参数                 | 含义                                                         |
| :------------------- | :----------------------------------------------------------- |
| anonymous_enable     | 是否允许匿名用户登录，YES为允许匿名登录，NO为不允许。        |
| local_enable         | 是否允许本地用户登入，YES 为允许本地用户登入，NO为不允许。   |
| write_enable         | 是否允许登录用户有写权限，YES为启用上传写入功能，NO为禁用。  |
| local_umask          | 本地用户新增档案时的umask值。                                |
| dirmessage_enable    | 当用户进入某个目录时，是否显示该目录需要注意的内容，YES为显示注意内容，NO为不显示。 |
| xferlog_enable       | 是否记录使用者上传与下载文件的操作，YES为记录操作，NO为不记录。 |
| connect_from_port_20 | Port模式进行数据传输是否使用端口20，YES为使用端口20，NO为不使用端口20。 |
| xferlog_std_format   | 传输日志文件是否以标准xferlog格式书写，YES为使用该格式书写，NO为不使用。 |
| listen               | 设置vsftpd是否以stand alone的方式启动，YES为使用stand alone方式启动，NO为不使用该方式。 |
| pam_service_name     | 支持PAM模块的管理，配置值为服务名称，例如vsftpd。            |
| userlist_enable      | 是否支持/etc/vsftpd/user_list文件内的帐号登录控制，YES为支持，NO为不支持。 |
| tcp_wrappers         | 是否支持TCP Wrappers的防火墙机制，YES为支持，NO为不支持。    |
| listen_ipv6          | 是否侦听IPv6的FTP请求，YES为侦听，NO为不侦听。listen和listen_ipv6不能同时开启。 |

### 配置本地时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-配置本地时间)

#### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-概述)

openEuler系统中，vsftpd默认使用GMT时间（格林尼治时间），可能和本地时间不一致，例如GMT时间比北京时间晚8小时，请用户改为本地时间，否则服务器和客户端时间不一致，在上传下载文件时可能引起错误。

#### 设置方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-设置方法)

在root权限下设置vsftpd时间为本地时间的操作步骤如下：

1. 打开配置文件vsftpd.conf，将参数use_localtime的参数值改为YES。命令如下：

   

   ```
   # vim /etc/vsftpd/vsftpd.conf
   ```

   配置内容如下：

   

   ```
   use_localtime=YES
   ```

2. 重启vsftpd服务。

   

   ```
   # systemctl restart vsftpd
   ```

3. 设置vsftpd服务开机启动。

   

   ```
   # systemctl enable vsftpd
   ```

### 配置欢迎信息 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-配置欢迎信息)

使用vsftpd服务，建议新建欢迎信息文件（没有也不影响使用）。在root权限下设置vsftpd的欢迎信息welcome.txt文件的操作步骤如下：

1. 打开配置文件vsftpd.conf，加入欢迎信息文件配置内容后保存退出。

   

   ```
   # vim /etc/vsftpd/vsftpd.conf
   ```

   需要加入的配置行如下：

   

   ```
   banner_file=/etc/vsftpd/welcome.txt
   ```

2. 建立欢迎信息。即打开welcome.txt文件，写入欢迎信息后保存退出。

   

   ```
   # vim /etc/vsftpd/welcome.txt
   ```

   欢迎信息举例如下：

   

   ```
   Welcome to this FTP server!
   ```

### 配置系统帐号登录权限 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-配置系统帐号登录权限)

一般情况下，用户需要限制部分帐号的登录权限。用户可根据需要进行配置。

vsftpd有两个默认存放用户名单的文件，来对访问FTP服务的用户身份进行管理和限制。vsftpd会分别检查两个配置文件，只要是被任何一个文件所禁止的用户，FTP访问到本机的请求都会被拒绝。

- /etc/vsftpd/user_list：可以作为用户白名单，或者是黑名单，或者无效名单，由userlist_enable和userlist_deny这两个参数决定。
- /etc/vsftpd/ftpusers：只能是用户黑名单，不受任何参数限制。

## 验证FTP服务是否搭建成功 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-验证ftp服务是否搭建成功)

可以使用openEuler提供的FTP客户端进行验证。命令和回显如下，根据提示输入用户名（用户为系统中存在的用户）和密码。如果显示Login successful，即说明FTP服务器搭建成功。



```
# ftp localhost
Trying 127.0.0.1...
Connected to localhost (127.0.0.1).
220-Welcome to this FTP server!
220
Name (localhost:root): USERNAME
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> bye
221 Goodbye.
```

说明：

如果没有**ftp**命令，可以在root权限下执行**dnf install ftp**命令安装后再使用**ftp**命令。

## 配置防火墙 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-配置防火墙)

如果要将FTP开放给Internet使用，需要在root权限下对防火墙和SElinux进行设置。



```
# firewall-cmd --add-service=ftp --permanent
success
# firewall-cmd --reload
success
# setsebool -P ftpd_full_access on
```

## 传输文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-传输文件)

### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-概述-1)

这里给出vsftpd服务启动后，如何进行文件传输的指导。

### 连接服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-连接服务器)

#### 命令格式 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-命令格式)

**ftp** [*hostname* | *ip-address*]

其中hostname为服务器名称，ip-address为服务器IP地址。

#### 操作说明 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-操作说明)

在openEuler系统的命令行终端，执行如下命令：



```
# ftp ip-address
```

根据提示输入用户名和密码，认证通过后显示如下，说明ftp连接成功，此时进入了连接到的服务器目录。



```
ftp>
```

在该提示符下，可以输入不同的命令进行相关操作：

- 显示服务器当前路径

  

  ```
  ftp>pwd
  ```

- 显示本地路径，用户可以将该路径下的文件上传到FTP服务器对应位置

  

  ```
  ftp>lcd
  ```

- 退出当前窗口，返回本地Linux终端

  

  ```
  ftp>！
  ```

### 下载文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-下载文件)

通常使用get或mget命令下载文件。

#### get使用方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-get使用方法)

- 功能说明：将文件从远端主机中传送至本地主机中

- 命令格式：**get** [*remote-file*] [*local-file*]

  其中 *remote-file* 为远程文件，*local-file* 为本地文件

- 示例：获取远程服务器上的/home/openEuler/openEuler.htm文件到本地/home/myopenEuler/，并改名为myopenEuler.htm，命令如下：

  

  ```
  ftp> get /home/openEuler/openEuler.htm /home/myopenEuler/myopenEuler.htm
  ```

#### mget使用方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-mget使用方法)

- 功能说明：从远端主机接收一批文件至本地主机

- 命令格式：**mget** [*remote-file*]

  其中 *remote-file* 为远程文件

- 示例：获取服务器上/home/openEuler/目录下的所有文件，命令如下：

  

  ```
  ftp> cd /home/openEuler/
  ftp> mget *.*
  ```

  说明：

  

  - 此时每下载一个文件，都会有提示信息。如果要屏蔽提示信息，则在 **mget \*.\*** 命令前先执行**prompt off**
  - 文件都被下载到Linux主机的当前目录下。比如，在/home/myopenEuler/下运行的ftp命令，则文件都下载到/home/myopenEuler/下。

### 上传文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-上传文件)

通常使用put或mput命令上传文件。

#### put使用方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-put使用方法)

- 功能说明：将本地的一个文件传送到远端主机中

- 命令格式：**put** [*local-file*] [*remote-file*]

  其中 *remote-file* 为远程文件，*local-file* 为本地文件

- 示例：将本地的myopenEuler.htm传送到远端主机/home/openEuler/，并改名为openEuler.htm，命令如下：

  

  ```
  ftp> put myopenEuler.htm /home/openEuler/openEuler.htm
  ```

#### mput使用方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-mput使用方法)

- 功能说明：将本地主机中一批文件传送至远端主机

- 命令格式：**mput** [*local-file*]

  其中 *local-file* 为本地文件

- 示例：将本地当前目录下所有htm文件上传到服务器/home/openEuler/下，命令如下：

  

  ```
  ftp> cd /home/openEuler/
  ftp> mput *.htm
  ```

### 删除文件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-删除文件)

通常使用delete或mdelete命令删除文件。

#### delete使用方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-delete使用方法)

- 功能说明：删除远程服务器上的一个或多个文件

- 命令格式：**delete** [*remote-file*]

  其中 *remote-file* 为远程文件

- 示例：删除远程服务器上/home/openEuler/下的openEuler.htm文件，命令如下：

  

  ```
  ftp> cd /home/openEuler/
  ftp> delete openEuler.htm
  ```

#### mdelete使用方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-mdelete使用方法)

- 功能说明：删除远程服务器上的文件，常用于批量删除

- 命令格式：**mdelete** [*remote-file*]

  其中 *remote-file* 为远程文件

- 示例：删除远程服务器上/home/openEuler/下所有a开头的文件，命令如下：

  

  ```
  ftp> cd /home/openEuler/
  ftp> mdelete a*
  ```

### 断开服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_ftp_server.html#user-content-断开服务器)

断开与服务器的连接，使用bye命令，如下：



```
ftp> bye
```

# 搭建web服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-搭建web服务器)

## Apache服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-apache服务器)

### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-概述)

Web（World Wide Web）是目前最常用的Internet协议之一。目前在Unix-Like系统中的web服务器主要通过Apache服务器软件实现。为了实现运营动态网站，产生了LAMP（Linux + Apache +MySQL + PHP）。web服务可以结合文字、图形、影像以及声音等多媒体，并支持超链接（Hyperlink）的方式传输信息。

openEuler系统中的web服务器版本是Apache HTTP服务器2.4版本，即httpd，一个由Apache软件基金会发展而来的开源web服务器。

### 管理httpd [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-管理httpd)

#### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-概述-1)

通过systemctl工具，可以对httpd服务进行管理，包括启动、停止、重启服务，以及查看服务状态等。本章介绍Apache HTTP服务的管理操作，以指导用户使用。

#### 前提条件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-前提条件)

- 为了能够使用Apache HTTP服务，请确保您的系统中已经安装httpd服务的rpm包。在root权限下执行如下命令进行安装：

  

  ```
  # dnf install httpd
  ```

  更多关于管理服务的内容，请参见[管理服务](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html)。

- 启动、停止和重启httpd服务，需要使用root权限。

#### 启动服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-启动服务)

- 启动并运行httpd服务，命令如下：

  

  ```
  # systemctl start httpd
  ```

- 假如希望在系统启动时，httpd服务自动启动，则命令和回显如下：

  

  ```
  # systemctl enable httpd
  Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
  ```

说明：

假如正在运行的Apache HTTP服务器作为一个安全服务器，系统开机启动后需要密码，这个密码使用的是加密的私有SSL密钥。

#### 停止服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-停止服务)

- 停止运行的httpd服务，命令如下：

  

  ```
  # systemctl stop httpd
  ```

- 如果希望防止服务在系统开机阶段自动开启，命令和回显如下：

  

  ```
  # systemctl disable httpd
  Removed /etc/systemd/system/multi-user.target.wants/httpd.service.
  ```

#### 重启服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-重启服务)

重启服务有三种方式：

- 完全重启服务

  

  ```
  # systemctl restart httpd
  ```

  该命令会停止运行的httpd服务并且立即重新启动它。一般在服务安装以后或者去除一个动态加载的模块（例如PHP）时使用这个命令。

- 重新加载配置

  

  ```
  # systemctl reload httpd
  ```

  该命令会使运行的httpd服务重新加载它的配置文件。任何当前正在处理的请求将会被中断，从而造成客户端浏览器显示一个错误消息或者重新渲染部分页面。

- 重新加载配置而不影响激活的请求

  

  ```
  # apachectl graceful
  ```

  该命令会使运行的httpd服务重新加载它的配置文件。任何当前正在处理的请求将会继续使用旧的配置文件。

#### 验证服务状态 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-验证服务状态)

验证httpd服务是否正在运行。



```
# systemctl is-active httpd
```

回显为“active”说明服务处于运行状态。

### 配置文件说明 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-配置文件说明)

当httpd服务启动后，默认情况下它会读取如[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#table24341012096)所示的配置文件。

**表 1** 配置文件说明



| 文件                       | 说明                                                         |
| :------------------------- | :----------------------------------------------------------- |
| /etc/httpd/conf/httpd.conf | 主要的配置文件                                               |
| /etc/httpd/conf.d          | 配置文件的辅助目录，这些配置文件也被包含在主配置文件当中一个配置文件的辅助目录被包含在主要的配置文件中 |

虽然默认配置可以适用于多数情况，但是用户至少需要熟悉里面的一些重要配置项。配置文件修改完成后，可以在root权限下使用如下命令检查配置文件可能出现的语法错误。



```
# apachectl configtest
```

如果回显如下，说明配置文件语法正确。



```
Syntax OK
```

说明：



- 在修改配置文件之前，请先备份原始文件，以便出现问题时能够快速恢复配置文件。
- 需要重启web服务，才能使修改后的配置文件生效。

### 管理模块和SSL [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-管理模块和ssl)

#### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-概述-2)

httpd服务是一个模块化的应用，它和许多动态共享对象DSO（Dynamic Shared Objects）一起分发。动态共享对象DSO，在必要情况下，可以在运行时被动态加载或卸载。服务器操作系统中这些模块位于/usr/lib64/httpd/modules/目录下。本节介绍如何加载和写入模块。

#### 加载模块 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-加载模块)

为了加载一个特殊的DSO模块，在配置文件中使用加载模块指示。独立软件包提供的模块一般在/etc/httpd/conf.modules.d目录下有他们自己的配置文件。

例如，加载asis DSO模块的操作步骤如下：

1. 在/etc/httpd/conf.modules.d/00-optional.conf文件中，使用root权限取消注释如下配置行。

   

   ```
   LoadModule asis_module modules/mod_asis.so
   ```

2. 加载完成后，请使用root权限重启httpd服务以便于重新加载配置文件。

   

   ```
   # systemctl restart httpd
   ```

3. 加载完成后，在root权限下使用httpd -M的命令查看是否已经加载了asis DSO模块。

   

   ```
   # httpd -M | grep asis
   ```

   回显如下，说明asis DSO模块加载成功。

   

   ```
   asis_module (shared)
   ```

说明：

**httpd 的常用命令**

- httpd -v : 查看httpd的版本号。
- httpd -l：查看编译进httpd程序的静态模块。
- httpd -M：查看已经编译进httpd程序的静态模块和已经加载的动态模块。

#### SSL介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-ssl介绍)

安全套接层SSL（Secure Sockets Layer）是一个允许服务端和客户端之间进行安全通信的加密协议。其中，传输层安全性协议TLS（Transport Layer Security）为网络通信提供了安全性和数据完整性保障。openEuler支持Mozilla NSS（Network Security Services）作为安全性协议TLS进行配置。加载SSL的操作步骤如下：

1. 在root权限下安装mod_ssl的rpm包。

   

   ```
   # dnf install mod_ssl
   ```

2. 安装完成后，请在root权限下重启httpd服务以便于重新加载配置文件。

   

   ```
   # systemctl restart httpd
   ```

3. 加载完成后，在root权限下使用httpd -M的命令查看是否已经加载了SSL。

   

   ```
   # httpd -M | grep ssl
   ```

   回显如下，说明SSL已加载成功。

   

   ```
   ssl_module (shared)
   ```

### 验证web服务是否搭建成功 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-验证web服务是否搭建成功)

Web服务器搭建完成后，可以通过如下方式验证是否搭建成功。

1. 在root权限下查看服务器的IP地址，命令如下：

   

   ```
   # ip a
   ```

2. 在root权限下配置防火墙：

   

   ```
   # firewall-cmd --add-service=http --permanent
   success
   # firewall-cmd --reload
   success
   ```

3. 验证web服务器是否搭建成功，用户可选择Linux或Windows系统进行验证。

   - 使用Linux系统验证

     执行如下命令，查看是否可以访问网页信息，服务搭建成功时，该网页可以正常访问。

     

     ```
     # curl http://192.168.1.60
     ```

     执行如下命令，查看命令返回值是否为0，返回值为0，说明httpd服务器搭建成功。

     

     ```
     # echo $?
     ```

   - 使用Windows系统验证

     打开浏览器，在地址栏输入如下地址，如果能正常访问网页，说明httpd服务器搭建成功。

     [http://192.168.1.60](http://192.168.1.60/)

     如果修改了端口号，输入地址格式如下：

     [http://192.168.1.60:端口号](http://192.168.1.60:80/)

## Nginx服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-nginx服务器)

### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-概述-3)

Nginx 是一款轻量级的 Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，其特点是占有内存少，并发能力强，支持FastCGI、SSL、Virtual Host、URL Rewrite、Gzip等功能，并且支持很多第三方的模块扩展。

### 安装 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-安装)

1. 配置本地yum源，详细信息请参考[搭建repo服务器](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html)。

2. 清除缓存。

   

   ```
   # dnf clean all
   ```

3. 创建缓存。

   

   ```
   # dnf makecache
   ```

4. 在root权限下安装nginx服务。

   

   ```
   # dnf install nginx
   ```

5. 查看安装后的rpm包。

   

   ```
   # dnf list all | grep nginx
   ```

### 管理nginx [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-管理nginx)

#### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-概述-4)

通过systemctl工具，可以对nginx服务进行管理，包括启动、停止、重启服务，以及查看服务状态等。本章介绍nginx服务的管理操作，以指导用户使用。

#### 前提条件 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-前提条件-1)

- 为了能够使用nginx服务，请确保您的系统中已经安装nginx服务。若未安装，可参考[安装](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#安装)进行安装。
- 更多关于管理服务的内容，请参见[管理服务](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/service_management.html)。
- 启动、停止和重启nginx服务，需要使用root权限。

#### 启动服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-启动服务-1)

- 启动并运行nginx服务，命令如下：

  

  ```
  # systemctl start nginx
  ```

- 假如希望在系统启动时，nginx服务自动启动，则命令和回显如下：

  

  ```
  # systemctl enable nginx
  Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
  ```

说明：

假如正在运行的nginx服务器作为一个安全服务器，系统开机启动后需要密码，这个密码使用的是加密的私有SSL密钥。

#### 停止服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-停止服务-1)

- 停止运行的nginx服务，命令如下：

  

  ```
  # systemctl stop nginx
  ```

- 如果希望防止服务在系统开机阶段自动开启，命令和回显如下：

  

  ```
  # systemctl disable nginx
  Removed /etc/systemd/system/multi-user.target.wants/nginx.service.
  ```

#### 重启服务 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-重启服务-1)

重启服务有三种方式：

- 完全重启服务

  

  ```
  # systemctl restart nginx
  ```

  该命令会停止运行的nginx服务并且立即重新启动它。一般在服务安装以后或者去除一个动态加载的模块（例如PHP）时使用这个命令。

- 重新加载配置

  

  ```
  # systemctl reload nginx
  ```

  该命令会使运行的nginx服务重新加载它的配置文件。任何当前正在处理的请求将会被中断，从而造成客户端浏览器显示一个错误消息或者重新渲染部分页面。

- 平滑重启nginx

  

  ```
  # kill -HUP 主进程ID
  ```

  该命令会使运行的nginx服务重新加载它的配置文件。任何当前正在处理的请求将会继续使用旧的配置文件。

#### 验证服务状态 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-验证服务状态-1)

验证nginx服务是否正在运行



```
# systemctl is-active nginx
```

回显为“active”说明服务处于运行状态。

### 配置文件说明 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-配置文件说明-1)

当nginx服务启动后，默认情况下它会读取如[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#table24341012096)所示的配置文件。

**表 2** 配置文件说明



| 文件                  | 说明                                                         |
| :-------------------- | :----------------------------------------------------------- |
| /etc/nginx/nginx.conf | 主要的配置文件                                               |
| /etc/nginx/conf.d     | 配置文件的辅助目录，这些配置文件也被包含在主配置文件当中一个配置文件的辅助目录被包含在主要的配置文件中 |

虽然默认配置可以适用于多数情况，但是用户至少需要熟悉里面的一些重要配置项。配置文件修改完成后，可以在root权限下使用如下命令检查配置文件可能出现的语法错误。



```
# nginx -t
```

如果回显信息中有“syntax is ok”，说明配置文件语法正确。

说明：



- 在修改配置文件之前，请先备份原始文件，以便出现问题时能够快速恢复配置文件。
- 需要重启web服务，才能使修改后的配置文件生效。

### 管理模块 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-管理模块)

#### 概述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-概述-5)

nginx服务是一个模块化的应用，它和许多动态共享对象DSO（Dynamic Shared Objects）一起分发。动态共享对象DSO，在必要情况下，可以在运行时被动态加载或卸载。服务器操作系统中这些模块位于/usr/lib64/nginx/modules/目录下。本节介绍如何加载和写入模块。

#### 加载模块 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-加载模块-1)

为了加载一个特殊的DSO模块，在配置文件中使用加载模块指示。独立软件包提供的模块一般在/usr/share/nginx/modules目录下有他们自己的配置文件。

openEuler操作系统中使用dnf install nginx安装nginx时会自动加载DSO。

### 验证web服务是否搭建成功 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_web_server.html#user-content-验证web服务是否搭建成功-1)

Web服务器搭建完成后，可以通过如下方式验证是否搭建成功。

1. 在root权限下查看服务器的IP地址，命令如下：

   

   ```
   # ip a
   ```

   回显信息如下，说明服务器IP为 192.168.1.60。

   

   ```
   enp3s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
   inet 192.168.1.60  netmask 255.255.255.0  broadcast 192.168.1.255
   inet6 fe80::5054:ff:fe95:499f  prefixlen 64  scopeid 0x20<link>
   ether 52:54:00:95:49:9f  txqueuelen 1000  (Ethernet)
   RX packets 150713207  bytes 49333673733 (45.9 GiB)
   RX errors 0  dropped 43  overruns 0  frame 0
   TX packets 2246438  bytes 203186675 (193.7 MiB)
   TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
   
   enp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
   ether 52:54:00:7d:80:9e  txqueuelen 1000  (Ethernet)
   RX packets 149937274  bytes 44652889185 (41.5 GiB)
   RX errors 0  dropped 1102561  overruns 0  frame 0
   TX packets 0  bytes 0 (0.0 B)
   TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
   
   lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
   inet 127.0.0.1  netmask 255.0.0.0
   inet6 ::1  prefixlen 128  scopeid 0x10<host>
   loop  txqueuelen 1000  (Local Loopback)
   RX packets 37096  bytes 3447369 (3.2 MiB)
   RX errors 0  dropped 0  overruns 0  frame 0
   TX packets 37096  bytes 3447369 (3.2 MiB)
   TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
   ```

2. 在root权限下配置防火墙：

   

   ```
   # firewall-cmd --add-service=http --permanent
   success
   # firewall-cmd --reload
   success
   ```

3. 验证web服务器是否搭建成功，用户可选择Linux或Windows系统进行验证。

   - 使用Linux系统验证

     执行如下命令，查看是否可以访问网页信息，服务搭建成功时，该网页可以正常访问。

     

     ```
     # curl http://192.168.1.60
     ```

     执行如下命令，查看命令返回值是否为0，返回值为0，说明nginx服务器搭建成功。

     

     ```
     # echo $?
     ```

   - 使用Windows系统验证

     打开浏览器，在地址栏输入如下地址，如果能正常访问网页，说明nginx服务器搭建成功。

     ```
     [http://192.168.1.60](http://192.168.1.60/)
     
     如果修改了端口号，输入地址格式如下：
     
     [http://192.168.1.60:端口号](http://192.168.1.60:80/)
     ```

   

   # 搭建数据库服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-搭建数据库服务器)

   ## PostgreSql服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-postgresql服务器)

   ### 软件介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-软件介绍)

   PostgreSQL的架构如[图1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#fig26022387391)所示，主要进程说明如[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#table62020913417)所示。

   **图 1** PostgreSql架构
   ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/PostgreSql_architecture.CbqNbLYm.png)

   **表 1** PostgreSql中的主要进程说明

   

   | 进程类别                   | 进程名称                                                     | 说明                                                         |
   | :------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
   | 主进程                     | Postmaster                                                   | Postmaster是整个数据库实例的总控进程，负责启动和关闭该数据库实例。 |
   | 常驻进程                   | Postgres（常驻进程）                                         | 管理后端的常驻进程，也称为“postmaster”。其默认侦听UNIXDomain Socket和TCP/IP（Windows等，一部分的平台只侦听TCP/IP）的5432端口，等待来自前端的连接处理。侦听的端口号可以在PostgreSql的设置文件postgresql.conf中修改。 |
   | 子进程                     | Postgres（子进程）                                           | 子进程根据pg_hba.conf定义的安全策略来判断是否允许进行连接，根据策略，会拒绝某些特定的IP及网络，或者也可以只允许某些特定的用户或者对某些数据库进行连接。Postgres会接受前端过来的查询，然后对数据库进行检索，最后把结果返回，有时也会对数据库进行更新。更新的数据同时还会记录在事务日志里面（PostgreSQL称为WAL日志）。这个主要是当停电、服务器宕机、重新启动的时候进行恢复处理的时候使用。另外，把日志归档保存起来，可在需要进行恢复的时候使用。在PostgreSQL 9.0以后，通过把WAL日志传送其他的postgreSQL，可以实时的进行数据库复制，这就是所谓的“数据库复制”功能。 |
   | 辅助进程                   | SysLogger（系统日志）                                        | 需要在Postgres.conf中logging_collection设置为on，此时主进程才会启动Syslogger辅助进程。 |
   | BgWriter（后台写）         | 把共享内存中的脏页写到磁盘上的进程。主要是为了提高插入、更新和删除数据的性能。 |                                                              |
   | WALWriter（预写式日志）    | 在修改数据之前把修改操作记录到磁盘中，以便后面更新实时数据时就不需要数据持久化到文件中。 |                                                              |
   | PgArch（归档）             | WAL日志会被循环使用，PgArch在归档前会把WAL日志备份出来。通过PITY（Point in Time Recovery）技术，可以在数据库进行一次全量备份后，将全量备份时间点之后的WAL日志通过归档进行备份，然后凭借数据库的全量备份再加上后面产生的WAL日志，即可把数据库向前推到全量备份后的任意一个时间点。 |                                                              |
   | AutoVacuum（系统自动清理） | 在PostgreSQL数据库中，对表进行DELETE操作后，旧的数据并不会立即被删除，并且，在更新数据时，也并不会在旧的数据上做更新，而是新生成一行数据。旧的数据只是被标识为删除状态，只有在没有并发的其他事务读到这些就数据时，它们才会被清除。这个清除工作就由AutoVacuum进程完成。 |                                                              |
   | PgStat（统计收集）         | 做数据的统计收集工作。主要用于查询优化时的代价估算，包括一个表和索引进行了多少次的插入、更新、删除操作，磁盘块读写的次数、行的读次数。pg_statistic中存储了PgStat收集的各类信息。 |                                                              |
   | CheckPoint（检查点）       | 检查点是系统设置的事务序列点，设置检查点保证检查点前的日志信息刷到磁盘中。 |                                                              |

   ### 配置环境 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-配置环境)

   说明：

   以下环境配置仅为参考示例，具体配置视实际需求做配置

   #### 关闭防火墙并取消开机自启动 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-关闭防火墙并取消开机自启动)

   说明：

   测试环境下通常会关闭防火墙以避免部分网络因素影响，视实际需求做配置。

   1. 在root权限下停止防火墙。

      

      ```
      systemctl stop firewalld
      ```

   2. 在root权限下关闭防火墙。

      

      ```
      systemctl disable firewalld
      ```

      说明：

      执行disable命令关闭防火墙的同时，也取消了开机自启动。

   #### 修改SELINUX为disabled [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改selinux为disabled)

   在root权限下修改配置文件。

   

   ```
   sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
   ```

   #### 创建组和用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建组和用户)

   说明：

   服务器环境下，为了系统安全，通常会为进程分配单独的用户，以实现权限隔离。本章节创建的组和用户都是操作系统层面的，不是数据库层面的。

   1. 在root权限下创建PostgreSQL用户（组）。

      

      ```
      groupadd postgres
      useradd -g postgres postgres
      ```

   2. 在root权限下设置postgres用户密码（重复输入密码）。

      

      ```
      passwd postgres
      ```

   #### 搭建数据盘 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-搭建数据盘)

   说明：

   

   - 测试极限性能时，建议单独挂载IO性能更优的NVME SSD存储介质创建PostgreSQL测试实例，避免磁盘IO对性能测试结果的影响，本文以单独挂载NVME SSD为例，参考步骤1~步骤4。
   - 非性能测试时，在root权限下执行以下命令，创建数据目录即可。然后跳过本小节：
     mkdir /data

   1. 在root权限下创建文件系统（以xfs为例，根据实际需求创建文件系统），若磁盘之前已做过文件系统，执行此命令会出现报错，可使用-f参数强制创建文件系统。

      

      ```
      mkfs.xfs /dev/nvme0n1
      ```

   2. 在root权限下创建数据目录。

      

      ```
      mkdir /data
      ```

   3. 在root权限下挂载磁盘。

      

      ```
      mount -o noatime,nobarrier /dev/nvme0n1 /data
      ```

   #### 数据目录授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-数据目录授权)

   1. 在root权限下修改目录权限。

      

      ```
      chown -R postgres:postgres /data/
      ```

   ### 安装、运行和卸载 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装运行和卸载)

   #### 安装 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装)

   1. 配置本地yum源，详细信息请参考[搭建repo服务器](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html)。

   2. 清除缓存。

      

      ```
      dnf clean all
      ```

   3. 创建缓存。

      

      ```
      dnf makecache
      ```

   4. 在root权限下安装PostgreSQL服务器。

      

      ```
      dnf install postgresql-server
      ```

   5. 查看安装后的rpm包。

      

      ```
      rpm -qa | grep postgresql
      ```

   #### 运行 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-运行)

   ##### 初始化数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-初始化数据库)

   须知：

   此步骤在postgres用户下操作。

   1. 切换到已创建的PostgreSQL用户。

      

      ```
      su - postgres
      ```

   2. 初始化数据库，其中命令中的/usr/bin是命令initdb所在的目录。

      

      ```
      usr/bin/initdb -D /data/
      ```

   ##### 启动数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-启动数据库)

   1. 启动PostgreSQL数据库。

      

      ```
      /usr/bin/pg_ctl -D /data/ -l /data/logfile start
      ```

   2. 确认PostgreSQL数据库进程是否正常启动。

      

      ```
      ps -ef | grep postgres
      ```

      命令执行后，打印信息如下图所示，PostgreSQL相关进程已经正常启动了。

      ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/postgres.KyOKQoiA.png)

   ##### 登录数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-登录数据库)

   1. 登录数据库。

      

      ```
      /usr/bin/psql -U postgres
      ```

      ![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAZ4AAABTCAIAAAAgH/aGAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAMu0lEQVR4nO2dO4jjSBrHy8ulzSVqjrukGdHjoTeYTAp6kwtEg8zCXOhIoTG0YQ83lzlU2sILNvg2O0XiomsYZGgUbDSBlF2wDW2j5aJLKlkmucwX6C2VXn5L/f8x0ExZ/qokVf391VcvstlsSAGCalNDKbpidwTFsCmllFLbUITD5gUAeBuUSBsAADSQb5L/VQzPfTq0o3Zqbq9WG3GzETcb0Rwc0/7FdHWYHHdAUO3GvvWyXsUxeh0VCB9xIx9yM/kmk+IuZI7r6ycoyxH58p/3HbvTeflx3Uz7e8WZiBw3tk5djIPgTMRzqMzOROQ4jpMX7qlL8nbIStshUYyT/GoJihpF89QWRfMuzQN5nRGKscULO7Sv1HT725NsQccu54na73YcV9pOgmIstbv1TOY4jpOf+eG8TeJ2SBTDplSTJI3atqEI1Z+a8jDkLfNwvlLT7YNjUCZtgmpTQw2cnuQIpqCyRjaj8c54shds0CRCJC0dcwgDEbaqGtSOlEcxqG2EmYTfYI+osvMliqFJ1lic6A4hhDiTmcXffSpupbfTD36kbPVhcBtLH0Tp0yj9YjBlphdwaa68SNxHc3BRku9teLG4Mi+D5IvpStxs3smEyH/3Pv2wZ+9NMTTJHctjyxpz4r3ZffhU9eda6Unu4tFThpRfoUTvN+d9BVcx3jvLPiGEkC6zfgZx48T3vfIUD8lnys+s//nlj6yn6vMJaHz73YHkCKmSfhVesb2sBEW1o4895zSTrhjUVqN7UBPVkuXQpuykHg2ltpqp9/71RCjPN8zRf3K2YcTaGiuoP/iw2XjKcjGYftysrm6j9I9TT4luL6fTy+D6K3PqK87t9ENSYrL2L6arSLluB7Hrc/IdmOIqyOt28GGasLbHDmn61QSaVLtDmhSzXGkrqifM955vn10/mfl71wdNR2DVx4r2c8tfUJ/30IGs2yFtevvdhQrSltLhWNVkp1Nbzem+MB5Nnh1maWrnG777IOdkbWZI28AUV9PQk7o0Nx89RyyZnkd0fY79i+mqiv1E+mZ1NbhlZl1D2pTYD7dXD7NyYijpb9iGYddzO1LvrEjacusJ670X2C+qMSxpK6xhVe3nlb+wBp9G2hrcfnehSqzNfXUyaUKXZ6YTvS8v3LvRfFktZp9rp+B6frgMRtI1qVq+QpcnlqkT4rysCwepLr59T1a/fK2cTgi5nK4+bvypHu/kCvdQy/5PvZcfV7//2z9uNhtxZV5V6u9mEdTe+p7jOE6+N3vzJaV03ls/FUeT9L7IiY9rQq4fbNuu1iIFdSRZs0mF91mznhTZr1N/Sq6vbj+n/HXrcxHRjJFIDEqqL5MWtN9tqCJtfDebkfPqMtMJIc6kL4oix8ljVxo+lLSIAju511tjLoYYVMS8fPmuQJxXl0g9hRDh5povsv/1lxV5/23WRcpLJwPz3Q+r/37XsTsdu9P5dVn5TiraJ+TrX3v/fv/e7nz360r+w2S77qcz6XuPyXH0vshxHCf2qwgQcV5d1+yLM1caVahuteLvtepJkf069WeP9pnlr1ufi/BnjMTr+FY0vP1uSyVpGz4EfemR5D4/OYQQopsWK11QjZwYCSHeb07qQaTtlBVGNy1JC8KMgqL63aqcfJ1Xl/DXN4Toj2OL1yid90jsZ+/rLysi/+Uy/o2f/vXb9Q9/CmJef5TXv/3zS5j+jhFrI2T9Sr6Q4PpE9gz7eeTkezE1rwa5rtr/Xtd5grgrgmrEgsjdwt+DgEx833sBgSrG329xPalun5Cc+rkNNeznlr+wPoe/sFvjW/Cf56c73l2/lH2nwe13JyrE2mqOsBTOIVMY416B420bjBEWhq8QH2GJnkdOvkI2tJwgGH9cRX29eiOkt1ex4c6rZKwtaz8VfUvE5nJGSGP2s8G+sEi7jpBmwyiCYth1xqzyAj/x8bDY+y2oJ+z3zrSfqZ/xD5J4o0m5I7YV7Veo54UjpGHMc+s+V5RxhdfS+Pa7AxlpS5b7yJMCiwLI2yKoNtbdF1G40KryCOkhXt3p7e9Y/w9d6FJa0H635vQ7fyhq6J+WeFhbE3NAzufJN4OK0tb09QF59iFtpRyh/W7H6aUNmxqBswXSViGPM22/2NQIANA+9ryGdJfNWxTj2EtvlbrzUQEATeEgXtsWbvBJXPfz3eEBALAT57Lzh9JLz0hizcUOPgnHssvFMDUJIClkzmRmVZqLCgBoFmcibVllC+ZiZ7ZIVIzlkDzLHMfJMzJcVnG63IUczH1Ob0v4snb565sdSw8AODcCaTvt5i01VqJ5M8YnDiHE0R8X7m6Tu4nz9LyrCQDA+RH32qThtXnPcZw8fua1YMNG5UHjn/1FX/dmL9q0y9/Py/Oe7koXWBRyc11lwQghaRGsuIItWI/bri12AQD5xKXNXTzqnjeU3LCRv+t6m404en8SdOj89Xbh9ccsdGJyYEl/MlpjLI+f+WodWABA00lI24k2b9ki4uVMxGDLgIruHiGOPrnfuQMLAGgEcWk71eYtdawlr9xdYIVPd9gGH4D2kZC2023eopvlRxbE81W9fDPbawnZLZuj8gvqPLMdV/UoHwCgSfhTdk++eYugplcGhMd2pGe3Fcxry0hb/KC+7BI3TNkFoK3Epe20y4Cx0AoAsC9+d+oCROh97sgxL70vIsoGQCs5k9UIAACwV7CpEQCgfZyr11Yz9rfLZkr1iMYw9hqlO9v7BaCZnKu01cRfcyAv6p/SWA/lYcg/y7sfoLYjR7tfABqKL22piRbHPUbA91iOPhNji3z3uwgDAHAo/BHScHRSMehoLZ/UIwEAgF0p7ZCmzqcMXJzC8w33dgwE21Ad+0XlrIUX3VoOeSJpeVOIs7s+UdtQGSc35nFG9wtAsymVtuSeaImlTcxNkIhiLDUykzmO4+R4en2kUc+89/bscCUtWL1a3z67nDXxolvywiWWt8VT4NkqxnLolYdln5fu1jOZvQ3mWd8vAA0nNfmDsawgFouKXLjUkYPR15Qq50eXs539bNgs1852ZDMoKk+N3M70fgFoKhVWIzhPz+5ypAr65KYnuQs59D1Y8XShyxNeWtJhmOKuty3bvuwfMu4vdHnimvux34T7BaAhVFlo5Uxm1nD0SXi9ltznx6jZ8F2BkFQrcl5dYs1K+15bs419Vjn3WZ67A9s/p/sFoCFUm9emmxY/XGqpo1lYmyAR3bQkLYheC4pqqPudz1Fon72ZErOceyzPoe3v8X6zO6MA0E4qTtnVTYsQYs0Sk0Ksxbo3p5QutTt3fB98pPflMRl5hxHMe8R82rMDV2hf748tXktNymOXc4/lWXjlOZj9s7pfAJpBxTWk6YB1U3Y6a0o598Vbu18AcqjotWX3swUAgPOlfBhBMagmEdcaY28zAEBjwKZGAID20ZKdPwAAIA6kDQDQQiBtAIAWAmkDALQQSBsAoIVA2gAALQTSBgBoIZA2AEALgbQBAFoIpA0A0EIgbQCAFgJpAwC0EEgbAKCFQNoAAC2kyrEvb4jPn39mpn///Z+PWg4AwG7AawMAtJA3Im2CatuqEP0FALSbw0hbdMr8mXBzzbuvDhE+3fE4gBiAN8Db8NqELu+uXwi5ucbhNQC8CQJpE1SbGqpq2JRSSu3gTF//Mz85kS4oQWo8WVBtSqkmESJp3oeR9+Z9Rim1VdWIH5ypGNQ2wkzCb0Q5lOebh2JQSulyyPPDJaWaRCQNZwwD8Abwj33xZMcITh23o+bvdS4z6YpBbTXSoOQZ8awOacpOStootdWkTkXXE6E031A144QlUAxqKJUCbZ8//8z8V/IQAQDnRkzaUn6U9z8l41+F6dRWFYEpFQxpy7OT/V/xFYX5svGVLfxbAKQNgHYQj7WxAuxCl2emE70vL9y70XxJKbWN0h5erp2C6/nhMnDANGmrfL3uqN8NDf6e1fgGAOAgxKWN72aFwnl1memEEGfSF0WR4+SxKw0fSvSiwE7u9daYiyFOnPx8czqkep/j5IXrLuToL9fHOAIArSchbcOHIBY2ktznJ4cQQnTTYqULqqHmx/Bf1hkhS9spK5huWpIWDBMIiupH1XLydSYil8HTsGjeB/FvCQDQeuLSZi3WvTmldKndueP7wEnS+/KCjJapdGfySHpzSimlyxFZyElXyJnMLOJ3J4P+n96XF7y2pJTOe+TZKi2Z3pfHXr6UznvEfNLL82UQm/eBGW0AvBk6m82m0+kQQbWX17OjddYUg47WctjHPBuwhhSAdnDEKbuKGvYvYx1eAADYP0eUNv3JDLqS8Q4vAADsH39eGwAAtIi3sYYUAPDGgLQBAFoIpA0A0EIgbQCAFgJpAwC0EEgbAKCF/B+voS+V4E8bXgAAAABJRU5ErkJggg==)

      说明：

      初次登录数据库，无需密码。

   ##### 配置数据库帐号密码 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-配置数据库帐号密码)

   1. 登录后，设置postgres密码。

      

      ```
      postgres=#alter user postgres with password '123456';
      ```

      ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/zh-cn_image_0230050789.BSkZWcr6.png)

   ##### 退出数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-退出数据库)

   1. 执行\q退出数据库。

      

      ```
      postgres=#\q
      ```

   ##### 停止数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-停止数据库)

   1. 停止PostgreSQL数据库。

      

      ```
      /usr/bin/pg_ctl -D /data/ -l /data/logfile stop
      ```

   #### 卸载 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-卸载)

   1. 在postgres用户下停止数据库。

      

      ```
      /usr/bin/pg_ctl -D /data/ -l /data/logfile stop
      ```

   2. 在root用户下执行**dnf remove postgresql-server**卸载PostgreSQL数据库。

      

      ```
      dnf remove postgresql-server
      ```

   ### 管理数据库角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库角色)

   #### 创建角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建角色)

   可以使用CREATE ROLE语句或createuser来创建角色。createuser是对CREATE ROLE命令的封装，需要在shell界面执行，而不是在数据库界面。

   

   ```
   CREATE ROLE rolename [ [ WITH ] option [ ... ] ];
   ```

   

   ```
   createuser rolename
   ```

   其中：

   - rolename：角色名。
   - option为参数选项，常用的有：
     - SUPERUSER | NOSUPERUSER：决定一个新角色是否为"超级用户"，若未指定，则默认为NOSUPERUSER，即不是超级用户。
     - CREATEDB | NOCREATEDB：定义一个角色是否能创建数据库，若未指定，则默认为NOCREATEDB，即不能创建数据库。
     - CREATEROLE | NOCREATEROLE：决定一个角色是否可以创建新角色，若未指定，则默认为NOCREATEROLE，即不能创建新角色。
     - INHERIT | NOINHERIT：决定一个角色是否"继承"它所在组的角色的权限。一个带有 INHERIT 属性的角色可以自动使用已经赋与它直接或间接所在组的任何权限。若未指定，则默认为INHERIT。
     - LOGIN | NOLOGIN：决定一个角色是否可以登录，一个拥有LOGIN属性的角色可以认为是一个用户，若无此属性的角色可以用于管理数据库权限，但是并不是用户，若未指定，则默认为NOLOGIN。但若创建角色是使用的是CREATE USER而不是CREATE ROLE，则默认是LOGIN属性。
     - [ ENCRYPTED | UNENCRYPTED ] PASSWORD 'password'：设置角色的密码，密码只对那些拥有 LOGIN 属性的角色有意义。ENCRYPTED | UNENCRYPTED表示是否对密码进行加密，若未指定，则默认为ENCRYPTED，即加密。
     - VALID UNTIL 'timestamp'：角色的密码失效的时间戳，若为指定，则表示密码永久有效。
     - IN ROLE rolename1：列出一个或多个现有的角色，新角色rolename将立即加入这些角色，成为rolename1的成员。
     - ROLE rolename2：列出一个或多个现有的角色，它们将自动添加为新角色rolename的成员，即新角色为"组"。

   要使用这条命令，必须拥有 CREATEROLE 权限或者是数据库超级用户。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例)

   创建一个可以登录的角色roleexample1。

   

   ```
   postgres=# CREATE ROLE roleexample1 LOGIN;
   ```

   创建一个密码为123456的角色roleexample2。

   

   ```
   postgres=# CREATE ROLE roleexample2 WITH LOGIN PASSWORD '123456';
   ```

   创建角色名为roleexample3的角色。

   

   ```
   [postgres@localhost ~]# createuser roleexample3
   ```

   #### 查看角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看角色)

   可以使用SELECT语句或psql的元命令\du查看角色。

   

   ```
   SELECT rolename FROM pg_roles;
   ```

   

   ```
   \du
   ```

   其中：rolename：角色名。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-1)

   查看roleexample1角色。

   

   ```
   postgres=# SELECT roleexample1 from pg_roles;
   ```

   查看现有角色。

   

   ```
   postgres=# \du
   ```

   #### 修改角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改角色)

   ##### 修改用户名 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户名)

   可以使用ALTER ROLE语句修改一个已经存在的角色名。

   

   ```
   ALTER ROLE oldrolername RENAME TO newrolename;
   ```

   其中：

   - oldrolername：旧的角色名。
   - newrolename：新的角色名。

   ##### 修改用户示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户示例)

   将角色名roleexample1修改为roleexapme2。

   

   ```
   postgres=# ALTER ROLE roleexample1 RENAME TO roleexample2;
   ```

   ##### 修改用户密码 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户密码)

   可以使用ALTER ROLE语句修改一个角色的登录密码。

   

   ```
   ALTER ROLE rolename PASSWORD 'password'
   ```

   其中：

   - rolename：角色名。
   - password：密码。

   ##### 修改角色密码示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改角色密码示例)

   将roleexample1的密码修改为456789。

   

   ```
   postgres=# ALTER ROLE roleexample1 WITH PASSWORD '456789';
   ```

   #### 删除角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除角色)

   可以使用DROP ROLE语句或dropuser来删除角色。dropuser是对DROP ROLE命令的封装，需要在shell界面执行，而不是在数据库界面。

   

   ```
   DROP ROLE rolename;
   ```

   

   ```
   dropuser rolename
   ```

   其中：rolename为角色名。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-2)

   删除userexample1角色。

   

   ```
   postgres=# DROP ROLE userexample1;
   ```

   删除userexample2角色。

   

   ```
   [postgres@localhost ~]# dropuser userexample2
   ```

   #### 角色授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-角色授权)

   可以使用GRANT语句来对角色授权。

   对角色授予表的操作权限：

   

   ```
   GRANT { { SELECT | INSERT | UPDATE | DELETE | REFERENCES | TRIGGER } [,...] | ALL [ PRIVILEGES ] } ON [ TABLE ] tablename [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   对角色授予序列的操作权限：

   

   ```
   GRANT { { USAGE | SELECT | UPDATE } [,...] | ALL [ PRIVILEGES ] } ON SEQUENCE sequencename [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   对角色授予数据库的操作权限：

   

   ```
   GRANT { { CREATE | CONNECT | TEMPORARY | TEMP } [,...] | ALL [ PRIVILEGES ] } ON DATABASE databasename [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   对角色授予函数的操作权限：

   

   ```
   GRANT { EXECUTE | ALL [ PRIVILEGES ] } ON FUNCTION funcname ( [ [ argmode ] [ argname ] argtype [, ...] ] ) [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   对角色授予过程语言的操作权限：

   

   ```
   GRANT { USAGE | ALL [ PRIVILEGES ] } ON LANGUAGE langname [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   对角色授予模式的操作权限：

   

   ```
   GRANT { { CREATE | USAGE } [,...] | ALL [ PRIVILEGES ] } ON SCHEMA schemaname [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   对角色授予表空间的操作权限：

   

   ```
   GRANT { CREATE | ALL [ PRIVILEGES ] } ON TABLESPACE tablespacename [, ...] TO { rolename | GROUP groupname | PUBLIC } [, ...] [ WITH GRANT OPTION ]
   ```

   将角色rolename1的成员关系赋予角色rolename2：

   

   ```
   GRANT rolename1 [, ...] TO rolename2 [, ...] [ WITH ADMIN OPTION ]
   ```

   其中：

   - SELECT、INSERT、UPDATE、DELETE、REFERENCES、TRIGGER、USAGE、CREATE、CONNECT、TEMPORARY、TEMP、EXECUTE、ALL [ PRIVILEGES ]：用户的操作权限，ALL [ PRIVILEGES ]表示所有的权限，PRIVILEGES关键字在 PostgreSQL里是可选的，但是严格的SQL 要求有这个关键字。
   - ON字句：用于指定权限授予的对象。
   - tablename：表名。
   - TO字句：用来指定被赋予权限的角色。
   - rolename、rolename1、rolename2：角色名。
   - groupname：角色组名。
   - PUBLIC：表示该权限要赋予所有角色，包括那些以后可能创建的用户。
   - WITH GRANT OPTION：表示权限的接收者也可以将此权限赋予他人，否则就不能授权他人。该选项不能赋予给PUBLIC。
   - sequencename：序列名。
   - databasename：数据库名。
   - funcname ( [ [ argmode ] [ argname ] argtype [, ...] ] )：函数名及其参数。
   - langname：过程语言名。
   - schemaname：模式名。
   - tablespacename：表空间名。
   - WITH ADMIN OPTION：表示成员随后就可以将角色的成员关系赋予其他角色，以及撤销其他角色的成员关系。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-3)

   对userexample授予数据库database1的CREATE权限。

   

   ```
   postgres=# GRANT CREATE ON DATABASE database1 TO userexample;
   ```

   对所有用户授予表table1的所有权限。

   

   ```
   postgres=# GRANT ALL PRIVILEGES ON TABLE table1 TO PUBLIC;
   ```

   #### 删除用户权限 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除用户权限)

   可以使用REVOKE语句来撤销以前赋予一个或多个角色的权限。

   撤销角色对表的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { { SELECT | INSERT | UPDATE | DELETE | REFERENCES | TRIGGER } [,...] | ALL [ PRIVILEGES ] } ON [ TABLE ] tablename [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...]
   ```

   撤销角色对序列的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { { USAGE | SELECT | UPDATE } [,...] | ALL [ PRIVILEGES ] } ON SEQUENCE sequencename [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...] [ CASCADE | RESTRICT ]
   ```

   撤销角色对数据库的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { { CREATE | CONNECT | TEMPORARY | TEMP } [,...] | ALL [ PRIVILEGES ] } ON DATABASE databasename [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...] [ CASCADE | RESTRICT ]
   ```

   撤销角色对函数的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { EXECUTE | ALL [ PRIVILEGES ] } ON FUNCTION funcname ( [ [ argmode ] [ argname ] argtype [, ...] ] ) [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...] [ CASCADE | RESTRICT ]
   ```

   撤销角色对过程语言的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { USAGE | ALL [ PRIVILEGES ] } ON LANGUAGE langname [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...] [ CASCADE | RESTRICT ]
   ```

   撤销角色对模式的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { { CREATE | USAGE } [,...] | ALL [ PRIVILEGES ] } ON SCHEMA schemaname [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...] [ CASCADE | RESTRICT ]
   ```

   撤销角色对表空间的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ] { CREATE | ALL [ PRIVILEGES ] } ON TABLESPACE tablespacename [, ...] FROM { rolename | GROUP groupname | PUBLIC } [, ...] [ CASCADE | RESTRICT ]
   ```

   删除rolename2的rolename1的成员关系：

   

   ```
   REVOKE [ ADMIN OPTION FOR ] rolename1 [, ...] FROM rolename2 [, ...] [ CASCADE | RESTRICT ]
   ```

   其中：

   - GRANT OPTION FOR：表示只是撤销对该权限的授权的权力，而不是撤销该权限本身。
   - SELECT、INSERT、UPDATE、DELETE、REFERENCES、TRIGGER、USAGE、CREATE、CONNECT、TEMPORARY、TEMP、EXECUTE、ALL [ PRIVILEGES ]：用户的操作权限，ALL [ PRIVILEGES ]表示所有的权限，PRIVILEGES关键字在 PostgreSQL里是可选的，但是严格的SQL 要求有这个关键字。
   - ON字句：用于指定撤销权限的对象。
   - tablename：表名。
   - FROM字句：用来指定被撤销权限的角色。
   - rolename、rolename1、rolename2：角色名。
   - groupname：角色组名。
   - PUBLIC：表示撤销隐含定义的、拥有所有角色的组，但并不意味着所有角色都失去了权限，那些直接得到的权限以及通过一个组得到的权限仍然有效。
   - sequencename：序列名。
   - CASCADE：撤销所有依赖性权限。
   - RESTRICT：不撤销所有依赖性权限。
   - databasename：数据库名。
   - funcname ( [ [ argmode ] [ argname ] argtype [, ...] ] )：函数名及其参数。
   - langname：过程语言名。
   - schemaname：模式名。
   - tablespacename：表空间名。
   - ADMIN OPTION FOR：表示传递的授权不会自动收回。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-4)

   对userexample授予数据库database1的CREATE权限。

   

   ```
   postgres=# GRANT CREATE ON DATABASE database1 TO userexample;
   ```

   对所有用户授予表table1的所有权限。

   

   ```
   postgres=# GRANT ALL PRIVILEGES ON TABLE table1 TO PUBLIC;
   ```

   ### 管理数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库)

   #### 创建数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建数据库)

   可以使用CREATE DATABASE语句或createdb来创建数据库。createrdb是对CREATE DATABASE命令的封装，需要在shell界面执行，而不是在数据库界面。

   

   ```
   CREATE DATABASE databasename;
   ```

   

   ```
   createdb databasename
   ```

   其中：databasename为数据库名。

   要使用这条命令，必须拥有CREATEDB权限。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-5)

   创建一个数据库database1。

   

   ```
   postgres=# CREATE DATABASE database1;
   ```

   #### 选择数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-选择数据库)

   可以使用\c语句来选择数据库。

   

   ```
   \c databasename;
   ```

   其中：databasename为数据库名称。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-6)

   选择databaseexample数据库。

   

   ```
   postgres=# \c databaseexample;
   ```

   #### 查看数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看数据库)

   可以使用\l语句来查看数据库。

   

   ```
   \l;
   ```

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-7)

   查看所有数据库。

   

   ```
   postgres=# \l;
   ```

   #### 删除数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除数据库)

   可以使用DROP DATABASE语句或dropdb来删除数据库。dropdb是对DROP DATABASE命令的封装，需要在shell界面执行，而不是在数据库界面。

   注意：

   删除数据库要谨慎操作，一旦删除，数据库中的所有表和数据都会删除。

   

   ```
   DROP DATABASE databasename;
   ```

   

   ```
   dropdb databasename
   ```

   其中：databasename为数据库名称。

   DROP DATABASE会删除数据库的系统目录项并且删除包含数据的文件目录。

   DROP DATABASE只能由超级管理员或数据库拥有者执行。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-8)

   删除databaseexample数据库。

   

   ```
   postgres=# DROP DATABASE databaseexample;
   ```

   #### 备份数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-备份数据库)

   可以使用pg_dump命令备份数据库，将数据库转储到一个脚本文件或其他归档文件中。

   

   ```
   pg_dump [option]... [databasename] > outfile
   ```

   其中：

   - databasename：数据库名称。如果没有声明这个参数，那么使用环境变量 PGDATABASE 。如果那个环境变量也没声明，那么使用发起连接的用户名。
   - outfile：数据库备份的文件。
   - option：pg_dump命令参数选项，多个参数之间可以使用空格分隔。常用的pg_dump命令参数选项如下：
     - -f，--file= *filename* ：指输出到指定的文件。如果忽略，则使用标准输出。
     - -d，--dbname= *databasename* ：指定转储的数据库。
     - -h，--host= *hostname* ：指定主机名。
     - -p，--port= *portnumber* ：指定端口。
     - -U，--username= *username* ：指定连接的用户名。
     - -W，--password：强制口令提示（自动）。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-9)

   备份主机为192.168.202.144，端口为3306，postgres用户下的database1数据库到db1.sql中。

   

   ```
   [postgres@localhost ~]#  pg_dump -h 192.168.202.144 -p 3306 -U postgres -W database1 > db1.sql
   ```

   #### 恢复数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-恢复数据库)

   可以使用psql命令恢复数据库。

   

   ```
   psql [option]... [databasename [username]] < infile
   ```

   其中：

   - databasename：数据库名称。如果没有声明这个参数，那么使用环境变量 PGDATABASE 。如果那个环境变量也没声明，那么使用发起连接的用户名。
   - username：用户名。
   - infile：pg_dump命令中的outfile参数。
   - option：psql命令参数选项，多个参数之间可以使用空格分隔。常用的psql命令参数选项如下：
     - -f，--file=filename：指输出到指定的文件。如果忽略，则使用标准输出。
     - -d，--dbname=databasename：指定转储的数据库。
     - -h，--host=hostname：指定主机名。
     - -p，--port=portnumber：指定端口。
     - -U，--username=username：指定连接的用户名。
     - -W，--password：强制口令提示（自动）。

   psql命令不会自动创建databasename数据库，所以在执行psql恢复数据库之前需要先创建databasename数据库。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-10)

   将db1.sql脚本文件导入到主机为192.168.202.144，端口为3306，postgres用户下newdb数据库中。

   

   ```
   [postgres@localhost ~]# createdb newdb
   [postgres@localhost ~]# psql -h 192.168.202.144 -p 3306 -U postgres -W -d newdb < db1.sql
   ```

   ## Mariadb服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-mariadb服务器)

   ### 软件介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-软件介绍-1)

   MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品，MariaDB还提供了许多更好的新特性。

   MariaDB的架构如[图2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#fig13492418164520)所示。

   **图 2** MariaDB逻辑架构
   ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/logical_architectureofMariaDB.ptwbA9j8.png)

   当Mariadb接受到Sql语句时，其详细的执行过程如下：

   1. 当客户端连接到mariadb的时候，会认证客户端的主机名、用户、密码，认证功能可以做成插件。
   2. 如果登录成功，客户端发送sql命令到服务端。由解析器解析sql语句。
   3. 服务端检查客户端是否有权限去获取它想要的资源。
   4. 如果查询已经存储在query cache当中，那么结果立即返回。
   5. 优化器将会找出最快的执行策略，或者是执行计划，也就是说优化器可以决定什么表将会被读，以及哪些索引会被访问，哪些临时表会被使用，一个好的策略能够减少大量的磁盘访问和排序操作等。
   6. 存储引擎读写数据和索引文件，cache用来加速这些操作，其他的诸如事物和外键特性，都是在存储引擎层处理的。

   存储引擎在物理层管控数据，它负责数据文件、数据、索引、cache等的管理，这使得管理和读取数据变得更高效，每一张表，都有一个.frm文件，这些文件包含着表的定义。

   每一个存储引擎管理、存储数据的方式都是不同的，所支持的特性和性能也不尽相同。例如：

   - MyISAM，适合读多写少的环境，且不支持事务，支持全文索引等。
   - noDB，支持事务，支持行锁和外键等。
   - MEMORY，将数据存储在内存当中。
   - CSV，将数据存储为CSV格式。

   ### 配置环境 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-配置环境-1)

   说明：

   以下环境配置仅为参考示例，具体配置视实际需求做配置

   #### 关闭防火墙并取消开机自启动 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-关闭防火墙并取消开机自启动-1)

   说明：

   测试环境下通常会关闭防火墙以避免部分网络因素影响，视实际需求做配置。

   1. 在root权限下停止防火墙。

      

      ```
      systemctl stop firewalld
      ```

   2. 在root权限下关闭防火墙。

      

      ```
      systemctl disable firewalld
      ```

      说明：

      执行disable命令关闭防火墙的同时，也取消了开机自启动。

   #### 修改SELINUX为disabled [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改selinux为disabled-1)

   1. 在root权限下修改配置文件。

      

      ```
      sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
      ```

   #### 创建组和用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建组和用户-1)

   说明：

   服务器环境下，为了系统安全，通常会为进程分配单独的用户，以实现权限隔离。本章节创建的组和用户都是操作系统层面的，不是数据库层面的。

   1. 在root权限下创建MySQL用户（组）。

      

      ```
      groupadd mysql
      ```

      

      ```
      useradd -g mysql mysql
      ```

   2. 在root权限下设置MySQL用户密码。

      

      ```
      passwd mysql
      ```

      重复输入密码（根据实际需求设置密码）。

   #### 搭建数据盘 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-搭建数据盘-1)

   说明：

   

   - 进行性能测试时，数据目录使用单独硬盘，需要对硬盘进行格式化并挂载，参考方法一或者方法二。
   - 非性能测试时，在root权限下执行`mkdir /data`创建数据目录即可。然后跳过本小节。

   ##### 方法一：在root权限下使用fdisk进行磁盘管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-方法一在root权限下使用fdisk进行磁盘管理)

   1. 创建分区（以/dev/sdb为例，根据实际情况创建）

      

      ```
      fdisk /dev/sdb
      ```

   2. 输入n，按回车确认。

   3. 输入p，按回车确认。

   4. 输入1，按回车确认。

   5. 采用默认配置，按回车确认。

   6. 采用默认配置，按回车确认。

   7. 输入w，按回车保存。

   8. 创建文件系统（以xfs为例，根据实际需求创建文件系统）

      

      ```
      mkfs.xfs /dev/sdb1
      ```

   9. 挂载分区到“/data”以供操作系统使用。

      

      ```
      mkdir /data
      ```

      

      ```
      mount /dev/sdb1 /data
      ```

   10. 执行命令“vi /etc/fstab", 编辑“/etc/fstab”使重启后自动挂载数据盘。如下图中，添加最后一行内容。

       其中，/dev/nvme0n1p1为示例，具体名称以实际情况为准。

       ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/creat_datadisk.D2WE5UBw.png)

   ##### 方法二：在root权限下使用LVM进行磁盘管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-方法二在root权限下使用lvm进行磁盘管理)

   说明：

   此步骤需要安装镜像中的lvm2相关包，步骤如下：

   - 配置本地yum源，详细信息请参考[搭建repo服务器](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html)。如果已经执行，则可跳过此步。
   - 在root权限下执行`yum install lvm2`命令安装lvm2。

   1. 创建物理卷（sdb为硬盘名称，具体名字以实际为准）。

      

      ```
      pvcreate /dev/sdb
      ```

   2. 创建物理卷组（其中datavg为创建的卷组名称，具体名字以实际规划为准）。

      

      ```
      vgcreate datavg  /dev/sdb
      ```

   3. 创建逻辑卷（其中600G为规划的逻辑卷大小，具体大小以实际情况为准；datalv为创建的逻辑卷的名字，具体名称以实际规划为准）。

      

      ```
      lvcreate -L 600G -n datalv datavg
      ```

   4. 创建文件系统。

      

      ```
      mkfs.xfs /dev/datavg/datalv
      ```

   5. 创建数据目录并挂载。

      

      ```
      mkdir /data
      mount /dev/datavg/datalv /data
      ```

   6. 执行命令**vi /etc/fstab**，编辑“/etc/fstab”使重启后自动挂载数据盘。如下图中，添加最后一行内容。

      其中，/dev/datavg/datalv为示例，具体名称以实际情况为准。

      ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/D1376B2A-D036-41C4-B852-E8368F363B5E.8fKPVZyb.png)

   #### 创建数据库目录并且授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建数据库目录并且授权)

   1. 在已创建的数据目录 **/data** 基础上，使用root权限继续创建进程所需的相关目录并授权MySQL用户（组）。

      

      ```
      mkdir -p /data/mariadb
      cd /data/mariadb
      mkdir data tmp run log
      chown -R mysql:mysql /data
      ```

   ### 安装、运行和卸载 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装运行和卸载-1)

   #### 安装 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装-1)

   1. 配置本地yum源，详细信息请参考[搭建repo服务器](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html)。

   2. 清除缓存。

      

      ```
      dnf clean all
      ```

   3. 创建缓存。

      

      ```
      dnf makecache
      ```

   4. 在root权限下安装mariadb服务器。

      

      ```
      dnf install mariadb-server
      ```

   5. 查看安装后的rpm包。

      

      ```
      rpm -qa | grep mariadb
      ```

   #### 运行 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-运行-1)

   1. 在root权限下开启mariadb服务器。

      

      ```
      systemctl start mariadb
      ```

   2. 在root权限下初始化数据库。

      

      ```
      /usr/bin/mysql_secure_installation
      ```

      命令执行过程中需要输入数据库的root设置的密码，若没有密码则直接按“Enter”。然后根据提示及实际情况进行设置。

   3. 登录数据库。

      

      ```
      mysql -u root -p
      ```

      命令执行后提示输入密码。密码为[2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#li197143190587)中设置的密码。

      说明：

      执行 **\q** 或者 **exit** 可退出数据库。

   #### 卸载 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-卸载-1)

   1. 在root权限下关闭数据库进程。

      

      ```
      ps -ef | grep mysql
      kill -9 进程ID
      ```

   2. 在root权限下执行**dnf remove mariadb-server**命令卸载mariadb。

      

      ```
      dnf remove mariadb-server
      ```

   ### 管理数据库用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库用户)

   #### 创建用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建用户)

   可以使用CREATE USER语句来创建一个或多个用户，并设置相应的口令。

   

   ```
   CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';
   ```

   其中：

   - username：用户名。
   - host：主机名，即用户连接数据库时所在的主机的名字。若是本地用户可用localhost，若在创建的过程中，未指定主机名，则主机名默认为“%”，表示一组主机。
   - password：用户的登录密码，密码可以为空，如果为空则该用户可以不需要密码登录服务器，但从安全的角度而言，不推荐这种做法。

   使用CREATE USER语句必须拥有数据库的INSERT权限或全局CREATE USER权限。

   使用CREATE USER语句创建一个用户帐号后，会在系统自身的数据库的user表中添加一条新记录。若创建的帐户已经存在，则语句执行时会出现错误。

   新创建的用户拥有的权限很少，只允许进行不需要权限的操作，如使用SHOW语句查询所有存储引擎和字符集的列表等。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-11)

   创建密码为123456，用户名为userexample1的本地用户。

   

   ```
   > CREATE USER 'userexample1'@'localhost' IDENTIFIED BY '123456';
   ```

   创建密码为123456，用户名为userexample2，主机名为192.168.1.100的用户。

   

   ```
   > CREATE USER 'userexample2'@'192.168.1.100' IDENTIFIED BY '123456';
   ```

   #### 查看用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看用户)

   可以使用SHOW GRANTS语句或SELECT语句查看一个或多个用户。

   查看特定用户：

   

   ```
   SHOW GRANTS [FOR 'username'@'hostname'];
   ```

   

   ```
   SELECT USER,HOST,PASSWORD FROM mysql.user WHERE USER='username';
   ```

   查看所有用户：

   

   ```
   SELECT USER,HOST,PASSWORD FROM mysql.user;
   ```

   其中：

   - username：用户名。
   - hostname：主机名。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-12)

   查看userexample1用户。

   

   ```
   > SHOW GRANTS FOR 'userexample1'@'localhost';
   ```

   查看mysql数据库中所有用户。

   

   ```
   > SELECT USER,HOST,PASSWORD FROM mysql.user;
   ```

   #### 修改用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户)

   ##### 修改用户名 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户名-1)

   可以使用RENAME USER语句修改一个或多个已经存在的用户名。

   

   ```
   RENAME USER 'oldusername'@'hostname' TO 'newusername'@'hostname';
   ```

   其中：

   - oldusername：旧的用户名。
   - newusername：新的用户名。
   - hostname：主机名。

   RENAME USER语句用于对原有的帐号进行重命名。若系统中旧帐号不存在或者新帐号已存在，则该语句执行时会出现错误。

   使用RENAME USER语句，必须拥有数据库的UPDATE权限或全局CREATE USER权限。

   ##### 修改用户示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户示例-1)

   将用户名userexample1修改为userexapme2，主机名为locahost。

   

   ```
   > RENAME USER 'userexample1'@'localhost' TO 'userexample2'@'localhost';
   ```

   ##### 修改用户密码 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户密码-1)

   可以使用SET PASSWORD语句修改一个用户的登录密码。

   

   ```
   SET PASSWORD FOR 'username'@'hostname' = PASSWORD('newpassword');
   ```

   其中：

   - FOR 'username'@'hostname'：FOR字句，可选项，指定欲修改密码的用户名及主机名。
   - PASSWORD('newpassword')：表示使用函数PASSWORD()设置新口令，即新口令必须传递到函数PASSWORD()中进行加密。

   注意：

   PASSWORD()函数为单向加密函数，一旦加密后不能解密出原明文。

   在SET PASSWORD语句中，若不加上FOR子句，表示修改当前用户的密码。

   FOR字句中必须以'username'@'hostname'的格式给定，username为帐户的用户名，hostname为帐户的主机名。

   欲修改密码的帐号必须在系统中存在，否则语句执行时会出现错误。

   ##### 修改用户密码示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户密码示例)

   将用户名为userexample的密码修改为0123456，主机名为locahost。

   

   ```
   > SET PASSWORD FOR 'userexample'@'localhost' = PASSWORD('0123456') ;
   ```

   #### 删除用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除用户)

   可以使用DROP USER语句来删除一个或多个用户帐号以及相关的权限。

   

   ```
   DROP USER 'username1'@'hostname1' [,'username2'@'hostname2']…;
   ```

   注意：

   用户的删除不会影响他们之前所创建的表、索引或其他数据库对象，因为数据库并不会记录创建了这些对象的帐号。

   DROP USER语句可用于删除一个或多个数据库帐号，并删除其原有权限。

   使用DROP USER语句必须拥有数据库的DELETE权限或全局CREATE USER权限。

   在DROP USER语句的使用中，若没有明确地给出帐号的主机名，则该主机名默认为“%”。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-13)

   删除用户名为userexample的本地用户。

   

   ```
   > DROP USER 'userexample'@'localhost';
   ```

   #### 用户授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-用户授权)

   可以使用GRANT语句来对新建用户的授权。

   

   ```
   GRANT privileges ON databasename.tablename TO 'username'@'hostname';
   ```

   其中：

   - ON字句：用于指定权限授予的对象和级别。
   - privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所有的权限则使用ALL。
   - databasename：数据库名。
   - tablename：表名。
   - TO字句：用来设定用户密码，以及指定被赋予权限的用户。
   - username：用户名。
   - hostname：主机名。

   如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*。

   如果在TO子句中给系统中存在的用户指定密码，则新密码会将原密码覆盖。

   如果权限被授予给一个不存在的用户，则会自动执行一条CREATE USER语句来创建这个用户，但同时必须为该用户指定密码。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-14)

   对本地用户userexample授予SELECT和INSERT权限。

   

   ```
   > GRANT SELECT,INSERT ON *.* TO 'userexample'@'localhost';
   ```

   #### 删除用户权限 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除用户权限-1)

   可以使用REVOKE语句来删除一个用户的权限，但此用户不会被删除。

   

   ```
   REVOKE privilege ON databasename.tablename FROM 'username'@'hostname';
   ```

   其中REVOKE语句的参数与GRANT语句的参数含义相同。

   要使用 REVOKE 语句，必须拥有数据库的全局CREATE USER权限或UPDATE权限。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-15)

   删除本地用户userexample的INSERT权限。

   

   ```
   > REVOKE INSERT ON *.* FROM 'userexample'@'localhost';
   ```

   ### 管理数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库-1)

   #### 创建数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建数据库-1)

   可以使用CREATE DATABASE语句来创建数据库。

   

   ```
   CREATE DATABASE databasename;
   ```

   其中：databasename为数据库名称，且数据库名称不区分大小写。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-16)

   创建数据库名为databaseexample的数据库。

   

   ```
   > CREATE DATABASE databaseexample;
   ```

   #### 查看数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看数据库-1)

   可以使用SHOW DATABASES语句来查看数据库。

   

   ```
   SHOW DATABASES;
   ```

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-17)

   查看所有数据库。

   

   ```
   > SHOW DATABASES;
   ```

   #### 选择数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-选择数据库-1)

   一般创建表，查询表等操作首先需要选择一个目标数据库。可以使用USE语句来选择数据库。

   

   ```
   USE databasename;
   ```

   其中：databasename为数据库名称。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-18)

   选择databaseexample数据库。

   

   ```
   > USE databaseexample;
   ```

   #### 删除数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除数据库-1)

   可以使用DROP DATABASE语句来删除数据库。

   注意：

   删除数据库要谨慎操作，一旦删除，数据库中的所有表和数据都会删除。

   

   ```
   DROP DATABASE databasename;
   ```

   其中：databasename为数据库名称。

   DROP DATABASE命令用于删除创建过(已存在)的数据库，且会删除数据库中的所有表，但数据库的用户权限不会自动删除。

   要使用DROP DATABASE，您需要数据库的DROP权限。

   DROP SCHEMA是DROP DATABASE的同义词。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-19)

   删除databaseexample数据库。

   

   ```
   > DROP DATABASE databaseexample;
   ```

   #### 备份数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-备份数据库-1)

   可以在root权限下使用mysqldump命令备份数据库。

   备份一个或多个表：

   

   ```
   mysqldump [options] databasename [tablename ...] > outfile
   ```

   备份一个或多个库：

   

   ```
   mysqldump [options] -databases databasename ... > outfile
   ```

   备份所有库：

   

   ```
   mysqldump [options] -all-databases > outputfile
   ```

   其中：

   - databasename：数据库名称。
   - tablename：数据表名称。
   - outfile：数据库备份的文件。
   - options：mysqldump命令参数选项，多个参数之间可以使用空格分隔。常用的mysqldump命令参数选项如下：
     - -u, --user= *username* ：指定用户名。
     - -p, --password[= *password*]：指定密码。
     - -P, --port= *portnumber* ：指定端口。
     - -h, --host= *hostname* ：指定主机名。
     - -r, --result-file= *filename* ：将导出结果保存到指定的文件中，等同于“>”。
     - -t：只备份数据。
     - -d：只备份表结构。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-20)

   备份主机为192.168.202.144，端口为3306，root用户下的所有数据库到alldb.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 --all-databases > alldb.sql
   ```

   备份主机为192.168.202.144，端口为3306，root用户下的db1数据库到db1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 --databases db1 > db1.sql
   ```

   备份主机为192.168.202.144，端口为3306，root用户下的db1数据库的tb1表到db1tb1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 db1 tb1 > db1tb1.sql
   ```

   只备份主机为192.168.202.144，端口为3306，root用户下的db1数据库的表结构到db1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 -d db1 > db1.sql
   ```

   只备份主机为192.168.202.144，端口为3306，root用户下的db1数据库的数据到db1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 -t db1 > db1.sql
   ```

   #### 恢复数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-恢复数据库-1)

   可以在root权限下使用mysql命令恢复数据库。

   恢复一个或多个表：

   

   ```
   mysql -h hostname -P portnumber -u username -ppassword databasename < infile
   ```

   其中：

   - hostname：主机名。
   - portnumber：端口号。
   - username：用户名。
   - password：密码。
   - databasename：数据库名。
   - infile：mysqldump命令中的outfile参数。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-21)

   恢复数据库。

   

   ```
   # mysql -h 192.168.202.144 -P 3306 -uroot -p123456 -t db1 < db1.sql
   ```

   ## MySQL服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-mysql服务器)

   ### 软件介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-软件介绍-2)

   MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle旗下产品。MySQL是业界最流行的RDBMS (Relational Database Management System，关系数据库管理系统)之一，尤其在WEB应用方面。

   关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就加快了速度并提高了灵活性。

   MySQL所使用的SQL语言是用于访问数据库的最常用标准化语言。MySQL软件采用了双授权模式，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择MySQL作为网站数据库。

   ### 配置环境 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-配置环境-2)

   说明：

   以下环境配置仅为参考示例，具体配置视实际需求做配置

   #### 关闭防火墙并取消开机自启动 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-关闭防火墙并取消开机自启动-2)

   说明：

   测试环境下通常会关闭防火墙以避免部分网络因素影响，视实际需求做配置。

   1. 在root权限下停止防火墙。

      

      ```
      systemctl stop firewalld
      ```

   2. 在root权限下关闭防火墙。

      

      ```
      systemctl disable firewalld
      ```

      说明：

      执行disable命令关闭防火墙的同时，也取消了开机自启动。

   #### 修改SELINUX为disabled [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改selinux为disabled-2)

   1. 在root权限下修改配置文件。

      

      ```
      sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
      ```

   #### 创建组和用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建组和用户-2)

   说明：

   服务器环境下，为了系统安全，通常会为进程分配单独的用户，以实现权限隔离。本章节创建的组和用户都是操作系统层面的，不是数据库层面的。

   1. 在root权限下创建MySQL用户（组）。

      

      ```
      groupadd mysql
      ```

      

      ```
      useradd -g mysql mysql
      ```

   2. 在root权限下设置MySQL用户密码。

      

      ```
      passwd mysql
      ```

      重复输入密码（根据实际需求设置密码）。

   #### 搭建数据盘 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-搭建数据盘-2)

   说明：

   

   - 进行性能测试时，数据目录使用单独硬盘，需要对硬盘进行格式化并挂载，参考方法一或者方法二。
   - 非性能测试时，在root权限下执行`mkdir /data`创建数据目录即可。然后跳过本小节：

   ##### 方法一：在root权限下使用fdisk进行磁盘管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-方法一在root权限下使用fdisk进行磁盘管理-1)

   1. 创建分区（以/dev/sdb为例，根据实际情况创建）

      

      ```
      fdisk /dev/sdb
      ```

   2. 输入n，按回车确认。

   3. 输入p，按回车确认。

   4. 输入1，按回车确认。

   5. 采用默认配置，按回车确认。

   6. 采用默认配置，按回车确认。

   7. 输入w，按回车保存。

   8. 创建文件系统（以xfs为例，根据实际需求创建文件系统）

      

      ```
      mkfs.xfs /dev/sdb1
      ```

   9. 挂载分区到“/data”以供操作系统使用。

      

      ```
      mkdir /data
      ```

      

      ```
      mount /dev/sdb1 /data
      ```

   10. 执行命令“vi /etc/fstab", 编辑“/etc/fstab”使重启后自动挂载数据盘。如下图中，添加最后一行内容。

       其中，/dev/nvme0n1p1为示例，具体名称以实际情况为准。

       ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/creat_datadisk.D2WE5UBw.png)

   ##### 方法二：在root权限下使用LVM进行磁盘管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-方法二在root权限下使用lvm进行磁盘管理-1)

   说明：

   此步骤需要安装镜像中的lvm2相关包，步骤如下：

   - 配置本地yum源，详细信息请参考[搭建repo服务器](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html)。如果已经执行，则可跳过此步。
   - 执行`yum install lvm2`安装lvm2。

   1. 创建物理卷（sdb为硬盘名称，具体名字以实际为准）。

      

      ```
      pvcreate /dev/sdb
      ```

   2. 创建物理卷组（其中datavg为创建的卷组名称，具体名字以实际规划为准）。

      

      ```
      vgcreate  datavg  /dev/sdb
      ```

   3. 创建逻辑卷（其中600G为规划的逻辑卷大小，具体大小以实际情况为准；datalv为创建的逻辑卷的名字，具体名称以实际规划为准）。

      

      ```
      lvcreate -L 600G -n datalv datavg
      ```

   4. 创建文件系统。

      

      ```
      mkfs.xfs /dev/datavg/datalv
      ```

   5. 创建数据目录并挂载。

      

      ```
      mkdir /data
      ```

      

      ```
      mount /dev/datavg/datalv /data
      ```

   6. 执行命令**vi /etc/fstab**，编辑“/etc/fstab”使重启后自动挂载数据盘。如下图中，添加最后一行内容。

      其中，/dev/datavg/datalv为示例，具体名称以实际情况为准。

      ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/D1376B2A-D036-41C4-B852-E8368F363B5E.8fKPVZyb.png)

   #### 创建数据库目录并且授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建数据库目录并且授权-1)

   1. 在已创建的数据目录 **/data** 基础上，使用root权限继续创建进程所需的相关目录并授权MySQL用户（组）。

      

      ```
      mkdir -p /data/mysql
      cd /data/mysql
      mkdir data tmp run log
      chown -R mysql:mysql /data
      ```

   ### 安装、运行和卸载 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装运行和卸载-2)

   #### 安装 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装-2)

   1. 配置本地yum源，详细信息请参考[搭建repo服务器](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/configuring_the_repo_server.html)章节。

   2. 清除缓存。

      

      ```
      dnf clean all
      ```

   3. 创建缓存。

      

      ```
      dnf makecache
      ```

   4. 在root权限下安装MySQL服务器。

      

      ```
      dnf install mysql-server
      ```

   5. 查看安装后的rpm包。

      

      ```
      rpm -qa | grep mysql-server
      ```

   #### 运行 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-运行-2)

   1. 修改配置文件。

      1. 在root权限下创建my.cnf文件，其中文件路径（包括软件安装路径basedir、数据路径datadir等）根据实际情况修改。

         

         ```
         vi /etc/my.cnf
         ```

         编辑my.cnf内容如下：

         

         ```
         [mysqld_safe]
         log-error=/data/mysql/log/mysql.log
         pid-file=/data/mysql/run/mysqld.pid
         [mysqldump]
         quick
         [mysql]
         no-auto-rehash
         [client]
         default-character-set=utf8
         [mysqld]
         basedir=/usr/local/mysql
         socket=/data/mysql/run/mysql.sock
         tmpdir=/data/mysql/tmp
         datadir=/data/mysql/data
         default_authentication_plugin=mysql_native_password
         port=3306
         user=mysql
         ```

      2. 确保my.cnf配置文件修改正确。

         

         ```
         cat /etc/my.cnf
         ```

         ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/zh-cn_image_0231563132.CQQX4fgd.png)

         注意：

         其中basedir为软件安装路径，请根据实际情况修改。

      3. 在root权限下修改/etc/my.cnf文件的组和用户为mysql:mysql

         

         ```
         chown mysql:mysql /etc/my.cnf
         ```

   2. 配置环境变量。

      1. 安装完成后，在root权限下将MySQL二进制文件路径到PATH。

         

         ```
         echo export  PATH=$PATH:/usr/local/mysql/bin  >> /etc/profile
         ```

         注意：

         其中PATH中的“/usr/local/mysql/bin“路径，为MySQL软件安装目录下的bin文件的绝对路径。请根据实际情况修改。

      2. 在root权限下使环境变量配置生效。

         

         ```
         source /etc/profile
         ```

   3. 在root权限下初始化数据库。

      说明：

      本步骤倒数第2行中有初始密码，请注意保存，登录数据库时需要使用。

      

      ```
      # mysqld --defaults-file=/etc/my.cnf --initialize
      2020-03-18T03:27:13.702385Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.17) initializing of server in progress as process 34014
      2020-03-18T03:27:24.112453Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: iNat=)#V2tZu
      2020-03-18T03:27:28.576003Z 0 [System] [MY-013170] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.17) initializing of server has completed
      ```

      查看打印信息，打印信息中包括“initializing of server has completed”表示初始化数据库完成，且打印信息中“A temporary password is generated for root@localhost: iNat=)V2tZu”的“iNat=)V2tZu”为初始密码。

   4. 启动数据库。

      注意：

      如果第一次启动数据库服务，以root用户启动数据库，则启动时会提示缺少mysql.log文件而导致失败。使用mysql用户启动之后，会在/data/mysql/log目录下生成mysql.log文件，再次使用root用户启动则不会报错。

      1. 在root权限下修改文件权限。

         

         ```
         chmod 777 /usr/local/mysql/support-files/mysql.server
         chown mysql:mysql /var/log/mysql/*
         ```

      2. 在root权限下启动MySQL。

         

         ```
         cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
         chkconfig mysql on
         ```

         以mysql用户启动数据库。

         

         ```
         su - mysql
         service mysql start
         ```

   5. 登录数据库。

      说明：

      

      - 提示输入密码时，请输入[3](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#li15634560582)产生的初始密码。
      - 如果采用官网RPM安装方式，则mysql文件在/usr/bin目录下。登录数据库的命令根据实际情况修改。

      

      ```
      # /usr/local/mysql/bin/mysql -uroot -p  -S /data/mysql/run/mysql.sock
      ```

      ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/zh-cn_image_0231563134.DsNI2wWs.png)

   6. 配置数据库帐号密码。

      1. 登录数据库以后，修改通过root用户登录数据库的密码。

         

         ```
         mysql> alter user 'root'@'localhost' identified by "123456";
         ```

      2. 创建全域root用户（允许root从其他服务器访问）。

         

         ```
         mysql> create user 'root'@'%' identified by '123456';
         ```

      3. 进行授权。

         

         ```
         mysql> grant all privileges on *.* to 'root'@'%';
         mysql> flush privileges;
         ```

         ![img](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/zh-cn_image_0231563135.CBF7PNfJ.png)

   7. 退出数据库。

      执行 **\q** 或者 **exit** 退出数据库。

      

      ```
      mysql> exit
      ```

      ![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAANQAAAAgCAYAAABq3VYXAAAEI0lEQVR4Ae1b7dmrIAxlKddxGodxFmdxB+4DAUwgUHptX6WeH31sBSEJ5+QDqTHGWHxgA2DgYxj42ECXEtMaY0f4ALi/gbfGOv6Ggo5MDSVv0TaCjHe34QDyXQfEadnsvu/hs9r5BCm+DVYv6zqfIua3ZRwAbKfsN4h+1xEqGWha7Lbfm1BJ1huT/hMyYozTfDg9wHmvMzqh5tXuHQ4BEeoGWDvhEDudzQ2UBKHOO6XvAwUy9th42ze7bVTHrItLvdz3zS6TI9ps132368xJN9llY/e8d451ELufJqcxqFba7LKsdt8WO6V2Y02FUL5uyfvy59j3lvef10O+PauDfBufw8vC9OD6Zc+S3GzsWA/m/YKcLRk7vR9Azdb8ljZzhHLkoQ0CqmMcyLZl8otXFOMivXFkieQj0s2rrIU8YBPAiIy9hPIGY4COMmmGrIFVmz8f59BXcyCkV2EHvrDCJtz5yO81GTV9cE/abhh7pM0AB4rgqQ+AldFDApQAmAP0UL4knGHzpH6VCJXaA3iJ9BQRZNSkd1B5f4qwkvDq/CES+yiayC8XFISS9ihtjXZvk5eEMhRViDQKQUKKFLe/Bbk0ovwloTLZoozRcXBQEFlldC3aK2TzJMWmBNJR5/hfE8rY5NU1MjRTH4WA2hga8eK4rn+oTQRZY3u46umUm79OkkSYMP/i3ovxeorNgQiFCJTwwnBR3OsiFEuJBKinxa6h1vIDe/BLAMsUsVJDqZsfoa6rADxXRCeUsXL+kMKKSONId2xCFP2D8ZqE8oTMUkvF6DUZc13we2Dy9hEqAFPx9pQqxZ0uDVQEWIoylV0+B77OSFQDWwusniRxB04QNMimEIx2A4MDiM+mq3QaTiYxhxjvAEdLxppeuH/YbxBb9Anc9NCKN64qr6V87zxf6TsCWEeQsbpuFbujf8Gf4oZSXMq06JQRQSjFvj1rgD6ncPd3DqG9UCmVqaQxbysJQoFQfwfuK2zdJtTbhLnIWC6dGuEzij0h53/z4r8fvIL9mPMihwWCdfOkuyPADDADA68wEF+apuunaqVXE6Md4PxFDMj/8fBjRohcSHOAgbcxIAkVTiekKKVtlxPp+OHUtBPoXnymZ7EYby/GL3rsp+kkCVVGqOKFrj/RcJwU8GRKJCqfB6jgWB6FgVQ7+WM1ytGh7OCqJJCLYNkzX3rP9KhFeZpX/yV9tQgl0zYedTICebLFc3zsKs7LwUPDGTwIA5JQ8ZDqkdJ5MMSoE6/JoziCZX1T24OMCJ2xYxkxkBPK10xFhKHNCZceir9vxFPWqYbS/h4BYiFCPQgDsoZyaZsecWgnr9UWUr6CjA8yZvRSuD45YvUBvtjtA2ieDBroXsd/D6Eo5ePvnpDG9NgNfR6Ik/aip5e2vE6qsxOeC7Z5NAb+ASmk1aK9qEsiAAAAAElFTkSuQmCC)

   #### 卸载 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-卸载-2)

   1. 在root权限下关闭数据库进程。

      

      ```
      ps -ef | grep mysql
      kill -9 进程ID
      ```

   2. 在root权限下执行**dnf remove mysql**命令卸载MySQL。

      

      ```
      dnf remove mysql
      ```

   ### 管理数据库用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库用户-1)

   #### 创建用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建用户-1)

   可以使用CREATE USER语句来创建一个或多个用户，并设置相应的口令。

   

   ```
   CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';
   ```

   其中：

   - username：用户名。
   - host：主机名，即用户连接数据库时所在的主机的名字。若是本地用户可用localhost，若在创建的过程中，未指定主机名，则主机名默认为“%”，表示一组主机。
   - password：用户的登录密码，密码可以为空，如果为空则该用户可以不需要密码登录服务器，但从安全的角度而言，不推荐这种做法。

   使用CREATE USER语句必须拥有数据库的INSERT权限或全局CREATE USER权限。

   使用CREATE USER语句创建一个用户帐号后，会在系统自身的数据库的user表中添加一条新记录。若创建的帐户已经存在，则语句执行时会出现错误。

   新创建的用户拥有的权限很少，只允许进行不需要权限的操作，如使用SHOW语句查询所有存储引擎和字符集的列表等。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-22)

   创建密码为123456，用户名为userexample1的本地用户。

   

   ```
   > CREATE USER 'userexample1'@'localhost' IDENTIFIED BY '123456';
   ```

   创建密码为123456，用户名为userexample2，主机名为192.168.1.100的用户。

   

   ```
   > CREATE USER 'userexample2'@'192.168.1.100' IDENTIFIED BY '123456';
   ```

   #### 查看用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看用户-1)

   可以使用SHOW GRANTS语句或SELECT语句查看一个或多个用户。

   查看特定用户：

   

   ```
   SHOW GRANTS [FOR 'username'@'hostname'];
   ```

   

   ```
   SELECT USER,HOST,PASSWORD FROM mysql.user WHERE USER='username';
   ```

   查看所有用户：

   

   ```
   SELECT USER,HOST FROM mysql.user;
   ```

   其中：

   - username：用户名。
   - hostname：主机名。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-23)

   查看userexample1用户。

   

   ```
   > SHOW GRANTS FOR 'userexample1'@'localhost';
   ```

   查看mysql数据库中所有用户。

   

   ```
   > SELECT USER,HOST FROM mysql.user;
   ```

   #### 修改用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户-1)

   ##### 修改用户名 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户名-2)

   可以使用RENAME USER语句修改一个或多个已经存在的用户名。

   

   ```
   RENAME USER 'oldusername'@'hostname' TO 'newusername'@'hostname';
   ```

   其中：

   - oldusername：旧的用户名。
   - newusername：新的用户名。
   - hostname：主机名。

   RENAME USER语句用于对原有的帐号进行重命名。若系统中旧帐号不存在或者新帐号已存在，则该语句执行时会出现错误。

   使用RENAME USER语句，必须拥有数据库的UPDATE权限或全局CREATE USER权限。

   ##### 修改用户示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户示例-2)

   将用户名userexample1修改为userexapme2，主机名为locahost。

   

   ```
   > RENAME USER 'userexample1'@'localhost' TO 'userexample2'@'localhost';
   ```

   ##### 修改用户密码 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户密码-2)

   可以使用SET PASSWORD语句修改一个用户的登录密码。

   

   ```
   SET PASSWORD FOR 'username'@'hostname' = 'newpassword';
   ```

   其中：

   - FOR 'username'@'hostname'：FOR字句，可选项，指定欲修改密码的用户名及主机名。
   - 'newpassword'：新密码。

   在SET PASSWORD语句中，若不加上FOR子句，表示修改当前用户的密码。

   FOR字句中必须以'username'@'hostname'的格式给定，username为帐户的用户名，hostname为帐户的主机名。

   欲修改密码的帐号必须在系统中存在，否则语句执行时会出现错误。

   ##### 修改用户密码示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户密码示例-1)

   将用户名为userexample的密码修改为0123456，主机名为locahost。

   

   ```
   > SET PASSWORD FOR 'userexample'@'localhost' = '0123456';
   ```

   #### 删除用户 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除用户-1)

   可以使用DROP USER语句来删除一个或多个用户帐号以及相关的权限。

   

   ```
   DROP USER 'username1'@'hostname1' [,'username2'@'hostname2']…;
   ```

   注意：

   用户的删除不会影响他们之前所创建的表、索引或其他数据库对象，因为数据库并不会记录创建了这些对象的帐号。

   DROP USER语句可用于删除一个或多个数据库帐号，并删除其原有权限。

   使用DROP USER语句必须拥有数据库的DELETE权限或全局CREATE USER权限。

   在DROP USER语句的使用中，若没有明确地给出帐号的主机名，则该主机名默认为“%”。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-24)

   删除用户名为userexample的本地用户。

   

   ```
   > DROP USER 'userexample'@'localhost';
   ```

   #### 用户授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-用户授权-1)

   可以使用GRANT语句来对新建用户的授权。

   

   ```
   GRANT privileges ON databasename.tablename TO 'username'@'hostname';
   ```

   其中：

   - ON字句：用于指定权限授予的对象和级别。
   - privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所有的权限则使用ALL。
   - databasename：数据库名。
   - tablename：表名。
   - TO字句：用来设定用户密码，以及指定被赋予权限的用户。
   - username：用户名。
   - hostname：主机名。

   如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*。

   如果在TO子句中给系统中存在的用户指定密码，则新密码会将原密码覆盖。

   如果权限被授予给一个不存在的用户，则会自动执行一条CREATE USER语句来创建这个用户，但同时必须为该用户指定密码。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-25)

   对本地用户userexample授予SELECT和INSERT权限。

   

   ```
   > GRANT SELECT,INSERT ON *.* TO 'userexample'@'localhost';
   ```

   #### 删除用户权限 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除用户权限-2)

   可以使用REVOKE语句来删除一个用户的权限，但此用户不会被删除。

   

   ```
   REVOKE privilege ON databasename.tablename FROM 'username'@'hostname';
   ```

   其中REVOKE语句的参数与GRANT语句的参数含义相同。

   要使用 REVOKE 语句，必须拥有数据库的全局CREATE USER权限或UPDATE权限。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-26)

   删除本地用户userexample的INSERT权限。

   

   ```
   > REVOKE INSERT ON *.* FROM 'userexample'@'localhost';
   ```

   ### 管理数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库-2)

   #### 创建数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建数据库-2)

   可以使用CREATE DATABASE语句来创建数据库。

   

   ```
   CREATE DATABASE databasename;
   ```

   其中：databasename为数据库名称，且数据库名称不区分大小写。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-27)

   创建数据库名为databaseexample的数据库。

   

   ```
   > CREATE DATABASE databaseexample;
   ```

   #### 查看数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看数据库-2)

   可以使用SHOW DATABASES语句来查看数据库。

   

   ```
   SHOW DATABASES;
   ```

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-28)

   查看所有数据库。

   

   ```
   > SHOW DATABASES;
   ```

   #### 选择数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-选择数据库-2)

   一般创建表，查询表等操作首先需要选择一个目标数据库。可以使用USE语句来选择数据库。

   

   ```
   USE databasename;
   ```

   其中：databasename为数据库名称。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-29)

   选择databaseexample数据库。

   

   ```
   > USE databaseexample;
   ```

   #### 删除数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除数据库-2)

   可以使用DROP DATABASE语句来删除数据库。

   注意：

   删除数据库要谨慎操作，一旦删除，数据库中的所有表和数据都会删除。

   

   ```
   DROP DATABASE databasename;
   ```

   其中：databasename为数据库名称。

   DROP DATABASE命令用于删除创建过(已存在)的数据库，且会删除数据库中的所有表，但数据库的用户权限不会自动删除。

   要使用DROP DATABASE，您需要数据库的DROP权限。

   DROP SCHEMA是DROP DATABASE的同义词。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-30)

   删除databaseexample数据库。

   

   ```
   > DROP DATABASE databaseexample;
   ```

   #### 备份数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-备份数据库-2)

   可以在root权限下使用mysqldump命令备份数据库。

   备份一个或多个表：

   

   ```
   mysqldump [options] databasename [tablename ...] > outfile
   ```

   备份一个或多个库：

   

   ```
   mysqldump [options] -databases databasename ... > outfile
   ```

   备份所有库：

   

   ```
   mysqldump [options] -all-databases > outputfile
   ```

   其中：

   - databasename：数据库名称。
   - tablename：数据表名称。
   - outfile：数据库备份的文件。
   - options：mysqldump命令参数选项，多个参数之间可以使用空格分隔。常用的mysqldump命令参数选项如下：
     - -u, --user= *username* ：指定用户名。
     - -p, --password[= *password*]：指定密码。
     - -P, --port= *portnumber* ：指定端口。
     - -h, --host= *hostname* ：指定主机名。
     - -r, --result-file= *filename* ：将导出结果保存到指定的文件中，等同于“>”。
     - -t：只备份数据。
     - -d：只备份表结构。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-31)

   备份主机为192.168.202.144，端口为3306，root用户下的所有数据库到alldb.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 --all-databases > alldb.sql
   ```

   备份主机为192.168.202.144，端口为3306，root用户下的db1数据库到db1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 --databases db1 > db1.sql
   ```

   备份主机为192.168.202.144，端口为3306，root用户下的db1数据库的tb1表到db1tb1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 db1 tb1 > db1tb1.sql
   ```

   只备份主机为192.168.202.144，端口为3306，root用户下的db1数据库的表结构到db1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 -d db1 > db1.sql
   ```

   只备份主机为192.168.202.144，端口为3306，root用户下的db1数据库的数据到db1.sql中。

   

   ```
   mysqldump -h 192.168.202.144 -P 3306 -uroot -p123456 -t db1 > db1.sql
   ```

   #### 恢复数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-恢复数据库-2)

   可以在root权限下使用mysql命令恢复数据库。

   恢复一个或多个表：

   

   ```
   mysql -h hostname -P portnumber -u username -ppassword databasename < infile
   ```

   其中：

   - hostname：主机名。
   - portnumber：端口号。
   - username：用户名。
   - password：密码。
   - databasename：数据库名。
   - infile：mysqldump命令中的outfile参数。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-32)

   恢复数据库。

   

   ```
   # mysql -h 192.168.202.144 -P 3306 -uroot -p123456 -t db1 < db1.sql
   ```

   ## openGauss服务器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-opengauss服务器)

   ### 软件介绍 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-软件介绍-3)

   openGauss是一款开源关系型数据库管理系统，采用木兰宽松许可证v2发行。openGauss内核深度融合华为在数据库领域多年的经验，结合企业级场景需求，持续构建竞争力特性。

   ### 安装 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-安装-3)

   安装指南请参见[《openGauss rpm安装》](https://docs.opengauss.org/zh/docs/latest/docs/InstallationGuide/RPM安装.html)。

   ### 管理数据库角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库角色-1)

   #### 创建角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建角色-1)

   可以使用CREATE ROLE语句来创建角色，在数据库界面执行。

   

   ```
   CREATE ROLE role_name [ [ WITH ] option [ ... ] ] [ ENCRYPTED | UNENCRYPTED ] { PASSWORD | IDENTIFIED BY } { 'password' [EXPIRED] | DISABLE };
   ```

   其中：

   - role_name：角色名。
   - password：登录密码。
   - option为参数选项，常用的有：
     - SYSADMIN | NOSYSADMIN：决定一个新角色是否为"系统管理员"，具有SYSADMIN属性的角色拥有系统最高权限。缺省为NOSYSADMIN。
     - CREATEDB | NOCREATEDB：定义一个角色是否能创建数据库，若未指定，则默认为NOCREATEDB，即不能创建数据库。
     - CREATEROLE | NOCREATEROLE：决定一个角色是否可以创建新角色，若未指定，则默认为NOCREATEROLE，即不能创建新角色。
     - INHERIT | NOINHERIT：这些子句决定一个角色是否“继承”它所在组的角色的权限。不推荐使用。
     - LOGIN | NOLOGIN：具有LOGIN属性的角色才可以登录数据库。一个拥有LOGIN属性的角色可以认为是一个用户。
     - ENCRYPTED | UNENCRYPTED：控制密码存储在系统表里的口令是否加密。按照产品安全要求，密码必须加密存储，所以，UNENCRYPTED在openGauss中禁止使用。因为系统无法对指定的加密口令字符串进行解密，所以如果目前的口令字符串已经是用SHA256加密的格式，则会继续照此存放，而不管是否声明了ENCRYPTED或UNENCRYPTED。这样就允许在dump/restore的时候重新加载加密的口令。
     - VALID UNTIL：设置角色失效的时间戳。如果省略了该子句，角色无有效结束时间限制。
     - IN ROLE：新角色立即拥有IN ROLE子句中列出的一个或多个现有角色拥有的权限。不推荐使用。
     - ROLE：ROLE子句列出一个或多个现有的角色，它们将自动添加为这个新角色的成员，拥有新角色所有的权限。

   要使用这条命令，必须拥有 CREATE ROLE 权限或者是系统管理员。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-33)

   创建一个角色，名为manager，密码为xxxxxxxxx。

   

   ```
   openGauss=# CREATE ROLE manager IDENTIFIED BY 'xxxxxxxxx';
   ```

   创建一个角色，从2015年1月1日开始生效，到2026年1月1日失效。

   

   ```
   openGauss=# CREATE ROLE miriam WITH LOGIN PASSWORD 'xxxxxxxxx' VALID BEGIN '2015-01-01' VALID UNTIL '2026-01-01';
   ```

   #### 查看角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看角色-1)

   可以使用SELECT语句或gsql的元命令du或者du+查看角色。

   

   ```
   SELECT * FROM pg_roles;
   ```

   

   ```
   \du
   ```

   

   ```
   \du+
   ```

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-34)

   查看所有角色名包括内置角色。

   

   ```
   openGauss=# SELECT rolname from pg_roles;
   ```

   查看现有角色不包含内置角色。

   

   ```
   openGauss=# \du
   ```

   #### 修改角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改角色-1)

   ##### 修改用户名 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户名-3)

   可以使用ALTER ROLE语句修改一个已经存在的角色名。

   

   ```
   ALTER ROLE oldrolename RENAME TO newrolename;
   ```

   其中：

   - oldrolename：旧的角色名。
   - newrolename：新的角色名。

   ##### 修改用户示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户示例-3)

   将角色名manager修改为newmanager。

   

   ```
   openGauss=# ALTER ROLE manager RENAME TO newmanager;
   ```

   ##### 修改用户密码 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改用户密码-3)

   可以使用ALTER ROLE语句修改一个角色的登录密码。

   

   ```
   ALTER ROLE rolename with PASSWORD 'password'
   ```

   其中：

   - rolename：角色名。
   - password：密码。

   ##### 修改角色密码示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-修改角色密码示例-1)

   将manager的密码修改为xxxxxxxxx。

   

   ```
   openGauss=# ALTER ROLE manager with PASSWORD 'xxxxxxxxx';
   ```

   #### 删除角色 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除角色-1)

   可以使用DROP ROLE语句来删除角色。在数据库界面执行。

   

   ```
   DROP ROLE rolename;
   ```

   其中：rolename为角色名。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-35)

   删除manager角色。

   

   ```
   openGauss=# DROP ROLE manager;
   ```

   #### 角色授权 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-角色授权-1)

   可以使用GRANT语句来对角色授权。

   对角色授予表或视图的操作权限：

   

   ```
   GRANT { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER | ALTER | DROP | COMMENT | INDEX | VACUUM } [, ...] 
         | ALL [ PRIVILEGES ] }
       ON { [ TABLE ] table_name [, ...]
          | ALL TABLES IN SCHEMA schema_name [, ...] }
       TO { [ GROUP ] role_name | PUBLIC } [, ...] 
       [ WITH GRANT OPTION ];
   ```

   对角色授予序列的操作权限：

   

   ```
   GRANT { { SELECT | UPDATE | USAGE | ALTER | DROP | COMMENT } [, ...] 
         | ALL [ PRIVILEGES ] }
       ON { [ [ LARGE ] SEQUENCE ] sequence_name [, ...]
          | ALL SEQUENCES IN SCHEMA schema_name [, ...] }
       TO { [ GROUP ] role_name | PUBLIC } [, ...] 
       [ WITH GRANT OPTION ];
   ```

   对角色授予数据库的操作权限：

   

   ```
   GRANT { { CREATE | CONNECT | TEMPORARY | TEMP | ALTER | DROP | COMMENT } [, ...]
         | ALL [ PRIVILEGES ] }
       ON DATABASE database_name [, ...]
       TO { [ GROUP ] role_name | PUBLIC } [, ...]
       [ WITH GRANT OPTION ];
   ```

   对角色授予函数的操作权限：

   

   ```
   GRANT { { EXECUTE | ALTER | DROP | COMMENT } [, ...] | ALL [ PRIVILEGES ] }
       ON { FUNCTION {function_name ( [ {[ argmode ] [ arg_name ] arg_type} [, ...] ] )} [, ...]
          | ALL FUNCTIONS IN SCHEMA schema_name [, ...] }
       TO { [ GROUP ] role_name | PUBLIC } [, ...]
       [ WITH GRANT OPTION ];
   ```

   对角色授予过程语言的操作权限：

   

   ```
   GRANT { USAGE | ALL [ PRIVILEGES ] }
       ON LANGUAGE lang_name [, ...]
       TO { [ GROUP ] role_name | PUBLIC } [, ...]
       [ WITH GRANT OPTION ];
   ```

   将模式的访问权限赋予指定的角色：

   

   ```
   GRANT { { CREATE | USAGE | ALTER | DROP | COMMENT } [, ...] | ALL [ PRIVILEGES ] }
       ON SCHEMA schema_name [, ...]
       TO { [ GROUP ] role_name | PUBLIC } [, ...]
       [ WITH GRANT OPTION ];
   ```

   对角色授予表空间的操作权限：

   

   ```
   GRANT { { CREATE | ALTER | DROP | COMMENT } [, ...] | ALL [ PRIVILEGES ] }
       ON TABLESPACE tablespace_name [, ...]
       TO { [ GROUP ] role_name | PUBLIC } [, ...]
       [ WITH GRANT OPTION ];
   ```

   将角色rolename1的成员关系赋予角色rolename2：

   

   ```
   GRANT rolename1 [, ...] TO rolename2 [, ...] [ WITH ADMIN OPTION ]
   ```

   其中：

   - SELECT、INSERT、UPDATE、DELETE、REFERENCES、TRIGGER、USAGE、CREATE、CONNECT、TEMPORARY、TEMP、EXECUTE、ALL PRIVILEGES：用户的操作权限，ALL PRIVILEGES表示所有的权限。
   - ON字句：用于指定权限授予的对象。
   - tablename：表名。
   - TO字句：用来指定被赋予权限的角色。
   - rolename1、rolename2：角色名。
   - PUBLIC：表示该权限要赋予所有角色，包括那些以后可能创建的用户。
   - WITH GRANT OPTION：被授权的用户也可以将此权限赋予他人，否则就不能授权给他人。这个选项不能赋予PUBLIC。
   - sequence_name：序列名。
   - database_name：数据库名。
   - function_name ( [ {[ argmode ] [ arg_name ] arg_type} [, ...] ] )} [, ...]：函数名及其参数。
   - lang_name：过程语言名。
   - schema_name：模式名。
   - tablespace_name：表空间名。
   - WITH ADMIN OPTION：表示成员随后就可以将角色的成员关系赋予其他角色，以及撤销其他角色的成员关系。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-36)

   对manager授予数据库database1的CREATE权限。·

   

   ```
   openGauss=# GRANT CREATE ON DATABASE database1 TO manager;
   ```

   对所有用户授予表table1的所有权限。

   

   ```
   openGauss=# GRANT ALL PRIVILEGES ON TABLE table1 TO PUBLIC;
   ```

   #### 删除用户权限 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除用户权限-3)

   可以使用REVOKE语句来撤销一个或多个角色的权限。

   撤销角色对表的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
       { { SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | ALTER | DROP | COMMENT | INDEX | VACUUM }[, ...] 
       | ALL [ PRIVILEGES ] }
       ON { [ TABLE ] table_name [, ...]
          | ALL TABLES IN SCHEMA schema_name [, ...] }
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   撤销角色对序列的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
       { { SELECT | UPDATE | ALTER | DROP | COMMENT }[, ...] 
       | ALL [ PRIVILEGES ] }
        ON { [ [ LARGE ] SEQUENCE ] sequence_name [, ...]
          | ALL SEQUENCES IN SCHEMA schema_name [, ...] }
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   撤销角色对数据库的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
       { { CREATE | CONNECT | TEMPORARY | TEMP | ALTER | DROP | COMMENT } [, ...] 
       | ALL [ PRIVILEGES ] }
       ON DATABASE database_name [, ...]
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   撤销角色对函数的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
       { { EXECUTE | ALTER | DROP | COMMENT } [, ...] | ALL [ PRIVILEGES ] }
       ON { FUNCTION {function_name ( [ {[ argmode ] [ arg_name ] arg_type} [, ...] ] )} [, ...]
          | ALL FUNCTIONS IN SCHEMA schema_name [, ...] }
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   撤销角色对过程语言的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
      { USAGE | ALL [ PRIVILEGES ] }
       ON LANGUAGE lang_name [, ...]
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   撤销角色对模式的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
       { { CREATE | USAGE | ALTER | DROP | COMMENT } [, ...] | ALL [ PRIVILEGES ] }
       ON SCHEMA schema_name [, ...]
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   撤销角色对表空间的操作权限：

   

   ```
   REVOKE [ GRANT OPTION FOR ]
       { { CREATE | ALTER | DROP | COMMENT } [, ...] | ALL [ PRIVILEGES ] }
       ON TABLESPACE tablespace_name [, ...]
       FROM { [ GROUP ] role_name | PUBLIC } [, ...]
       [ CASCADE | RESTRICT ];
   ```

   删除rolename2的rolename1之间的成员关系：

   

   ```
   REVOKE [ ADMIN OPTION FOR ] rolename1 [, ...] FROM rolename2 [, ...] [ CASCADE | RESTRICT ]
   ```

   其中：

   - GRANT OPTION FOR：表示只是撤销对该权限的授权的权力，而不是撤销该权限本身。
   - SELECT、INSERT、UPDATE、DELETE、REFERENCES、TRIGGER、USAGE、CREATE、CONNECT、TEMPORARY、TEMP、EXECUTE、ALL PRIVILEGES：用户的操作权限，ALL PRIVILEGES表示所有的权限。
   - ON字句：用于指定撤销权限的对象。
   - table_name：表名。
   - FROM字句：用来指定被撤销权限的角色。
   - rolename1、rolename2：角色名。
   - PUBLIC：表示撤销隐含定义的、拥有所有角色的组，但并不意味着所有角色都失去了权限，那些直接得到的权限以及通过一个组得到的权限仍然有效。
   - sequence_name：序列名。
   - CASCADE：撤销所有依赖性权限。
   - RESTRICT：不撤销所有依赖性权限。
   - database_name：数据库名。
   - function_name ( [ {[ argmode ] [ arg_name ] arg_type} [, ...] ] )} [, ...]：函数名及其参数。
   - lang_name：过程语言名。
   - schema_name：模式名。
   - tablespace_name：表空间名。
   - ADMIN OPTION FOR：表示传递的授权不会自动收回。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-37)

   对manager授予数据库database1的CREATE权限。

   

   ```
   openGauss=# GRANT CREATE ON DATABASE database1 TO manager;
   ```

   对所有用户撤销表table1的所有权限。

   

   ```
   openGauss=# REVOKE ALL PRIVILEGES ON TABLE table1 FROM PUBLIC;
   ```

   ### 管理数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-管理数据库-3)

   #### 创建数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-创建数据库-3)

   可以使用CREATE DATABASE语句来创建数据库。在数据库界面使用。

   

   ```
   CREATE DATABASE databasename;
   ```

   其中：databasename为数据库名。

   要使用这条命令，必须拥有CREATEDB权限。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-38)

   创建一个数据库database1。

   

   ```
   openGauss=# CREATE DATABASE database1;
   ```

   #### 选择数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-选择数据库-3)

   可以使用\c语句来选择数据库。

   

   ```
   \c databasename;
   ```

   其中：databasename为数据库名称。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-39)

   选择database_example数据库。

   

   ```
   openGauss=# \c database_example;
   ```

   #### 查看数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-查看数据库-3)

   可以使用\l语句来查看数据库。

   

   ```
   \l;
   ```

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-40)

   查看所有数据库。

   

   ```
   openGauss=# \l;
   ```

   #### 删除数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-删除数据库-3)

   可以使用DROP DATABASE语句来删除数据库。

   注意：

   删除数据库要谨慎操作，一旦删除，数据库中的所有表和数据都会删除。

   

   ```
   DROP DATABASE databasename;
   ```

   其中：databasename为数据库名称。

   DROP DATABASE会删除数据库的系统目录项并且删除包含数据的文件目录。

   DROP DATABASE只能由超级管理员或数据库管理者执行。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-41)

   删除databaseexample数据库。

   

   ```
   openGauss=# DROP DATABASE databaseexample;
   ```

   #### 备份数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-备份数据库-3)

   gs_dump支持将数据库信息导出至纯文本格式的SQL脚本文件或其他归档文件中。

   

   ```
   gs_dump [OPTION]... [DBNAME]
   gs_dump -p port_number postgres -f dump1.sql
   ```

   其中：

   - DBNAME前面不需要加短或长选项。DBNAME指定要连接的数据库。 例如： 不需要-d，直接指定DBNAME。
   - dump1.sql：数据库备份的文件。
   - option：gs_dump命令参数选项，多个参数之间可以使用空格分隔。常用的gs_dump命令参数选项如下：
     - -F, --format=c|d|t|p：选择输出格式。格式如下：
       - p|plain：输出一个文本SQL脚本文件（默认）。
       - c|custom：输出一个自定义格式的归档，并且以目录形式输出，作为gs_restore输入信息。该格式是最灵活的输出格式，因为能手动选择，而且能在恢复过程中将归档项重新排序。该格式默认状态下会被压缩。
       - d|directory：该格式会创建一个目录，该目录包含两类文件，一类是目录文件，另一类是每个表和blob对象对应的数据文件。
       - t|tar：输出一个tar格式的归档形式，作为gs_restore输入信息。tar格式与目录格式兼容；tar格式归档形式在提取过程中会生成一个有效的目录格式归档形式。但是，tar格式不支持压缩且对于单独表有8GB的大小限制。此外，表数据项的相应排序在恢复过程中不能更改.
     - -h, --host=HOSTNAME：指定主机名。
     - -p, --port=PORT：指定端口。
     - -U, --username=NAME：指定连接的用户名。
     - -W, --password=PASSWORD：指定用户连接的密码。如果主机的认证策略是trust，则不会对系统管理员进行密码验证，即无需输入-W选项；如果没有-W选项，并且不是系统管理员，“Dump Restore工具”会提示用户输入密码。
     - -w, --no-password：不出现输入密码提示。如果主机要求密码认证并且密码没有通过其它形式给出，则连接尝试将会失败。 该选项在批量工作和不存在用户输入密码的脚本中很有帮助。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-42)

   执行gs_dump，导出postgres数据库全量信息，导出的MPPDB_backup.sql文件格式为纯文本格式。

   

   ```
   [openGauss@localhost ~]#  gs_dump -U omm -W password -f backup/MPPDB_backup.sql -p port postgres -F p
   ```

   执行gs_dump，导出postgres数据库全量信息，导出的MPPDB_backup.dmp文件格式为自定义归档格式。

   

   ```
   gs_dump -U omm -W password -f backup/MPPDB_backup.dmp -p port postgres -F c
   ```

   #### 恢复数据库 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-恢复数据库-3)

   gs_restore是openGauss提供的针对gs_dump导出数据的导入工具。通过此工具可将由gs_dump生成的导出文件进行导入。

   

   ```
   gs_restore [OPTION]... FILE
   ```

   其中：

   - FILE没有短选项或长选项。用来指定归档文件所处的位置。
   - 作为前提条件，需输入dbname或-l选项。不允许用户同时输入dbname和-l选项。
   - gs_restore默认是以追加的方式进行数据导入。为避免多次导入造成数据异常，在进行导入时，建议使用“-c” 参数，在重新创建数据库对象前，清理（删除）已存在于将要还原的数据库中的数据库对象。
   - option：通用参数如下：
     - -f，--file=FILENAME：指定生成脚本的输出文件，或使用-l时列表的输出文件。
     - -d, --dbname=NAME：连接数据库dbname并直接导入到该数据库中。
     - -h, --host=HOSTNAME：指定主机名。
     - -p, --port=PORT：指定端口。
     - -U, --username=NAME：指定连接的用户名。
     - -W, --password=PASSWORD：指定用户连接的密码。如果主机的认证策略是trust，则不会对系统管理员进行密码验证，即无需输入-W参数；如果没有-W参数，并且不是系统管理员，“gs_restore”会提示用户输入密码。

   ##### 示例 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/administration/administrator/setting_up_the_database_server.html#user-content-示例-43)

   执行gs_restore，将导出的MPPDB_backup.dmp文件（自定义归档格式）导入到postgres数据库。

   

   ```
   [openGauss@localhost ~]# gs_restore -W password backup/MPPDB_backup.dmp -p port -d postgres
   ```

   



---

> 作者: <no value>  
> URL: http://localhost:1313/posts/7f40eb3/  

