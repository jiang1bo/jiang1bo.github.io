# Ssl自动续签


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

## Certbot 实现ssl申请以及自动续签

### 在 openEuler 服务器安装 Certbot 及阿里云插件

```shell
[root@localhost ~]# dnf install python3 python3-pip -y
OS                                                                                                              7.2 kB/s | 2.2 kB     00:00
everything                                                                                                      8.0 kB/s | 2.3 kB     00:00
EPOL                                                                                                            7.9 kB/s | 2.3 kB     00:00
debuginfo                                                                                                       8.5 kB/s | 2.3 kB     00:00
source                                                                                                          7.9 kB/s | 2.2 kB     00:00
update                                                                                                          8.4 kB/s | 2.3 kB     00:00
update-source                                                                                                   8.1 kB/s | 2.3 kB     00:00
Package python3-3.11.6-18.oe2403sp1.x86_64 is already installed.
Package python3-pip-23.3.1-5.oe2403sp1.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[root@localhost ~]# pip3 install certbot certbot-dns-aliyun
Collecting certbot
  Downloading certbot-5.2.2-py3-none-any.whl.metadata (7.0 kB)
Collecting certbot-dns-aliyun
  Downloading certbot_dns_aliyun-2.0.0-py2.py3-none-any.whl.metadata (1.4 kB)
Collecting acme>=5.2.2 (from certbot)
  Downloading acme-5.2.2-py3-none-any.whl.metadata (1.3 kB)
Collecting ConfigArgParse>=1.5.3 (from certbot)
  Downloading configargparse-1.7.1-py3-none-any.whl.metadata (24 kB)
Requirement already satisfied: configobj>=5.0.6 in /usr/lib/python3.11/site-packages (from certbot) (5.0.8)
Collecting cryptography>=43.0.0 (from certbot)
  Downloading cryptography-46.0.3-cp311-abi3-manylinux_2_34_x86_64.whl.metadata (5.7 kB)
Requirement already satisfied: distro>=1.0.1 in /usr/lib/python3.11/site-packages (from certbot) (1.9.0)
Collecting josepy>=2.0.0 (from certbot)
  Downloading josepy-2.2.0-py3-none-any.whl.metadata (1.7 kB)
Collecting parsedatetime>=2.6 (from certbot)
  Downloading parsedatetime-2.6-py3-none-any.whl.metadata (4.7 kB)
Collecting pyrfc3339 (from certbot)
  Downloading pyrfc3339-2.1.0-py3-none-any.whl.metadata (2.8 kB)
Collecting mock (from certbot-dns-aliyun)
  Downloading mock-5.2.0-py3-none-any.whl.metadata (3.1 kB)
Requirement already satisfied: setuptools in /usr/lib/python3.11/site-packages (from certbot-dns-aliyun) (68.0.0)
Collecting PyOpenSSL>=25.0.0 (from acme>=5.2.2->certbot)
  Downloading pyopenssl-25.3.0-py3-none-any.whl.metadata (17 kB)
Requirement already satisfied: requests>=2.25.1 in /usr/lib/python3.11/site-packages (from acme>=5.2.2->certbot) (2.31.0)
Requirement already satisfied: six in /usr/lib/python3.11/site-packages (from configobj>=5.0.6->certbot) (1.16.0)
Collecting cffi>=2.0.0 (from cryptography>=43.0.0->certbot)
  Downloading cffi-2.0.0-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (2.6 kB)
Collecting pycparser (from cffi>=2.0.0->cryptography>=43.0.0->certbot)
  Downloading pycparser-2.23-py3-none-any.whl.metadata (993 bytes)
Collecting typing-extensions>=4.9 (from PyOpenSSL>=25.0.0->acme>=5.2.2->certbot)
  Downloading typing_extensions-4.15.0-py3-none-any.whl.metadata (3.3 kB)
Requirement already satisfied: charset_normalizer<4,>=2 in /usr/lib/python3.11/site-packages (from requests>=2.25.1->acme>=5.2.2->certbot) (3.3.                                                              2)
Requirement already satisfied: idna<4,>=2.5 in /usr/lib/python3.11/site-packages (from requests>=2.25.1->acme>=5.2.2->certbot) (3.6)
Requirement already satisfied: urllib3<3,>=1.21.1 in /usr/lib/python3.11/site-packages (from requests>=2.25.1->acme>=5.2.2->certbot) (1.26.18)
Downloading certbot-5.2.2-py3-none-any.whl (411 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 412.0/412.0 kB 264.4 kB/s eta 0:00:00
Downloading certbot_dns_aliyun-2.0.0-py2.py3-none-any.whl (11 kB)
Downloading acme-5.2.2-py3-none-any.whl (94 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 94.2/94.2 kB 249.3 kB/s eta 0:00:00
Downloading configargparse-1.7.1-py3-none-any.whl (25 kB)
Downloading cryptography-46.0.3-cp311-abi3-manylinux_2_34_x86_64.whl (4.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 4.5/4.5 MB 403.5 kB/s eta 0:00:00
Downloading josepy-2.2.0-py3-none-any.whl (29 kB)
Downloading parsedatetime-2.6-py3-none-any.whl (42 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 42.5/42.5 kB 1.0 MB/s eta 0:00:00
Downloading mock-5.2.0-py3-none-any.whl (31 kB)
Downloading pyrfc3339-2.1.0-py3-none-any.whl (6.8 kB)
Downloading cffi-2.0.0-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (215 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 215.6/215.6 kB 594.7 kB/s eta 0:00:00
Downloading pyopenssl-25.3.0-py3-none-any.whl (57 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 57.3/57.3 kB 816.2 kB/s eta 0:00:00
Downloading typing_extensions-4.15.0-py3-none-any.whl (44 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 44.6/44.6 kB 668.7 kB/s eta 0:00:00
Downloading pycparser-2.23-py3-none-any.whl (118 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 118.1/118.1 kB 792.9 kB/s eta 0:00:00
Installing collected packages: parsedatetime, typing-extensions, pyrfc3339, pycparser, mock, ConfigArgParse, cffi, cryptography, PyOpenSSL, jose                                                              py, acme, certbot, certbot-dns-aliyun
Successfully installed ConfigArgParse-1.7.1 PyOpenSSL-25.3.0 acme-5.2.2 certbot-5.2.2 certbot-dns-aliyun-2.0.0 cffi-2.0.0 cryptography-46.0.3 jo                                                              sepy-2.2.0 mock-5.2.0 parsedatetime-2.6 pycparser-2.23 pyrfc3339-2.1.0 typing-extensions-4.15.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recomm                                                              ended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

```

### 查看 certbot 主程序安装位置



```shell
# 1. 查看 certbot 主程序安装到哪里了
which certbot
# 通常输出类似：/usr/local/bin/certbot

# 2. 查看 certbot 及其阿里云插件的 Python 模块安装路径
sudo pip3 show certbot certbot-dns-aliyun | grep Location
# 会显示模块的安装路径，通常类似：/usr/local/lib/python3.9/site-packages
#执行 pip3 install certbot certbot-dns-aliyun 后，找不到 /etc/letsencrypt 目录是完全正常的，因为#这个目录是 Certbot 在首次运行时自动创建的，而不是由 pip 安装的。

[root@localhost work]# which certbot
/usr/local/bin/certbot
[root@localhost work]# pip3 show certbot certbot-dns-aliyun | grep Location
Location: /usr/local/lib/python3.11/site-packages
Location: /usr/local/lib/python3.11/site-packages
[root@localhost work]# ll /etc/letsencrypt/
total 4
drwxr-xr-x. 5 root root 4096 Dec 30 01:52 renewal-hooks
```

### 📜 **Certbot 常用命令手册**

Certbot的命令通常遵循 `certbot [子命令] [选项]` 的格式，其中**子命令**决定了核心操作。

| 场景/目的                 | 常用命令示例                                                 | 关键参数与说明                                               |
| :------------------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
| **1. 申请证书（核心）**   | `sudo certbot certonly`                                      | `--webroot`：通过网站根目录验证。 `--standalone`：临时启动服务验证（需停用Web服务器）。 `--manual`：手动进行DNS或文件验证。 |
| **2. 使用DNS插件申请**    | `sudo certbot certonly \` `--dns-cloudflare \` `--dns-cloudflare-credentials ~/cloudflare.ini \` `-d example.com` | `--dns-*`：指定DNS服务商（如`--dns-aliyun`）。 `--dns-*-credentials`：指定API密钥文件路径。 |
| **3. 测试申请（无限制）** | `sudo certbot certonly --dry-run`                            | **`--dry-run`**：完整测试申请流程，**不签发真实证书**，不消耗Let's Encrypt限额。**新配置必测！** |
| **4. 续期所有证书**       | `sudo certbot renew`                                         | 默认检查所有证书，**仅续期到期前30天内的**。常与 `--quiet`（静默模式）搭配用于Cron。 |
| **5. 续期并立即生效**     | `sudo certbot renew --force-renewal`                         | **`--force-renewal`**：强制立即续期，即使未到期。**谨慎使用**，避免触发限流。 |
| **6. 查看证书列表**       | `sudo certbot certificates`                                  | 列出所有由Certbot管理的证书，显示**到期时间**和**存放路径**。 |
| **7. 撤销证书**           | `sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem` | **`--cert-path`**：指定要撤销的证书文件。证书私钥泄露时使用。 |
| **8. 删除证书记录**       | `sudo certbot delete --cert-name example.com`                | 从Certbot记录中移除该证书的管理，**不会删除证书文件**。      |



### 阿里云 API 配置

#### 详细配置步骤（阿里云 DNS）**在阿里云获取 API 密钥**

1. 登录 [阿里云控制台](https://ram.console.aliyun.com/users)，进入 **访问控制 (RAM)**。
2. 在左侧菜单选择 **“用户”** -> **“创建用户”**。
   - 登录名称：例如 `certbot-dns`。
   - 勾选 **“Open API 调用访问”**。
3. 创建完成后，**保存并复制**生成的 **AccessKey ID** 和 **AccessKey Secret**（仅显示一次）。
4. 点击新用户，进入 **“权限管理”** -> **“添加权限”**。
   - 搜索并选择 **`AliyunDNSFullAccess`**（为安全起见，可创建更细粒度权限，但此为最简单授权），完成授权。

#### **在 openEuler 服务器安装 Certbot 及阿里云插件**

```
# 确保已安装 Python3 和 pip
sudo dnf install python3 python3-pip -y

# 使用 pip 安装 Certbot 和阿里云 DNS 插件
sudo pip3 install certbot certbot-dns-aliyun
```

```shell
# 请确保你已经按照之前步骤，创建并正确配置了阿里云 API 密钥文件 (/etc/letsencrypt/aliyun.ini)。即使 letsencrypt 目录还不存在，你也可以先创建这个文件：
sudo mkdir -p /etc/letsencrypt
sudo vim /etc/letsencrypt/aliyun.ini
# （粘贴你的阿里云AccessKey配置并保存）
[root@localhost work]# vim /etc/letsencrypt/aliyun.ini
[root@localhost work]# cat /etc/letsencrypt/aliyun.ini
# 阿里云 API 配置
dns_aliyun_access_key = xxxI5tLLamCaaK6BHaRx2G7K
dns_aliyun_access_key_secret = xxxR9LbdnfvBhFNaJOZTGa5gVdImuS
# 设置严格权限：
[root@localhost work]# chmod 600 /etc/letsencrypt/aliyun.ini

[root@localhost work]# ls -la /etc/letsencrypt
total 24
drwxr-xr-x.  3 root root  4096 Dec 30 02:10 .
drwxr-xr-x. 98 root root 12288 Dec 30 01:52 ..
-rw-------.  1 root root   134 Dec 30 02:10 aliyun.ini
drwxr-xr-x.  5 root root  4096 Dec 30 01:52 renewal-hooks
```

### 使用certbot申请证书

```shell
#1.请确保你已经按照之前步骤，创建并正确配置了阿里云 API 密钥文件 (/etc/letsencrypt/aliyun.ini)。即使 letsencrypt 目录还不存在，你也可以先创建这个文件：

#2.然后，运行你计划用来申请证书的命令（例如申请测试证书，或直接申请正式证书）。以下是一个测试命令，它不会签发真实证书，但会触发创建所有必要的目录和配置文件：

sudo certbot certonly --dry-run \
  --authenticator dns-aliyun \
  --dns-aliyun-credentials /etc/letsencrypt/aliyun.ini \
  -d fermis.top \
  --email your-email@example.com \
  --agree-tos \
  --non-interactive \
  --server https://acme-staging-v02.api.letsencrypt.org/directory
#注意：这里使用了 --dry-run（模拟运行）和 Let‘s Encrypt 的 staging 测试服务器，专用于测试配置是否正确，不会产生签发次数限制。


#执行完上述命令后，无论是否成功，/etc/letsencrypt 目录都会被创建出来。你可以用 ls -la /etc/letsencrypt 命令查看，里面应该已经有了 accounts、archive、live、renewal 等子目录的框架。


#目录创建成功，你就可以移除 --dry-run 和 --server 参数，使用正式的 Let‘s Encrypt 生产服务器命令来申请
#真实的证书了：
#命令关键点说明：
#certonly：只申请而不安装证书，之后可以手动安装到 Nginx 服务器
#--authenticator dns-aliyun: 指定使用 aliyun DNS 插件。
#--dns-aliyun-credentials：指定 aliyun 凭据路径
#--preferred-challenges：指定验证域名所有权的方式为 DNS
#-d 20000915.xyz -d '*.20000915.xyz': 申请包含根域名和所有二级子域名的泛域名证书。
#--non-interactive: 非交互模式，适合脚本自动化。
#--server: 指定使用 Let‘s Encrypt 的生产环境 API（v02）。
#成功后，你会在屏幕上看到祝贺信息，证书文件（fullchain.pem 私钥 privkey.pem 等）保存在 /etc/letsencrypt/live/20000915.xyz/ 目录下。


[root@localhost work]# sudo certbot certonly \
  --authenticator dns-aliyun \
  --dns-aliyun-credentials /etc/letsencrypt/aliyun.ini \
  --preferred-challenges dns-01 \
  -d fermis.top \
  -d '*.fermis.top' \
  --email 3044483124@qq.com \
  --agree-tos \
  --non-interactive \
  --server https://acme-v02.api.letsencrypt.org/directory
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Account registered.
Requesting a certificate for fermis.top and *.fermis.top
Waiting 30 seconds for DNS changes to propagate

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/fermis.top/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/fermis.top/privkey.pem
This certificate expires on 2026-03-29.
These files will be updated when the certificate renews.

NEXT STEPS:
- The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[root@localhost work]# ls -l /etc/letsencrypt/
total 24
drwx------. 3 root root 4096 Dec 30 02:17 accounts
-rw-------. 1 root root  134 Dec 30 02:10 aliyun.ini
drwx------. 3 root root 4096 Dec 30 02:18 archive
drwx------. 3 root root 4096 Dec 30 02:18 live
drwxr-xr-x. 2 root root 4096 Dec 30 02:18 renewal
drwxr-xr-x. 5 root root 4096 Dec 30 01:52 renewal-hooks
```

### **配置自动续期与部署**

1. **测试自动续期流程**：

   ```shell
   sudo certbot renew --dry-run
   
   Saving debug log to /var/log/letsencrypt/letsencrypt.log
   
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   Processing /etc/letsencrypt/renewal/fermis.top.conf
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   Account registered.
   Simulating renewal of an existing certificate for fermis.top and *.fermis.top
   Waiting 30 seconds for DNS changes to propagate
   
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   Congratulations, all simulated renewals succeeded:
     /etc/letsencrypt/live/fermis.top/fullchain.pem (success)
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   
   ```

   

   如果看到 `Congratulations, all renewals succeeded.` 的模拟成功信息，说明配置完全正确。

2. **配置续期后自动执行的钩子脚本**（例如，重启 Nginx 使新证书生效）：

   ```
   sudo vim /etc/letsencrypt/renewal-hooks/deploy/reload-webserver.sh
   ```

   写入以下内容（假设使用 Nginx）：

   ```
   #!/bin/bash
   # 在证书成功续期后，重载 Nginx 配置
   systemctl reload nginx
   # 如果你需要将证书复制到其他自定义位置，可以在此添加 cp 命令
   ```

   赋予脚本执行权限：

   ```
   sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-webserver.sh
   ```

3. **添加系统定时任务（Cron Job）**：

   ```
   sudo crontab -e
   ```

   在文件末尾添加一行，设定每天凌晨 3:00 自动检查并续期（证书到期前 30 天内才会实际续签）：

   ```shell
   0 3 * * * /usr/bin/certbot renew --quiet --deploy-hook "/etc/letsencrypt/renewal-hooks/deploy/reload-webserver.sh"
   
   #0 3 1 */3 * → 每隔 3 个月的1号3点执行一次
   #--quiet → 静默模式
   #--post-hook "nginx -s reload" → 续签后自动重载 Nginx
   ```

## acme.sh 实现ssl申请以及自动续签

### 🚀 acme.sh 的核心特点与工作原理

acme.sh 是一个纯 Shell (Unix shell) 语言编写的 ACME 客户端，其核心特点是 **“零依赖、零侵入、全自动”**。

- **零依赖**：它只是一个 Shell 脚本，安装后就是单个文件，几乎不依赖系统特定的包管理器，因此在任何 Unix-like 系统（包括 openEuler）上都能以相同方式运行。
- **零侵入**：它默认安装到用户主目录（`~/.acme.sh/`），**不需要 root 权限**。所有操作都在此目录下完成，不会修改系统全局的软件配置，对系统非常“干净”。
- **全自动**：证书自动续期是其设计核心。安装时会**自动创建一个隐藏的 Cron 定时任务**，每天检查证书并在到期前自动续签，你几乎可以“一次设置，终身忘记”。

### 🐚 **acme.sh 常用命令手册**

acme.sh 的命令风格更统一，基本遵循 `acme.sh --action [选项]`。

| 场景/目的               | 常用命令示例                                                 | 关键参数与说明                                               |
| :---------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **1. 签发证书（核心）** | `acme.sh --issue -d example.com`                             | `--dns dns_ali`：使用阿里云DNS API验证（其他：`dns_cf`, `dns_dp`）。 `--webroot /var/www/html`：Webroot验证。 |
| **2. 安装证书（关键）** | `acme.sh --install-cert -d example.com \` `--key-file /nginx/ssl/key.pem \` `--fullchain-file /nginx/ssl/cert.pem \` `--reloadcmd "systemctl reload nginx"` | **`--install-cert`**：将证书安装到指定位置。 **`--reloadcmd`**：**设置续期后自动执行的命令**（如重载Web服务），是实现全自动化的灵魂。 |
| **3. 查看已签发证书**   | `acme.sh --list`                                             | 列出所有由该acme.sh实例管理的证书。                          |
| **4. 手动强制续期**     | `acme.sh --renew -d example.com --force`                     | **`--force`**：强制立即续期。                                |
| **5. 吊销证书**         | `acme.sh --revoke -d example.com`                            | 吊销指定域名的证书。                                         |
| **6. 更新acme.sh自身**  | `acme.sh --upgrade`                                          | 自动升级acme.sh到最新版本。                                  |
| **7. 切换CA机构**       | `acme.sh --set-default-ca --server letsencrypt`              | 切换默认的CA，例如换成 `--server zerossl`。                  |
| **8. 创建ECC证书**      | `acme.sh --issue -d example.com --keylength ec-256`          | **`--keylength ec-256`**：签发更安全、更高效的**ECC椭圆曲线证书**（推荐）。 |



### 基于场景 ([fermis.top](https://fermis.top/) + 阿里云) 的 acme.sh 操作指南

假设你已登录 openEuler 服务器（非root普通用户即可），以下是具体步骤：

1. **一键安装 acme.sh**

   ```
   # 通过curl下载并安装到当前用户目录
   curl https://get.acme.sh | sh
   # 安装完成后，重载一下Shell配置（如.bashrc）
   source ~/.bashrc
   ```

   安装后，脚本会提示已添加 Cron 任务。主程序位于 `~/.acme.sh/acme.sh`。

2. **配置阿里云 DNS API**
   和之前一样，你需要阿里云的 `AccessKey ID` 和 `AccessKey Secret`。通过环境变量配置（比文件更安全）：

   ```
   export Ali_Key="你的AccessKeyId"
   export Ali_Secret="你的AccessKeySecret"
   ```

   1. 执行 `export` 命令只会将这两个变量添加到你**当前终端会话的内存环境**中，而**不会**自动写入任何文件。它们的作用是临时的，一旦你关闭这个终端窗口或启动新的会话，这些变量就会消失。

      为了永久、安全地配置阿里云 API 密钥，你需要将它们写入一个**配置文件**，让 `acme.sh` 每次执行时都能自动读取。最标准的做法是使用 `acme.sh` 内置的配置命令。

   2. **使用 `acme.sh` 的专用配置命令**。这条命令会将你的密钥**加密保存**到 `acme.sh` 的配置目录 (`~/.acme.sh/account.conf`) 中，是最安全、最推荐的方式。

      ```
      ~/.acme.sh/acme.sh --register-account \
        --server letsencrypt \
        --config-home ~/.acme.sh \
        --accountemail '你的邮箱@example.com'
      ```

      执行后，它会自动引导你交互式地输入 `Ali_Key` 和 `Ali_Secret` 等关键信息，并安全存储。

   3. **方法二：手动创建配置文件（备选）**。如果上述命令不奏效，你也可以手动在 `~/.acme.sh/account.conf` 这个配置文件中添加以下两行：

      ```
      # 使用文本编辑器（如vim）打开配置文件
      vim ~/.acme.sh/account.conf
      ```

      在文件中添加：

      ```
      SAved_Ali_Key='你的AccessKeyId'
      SAved_Ali_Secret='你的AccessKeySecret'
      ```

      > **注意**：变量名**必须**以 `SAved_` 开头，这是 `acme.sh` 识别并自动加载的固定前缀。保存并退出编辑器即可。

   ### 如何验证配置是否成功？

   配置完成后，你可以通过以下命令验证 `acme.sh` 是否能正确读取密钥来申请证书：

   ```
   # 使用 --test 或 --staging 参数在测试环境试运行，不会消耗正式签发限额
   ~/.acme.sh/acme.sh --issue --dns dns_ali -d fermis.top --staging
   ```

   如果看到 `Verify finished, start issue certificate` 或 `Cert success` 之类的成功提示，说明配置完全正确。之后你就可以放心地使用正式命令（去掉 `--staging`）来签发真实证书了。

   ### 重要安全提醒

   - **绝对不要**将 `AccessKey` 明文提交到公开的 Git 仓库、论坛或聊天记录中。
   - 在阿里云控制台创建的 RAM 用户密钥，请务必只授予 **`AliyunDNSFullAccess`** 权限，遵循最小权限原则。
   - 定期检查阿里云RAM的访问密钥管理，删除不再使用的密钥。

   总结：请优先使用 **`acme.sh --register-account`** 命令来配置密钥，这是最规范和安全的方法。如果遇到问题，再使用手动写入 `~/.acme.sh/account.conf` 文件的方式。

   

   **然后运行命令来申请证书：**

   ```
   ~/.acme.sh/acme.sh --issue --dns dns_ali \
     -d fermis.top \
     -d '*.fermis.top'
   ```

   这个命令会自动调用阿里云 API 添加并删除验证记录。**无需** `--email`、`--agree-tos` 等参数（首次安装时已默认同意）。

3. **安装证书到 Web 服务器（以 Nginx 为例）**
   申请成功后，证书在 `~/.acme.sh/fermis.top/` 目录。你需要将它们**复制或链接**到 Web 服务器（如 Nginx）能读取的位置。acme.sh 可以自动完成这个步骤：

   ```
   # 假设你的 Nginx 配置中证书期望的路径是 /etc/nginx/ssl/
   sudo mkdir -p /etc/nginx/ssl/fermis.top
   
   ~/.acme.sh/acme.sh --install-cert -d fermis.top \
     --key-file       /etc/nginx/ssl/fermis.top/privkey.pem \
     --fullchain-file /etc/nginx/ssl/fermis.top/fullchain.pem \
     --reloadcmd     "sudo systemctl reload nginx"
   ```

   

   这个命令做了三件事：① 复制证书到指定路径；② 设置未来**续期后自动复制到相同路径**；③ 续期后自动执行 `sudo systemctl reload nginx` 来重载服务。

**至此，全部设置完成。** 后续 acme.sh 会通过自带的 Cron 任务自动检查并续期证书，续期后会执行你指定的 `--reloadcmd` 命令，完全无需干预。

### 选择建议与总结

- **选择 acme.sh，如果你**：追求**极致的简单、自动化**，不喜欢处理权限问题（sudo），或者环境受限（如轻量容器、嵌入式设备）。它的“设置即忘记”体验非常出色。
- **坚持使用 Certbot，如果你**：更习惯主流的、由系统包管理器维护的软件，环境是标准的服务器，且需要与 Apache/Nginx 等深度集成的官方插件支持。

对于个人域名 `fermis.top` 和 openEuler 服务器，**两者都能完美工作**。如果你希望流程更“傻瓜化”、更符合 Shell 运维习惯，**acme.sh 是更精妙的选择**；如果你更看重官方文档和社区支持的广泛性，Certbot 则是更稳妥的选项。


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/e05f4ac/  

