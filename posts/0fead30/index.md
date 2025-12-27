# Openeuler故障应急处理


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

## 触发kdump重启 

```
# 向sysrq文件中写入1, 开启SysRq功能，开启该功能后，内核会响应任何操作。
echo 1 > /proc/sys/kernel/sysrq

# 让系统崩溃
echo c > /proc/sysrq-trigger
```

## 强制重启 

强制重启有以下两种方式：

1.手动重启OS。

```
reboot -f
```

2.通过iBMC强制上下电。

![zh-cn_image_0000001372249333](https://docs.openeuler.openatom.cn/assets/24.03_LTS_SP1/zh-cn_image_0000001372249333.B4Koduq2.png)

## 重启网络 

openEuler使用**NetworkManager**来管理网络，执行下面命令即可重启网络。

```
systemctl restart NetworkManager
```

## 修复文件系统 

当系统强行上下电重启后，文件系统可能受到损坏，系统启动时会自动检查并修复文件系统，当文件系统没有自动修复成功时，便需要手动使用**fsck**进行扫描和修复。

```
# 此时一般会进入救援模式，在日志中查看是哪个文件系统路径损坏。
journalctl -xb
# 修复前检查该分区是否已经挂载
cat /proc/mounts
# 卸载该目录
umount xx
# 若无法卸载，**kill**占用该目录的进程
lsof | grep xxx
kill xxx
# 执行fsck命令进行修复，过程中需要输入yes or no
fsck -y /dev/xxx
```

## 手动dropcache

```
#当N数值不同时，可以达到不同的清理目的。根据linux内核文档建议，在清理前先执行sync（因为drop操作不会释放任何脏对象，而sync命令将所有未写的系统缓冲区写到磁盘中，包含已修改的inodes、已延迟的块I/O和读写映射文件，这样可以减少脏对象，从而让更多对象可以被释放。）
echo N > /proc/sys/vm/drop_caches

#释放pagecache
echo 1 > /proc/sys/vm/drop_caches

#释放dentries和inodes
echo 2 > /proc/sys/vm/drop_caches

#释放pagecache, dentries和inodes
echo 3 > /proc/sys/vm/drop_caches
```

## 救援模式和单用户模式 

- 救援模式

  挂载 openEuler 镜像进入救援模式。

  1. 选择Troubleshooting。

  2. 选择Rescue a openEuler system。

  3. 按提示操作进行。

     1）Continue 2）Read-only mount 3）Skip to shell 4）Quit(Reboot)

- 单用户模式

  在登录界面，输入字母e，进入grub界面，在linux行加入init=/bin/sh，按`ctrl+x`进入界面。

  1. 执行`mount -o remount,rw /`。
  2. 执行修改密码等操作。
  3. exit退出。


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/0fead30/  

