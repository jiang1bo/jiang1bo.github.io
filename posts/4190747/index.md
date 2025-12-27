# Openeuler，docker开箱指南


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

# 安装配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-安装配置)

本章节主要介绍和开源容器Docker安装相关的重要配置。

## 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-注意事项)

- Docker容器的安装需要使用root权限。
- docker-engine rpm包与containerd rpm包、runc rpm包、podman rpm包不能同时安装。因为docker-engine rpm包中已经包含Docker运行所需的所有组件，其中包括containerd、runc、docker二进制，且containerd、runc和podman rpm包也分别提供了对应的二进制，所以重复安装时会出现软件包冲突。

## 基本安装配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-基本安装配置)

### 配置daemon参数 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-配置daemon参数)

可以通过在/etc/docker/daemon.json文件中添加配置项自定义配置参数，相关配置项以及如何使用可以通过dockerd --help查看。配置示例如下：



```
cat /etc/docker/daemon.json 
{        
    "debug": true,        
    "storage-driver": "overlay2",        
    "storage-opts": ["overlay2.override_kernel_check=true"] 
}
```

### daemon运行目录配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-daemon运行目录配置)

用户需要明白重新指定各种运行目录和文件（包括--graph、--exec-root等），可能会存在目录冲突，或文件属性变换，对应用的正常使用造成影响。

须知



用户指定的目录或文件应为docker专用，避免冲突导致的文件属性变化带来安全问题。

- 以--graph为例，当我们使用/new/path/作为daemon新的Root Dir时，如果/new/path/下已经存在文件，且目录或文件名与docker需要使用的目录或文件名冲突（例如： containers、hooks、tmp等目录）时，docker可能会更新原有目录或文件的属性，包括属主、权限等为自己的属主和权限。

须知



从docker-17.05开始，--graph参数被标记为Deprecated，用新的参数--data-root替代。

### daemon自带网络配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-daemon自带网络配置)

- Docker daemon使用--bip参数指定docker0网桥的网段之后，如果在下一次重启的时候去掉--bip参数，docker0网桥会沿用上一次的--bip配置，即使重启之前已经删除docker0网桥。原因是docker会保存网络配置并在下一次重启的时候默认恢复上一次配置。
- Docker network create 并发创建网络的时候，可以创建具有相同名字的两个网络。原因是docker network是通过id来区分的，name只是个便于识别的别名而已，不保证唯一性。
- Docker在桥接bridge网络模式下，Docker容器是通过宿主机上的NAT模式，建立与宿主机之外世界的通信。Docker Daemon在启动一个容器时，每在宿主机上映射一个端口都会启动一个docker-proxy进程来实现访问代理。建议用户在使用这种userland-proxy时，只映射必须的端口，减少docker-proxy进行端口映射所消耗的资源。

### daemon-umask配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-daemon-umask配置)

容器主进程和exec进程的默认umask为0022，为了满足安全性需求，避免容器受到攻击，修改runc的实现，将默认umask修改为0027。修改后others群组将无法访问新建文件或目录。

docker启动容器时的默认umask值为0027，可以在dockerd启动时，使用--exec-opt native.umask=normal参数将容器启动时的umask修改为0022。

须知



如果docker create/run也配置了native.umask参数，则以docker create/run中的配置为准。

详细的配置见[docker create](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/容器管理-4.html#create)和[docker run](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/容器管理-4.html#run)章节的参数说明。

### daemon启动时间 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-daemon启动时间)

Docker服务由systemd管理，systemd对各个服务的启动时间有限制，如果指定时间内docker服务未能成功启动，则可能由以下原因导致：

- 如果使用devicemapper且为第一次启动，docker daemon需要对该设备做文件系统初始化操作，而该操作会进行大量磁盘IO操作，在磁盘性能不佳或存在大量IO竞争时，很可能会导致docker daemon启动超时。devicemapper设备只需要初始化一次，后续docker daemon启动时不再需要重复初始化。
- 如果当前系统资源占用太高，导致系统卡顿，系统所有的操作都会变慢，也可能会出现docker服务启动超时的情况。
- daemon重启过程中，需要遍历并读取docker工作目录下每一个容器的配置文件、容器init层和可写层的配置，如果当前系统存在过多容器（包含created和exited的容器），并且磁盘读写性能受限，也会出现daemon遍历文件过久导致docker服务启动超时的情况。

出现服务启动超时情况，建议对以下两种情况进行排查调整：

- 容器编排层定期清理不需要的容器，尤其是exited的容器。
- 结合解决方案的性能要求场景，调整编排层的清理周期和docker服务的启动时间。

### 关联组件journald [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-关联组件journald)

重启systemd-journald后需要重启docker daemon。journald通过pipe获取docker daemon的日志，如果journald服务重启，会导致该pipe被关闭，docker的日志写入操作便会触发SIGPIPE信号，该错误信号会导致docker daemon crash。由于忽略该信号影响严重，可能导致后续docker daemon的日志无法记录，因此建议用户在重启journald服务或者journald 异常后主动去重启docker daemon，保证docker日志能够被正常记录，避免daemon crash导致的状态异常。

### 关联组件firewalld [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-关联组件firewalld)

需要在重启或拉起firewalld之后重启docker服务，保证docker服务在firewalld之后启动。

- firewalld服务启动会清空当前系统的iptables规则，所以在启动docker daemon过程中，重启firewalld可能会导致docker服务插入iptables规则失败，从而导致docker服务启动失败。
- docker服务启动后重启firewalld服务，或者状态发生了变化（从启动到停止，或者从停止到启动），会导致docker的iptables规则被删除，创建带端口映射的容器失败。

### 关联组件iptables [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-关联组件iptables)

docker使用--icc=false选项时，可以限制容器之间互通，但若os自带某些规则，可以造成限制容器之间互通失效，例如：



```
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes) 
... 
0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0 
... 
0     0 DROP       all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
...
```

在Chain FORWARD中，DROP上面多出了一条ACCEPT icmp的规则，造成加了--icc=false后，容器之间也能ping通，但容器之间如果使用udp/tcp协议，对端仍然是不可达的。

因此，在容器os中使用docker，如果需要使用--icc=false选项时，建议先在host上清理一下iptables相关的规则。

### 关联组件audit [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-关联组件audit)

docker支持配置audit，但不是强制的。例如：



```
-w /var/lib/docker -k docker 
-w /etc/docker -k docker 
-w /usr/lib/systemd/system/docker.service -k docker 
-w /usr/lib/systemd/system/docker.socket -k docker 
-w /etc/sysconfig/docker -k docker 
-w /usr/bin/docker-containerd -k docker 
-w /usr/bin/docker-runc -k docker 
-w /etc/docker/daemon.json -k docker
```

配置docker的audit，好处在于可以记录更多信息便于审计，但从安全角度来看，它对防攻击并没有实质性的作用。另一方面，audit配置会导致严重的效率问题，可能导致系统卡顿，生产环境中请谨慎使用。

下面以“-w /var/lib/docker -k docker”为例，演示docker audit的配置：



```
[root@localhost signal]# cat /etc/audit/rules.d/audit.rules | grep docker -w /var/lib/docker/  -k docker 
[root@localhost signal]# auditctl -R /etc/audit/rules.d/audit.rules | grep docker 
[root@localhost signal]# auditctl -l | grep docker -w /var/lib/docker/ -p rwxa -k docker
```

说明



-p [r|w|x|a] 和-w一起使用，观察用户对这个目录的读、写、执行或者属性变化（如时间戳变化）。这样的话，在/var/lib/docker目录下的任何文件、目录操作，都会打印日志到audit.log中，从而会有太多的日志往audit.log中记录，会严重地影响auditd， 比如内存、cpu占用等，进而影响os的运行。例如：每次执行"ls /var/lib/docker/containers"都会有类似如下日志记录到/var/log/audit/audit.log中。



```
type=SYSCALL msg=audit(1517656451.457:8097): arch=c000003e syscall=257 success=yes exit=3 a0=ffffffffffffff9c a1=1b955b0 a2=90800 a3=0 items=1 ppid=17821 pid=1925 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts6 ses=4 comm="ls" exe="/usr/bin/ls" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="docker"type=CWD msg=audit(1517656451.457:8097):  cwd="/root"type=PATH msg=audit(1517656451.457:8097): item=0 name="/var/lib/docker/containers" inode=1049112 dev=fd:00 mode=040700 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:container_var_lib_t:s0 objtype=NORMAL
```

### 安全配置seccomp [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-安全配置seccomp)

在做容器网络性能测试时发现，Docker相对于原生内核namespace性能有所下降，经分析开启seccomp后，系统调用（如：sendto）不会通过system_call_fastpath进行，而是调用tracesys，这会带来性能大幅下降。因此，建议在有高性能要求的业务的容器场景下关闭seccomp，示例如下：



```
docker run -itd --security-opt seccomp=unconfined busybox:latest
```

### 禁止修改docker-daemon的私有目录 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-禁止修改docker-daemon的私有目录)

不允许对Docker用的根目录（默认/var/lib/docker）和运行时目录（默认/run/docker）以及其文件作任何修改，包括在该目录下删除文件，添加文件，对目录或者文件做软/硬链接，修改文件的属性/权限，修改文件的内容等，如果确实需要做修改，后果自负。

### 普通用户大量部署容器场景下的配置注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-普通用户大量部署容器场景下的配置注意事项)

普通用户在OS主机上能创建的进程数的上限，例如：可以在系统中创建配置文件“/etc/security/limits.d/20-nproc.conf”限制；类似的，普通用户在容器里也能创建的进程数的上限，由容器镜像中“/etc/security/limits.d/20-nproc.conf”文件对应的值决定，如下所示：



```
cat /etc/security/limits.conf 
*       soft    nproc   4096
```

当普通用户大量部署容器，导致容器内进程过多资源不够出现报错时，需要把容器镜像“/etc/security/limits.d/20-nproc.conf”文件中如上所示的4096配置值加大。

可配置的最大值请参考内核的最大能力，如下：



```
[root@localhost ~]# sysctl -a | grep pid_max 
kernel.pid_max = 32768
```

## 存储驱动配置 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-存储驱动配置)

本发行版docker支持overlay2和devicemapper两种存储驱动。由于overlay2较devicemapper而言，拥有更好的性能，建议用户在生产环境中优先考虑。

### 配置overlay2存储驱动 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-配置overlay2存储驱动)

#### 配置方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-配置方法)

docker默认为使用overlay2存储驱动，也可以通过如下两种方式显式指定。

- 编辑/etc/docker/daemon.json，通过storage-driver字段显式指定。

  

  ```
  cat /etc/docker/daemon.json
  {
      "storage-driver": "overlay2"
  }
  ```

- 编辑/etc/sysconfig/docker-storage，通过docker daemon启动参数显式指定。

  

  ```
  cat /etc/sysconfig/docker-storage 
  DOCKER_STORAGE_OPTIONS="--storage-driver=overlay2"
  ```

#### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-注意事项-1)

- 部分容器生命周期管理的操作会报找不到相应的rootfs或者相关的可执行文件。

- 如果容器的健康检查配置的是执行容器内的可执行文件，也会报错，导致容器的健康检查失败。

- 如果将overlay2作为graphdriver，在容器中第一次修改镜像中的文件时，若该文件的大小大于系统剩余的空间，修改将会失败。因为即使修改很小，也要把这个文件完整的拷贝到上层，剩余空间不足导致失败。

- overlay2文件系统相比普通文件系统天然存在一些行为差异，归纳如下：

  - 内核版本

    overlay2只兼容原生4.0以上内核，建议配合使用ext4文件系统。

  - Copy-UP性能问题

    修改lower层文件会触发文件复制到upper层，其中数据块复制和fsync比较耗时。

  - rename目录问题

    - 只有源路径和目标路径都在merged层时，才允许rename系统调用，否则rename系统调用会报错-EXDEV。

    - 内核4.10引入了redirect dir特性来修复rename问题，对应内核选项为CONFIG_OVERLAY_FS_REDIRECT_DIR。

      在使用overlay2场景下，对文件系统目录进行重命名时，如果系统配置文件/sys/module/overlay/parameters/redirect_dir中配置的特性开关为关闭状态，则会导致使用失败；如果用户要使用相关特性，需要用户手动设置/sys/module/overlay/parameters/redirect_dir为“Y”。

  - Hard link break问题

    - 当lower层目录中有多个硬链接，在merged层写入数据会触发Copy-UP，导致硬链接断开。
    - 内核4.13引入了index feature来修复这个问题，对应内核选项为 CONFIG_OVERLAY_FS_INDEX。注意这个选项没有前向兼容性，不支持热升级。

  - st_dev和st_ino变化

    触发Copy-UP之后，用户只能看到merged层中的新文件，inode会变化。虽然attr和xattr可以复制，但st_dev和st_ino具有唯一性，不可复制。这会导致stat和ls查看 到相应的变化。

  - fd变化

    Copy-UP之前，以只读模式打开文件得到描述符fd1，Copy-UP之后，打开同名文件得到文件描述符fd2， 二者实际指向不同的文件。向fd2写入的数据不会在fd1中体现。

#### 异常场景 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-异常场景)

容器使用配置了overlay2存储驱动的过程中，可能出现挂载点被覆盖的异常情况。例如

#### 异常场景-挂载点被覆盖 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-异常场景-挂载点被覆盖)

挂载关系：在问题容器的挂载点的下面，存在一个/var/lib/docker/overlay2的挂载点：



```
[root@localhost ~]# mount -l | grep overlay 
overlay on /var/lib/docker/overlay2/844fd3bca8e616572935808061f009d106a8748dfd29a0a4025645457fa21785/merged type overlay (rw,relatime,seclabel,lowerdir=/var/lib/docker/overlay2/l/JL5PZQLNDCIBU3ZOG3LPPDBHIJ:/var/lib/docker/overlay2/l/ELRPYU4JJG4FDPRLZJCZZE4UO6,upperdir=/var/lib/docker/overlay2/844fd3bca8e616572935808061f009d106a8748dfd29a0a4025645457fa21785/diff,workdir=/var/lib/docker/overlay2/844fd3bca8e616572935808061f009d106a8748dfd29a0a4025645457fa21785/work) 
/dev/mapper/dm-root on /var/lib/docker/overlay2 type ext4 (rw,relatime,seclabel,data=ordered)
```

执行部分docker命令会遇到错误，比如：



```
[root@localhost ~]# docker rm 1348136d32
docker rm: Error response from daemon: driver "overlay2" failed to remove root filesystem for 1348136d32: error while removing /var/lib/docker/overlay2/844fd3bca8e616572935808061f009d106a8748dfd29a0a4025645457fa21785: invalid argument
```

此时，在主机侧可以发现对应容器的rootfs找不到，但这并不意味着rootfs丢失，只是被/var/lib/docker/overlay2挂载点覆盖，业务仍然可以正常运行，不受影响。修复方案可以参考如下：

- 修复方案一

  1. 确定当前docker所使用graphdriver：

     

     ```
     docker info | grep "Storage Driver"
     ```

  2. 查询当前的挂载点：

     

     ```
     Devicemapper: mount -l | grep devicemapper 
     Overlay2: mount -l | grep overlay2
     ```

     输出格式为： A on B type C (D)

     其中，A：块设备名称或overlay，B：挂载点，C：文件系统类型，D：挂载属性。

  3. 从下往上逐一umount这些挂载点B。

  4. 然后全部docker restart这些容器，或者删除所有容器。

  5. 重启docker。

     

     ```
     systemctl restart docker
     ```

- 修复方案二

  1. 业务迁移
  2. 节点重启

### 配置devicemapper存储驱动 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-配置devicemapper存储驱动)

用户如果需要使用devicemapper存储驱动，可以通过如下两种方式显式指定。

- 编辑/etc/docker/daemon.json，通过storage-driver字段显式指定。

  

  ```
  cat /etc/docker/daemon.json
  {
      "storage-driver": "devicemapper"
  }
  ```

- 编辑/etc/sysconfig/docker-storage，通过docker daemon启动参数显式指定。

  

  ```
  cat /etc/sysconfig/docker-storage 
  DOCKER_STORAGE_OPTIONS="--storage-driver=devicemapper"
  ```

#### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-注意事项-2)

- 使用devicemapper必须使用devicemapper+direct-lvm的方式。

- 配置devicemapper时，如果系统上没有足够的空间给thinpool做自动扩容，请禁止自动扩容功能。

- 禁止把/etc/lvm/profile/docker-thinpool.profile中如下两个值都改成100。

  

  ```
  activation {   
    thin_pool_autoextend_threshold=80   
    thin_pool_autoextend_percent=20 
  }
  ```

- 使用devicemapper时推荐加上--storage-opt dm.use_deferred_deletion=true --storage-opt dm.use_deferred_removal=true。

- 使用devicemapper时，容器文件系统推荐使用ext4，需要在docker daemon的配置参数中加 上--storage-opt dm.fs=ext4。

- 当graphdriver为devicemapper时，如果metadata文件损坏且不可恢复，需要人工介入恢复。禁止直接操作或篡改daemon存储devicemapper的元数据。

- 使用devicemapper lvm时，异常掉电导致的devicemapper thinpool损坏，无法保证thinpool损坏后可以修复，也不能保证数据的完整性，需重建thinpool。

docker daemon开启了user namespace特性，切换devicemapper存储池时的**注意事项**

- 一般启动容器时，deviceset-metadata文件为：/var/lib/docker/devicemapper/metadata/deviceset-metadata。
- 使用了user namespace场景下，deviceset-metadata文件使用的是：/var/lib/docker/{userNSUID.GID}/devicemapper/metadata/deviceset-metadata。
- 使用devicemapper存储驱动，容器在user namespace场景和普通场景之间切换时，需要将对应deviceset-metadata文件中的BaseDeviceUUID内容清空；针对thinpool扩容或者重建的场景下，也同样的需要将对应deviceset-metadata文件中的BaseDeviceUUID内容清空，否则docker服务会重启失败。

## 强制退出docker相关后台进程的影响 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-强制退出docker相关后台进程的影响)

### 信号量残留 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-信号量残留)

使用devicemapper作为graphdriver时，强制退出强制退出可能导致信号量残留。docker在操作dm的过程中会创建信号量，如果在释放信号量前，daemon被强制退出，可能导致该信号量无法释放，一次强制退出最多泄露一个信号量，泄露概率低。而linux系统有信号量上限限制，当信号量泄露次数达到上限值时将无法创建新的信号量，进而导致docker daemon启动失败。排查方法如下：

1. 首先查看系统上残留的信号量

   

   ```
   $ ipcs 
   ------ Message Queues -------- 
   key        msqid      owner      perms      used-bytes   messages 
   ------ Shared Memory Segments -------- 
   key        shmid      owner      perms      bytes      nattch     status 
   ------ Semaphore Arrays -------- 
   key        semid      owner      perms      nsems 
   0x0d4d3358 238977024  root       600        1 
   0x0d4d0ec9 270172161  root       600        1 
   0x0d4dc02e 281640962  root       600        1
   ```

2. 接着用dmsetup查看devicemapper创建的信号量，该信号量集合是上一步中查看到的系统信号量的子集

   

   ```
   dmsetup udevcookies
   ```

3. 最后查看内核信号量设置上限，第四个值就是当前系统的信号量使用上限

   

   ```
   # cat /proc/sys/kernel/sem 
   250     32000   32      128
   ```

   如果步骤1中残留的信号量数量与步骤3中看到的信号量上限相等，则是达到上限，此时docker daemon无法正常启动。可以使用下述命令增加信号量使用上限值来让docker恢复启动

   

   ```
   echo 250 32000  32  1024 > /proc/sys/kernel/sem
   ```

   也可以手动清理devicemapper残留的信号量（下面是清理一分钟以前申请的dm相关信号量）

   

   ```
   # dmsetup udevcomplete_all 1 
   This operation will destroy all semaphores older than 1 minutes with keys that have a prefix 3405 (0xd4d). 
   Do you really want to continue? [y/n]: y 
   0 semaphores with keys prefixed by 3405 (0xd4d) destroyed. 0 skipped.
   ```

### 网卡残留 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-网卡残留)

使用bridge模式启动容器的过程中，强制退出daemon可能导致网卡残留。使用bridge网络模式，当docker创建容器时，会先在host上创建一对veth，然后再把该网卡信息存到数据库中，如果在创建完成，存到docker的数据库之前，daemon被强制退出，那么该网卡无法被docker关联，下次启动也无法删除（docker本身会清理自己数据库中不用的网卡），从而造成网卡残留。

### 重启容器失败 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-重启容器失败)

容器hook耗时较长，且启动阶段遇到containerd被强制退出，再次执行容器start操作可能失败。容器启动阶段遇到containerd被强制退出，docker start操作直接返回错误；containerd被重新拉起后，上次启动可能仍处于runc create执行阶段（执行用户自定义hook，可能耗时较长），此时再次下发docker start命令启动该容器，可能提示以下错误：



```
Error response from daemon: oci runtime error: container with id exists: xxxxxx
```

该错误是由runc create一个已经存在（创建中）的容器导致，等第一次start对应的runc操作结束后再次执行docker start便可以成功。

由于hook的执行不受docker控制，这种场景下尝试回收该容器有可能导致containerd进程启动卡死（执行未知hook程序），且问题的风险可控（短期影响当前容器的创建）：

- 问题出现后等待第一次操作结束可以再次成功启动该容器。
- 一般是在容器启动失败后创建新的容器，不复用已经失败的容器。

综上，该问题暂时作为场景约束。

### 服务无法正常重启 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-服务无法正常重启)

短时间内频繁重启docker服务导致该服务无法正常重启。docker系统服务由systemd负责监控，如果docker服务在10s内重启次数超过5次，systemd服务就会监控到该异常行为，因此会禁止docker服务启动。只有等到下一个10s周期开始后，docker服务才能响应重启命令正常重启。

## 系统掉电影响 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/installation_and_configuration_3.html#user-content-系统掉电影响)

主机意外掉电或系统panic等场景下，由于docker daemon的状态无法及时刷新到磁盘，导致重启后docker daemon状态不正常，可能出现的问题有（包括但不限于）：

- 掉电前创建的容器，重启后docker ps -a看不到，该问题是因为该容器的状态文件没有刷新到磁盘，从而导致重启后daemon无法获取到该容器的状态（镜像、卷、网络等也可能会有类似问题）。

- 掉电前某个文件正处于写入状态，尚未完全写入，重启后daemon重新加载该文件发现文件格式不正常或内容不完整，导致重启加载出错。

- 针对掉电时会破坏docker DB的情况，在重启节点时会清理data-root下面的db文件。因此重启前创建的如下信息在重启后会被删除：

  - network，用docker network创建的资源会在重启后清除。

  - volume，用 docker volume创建的资源会在重启后删除。

  - 构建缓存，构建缓存信息会在重启后删除。

  - containerd保存的元数据，由于启动容器会重建containerd元数据，重启节点会清理containerd中保存的元数据。

    说明

    

    用户若选择采用手动清理恢复环境的方式，可通过配置环境变量“DISABLE_CRASH_FILES_DELETE=true”屏蔽daemon掉电重启时db文件清理功能。





# 容器管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-容器管理)

## 创建容器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-创建容器)

### 下载镜像 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-下载镜像)

运行docker命令需要root权限，当你使用普通用户登录时，需要用sudo权限执行docker命令。



```
[root@localhost ~]# docker pull busybox
```

该命令行将在docker官方的镜像库中下载busybox:latest（命令行中没指定TAG，所以使用默认的TAG名latest），镜像在下载过程中将检测所依赖的层本地是否存在，如果存在就跳过。从私有镜像库下载镜像时，请带上registry描述，例如：假如建立了一个私有镜像库，地址为192.168.1.110:5000，里面有一些常用镜像。使用下面命令行从私有镜像库中下载镜像。



```
[root@localhost ~]# docker pull 192.168.1.110:5000/busybox
```

从私有镜像库中下载下来的image名字带有镜像库地址的信息名字比较长，可以用docker tag命令生成一个名字简单点的image。



```
[root@localhost ~]# docker tag 192.168.1.110:5000/busybox busybox
```

可以通过docker images命令查看本地镜像列表。

### 运行一个简单的应用 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-运行一个简单的应用)



```
[root@localhost ~]# docker run busybox /bin/echo "Hello world"
Hello world
```

该命令行使用busybox:latest（命令行中没有指定tag，所以使用默认的tag名latest）镜像创建了一个容器，在容器内执行了echo "Hello world"。使用下面命令行可以查看刚才创建的这个容器。



```
[root@localhost ~]# docker ps -l
CONTAINER ID        IMAGE               COMMAND                   CREATED             STATUS                     PORTS               NAMES
d8c0a3315bc0        busybox "/bin/echo 'Hello wo…"   5 seconds ago       Exited (0) 3 seconds ago                       practical_franklin
```

### 创建一个交互式的容器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-创建一个交互式的容器)



```
[root@localhost ~]# docker run -it busybox /bin/bash
root@bf22919af2cf:/# ls 
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var 
root@bf22919af2cf:/# pwd 
/
```

-ti选项分配一个伪终端给容器并可以使用STDIN进行交互，可以看到这时可以在容器内执行一些命令。这时的容器看起来完全是一个独立的linux虚拟机。使用exit命令退出容器。

### 后台运行容器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-后台运行容器)

执行下面命令行，-d指示这个容器在后台运行，--name=container1 指定容器的名字为container1。



```
[root@localhost ~]# docker run -d --name=container1 busybox /bin/sh -c "while true;do echo hello world;sleep 1;done"
7804d3e16d69b41aac5f9bf20d5f263e2da081b1de50044105b1e3f536b6db1c
```

命令行的执行结果是返回了这个容器的ID，没有返回命令的执行结果hello world，此时容器在后台运行，可以用docker ps命令查看正在运行的容器:



```
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
7804d3e16d69        busybox "/bin/sh -c 'while tr"   11 seconds ago      Up 10 seconds                           container1
```

用docker logs查看容器运行的输出：



```
[root@localhost ~]# docker logs container1
hello world
hello world
hello world
...
```

### 容器网络连接 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-容器网络连接)

默认情况下，容器可以访问外部网络，而外部网络访问容器时需要通过端口映射，下面以在docker中运行私有镜像库服务registry为例。下面的命令行中-P使registry镜像中开放的端口暴露给主机。



```
[root@localhost ~]# docker run --name=container_registry -d -P registry 
cb883f6216c2b08a8c439b3957fb396c847a99079448ca741cc90724de4e4731
```

container_registry这个容器已经启动了，但是并不知道容器中的服务映射到主机的哪个端口，通过docker port查看端口映射。



```
[root@localhost ~]# docker port container_registry 
5000/tcp -> 0.0.0.0:49155
```

从输出可以看出，容器内的5000端口映射到了主机的49155端口。通过主机IP:49155就可以访问registry服务了，在浏览器中输入[http://localhost:49155](http://localhost:49155/)就可以返回registry的版本信息。

在运行registry镜像的时候还可以直接指定端口映射如：



```
docker run --name=container_registry -d -p 5000:5000 registry
```

通过-p 5000:5000指定容器的5000端口映射到主机的5000端口。

### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-注意事项)

- **启动容器不能单独加-a stdin**

  启动容器时，不能单独加-a stdin，必须要同时加上-a stdout或者-a stderr，否则会导致终端即使在容器退出后也会卡住。

- **避免使用已有容器的长id、短id作为新容器的name**

  创建容器时，避免使用已有容器A的长id或短id作为新容器B的name。若使用容器A的长id作为容器B的name，当使用容器B的name作为指定容器进行操作时，docker匹配到的是容器A。若使用容器A的短id作为容器B的name，当使用容器A的短id作为指定容器进行相关操作时，docker匹配到的是容器B。这是因为，docker在匹配容器时，先精确匹配所有容器的长id。若未匹配成功，再根据container_name进行精确匹配；若还未匹配成功，直接对容器id进行模糊匹配。

- **使用sh/bash等依赖标准输入输出的容器应该使用`-ti`参数，避免出现异常**

  正常情况：不用`-ti`参数启动sh/bash等进程容器，容器会马上退出。

  出现这种问题的原因在于，docker会先创建一个匹配用于容器内业务的stdin，在不设置-ti等交互式参数时，docker会在容器启动后关闭该pipe，而业务容器进程sh/bash在检测到stdin被关闭后会直接退出。

  异常情况：如果在上述过程中的特定阶段（关闭该pipe之前）强制杀死docker daemon，会导致该pipe的daemon端没有被及时关闭，这样即使不带`-ti`的sh/bash进程也不会退出，导致异常场景，这种容器就需要手动清理。

  Daemon重启后会接管原有的容器stream，而不带`-ti`参数的容器可能就无法处理（因为正常情况下这些容器不存在stream需要接管）；真实业务下几乎不存在这种使用方式(不带 `-ti`的sh/bash没有任何作用)，为了避免这类问题发生，限制交互类容器应该使用 `-ti`参数。

- **容器存储卷**

  启动容器时如果通过`-v`参数将主机上的文件挂载到容器中，在主机或容器中使用vi或sed命令修改文件可能会使文件inode发生改变，从而导致主机和容器内的文件不同步。容器中挂载文件时应该尽量避免使用这种文件挂载的方式（或不与vi和sed同时使用），也可以通过挂载文件上层目录来避免该问题。在docker挂载卷时“nocopy”选项可以避免将容器内挂载点目录下原有的文件拷贝到主机源目录下，但是这个选项只能在挂载匿名卷时使用，不能在bind mount的场景下使用。

- **避免使用可能会对host造成影响的选项**

  --privileged 选项会让容器获得所有权限，容器可以做挂载操作和修改/proc、/sys等目录，可能会对host造成影响，普通容器需要避免使用该选项。

  共享host的namespace，比如--pid host/--ipc host/--net host等选项可以让容器跟host共享命名空间，同样会导致容器影响host的结果，需要避免使用。

- **kernel memory cgroup不稳定，禁止使用**

  kernel memory cgroup在小于4.0版本的Linux内核上仍属于实验阶段，运行起来不稳定，虽然Docker的Warning说是小于4.0就可以，但是我们评估认为，kmemcg在高版本内核仍然不稳定，所以不管是低版本还是高版本，均禁止使用。

  当docker run --kernel-memory时，会产生如下告警：

  

  ```
  WARNING: You specified a kernel memory limit on a kernel older than 4.0. Kernel memory limits are experimental on older kernels, it won't work as expected as expected and can cause your system to be unstable.
  ```

- **blkio-weight参数在支持blkio精确控制的内核下不可用**

  --blkio-weight-device 可以实现容器内更为精确的blkio控制，该控制需要指定磁盘设备，可以通过docker --blkio-weight-device参数实现。同时在这种内核下docker不再提供--blkio-weight方式限制容器blkio，使用该参数创建容器将会报错:

  

  ```
  docker: Error response from daemon: oci runtime error: container_linux.go:247: starting container process caused "process_linux.go:398: container init caused \"process_linux.go:369: setting cgroup config for ready process caused \\\"blkio.weight not supported, use weight_device instead\\\"\""
  ```

- **使用--blkio-weight-device需要磁盘支持CFQ调度策略**

  --blkio-weight-device参数需要磁盘工作于完全公平队列调度（CFQ：Completely Fair Queuing）的策略时才能工作。

  通过查看磁盘scheduler文件（‘/sys/block/<磁盘>/queue/scheduler’）可以获知磁盘支持的策略以及当前所采用的策略，如查看sda：

  

  ```
  # cat /sys/block/sda/queue/scheduler noop [deadline] cfq
  ```

  当前sda支持三种调度策略：noop, deadline, cfq，并且正在使用deadline策略。通过echo修改策略为cfq：

  

  ```
  # echo cfq > /sys/block/sda/queue/scheduler
  ```

- **容器基础镜像中systemd使用限制**

  通过基础镜像创建的容器在使用过程中，容器基础镜像中的systemd仅用于系统容器，普通容器不支持使用。

### 并发性能 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-并发性能)

- docker内部的消息缓冲有一个上限，超过这个上限就会将消息丢弃，因此在并发执行命令时建议不要超过1000条命令，否则有可能会造成docker内部消息丢失，从而造成容器无法启动等严重问题。
- 并发创建容器并对容器执行restart时会偶现“oci runtime error: container init still running”报错，这是因为containerd对事件等待队列进行了性能优化，容器stop过程中执行runc delete，尝试在1s内kill掉容器的init进程，如果1s内init进程还没有被kill掉的话runc会返回该错误。由于containerd的GC（垃圾回收机制）每隔10s会回收之前runc delete的残留资源， 所以并不影响下次对容器的操作，一般出现上述报错的话等待4~5s之后再次启动容器即可。

### 安全特性解读 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-安全特性解读)

1. docker默认的权能配置分析

   原生的docker默认配置如下，默认进程携带的Cap如下:

   

   ```
   "CAP_CHOWN", 
   "CAP_DAC_OVERRIDE", 
   "CAP_FSETID", 
   "CAP_FOWNER", 
   "CAP_MKNOD", 
   "CAP_NET_RAW", 
   "CAP_SETGID", 
   "CAP_SETUID", 
   "CAP_SETFCAP", 
   "CAP_SETPCAP", 
   "CAP_NET_BIND_SERVICE", 
   "CAP_SYS_CHROOT", 
   "CAP_KILL", 
   "CAP_AUDIT_WRITE",
   ```

   默认的seccomp配置是白名单，不在白名单的syscall默认会返回SCMP_ACT_ERRNO，根据给docker不同的Cap开放不同的系统调用，不在上面的权限，默认docker都不会给到容器。

2. CAP_SYS_MODULE

   CAP_SYS_MODULE这个Cap是让容器可以插入或移除ko，增加该Cap可以让容器逃逸，甚至破坏内核。因为容器最大的隔离是Namespace，在ko中只要把他的Namespace指向init_nsproxy即可。

3. CAP_SYS_ADMIN

   sys_admin权限给容器带来的能力有：

   - 文件系统（mount，umount，quotactl）
   - namespace设置相关的（setns，unshare，clone new namespace）
   - driver ioctl
   - 对pci的控制，pciconfig_read, pciconfig_write, pciconfig_iobase
   - sethostname

4. CAP_NET_ADMIN

   容器中有访问网络接口的和sniff网络流量的权限，容器可以获取到所有容器包括host的网络流量，对网络隔离破坏极大。

5. CAP_DAC_READ_SEARCH

   该权限开放了open_by_handle_at和name_to_handle_at两个系统调用，如果host上没有selinux保护，容器中可通过暴力搜索file_handle结构的inode号，进而可以打开host上的任意文件，影响文件系统的隔离性。

6. CAP_SYS_RAWIO

   容器中可对host写入io端口，可造成host内核崩溃。

7. CAP_SYS_PTRACE

   容器中有ptrace权限，可对容器的进程进行ptrace调试。现runc已经修补该漏洞，但有些工具比如nsenter和docker-enter并没有改保护，容器中可对这些工具执行的进程进行调试，获取这些工具带入的资源信息（Namespace、fd等），另外， ptrace可以绕过seccomp，极大增加内核攻击面。

8. Docker Cap接口 --cap-add all

   --cap-add all表示赋予容器所有的权能，包括本节提到的比较危险的权能，使得容器可以逃逸。

9. 不要禁用docker的seccomp特性

   默认的docker有一个seccomp的配置，配置中使用的是白名单，不在配置的sys_call会被seccomp禁掉，使用接口--security-opt 'seccomp:unconfined'可以禁止使用seccomp特性。如果禁用seccomp或使用自定义seccomp配置但过滤名单不全，都会增加容器对内核的攻击面。

10. 不要配置/sys和/proc目录可写

    /sys和/proc目录包含了linux维护内核参数、设备管理的接口，容器中配置该目录可写可能会导致容器逃逸。

11. Docker开放Cap --CAP_AUDIT_CONTROL

    容器可以通过控制系统audit系统，并且通过AUDIT_TTY_GET/AUDIT_TTY_SET等命令可以获取审计系统中记录的tty执行输入记录，包括root密码。

12. CAP_BLOCK_SUSPEND和CAP_WAKE_ALARM

    容器可拥有阻塞系统挂起(epoll)的能力。

13. CAP_IPC_LOCK

    容器拥有该权限后，可以突破ulimit中的max locked memory限制，任意mlock超大内存块，造成一定意义的DoS攻击。

14. CAP_SYS_LOG

    容器拥有该权限后，可以dmesg读取系统内核日志，突破内核kaslr防护。

15. CAP_SYS_NICE

    容器拥有该权限后，可以改变进程的调度策略和优先级，造成一定意义的DoS攻击。

16. CAP_SYS_RESOURCE

    容器可以绕过对其的一些资源限制，比如磁盘空间资源限制、keymaps数量限制、pipe-size-max限制等，造成一定意义的DoS攻击。

17. CAP_SYS_TIME

    容器可以改变host上的时间。

18. Docker默认Cap风险分析

    Docker默认的Cap，包含了CAP_SETUID和CAP_FSETID，如host和容器共享目录，容器可对共享目录的二进制文件进行+s设置，host上的普通用户可使用其进行提权CAP_AUDIT_WRITE，容器可以对host写入，容器可以对host写入日志，host需配置日志防爆措施。

19. Docker和host共享namespace参数，比如 --pid，--ipc, --uts

    该参数为容器和host共享namespace空间，容器和host的namespace隔离没有了，容器可对host进行攻击。比如，使用--pid 和host共享pid namespace，容器中可以看到host上的进程pid号，可以随意杀死host的进程。

20. --device 把host的敏感目录或者设备，映射到容器中

    Docker管理面有接口可以把host上的目录或者设备映射到容器中，比如--device，-v等参数，不要把host上的敏感目录或者设备映射到容器中。

## 创建容器使用hook-spec [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-创建容器使用hook-spec)

### 原理及使用场景 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-原理及使用场景)

docker支持hook的扩展特性，hook应用与底层runc的执行过程中，遵循OCI标准：[https://github.com/opencontainers/runtime-spec/blob/main/config.md#hooks](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#https://github.com/opencontainers/runtime-spec/blob/main/config.md#hooks) 。

hook主要有三种类型：prestart，poststart，poststop。分别作用于容器内用户应用程序启动之前，容器应用程序启动之后，容器应用程序停止之后。

### 接口参考 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-接口参考)

当前为docker run和create命令增加了参数“--hook-spec”，后面接spec文件的绝对路径，可以指定容器启动时的需要添加的hook，这些hook会自动附加在docker自己动态创建的hook后面（当前docker只有一个libnetwork的prestart hook），随容器的启动/销毁过程执行用户指定的程序。

spec的结构体定义为：



```
// Hook specifies a command that is run at a particular event in the lifecycle of a container
type Hook struct{       
               Path    string   `json:"path"`    
               Args    []string `json:"args,omitempty"`    
               Env     []string `json:"env,omitempty"`      
               Timeout *int     `json:"timeout,omitempty"`
}
// Hooks for container setup and teardown
type  Hooks struct{
               // Prestart is a list of hooks to be run before the container process is executed.
               // On Linux, they are run after the container namespaces are created.         
               Prestart []Hook `json:"prestart,omitempty"`
               // Poststart is a list of hooks to be run after the container process is started.         
               Poststart []Hook `json:"poststart,omitempty"`
               // Poststop is a list of hooks to be run after the container process exits.         
               Poststop []Hook `json:"poststop,omitempty"`
}
```

- Spec文件的path、args、env 都是必填信息；
- Timeout选填(建议配置)，参数类型为int，不接受浮点数，范围为[1, 120]。
- Spec内容应该是json格式的，格式不对会报错，示例参考前面。
- 使用的时候既可以`docker run --hook-spec /tmp/hookspec.json xxx`, 也可以 `docker create --hook-spec /tmp/hookspec.json xxx && docker start xxx`。

### 为容器定制特有的hook [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-为容器定制特有的hook)

以启动过程中添加一个网卡的过程来说明。下面是相应的hook spec文件内容：



```
{
    "prestart": [
         {
             "path": "/var/lib/docker/hooks/network-hook",             
             "args": ["network-hook", "tap0", "myTap"],             
             "env": [],
             "timeout": 5
         }
     ],
     "poststart":[],     
     "poststop":[]
}
```

指定prestart hook增加一个网络hook的执行。路径是/var/lib/docker/hooks/network-hook，args代表程序的参数，第一个参数一般是程序名字，第二个是程序接受的参数。对于network-hook这个hook程序，需要两个参数，第一个是主机上的网卡名字，第二个是在容器内的网卡重命名。

- 注意事项

  1. hook path必须为docker的graph目录（--graph）下的hooks文件夹下，默认一般为 /var/lib/docker/hooks，可以通过docker info命令查看root路径。

     

     ```
     [root@localhost ~]# docker info 
     ... 
     Docker Root Dir: /var/lib/docker 
     ...
     ```

     这个路径可能会跟随用户手动配置，以及user namespace的使用（daemon --userns-remap）而变化。 path进行软链接解析后，必须以Docker Root Dir/hooks开头（如本例中使用 /var/lib/docker/hooks开头），否则会直接报错。

  2. hooks path必须指定绝对路径，因为这个是由daemon处理，相对路径对daemon无意义。同时绝对路径也更满足安全要求。

  3. hook程序打印到stderr的输出会打印给客户端并对容器的声明周期产生影响（比如启动失败），而输出到stdout的打印信息会被直接忽略。

  4. 严禁在hook里反向调用docker的指令。

  5. 配置的hook执行文件必须要有可执行权限，否则hook执行会报错。

  6. 使用hook时，执行时间应尽量短。如果hook中的prestart时间过长（超过2分钟），则会导致容器启动超时失败，如果hook中的poststop时间过长（超过2分钟），也会导致容器异常。

     目前已知的异常如下：执行docker stop命令停止容器时，2分钟超时执行清理时，由于hook还没执行结束，因此会等待hook执行结束（该过程持有锁），从而导致和该容器相关的操作都会卡住，需要等到hook执行结束才能恢复。另外，由于docker stop命令的2分钟超时处理是异步的过程，因此即使docker stop命令返回了成功，容器的状态也依然是up状态，需要等到hook执行完后状态才会修改为exited。

- 使用建议

  1. 建议配置hook的Timeout超时时间阈值，超时时间最好在5s以内。
  2. 建议不要配置过多hook，每个容器建议prestart、poststart、poststop这三个hook都只配置一个，过多hook会导致启动时间长。
  3. 建议用户识别多个hook之间的依赖关系，如果存在依赖关系，在组合hook配置文件时要根据依赖关系灵活调整顺序，hook的执行顺序是按照配置的spec文件上的先后顺序。

### 多个hook-spec [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-多个hook-spec)

当有多个hook配置文件，要运行多个hook时，用户必须自己手工将多个hook配置文件组合成一个配置文件，使用--hook-spec参数指定此合并后的配置文件，方可生效所有的hook；如果配置多个--hook-spec参数，则只有最后一个生效。

配置举例：

hook1.json内容如下：



```
# cat /var/lib/docker/hooks/hookspec.json 
{
    "prestart": [
        {
            "path": "/var/lib/docker/hooks/lxcfs-hook",             
            "args": ["lxcfs-hook", "--log", "/var/log/lxcfs-hook.log"],             
            "env": []
        }
     ],     
     "poststart":[],     
     "poststop":[]
}
```

hook2.json内容如下：



```
# cat /etc/isulad-tools/hookspec.json 
{
      "prestart": [
         {
               "path": "/docker-root/hooks/docker-hooks",             
               "args": ["docker-hooks", "--state", "prestart"],             
               "env": []
         }
       ],     
       "poststart":[],     
       "poststop":[
          {
               "path": "/docker-root/hooks/docker-hooks",             
               "args": ["docker-hooks", "--state", "poststop"],             
               "env": []
          }
        ]
}
```

手工合并后的json内容如下：



```
{
       "prestart":[
          {
                "path": "/var/lib/docker/hooks/lxcfs-hook",             
                "args": ["lxcfs-hook", "--log", "/var/log/lxcfs-hook.log"],             
                "env": []
           },         
           {
                "path": "/docker-root/hooks/docker-hooks",             
                "args": ["docker-hooks", "--state", "prestart"],             
                "env": []
           }
        ],     
        "poststart":[],     
        "poststop":[
            {
                "path": "/docker-root/hooks/docker-hooks",             
                "args": ["docker-hooks", "--state", "poststop"],             
                "env": []
            }
         ]
}
```

需要注意的是，docker daemon会按照数组顺序依次读取hook配置文件中prestart等action中的hook二进制，进行执行动作。用户需要识别多个hook之间的依赖关系，如果有依赖关系，在组合hook配置文件时要根据依赖关系灵活调整顺序。

### 为所有容器定制默认的hook [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-为所有容器定制默认的hook)

Docker daemon同样可以接收--hook-spec的参数，--hook-spec的语义与docker create/run的--hook-spec参数相同，这里不再复述。也可以在/etc/docker/daemon.json里添加hook配置：



```
{
     "hook-spec": "/tmp/hookspec.json"
}
```

容器在运行时，会首先执行daemon定义的--hook-spec中指定的hooks，然后再执行每个容器单独定制的hooks。

## 创建容器配置健康检查 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-创建容器配置健康检查)

Docker提供了用户定义的对容器进行健康检查的功能。在Dockerfile中配置HEALTHCHECK CMD选项，或在容器创建时配置--health-cmd选项，在容器内部周期性地执行命令，通过命令的返回值来监测容器的健康状态。

### 配置方法 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-配置方法)

- 在Dockerfile中添加配置，如：

  

  ```
  HEALTHCHECK --interval=5m --timeout=3s --health-exit-on-unhealthy=true \
     CMD curl -f http://localhost/ || exit 1
  ```

  可配置的选项：

  1. --interval=DURATION，默认 30s，相邻两次命令执行的间隔时间。另外，容器启动后，经过interval时间进行第一次检查。

  2. --timeout=DURATION，默认 30s，单次检查命令执行的时间上限，超时则任务命令执行失败。

  3. --start-period=DURATION，默认 0s，容器初始化时间。初始化期间也会执行健康检查，健康检查失败不会计入最大重试次数。但是，如果在初始化期间运行状况检查成功，则认为容器已启动。之后所有连续的检查失败都将计入最大重试次数。

  4. --retries=N，默认 3，健康检查失败最大的重试次数。

  5. --health-exit-on-unhealthy=BOOLEAN，默认false，检测到容器非健康时是否杀死容器

  6. CMD，必选，在容器内执行的命令。返回值为0表示成功，非0表示失败。

     在配置了HEALTHCHECK后创建镜像，HEALTHCHECK相关配置会被写入镜像的配置中。通过docker inspect可以看到。如：

     

     ```
     "Healthcheck": {
         "Test": [
             "CMD-SHELL",
             "/test.sh"
         ]
     },
     ```

- 在容器创建时的配置：

  

  ```
  docker run -itd --health-cmd "curl -f http://localhost/ || exit 1" --health-interval 5m --health-timeout 3s --health-exit-on-unhealthy centos bash
  ```

  可配置的选项：

  1. --health-cmd，必选，在容器内执行的命令。返回值为0表示成功，非0表示失败。

  2. --health-interval，默认 30s，最大为int64上限（纳秒）相邻两次命令执行的间隔时间。

  3. --health-timeout，默认 30s，最大为int64上限（纳秒），单次检查命令执行的时间上限，超时则任务命令执行失败。

  4. --health-start-period，默认 0s，最大为int64上限（纳秒），容器初始化时间。

  5. --health-retries，默认 3，最大为int32上限，健康检查失败最大的重试次数。

  6. --health-exit-on-unhealthy，默认false，检测到容器非健康时是否杀死容器。

     容器启动后，HEALTHCHECK相关配置会被写入容器的配置中。通过docker inspect可以看到。如：

     

     ```
     "Healthcheck": {
         "Test": [
             "CMD-SHELL",
             "/test.sh"
         ]
     },
     ```

### 检查规则 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-检查规则)

1. 容器启动后，容器状态中显示health:starting。
2. 经过start-period时间后开始，以interval为间隔周期性在容器中执行CMD。即：当一次命令执行完毕后，经过interval时间，执行下一次命令。
3. 若CMD命令在timeout限制的时间内执行完毕，并且返回值为0，则视为一次检查成功，否则视为一次检查失败。检查成功后，容器状态变为health:healthy。
4. 若CMD命令连续retries次检查失败，则容器状态变为health:unhealthy。失败后容器也会继续进行健康检查。
5. 容器状态为health:unhealthy时，任意一次检查成功会使得容器状态变为health:healthy。
6. 设置--health-exit-on-unhealthy的情况下，如果容器因为非被杀死退出（退出返回值137）后，健康检查只有容器在重新启动后才会继续生效。
7. CMD执行完毕或超时时，docker daemon会将这次检查的起始时间、返回值和标准输出记录到容器的配置文件中。最多记录最新的5条数据。此外，容器的配置文件中还存储着健康检查的相关参数。

通过docker ps可以看到容器状态。



```
[root@bac shm]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
7de2228674a2        testimg             "bash"              About an hour ago   Up About an hour (unhealthy)                       cocky_davinci
```

运行中的容器的健康检查状态也会被写入容器配置中。通过docker inspect可以看到。



```
"Health": {
    "Status": "healthy",
    "FailingStreak": 0,
    "Log": [
        {
            "Start": "2018-03-07T07:44:15.481414707-05:00",
            "End": "2018-03-07T07:44:15.556908311-05:00",
            "ExitCode": 0,
            "Output": ""
        },
        {
            "Start": "2018-03-07T07:44:18.557297462-05:00",
            "End": "2018-03-07T07:44:18.63035891-05:00",
            "ExitCode": 0,
            "Output": ""
        },
        ......
}
```

说明



- 容器内健康检查的状态信息最多保存5条。会保存最后得到的5条记录。
- 容器内健康检查相关配置同时最多只能有一条生效。Dockerfile中配置的靠后的条目会覆盖靠前的；容器创建时的配置会覆盖镜像中的。
- 在Dockerfile中可以通过 HEALTHCHECK NONE来取消引用的镜像中的健康检查配置。在容器运行时可以通过配置--no-healthcheck来取消镜像中的健康检查配置。不允许在启动时同时配置健康检查相关选项与--no-healthcheck选项。
- 带有健康检查配置的容器启动后，若docker daemon退出，则健康检查不会执行，一直等待。docker daemon再次启动后，容器健康状态会变为starting。之后检查规则同上。
- 构建容器镜像时若健康检查相关参数配置为空，则按照缺省值处理。
- 容器启动时若健康检查相关参数配置为0，则按照缺省值处理。

## 停止与删除容器 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-停止与删除容器)

用docker stop停止名为container1的容器：



```
[root@localhost ~]# docker stop container1
```

也可以用docker kill来杀死容器达到停止容器的目的：



```
[root@localhost ~]# docker kill container1
```

当容器停止之后，可以使用docker rm删除容器：



```
[root@localhost ~]# docker rm container1
```

当然，使用docker rm -f 强制删除容器也是可以的：



```
[root@localhost ~]# docker rm -f container1
```

### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-注意事项-1)

- 禁止使用docker rm -f XXX 删除容器。如果使用强制删除，docker rm会忽略过程中的错误，可能导致容器相关元数据残留。如果使用普通删除，如果删除过程出错，则会删除失败，不会导致元数据残留。
- 避免使用docker kill命令。docker kill命令发送相关信号给容器内业务进程，依赖于容器内业务进程对信号的处理策略，可能导致业务进程的信号处理行为与指令的预期不符合的情况。
- docker stop处于restarting状态的容器可能容器不会马上停止。如果一个容器使用了重启规则，当容器处于restarting状态时，docker stop这个容器时有很低的概率会立即返回，容器仍然会在重启规则的作用下再次启动。
- 不能用docker restart重启加了--rm参数的容器。加了--rm参数的容器在退出时，容器会主动删除，如果重启一个加了--rm的参数的容器， 可能会导致一些异常情况，比如启动容器时，同时加了--rm与-ti参数，对容器执行restart操作，可能会概率性卡住无法退出。

### docker stop/restart 指定t参数且t<0时，请确保自己容器的应用会处理stop信号 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-docker-stoprestart-指定t参数且t0时请确保自己容器的应用会处理stop信号)

Stop的原理：（Restart会调用Stop流程）

1. Stop会首先给容器发送Stop 信号（15）
2. 然后等待一定的时间（这个时间就是用户输入的 t）
3. 过了一定时间，如果容器还活着，那么就发送kill信号（9）使容器强制退出

输入参数t（单位s）的含义：

- t<0 : 表示死等，不管多久都等待程序优雅退出，既然用户这么输入了，表示对自己的应用比较放心，认为自己的程序有合理的stop信号的处理机制
- t=0 ： 表示不等，立即发送kill -9 到容器
- t>0 ： 表示等一定的时间，如果容器还未退出，就发送kill -9 到容器

所以如果用户使用t<0 (比如t=-1)，请确保自己容器的应用会正确处理signal 15，如果容器忽略了该信号，会导致docker stop一直卡住。

### 如果容器处于Dead状态，可能底层文件系统处于busy状态，需要手动删除 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-如果容器处于dead状态可能底层文件系统处于busy状态需要手动删除)

Docker在执行容器删除时，先停止容器的相关进程，之后将容器状态更改为Dead，最后执行容器rootfs的删除操作。当文件系统或者device mapper处于忙碌状态时，最后一步rootfs的删除会失败。docker ps -a查看会发现容器处于Dead状态。Dead状态的容器不能再次启动，需要等待文件系统不繁忙时，手动再次执行docker rm进行删除。

### 共享pid namespace容器，子容器处于pause状态会使得父容器stop卡住，并影响docker run命令执行 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-共享pid-namespace容器子容器处于pause状态会使得父容器stop卡住并影响docker-run命令执行)

使用--pid参数创建共享pid namespace的父子容器，在执行docker stop父容器时，如果子容器中有进程无法退出（比如处于D状态、pause状态），会产生父容器docker stop命令等待的情况，需要手动恢复这些进程，才能正常执行命令。

遇到该问题的时候，请对pause状态的容器使用docker inspect 命令查询 PidMode对应的父容器是否为需要docker stop的容器。如果是该容器，请使用docker unpause将子容器解除pause状态，指令即可继续执行。

一般来说，导致该类问题的可能原因是容器对应的pid namespace由于进程残留导致无法被销毁。如果上述方法无法解决问题，可以通过借助linux工具，获取容器内残留进程，确定pid namespace中进程无法退出的原因，解决后容器就可以退出：

- 获取容器pid namespace id

  

  ```
  docker inspect --format={{.State.Pid}} CONTAINERID | awk '{print  "/proc/"$1"/ns/pid"}' |xargs readlink
  ```

- 获取该namespace下的线程

  

  ```
   ls -l /proc/*/task/*/ns/pid |grep -F PIDNAMESPACE_ID |awk '{print $9}' |awk -F  \/ '{print $5}'
  ```

## 容器信息查询 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-容器信息查询)

在任何情况下，容器的状态都不应该以docker命令执行是否成功返回为判断标准。如想查看容器状态，建议使用：



```
docker inspect <NAME|ID>
```

## 修改操作 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-修改操作)

### docker exec进入容器启动多个进程的注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-docker-exec进入容器启动多个进程的注意事项)

docker exec进入容器执行的第一个命令为 bash 命令时，当退出 exec 时，要保证在这次exec启动的进程都退出了，再执行exit退出，否则会导致exit退出时终端卡住的情况。如果要在exit退出时，exec中启动的进程仍然在后台保持运行，要在启动进程时加上nohup。

### docker rename和docker stats <container_name>的使用冲突 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-docker-rename和docker-stats-container_name的使用冲突)

如果使用`docker stats <container_name>` 实时监控容器，当使用docker rename重命名容器之后，docker stats中显示的名字将还是原来的名字，不是rename后的名字。

### docker rename操作restarting状态的容器可能会失败 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-docker-rename操作restarting状态的容器可能会失败)

对一个处于restarting状态的容器执行rename操作的时候，docker会同步修改容器网络的相关配置。由于restarting状态的容器可能还未真正启动起来，网络是不存在的，导致rename操作报错sandbox不存在。建议rename只操作非restarting的稳定状态的容器。

### docker cp [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-docker-cp)

1. 使用docker cp向容器中拷贝文件时，docker ps以及所有对这个容器的操作都将等待docker cp结束之后才能进行。
2. 容器以非root用户运行，当使用docker cp命令复制主机上的一个非root权限的文件到容器时，文件在容器中的权限角色会变成root。docker cp与cp命令不同，docker cp会修改复制到容器中文件的uid和gid为root。

### docker login [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_1.html#user-content-docker-login)

执行docker login后，会将user/passwd经 aes（256位）加密后保存在/root/.docker/config.json，同时生成 *root*.docker/aeskey(权限0600)，用来解密/root/.docker/config.json中的 user/passwd。目前不能定时更新aeskey，只能由用户手动删除aeskey来更新。aeskey更新后，不管是否重启过docker daemon，都需要重新login，才可以push。例如：



```
root@hello:~/workspace/dockerfile# docker login 
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one. 
Username: example Password: 
Login Succeeded 
root@hello:~/workspace/dockerfile# docker push example/empty 
The push refers to a repository [docker.io/example/empty] 
547b6288eb33: Layer already exists 
latest: digest: sha256:99d4fb4ce6c6f850f3b39f54f8eca0bbd9e92bd326761a61f106a10454b8900b size: 524 
root@hello:~/workspace/dockerfile# rm /root/.docker/aeskey 
root@hello:~/workspace/dockerfile# docker push example/empty 
WARNING: Error loading config file:/root/.docker/config.json - illegal base64 data at input byte 0 
The push refers to a repository [docker.io/example/empty] 
547b6288eb33: Layer already exists 
errors: 
denied: requested access to the resource is denied 
unauthorized: authentication required 
root@hello:~/workspace/dockerfile# docker login 
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one. 
Username: example 
Password: 
Login Succeeded 
root@hello:~/workspace/dockerfile# docker push example/empty 
The push refers to a repository [docker.io/example/empty] 
547b6288eb33: Layer already exists 
latest: digest: sha256:99d4fb4ce6c6f850f3b39f54f8eca0bbd9e92bd326761a61f106a10454b8900b size: 524
```

# 镜像管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-镜像管理)

## 创建镜像 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-创建镜像)

docker pull、docker build、docker commit、docker import、docker load都可以创建一个新的镜像，关于这些命令的使用详见命令行参考镜像管理。

### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-注意事项)

1. 避免并发docker load和docker rmi操作。 如果同时满足如下两个条件，可能导致并发性问题：

   - 某个镜像存在于系统中。
   - 同时对该镜像进行docker rmi和docker load操作。

   所以使用时应该避免这种场景（注：所有的镜像创建操作如tag，build，load和rmi并发都有可能会导致类似的错误，应该尽量避免这类操作与rmi的并发）。

2. 如果Docker操作镜像时系统掉电，可能导致镜像损坏，需要手动恢复。

   由于Docker在操作镜像（pull/load/rmi/build/combine/commit/import等）时,镜像数据的操作是异步的、镜像元数据是同步的。所以如果在镜像数据未全部刷到磁盘时掉电，可能导致镜像数据和元数据不一致。对用户的表现是镜像可以看到(有可能是none 镜像)，但是无法启动容器，或者启动后的容器有异常。这种情况下应该先使用docker rmi删除该镜像，然后重新进行之前的操作，系统可以恢复。

3. 生产环境节点应避免存留超大数量镜像，请及时清理不使用的镜像。

   镜像数目过多会导致docker image等命令执行过慢，从而导致docker build/docker commit等相关命令执行失败，并可能导致内存堆积。在生产环境中，请及时清理不再使用的镜像和中间过程镜像。

4. 使用--no-parent参数build镜像时，如果有多个build操作同时进行，并且Dockerfile里 FROM的镜像相同，则可能会残留镜像，分为以下两种情况：

   - FROM的镜像不是完整镜像，则有可能会残留FROM的镜像运行时生成的镜像。残留的镜像名类似base_v1.0.0-app_v2.0.0，或者残留<none>镜像。
   - 如果Dockerfile里的前几条指令相同，则有可能会残留<none>镜像。

### 可能会产生none镜像场景 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-可能会产生none镜像场景)

1. none镜像是指没有tag的最顶层镜像，比如ubuntu的imageID，只有一个tag是ubuntu，如果这个tag没了，但是imageID还在，那么这个imageID就变成了none镜像。
2. Save镜像的过程中因为要把镜像的数据导出来，所以对image进行保护，但是如果这个时候来一个删除操作，可能会untag成功，删除镜像ID失败，造成该镜像变成none镜像。
3. 执行docker pull时掉电，或者系统panic，可能出现none镜像，为保证镜像完整性，此时可通过docker rmi 删除镜像后重新拉取。
4. 执行docker save保存镜像时，如果指定的名字为镜像ID，则load后的镜像也没有tag，其镜像名为none。

### build镜像的同时删除该镜像，有极低概率导致镜像build失败 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-build镜像的同时删除该镜像有极低概率导致镜像build失败)

目前的build镜像的过程是通过引用计数来保护的，当build完一个镜像后，紧接着就给该镜像的引用计数加1（holdon操作），一旦holdon操作成功，该镜像就不会被删除了，但是在holdon之前，有极低的概率，还是可以删除成功，导致build镜像失败。

## 查看镜像 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-查看镜像)

查看本地镜像列表：



```
docker images
```

## 删除镜像 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-删除镜像)

删除镜像（image处为具体镜像名）：



```
docker rmi image
```

### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_1.html#user-content-注意事项-1)

禁止使用docker rmi -f XXX删除镜像。如果使用强制删除，docker rmi会忽略过程中的错误，可能导致容器或者镜像元数据残留。如果使用普通删除，如果删除过程出错，则会删除失败，不会导致元数据残留。



# docker命令行参考



# 容器引擎 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_engine.html#user-content-容器引擎)

Docker daemon是一个常驻后台的系统进程，docker 子命令执行前先要启动docker daemon。

如果是通过rpm包或者系统包管理工具安装的，就可以使用systemctl start docker来启动docker daemon。

docker命令支持多个参数选项，对于参数选项有以下约定：

1. 单个字符的选项可以合并在一起，如：

   

   ```
   docker run -t -i busybox /bin/sh
   ```

   可以写成

   

   ```
   docker run -ti busybox /bin/sh
   ```

2. 在命令帮助中看到的如--icc=true之类的bool命令选项，如果没有使用这个选项，则这个标志位的值就是在命令帮助中看到的缺省值，如果使用了这个选项则这个标志位的值就是命令帮助中看的值的相反值，如果启动docker daemon没有加上使用--icc选项，则默认设置了--icc=true,如果使用了--icc选项则表示是--icc=false。

3. 在命令帮助中看到的--attach=[]之类的选项，表示这类的选项可以多次设置，如：

   

   ```
   docker run --attach=stdin --attach=stdout -i -t busybox /bin/sh
   ```

4. 在命令帮助中看到的-a, --attach=[]之类的选项，表示这种选项既可以用-a value指定也可以用--attach=value指定。如：

   

   ```
   docker run -a stdin --attach=stdout -i -t busybox /bin/sh
   ```

5. --name=””之类的选项需要的是一个字符串，只能指定一次，-c=0之类的选项需要的是一个整数，只能指定一次。

**表 1** docker daemon启动时指定参数详解



| **参数名称**                          | **说明**                                                     |
| :------------------------------------ | :----------------------------------------------------------- |
| --api-cors-header                     | 开放远程API调用的 [CORS 头信息](https://en.wikipedia.org/wiki/Cross-Origin_Resource_Sharing)。这个接口开关对想进行二次开发的上层应用提供了支持。为remote API设置CORS头信息。 |
| --authorization-plugin=[]             | 指定认证插件。                                               |
| -b, --bridge=""                       | 挂载已经存在的网桥设备到 Docker 容器里。注意，使用 **none** 可以停用容器里的网络。 |
| --bip=""                              | 使用 CIDR 地址来设定自动创建的网桥的 IP。注意，此参数和 -b 不能一起使用。 |
| --cgroup-parent                       | 为所有容器设定cgroup父目录。                                 |
| --config-file=/etc/docker/daemon.json | 启动docker daemon的配置文件。                                |
| --containerd                          | 指定containerd的socket路径。                                 |
| -D, --debug=false                     | 开启Debug模式。                                              |
| --default-gateway                     | 容器IPv4地址的默认网关。                                     |
| --default-gateway-v6                  | 容器IPv6地址的默认网关。                                     |
| --default-ulimit=[]                   | 容器的默认ulimit值。                                         |
| --disable-legacy-registry             | 不允许使用原版registry。                                     |
| --dns=[]                              | 强制容器使用DNS服务器。例如： --dns 8.8.x.x                  |
| --dns-opt=[]                          | 指定使用DNS的选项。                                          |
| --dns-search=[]                       | 强制容器使用指定的DNS搜索域名。例如： --dns-search example.com |
| --exec-opt=[]                         | 设置运行时执行选项。例如支持native.umask选项：`# 启动的容器umask值为0022 --exec-opt native.umask=normal # 启动的容器umask值为0027（缺省值）--exec-opt  native.umask=secure`注意如果docker create/run也配置了native.umask参数则以docker create/run中的配置为准。 |
| --exec-root=/var/run/docker           | 指定执行状态文件存放的根目录。                               |
| --fixed-cidr=""                       | 设定子网固定IP（ex: 10.20.0.0/16），这个子网IP必须属于网桥内的。 |
| --fixed-cidr-v6                       | 同上，使用与IPv6。                                           |
| -G, --group="docker"                  | 在后台运行模式下，赋予指定的Group到相应的unix socket上。注意，当此参数 --group 赋予空字符串时，将去除组信息。 |
| -g, --graph="/var/lib/docker"         | 配置Docker运行时根目录。                                     |
| -H, --host=[]                         | 在后台模式下指定socket绑定，可以绑定一个或多个 tcp://host:port, unix:///path/to/socket, fd://* 或 fd://socketfd。例如：$ dockerd -H tcp://0.0.0.0:2375或者$ export DOCKER_HOST="tcp://0.0.0.0:2375" |
| --insecure-registry=[]                | 指定非安全连接的仓库，docker默认所有的连接都是TLS证书来保证安全的，如果仓库不支持https连接或者证书是docker daemon不清楚的证书颁发机构颁发的，则启动daemon的时候要指定如--insecure-registry=192.168.1.110:5000，使用私有仓库都要指定。 |
| --image-layer-check=true              | 开启镜像层完整性检查功能，设置为true；关闭该功能，设置为false。如果没有该参数，默认为关闭。docker启动时会检查镜像层的完整性，如果镜像层被破坏，则相关的镜像不可用。docker进行镜像完整性校验时，无法校验内容为空的文件和目录，以及链接文件。因此若镜像因掉电导致上述类型文件丢失，docker的镜像数据完整性校验可能无法识别。docker版本变更时需要检查是否支持该参数，如果不支持，需要从配置文件中删除。 |
| --icc=true                            | 启用容器间的通信。                                           |
| --ip="0.0.0.0"                        | 容器绑定端口时使用的默认IP地址。                             |
| --ip-forward=true                     | 启动容器的 net.ipv4.ip_forward。                             |
| --ip-masq=true                        | 使能IP伪装。                                                 |
| --iptables=true                       | 启动Docker容器自定义的iptable规则。                          |
| -l, --log-level=info                  | 设置日志级别。                                               |
| --label=[]                            | 设置daemon标签，以key=value形式设置。                        |
| --log-driver=json-file                | 设置容器日志的默认日志驱动。                                 |
| --log-opt=map[]                       | 设置日志驱动参数。                                           |
| --mtu=0                               | 设置容器网络的MTU值，如果没有这个参数，选用默认 route MTU，如果没有默认route，就设置成常量值 1500。 |
| -p, --pidfile="/var/run/docker.pid"   | 后台进程PID文件路径。                                        |
| --raw-logs                            | 带有全部时间戳并不带ANSI颜色方案的日志。                     |
| --registry-mirror=[]                  | 指定dockerd优先使用的镜像仓库。                              |
| -s, --storage-driver=""               | 强制容器运行时使用指定的存储驱动                             |
| --selinux-enabled=false               | 启用selinux支持，3.10.0-862.14及以上内核版本不支持--selinux-enabled=true。 |
| --storage-opt=[]                      | 配置存储驱动的参数，存储驱动为devicemapper的时候有效（e.g. dockerd --storage-opt dm.blocksize=512K）。 |
| --tls=false                           | 启动TLS认证开关。                                            |
| --tlscacert="/root/.docker/ca.pem"    | 通过CA认证过的certificate文件路径。                          |
| --tlscert="/root/.docker/cert.pem"    | TLS的certificate文件路径。                                   |
| --tlskey="/root/.docker/key.pem"      | TLS的key文件路径。                                           |
| --tlsverify=false                     | 使用TLS并做后台进程与客户端通讯的验证。                      |
| --insecure-skip-verify-enforce        | 是否强制跳过证书的主机名/域名验证，默认为false（不跳过）。   |
| --use-decrypted-key=true              | 指定使用解密私钥。                                           |
| --userland-proxy=true                 | 容器LO设备使用userland proxy。                               |
| --userns-remap                        | 容器内使用user命名空间的用户映射表。说明：当前版本不支持该参数。 |

# 容器管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-容器管理)

# 总体说明 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-总体说明)

当前docker支持的子命令，按照功能划分为以下几组：



| **功能划分**    | **命令**                                 | **命令功能**                     |                                 |
| :-------------- | :--------------------------------------- | :------------------------------- | ------------------------------- |
| 主机环境相关    | version                                  | 查看docker版本信息               |                                 |
| info            | 查看docker系统和主机环境信息             |                                  |                                 |
| 容器相关        | 容器生命周期管理                         | create                           | 由image创建一个容器             |
| run             | 由image创建一个容器并运行                |                                  |                                 |
| start           | 开始一个已停止运行的容器                 |                                  |                                 |
| stop            | 停止一个运行中的容器                     |                                  |                                 |
| restart         | 重启一个容器                             |                                  |                                 |
| wait            | 等待一个容器停止，并打印出退出码         |                                  |                                 |
| rm              | 删除一个容器                             |                                  |                                 |
| 容器内进程管理  | pause                                    | 暂停一个容器内的所有进程         |                                 |
| unpause         | 恢复一个容器内被暂停的所用进程           |                                  |                                 |
| top             | 查看容器内的进程                         |                                  |                                 |
| exec            | 在容器内执行进程                         |                                  |                                 |
| 容器检视工具    | ps                                       | 查看运行中的容器（不加任何选项） |                                 |
| logs            | 显示一个容器的日志信息                   |                                  |                                 |
| attach          | 连接到一个容器的标准输入输出             |                                  |                                 |
| inspect         | 返回容器的底层信息                       |                                  |                                 |
| port            | 列出容器与主机的端口映射                 |                                  |                                 |
| diff            | 返回容器相对于镜像中的rootfs所作的改动   |                                  |                                 |
| cp              | 容器与主机之间复制文件                   |                                  |                                 |
| export          | 将一个容器中的文件系统导出为一个tar包    |                                  |                                 |
| stats           | 实时查看容器的资源占用情况               |                                  |                                 |
| images相关      | 生成一个新image                          | build                            | 通过一个Dockerfile构建一个image |
| commit          | 基于容器的rootfs创建一个新的image        |                                  |                                 |
| import          | 将tar包中的内容作为文件系统创建一个image |                                  |                                 |
| load            | 从一个tar包中加载一个image               |                                  |                                 |
| 与image仓库有关 | login                                    | 登录一个registry                 |                                 |
| logout          | 登出一个registry                         |                                  |                                 |
| pull            | 从registry中拉取一个image                |                                  |                                 |
| push            | 将一个image推送到registry中              |                                  |                                 |
| search          | 在registry中搜寻image                    |                                  |                                 |
| 与image管理有关 | images                                   | 显示系统中的image                |                                 |
| history         | 显示一个image的变化历史                  |                                  |                                 |
| rmi             | 删除image                                |                                  |                                 |
| tag             | 给image打标签                            |                                  |                                 |
| save            | 将一个image保存到一个tar包中             |                                  |                                 |
| 其他            | events                                   | 从docker daemon中获取实时事件    |                                 |
| rename          | 重命名容器                               |                                  |                                 |

其中有些子命令还有一些参数选项如docker run,通过docker COMMAND --help可以查看相应COMMAND命令的帮助，命令选项参考上文的命令选项约定。下面详细介绍每个命令的使用。

## attach [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-attach)

用法：**docker attach [OPTIONS] CONTAINER**

功能：附加到一个运行着的容器

选项：

--no-stdin=false 不附加STDIN

--sig-proxy=true 代理所有到容器内部的信号，不代理SIGCHLD, SIGKILL, SIGSTOP

示例：



```
$ sudo docker attach attach_test
root@2988b8658669:/# ls bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```sh

## commit

用法：**docker commit \[OPTIONS\] CONTAINER \[REPOSITORY\[:TAG\]\]**

功能：由一个容器创建一个新的image

选项：

-a, \--author=""    指定作者

-m, \--message=""  提交的信息

-p, \--pause=true   在提交过程中暂停容器

示例：

运行一个容器，然后将这个容器提交成一个新的image

```sh
$ sudo docker commit test busybox:test
sha256:be4672959e8bd8a4291fbdd9e99be932912fe80b062fba3c9b16ee83720c33e1

$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              e02e811dd08f        2 years ago         1.09MB
```

## cp [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-cp)

用法：docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-

docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

功能：从指定的容器内的一个路径复制文件或文件夹到主机的指定路径中，或者把主机的文件或者文件夹拷贝到容器内。

注意：docker cp不支持容器内/proc，/sys，/dev，/tmp等虚拟文件系统以及用户在容器内自行挂载的文件系统内的文件拷贝。

选项：

-a, --archive 将拷贝到容器的文件属主设置为容器运行用户（--user）

-L, --follow-link 解析并跟踪文件的符号链接

示例：

复制registry容器中/test目录到主机的/home/aaa目录中



```
$ sudo docker cp registry:/test /home/aaa
```

## create [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-create)

用法：**docker create [OPTIONS] IMAGE [COMMAND] [ARG...]**

功能：使用image创建一个新的容器，并将返回一个容器的ID，创建之后的容器用docker start命令启动，OPTIONS用于创建容器时对容器进行配置，有些选项将覆盖image中对容器的配置，COMMAND指定容器启动时执行的命令。

选项：

**表 2** 参数说明



| 参数                       | 参数含义                                                     |
| :------------------------- | :----------------------------------------------------------- |
| -a --attach=[]             | 使控制台Attach到容器内进程的STDIN,STDOUT,STDERR              |
| --name=""                  | 指定容器的名字                                               |
| --add-host=[host:ip]       | 在容器内的/etc/hosts中添加一个hostname到IP地址的映射e.g. --add-host=test:10.10.10.10 |
| --annotation               | 设置容器的annotations。例如支持native.umask选项：`--annotation native.umask=normal  启动的容器umask值为0022 --annotation native.umask=secure # 启动的容器umask值为0027`注意如果没有配置该参数，则使用dockerd中的umask配置。 |
| --blkio-weight             | blockio的相对权重，从10到1000                                |
| --blkio-weight-device=[]   | blockio权重（设置相对权重）                                  |
| -c, --cpu-shares=0         | 容器获得主机CPU的相对权重，通过设置这个选项获得更高的优先级，默认所有的容器都是获得相同的CPU优先权。 |
| --cap-add=[]               | 添加Linux权能                                                |
| --cap-drop=[]              | 清除Linux权能                                                |
| --cgroup-parent            | 为容器设置cgroup父目录                                       |
| --cidfile=""               | 将容器的ID写到指定的文件中e.g. --cidfile=/home/cidfile-test 将该容器的ID写入到/home/cidfile-test中 |
| --cpu-period               | 设置CFS（完全公平调度策略）进程的CPU周期。缺省值为100ms；一般--cpu-period参数和--cpu-quota是配合使用的，比如--cpu-period=50000 --cpu-quota=25000，意味着如果有1个CPU，该容器可以每50ms获取到50%的CPU。使用--cpus=0.5也可达到同样的效果 |
| --cpu-quota                | 设置CFS(完全公平调度策略)进程的CPU配额，默认为0，即没有限制  |
| --cpuset-cpus              | 设置容器中进程允许运行的CPU (0-3, 0,1)。默认没有限制         |
| --cpuset-mems              | 设置容器中进程运行的内存节点 (0-3, 0,1)，只对NUMA系统起作用  |
| --device=[]                | 将主机的设备添加到容器中 (e.g. --device=/dev/sdc:/dev/xvdc:rwm) |
| --dns=[]                   | 强制容器使用指定的dns服务器（e.g. 创建容器时指定--dns=114.114.xxx.xxx，将在容器的/etc/resolv.conf中写入nameserver 114.114.xxx.xxx并将覆盖原来的内容） |
| --dns-opt=[]               | 设置DNS选项                                                  |
| --dns-search=[]            | 强制容器使用指定的dns搜索域名                                |
| -e, --env=[]               | 设置容器的环境变量--env=[KERNEL_MODULES=]:在容器中插入指定模块。目前仅支持Host主机上有的模块，且容器删除后Host主机上模块仍然驻留，且容器需要同时指定--hook-spec选项。以下都是参数的合法格式：KERNEL_MODULERS=KERNEL_MODULERS=aKERNEL_MODULERS=a,bKERNEL_MODULERS=a,b, |
| --entrypoint=""            | 覆盖image中原有的entrypoint，entrypoint设置容器启动时执行的命令 |
| --env-file=[]              | 从一个文件中读取环境变量，多个环境变量在文件中按行分割（e.g. --env-file=/home/test/env,其中env文件中存放了多个环境变量） |
| --expose=[]                | 开放一个容器内部的端口，使用下文介绍的-P选项将会使开放的端口映射到主机的一个端口。 |
| --group-add=[]             | 指定容器添加到额外的组                                       |
| -h, --hostname=""          | 设置容器主机名                                               |
| --health-cmd               | 设置容器健康检查执行的命令                                   |
| --health-interval          | 相邻两次命令执行的间隔时间，默认 30s                         |
| --health-timeout           | 单次检查命令执行的时间上限，超时则任务命令执行失败，默认30s  |
| --health-start-period      | 容器启动距离第一次执行健康检查开始的时间，默认0s             |
| --health-retries           | 健康检查失败最大的重试次数，默认3                            |
| --health-exit-on-unhealthy | 容器被检查为非健康后停止容器，默认false                      |
| --host-channel=[]          | 设置一个通道供容器内进程与主机进行通信，格式：<host path>:<container path>:<rw/ro>:<size limit> |
| -i, --interactive=false    | 设置STDIN打开即使没有attached                                |
| --ip                       | 设置容器的IPv4地址                                           |
| --ip6                      | 设置容器的IPv6地址                                           |
| --ipc                      | 指定容器的ipc命名空间                                        |
| --isolation                | 指定容器隔离策略                                             |
| -l, --label=[]             | 设置容器的标签                                               |
| --label-file=[]            | 从文件中获取标签                                             |
| --link=[]                  | 链接到其他容器，这个选项将在容器中添加一些被链接容器IP地址和端口的环境变量及在/etc/hosts中添加一条映射（e.g. --link=name:alias） |
| --log-driver               | 设置容器的日志驱动                                           |
| --log-opt=[]               | 设置日志驱动选项                                             |
| -m, --memory=""            | 设置容器的内存限制，格式<number><optional unit>, 其中 unit = b, k, m or g。该参数最小值为4m。 |
| --mac-address              | 设置容器的mac地址 (e.g. 92:d0:c6:0a:xx:xx)                   |
| --memory-reservation       | 设置容器内存限制，默认与--memory一致。可认为--memory是硬限制，--memory-reservation是软限制；当使用内存超过预设值时，会动态调整（系统回收内存时尝试将使用内存降低到预设值以下），但不确保一定不超过预设值。一般可以和--memory一起使用，数值小于--memory的预设值。 |
| --memory-swap              | 设置普通内存和交换分区的使用总量，-1为不做限制。如果不设置，则为--memory值的2倍，即SWAP可再使用与--memory相同的内存量。 |
| --memory-swappiness=-1     | 设置容器使用交换内存的时机,以剩余内存百分比为度量（0-100）   |
| --net="bridge"             | 设置容器的网络模式，当前1.3.0版本的docker有四个模式：bridge、host、none、container:<name\|id>。默认使用的是bridge。bridge：使用桥接模式在docker daemon启动时使用的网桥上创建一个网络栈。host：在容器内使用主机的网络栈none：不使用网络container:<name\|id>：重复利用另外一个容器的网络栈 |
| --no-healthcheck           | 设置容器不使用健康检查                                       |
| --oom-kill-disable         | 禁用OOM killer，建议如果不设置-m参数，也不要设置此参数。     |
| --oom-score-adj            | 调整容器的oom规则（-1000到1000）                             |
| -P, --publish-all=false    | 将容器开放的所有端口一一映射到主机的端口，通过主机的端口可以访问容器内部，通过下文介绍的docker port命令可以查看具体容器端口和主机端口具体的映射关系。 |
| -p, --publish=[]           | 将容器内的一个端口映射到主机的一个端口，format: ip:hostPort:containerPort \| ip::containerPort \| hostPort:containerPort \| containerPort，如果没有指定IP代表侦听主机所有网卡的访问，如果没有指定hostPort,表示自动分配主机的端口。 |
| --pid                      | 设置容器的PID命名空间                                        |
| --privileged=false         | 给予容器额外的权限，当使用了--privileged选项，容器将可以访问主机的所有设备。 |
| --restart=""               | 设置容器退出时候的重启规则，当前1.3.1版本支持3个规则：no：当容器停止时，不重启。on-failure：当容器退出码为非0时重启容器，这个规则可以附加最大重启次数，如on-failure:5，最多重启5次。always：无论退出码是什么都退出。 |
| --read-only                | 将容器的根文件系统以只读的形式挂载                           |
| --security-opt=[]          | 容器安全规则                                                 |
| --shm-size                 | /dev/shm设备的大小，缺省值是64M                              |
| --stop-signal=SIGTERM      | 容器停止信号，默认为SIGTERM                                  |
| -t, --tty=false            | 分配一个伪终端                                               |
| --tmpfs=[]                 | 挂载tmpfs目录                                                |
| -u, --user=""              | 指定用户名或者用户ID                                         |
| --ulimit=[]                | ulimit选项                                                   |
| --userns                   | 指定容器的user命名空间                                       |
| -v, --volume=[]            | 将主机的一个目录挂载到容器内部，或者在容器中创建一个新卷（e.g. -v /home/test:/home将主机的/home/test目录挂载到容器的/home目录下，-v /tmp 在容器中的根目录下创建tmp文件夹，该文件夹可以被其他容器用--volumes-from选项共享 ）。不支持将主机目录挂载到容器/proc子目录，否则启动容器会报错。 |
| --volume-driver            | 设置容器的数据卷驱动，可选。                                 |
| --volumes-from=[]          | 将另外一个容器的卷挂载到本容器中，实现卷的共享（e.g. -volumes-from container_name将container_name中的卷挂载到这个容器中 ）。-v和--volumes-from=[]是两个非常重要的选项用于数据的备份和热迁移。 |
| -w, --workdir=""           | 指定容器的工作目录，进入容器时的目录                         |

示例：

创建了一个名为busybox的容器，创建之后的容器用docker start命令启动。



```
$ sudo docker create -ti --name=busybox busybox /bin/bash
```

## diff [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-diff)

用法：**docker diff CONTAINER**

功能：检视容器的差异，相比于容器刚创建时做了哪些改变

选项：无

示例：



```
$ sudo docker diff registry
C /root
A /root/.bash_history
A /test
```

## exec [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-exec)

### 接口原型 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-接口原型)



```
rpc Exec(ExecRequest) returns (ExecResponse) {}
```

### 接口描述 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-接口描述)

在容器中执行命令，采用的gRPC通讯方式从CRI服务端获取url，再通过获得的url与websocket服务端建立长连接，实现与容器的交互。

### 注意事项 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-注意事项)

执行一条单独的命令，也能打开终端与容器交互。stdin/stdout/stderr之一必须是真的。如果tty为真，stderr必须是假的。 不支持多路复用，在这种情况下，stdout和stderr的输出将合并为单流。

### 参数 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-参数)



| **参数成员**        | **描述**             |
| ------------------- | -------------------- |
| string container_id | 容器ID               |
| repeated string cmd | 待执行的命令         |
| bool tty            | 是否在TTY中执行命令  |
| bool stdin          | 是否流式标准输入     |
| bool stdout         | 是否流式标准输出     |
| bool stderr         | 是否流式输出标准错误 |

### 返回值 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-返回值)



| **返回值** | **描述**                  |
| ---------- | ------------------------- |
| string url | exec流服务器的完全限定URL |

## export [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-export)

用法：**docker export CONTAINER**

功能：将一个容器的文件系统内容以tar包导出到STDOUT

选项：无

示例：

将名为busybox的容器的内容导出到busybox.tar包中：



```
$ sudo docker export busybox > busybox.tar
$ ls
busybox.tar
```

## inspect [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-inspect)

用法：**docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE...]**

功能：返回一个容器或者镜像的底层信息

选项：

-f, --format="" 按照给定的格式输出信息

-s, --size 若查询类型为容器，显示该容器的总体文件大小

--type 返回指定类型的JSON格式

-t, --time=120 超时时间的秒数，若在该时间内docker inspect未执行成功，则停止等待并立即报错。默认为120秒。

示例：

1. 返回一个容器的信息

   

   ```
   $ sudo docker inspect busybox_test
   [
       {
           "Id": "9fbb8649d5a8b6ae106bb0ac7686c40b3cbd67ec2fd1ab03e0c419a70d755577",
           "Created": "2019-08-28T07:43:51.27745746Z",
           "Path": "bash",
           "Args": [],
           "State": {
               "Status": "running",
               "Running": true,
               "Paused": false,
               "Restarting": false,
               "OOMKilled": false,
               "Dead": false,
               "Pid": 64177,
               "ExitCode": 0,
               "Error": "",
               "StartedAt": "2019-08-28T07:43:53.021226383Z",
               "FinishedAt": "0001-01-01T00:00:00Z"
           },
   ......
   ```

2. 按照给定格式返回一个容器的指定信息，下面的例子返回busybox_test容器IP地址

   

   ```
   $ sudo docker inspect -f {{.NetworkSettings.IPAddress}} busybox_test
   172.17.0.91
   ```

## logs [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-logs)

用法：**docker logs [OPTIONS] CONTAINER**

功能：抓取容器内的日志信息，容器可以是运行状态的也可以是停止状态的

选项：

-f, --follow=false 实时打印日志信息

-t, --timestamps=false 显示日志的时间戳

--since 显示指定时间之后的日志

--tail="all" 设置显示的行数，默认显示所有

示例：

1. 查看jaegertracing容器的日志信息，该容器上跑了一个jaegertracing服务

   

   ```
   $ sudo docker logs jaegertracing
   {"level":"info","ts":1566979103.3696961,"caller":"healthcheck/handler.go:99","msg":"Health Check server started","http-port":14269,"status":"unavailable"}
   {"level":"info","ts":1566979103.3820567,"caller":"memory/factory.go:55","msg":"Memory storage configuration","configuration":{"MaxTraces":0}}
   {"level":"info","ts":1566979103.390773,"caller":"tchannel/builder.go:94","msg":"Enabling service discovery","service":"jaeger-collector"}
   {"level":"info","ts":1566979103.3908608,"caller":"peerlistmgr/peer_list_mgr.go:111","msg":"Registering active peer","peer":"127.0.0.1:14267"}
   {"level":"info","ts":1566979103.3922884,"caller":"all-in-one/main.go:186","msg":"Starting agent"}
   {"level":"info","ts":1566979103.4047635,"caller":"all-in-one/main.go:226","msg":"Starting jaeger-collector TChannel server","port":14267}
   {"level":"info","ts":1566979103.404901,"caller":"all-in-one/main.go:236","msg":"Starting jaeger-collector HTTP server","http-port":14268}
   {"level":"info","ts":1566979103.4577134,"caller":"all-in-one/main.go:256","msg":"Listening for Zipkin HTTP traffic","zipkin.http-port":9411}
   ```

2. 加上-f选项，实时打印jaegertracing容器的日志信息

   

   ```
   $ sudo docker logs -f jaegertracing
   {"level":"info","ts":1566979103.3696961,"caller":"healthcheck/handler.go:99","msg":"Health Check server started","http-port":14269,"status":"unavailable"}
   {"level":"info","ts":1566979103.3820567,"caller":"memory/factory.go:55","msg":"Memory storage configuration","configuration":{"MaxTraces":0}}
   {"level":"info","ts":1566979103.390773,"caller":"tchannel/builder.go:94","msg":"Enabling service discovery","service":"jaeger-collector"}
   {"level":"info","ts":1566979103.3908608,"caller":"peerlistmgr/peer_list_mgr.go:111","msg":"Registering active peer","peer":"127.0.0.1:14267"}
   {"level":"info","ts":1566979103.3922884,"caller":"all-in-one/main.go:186","msg":"Starting agent"}
   ```

## pause-unpause [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-pause-unpause)

用法：**docker pause CONTAINER**

**docker unpause CONTAINER**

功能：这两个命令是配对使用的，docker pause暂停容器内的所有进程，docker unpause恢复暂停的进程

选项：无

示例：

本示例将演示一个跑了docker registry（docker镜像服务）服务的容器，当使用docker pause 命令暂停这个容器的进程后，使用curl命令访问该registry服务将阻塞，使用docker unpause命令将恢复registry服务，可以用curl命令访问。

1. 启动一个registry容器

   

   ```
   $ sudo docker run -d --name pause_test -p 5000:5000 registry
   ```

   此时可以用curl命令访问这个服务，请求状态码会返回200 OK。

   

   ```
   $ sudo curl -v 127.0.0.1:5000
   ```

2. 暂停这个容器内的进程

   

   ```
   $ sudo docker pause pause_test
   ```

   此时用curl命令访问这个服务将阻塞，等待服务开启。

3. 恢复运行这个容器内的进程

   

   ```
   $ sudo docker unpause pause_test
   ```

   此时步骤2中的curl访问将恢复运行，请求状态码返回200 OK。

## port [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-port)

用法：**docker port CONTAINER [PRIVATE_PORT[/PROTO]]**

功能：列出容器的端口映射，或者查找指定端口在主机的哪个端口

选项：无

示例：

1. 列出容器所有的端口映射

   

   ```
   $ sudo docker port registry
   5000/tcp -> 0.0.0.0.：5000
   ```

2. 查找容器指定端口的映射

   

   ```
   $ sudo docker port registry 5000
   0.0.0.0.：5000
   ```

## ps [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-ps)

用法：**docker ps [OPTIONS]**

功能：根据不同的选项列出不同状态的容器，在不加任何选项的情况下，将列出正在运行的容器

选项：

-a, --all=false 显示所用的容器

-f, --filter=[] 筛选值，可用的筛选值有：exited=<int>容器的退出码status=(restarting|running|paused|exited)容器的状态码（e.g. -f status=running，列出正在运行的容器）

-l, --latest=false 列出最近创建的一个容器

-n=-1 列出最近n次创建的容器

--no-trunc=false 将64位的容器ID全部显示出来，默认显示12位容器的ID

-q, --quiet=false 显示容器的ID

-s, --size=false 显示容器的大小

示例：

1. 列出正在运行的容器

   

   ```
   $ sudo docker ps
   ```

2. 列出所有的容器

   

   ```
   $ sudo docker ps -a
   ```

## rename [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-rename)

用法：**docker rename OLD_NAME NEW_NAME**

功能：重命名容器

示例：

示例中，用docker run创建并启动一个容器，使用docker rename对容器重命名，并查看容器名是否改变。



```
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b15976967abb        busybox:latest        "bash"              3 seconds ago       Up 2 seconds                            festive_morse
$ sudo docker rename pedantic_euler new_name
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b15976967abb        busybox:latest        "bash"              34 seconds ago      Up 33 seconds                           new_name
```

## restart [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-restart)

用法：**docker restart [OPTIONS] CONTAINER [CONTAINER...]**

功能：重启一个运行中的容器

选项：

-t, --time=10 在杀掉容器之前等待容器停止的秒数，如果容器已停止，就重启。默认为10秒。

示例：



```
$ sudo docker restart busybox
```

> ![NOTE]说明
> 容器在restart过程中，如果容器内存在D状态或Z状态的进程，可能会导致容器重启失败，这需要进一步分析导致容器内进程D状态或Z状态的原因，待容器内进程D状态或Z状态解除后，再进行容器restart操作。

## rm [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-rm)

用法：**docker rm [OPTIONS] CONTAINER [CONTAINER...]**

功能：删除一个或多个容器

选项：

-f, --force=false 强制删除运行中的容器

-l, --link=false 删除指定的链接，而不是底层容器

-v, --volumes=false 删除与容器关联的卷

示例：

1. 删除一个停止运行的容器

   

   ```
   $ sudo docker rm test
   ```

2. 删除一个正在运行的容器

   

   ```
   $ sudo docker rm -f rm_test
   ```

## run [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-run)

用法：**docker run [OPTIONS] IMAGE [COMMAND] [ARG...]**

功能：该命令将由指定的image（如果指定的IMAGE不存在，则从官方镜像库中下载一个镜像）创建一个容器，并启动这个容器，并在容器中执行指定的命令。该命令集成了docker create命令、docker start命令、docker exec命令。

选项：（该命令的选项与docker create命令的选项一样，请参考docker create命令选项，仅仅多了以下两个选项）

--rm=false 设置容器退出时自动删除容器

-v 挂载本地目录或匿名卷到容器内。注意：当将本地目录以带有selinux的安全标签的方式挂载到容器内的同时，尽量不要同时做该本地目录的增删操作，否则该安全标签可能不生效

--sig-proxy=true 发往进程信号的代理，SIGCHLD, SIGSTOP, SIGKILL不使用代理

示例：

使用busybox镜像运行一个容器，在容器启动后执行/bin/sh



```
$ sudo docker run -ti busybox /bin/sh
```

## start [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-start)

用法：**docker start [OPTIONS] CONTAINER [CONTAINER...]**

功能：启动一个或多个未运行容器

选项：

-a, --attach=false 容器的标准输出和错误输出附加到host的STDOUT和STDERR上

-i, --interactive=false 容器的标准输入附加到host的STDIN上

示例：

启动一个名为busybox的容器，添加-i -a选项附加标准输入输出，容器启动后直接进入容器内部，输入exit可以退出容器。

如果启动容器时不加-i -a选项，容器将在后台启动。



```
$ sudo docker start -i -a busybox
```

## stats [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-stats)

用法：**docker stats [OPTIONS] [CONTAINER...]**

功能：持续监控并显示指定容器（若不指定，则默认全部容器）的资源占用情况

选项：

-a, --all 显示所有容器（默认仅显示运行状态的容器）

--no-stream 只显示第一次的结果，不持续监控

示例：

示例中，用docker run创建并启动一个容器，docker stats将输出容器的资源占用情况。



```
$ sudo docker stats
CONTAINER ID        NAME                    CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
2e242bcdd682        jaeger                  0.00%               77.08MiB / 125.8GiB   0.06%               42B / 1.23kB        97.9MB / 0B         38
02a06be42b2c        relaxed_chandrasekhar   0.01%               8.609MiB / 125.8GiB   0.01%               0B / 0B             0B / 0B             10
deb9e49fdef1        hardcore_montalcini     0.01%               12.79MiB / 125.8GiB   0.01%               0B / 0B             0B / 0B             9
```

## stop [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-stop)

用法：**docker stop [OPTIONS] CONTAINER [CONTAINER...]**

功能：通过向容器发送一个SIGTERM信号并在一定的时间后发送一个SIGKILL信号停止容器

选项：

-t, --time=10 在杀掉容器之前等待容器退出的秒数，默认为10S

示例：



```
$ sudo docker stop -t=15 busybox
```

## top [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-top)

用法：**docker top CONTAINER [ps OPTIONS]**

功能：显示一个容器内运行的进程

选项：无

示例：

先运行了一个名为top_test的容器，并在其中执行了top指令



```
$ sudo docker top top_test
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                70045               70028               0                   15:52               pts/0               00:00:00            bash
```

显示的PID是容器内的进程在主机中的PID号。

## update [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-update)

用法：**docker update [OPTIONS] CONTAINER [CONTAINER...]**

功能：热变更一个或多个容器配置。

选项：

**表 3** 参数说明



| 参数                 | 参数含义                                                     |
| :------------------- | :----------------------------------------------------------- |
| --accel=[]           | 设置容器加速器，可设置一个或多个                             |
| --blkio-weight       | 设置容器blockio的相对权重，从10到1000                        |
| --cpu-shares         | 设置容器获得主机CPU的相对权重，通过设置这个选项获得更高的优先级，默认所有的容器都是获得相同的CPU优先权。 |
| --cpu-period         | 设置CFS（完全公平调度策略）进程的CPU周期。缺省值为100ms；一般--cpu-period参数和--cpu-quota是配合使用的，比如--cpu-period=50000 --cpu-quota=25000，意味着如果有1个CPU，该容器可以每50ms获取到50%的CPU。 |
| --cpu-quota          | 设置CFS(完全公平调度策略)进程的CPU配额，默认为0，即没有限制  |
| --cpuset-cpus        | 设置容器中进程允许运行的CPU (0-3, 0,1)。默认没有限制         |
| --cpuset-mems        | 设置容器中进程运行运行的内存内存节点 (0-3, 0,1)，只对NUMA系统起作用 |
| --kernel-memory=""   | 设置容器的kernerl内存限制，格式<number><optional unit>, 其中 unit = b, k, m or g |
| -m, --memory=""      | 设置容器的内存限制，格式<number><optional unit>, 其中 unit = b, k, m or g。该参数最小值为4m。 |
| --memory-reservation | 设置容器内存限制，默认与--memory一致。可认为--memory是硬限制，--memory-reservation是软限制；当使用内存超过预设值时，会动态调整（系统回收内存时尝试将使用内存降低到预设值以下），但不确保一定不超过预设值。一般可以和--memory一起使用，数值小于--memory的预设值。 |
| --memory-swap        | 设置普通内存和交换分区的使用总量，-1为不做限制。如果不设置，则为--memory值的2倍，即SWAP可再使用与--memory相同的内存量。 |
| --restart=""         | 设置容器退出时候的重启规则，当前1.3.1版本支持3个规则：no：当容器停止时，不重启。on-failure：当容器退出码为非0时重启容器，这个规则可以附加最大重启次数，如on-failure:5，最多重启5次。always：无论退出码是什么都退出。 |
| --help               | 打印help信息                                                 |

示例：

变更一个容器名为busybox的cpu和mem配置，包括容器获得主机CPU的相对权重值为512，容器中进程允许运行的CPU核心为0,1,2,3，容器运行内存限制为512m。



```
$ sudo docker update  --cpu-shares 512  --cpuset-cpus=0,3 --memory 512m ubuntu
```

## wait [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/container_management_2.html#user-content-wait)

用法：**docker wait CONTAINER [CONTAINER...]**

功能：等待一个容器停止，并打印出容器的退出码

选项：无

示例：

先开启一个名为busybox的容器



```
$ sudo docker start -i -a busybox
```

执行docker wait



```
$ sudo docker wait busybox
0
```

将阻塞等待busybox容器的退出，退出busybox容器后将看到打印退出码“0”。

# 镜像管理 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-镜像管理)

## build [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-build)

用法：**docker build [OPTIONS] PATH | URL | -**

功能：使用指定路径中的Dockerfile生成构建一个新的image

选项：常用选项参数如下，更多选项可以查看docker help build

**表 4** 参数说明



| 参数              | 参数含义                                   |
| :---------------- | :----------------------------------------- |
| --force-rm=false  | 即使没有构建成功也删除构建过程中生成的容器 |
| --no-cache=false  | 构建image的过程中不使用缓存                |
| -q, --quiet=false | 禁止构建过程中产生的冗余信息               |
| --rm=true         | 构建成功后删除过程中生成的容器             |
| -t, --tag=""      | 指定构建生成的image的tag名                 |
| --build-arg=[]    | 设置构建参数                               |
| --label=[]        | 镜像相关参数设置，各参数意义与create类似   |
| --isolation       | 指定容器的隔离方法                         |
| --pull            | 构建时总是尝试获取最新版本镜像             |

Dockerfile介绍：

Dockerfile是一个镜像的表示，可以通过Dockerfile来描述构建镜像的步骤，并自动构建一个容器，所有的 Dockerfile 命令格式都是：**INSTRUCTION arguments**

**FROM命令**

格式：FROM <image> 或 FROM <image>:<tag>

功能：该命令指定基本镜像，是所有Dockerfile文件的第一个命令，如果没有指定基本镜像的tag，使用默认tag名latest。

**RUN命令**

格式：RUN <command> (the command is run in a shell - `/bin/sh -c`) 或者

RUN ["executable", "param1", "param2" ... ] (exec form)

功能：RUN命令会在上面FROM指定的镜像里执行指定的任何命令，然后提交(commit)结果，提交的镜像会在后面继续用到。RUN命令等价于:

docker run image command

docker commit container_id

**注释**

使用#注释

**MAINTAINER命令**

格式：MAINTAINER <name>

功能：命令用来指定维护者的姓名和联系方式

**ENTRYPOINT命令**

格式：ENTRYPOINT cmd param1 param2 ... 或者ENTRYPOINT ["cmd", "param1", "param2"...]

功能：设置在容器启动时执行命令

**USER命令**

格式：USER name

功能：指定 memcached 的运行用户

**EXPOSE命令**

格式：EXPOSE <port> [<port>...]

功能：开放镜像的一个或多个端口

**ENV命令**

格式：ENV <key> <value>

功能：设置环境变量，设置了后，后续的RUN命令都可以使用

**ADD命令**

格式：ADD <src> <dst>

功能：从src复制文件到container的dest路径，<src> 是相对被构建的源目录的相对路径，可以是文件或目录的路径，也可以是一个远程的文件url，<dest> 是container中的绝对路径

**VOLUME命令**

格式：VOLUME ["<mountpoint>"]

功能：创建一个挂载点用于共享目录

**WORKDIR命令**

格式：workdir <path>

功能：配置RUN, CMD, ENTRYPOINT 命令设置当前工作路径可以设置多次，如果是相对路径，则相对前一个 WORKDIR 命令

**CMD命令**

格式：CMD ["executable","param1","param2"] (like an exec, preferred form)

CMD ["param1","param2"] (as default parameters to ENTRYPOINT)

CMD command param1 param2 (as a shell)

功能：一个Dockerfile里只能有一个CMD，如果有多个，只有最后一个生效

**ONBUILD命令**

格式：ONBUILD [其他指令]

功能：后面跟其他指令，比如 RUN、COPY 等，这些指令，在当前镜像构建时并不会被执行，只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行

下面是Dockerfile的一个完整例子，该Dockerfile将构建一个安装了sshd服务的image



```
FROM busybox
ENV  http_proxy http://192.168.0.226:3128
ENV  https_proxy https://192.168.0.226:3128
RUN apt-get update && apt-get install -y openssh-server
RUN mkdir -p /var/run/sshd
EXPOSE 22
ENTRYPOINT /usr/sbin/sshd -D
```

示例：

1. 以上文的Dockerfile构建一个image

   

   ```
   $ sudo docker build -t busybox:latest
   ```

2. 通过以下命令可以看到这个生成的image：

   

   ```
   docker images | grep busybox
   ```

## history [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-history)

用法：**docker history [OPTIONS] IMAGE**

功能：显示一个image的变化历史

选项：

-H, --human=true

--no-trunc=false 不对输出进行删减

-q, --quiet=false 只显示ID

示例：



```
$ sudo docker history busybox:test
IMAGE               CREATED             CREATED BY          SIZE                COMMENT
be4672959e8b        15 minutes ago      bash                23B
21970dfada48        4 weeks ago                             128MB               Imported from -
```

## images [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-images)

用法：**docker images [OPTIONS] [NAME]**

功能：列出存在的image，不加选项时不显示中间的image

选项：

-a, --all=false 显示所有的镜像，

-f, --filter=[] 指定一个过滤值(i.e. 'dangling=true')

--no-trunc=false 不对输出进行删减

-q, --quiet=false 只显示ID

示例：



```
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              e02e811dd08f        2 years ago         1.09MB
```

## import [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-import)

用法：**docker import URL|- [REPOSITORY[:TAG]]**

功能：把包含了一个rootfs的tar包导入为镜像。与docker export相对应。

选项：无

示例：

从上文介绍的docker export命令时导出的busybox.tar用docker import命令生成一个新的image



```
$ sudo docker import busybox.tar busybox:test
sha256:a79d8ae1240388fd3f6c49697733c8bac4d87283920defc51fb0fe4469e30a4f
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             test                a79d8ae12403        2 seconds ago       1.3MB
```

## load [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-load)

用法：**docker load [OPTIONS]**

功能：把docker save出来的tar包重新加载一个镜像。与docker save相对应。

选项：

-i, --input=""

示例：



```
$ sudo docker load -i busybox.tar
Loaded image ID: sha256:e02e811dd08fd49e7f6032625495118e63f597eb150403d02e3238af1df240ba
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              e02e811dd08f        2 years ago         1.09MB
```

## login [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-login)

用法：**docker login [OPTIONS] [SERVER]**

功能：登录到一个镜像服务库，没有指定server时，默认登录到https://index.docker.io/v1/

选项：

-e, --email="" Email

-p, --password="" 密码

-u, --username="" 用户名

示例：



```
$ sudo docker login
```

## logout [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-logout)

用法：**docker logout [SERVER]**

功能：从一个镜像服务器中登出，没有指定server时，默认登出https://index.docker.io/v1/

选项：无

示例：



```
$ sudo docker logout
```

## pull [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-pull)

用法：**docker pull [OPTIONS] NAME[:TAG]**

功能：从一个镜像库（官方的或私有的）中拉取一个镜像

选项：

-a, --all-tags=false 下载一个镜像仓库的所有镜像（一个镜像仓库可以被打多个标签，比如一个busybox镜像库，可能有多个标签如busybox:14.04,busybox:13.10,busybox:latest等，使用-a选项后，将所有标签的busybox镜像拉取下来）

示例：

1. 从官方镜像库中拉取nginx镜像

   

   ```
   $ sudo docker pull nginx
   Using default tag: latest
   latest: Pulling from official/nginx
   94ed0c431eb5: Pull complete
   9406c100a1c3: Pull complete
   aa74daafd50c: Pull complete
   Digest: sha256:788fa27763db6d69ad3444e8ba72f947df9e7e163bad7c1f5614f8fd27a311c3
   Status: Downloaded newer image for nginx:latest
   ```

   拉取镜像时会检测所依赖的层是否存在，如果存在就用本地的层。

2. 从私有镜像库中拉取镜像

   从私有镜像库中拉取Fedora镜像，比如所使用的私有镜像库的地址是192.168.1.110:5000：

   

   ```
   $ sudo docker pull 192.168.1.110:5000/fedora
   ```

## push [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-push)

用法：**docker push NAME[:TAG]**

功能：将一个image推送到镜像库中

选项：无

示例：

1. 将一个image推送到私有镜像库192.168.1.110:5000中

2. 将要推送的镜像打标签（docker tag命令将在下文介绍），本例中要推送的镜像为busybox:sshd

   

   ```
   $ sudo docker tag ubuntu:sshd 192.168.1.110:5000/busybox:sshd
   ```

3. 将打好标签的镜像推送到私有镜像库中

   

   ```
   $ sudo docker push 192.168.1.110:5000/busybox:sshd
   ```

   推送的时候会自动检测所依赖的层在镜像库中是否已存在，如果以存在，跳过该层。

## rmi [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-rmi)

用法：**docker rmi [OPTIONS] IMAGE [IMAGE...]**

功能：删除一个或多个镜像，如果一个镜像在镜像库中有多个标签，删除镜像的时候只是进行untag操作，当删除的是只有一个标签的镜像时，将依次删除所依赖的层。

选项：

-f, --force=false 强制删除image

--no-prune=false 不删除没有标签的父镜像

示例：



```
$ sudo docker rmi 192.168.1.110:5000/busybox:sshd
```

## save [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-save)

用法：**docker save [OPTIONS] IMAGE [IMAGE...]**

功能：保存一个image到一个tar包，输出默认是到STDOUT

选项：

-o, --output="" 输出到文件中而不是STDOUT

示例：



```
$ sudo docker save -o nginx.tar nginx:latest
$ ls
nginx.tar
```

## search [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-search)

用法：**docker search [OPTIONS] TERM**

功能：在镜像库中查找特定的镜像

选项：

--automated=false 显示自动构建的image

--no-trunc=false 不对输出进行删减

-s, --stars=0 只显示特定星级以上的image

示例：

1. 在官方镜像库中搜寻nginx

   

   ```
   $ sudo docker search nginx
   NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
   nginx                             Official build of Nginx.                       11873               [OK]
   jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   1645                                    [OK]
   richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   739                                     [OK]
   linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   74
   bitnami/nginx                     Bitnami nginx Docker Image                      70                                      [OK]
   tiangolo/nginx-rtmp               Docker image with Nginx using the nginx-rtmp…   51                                      [OK]
   ```

2. 在私有镜像库中搜寻busybox，在私有镜像库中搜寻时要加上私有镜像库的地址

   

   ```
   $ sudo docker search 192.168.1.110:5000/busybox
   ```

## tag [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/image_management_2.html#user-content-tag)

用法：**docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]**

功能：将一个镜像打标签到一个库中

选项：

-f, --force=false 如果存在相同的tag名将强制替换原来的image

示例：



```
$ sudo docker tag busybox:latest busybox:test
```

# 统计信息 [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/statistics.html#user-content-统计信息)

## events [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/statistics.html#user-content-events)

用法：**docker events [OPTIONS]**

功能：从docker daemon中获取实时事件

选项：

--since="" 显示指定时间戳之后的事件

--until="" 显示直到指定时间戳的事件

示例：

该示例中，执行docker events后，用docker run创建并启动一个容器，docker events将输出create事件和start事件。



```
$ sudo docker events
2019-08-28T16:23:09.338838795+08:00 container create 53450588a20800d8231aa1dc4439a734e16955387efb5f259c47737dba9e2b5e (image=busybox:latest, name=eager_wu)
2019-08-28T16:23:09.339909205+08:00 container attach 53450588a20800d8231aa1dc4439a734e16955387efb5f259c47737dba9e2b5e (image=busybox:latest, name=eager_wu)
2019-08-28T16:23:09.397717518+08:00 network connect e2e20f52662f1ee2b01545da3b02e5ec7ff9c85adf688dce89a9eb73661dedaa (container=53450588a20800d8231aa1dc4439a734e16955387efb5f259c47737dba9e2b5e, name=bridge, type=bridge)
2019-08-28T16:23:09.922224724+08:00 container start 53450588a20800d8231aa1dc4439a734e16955387efb5f259c47737dba9e2b5e (image=busybox:latest, name=eager_wu)
2019-08-28T16:23:09.924121158+08:00 container resize 53450588a20800d8231aa1dc4439a734e16955387efb5f259c47737dba9e2b5e (height=48, image=busybox:latest, name=eager_wu, width=210)
```

## info [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/statistics.html#user-content-info)

用法：**docker info**

功能：显示docker系统级的相关信息，包括系统中的Container数量、Image数量、Image的存储驱动、容器的执行驱动、内核版本、主机操作系统版本等信息。

选项：无

示例：



```
$ sudo docker info
Containers: 4
 Running: 3
 Paused: 0
 Stopped: 1
Images: 45
Server Version: 18.09.0
Storage Driver: devicemapper
 Pool Name: docker-thinpool
 Pool Blocksize: 524.3kB
 Base Device Size: 10.74GB
 Backing Filesystem: ext4
 Udev Sync Supported: true
 Data Space Used: 11GB
 Data Space Total: 51GB
 Data Space Available: 39.99GB
 Metadata Space Used: 5.083MB
 Metadata Space Total: 532.7MB
 Metadata Space Available: 527.6MB
 Thin Pool Minimum Free Space: 5.1GB
 Deferred Removal Enabled: true
 Deferred Deletion Enabled: true
 Deferred Deleted Device Count: 0
......
```

## version [](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/cloud/container_engine/docker_engine/statistics.html#user-content-version)

用法：**docker version**

功能：显示docker的版本信息，包括Client版本、Server版本、Go版本、OS/Arch等信息

选项：无

示例：

```
$ sudo docker version
Client:
 Version:           18.09.0
 EulerVersion:      18.09.0.48
 API version:       1.39
 Go version:        go1.11
 Git commit:        cbf6283
 Built:             Mon Apr  1 00:00:00 2019
 OS/Arch:           linux/arm64
 Experimental:      false

Server:
 Engine:
  Version:          18.09.0
  EulerVersion:     18.09.0.48
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.11
  Git commit:       cbf6283
  Built:            Mon Apr  1 00:00:00 2019
  OS/Arch:          linux/arm64
  Experimental:     false
```


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/4190747/  

