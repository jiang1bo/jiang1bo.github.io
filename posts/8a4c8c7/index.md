# Cloudfared白嫖指南


cloudfared+warp+tunel+webssh+warp

<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

家里的 NAS、自建博客、开发环境想从外网访问，但你没有公网 IP? 运营商不给力，DDNS 又不稳定，端口映射还有安全风险？Cloudflare Tunnel 提供了一个优雅的解决方案: **零公网 IP、零入站端口、默认 HTTPS, 还能和 Zero Trust 深度集成做身份验证**。

简单来说，你在内网跑一个轻量级的 `cloudflared` 进程，它主动向 Cloudflare 边缘网络建立加密隧道。当用户访问你的域名时，流量先到 Cloudflare, 经过 WAF、Access 等安全检查后，再通过隧道转发到你的内网服务。整个过程，你的源站 IP 完全不暴露，防火墙也不用开任何入站端口。

**本文适合:**

- 想从外网访问家里 NAS / 自建服务的个人用户
- 需要为团队提供安全内网访问的小团队运维
- 不想折腾端口映射和动态 DNS 的开发者

**读完你将学会:**

- 两种部署方式：快速上手的控制台 Token 模式 vs 灵活的命名隧道配置模式
- 如何配置多服务、非 HTTP 协议 (SSH/RDP) 支持
- 结合 Zero Trust Access 做身份验证，让敏感服务更安全
- 常见问题排查和安全加固最佳实践

## 工作原理

在传统方式中，要从外网访问内网服务，你需要：公网 IP → 端口映射 → 防火墙放行。这意味着你的源站直接暴露在互联网上，容易被扫描、攻击。

Cloudflare Tunnel 反其道而行: **由内网主动连出去**, 建立一条加密隧道到 Cloudflare 边缘。用户访问你的域名时:

Copy

```
   用户浏览器 
    ↓ HTTPS
Cloudflare 边缘服务器(WAF / Access / TLS 终止)
    ↓ 加密隧道
cloudflared 连接器(内网)
    ↓ HTTP/SSH/RDP 等
你的本地服务(NAS/博客/面板)
```

**核心优势:**

- **无需公网 IP**、CGNAT 环境都能用
- **默认 HTTPS** 自动签发证书
- **隐藏源站 IP** 防护自带
- **Zero Trust 集成**: 可以加身份验证、设备态势检查等安全策略

## 准备工作

开始之前，确保你有:

1. **Cloudflare 账号**: 免费计划即可，访问 [dash.cloudflare.com](https://dash.cloudflare.com/) 注册
2. **域名托管在 Cloudflare**: 需要将域名的 NS 记录指向 Cloudflare (在域名注册商处修改)
3. **Cloudflare Zero Trust 开通**: 首次开通可能需要绑定支付方式 (用于防滥用), 但免费计划不会扣费
4. **一台内网设备** / Linux /macOS 或支持 Docker 的环境都可以

## 路径选择：快速 vs 灵活

Cloudflare Tunnel 提供两种配置方式:

**路径 A - 控制台托管 (推荐新手):**

- 在 Zero Trust 控制台创建隧道，复制 Token 命令直接跑
- 适合快速验证、单个服务、不需要复杂配置的场景
- 配置都在控制台可视化管理，重装机器只需复制命令

**路径 B - 命名隧道 + config.yml (推荐进阶):**

- 本地创建隧道并生成凭据文件，写 YAML 配置管理多服务
- 适合多服务编排、非 HTTP 协议、私网路由、需要版本控制的场景
- 更灵活，但需要手动管理配置文件

**建议:** 先走路径 A 快速跑通，确认效果；长期使用或需要高级功能，再切换到路径 B。

------

## 路径 A: 控制台托管隧道 (10 分钟上手)

这是最简单的方式，几乎不需要写配置文件。

### A1. 创建隧道

1. 登录 [Cloudflare Zero Trust](https://one.dash.cloudflare.com/)
2. 左侧导航: **Networks → Tunnels**
3. 点击 **Create a tunnel**
4. 选择 **Cloudflared** 连接器类型，点 **Next**
5. 给隧道起个名字 (比如 `home-tunnel`), 点 **Save tunnel**

### A2. 安装并运行 cloudflared

创建隧道后，页面会显示不同平台的安装和启动命令。

**选择你的操作系统:**

控制台会根据你选择的平台 (Windows /macOS/ Linux / Docker) 显示对应的安装步骤。如果机器上还没安装 cloudflared, 按页面指引先完成安装。

**复制启动命令:**

安装完成后，页面会给出一条带 Token 的命令，类似:

```
   # 临时运行(测试用)
cloudflared tunnel --no-autoupdate run --token <你的TOKEN>

# 或安装为系统服务(推荐)
cloudflared service install <你的TOKEN>
```

> **提示:** Token 模式的优势在于凭据直接包含在命令中，重装机器只需复制这条命令即可恢复服务，无需管理凭据文件。

**在你的机器上执行该命令**

- **如果是临时测试**: 直接运行带 `--token` 的命令，关闭终端会停止
- **如果是长期运行**: 使用 `cloudflared service install` 安装为系统服务，开机自启

执行后，回到控制台页面，底部应显示连接器状态为 **HEALTHY**, 点 **Next** 继续。

### A3. 绑定公开域名

在隧道详情页，切换到 **Public Hostname** 标签:

1. 点击 **Add a public hostname**
2. **Subdomain**: 填写子域名，比如 `admin`
3. **Domain**: 选择你的域名，比如 `example.com`
4. **Service**:
   - **Type**: 选择 `HTTP`(或 `HTTPS` 如果内网服务用了 SSL)
   - **URL**: 填写内网服务地址，比如 `http://localhost:8080` 或 `http://192.168.1.100:9000`
5. 点 **Save hostname**

**DNS 记录会自动创建**,Cloudflare 会添加一条 CNAME 记录指向你的隧道。

### A4. 验证

1. 等待 1-2 分钟让 DNS 生效
2. 访问 `https://admin.example.com`
3. 如果一切正常，应该能看到你的内网服务

**常见问题:**

- **502 错误**: 检查内网服务是否正常运行，端口是否正确
- **连接器显示 Disconnected**: 检查 cloudflared 进程是否还在运行

------

## 路径 B: 命名隧道 + config.yml (进阶配置)

这条路径适合需要精细管理、多服务编排、非 HTTP 协议支持的场景。

### B0. 安装 cloudflared

根据你的系统选择安装方式:

**Linux (Debian/Ubuntu, 推荐方式):**

```
   # 从 GitHub 下载最新版本
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# 验证安装
cloudflared --version
```

**macOS:**

```
   brew install cloudflare/cloudflare/cloudflared

cloudflared --version
```

**Windows:**

```
   # 使用 winget(Windows 10/11)
winget install --id Cloudflare.cloudflared

# 验证安装
cloudflared --version
```

如果你的系统没有包管理器，可以从 [GitHub Releases](https://github.com/cloudflare/cloudflared/releases/latest) 下载对应平台的二进制文件。

### B1. 登录并创建命名隧道

**1. 登录 Cloudflare 账号:**

```
   cloudflared tunnel login
```

这会打开浏览器让你授权，完成后会在本地生成 `cert.pem` 证书文件:

- Linux/macOS: `~/.cloudflared/cert.pem`
- Windows: `%USERPROFILE%\.cloudflared\cert.pem`

**2. 创建隧道:**

```
   cloudflared tunnel create my-tunnel
```

你会看到类似这样的输出:

```
   Tunnel credentials written to /home/user/.cloudflared/12345678-abcd-1234-abcd-1234567890ab.json.
cloudflared chose this file based on where your origin certificate was found.
Keep this file secret. To revoke these credentials, delete the tunnel.

Created tunnel my-tunnel with id 12345678-abcd-1234-abcd-1234567890ab
```

记下这个隧道 UUID ( `12345678-abcd-1234-abcd-1234567890ab`), 后面配置会用到。

**3. 绑定域名到隧道:**

```
   cloudflared tunnel route dns my-tunnel app.example.com
```

这会在 Cloudflare DNS 中自动创建一条 CNAME 记录，指向你的隧道。

### B2. 编写配置文件

创建配置文件 `config.yml`:

- Linux/macOS: `~/.cloudflared/config.yml`
- Windows: `%USERPROFILE%\.cloudflared\config.yml`

**基础示例 (单服务):**

```
   tunnel: 12345678-abcd-1234-abcd-1234567890ab
credentials-file: /home/user/.cloudflared/12345678-abcd-1234-abcd-1234567890ab.json

ingress:
  - hostname: app.example.com
    service: http://localhost:8080
  - service: http_status:404
```

**多服务示例:**

```
   tunnel: 12345678-abcd-1234-abcd-1234567890ab
credentials-file: /home/user/.cloudflared/12345678-abcd-1234-abcd-1234567890ab.json

ingress:
  # 博客
  - hostname: blog.example.com
    service: http://localhost:8080
  
  # NAS 管理界面
  - hostname: nas.example.com
    service: https://192.168.1.10:5001
    originRequest:
      noTLSVerify: true  # 如果是自签证书
  
  # 开发环境
  - hostname: dev.example.com
    service: http://localhost:3000
  
  # SSH 服务
  - hostname: ssh.example.com
    service: ssh://localhost:22
  
  # 默认规则(必需,匹配不到时返回 404)
  - service: http_status:404
```

**Windows 路径注意:**

```
   # Windows 路径需要用双反斜杠或正斜杠
credentials-file: C:\\Users\\YourName\\.cloudflared\\12345678-abcd-1234-abcd-1234567890ab.json
# 或
credentials-file: C:/Users/YourName/.cloudflared/12345678-abcd-1234-abcd-1234567890ab.json
```

### B3. 运行隧道

**前台运行 (测试):**

```
   cloudflared tunnel run my-tunnel
```

你会看到类似输出:

```
   2024-12-07T10:30:00Z INF Starting tunnel tunnelID=12345678-abcd-1234-abcd-1234567890ab
2024-12-07T10:30:01Z INF Connection registered connIndex=0 location=NRT
2024-12-07T10:30:01Z INF Connection registered connIndex=1 location=NRT
```

看到 `Connection registered` 说明隧道已建立。

**安装为系统服务 (生产环境):**

**Linux:**

```
   # 将配置复制到系统目录
sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/config.yml /etc/cloudflared/
sudo cp ~/.cloudflared/*.json /etc/cloudflared/

# 安装并启动服务
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared

# 查看状态
sudo systemctl status cloudflared
```

**macOS:**

```
   # 安装为 launchd 服务
sudo cloudflared service install
sudo launchctl start com.cloudflare.cloudflared
```

**Windows:**

```
   # 以管理员身份运行
cloudflared service install

# 启动服务
Get-Service cloudflared | Start-Service
```

### B4. 非 HTTP 协议与私网路由

**SSH / RDP 访问:**

在 `config.yml` 中配置非 HTTP 协议:

```
   ingress:
  - hostname: ssh.example.com
    service: ssh://127.0.0.1:22
  
  - hostname: rdp.example.com
    service: rdp://192.168.1.20:3389
  
  - service: http_status:404
```

**客户端连接 SSH:**

用户需要先安装 `cloudflared`, 然后:

```
   # 方式 1:直接连接
cloudflared access ssh --hostname ssh.example.com

# 方式 2:配置 SSH 客户端
# 在 ~/.ssh/config 添加:
Host ssh.example.com
  ProxyCommand cloudflared access ssh --hostname %h
```

**私网路由 (访问整个内网段):**

1. 在配置文件中启用:

```
   warp-routing:
  enabled: true
```

1. 在 Zero Trust 控制台添加私网路由:
   - **Networks → Tunnels** → 选择你的隧道 → **Private Networks** 标签
   - 点击 **Add a private network**
   - 输入 CIDR, 比如 `192.168.1.0/24`
2. 访问端安装 [Cloudflare WARP 客户端](https://1.1.1.1/), 登录你的 Zero Trust 团队后，就能直接访问内网 IP 了。

------

## 安全加固清单

Cloudflare Tunnel 虽然默认就比传统端口映射安全，但还可以做得更好:

### 1. 为敏感服务启用 Zero Trust Access

不要让管理面板、数据库管理工具等敏感服务裸奔在互联网上，哪怕通过隧道访问。

**步骤:**

1. 在 Zero Trust 控制台: **Access → Applications → Add an application**
2. 选择 **Self-hosted**
3. 配置应用:
   - **Application name**: `NAS Admin Panel`
   - **Session Duration**: `24 hours`
   - **Application domain**: `nas.example.com`
4. 添加策略:
   - **Policy name**: `Allow My Email Only`
   - **Action**: `Allow`
   - **Include**: 选择 **Emails** → 输入你的邮箱 `you@example.com`
5. 保存后，访问 `nas.example.com` 会先跳转到 Cloudflare Access 登录页，验证通过才能访问

**支持的身份提供商:**

- Google、GitHub、Microsoft 等 OAuth
- 一次性 PIN 码 (发送到邮箱)
- SAML / OIDC 企业身份源

### 2. 最小暴露原则

- **不要暴露数据库端口**(MySQL 3306、PostgreSQL 5432 等), 用私网路由 + WARP 访问
- **不要暴露消息队列、Redis 等中间件**, 除非你清楚在做什么
- **管理面板和对外服务分开子域**, 便于差异化安全策略

### 3. 正确处理 HTTPS 回源

如果内网服务用了自签证书，测试阶段可以在配置中跳过验证:

```
   ingress:
  - hostname: admin.example.com
    service: https://localhost:8443
    originRequest:
      noTLSVerify: true  # 仅用于开发/测试
```

**生产环境建议:**

- 让 Cloudflare 终止 TLS (用 HTTP 回源)
- 或给内网服务配置由内网 CA 签发的证书，并在 `originRequest` 中指定 CA

### 4. 部署冗余连接器

同一个隧道可以同时运行多个 `cloudflared` 实例 (比如在不同机器上), 提供高可用:

```
   # 机器 A
cloudflared tunnel run my-tunnel

# 机器 B(使用相同的 tunnel ID 和凭据文件)
cloudflared tunnel run my-tunnel
```

Cloudflare 会自动负载均衡，任一实例挂掉不影响服务。

### 5. 日志和监控

设置日志级别以便排查问题:

```
   cloudflared tunnel --loglevel debug run my-tunnel
```

或在配置文件中:

```
   loglevel: info
```

**查看日志:**

- **Linux systemd**: `sudo journalctl -u cloudflared -f`
- **Docker**: `docker logs -f cloudflared`
- **Windows**: 事件查看器或服务日志

------

## 常见问题排查

### Q1: 浏览器显示 502 Bad Gateway

**可能原因:**

- 内网服务没启动
- 端口号填错了
- 防火墙拦截了 cloudflared 到内网服务的连接

**排查步骤:**

```
   # 在运行 cloudflared 的机器上测试内网服务
curl http://localhost:8080

# 检查端口是否在监听
netstat -tlnp | grep 8080  # Linux
netstat -an | findstr 8080  # Windows
```

### Q2: 隧道状态显示 Unhealthy

**可能原因:**

- cloudflared 进程挂了
- 网络出站受限，无法连接 Cloudflare
- 配置文件有错误

**排查:**

```
   # 查看进程
ps aux | grep cloudflared  # Linux
Get-Process cloudflared     # Windows

# 测试网络连通性
curl https://api.cloudflare.com/cdn-cgi/trace

# 前台运行查看详细日志
cloudflared tunnel --loglevel debug run my-tunnel
```

### Q3: HTTPS 证书警告

如果用户访问时看到证书警告，通常是域名配置问题:

- 确保域名已托管在 Cloudflare
- 确保 Cloudflare SSL/TLS 设置不是 "关闭"
- 如果是多级子域 ( `admin.sub.example.com`), 需要申请 Advanced Certificate

### Q4: 访问速度慢或不稳定

- **检查连接器是否健康**: 控制台查看连接器状态
- **使用 Docker 并设置自动重启**: `docker run --restart unless-stopped ...`
- **部署多个连接器**: 提升可用性
- **检查本地网络**: 如果是家用宽带，上行带宽可能是瓶颈

### Q5: Docker 容器启动后立即退出

**常见原因:**

- Token 或凭据文件路径不对
- 配置文件格式错误 (YAML 缩进)

**排查:**

```
   # 查看容器日志
docker logs cloudflared

# 挂载正确的配置路径
docker run -d --name cloudflared --restart unless-stopped \
  -v ~/.cloudflared:/home/nonroot/.cloudflared \
  cloudflare/cloudflared:latest tunnel run my-tunnel
```

------

## 与其他方案对比

| 方案                  | 公网 IP        | 端口映射 | 配置难度 | 安全性           | HTTPS    | 成本                 |
| --------------------- | -------------- | -------- | -------- | ---------------- | -------- | -------------------- |
| **Cloudflare Tunnel** | 不需要         | 不需要   | 低 - 中  | 高 (可加 Access) | 自动     | 免费                 |
| **FRP**               | 需要中转服务器 | 需要     | 中       | 中 (需自配 SSL)  | 需自配   | 中转服务器成本       |
| **Ngrok**             | 不需要         | 不需要   | 低       | 中               | 自动     | 免费版受限，付费较贵 |
| **传统端口映射**      | 需要           | 需要     | 高       | 低 (IP 暴露)     | 需自配   | 无 (但需公网 IP)     |
| **Tailscale**         | 不需要         | 不需要   | 低       | 高               | N/A(P2P) | 免费版有设备数限制   |

**Cloudflare Tunnel 的独特优势:**

- **和 Cloudflare 生态深度集成**、DDoS 防护、CDN、Access、Gateway 一条龙
- **免费且无流量限制**: 对个人和小团队非常友好 (免费计划受 Cloudflare 的公平使用和反滥用策略约束，大规模或高风险场景可能需要付费计划)
- **全球边缘网络**: 延迟低、可用性高

## 写在最后

Cloudflare Tunnel 把 "从外网安全访问内网服务" 这件事变得非常简单。对于个人用户，用控制台 Token 模式几分钟就能跑起来；对于需要精细管理的场景，命名隧道 + `config.yml` 也提供了足够的灵活性。

**我的实际使用建议:**

1. **先用路径 A 快速验证**: 确认 Cloudflare Tunnel 符合你的需求
2. **生产环境切到路径 B**: 把配置文件纳入版本控制，便于维护
3. **敏感服务一定加 Access**: 不要偷懒，设置一个邮箱验证就能大幅提升安全性
4. **结合私网路由用 WARP**: 对于需要访问内网多个设备的场景，比单独配置公开主机名更方便

如果你在使用过程中遇到问题，欢迎查阅官方文档，或在 评论区 / Cloudflare 社区提问。祝你折腾愉快！

此文由 [Mix Space](https://github.com/mx-space) 同步更新至 xLog
原始链接为 https://blog.astrasolis.top/posts/tutorial/cloudflare-tunnel-tutorial





## 第一步：在VPS上安装 Cloudflared

首先，用手机热点或者有IPv6的网络，通过SSH登录到你的VPS上。

登录成功后，复制下面的命令粘贴进去，下载并安装Cloudflared（以Debian/Ubuntu系统为例）：

```bash
# 下载安装包
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# 安装
sudo dpkg -i cloudflared.deb
```

安装完成后，输入 `cloudflared --version`，如果能看到版本号，说明安装成功。

------

## 第二步：登录并创建隧道

这一步是把你的VPS和Cloudflare账号绑定。

### 1. 登录认证

在VPS里输入命令：

```bash
cloudflared tunnel login
```

这时候屏幕上会弹出一长串URL链接。

**操作：** 复制这个链接，在你电脑的浏览器里打开。登录你的Cloudflare账号，选择你的域名，点击"Authorize"（授权）。

授权成功后，VPS的屏幕上会提示"Successfully logged in"。

> ⚠️ **安全提示：** 认证后会在 `/root/.cloudflared/` 目录生成凭证文件，务必妥善保管，不要泄露！

### 2. 创建隧道

给你的隧道起个名字，比如叫 `vps299`。输入命令：

```bash
cloudflared tunnel create vps299
```

**关键点：** 命令执行后，屏幕上会显示一个 **Tunnel ID**（UUID格式，一串长长的字符），一定要复制下来记在记事本里！同时系统会自动生成一个 `你的UUID.json` 文件。

### 3. 绑定域名

我们要把隧道绑定到一个二级域名上，比如 `ssh.你的域名.com`。输入命令：

```bash
# 格式：cloudflared tunnel route dns <隧道名> <二级域名>
cloudflared tunnel route dns vps299 ssh.yourdomain.com
```

**记得把 `yourdomain.com` 替换成你自己的域名！**

> 💡 **说明：** 这条命令会自动在Cloudflare创建一条CNAME记录，无需手动添加DNS。

------

## 第三步：配置隧道转发SSH

我们需要告诉隧道：当外面的请求进来时，把它转发给VPS内部的SSH端口（默认是22）。

### 1. 创建配置文件

输入命令：

```bash
mkdir -p /root/.cloudflared/
nano /root/.cloudflared/config.yml
```

### 2. 写入配置

在编辑器里粘贴以下内容（**注意替换你的UUID**）：

```yaml
tunnel: 你的UUID
credentials-file: /root/.cloudflared/你的UUID.json

ingress:
  - hostname: ssh.yourdomain.com
    service: ssh://localhost:22
  - service: http_status:404
```

**说明：**

- 第1行：填你刚才记下来的Tunnel ID
- 第2行：这个.json文件在创建隧道时已自动生成
- 第5行：填你绑定的域名
- 最后一行：默认404规则（必须保留）

按 **Ctrl+O** 保存，按 **Ctrl+X** 退出。

------

## 第四步：启动并设置开机自启

配置写好了，现在我们要启动它，并让它以后开机自动运行。

```bash
# 安装服务
cloudflared service install

# 启动服务
systemctl start cloudflared

# 查看状态
systemctl status cloudflared
```

如果看到绿色的 **active (running)**，恭喜你，服务端配置完成！

### 常见问题排查

如果启动失败，用以下命令查看日志：

```bash
journalctl -u cloudflared -f
```

如果连接不上，检查DNS是否生效：

```bash
dig ssh.yourdomain.com
```

------

## 第五步：在本地电脑安装 Cloudflared

**重要：** 要连接隧道，你的本地电脑（客户端）也需要安装 cloudflared！

### Windows系统

1. 访问：https://github.com/cloudflare/cloudflared/releases
2. 下载 `cloudflared-windows-amd64.exe`
3. 重命名为 `cloudflared.exe` 并放到 `C:\Windows\System32\` 目录

### macOS系统

```bash
brew install cloudflared
```

### Linux系统

```bash
# 下载安装包
curl -L --output cloudflared-linux-amd64.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# 安装
sudo dpkg -i cloudflared-linux-amd64.deb
```

安装完成后，在终端输入 `cloudflared --version` 验证是否成功。

------

## 第六步：配置SSH连接

现在，你可以断开手机热点，切回你只有IPv4的公司/家里WiFi了。

### 方法一：修改SSH配置文件（推荐）

**编辑SSH配置文件：**

Windows: 在用户目录下创建 `.ssh\config` 文件
macOS/Linux: 编辑 `~/.ssh/config` 文件

**写入以下内容：**

```
Host ssh.yourdomain.com
  ProxyCommand cloudflared access ssh --hostname %h
```

**保存后，就可以像平时一样用SSH连接了：**

```bash
ssh root@ssh.yourdomain.com
```

### 方法二：直接使用命令连接

如果不想配置文件，也可以每次用这个命令连接：

```bash
cloudflared access ssh --hostname ssh.yourdomain.com
```

然后按提示输入用户名和密码。

### 方法三：使用SSH客户端（XShell、FinalShell等）

大部分SSH客户端都支持ProxyCommand。以XShell为例：

1. 新建会话
2. 主机：`ssh.yourdomain.com`
3. 在"连接"设置里，找到"代理"选项
4. 命令行填写：`cloudflared access ssh --hostname ssh.yourdomain.com`

------

## 为什么不能直接SSH连接？

你可能会问：为什么不能直接 `ssh root@ssh.yourdomain.com` 就连上？

**原因：** Cloudflare的HTTP代理默认不支持SSH协议的22端口。直接连接会显示"Connection closed"错误。

**必须通过 cloudflared 客户端建立专用隧道连接才行。**

------

## 进阶玩法：同时暴露Web面板

如果你的VPS上装了宝塔、1Panel等Web面板，可以同时把它也暴露出来。

修改 `/root/.cloudflared/config.yml`，改成这样：

```yaml
tunnel: 你的UUID
credentials-file: /root/.cloudflared/你的UUID.json

ingress:
  - hostname: ssh.yourdomain.com
    service: ssh://localhost:22
  - hostname: panel.yourdomain.com
    service: http://localhost:8888
  - service: http_status:404
```

然后重启服务：

```bash
systemctl restart cloudflared
```

记得也绑定一下panel的DNS：

```bash
cloudflared tunnel route dns vps299 panel.yourdomain.com
```

现在你可以用 `http://panel.yourdomain.com` 直接访问面板了！这个不需要客户端安装cloudflared，浏览器直接打开就行。

------

## 总结

经过这波操作，这台只有IPv6的VPS，在Cloudflare Tunnel的加持下，已经变成了一台全网通的神机。

**关键点回顾：**

- 服务端（VPS）要安装并配置 cloudflared
- 客户端（你的电脑）也要安装 cloudflared
- SSH连接必须通过 ProxyCommand 方式，不能直接连
- 无痕界面可以正常使用

## 参考资料：

[使用 Cloudflare Tunnel 实现内网穿透 - Undefined443 - 博客园](https://www.cnblogs.com/Undefined443/p/18837734)

[CloudFlare Tunnel 免费内网穿透的简明教程 - 少数派](https://sspai.com/post/79278)

[利用 Cloudflare Tunnel 搭建一个浏览器端的ssh页面 | 败草寒风集](https://www.aspchang.cn/posts/cloudflare-tunnel-web-ssh-tutorial/)

[只有IPv6怎么连SSH？Cloudflare Tunnel让IPv4网络也能丝滑访问！](https://serverinfo.cc/zhi-you-ipv6zen-yao-lian-ssh-cloudflare-tunnelrang-ipv4wang-luo-ye-neng-si-hua-fang-wen/)

[Cloudflare Tunnel 零公网 IP 安全外网访问教程 | IT瘾](https://itindex.net/detail/63121-cloudflare-tunnel-公网)

**官方文档:**

- [Cloudflare Tunnel 概览](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- [cloudflared 安装指南](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/)
- [配置文件参考](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/local/local-management/configuration-file/)
- [Zero Trust Access 文档](https://developers.cloudflare.com/cloudflare-one/policies/access/)

**下载链接:**

- [cloudflared GitHub Releases](https://github.com/cloudflare/cloudflared/releases/latest)
- [Cloudflare WARP 客户端](https://1.1.1.1/)

**社区资源:**

- [Cloudflare 社区论坛](https://community.cloudflare.com/)
- [GitHub Issues](https://github.com/cloudflare/cloudflared/issues)


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/8a4c8c7/  

