# Openeuler:ssh常见配置和操作命令+mosh编译食用指南


ssh常见配置和操作命令+mosh编译食用指南

<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

在 openEuler 上从源码编译安装 mosh-1.4.0，虽然步骤较多，但能让你更灵活地控制版本和编译选项。整个过程可总结为以下流程：

```mermaid
flowchart TD
    A[开始: 源码编译安装 Mosh] --> B
    
    subgraph B[第一阶段: 安装编译环境与依赖]
        direction LR
        B1[安装基础开发工具] --> B2[安装必需的核心依赖库<br>protobuf, ncurses, openssl等] --> B3[安装可选但推荐的依赖<br>如zlib]
    end
    
    B --> C[第二阶段: 获取并编译源码]
    C --> C1[下载 mosh-1.4.0.tar.gz 并解压]
    C1 --> C2[进入源码目录并配置编译选项]
    C2 --> C3[执行 make 编译]
    
    C3 --> D[第三阶段: 安装与验证]
    D --> D1[sudo make install 安装]
    D1 --> D2[更新动态链接库缓存]
    D2 --> D3[验证 mosh 命令可用性]
    
    D3 --> E[第四阶段: 必要配置]
    E --> E1[配置服务器防火墙<br>开放UDP 60000-61000端口]
    E1 --> F[完成安装]
```





## ssh 利用非对称加密实现安全的登录

非对称加密中有两个密钥：公钥和私钥。公钥由私钥产生，但却无法推算出私钥；公钥加密后的密文，只能通过对应的私钥来解密。

终端要登录 Server 服务器，发起登录请求 ssh work@server.com。服务端运行有 sshd 服务，并监听指定的端口，默认为 22 号端口。

服务端会生成一对公钥和私钥；此时将公钥返回给客户端；客户端使用公钥，对登录密码进行加密（如服务器work用户密码为xxx），生成公钥加密字符串；客户端将公钥加密字符串发送给服务端；服务端使用私钥，解密公钥加密字符串，得到原始密码；校验密码是否合法（此为本机 work 密码）；返回登录结果给客户端：成功登录或密码错。

**在非对称加密中，由于只有公钥会被传输，而私钥是服务端本地保存，因此即便公钥被监听，也无法拿到原始密码，从而安全地登录服务器。**

```css
使用指定用户名和端口登录远程主机。ssh -p3600 root@9.134.114.170
```

#### **配置SSH免密登录**

#### **SSH 服务的配置文件**

**通常位于 `/etc/ssh/sshd_config`。可以使用文本编辑器编辑该文件以调整设置。常见配置选项包括：**

```shell
Port：指定 SSH 服务监听的端口（默认为 22）。Port 22
PermitRootLogin：控制是否允许 root 用户通过 SSH 登录。出于安全考虑，通常建议设置为 no。 PermitRootLogin no
PasswordAuthentication：控制是否允许密码认证。启用密钥认证时，可以将此选项设置为 no。 PasswordAuthentication yes
AllowUsers 和 DenyUsers：控制允许和拒绝登录的用户。AllowUsers user1 user2

## 修改配置文件后，使用以下命令重启 SSH 服务使更改生效：sudo systemctl restart ssh
```

#### **通过 ssh 实现免密登录**

免密登录的实现过程如下：

**1.在客户端使用 ssh-keygen 生成一对密钥：公钥+私钥；``**

**2.将客户端公钥追加到服务端的 authorized_key 文件中，完成公钥认证操作；**

**3.认证完成后，客户端向服务端发起登录请求，并传递公钥到服务端；**

**4.服务端检索 authorized_key 文件，确认该公钥是否存在。如果存在该公钥，则生成随机数 R，并用公钥进行加密，生成公钥加密字符串 pubKey(R)；**

**5.将公钥加密字符串传递给客户端；**

**6客户端使用私钥解密公钥加密字符串，得到 R；**

**7.服务端和客户端通信时会产生一个会话 ID(sessionKey)，用 MD5 对 R 和 SessionKey 进行加密，生成摘要；**

**8.客户端将生成的 MD5 加密字符串传给服务端；**

**9.服务端同样生成 MD5(R,SessionKey) 加密字符串；**

**10.如果客户端传来的加密字符串等于服务端自身生成的加密字符串，则认证成功。此时不用输入密码，即完成建连，可以开始远程执行 Shell 命令了。**

##### **1. 生成SSH密钥对**

```shell
# 检查是否已有SSH密钥
ls -la ~/.ssh/

##第一步使用 ssh-keygen 命令在客户端生成 RSA 公钥和私钥，一直回车确认。公钥和私钥默认名称为 id_rsa.pub（公钥）和私钥（id_rsa），默认保存在 ~/.ssh/目录下。
ssh-keygen -t rsa

# 生成ED25519密钥（推荐）
ssh-keygen -t ed25519 -C "mosh@$(hostname)"

# 或生成RSA密钥
ssh-keygen -t rsa -b 4096 -C "mosh@$(hostname)"

# 生成过程中会询问：
# - 保存位置（按回车使用默认 ~/.ssh/id_ed25519）
# - 密码短语（可留空）

```

##### **2. 复制公钥到远程主机**

```shell
##第二步将客户端公钥追加至服务端 ~/.ssh/authorized_keys 文件中，authorized_keys 是用来存放客户端公钥的文件。

##有三种方法，
#一是通过 ssh-copy-id 命令，
#例如使用 ssh-copy-id 命令实现如下
sudo dnf install -y openssh-clients
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 3600 root@9.134.114.170

# 使用ssh-copy-id并指定端口
ssh-copy-id -i ~/.ssh/id_ed25519_55.pub -p 55 root@192.168.8.55

# 使用openEuler的ssh-copy-id
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@remote-server

# 或者使用已有密钥
ssh-copy-id -p 55 root@192.168.8.55

#二是通过 scp 命令，此处不展示。
#三是手动复制。
# 方法2：手动复制
cat ~/.ssh/id_ed25519.pub | ssh user@remote-server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

##第三步使用 ssh 进行免密登录。
ssh -p3600 root@9.134.114.170
```

##### **3. 测试免密登录**

```shell
# 测试SSH连接
ssh user@remote-server

# 测试mosh连接
mosh user@remote-server

# 尝试SSH连接，看是否已经可以免密登录
ssh -o BatchMode=yes root@198.23.196.252 "echo 'SSH连接测试成功'"
```



### **ssh端口转发**

**SSH 支持端口转发功能，可以将本地端口转发到远程主机，或将远程端口转发到本地。**

例如：本地端口转发：将本地端口 8080 转发到远程主机的 80 端口。

```ruby
ssh -L 8080:localhost:80 user@example.com
```

远程端口转发：将远程主机的 8080 端口转发到本地的 80 端口。

```ruby
ssh -R 8080:localhost:80 user@example.com
```

动态端口转发：使用 SOCKS 代理将本地端口转发到远程主机。ssh -D 1080 user@example.com``SSH 配置文件``可以在 ~/.ssh/config 文件中配置 SSH 客户端的选项，以简化连接。例如：

```shell
Host myserver
HostName example.com
User myuser
Port 2222
IdentityFile ~/.ssh/id_rsa
```

这样，可以使用简化的命令连接：

```undefined
ssh myserver
```

### 删除或更改私钥上的密码

**如果您已经为您的私钥生成了密码并希望更改或删除它，您可以轻松地做到这一点。**

注意：要更改或删除密码，您必须知道原始密码。如果您丢失了密钥的密码短语，则没有追索权，您将不得不生成一个新的密钥对。

要更改或删除密码，只需键入：

```shell
ssh-keygen -pEnter file in which the key is (/root/.ssh/id_rsa):
#您可以键入要修改的键的位置或按 ENTER 接受默认值：
Enter old passphrase:
```

输入您要更改的旧密码。然后系统会提示您输入新密码：

```shell
Enter new passphrase (empty for no passphrase): Enter same passphrase again:
```

在这里，输入您的新密码或按 ENTER 删除密码。

## mosh编译食用指南

### 第一阶段：准备编译环境与依赖

**下载源码文件：[Release mosh-1.4.0 · mobile-shell/mosh](https://github.com/mobile-shell/mosh/releases/tag/mosh-1.4.0)**

**https://github.com/mobile-shell/mosh/archive/refs/tags/mosh-1.4.0.tar.gz**

**最好下载source code源代码，本人用其他版本代码总是莫名报错，**

**mosh-1.4.0项目中每个目录和文件的作用：**

#### **配置和构建相关文件**
1. **`configure.ac`** - **最重要的文件**
   - Autoconf的输入文件，用于生成`configure`脚本
   - 包含项目的配置需求，如检查编译器、库依赖、系统特性等
   - 定义了软件版本、安装路径、特性开关等

2. **`Makefile.am`** - **构建规则定义**
   - Automake的输入文件，用于生成`Makefile.in`
   - 定义了源代码如何编译、哪些文件需要安装、测试规则等

3. **`autogen.sh`** - **自动配置生成脚本**
   - 运行`autoreconf`等命令生成`configure`脚本
   - 用于从版本控制系统中检出的代码（您当前的情况）

4. **`build-package.sh`** / **`build-source-package.sh`** - **打包脚本**
   - 用于构建发行版软件包（如RPM、DEB）
   - `build-package.sh`：构建二进制包
   - `build-source-package.sh`：构建源码包

#### **文档和许可证文件**
5. **`README.md`** - **项目说明**
   - Markdown格式的说明文档
   - 包含项目简介、构建说明、使用方法等

6. **`ChangeLog`** - **变更日志**
   - 记录每个版本的详细变更历史
   - 格式通常遵循GNU ChangeLog规范

7. **`NEWS`** - **新闻文件**
   - 面向用户的版本更新摘要
   - 比ChangeLog更简洁，突出重要变化

8. **`COPYING`** - **主要许可证文件**
   - 包含GNU GPL许可证文本
   - mosh使用GPLv3许可证

9. **`COPYING.iOS`** - **iOS版许可证**
   - 针对iOS版本的特殊许可证条款

10. **`ocb-license.html`** - **OCB模式许可证**
    - OCB（Offset Codebook）加密模式的许可证
    - mosh使用OCB进行加密，这是其专利许可证

11. **`AUTHORS`** / **`THANKS`** - **贡献者列表**
    - `AUTHORS`：主要作者和版权持有者
    - `THANKS`：感谢列表，包括贡献者和支持者

#### **CI/CD配置文件**
12. **`appveyor.yml`** / **`appveyor.sh`** - **Windows CI配置**
    - AppVeyor持续集成配置
    - 用于在Windows平台上自动构建和测试

#### **目录结构**

##### **`src/`** - **源代码目录（核心）**
包含所有C++源代码文件：
- **`src/crypto/`** - 加密相关代码
  - OCB模式实现、密钥交换等
- **`src/frontend/`** - 客户端代码
  - 终端界面处理、用户输入输出
- **`src/network/`** - 网络通信代码
  - UDP传输、网络协议实现
- **`src/protobufs/`** - Protocol Buffers定义
  - `.proto`文件及生成的代码
- **`src/statesync/`** - 状态同步机制
  - 终端状态同步算法
- **`src/terminal/`** - 终端模拟器
  - 终端属性、字符编码处理
- **`src/util/`** - 工具函数
  - 日志、错误处理、辅助函数

##### **`man/`** - **手册页目录**
包含Unix手册页（man pages）：
- `man/mosh.1` - mosh客户端手册
- `man/mosh-server.1` - mosh服务器手册
- 通常使用troff/groff格式编写

##### **`scripts/`** - **辅助脚本目录**
包含各种构建和维护脚本：
- **自动补全脚本**（bash、zsh等）
- **安装/卸载脚本**
- **测试辅助脚本**
- **开发工具脚本**

##### **`conf/`** - **配置文件目录**
包含配置模板和示例：
- **PAM（可插拔认证模块）配置**
- **系统服务配置文件**
- **日志配置文件**
- **其他运行时配置**

##### **`m4/`** - **Autoconf宏目录**
包含自定义Autoconf宏：
- 用于`configure.ac`中检查特定库或功能
- 例如检查C++11支持、特定库版本等
- 扩展Autoconf的功能

##### **`debian/`** - **Debian/Ubuntu打包支持**
Debian系列发行版的打包文件：
- **`debian/control`** - 包依赖和描述
- **`debian/rules`** - 构建规则
- **`debian/changelog`** - Debian包变更日志
- **`debian/copyright`** - 版权信息
- **`debian/*.install`** - 文件安装规则

##### **`fedora/`** - **Fedora/RHEL打包支持**
Red Hat系列发行版的打包文件：
- **`.spec`文件** - RPM包的构建规范
- 可能包含`mosh.spec`或类似文件
- 定义了RPM包的构建、依赖和安装规则

##### **`macosx/`** - **macOS特定支持**
macOS平台特定文件：
- **Xcode项目文件**（`.xcodeproj`）
- **macOS特定补丁**
- **启动脚本和配置**

##### **为什么没有`configure`脚本？**

您遇到`configure`脚本缺失的情况是因为：

1. **从Git仓库直接打包**：这个tarball可能是从版本控制系统直接导出的，不包含生成的构建文件

2. **Autotools工作流程**：
   
   ```
   configure.ac + Makefile.am → autogen.sh/autoreconf → configure + Makefile.in
   ```
   
3. **需要生成步骤**：
   
   ```bash
   # 您需要执行的步骤
   ./autogen.sh              # 运行生成脚本
   # 或
   autoreconf -fvi           # 手动生成
   ```

##### mosh的目录结构特点：

1. **跨平台支持**：包含macOS、Windows（AppVeyor）、Linux（debian/fedora）的支持文件
2. **现代构建系统**：使用Autotools（Autoconf+Automake）确保跨平台兼容性
3. **模块化设计**：源代码按功能分目录，便于维护
4. **完整文档**：包含手册页、多种格式的说明文档
5. **打包友好**：直接支持主流Linux发行版的打包系统

这种目录结构是典型的GNU风格开源项目，理解了各部分的作用，您就能更好地进行定制化编译和问题排查。

在开始编译前，**必须**安装所有必要的依赖包。请**逐条执行**以下命令：

#### **安装基础编译工具**

```bash
sudo dnf groupinstall "Development Tools" -y
```

##### **安装 Mosh 核心依赖库**

这是最关键的一步，请确保全部安装成功：

```bash
sudo dnf install protobuf-devel ncurses-devel openssl-devel libutempter-devel gcc8 gcc8-c++ make automake autoconf wget tar perl-IO-Tty pam-devel perl-ExtUtils-MakeMaker -y
```
*   `protobuf-devel`：Mosh 通信协议的核心，**必须安装**。
*   `ncurses-devel`：终端处理库。
*   `openssl-devel`：加密通信支持。
*   `libutempter-devel`：记录终端会话。

##### **安装可选但推荐的依赖**

```bash
sudo dnf install zlib-devel -y
```

###  第二阶段：编译与安装
假设你已将 `mosh-1.4.0.tar.gz` 下载到当前用户的 `~/` 目录下。

1.  **解压源码并进入目录**
    
    ```bash
    # 设置编译环境
    export CC=gcc-11
    export CXX=g++-11
    export CXXFLAGS="-std=c++17"
    export LDFLAGS="-L/usr/local/lib"
    
    tar -xzvf ~/mosh-1.4.0.tar.gz
    # 进入源码目录
    cd mosh-1.4.0
    
    # 运行 autogen.sh 生成 configure 脚本
    ./autogen.sh
    
    ```
    
2.  **配置编译选项**
    运行 `configure` 脚本检查环境并生成编译规则：
    
    ```bash
    # 配置编译选项
    ./configure \
      --prefix=/usr/local \
      --sysconfdir=/etc \
      --localstatedir=/var \
      --enable-completion
    
    # 如果需要安装到特定目录，使用：
    # ./configure --prefix=/opt/mosh
    ```
    **请注意**：如果希望安装到特定路径（如 `/opt/mosh`），可使用 `./configure --prefix=/opt/mosh`。
    
3.  **编译源码**
    使用 `make` 命令进行编译，`-j$(nproc)` 表示使用所有CPU核心加速：
    ```bash
    # 编译（使用多核加速）
    make -j$(nproc)
    ```
    此过程若无错误，会在当前目录生成可执行文件。
    
    ```
    configure:  === Configuration results ===
    configure: Version:             1.4.0
    configure: c++ compiler:        g++ -std=c++17 -std=gnu++11
    configure: Warning CXXFLAGS:    -Wall
    configure: Picky CXXFLAGS:
    configure: Harden CFLAGS:        -fno-strict-overflow -D_FORTIFY_SOURCE=2 -fstack-protector-all -Wstack-protector --param ssp-buffer-size=1 -fPIE
    configure: Cryptography:        internal OCB, OpenSSL AES
    configure:  =============================
    ```
    
    
    
4.  **安装到系统**
    将编译好的程序安装到系统路径（默认是 `/usr/local/`）：
    
    ```bash
    # 运行测试（可选）
    make check
    
    # 安装
    sudo make install
    ```

###  第三阶段：验证与配置

1.  **刷新动态链接库缓存**
    安装后，建议运行以下命令以确保系统能找到新安装的库：
    
    ```bash
    sudo ldconfig
    ```
    
2.  **验证安装是否成功**
    执行以下命令，若显示版本号 `1.4.0` 则说明安装成功：
    
    ```bash
    mosh --version
    
    # 检查二进制文件
    which mosh
    which mosh-server
    
    # 查看版本
    mosh-server --version
    mosh-client --version
    
    # 测试基本功能
    mosh-server new
    
    # 连接测试（在另一个终端）
    # mosh-client <IP地址> <端口> <密钥>
    ```
    
3.  **配置服务器防火墙**
    这是 Mosh 能成功连接的**关键一步**。Mosh 使用 UDP 端口 60000-61000，你必须在防火墙中放行：
    
    ```bash
    # 如果使用 firewalld（openEuler 默认）
    sudo firewall-cmd --permanent --add-port=60000-61000/udp
    sudo firewall-cmd --reload
    
    # 如果使用 iptables
    sudo iptables -I INPUT -p udp --dport 60000:61000 -j ACCEPT
    # 并记得保存iptables规则
    ```

 4.**完整的安装验证脚本**

```shell
# 完整的安装验证脚本
#!/bin/bash
echo "1. 检查二进制文件:"
which mosh && which mosh-server
echo -e "\n2. 检查版本:"
mosh-server --version
echo -e "\n3. 检查动态依赖:"
ldd $(which mosh-server)
echo -e "\n4. 测试基本功能:"
timeout 2 mosh-server new 2>&1 | head -5
echo -e "\n5. 安装完成!"
```



###  第四阶段：使用与进阶玩法

**使用示例**

```
# 基本用法
mosh user@hostname

# 指定端口范围
mosh -p 60001:60010 user@hostname

# 通过SSH连接
mosh --ssh="ssh -p 2222" user@hostname
```

**卸载**

```
# 进入源码目录
cd mosh-1.4.0

# 卸载
sudo make uninstall

# 清理编译中间文件（保留可执行文件）
make clean

# 完全清理，回到配置前状态
# make distclean

# 清理配置文件
sudo rm -f /etc/ld.so.conf.d/mosh.conf
sudo ldconfig
```

#### 简化使用mosh连接远程主机的流程

**mosh通过ssh进行免密登录**

##### 为远程主机设置别名

##### **1. 配置SSH Config文件**

```shell
# 编辑SSH配置文件
mkdir -p ~/.ssh
chmod 700 ~/.ssh

cat >> ~/.ssh/config << 'EOF'
# 通用设置
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes
    Compression yes
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes

# 示例服务器配置
Host server1
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host server2
    HostName example.com
    User root
    Port 2222
    IdentityFile ~/.ssh/id_rsa
    # 使用代理跳板
    # ProxyJump jumpserver

# 通用模板
Host dev-*
    User developer
    IdentityFile ~/.ssh/dev_key

Host prod-*
    User deploy
    IdentityFile ~/.ssh/prod_key
EOF

# 设置正确的权限
chmod 600 ~/.ssh/config
```

##### **2. 配置mosh别名**

```shell
# 创建mosh专用别名文件
cat >> ~/.bashrc << 'EOF'
# Mosh别名
alias m1='mosh server1'
alias m2='mosh server2'
alias mdev='mosh dev-server'
alias mprod='mosh prod-server'

# 使用ssh config中的主机名
alias mssh='mosh --ssh="ssh -F ~/.ssh/config"'
EOF

source ~/.bashrc
```



### 故障排查

如果在 `./configure` 或 `make` 阶段失败，请检查：

*   **错误：`configure: error: Package requirements (protobuf >= 3.0.0) were not met`**
    *   **原因**：protobuf 库版本太低或未安装。
    *   **解决**：确保已执行 `sudo dnf install protobuf-devel`。openEuler 22.03+ 的仓库版本通常是够的。可通过 `pkg-config --modversion protobuf` 查看版本。

*   **错误：`fatal error: utempter.h: No such file or directory`**
    *   **原因**：缺少 `libutempter-devel`。
    *   **解决**：执行 `sudo dnf install libutempter-devel`。

*   **安装后运行 `mosh` 提示 “command not found”**
    *   **原因**：默认安装路径 `/usr/local/bin` 可能不在你的 `PATH` 中。
    *   **解决**：退出当前终端重新登录，或直接使用完整路径 `/usr/local/bin/mosh`。

###  总结与建议
*   **卸载**：如需卸载，在源码目录执行 `sudo make uninstall`（如果该目标存在），或手动删除相关文件。
*   **推荐方案**：除非有特定需求，**对于大多数用户，直接通过 `sudo dnf install mosh` 安装是更简单可靠的选择**。源码编译主要用于需要特定版本、定制功能或学习的目的。

如果在任何步骤遇到问题，请提供具体的错误信息，我可以为你提供更具体的解决方案。



---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/d0a3efc/  

