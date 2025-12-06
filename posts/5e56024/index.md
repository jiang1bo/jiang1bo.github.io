# 通过giscus为博客添加评论系统（Hugo+fixlt+giscus）




一个 Hugo 网站的 Giscus 评论系统配置。

## 配置概览

这是一个 **FixIt 主题**（v0.2.14+）的 Giscus 评论系统配置。

<!--more-->

### 关键参数分析

```
# 基础配置
enable = false                # 是否启用 Giscus，当前为关闭状态
repo = "jiang1bo/jiang1bo"   # GitHub 仓库：用户 jiang1bo 下的 jiang1bo 仓库
repoId = "xxxxxOQgMS0A"      # GitHub 仓库的唯一 ID
category = "General"         # 讨论分类名称
categoryId = "xxxxxwDOQgMS0M4CzSCX"  # 分类的唯一 ID

# 映射设置
mapping = "pathname"         # 页面映射方式：使用路径名作为标识
origin = "https://giscus.app" # Giscus 客户端源（FixIt 0.3.7+ 支持自托管）

# 功能开关
strict = "0"                 # 严格模式：0=不严格，1=严格匹配
reactionsEnabled = "1"       # 是否启用表情反应：1=启用
emitMetadata = "0"           # 是否发送元数据：0=不发送
inputPosition = "bottom"     # 评论框位置：bottom（底部），可选 top（顶部）

# 主题配置
lightTheme = "light"         # 亮色主题名称
darkTheme = "dark"           # 深色主题名称
lazyLoad = true              # 是否懒加载评论组件
```

## 工作原理

### 1. **Giscus 是什么？**
- 基于 GitHub Discussions 的评论系统
- 开源、免费、无需数据库
- 使用 GitHub 账号登录评论

### 2. **配置步骤分析**

你需要已经完成了以下步骤：
1. 创建了 GitHub 仓库 `jiang1bo/jiang1bo`
2. 启用了 GitHub Discussions 功能
3. 安装了 Giscus App 到仓库
4. 获取了 `repoId` 和 `categoryId`

### 3. **如何获取这些 ID？**

**方法一：通过 Giscus 官网**
```
https://giscus.app/
```
输入仓库 URL 后会自动生成配置

**方法二：通过 GitHub API**

```
# 获取 repoId
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/jiang1bo/jiang1bo

# 获取 categoryId
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/jiang1bo/jiang1bo/discussions/categories
```

## 优化建议

### 1. **启用配置**
```
enable = true  # 改为 true 以启用评论系统
```

### 2. **添加语言配置**
```
lang = "zh-CN"  # 设置评论界面语言
```

### 3. **优化加载策略**
```
lazyLoad = true  # 已启用，减少首屏加载时间
```

### 4. **安全配置建议**
```
# 如果你有自部署的 Giscus，可以修改 origin
origin = "https://giscus.yourdomain.com"  # 自托管地址
```

### 5. **完整的优化配置示例**
```
[params.page.comment.giscus]
  enable = true
  repo = "jiang1bo/jiang1bo"
  repoId = "xxxxDOQgMS0A"
  category = "Announcements"  # 可以创建专用分类
  categoryId = "DIC_xxxxgMS0M4CzSCX"
  mapping = "pathname"
  origin = "https://giscus.app"
  strict = "0"
  term = ""  # 搜索关键词，留空使用默认
  reactionsEnabled = "1"
  emitMetadata = "0"
  inputPosition = "bottom"
  lightTheme = "light"  # 可选：light、light_high_contrast、light_protanopia
  darkTheme = "dark"    # 可选：dark、dark_high_contrast、dark_protanopia
  darkDimmed = "dark_dimmed"  # 可添加更多主题
  lazyLoad = true
  lang = "zh-CN"  # 设置语言
```

## 故障排查

如果评论不显示，检查以下内容：

1. **仓库权限**：确保仓库是公开的
2. **Discussions 已启用**：仓库 Settings → Features → Discussions
3. **Giscus App 已安装**：仓库 Settings → Integrations → Giscus
4. **分类存在**：确保 "General" 分类在 Discussions 中存在

## 性能影响

**优点**：
- ✅ 无数据库依赖
- ✅ 使用 GitHub 账号，无需注册
- ✅ 支持 Markdown 格式
- ✅ 响应式设计

## 推荐设置

对于个人博客，建议：

1. **保留懒加载**：`lazyLoad = true`
2. **启用表情反应**：`reactionsEnabled = "1"`
3. **输入框在底部**：`inputPosition = "bottom"`
4. **使用 pathname 映射**：`mapping = "pathname"`

只需要将 `enable` 改为 `true` 即可使用。


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/5e56024/  

