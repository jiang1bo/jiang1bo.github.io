# Openeuler安全加固指南


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

# 操作系统加固概述 

介绍对openEuler系统进行加固的目的和加固方案。

## 须知

由于安全加固对系统至关重要，因此只有root用户允许修改并应用安全加固策略。

## 加固目的 

操作系统作为信息系统的核心，承担着管理硬件资源和软件资源的重任，是整个信息系统安全的基础。操作系统之上的各种应用，要想获得信息的完整性、机密性、可用性和可控性，必须依赖于操作系统。脱离了对操作系统的安全保护，仅依靠其他层面的防护手段来阻止黑客和病毒等对网络信息系统的攻击，是无法满足安全需求的。

因此，需要对操作系统进行安全加固，构建动态、完整的安全体系，增强产品的安全性，提升产品的竞争力。

## 加固方案 

本章描述openEuler的安全加固方案，包括加固方式和加固内容。

### 加固方式 

用户可以通过手动修改加固配置或执行相关命令对系统进行加固，也可以通过加固工具批量修改加固项。openEuler的安全加固工具security-tool以openEuler-security.service服务的形式运行。系统首次启动时会自动运行该服务去执行默认加固策略，服务运行后会将该服务自动设置为后续开机不启动。

用户可以通过修改security.conf，使用安全加固工具实现个性化安全加固的效果。

### 加固内容 

openEuler系统加固内容主要分为以下5个部分：

- 系统服务
- 文件权限
- 内核参数
- 授权认证
- 帐号口令

## 加固影响 

对文件权限、帐户口令等安全加固，可能造成用户使用习惯变更，从而影响系统的易用性。影响系统易用性的常见加固项请参见[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/os_hardening_overview.html#zh-cn_topic_0152100325_ta4a48f54ff2849ada7845e2380209917)。

**表 1** 加固影响说明

| **加固项**                  | **建议加固**                                                 | **易用性影响**                                               | **openEuler默认是否设置了该加固项** |
| :-------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------- |
| 字符界面等待超时限制        | 当字符界面长时间处在空闲状态，字符界面会自动退出。说明：当用户通过SSH登录，超时时间由/etc/profile文件的TMOUT字段和/etc/ssh/sshd_config文件的ClientAliveInterval字段两个值中较小的值决定。建议加固为300秒。 | 用户长时间不操作字符界面，字符界面会自动退出。               | 否                                  |
| 口令复杂度限制              | 口令长度最小为8位，口令至少包含大写字母、小写字母、数字和特殊字符中的3种。 | 系统中所有用户不能设置简单的口令，口令必须符合复杂度要求。   | 否                                  |
| 限定登录失败时的尝试次数    | 当用户登录系统时，口令连续输错3次，帐户将被锁定60秒，锁定期间不能登录系统。 | 用户不能随意登录系统，帐户被锁定后必须等待60秒。             | 是                                  |
| 用户默认umask值限制         | 设置所有用户的默认umask值为077，使用户创建文件的默认权限为600、目录权限为700。 | 用户需要按照需求修改指定文件或目录的权限。                   | 否                                  |
| 口令有效期                  | 口令有效期的设置通过修改/etc/login.defs文件实现，加固默认值为口令最大有效期90天，两次修改口令的最小间隔时间为0，口令过期前开始提示天数为7。 | 口令过期后用户重新登录时，提示口令过期并强制要求修改，不修改则无法进入系统。 | 否                                  |
| su权限限制                  | su命令用于在不同帐户之间切换。为了增强系统安全性，有必要对su命令的使用权进行控制，只允许root和wheel群组的帐户使用su命令，限制其他帐户使用。 | 普通帐户执行su命令失败，必须加入wheel群组才可以su成功。      | 是                                  |
| 禁止root帐户直接SSH登录系统 | 设置/etc/ssh/sshd_config文件的PermitRootLogin字段的值为no，用户无法使用root帐户直接SSH登录系统。 | 用户需要先使用普通帐户SSH登录后，再切换至root帐户。          | 否                                  |
| SSH强加密算法               | SSH服务的MACs和Ciphers配置，禁止对CBC、MD5、SHA1算法的支持，修改为CTR、SHA2算法。 | 当前 Windows下使用的部分低版本的Xshell、PuTTY不支持aes128-ctr、aes192-ctr、aes256-ctr、hmac-sha2-256、hmac-sha2-512算法，可能会出现无法通过SSH登录系统的情况，请使用最新的PuTTY（0.63版本以上）、Xshell（5.0版本及以上版本）登录。 | 是                                  |

# 账户口令 

## 屏蔽系统账户 

### 说明 

除了用户账户外，其他账号称为系统账户。系统账户仅系统内部使用，禁止用于登录系统或其他操作，因此屏蔽系统账户。

### 实现 

将系统账户的Shell修改为/sbin/nologin。

```
usermod -L -s /sbin/nologin $systemaccount
```

说明：

$systemaccount 指系统账户。

## 限制使用su命令的账户 

### 说明 

su命令用于在不同账户之间切换。为了增强系统安全性，有必要对su命令的使用权进行控制，只允许root和wheel群组的账户使用su命令，限制其他账户使用。

### 实现

su命令的使用控制通过修改/etc/pam.d/su文件实现，配置如下：

```
auth         required      pam_wheel.so use_uid
```

**表 1** pam_wheel.so配置项说明

| **配置项** | **说明**            |
| :--------- | :------------------ |
| use_uid    | 基于当前账户的uid。 |

## 设置口令复杂度 

### 说明 

用户可以通过修改对应配置文件设置口令的复杂度要求，建议用户根据实际情况设置口令复杂度。

### 实现 

口令复杂度通过/etc/pam.d/password-auth和/etc/pam.d/system-auth文件中的pam_pwquality.so和pam_pwhistory.so模块实现。用户可以通过修改这两个模块中的配置项修改口令复杂度。

### 设置举例 

这里给出一个配置口令复杂度的例子，供用户参考。

**密码复杂度要求**

1. 口令长度至少8个字符。

2. 口令必须包含如下至少3种字符的组合：

   －至少一个小写字母

   －至少一个大写字母

   －至少一个数字

   －至少一个特殊字符：`~!@#$%^&*()-_=+\|[{}];:'",<.>/?和空格

3. 口令不能和账号或者账号的倒写一样。

4. 不能修改为过去5次使用过的旧口令。

**配置实现**

在/etc/pam.d/password-auth和/etc/pam.d/system-auth文件中password配置项的前两行添加如下配置内容：



```
password    requisite     pam_pwquality.so minlen=8 minclass=3 enforce_for_root try_first_pass local_users_only retry=3 dcredit=0 ucredit=0 lcredit=0 ocredit=0 
password    required      pam_pwhistory.so use_authtok remember=5 enforce_for_root
```

**配置项说明**

pam_pwquality.so和pam_pwhistory.so的配置项请分别参见[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/account_passwords.html#table201221044172117)和[表3](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/account_passwords.html#table1212544452120)。

**表 2** pam_pwquality.so配置项说明



| **配置项**       | **说明**                                                  |
| :--------------- | :-------------------------------------------------------- |
| minlen=8         | 口令长度至少包含8个字符                                   |
| minclass=3       | 口令至少包含大写字母、小写字母、数字和特殊字符中的任意3种 |
| ucredit=0        | 口令包含任意个大写字母                                    |
| lcredit=0        | 口令包含任意个小写字母                                    |
| dcredit=0        | 口令包含任意个数字                                        |
| ocredit=0        | 口令包含任意个特殊字符                                    |
| retry=3          | 每次修改最多可以尝试3次                                   |
| enforce_for_root | 本设置对root账户同样有效                                  |

**表 3** pam_pwhistory.so配置项说明



| **配置项**       | **说明**                            |
| :--------------- | :---------------------------------- |
| remember=5       | 口令不能修改为过去5次使用过的旧口令 |
| enforce_for_root | 本设置对root账户同样有效            |

## 设置口令有效期

### 说明

出于系统安全性考虑，建议设置口令有效期限，且口令到期前通知用户更改口令。

### 实现 

口令有效期的设置通过修改/etc/login.defs文件实现，加固项如[表7](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/account_passwords.html#zh-cn_topic_0152100281_t77b5d0753721450c81911c18b74e82eb)所示。表中所有的加固项都在文件/etc/login.defs中。表中字段直接通过修改配置文件完成。

**表 4** login.defs配置项说明所示



| **加固项**    | **加固项说明**             | **建议加固** | **openEuler默认是否已加固为建议值** |
| :------------ | :------------------------- | :----------- | :---------------------------------- |
| PASS_MAX_DAYS | 口令最大有效期             | 90           | 否                                  |
| PASS_MIN_DAYS | 两次修改口令的最小间隔时间 | 0            | 否                                  |
| PASS_WARN_AGE | 口令过期前开始提示天数     | 7            | 否                                  |

说明：

login.defs是设置用户账号限制的文件，可配置口令的最大过期天数、最大长度约束等。该文件里的配置对root用户无效。如果/etc/shadow文件里有相同的选项，则以/etc/shadow配置为准，即/etc/shadow的配置优先级高于/etc/login.defs。口令过期后用户重新登录时，提示口令过期并强制要求修改，不修改则无法进入系统。

## 设置口令的加密算法

### 说明 

出于系统安全考虑，口令不允许明文存储在系统中，应该加密保护。在不需要还原口令的场景，必须使用不可逆算法加密。设置口令的加密算法为sha512，openEuler默认已设置。通过上述设置可以有效防范口令泄露，保证口令安全。

### 实现 

口令的加密算法设置通过修改/etc/pam.d/password-auth和/etc/pam.d/system-auth文件实现，添加如下配置：

```
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
```

**表 5** pam_unix.so配置项说明

| **配置项** | **说明**                   |
| :--------- | :------------------------- |
| sha512     | 使用sha512算法对口令加密。 |

## 登录失败超过三次后锁定 

### 说明 

为了保障用户系统的安全，建议用户设置口令出错次数的阈值（建议3次），以及由于口令出错次数超过阈值，导致账户被锁定的自动解锁时间（建议300秒）。

用户锁定期间，任何输入被判定为无效，锁定时间不因用户的再次输入而重新计时；解锁后，用户的错误输入记录被清空。通过上述设置可以有效防范口令被暴力破解，增强系统的安全性。

说明：

openEuler默认口令出错次数的阈值为3次，系统被锁定后自动解锁时间为60秒。

### 实现 

口令复杂度的设置通过修改/etc/pam.d/password-auth和/etc/pam.d/system-auth文件实现，设置口令的最大出错次数为3次，系统锁定后的解锁时间为300秒的配置如下：



```
auth        required      pam_faillock.so preauth audit deny=3 even_deny_root unlock_time=300
auth        [default=die] pam_faillock.so authfail audit deny=3 even_deny_root unlock_time=300
auth        sufficient    pam_faillock.so authsucc audit deny=3 even_deny_root unlock_time=300
```

**表 6** pam_faillock.so配置项说明



| **配置项**      | **说明**                                 |
| :-------------- | :--------------------------------------- |
| authfail        | 捕获用户登录失败的事件。                 |
| deny=3          | 用户连续登录失败次数超过3次即被锁定。    |
| unlock_time=300 | 普通用户自动解锁时间为300秒（即5分钟）。 |
| even_deny_root  | 同样限制root账户。                       |

## 加固su命令 

### 说明 

为了增强系统安全性，防止使用“su”切换用户时将当前用户环境变量带入其他环境，openEuler默认已做配置。总是在使用su切换用户时初始化PATH。

### 实现 

通过修改/etc/login.defs实现，配置如下：

```
ALWAYS_SET_PATH=yes
```

# 授权认证

## 设置网络远程登录的警告信息 

### 说明 

设置网络远程登录的警告信息，用于在登录进入系统之前向用户提示警告信息，明示非法侵入系统可能受到的惩罚，吓阻潜在的攻击者。同时也可以隐藏系统架构及其他系统信息，避免招致对系统的目标性攻击。

### 实现 

该设置可以通过修改/etc/issue.net文件的内容实现。将/etc/issue.net文件原有内容替换为如下信息（openEuler默认已设置）：

```
Authorized users only. All activities may be monitored and reported.
```

# 禁止通过Ctrl+Alt+Del重启系统 

### 说明 

操作系统默认能够通过“Ctrl+Alt+Del”进行重启，建议禁止该项特性，防止因为误操作而导致数据丢失。

### 实现 

禁止通过“Ctrl+Alt+Del”重启系统的操作步骤如下：

1. 删除两个ctrl-alt-del.target文件，参考命令如下：

   ```
   rm -f /etc/systemd/system/ctrl-alt-del.target
   rm -f /usr/lib/systemd/system/ctrl-alt-del.target
   ```
   
2. 修改/etc/systemd/system.conf文件，将#CtrlAltDelBurstAction=reboot-force修改为CtrlAltDelBurstAction=none。

3. 重启systemd，使修改生效，参考命令如下： （注：使用 systemctl daemon-reexec 命令可能会导致短暂的系统服务不可用或重启，并且必须具有 root 或具有 sudo 权限的用户才能执行这个操作）

   ```
   systemctl daemon-reexec
   ```

## 设置终端的自动退出时间

### 说明

无人看管的终端容易被侦听或被攻击，可能会危及系统安全。因此建议设置终端在停止运行一段时间后能够自动退出。

### 实现 

自动退出时间由/etc/profile文件的TMOUT字段（单位为秒）控制，在/etc/profile的尾部添加如下配置：

```
export TMOUT=300
```

## 设置用户的默认umask值为077 

### 说明 

umask值用于为用户新创建的文件和目录设置缺省权限。如果umask的值设置过小，会使群组用户或其他用户的权限过大，给系统带来安全威胁。因此设置所有用户默认的umask值为0077，即用户创建的目录默认权限为700，文件的默认权限为600。umask值代表的是权限的“补码”，umask值和权限的换算方法请参见"附录 > umask值含义"。

说明：

openEuler默认已设置用户的默认umask值为022。

### 实现 

1. 分别在/etc/bashrc文件和/etc/profile.d/目录下的所有文件中加入“umask 0077”。

   ```
echo "umask 0077" >> $FILE
   ```
   
   说明：

   *$FILE* 为具体的文件名，例如：echo "umask 0077" >> /etc/bashrc

2. 设置/etc/bashrc文件和/etc/profile.d/目录下所有文件的属主为root，群组为root。

   ```
   chown root:root $FILE
   ```
   
   说明：

   *$FILE* 为具体的文件名，例如：chown root:root /etc/bashrc

## 设置GRUB2加密口令 

### 说明 

GRUB是GRand Unified Bootloader的缩写，它是一个操作系统启动管理器，用来引导不同系统（如Windows、Linux），GRUB2是GRUB的升级版。

系统启动时，可以通过GRUB2界面修改系统的启动参数。为了确保系统的启动参数不被任意修改，需要对GRUB2界面进行加密，仅在输入正确的GRUB2口令时才能修改启动参数。

说明：

GRUB2默认设置的口令为openEuler#12，建议用户首次登录时修改默认密码并定期更新，避免密码泄露后，启动选项被篡改，导致系统启动异常。

### 实现 

1. 使用grub2-mkpasswd-pbkdf2命令生成加密的口令：

   说明：

   GRUB2加密算法使用PBKDF2。

   ```
   # grub2-mkpasswd-pbkdf2
   Enter password: 
   Reenter password: 
   PBKDF2 hash of your password is 
   grub.pbkdf2.sha512.10000.5A45748D892672FDA02DD3B6F7AE390AC6E6D532A600D4AC477D25C7D087644697D8A0894DFED9D86DC2A27F4E01D925C46417A225FC099C12DBD3D7D49A7425.2BD2F5BF4907DCC389CC5D165DB85CC3E2C94C8F9A30B01DACAA9CD552B731BA1DD3B7CC2C765704D55B8CD962D2AEF19A753CBE9B8464E2B1EB39A3BB4EAB08
   ```
   
   说明：

   在Enter password和Reenter password输入相同的口令。
   grub.pbkdf2.sha512.10000.5A45748D892672FDA02DD3B6F7AE390AC6E6D532A600D4AC477D25C7D087644697D8A0894DFED9D86DC2A27F4E01D925C46417A225FC099C12DBD3D7D49A7425.2BD2F5BF4907DCC389CC5D165DB85CC3E2C94C8F9A30B01DACAA9CD552B731BA1DD3B7CC2C765704D55B8CD962D2AEF19A753CBE9B8464E2B1EB39A3BB4EAB08为openEuler#12经过grub2-mkpasswd-pbkdf2加密后的输出，每次输出的密文不同。
   
2. 使用vi工具打开/boot/efi/EFI/openEuler/grub.cfg文件（不同模式下grub.cfg文件所在路径不同，详见说明），并在开头位置追加如下字段：

   ```
   set superusers="root"
   password_pbkdf2 root grub.pbkdf2.sha512.10000.5A45748D892672FDA02DD3B6F7AE390AC6E6D532A600D4AC477D25C7D087644697D8A0894DFED9D86DC2A27F4E01D925C46417A225FC099C12DBD3D7D49A7425.2BD2F5BF4907DCC389CC5D165DB85CC3E2C94C8F9A30B01DACAA9CD552B731BA1DD3B7CC2C765704D55B8CD962D2AEF19A753CBE9B8464E2B1EB39A3BB4EAB08
   ```
   
   说明：

   

   - 不同模式下grub.cfg文件所在路径不同：x86架构的UEFI模式下路径为/boot/efi/EFI/openEuler/grub.cfg，legacy BIOS模式下路径为/boot/grub2/grub.cfg；aarch64架构下路径为/boot/efi/EFI/openEuler/grub.cfg。
   - superusers字段用于设置GRUB2的超级管理员的帐户名。
   - password_pbkdf2字段后的参数，第1个参数为GRUB2的帐户名，第2个为该帐户的加密口令。

## 安全单用户模式 

### 说明 

单用户模式是以root权限进入系统，如不设置密码，将存在较大安全隐患。

### 实现 

该设置可以通过修改/etc/sysconfig/init文件内容实现。将SINGLE选项配置为SINGLE=/sbin/sulogin。

## 禁止交互式启动 

### 说明

使用交互式引导，控制台用户可以禁用审计、防火墙或其他服务，削弱了系统安全性。用户可以禁止使用交互式引导，提升安全性。openEuler默认已禁止。

### 实现 

该设置可以通过修改/etc/sysconfig/init文件内容实现。将PROMPT选项配置为PROMPT=no。

# 系统服务 

## 加固SSH服务 

### 说明 

SSH（Secure Shell）是目前较可靠，专为远程登录会话和其他网络服务提供安全性保障的协议。利用SSH协议可以有效防止远程管理过程中的信息泄露问题。通过SSH可以对所有传输的数据进行加密，并防止DNS欺骗和IP欺骗。OpenSSH是SSH协议的免费开源实现。

加固SSH服务，是指修改SSH服务中的配置来设置系统使用OpenSSH协议时的算法、认证等参数，从而提高系统的安全性。[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/system_services.html#zh-cn_topic_0152100390_ta2fdb8e4931b4c1a8f502b3c7d887b95)中详细说明了各加固项含义、建议加固值及其默认策略。

### 实现 

服务端加固操作如下：

1. 打开服务端SSH服务的配置文件/etc/ssh/sshd_config，在该文件中修改或添加对应加固项及其加固值。

2. 保存/etc/ssh/sshd_config文件。

3. 重启SSH服务，命令如下：

   ```
   # systemctl restart sshd
   ```

客户端加固操作如下：

1. 打开客户端SSH服务的配置文件/etc/ssh/ssh_config，在该文件中修改或添加对应加固项及其加固值。

2. 保存/etc/ssh/ssh_config文件。

3. 重启SSH服务，命令如下：

   ```
   # systemctl restart sshd
   ```

### 加固项说明 

- 服务端加固策略

  SSH服务的所有加固项均保存在配置文件/etc/ssh/sshd_config中，服务端各加固项的含义、加固建议以及openEuler默认是否已经加固为建议加固值请参见[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/system_services.html#zh-cn_topic_0152100390_ta2fdb8e4931b4c1a8f502b3c7d887b95)。

  **表 1** SSH服务端加固项说明

  

  | **加固项**                                      | **加固项说明**                                               | **加固建议**                                                 | openEuler默认是否已加固为建议值 |
  | :---------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------------ |
  | Protocol                                        | 设置使用SSH协议的版本                                        | 2                                                            | 是                              |
  | SyslogFacility                                  | 设置SSH服务的日志类型。加固策略将其设置为“AUTH”，即认证类日志 | AUTH                                                         | 是                              |
  | LogLevel                                        | 设置记录sshd日志消息的层次                                   | VERBOSE                                                      | 是                              |
  | X11Forwarding                                   | 设置使用SSH登录后，能否使用图形化界面                        | no                                                           | 是                              |
  | MaxAuthTries                                    | 最大认证尝试次数                                             | 3                                                            | 否                              |
  | PubkeyAuthentication                            | 设置是否允许公钥认证。                                       | yes                                                          | 是                              |
  | RSAAuthentication                               | 设置是否允许只有RSA安全验证                                  | yes                                                          | 是                              |
  | IgnoreRhosts                                    | 设置是否使用rhosts文件和shosts文件进行验证。rhosts文件和shosts文件用于记录可以访问远程计算机的计算机名及关联的登录名 | yes                                                          | 是                              |
  | RhostsRSAAuthentication                         | 设置是否使用基于rhosts的RSA算法安全验证。rhosts文件记录可以访问远程计算机的计算机名及关联的登录名 | no                                                           | 是                              |
  | HostbasedAuthentication                         | 设置是否使用基于主机的验证。基于主机的验证是指已信任客户机上的任何用户都可以使用SSH连接 | no                                                           | 是                              |
  | PermitRootLogin                                 | 是否允许root帐户直接使用SSH登录系统说明：若需要直接使用root帐户通过SSH登录系统，请修改/etc/ssh/sshd_config文件的PermitRootLogin字段的值为yes。 | no                                                           | 否                              |
  | PermitEmptyPasswords                            | 设置是否允许用口令为空的帐号登录                             | no                                                           | 是                              |
  | PermitUserEnvironment                           | 设置是否解析 ~/.ssh/environment和~/.ssh/authorized_keys中设定的环境变量 | no                                                           | 是                              |
  | Ciphers                                         | 设置SSH数据传输的加密算法                                    | aes128-ctr,aes192-ctr,aes256-ctr,chacha20-poly1305@openssh.com,aes128-gcm@openssh.com,aes256-gcm@openssh.com | 是                              |
  | ClientAliveCountMax                             | 设置超时次数。服务器发出请求后，客户端没有响应的次数达到一定值，连接自动断开 | 0                                                            | 否                              |
  | Banner                                          | 指定登录SSH前后显示的提示信息的文件                          | /etc/issue.net                                               | 是                              |
  | MACs                                            | 设置SSH数据校验的哈希算法                                    | hmac-sha2-512,hmac-sha2-512-etm@openssh.com,hmac-sha2-256,hmac-sha2-256-etm@openssh.com | 是                              |
  | StrictModes                                     | 设置SSH在接收登录请求之前是否检查用户HOME目录和rhosts文件的权限和所有权 | yes                                                          | 是                              |
  | UsePAM                                          | 使用PAM登录认证                                              | yes                                                          | 是                              |
  | AllowTcpForwarding                              | 设置是否允许TCP转发                                          | no                                                           | 是                              |
  | Subsystem sftp /usr/libexec/openssh/sftp-server | sftp日志记录级别，记录INFO级别以及认证日志。                 | -l INFO -f AUTH                                              | 是                              |
  | AllowAgentForwarding                            | 设置是否允许SSH Agent转发                                    | no                                                           | 是                              |
  | GatewayPorts                                    | 设置是否允许连接到转发客户端端口                             | no                                                           | 是                              |
  | PermitTunnel                                    | Tunnel设备是否允许使用                                       | no                                                           | 是                              |
  | KexAlgorithms                                   | 设置SSH密钥交换算法                                          | curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256 |                                 |
  | LoginGraceTime                                  | 限制用户必须在指定的时限内认证成功，0 表示无限制。缺省值是 60 秒。 | 60                                                           | 否                              |

  说明：

  默认情况下，登录SSH前后显示的提示信息保存在/etc/issue.net文件中，/etc/issue.net默认信息为“Authorized users only. All activities may be monitored and reported.”。

- 客户端加固策略

  SSH服务的所有加固项均保存在配置文件/etc/ssh/ssh_config中，客户端各加固项的含义、加固建议以及openEuler默认是否已经加固为建议加固值请参见[表2](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/system_services.html#zh-cn_topic_0152100390_tb289c5a6f1c7420ab4339187f9018ea4)。

  **表 2** SSH客户端加固项说明

  

  | **加固项**       | **加固项说明**                          | **加固建议**                                                 | openEuler默认是否已加固为建议值 |
  | :--------------- | :-------------------------------------- | :----------------------------------------------------------- | :------------------------------ |
  | KexAlgorithms    | 设置SSH密钥交换算法                     | ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256 | 否                              |
  | VerifyHostKeyDNS | 是否使用DNS或者SSHFP资源记录验证HostKey | ask                                                          | 否                              |

  说明：

  对于使用dh算法进行密钥交换的第三方客户端和服务端工具，要求允许建立连接的最低长度为2048bits。

### 其他安全建议

- SSH服务仅侦听指定IP地址

  出于安全考虑，建议用户在使用SSH服务时，仅在必需的IP上进行绑定侦听，而不是侦听0.0.0.0，可修改/etc/ssh/sshd_config文件中的ListenAddress配置项。

  1. 打开并修改/etc/ssh/sshd_config文件

     

     ```
     # vi /etc/ssh/sshd_config
     ```

     修改内容如下，表示绑定侦听IP为 *192.168.1.100*，用户可根据实际情况修改需要侦听的IP

     

     ```
     ...
     ListenAddress 192.168.1.100
     ...
     ```

  2. 重启SSH服务

     

     ```
     # systemctl restart sshd.service
     ```

- 限制SFTP用户向上跨目录访问

  SFTP是FTP over SSH的安全FTP协议，对于访问SFTP的用户建议使用专用帐号，只能上传或下载文件，不能用于SSH登录，同时对SFTP可以访问的目录进行限定，防止目录遍历攻击，具体配置如下：

  说明：

  sftpgroup为示例用户组，sftpuser为示例用户名。

  1. 创建SFTP用户组

     

     ```
     # groupadd sftpgroup
     ```

  2. 创建SFTP根目录

     

     ```
     # mkdir /sftp
     ```

  3. 修改SFTP根目录属主和权限

     

     ```
     # chown root:root /sftp
     # chmod 755 /sftp
     ```

  4. 创建SFTP用户

     

     ```
     # useradd -g sftpgroup -s /sbin/nologin sftpuser
     ```

  5. 设置SFTP用户的口令

     

     ```
     # passwd sftpuser
     ```

  6. 创建SFTP用户上传目录

     

     ```
     # mkdir /sftp/sftpuser
     ```

  7. 修改SFTP用户上传目录属主和权限

     

     ```
     # chown root:root /sftp/sftpuser
     # chmod 777 /sftp/sftpuser
     ```

  8. 修改/etc/ssh/sshd_config文件

     

     ```
     # vi /etc/ssh/sshd_config
     ```

     修改内容如下：

     

     ```
     #Subsystem sftp /usr/libexec/openssh/sftp-server -l INFO -f AUTH
     Subsystem sftp internal-sftp -l INFO -f AUTH
     ...
      
     Match Group sftpgroup                  
         ChrootDirectory /sftp/%u
         ForceCommand internal-sftp
     ```

     说明：

     

     - %u代表当前sftp用户的用户名，这是一个通配符，用户原样输入即可。

     - 以下内容必须加在/etc/ssh/sshd_config文件的末尾。

       

       ```
         Match Group sftpgroup                    
           ChrootDirectory /sftp/%u  
           ForceCommand internal-sftp
       ```

  9. 重启SSH服务

     

     ```
     # systemctl restart sshd.service
     ```

- SSH远程执行命令

  OpenSSH通用机制，在远程执行命令时，默认不开启tty，如果执行需要密码的命令，密码会明文回显。出于安全考虑，建议用户增加-t选项，确保密码输入安全。如下：

  

  ```
  # ssh -t testuser@192.168.1.100 su
  ```

  说明：

  192.168.1.100为示例IP，testuser为示例用户。

# 文件权限 

## 设置文件的权限和属主 

### 说明

Linux将所有对象都当作文件来处理，即使一个目录也被看作是包含有多个其他文件的大文件。因此，Linux中最重要的就是文件和目录的安全性。文件和目录的安全性主要通过权限和属主来保证。

openEuler默认对系统中的常用目录、可执行文件和配置文件设置了权限和属主。

### 实现 

以/bin目录为例，修改文件权限和文件属主的操作如下：

- 修改文件权限。例如将/bin目录权限设置为755。

  ```
  chmod 755 /bin
  ```
  
- 修改文件属主。例如将/bin目录的拥有者和群组设置为root:root。

  ```
  chown root:root /bin
  ```

## 删除无主文件

### 说明 

系统管理员在删除用户/群组时，存在着忘记删除该用户/该群组所拥有文件的问题。如果后续新创建的用户/群组与被删除的用户/群组同名，则新用户/新群组会拥有部分不属于其权限的文件，建议将此类文件删除。

### 实现 

删除用户ID不存在的文件

1. 查找用户ID不存在的文件。

   ```
   find / -nouser
   ```
   
2. 删除查找到的文件。其中 filename 为用户ID不存在文件的文件名。

   ```
   rm -f filename
   ```

删除群组ID不存在的文件

1. 查找群组ID不存在的文件。

   ```
   find / -nogroup
   ```
   
2. 删除查找到的文件。其中 filename 为群组ID不存在文件的文件名。

   ```
   rm -f filename
   ```

## 处理空链接文件 

### 说明 

无指向的空链接文件，可能会被恶意用户利用，影响系统安全性。建议用户删除无效的空链接文件，提高系统安全性。

### 特殊场景 

openEuler系统安装完成后，可能存在空链接文件，这些空链接文件可能有对应用途（有些空链接文件是预制的，会被其他组件依赖）。请用户根据实际环境进行处理，处理方式请参见[实现](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/file_permissions.html#zh-cn_topic_0152100319_s1b24647cdd834a8eaca3032611baf072)。

例如，openEuler支持UEFI和legacy BIOS两种安装模式，两种引导场景支持的grub相关包默认都安装，当用户选择legacy BIOS模式安装时，形成空链接文件“/etc/grub2-efi.cfg”；当用户选择UEFI模式安装时，会形成空链接文件“/etc/grub2.cfg”，需要用户根据实际情况处理空链接。

### 实现

1. 通过如下命令查找系统中的空链接文件。

   ```
find dirname -type l -follow 2>/dev/null
   ```
   
   说明：

   dirname为搜索目录的名称，通常需要关注系统关键目录：/bin、/boot、/usr、/lib64、/lib、/var等。

2. 如果此类文件无实际作用，可通过如下命令删除。

   ```
   rm -f filename
   ```
   
   说明：

   filename为[步骤1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/file_permissions.html#zh-cn_topic_0152100319_l4dc74664c4fb400aaf91fb314c4f9da6)找出的文件名。

## 设置守护进程的umask值 

### 说明 

umask值用来为新创建的文件和目录设置缺省权限。如果没有设定umask值，则生成的文件具有全局可写权限，存在一定的风险。守护进程负责系统上某个服务，让系统可以接受来自用户或者是网络客户的要求。为了提高守护进程所创建文件和目录的安全性，建议设置其umask值为0027。umask值代表的是权限的“补码”，umask值和权限的换算方法请参见 "[附录 > umask值含义](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/附录.md/#umask值含义)" 。

说明：

openEuler默认已设置守护进程的umask值为0022。

### 实现

在配置文件/etc/sysconfig/init中新增一行：umask 0027。

## 为全局可写目录添加粘滞位属性 

### 说明 

任意用户可以删除、修改全局可写目录中的文件和目录，为了确保全局可写目录中的文件和目录不会被任意删除，需要为全局可写目录添加粘滞位属性。

### 实现

1. 搜索全局可写目录。

   ```
   find / -type d -perm -0002 ! -perm -1000 -ls | grep -v proc
   ```
   
2. 为全局可写目录添加粘滞位属性。dirname 为实际查找到的目录名。

   

   ```
   chmod +t dirname
   ```

## 删除非授权文件的全局可写属性

### 说明 

全局可写文件可被系统中的任意用户修改，影响系统完整性。

### 实现 

1. 列举系统中所有的全局可写文件。

   

   ```
   find / -type d ( -perm -o+w ) | grep -v proc
   find / -type f ( -perm -o+w ) | grep -v proc
   ```

2. 查看步骤1列举的所有文件(粘滞位的文件和目录可以排除在外)，删除文件或去掉其全局可写权限。使用以下命令去掉权限，其中filename为对应文件名：

   

   ```
   chmod o-w  filename
   ```

   说明：

   可通过如下命令确定对应文件或目录是否设置了粘滞位，若回显中包含T标记，则为粘滞位文件或目录。命令中的filename为需要查询文件或目录的名称。

> ```
> ls -l filename
> ```

## 限制at命令的使用权限 

### 说明 

at命令用于创建在指定时间自动执行的任务。为避免任意用户通过at命令安排工作，造成系统易受攻击，需要指定可使用该命令的用户。

### 实现 

1. 删除/etc/at.deny文件。

   ```
   rm -f /etc/at.deny
   ```
   
2. 创建/etc/at.allow文件并将/etc/at.allow的文件属主改为root:root。

   ```
   touch /etc/at.allow
   chown root:root /etc/at.allow
   ```
   
3. 控制/etc/at.allow的文件权限，仅root可操作。

   ```
   chmod og-rwx /etc/at.allow
   ```

## 限制cron命令的使用权限 

### 说明

cron命令用于创建例行性任务。为避免任意用户通过cron命令安排工作，造成系统易受攻击，需要指定可使用该命令的用户。

### 实现

1. 删除/etc/cron.deny文件。

   ```
   rm -f /etc/cron.deny
   ```
   
2. 创建/etc/cron.allow文件并将/etc/cron.allow的文件属主改为root:root。

   ```
   touch /etc/cron.allow
   chown root:root /etc/cron.allow
   ```
   
3. 控制/etc/cron.allow的文件权限，仅root可操作。

   ```
   chmod og-rwx /etc/cron.allow
   ```

## 限制sudo命令的使用权限 

### 说明

sudo命令用于普通用户以root权限执行命令。为了增强系统安全性，有必要对sudo命令的使用权进行控制，只允许root使用sudo命令，限制其他帐户使用。openEuler默认未限制非root用户使用sudo命令的权限。

### 实现 

sudo命令的使用控制通过修改/etc/sudoers文件实现，需要注释掉如下配置行：

```
#%wheel ALL=(ALL)       ALL
```

# 内核参数 

## 加固内核参数 

### 说明 

内核参数决定配置和应用特权的状态。内核提供用户可配置的系统控制，这一系统控制可微调或配置，该功能特性可通过控制各种可配置的内核参数，来提高操作系统的安全特性。比如：通过微调或配置网络选项，可有效提高系统的安全性。

### 实现 

1. 将[表1](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/kernel_parameters.html#zh-cn_topic_0152100187_t69b5423c26644b26abe94d88d38878eb)中的加固项写入/etc/sysctl.conf文件中。

   说明：

   写入方式如下：

   ```
   net.ipv4.icmp_echo_ignore_broadcasts = 1  
   net.ipv4.conf.all.rp_filter = 1  
   net.ipv4.conf.default.rp_filter = 1
   ```
   
   **表 1** 内核参数加固策略说明

   

   | **加固项**                                 | **加固项说明**                                               | **加固建议** | **openEuler默认是否已加固为建议值** |
   | :----------------------------------------- | :----------------------------------------------------------- | :----------- | :---------------------------------- |
   | net.ipv4.icmp_echo_ignore_broadcasts       | 是否接受ICMP广播报文。加固策略为不接受。                     | 1            | 是                                  |
   | net.ipv4.conf.all.rp_filter                | 验证数据包使用的实际源地址是否与路由表相关，以及使用该特定源IP地址的数据包是否通过接口获取其响应。加固策略为启用该项。 | 1            | 是                                  |
   | net.ipv4.conf.default.rp_filter            | 1                                                            | 是           |                                     |
   | net.ipv4.ip_forward                        | IP Forwarding可阻止未授权的IP数据包渗透至网络。加固策略为禁用该特性。 | 0            | 是                                  |
   | net.ipv4.conf.all.accept_source_route      | accept_source_route指允许数据包的发送者指定数据包的发送路径，以及返回给发送者的数据包所走的路径。加固策略为禁用该特性。 | 0            | 是                                  |
   | net.ipv4.conf.default.accept_source_route  | 0                                                            | 是           |                                     |
   | net.ipv4.conf.all.accept_redirects         | 是否发送ICMP重定向报文。加固策略为禁止发送。                 | 0            | 是                                  |
   | net.ipv4.conf.default.accept_redirects     | 0                                                            | 是           |                                     |
   | net.ipv6.conf.all.accept_redirects         | 0                                                            | 是           |                                     |
   | net.ipv6.conf.default.accept_redirects     | 0                                                            | 是           |                                     |
   | net.ipv4.conf.all.send_redirects           | 是否将ICMP重定向报文发送至其他主机。只有当主机作为路由时，应启用该策略。加固策略为禁用该项。 | 0            | 是                                  |
   | net.ipv4.conf.default.send_redirects       | 0                                                            | 是           |                                     |
   | net.ipv4.icmp_ignore_bogus_error_responses | 忽略伪造的ICMP数据包，不会将其记录到日志，将节省大量的硬盘空间。加固策略为启用该项。 | 1            | 是                                  |
   | net.ipv4.tcp_syncookies                    | SYN Attack是一种通过占用系统资源迫使系统重启的DoS攻击。加固策略为开启TCP-SYN cookie保护。 | 1            | 是                                  |
   | kernel.dmesg_restrict                      | 加固dmesg信息，仅允许管理员查看。                            | 1            | 是                                  |
   | kernel.sched_autogroup_enabled             | 该选项决定内核是否对线程进行自动分组调度。开启后调度组之间互相竞争时间片，调度组内的线程再竞争调度组分配到的时间片。加固策略为不启用该项。 | 0            | 否                                  |
   | kernel.sysrq                               | 禁用魔术键。说明：建议禁用魔术键，避免由于直接发送命令到内核对系统造成影响，增强内核安全性。 | 0            | 是                                  |
   | net.ipv4.conf.all.secure_redirects         | 设置系统是接收来自任何主机的ICMP重定向消息还是从默认网关列表中的网关处接收ICMP重定向消息。加固策略为采用前者。 | 0            | 是                                  |
   | net.ipv4.conf.default.secure_redirects     | 0                                                            | 是           |                                     |
   
2. 加载sysctl.conf文件中设置的内核参数。

   

   ```
   # sysctl -p  /etc/sysctl.conf
   ```

### 其他安全建议 

- net.ipv4.icmp_echo_ignore_all：忽略ICMP请求。

  出于安全考虑，建议开启此项（当前默认值为0，开启将值设为1）。

  但开启后会忽略所有接收到的icmp echo请求的包(会导致机器无法ping通)，建议用户根据实际组网场景决定是否开启此项。

- net.ipv4.conf.all.log_martians/net.ipv4.conf.default.log_martians：对于仿冒/源路由/重定向数据包开启日志记录。

  出于安全考虑，建议开启此项（当前默认值为0，开启将值设为1）。

  但是开启后会记录带有不允许的地址的数据到内核日志中，存在冲日志风险，建议用户根据实际使用场景决定是否开启此项。

- net.ipv4.tcp_timestamps：关闭tcp_timestamps。

  出于安全考虑，建议关闭tcp_timestamps（当前默认值为1，关闭将值设为0）。

  但是关闭此项会影响TCP超时重发的性能，建议用户根据实际使用场景决定是否关闭此项。

- net.ipv4.tcp_max_syn_backlog：决定了SYN_RECV状态队列的数量。

  该参数决定了SYN_RECV状态队列的数量，超过这个数量，系统将不再接受新的TCP连接请求，一定程度上可以防止系统资源耗尽。建议由用户根据实际使用场景配置合适的值。





# SELinux配置 

## 概述 

自主访问控制DAC（Discretionary Access Control）基于用户、组和其他权限，决定一个资源是否能被访问的因素是某个资源是否拥有对应用户的权限，它不能使系统管理员创建全面和细粒度的安全策略。SELinux（Security-Enhanced Linux）是Linux内核的一个模块，也是Linux的一个安全子系统。SELinux的实现了强制访问控制MAC（Mandatory Access Control ），每个进程和系统资源都有一个特殊的安全标签，资源能否被访问除了DAC规定的原则外，还需要判断每一类进程是否拥有对某一类资源的访问权限。

openEuler默认使用SELinux提升系统安全性。SELinux分为三种模式：

- permissive：SELinux仅打印告警而不强制执行。
- enforcing：SELinux安全策略被强制执行。
- disabled：不加载SELinux安全策略。

## 配置说明 

- 获取当前SELinux运行状态：

  

  ```
  # getenforce
  Enforcing
  ```

- SELinux开启的前提下，设置运行状态为enforcing模式：

  

  ```
  # setenforce 1
  # getenforce
  Enforcing
  ```

- SELinux开启的前提下，设置运行状态为permissive模式：

  

  ```
  # setenforce 0
  # getenforce
  Permissive
  ```

- SELinux开启的前提下，设置当前SELinux运行状态为disabled（关闭SELinux，需要重启系统）。

  1. 修改SELinux配置文件/etc/selinux/config，设置“SELINUX=disabled”。

  

  ```
  # cat /etc/selinux/config | grep "SELINUX="
  SELINUX=disabled
  ```

  1. 重启系统：

  

  ```
  # reboot
  ```

  1. 状态切换成功：

  

  ```
  # getenforce
  Disabled
  ```

- SELinux关闭的前提下，设置SELinux运行状态为permissive。

  1. 修改SELinux配置文件/etc/selinux/config，设置“SELINUX=permissive”：

  

  ```
  # cat /etc/selinux/config | grep "SELINUX="
  SELINUX=permissive
  ```

  1. 在根目录下创建.autorelabel文件：

  

  ```
  # touch /.autorelabel
  ```

  1. 重启系统，此时系统会重启两次：

  

  ```
  # reboot
  ```

  1. 状态切换成功：

  

  ```
  # getenforce
  Permissive
  ```

- SELinux关闭的前提下，设置SELinux运行状态为enforcing。

  1. 按照上一步骤所述，设置SELinux运行状态为permissive。
  2. 修改SELinux配置文件/etc/selinux/config，设置“SELINUX=enforcing”：

  

  ```
  # cat /etc/selinux/config | grep "SELINUX="
  SELINUX=enforcing
  ```

  1. 重启系统：

  

  ```
  # reboot
  ```

  1. 状态切换成功：

  

  ```
  # getenforce
  Enforcing
  ```

## SELinux相关命令 

- 查询运行SELinux的系统状态。SELinux status表示SELinux的状态，enabled表示启用SELinux，disabled表示关闭SELinux。Current mode表示SELinux当前的安全策略。

  

  ```
  # sestatus
  SELinux status:                 enabled
  SELinuxfs mount:                /sys/fs/selinux
  SELinux root directory:         /etc/selinux
  Loaded policy name:             targeted
  Current mode:                   enforcing
  Mode from config file:          enforcing
  Policy MLS status:              enabled
  Policy deny_unknown status:     allowed
  Memory protection checking:     actual (secure)
  Max kernel policy version:      31
  ```

## 策略添加 

- 从audit日志中获取并添加缺失策略（需要audit服务开启且audit日志中已经存在SELinux访问拒绝日志）。

  1. 查询audit日志中是否有SELinux访问拒绝日志，其中audit日志的路径视具体情况决定。

  

  ```
  # grep avc /var/log/audit/audit.log*
  ```

  1. 查询缺失规则。

  

  ```
  # audit2allow -a /var/log/audit/audit.log*
  ```

  1. 根据缺失规则生成一个策略模块，命名为demo。

  

  ```
  # audit2allow -a /var/log/audit/audit.log* -M demo
  ******************** IMPORTANT ***********************
  To make this policy package active, execute:
  semodule -i demo.pp
  ```

  1. 加载demo策略模块。

  

  ```
  # semodule -i demo.pp
  ```

- 编写并添加SELinux策略模块。

  1. 编写FC文件（涉及新增文件安全上下文需要编写）。

  

  ```
  # cat demo.fc 
  /usr/bin/example -- system_u:object_r:example_exec_t:s0
  /resource -- system_u:object_r:resource_file_t:s0
  ```

  1. 编写TE文件（仅供参考）。

  

  ```
  # cat demo.te 
  module demo 1.0;
  require
  {
   role unconfined_r;
   role system_r;
   type user_devpts_t;
   type root_t;
   attribute file_type;
   attribute domain;
   class dir { getattr search add_name create open remove_name rmdir setattr write };
   class file { entrypoint execute getattr open read map setattr write create };
   class process { sigchld rlimitinh siginh transition setcap getcap };
   class unix_stream_socket { accept bind connect listen recvfrom sendto listen create lock read write getattr setattr getopt setopt append shutdown ioctl connectto };
   class capability { chown dac_override dac_read_search };
   class chr_file { append getattr ioctl read write };
  };
  role unconfined_r types example_t;
  role system_r types example_t;
  type example_exec_t, file_type;
  type resource_file_t, file_type;
  type example_t, domain;
  allow example_t user_devpts_t : chr_file { append getattr ioctl read write };
  allow example_t file_type : dir { getattr search };
  allow example_t example_exec_t : file { entrypoint execute getattr map open read };
  allow domain example_exec_t : file { execute getattr map open read };
  allow example_t example_exec_t : process { sigchld };
  allow domain example_t : process { rlimitinh siginh transition };
  allow example_t resource_file_t : file { create getattr open read setattr write };
  allow example_t root_t : dir { add_name create getattr open remove_name rmdir search setattr write };
  allow example_t example_t : unix_stream_socket { accept append bind connect create getattr getopt ioctl listen listen lock read recvfrom sendto setattr setopt shutdown write };
  allow example_t domain : unix_stream_socket { connectto };
  allow example_t example_t : capability { chown dac_override dac_read_search };
  allow example_t example_t : process { getcap setcap };
  type_transition domain example_exec_t : process example_t;
  type_transition example_t root_t : file resource_file_t "resource";
  ```

  1. 编译demo.te为demo.mod。

  

  ```
  # checkmodule -Mmo demo.mod demo.te
  ```

  1. 打包demo.mod和demo.fc为策略模块文件。

  

  ```
  semodule_package -m demo.mod -f demo.fc -o demo.pp
  ```

  1. 加载策略模块。

  

  ```
  # semodule -i demo.pp
  ```

  1. 删除加载的策略模块。

  

  ```
  # semodule -r demo
  libsemanage.semanage_direct_remove_key: Removing last demo module (no other demo module exists at another priority).
  ```

## 功能验证 

- SELinux为白名单机制，未配置合理策略的模块可能会由于缺少权限无法正常运行。故对模块进行功能验证并适配合理的规则是很有必要的。

  1. 查看audit服务是否开启。

  

  ```
  # systemctl status auditd
  ```

  1. 设置SELinux模式为permissive(仅打印告警而不强制执行，参考 配置说明 )。

  

  ```
  # getenforce
  Permissive
  ```

  1. 全量跑测试模块的功能用例，查看audit日志中SELinux访问拒绝日志。

  

  ```
  # grep avc /var/log/audit/audit.log*
  ```

  1. 分析访问拒绝日志，并过滤出缺失的合理规则。

  

  ```
  type=AVC msg=audit(1596161643.271:1304): avc: denied { read } for pid=1782603 comm="smbd" name=".viminfo" dev="dm-0" ino=2488208 scontext=system_u:system_r:smbd_t:s0 tcontext=staff_u:object_r:user_home_t:s0 tclass=file permissive=1
  表示进程smbd（安全上下文为system_u:system_r:smbd_t:s0）对文件.viminfo（安全上下文为staff_u:object_r:user_home_t:s0）执行文件读操作被权限拒绝。
  permissive=1表示当前运行的是permissive模式，该日志只记录未执行禁止。
  ```

  1. 参考“策略添加”章节，将缺少的合理规则补全。

## 注意事项 

- 如用户需使能SELinux功能，建议通过dnf升级方式将selinux-policy更新为最新版本，否则应用程序有可能无法正常运行。升级命令示例：

  

  ```
  dnf update selinux-policy -y
  ```

- 如果用户由于SELinux配置不当（如误删策略或未配置合理的规则或安全上下文）导致系统无法启动，可以在启动参数中添加selinux=0，关闭SELinux功能，系统即可正常启动。

- 开启SELinux后，会对访问行为进行权限检查，对操作系统性能会有一定程度（与运行环境访问操作频率相关）的影响。



# 安全加固工具 

## 加固操作 

### 概述 

安全加固工具会根据usr-security.conf设置加固策略，使用加固工具设置加固策略需要用户修改usr-security.conf。本节介绍usr-security.conf的修改规则。用户可配置的加固项请参见[加固指导](https://openeuler.org/zh/docs/21.03/docs/SecHarden/加固指导.html)对应内容。

### 注意事项 

- 修改配置后，需要重启安全加固服务使配置生效。重启方法请参见[加固生效](https://docs.openeuler.openatom.cn/zh/docs/24.03_LTS_SP1/server/security/secharden/security_hardening_tools.html#加固生效)对应内容。
- 用户修改加固配置时，仅修改/etc/openEuler_security/usr-security.conf文件，不建议修改/etc/openEuler_security/security.conf。security.conf中为基本加固项，仅运行一次。
- 当重启安全加固服务使配置生效后，在usr-security.conf中删除对应加固项并重启安全加固服务并不能清除之前已生效的配置。
- 安全加固操作记录在日志文件/var/log/openEuler-security.log中。

### 配置格式 

usr-security.conf中的每一行代表一项配置，根据配置内容的不同有不同配置格式，这里给出各类配置的格式说明。

说明：

- 所有配置项以执行ID开头，执行ID仅为了方便用户识别配置内容，取值为正整数，由用户自行定义。
- 配置项的各内容之间使用@作为分隔符。
- 若实际配置内容中包含@，需要使用@@表示以和分隔符区分，例如实际内容为xxx@yyy，则配置为xxx@@yyy。目前不支持@位于配置内容的开头和结尾。

- d：注释

  格式：执行ID@d@对象文件@匹配项

  功能：将对象文件中以匹配项开头（行首可以有空格）的行注释（在行首添加#）。

  示例：执行ID为401，注释/etc/sudoers文件中以%wheel开头的行。

  

  ```
  401@d@/etc/sudoers@%wheel
  ```

- m：替换

  格式：执行ID@m@对象文件@匹配项@替换目标值

  功能：将对象文件中以匹配项开头（行首可以有空格）的行替换为“匹配项加替换目标值 ”。若匹配行开头有空格，替换后将删除这些空格。

  示例：执行ID为101，将/etc/ssh/sshd_config文件中以Protocol 开头的行替换为Protocol 2。匹配和替换时也会考虑Protocol后的空格。

  

  ```
  101@m@/etc/ssh/sshd_config@Protocol @2
  ```

- sm：精确修改

  格式：执行ID@sm@对象文件@匹配项@替换目标值

  功能：将对象文件中以匹配项开头（行首可以有空格）的行替换为“匹配项加替换目标值 ”。若匹配行开头有空格，替换后将保留这些空格，这是sm和m的区别。

  示例：执行ID为201，将/etc/audit/hzqtest文件中以size开头的行替换为size 2048。

  

  ```
  201@sm@/etc/audit/hzqtest@size@ 2048
  ```

- M：修改子项

  格式：执行ID@M@对象文件@匹配项@匹配子项[@匹配子项的值]

  功能：匹配对象文件中以匹配项开头（行首可以有空格）的行，并将该行中以匹配子项开始的内容替换为“匹配子项和匹配子项的值”，其中匹配子项的值可选。

  示例：执行ID为101，找到file文件中以key开头的行，并将这些行中以key2开始的内容替换为key2value2。

  

  ```
  101@M@file@key@key2@value2
  ```

- systemctl：管理服务

  格式：执行ID@systemctl@对象服务@具体操作

  功能：使用systemctl管理对象服务，具体操作可取值为start、stop、restart、disable等systemctl所有可用的命令。

  示例：执行ID为218，停止cups.service服务，等同于systemctl stop cups.service的配置行。

  

  ```
  218@systemctl@cups.service@stop
  ```

- 其他命令

  格式：执行ID@命令@对象文件

  功能：执行对应命令，即执行命令行“命令 对象文件”。

  示例一：执行ID为402，使用rm -f命令删除文件/etc/pki/ca-trust/extracted/pem/email-ca-bundle.pem。

  

  ```
  402@rm -f @/etc/pki/ca-trust/extracted/pem/email-ca-bundle.pem
  ```

  示例二：执行ID为215，使用touch命令创建文件/etc/cron.allow。

  

  ```
  215@touch @/etc/cron.allow
  ```

  示例三：执行ID为214，使用chown命令将文件/etc/at.allow的属主改为root:root。

  

  ```
  214@chown root:root @/etc/at.allow
  ```

  示例四：执行ID为214，使用chmod命令去除文件/etc/at.allow属主所在群组及其他非属主用户的rwx权限。

  

  ```
  214@chmod og-rwx @/etc/at.allow
  ```

## 加固生效 

完成修改usr-security.conf文件后，请运行如下命令使新添加的配置生效。

```
systemctl restart openEuler-security.service
```

# 安全配置加固工具 

## 前言 

本文档为安全配置加固工具sec_conf基本介绍以及使用说明。

## sec_conf简介 

### 背景和概述 

openEuler已支持多种安全特性，包括Linux原生安全特性和社区自研安全特性，但是存在特性分散，配置难度大，用户学习成本高等问题。同时对于一些具备拦截功能的安全特性（如IMA评估、安全启动、访问控制等），一旦用户配置错误，可能导致系统无法启动或无法正常运行。因此，sec_conf旨在实现自动化安全配置机制，用户可基于工具进行系统的安全检查和加固，以更好地推进openEuler安全特性在各应用场景的落地。

### 功能介绍 

sec_conf是一个帮助管理员配置openEuler安全特性（如IMA、DIM、secure boot等）的安全加固工具。用户可以输入配置信息，即需要实现的安全加固目标，生成相应的安全特性配置脚本。

目前sec_conf支持可配置的安全机制为IMA、DIM、secure boot、modsign，其他安全特性暂未支持。

## 安装与部署 

### 安装依赖软件包 

编译secPaver需要的软件有：

- make
- golang 1.11+

### 下载源码 

```
git clone https://gitee.com/openeuler/secpaver.git -b sec_conf
```

### 编译安装 

```
cd secpaver
make
```

安装软件：

```
make install
```

## 工程文件格式说明 

sec_conf工程文件由策略配置文件、检查脚本模板文件、配置脚本模板文件组成。

### 策略配置文件 

策略配置文件保护DIM\IMA\安全启动\内核模块校验相关特性配置，采用yaml格式表示，各字段说明如下：

| 一级配置项    | 二级配置项   | 类型         | 属性                                                         | 说明                                |
| :------------ | :----------- | :----------- | :----------------------------------------------------------- | :---------------------------------- |
| name          | N/A          | string       | optional                                                     | 配置文件命名                        |
| version       | N/A          | string       | optional                                                     | 配置策略版本号                      |
| dim           | enable       | bool         | optional                                                     | 打开/关闭DIM功能                    |
| measure_list  | string array | optional     | DIM需要度量的文件；用户态文件，需要指定绝对路径；内核模块，需要指定有效的内核模块名称；内核，需要指定为“kernel” |                                     |
| log_cap       | int          | optional     | 度量日志最大条数，当记录的度量日志数量达到参数设置时，停止记录度量日志，默认值为100000 |                                     |
| schedule      | int          | optional     | 度量完一个进程/模块后调度的时间，单位毫秒，设置为0代表不调度，默认值为0 |                                     |
| interval      | int          | optional     | 自动度量周期，单位分钟，设置为0代表不设置自动度量，默认值为0 |                                     |
| hash          | string       | optional     | 度量哈希算法，默认值为sha256                                 |                                     |
| core_pcr      | int          | optional     | 将dim_core度量结果扩展至TPM芯片的PCR寄存器，设置为0代表不扩展（注意需要与芯片实际的PCR编号保持一致），默认值为0 |                                     |
| monitor_pcr   | int          | optional     | 将dim_monitor度量结果扩展至TPM芯片的PCR寄存器，设置为0代表不扩展（注意需要与芯片实际的PCR编号保持一致），默认值为0 |                                     |
| signature     | bool         | optional     | 是否启用策略文件和签名机制                                   |                                     |
| auto_baseline | bool         | optional     | 是否建立DIM基线，若为false，则需管理员手动生成基线           |                                     |
| secure_boot   | enable       | bool         | optional                                                     | 是否使能安全启动                    |
| anti_rollback | bool         | optional     | 打开/关闭安全启动防回滚策略                                  |                                     |
| verbose       | bool         | optional     | 打开/关闭安全启动相关日志                                    |                                     |
| modsign       | enable       | bool         | optional                                                     | 是否使能内核模块校验特性            |
| ima           | measure_list | string array | optional                                                     | IMA保护文件列表（需要指定绝对路径） |
| appraise_list | string array | optional     | IMA评估文件列表（需要指定绝对路径）                          |                                     |

说明：



1. sec_conf.yaml文件必须放在/usr/share/secpaver/scripts/sec_conf/sec_conf.yaml，不可重命名。
2. 参数类型需遵守上述表格要求。
3. 相关配置若不存在，则使用默认值。

### 检查脚本模板、配置脚本模板文件 

模板文件实现利用go-tamplate引擎，将脚本文件与数据结合，生成最终的文本输出。

检查脚本模板统一放置/usr/share/secpaver/scripts/sec_conf/check/目录，该目录下存放DIM、IMA等特性的脚本模板，这些脚本模板不能单独执行，只能通过sec_conf生成最新的脚本去执行openEuler特性检查。

配置脚本模板统一放置/usr/share/secpaver/scripts/sec_conf/gen/目录，该目录下存放DIM、IMA等特性的脚本模板，这些脚本模板不能单独执行，只能通过sec_conf生成最新的脚本去执行openEuler特性配置。

说明：



1. 配置、检查模板文件不可修改，仅用于被sec_conf解析生成脚本。

## 安全配置命令行接口 

|    参数     |                 功能介绍                 |             命令格式             |
| :---------: | :--------------------------------------: | :------------------------------: |
|  --help,-h  |        打印sec_conf命令行帮助信息        |           sec_conf -h            |
|  gen_check  | 生成安全配置检查脚本，并输出到命令行界面 |        sec_conf gen_check        |
| gen_config  |   生成安全配置脚本，并输出到命令行界面   |       sec_conf gen_config        |
| --output,-o |      将生成的配置脚本输出到指定文件      | sec_conf gen_config -o config.sh |

## 使用说明 

配置yaml文件，示例：

```
# cat /usr/share/secpaver/scripts/sec_conf/sec_conf.yaml
---
name: "openEuler security configuration"
version: "1"
dim:
  enable: true
  measure_list:
    - "/usr/bin/bash"
    - "nf_nat"
    - "kernel"
  log_cap: 100000
  schedule: 0
  interval: 0
  hash: "sha256"
  core_pcr: 11
  monitor_pcr: 12
  signature: true
  auto_baseline: true
secure_boot:
  enable: true
  anti_rollback: true
  verbose: true
modsign:
  enable: true
ima:
  measure_list:
  - "/usr/bin/ls"
  - "/usr/bin/cat"
  - "/usr/bin/base64"
  - "/usr/bin/base32"
  appraise_list:
  - "/usr/bin/base64"
  - "/usr/bin/base32"
  - "/usr/bin/sleep"
  - "/usr/bin/date"
...
```

生成特性配置脚本、检查脚本



```
sec_conf gen_config -o ./config.sh
sec_conf gen_check -o ./check.sh
```

执行配置脚本，并检查配置是否正确，若配置正确，则重启系统使配置生效



```
sh ./config.sh -s
sh ./check.sh -s
reboot
```

重启后再次执行配置脚本，并检查配置是否正确，此时预期所有功能检查通过



```
sh ./config.sh -s
sh ./check.sh -s
```





# 附录 

介绍文件权限的含义和umask值的含义。

## 文件和目录权限含义 

Linux系统中文件和目录权限用于限定谁能通过何种方式对文件和目录进行访问和操作。文件和目录的访问权限分为只读，只写和可执行三种。

有三种不同类型的用户可对文件和目录进行访问：

- 文件所有者：文件的创建者。
- 同组用户：与文件所有者在同一个属组的用户。
- 其他用户：与文件所有者不在同一个属组的用户。

文件和目录的权限含义通过以下例子说明：

**假设/usr/src的权限为755，将每位数字转化为二进制后为：111101101，含义如下**：

- **左侧三个bit位111表示文件所有者的权限依次为：可读、可写、可执行。**
- **中间三个bit位101表示同组用户的权限依次为：可读、不可写、可执行。**
- **右侧三个bit位101表示其他用户的权限依次为：可读、不可写、可执行。**

## umask值含义 

当用户新创建文件或目录时，该文件或目录具有一个缺省权限。该缺省权限由umask值来指定。

umask值代表的是权限的“补码”，即用缺省最大权限值减去umask值得到实际权限值。文件的缺省最大权限为可读可写，目录的缺省最大权限为可读可写可执行。即一个文件的实际缺省权限为666减去umask值。目录的实际缺省权限为777减去umask值。


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/7cacc3d/  

