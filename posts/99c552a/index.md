# K3s 常见使用问题、排查与解决总结


k3s 常见使用问题、排查与解决总结

<!--more-->

## 一、镜像拉取失败：ImagePullBackOff

### 现象

- Pod 状态：`ImagePullBackOff` 或 `ErrImagePull`
- 日志报错：
  - `read: network is unreachable`（IPv6 不可达）
  - `lookup docker.m.daocloud.io: Try again`（DNS 解析失败）
  - `dial tcp ... i/o timeout`（连接超时）

### 排查命令

```bash
kubectl get pods -A
# kubectl：Kubernetes 命令行工具
# get pods：获取 Pod 列表
# -A：--all-namespaces，查看所有命名空间

kubectl -n local-ai describe pod <pod-name>
# -n local-ai：指定命名空间 local-ai
# describe pod：查看 Pod 详细信息
# <pod-name>：替换为实际 Pod 名称
# 重点看 Events 部分，会显示 Failed to pull image 的具体原因

sudo journalctl -u k3s -n 200 | grep -i ollama
# sudo：以 root 权限执行
# journalctl：查看 systemd 日志
# -u k3s：指定 k3s 服务
# -n 200：显示最近 200 行
# grep -i ollama：忽略大小写过滤包含 ollama 的行

sudo k3s ctr images list | grep ollama
# sudo：以 root 权限执行
# k3s ctr：k3s 内置的 containerd 命令行工具
# images list：列出本地镜像
# grep ollama：过滤 ollama 相关镜像
```

### 原因

1. 虚拟机 IPv6 不可达，k3s 尝试通过 IPv6 访问 Docker Hub 失败。
2. 国内镜像加速器 `docker.m.daocloud.io` 域名解析失败。
3. WARP 客户端干扰 DNS 和路由。
4. `imagePullPolicy` 为 `Always`，即使本地有镜像也强制重新拉取。

### 解决方案

#### 1. 断开 WARP，修复 DNS

```bash
warp-cli disconnect
# warp-cli：Cloudflare WARP 命令行工具
# disconnect：断开 WARP 连接

echo -e "nameserver 223.5.5.5\nnameserver 119.29.29.29" | sudo tee /etc/resolv.conf
# echo -e：输出字符串，-e 启用转义
# nameserver 223.5.5.5：指定阿里 DNS
# nameserver 119.29.29.29：指定腾讯 DNS
# |：管道，将输出传给 tee
# sudo tee /etc/resolv.conf：以 root 权限写入 DNS 配置文件

ping -c 3 baidu.com
# ping：测试网络连通性
# -c 3：发送 3 个 ICMP 包
# baidu.com：目标域名
```

#### 2. 配置 k3s 镜像加速器

```bash
sudo mkdir -p /etc/rancher/k3s
# mkdir -p：创建目录，-p 表示递归创建且已存在不报错
# /etc/rancher/k3s：k3s 配置目录

sudo tee /etc/rancher/k3s/registries.yaml << 'EOF'
mirrors:
  docker.io:
    endpoint:
      - "https://docker.1panel.live"
      - "https://docker.m.daocloud.io"
      - "https://docker.1ms.run"
EOF
# tee：写入文件
# << 'EOF'：Here Document，多行输入直到 EOF
# mirrors：镜像加速配置
# docker.io：对 Docker Hub 的请求进行加速
# endpoint：加速器地址列表，按顺序尝试

sudo systemctl restart k3s
# systemctl：系统服务管理工具
# restart k3s：重启 k3s 服务使配置生效
```

#### 3. 手动拉取并导入镜像（最可靠）

```bash
podman pull ollama/ollama:latest
# podman：容器引擎
# pull：拉取镜像
# ollama/ollama:latest：镜像名称和标签

podman save --format docker-archive -o /tmp/ollama-image.tar ollama/ollama:latest
# save：导出镜像
# --format docker-archive：指定导出格式为 docker-archive
# -o /tmp/ollama-image.tar：输出文件路径
# ollama/ollama:latest：要导出的镜像

sudo k3s ctr images import /tmp/ollama-image.tar
# k3s ctr：k3s 的 containerd 工具
# images import：导入镜像 tar 文件
# /tmp/ollama-image.tar：镜像归档文件路径

sudo k3s ctr images list | grep ollama
# 验证镜像是否导入成功
```

#### 4. 修改 imagePullPolicy

```yaml
# 在 Deployment YAML 中：
containers:
  - name: ollama
    image: ollama/ollama:latest
    imagePullPolicy: IfNotPresent   # 本地有则用本地，无则拉取
kubectl apply -f ollama-k3s.yaml
# apply：应用 YAML 配置
# -f：指定文件名

kubectl -n local-ai delete pod -l app=ollama
# delete pod：删除 Pod
# -l app=ollama：按标签选择器删除
```

#### 5. 预导入镜像到 k3s（防重启丢失）

```bash
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
# 创建 k3s 自动导入镜像目录

sudo cp /tmp/ollama-image.tar /var/lib/rancher/k3s/agent/images/
# cp：复制文件到该目录
# k3s 启动时会自动加载该目录下的 tar 镜像
```

------

## 二、容器启动失败：RunContainerError / CrashLoopBackOff

### 现象

- Pod 状态：`RunContainerError` 或 `CrashLoopBackOff`
- 报错：`exec: "uvicorn": executable file not found in $PATH`

### 排查命令

```bash
kubectl -n local-ai describe pod <pod-name>
# 查看 Pod 详情，重点看 Last State 和 Events

kubectl -n local-ai logs <pod-name> --previous
# logs：查看容器日志
# --previous：查看上一个已终止容器的日志
```

### 原因

`requirements.txt` 中缺少 `fastapi`、`uvicorn` 等 Python 依赖，镜像内没有安装 `uvicorn` 可执行文件。

### 解决方案

1. 补全 `requirements.txt`：

```txt
fastapi==0.115.0
uvicorn[standard]==0.32.0
requests==2.32.3
protobuf==4.21.12
```

1. 重新构建镜像：

```bash
podman build -t localhost/k3s-ollama-chat:0.1.1 .
# build：构建镜像
# -t：指定镜像标签
# localhost/k3s-ollama-chat:0.1.1：新版本标签
# .：构建上下文为当前目录
```

1. 导出并导入 k3s：

```bash
podman save --format docker-archive -o /tmp/k3s-ollama-chat-0.1.1.tar localhost/k3s-ollama-chat:0.1.1
# 导出新镜像

sudo k3s ctr images import /tmp/k3s-ollama-chat-0.1.1.tar
# 导入到 k3s
```

1. 修改 YAML 中的镜像标签并重建：

```bash
kubectl apply -f deploy/k3s.yaml
# 应用更新后的 YAML

kubectl -n local-ai rollout restart deployment/ollama-chat
# rollout restart：滚动重启 Deployment
```

------

## 三、镜像名称不匹配：ErrImageNeverPull

### 现象

- Pod 状态：`ErrImageNeverPull`
- `imagePullPolicy: Never`，但本地找不到完全匹配的镜像名。

### 排查命令

```bash
sudo k3s ctr images list | grep ollama-chat
# 查看 k3s 中实际镜像名，通常带 localhost/ 前缀
```

### 原因

`podman save` 导入后镜像名变为 `localhost/k3s-ollama-chat:0.1.0`，而 YAML 中写的是 `k3s-ollama-chat:0.1.0`，名称不匹配。

### 解决方案

**方案一：修改 YAML 镜像名**

```yaml
image: localhost/k3s-ollama-chat:0.1.0
imagePullPolicy: Never   # 或 IfNotPresent
kubectl apply -f deploy/k3s.yaml
kubectl -n local-ai delete pod -l app.kubernetes.io/name=ollama-chat
```

**方案二：给 k3s 镜像重新打标签**

```bash
sudo k3s ctr images tag localhost/k3s-ollama-chat:0.1.0 k3s-ollama-chat:0.1.0
# tag：为镜像创建新标签
# 原镜像 新标签
```

------

## 四、metrics-server 无端点：no endpoints available

### 现象

- k3s 日志反复报：

  ```
  error resolving kube-system/metrics-server: no endpoints available for service "metrics-server"
  ```

- `kubectl top nodes` 失败或 metrics-server 就绪探针返回 500。

### 排查命令

```bash
kubectl -n kube-system get pods
# 查看 metrics-server Pod 是否 Running/Ready

kubectl -n kube-system describe pod -l k8s-app=metrics-server
# 查看就绪探针失败信息

kubectl -n kube-system logs deployment/metrics-server
# 查看 metrics-server 日志，常见 Failed to scrape node

kubectl -n kube-system get endpoints metrics-server
# 查看 Service 端点是否为空

kubectl get apiservice v1beta1.metrics.k8s.io
# 查看 metrics API 是否可用
```

### 原因

k3s 使用自签名 kubelet 证书，metrics-server 无法验证证书，导致抓取指标失败，就绪探针不通过，Service 无端点。

### 解决方案

使用 `kubectl patch` 添加 `--kubelet-insecure-tls` 参数：

```bash
kubectl -n kube-system patch deployment metrics-server --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
# patch：修改资源
# -n kube-system：指定命名空间
# deployment metrics-server：目标 Deployment
# --type=json：使用 JSON Patch
# -p：patch 数据
# op:add：添加操作
# path:/spec/template/spec/containers/0/args/-：在 args 数组末尾追加
# value:--kubelet-insecure-tls：添加的参数
```

验证：

```bash
kubectl -n kube-system rollout status deployment/metrics-server
# 等待滚动更新完成

kubectl top nodes
# 测试指标是否可用
```

------

## 五、编辑 Deployment 报 invalid：YAML 格式错误

### 现象

```bash
kubectl -n kube-system edit deployment metrics-server
error: deployments.apps "metrics-server" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-xxx.yaml"
```

### 原因

手动编辑 YAML 时缩进或列表符号 `-` 错误。

### 解决方案

**推荐用 patch 命令代替手动编辑**（见上一节）。
 如果必须编辑，先导出再修改：

```bash
kubectl -n kube-system get deployment metrics-server -o yaml > /tmp/ms.yaml
# 导出为 YAML 文件

nano /tmp/ms.yaml
# 用 nano 编辑，注意缩进

kubectl apply -f /tmp/ms.yaml
# 应用修改
```

------

## 六、端口转发开机自启（Grafana 内网访问）

### 目标

让内网通过 `http://192.168.8.80:3000` 访问 Grafana，且开机自动启动。

### 方案：systemd 服务

```bash
sudo nano /etc/systemd/system/grafana-portforward.service
# 创建 systemd 服务文件
```

文件内容：

```ini
[Unit]
Description=Port-forward for Grafana Service
# 确保在 K3s 和网络就绪后再启动此服务
After=network-online.target k3s.service
Wants=network-online.target k3s.service

[Service]
# 使用你的普通用户 fermi 来运行，避免权限问题
User=fermi
# 设置用户的家目录和 Kubectl 的配置文件路径
Environment=HOME=/home/fermi
Environment=KUBECONFIG=/home/fermi/.kube/config
Environment=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# 核心命令：将本地 3000 端口转发到 Grafana 服务
# --address 0.0.0.0 表示允许内网其他设备访问
ExecStart=/usr/local/bin/kubectl -n monitoring port-forward svc/prometheus-grafana 3000:80 --address 0.0.0.0

# 无论何种原因退出，总是在 5 秒后重启，保证服务高可用
Restart=always
RestartSec=5

[Install]
# 定义为多用户模式下启动
WantedBy=multi-user.target
```

启用并启动：

```bash
sudo systemctl daemon-reload
# 重新加载 systemd 配置

sudo systemctl enable grafana-portforward.service
# 设置开机自启

sudo systemctl start grafana-portforward.service
# 立即启动

sudo systemctl status grafana-portforward.service
# 查看状态
```

------

## 八、常用命令速查（含注释）

```bash
# 查看所有 Pod
kubectl get pods -A
# -A：所有命名空间

# 查看 Pod 详细信息
kubectl -n local-ai describe pod <pod-name>
# -n：命名空间
# describe：详细信息

# 查看容器日志
kubectl -n local-ai logs <pod-name> --previous
# --previous：上一个容器实例日志

# 查看 Deployment 滚动状态
kubectl -n local-ai rollout status deployment/ollama
# rollout status：等待滚动更新完成

# 重启 Deployment
kubectl -n local-ai rollout restart deployment/ollama-chat
# rollout restart：触发滚动重启

# 删除 Pod
kubectl -n local-ai delete pod -l app=ollama
# -l：标签选择器

# 删除 ReplicaSet
kubectl -n local-ai delete rs <rs-name>
# rs：ReplicaSet 缩写

# 查看 k3s 镜像列表
sudo k3s ctr images list
# ctr：containerd 命令行工具

# 导入镜像
sudo k3s ctr images import /tmp/image.tar
# import：导入 tar 镜像

# 给镜像打标签
sudo k3s ctr images tag <old> <new>
# tag：创建新标签

# 查看 k3s 服务状态
sudo systemctl status k3s
# systemctl：服务管理

# 重启 k3s
sudo systemctl restart k3s
# restart：重启服务

# 查看节点指标
kubectl top nodes
# top：查看资源使用

# 应用 YAML
kubectl apply -f <file>.yaml
# apply：应用配置

# 查看 APIService
kubectl get apiservice v1beta1.metrics.k8s.io
# apiservice：聚合 API 服务
```

## 九、CoreDNS Pod 处于 `CrashLoopBackOff` 状态

退出码 `1` 通常意味着 **CoreDNS 进程在启动时因配置解析错误或上游 DNS 不可达而立即退出**。需要先查看日志确认具体原因。

------

## 🛠️ 第一步：查看 CoreDNS 日志（定位根因）

```bash
# 查看 CoreDNS 容器日志，--previous 表示查看上一个已崩溃容器的日志
kubectl -n kube-system logs <coredns-pod-name> --previous

# 如果 --previous 没有输出，查看当前日志
kubectl -n kube-system logs <coredns-pod-name>

# 也可以按标签批量查看
kubectl -n kube-system logs -l k8s-app=kube-dns --tail=100
```

**重点关注以下报错关键词**：

| 日志关键词                                | 含义              | 修复方向                                      |
| ----------------------------------------- | ----------------- | --------------------------------------------- |
| `plugin/forward: no healthy upstream`     | 上游 DNS 不可达   | 检查 `/etc/resolv.conf` 中的 DNS 地址是否有效 |
| `Loop detected`                           | 转发循环          | Corefile 中的 forward 指向了自身              |
| `Error during parsing: Unknown directive` | Corefile 语法错误 | 修正 ConfigMap 中的 Corefile                  |
| `Failed to list *v1.Service` / `403`      | RBAC 权限问题     | 检查 ServiceAccount 权限                      |
| `bind: address already in use`            | 端口 53 被占用    | 检查宿主机是否有其他 DNS 服务                 |

------

## 🛠️ 第二步：根据日志内容选择修复方案

### 方案 A：上游 DNS 不可达（最常见）

如果日志显示 `no healthy upstream` 或 `timeout`，说明 CoreDNS 无法访问上游 DNS 服务器。这与你之前遇到的 WARP 干扰 DNS 问题一致。

**重启 CoreDNS：**

```bash
# 删除 CoreDNS Pod，让 ReplicaSet 自动重建
kubectl -n kube-system delete pod -l k8s-app=kube-dns

# 观察重建后的状态
kubectl -n kube-system get pods -l k8s-app=kube-dns -w
```

### 方案 B：Corefile 配置错误

如果日志显示 `Error during parsing` 或 `Unknown directive`，说明 Corefile 有语法错误。

```bash
# 查看当前 CoreDNS 的 ConfigMap
kubectl -n kube-system get configmap coredns -o yaml

# 编辑 ConfigMap（注意 YAML 缩进）
kubectl -n kube-system edit configmap coredns
```

**正确的 Corefile 参考**（注意缩进，`{` 不能与 `.:53` 粘连）：

```
.:53 {
    errors
    health {
        lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
        ttl 30
    }
    prometheus :9153
    forward . 223.5.5.5 119.29.29.29
    cache 30
    loop
    reload
    loadbalance
}
```

> **注意**：`forward . 223.5.5.5 119.29.29.29` 中的 DNS 地址必须与宿主机 `/etc/resolv.conf` 中的一致，且不能是 `127.0.0.1` 或 `127.0.0.53`（loopback 地址不可达）。

修改后等待 CoreDNS 自动重载，或手动删除 Pod：

```bash
kubectl -n kube-system delete pod -l k8s-app=kube-dns
```

### 方案 C：资源不足（OOMKilled）

如果 `describe pod` 中 `Last State` 显示 `Reason: OOMKilled`，说明内存不足。

```bash
# 查看当前内存限制
kubectl -n kube-system get deployment coredns -o yaml | grep -A2 memory

# 用 patch 命令将内存限制提升到 256Mi
kubectl -n kube-system patch deployment coredns --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/resources/limits/memory","value":"256Mi"}]'
```

### 方案 D：就绪探针配置缺失（ready 插件问题）

你的就绪探针访问 `8181/ready` 连接被拒绝，可能是因为 Corefile 中缺少 `ready` 插件。

```bash
# 检查 Corefile 中是否有 ready 插件
kubectl -n kube-system get configmap coredns -o yaml | grep -i ready

# 如果没有，编辑 ConfigMap 添加
kubectl -n kube-system edit configmap coredns
```

在 `.:53 {` 块内添加一行：

```
    ready
```

然后删除 Pod 重建：

```bash
kubectl -n kube-system delete pod -l k8s-app=kube-dns
```

------

## 🛠️ 第三步：强制清理重启（通用兜底方案）

如果上述方案都无效，可以强制清理 CoreDNS 的 ReplicaSet 和 Pod：

```bash
# 将副本数缩放为 0（停止所有 CoreDNS Pod）
kubectl -n kube-system scale deployment coredns --replicas=0

# 等待 10 秒
sleep 10

# 将副本数缩放回 2（重新创建 Pod）
kubectl -n kube-system scale deployment coredns --replicas=2

# 观察重建状态
kubectl -n kube-system get pods -l k8s-app=kube-dns -w
```

> **注意**：缩放期间集群 DNS 不可用，所有 Pod 的域名解析会短暂中断，但现有连接不受影响。

------

## ✅ 第四步：验证修复

```bash
# 1. 确认 CoreDNS Pod 状态为 Running，READY 为 1/1
kubectl -n kube-system get pods -l k8s-app=kube-dns

# 2. 确认 Service 端点已填充
kubectl -n kube-system get endpoints kube-dns

# 3. 用临时 Pod 测试 DNS 解析
kubectl run dns-test --image=busybox:1.28 --rm -it --restart=Never -- nslookup kubernetes.default
```

如果第三步能成功解析 `kubernetes.default`，说明 CoreDNS 已恢复正常。

------

## 📋 命令速查（含注释）

```bash
# 查看 CoreDNS Pod 状态
kubectl -n kube-system get pods -l k8s-app=kube-dns
# -n kube-system：指定命名空间
# -l k8s-app=kube-dns：按标签过滤

# 查看上一个崩溃容器的日志
kubectl -n kube-system logs <pod-name> --previous
# --previous：查看上一次运行（已终止）的容器日志

# 查看 Pod 详细事件
kubectl -n kube-system describe pod <pod-name>
# 重点看 Events 和 Last State 的 Reason

# 编辑 CoreDNS ConfigMap
kubectl -n kube-system edit configmap coredns
# 保存退出后 CoreDNS 会自动重载（通过 reload 插件）

# 删除 Pod 触发重建
kubectl -n kube-system delete pod -l k8s-app=kube-dns
# 删除后 ReplicaSet 会自动创建新 Pod

# 缩放副本数（强制清理）
kubectl -n kube-system scale deployment coredns --replicas=0
kubectl -n kube-system scale deployment coredns --replicas=2
# --replicas：指定副本数

# 测试 DNS 解析
kubectl run dns-test --image=busybox:1.28 --rm -it --restart=Never -- nslookup kubernetes.default
# --rm：退出后自动删除 Pod
# -it：交互式终端
# --restart=Never：不自动重启
# -- nslookup ...：容器内执行的命令
```

## 

## 九、核心经验总结

1. **镜像拉取策略**：`latest` 标签默认 `imagePullPolicy: Always`，网络不稳时改为 `IfNotPresent` 或 `Never`。
2. **镜像名称前缀**：`podman save` + `k3s ctr images import` 后镜像名通常带 `localhost/`，YAML 必须同步。
3. **依赖管理**：`requirements.txt` 只放 PyPI 包，系统包用 `apt-get` 安装，避免 `pip install` 失败。
4. **metrics-server**：k3s 自签名证书需加 `--kubelet-insecure-tls`，推荐用 `kubectl patch` 修改。
5. **网络干扰**：WARP 会干扰 DNS 和路由，构建/拉取镜像前先断开。
6. **持久化端口转发**：用 systemd 服务守护 `kubectl port-forward --address 0.0.0.0`，实现开机自启和内网访问。
7. **排查顺序**：`get pods` → `describe pod` → `logs` → `get endpoints` → 检查网络/DNS/证书。

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/99c552a/  

