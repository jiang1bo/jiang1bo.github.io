# Ssh常见配置和操作命令




## ssh 利用非对称加密实现安全的登录

非对称加密中有两个密钥：公钥和私钥。公钥由私钥产生，但却无法推算出私钥；公钥加密后的密文，只能通过对应的私钥来解密。

<!--more-->

终端要登录 Server 服务器，发起登录请求 ssh work@server.com。服务端运行有 sshd 服务，并监听指定的端口，默认为 22 号端口。

服务端会生成一对公钥和私钥；此时将公钥返回给客户端；客户端使用公钥，对登录密码进行加密（如服务器work用户密码为xxx），生成公钥加密字符串；客户端将公钥加密字符串发送给服务端；服务端使用私钥，解密公钥加密字符串，得到原始密码；校验密码是否合法（此为本机 work 密码）；返回登录结果给客户端：成功登录或密码错。

**在非对称加密中，由于只有公钥会被传输，而私钥是服务端本地保存，因此即便公钥被监听，也无法拿到原始密码，从而安全地登录服务器。**

```css
使用指定用户名和端口登录远程主机。ssh -p3600 root@9.134.114.170
```

### **SSH 服务的配置文件**

**通常位于 `/etc/ssh/sshd_config`。可以使用文本编辑器编辑该文件以调整设置。常见配置选项包括：**

```shell
Port：指定 SSH 服务监听的端口（默认为 22）。Port 22
PermitRootLogin：控制是否允许 root 用户通过 SSH 登录。出于安全考虑，通常建议设置为 no。 PermitRootLogin no
PasswordAuthentication：控制是否允许密码认证。启用密钥认证时，可以将此选项设置为 no。 PasswordAuthentication yes
AllowUsers 和 DenyUsers：控制允许和拒绝登录的用户。AllowUsers user1 user2

## 修改配置文件后，使用以下命令重启 SSH 服务使更改生效：sudo systemctl restart ssh
```

### **通过 ssh 实现免密登录。**



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

```shell
##第一步使用 ssh-keygen 命令在客户端生成 RSA 公钥和私钥，一直回车确认。公钥和私钥默认名称为 id_rsa.pub（公钥）和私钥（id_rsa），默认保存在 ~/.ssh/目录下。
ssh-keygen -t rsa

##第二步将客户端公钥追加至服务端 ~/.ssh/authorized_keys 文件中，authorized_keys 是用来存放客户端公钥的文件。

##有三种方法，
#一是通过 ssh-copy-id 命令，
#二是通过 scp 命令，
#三是手动复制。例如使用 ssh-copy-id 命令实现如下
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 3600 root@9.134.114.170

##第三步使用 ssh 进行免密登录。
ssh -p3600 root@9.134.114.170
```

### **端口转发**

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


---

> 作者: <no value>  
> URL: http://localhost:1313/posts/de36ed4/  

