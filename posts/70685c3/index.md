# 个人建站总结


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->



# 一，一个由hugo+fixlt搭建的博客网站

<img src=".\assets\1.png"  title="blog images" style="zoom:67%;"  />



## **使用hugo搭建个人网站**

建站步骤：[FixIt (lruihao.cn)](https://fixit.lruihao.cn/zh-cn/)

## 1.先决条件

**技巧：如果这是你第一次使用 [Hugo](https://gohugo.io/)，我们强烈建议你通过阅读这篇 [入门文档](https://gohugo.io/getting-started/) 来更多地了解它。**

在开始本教程之前，你必须：

1. [安装 Hugo](https://gohugo.io/getting-started/installing/)（扩展版，v0.147.7 或更高版本）
2. [安装 Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

你还必须熟悉使用命令行。

## 2.创建网站

提示：接下来的步骤将一步步指导你创建一个 Hugo 网站。
如果你已经熟悉 Hugo，可以跳过这些步骤，使用 [FixIt CLI](https://fixit.lruihao.cn/zh-cn/documentation/installation/#cli) 来快速创建一个网站。

命令：验证你是否安装了 Hugo v0.147.7 或更高版本。

```
hugo version
```

运行以下命令来创建一个带有 [FixIt](https://github.com/hugo-fixit/FixIt) 主题的 Hugo 网站。

### 1.1为你的项目创建网站

```go
#在 my-blog 目录中为你的项目创建网站
hugo new site my-blog

git init

#将 FixIt 主题作为 Git 子模块 添加到你的项目中的 themes 目录。
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
#在站点配置文件中添加一行，指定当前主题。
echo "theme = 'FixIt'" >> hugo.toml
#在站点配置文件中添加一行，指定默认的内容语言。
echo "defaultContentLanguage = 'zh-cn'" >> hugo.toml
#启动 Hugo 的开发服务器以查看站点。
hugo server
#按 Ctrl + C 停止 Hugo 的开发服务器。
```

### 1.2给你的网站添加新页面

```go
1.（第一种方式）给你的网站添加新页面。
hugo new content posts/my-first-post.md

2.（第二种方式）Hugo 支持 Page bundles 模式，即文章内容打包在一个文件夹下，内部可以独立包含图片、子页面等静态资源，文章以 index.md 作为入口，可基于 FixIt 主题提供的 post-bundle 原型进行自定义修改：
# 拷贝主题原型到站点目录
$ cp -r themes/FixIt/archetypes/post-bundle archetypes
# 增加博客图片目录，最终结构如下，注意空目录在原型使用时不会生效
$ tree -a archetypes/post-bundle
archetypes/post-bundle
├── images
│   └── .gitkeep
└── index.md

# 创建 Page bundles 文章
$ hugo new content --kind post-bundle posts/hello-world
# 文章目录结构
$ tree -a content/posts/hello-world
content/posts/hello-world
├── images
│   └── .gitkeep
└── index.md

# 添加 markdown 内容
$ echo "\n## Hello World" >> content/posts/hello-world/index.md

#Hugo 在 content/posts 目录中创建了该文件，使用编辑器打开文件。

请注意，front matter 中的 draft 值为 true。默认情况下，Hugo 在你构建网站时不会发布草稿内容。详细了解 草稿、未来和过期内容。
在帖子正文中添加一些 Markdown，但不要更改 draft 值。
---
title: 我的第一篇文章
date: 2024-03-01T17:10:04+08:00
draft: true
# ...
---
#保存文件，然后启动 Hugo 的开发服务器来查看站点。你可以运行以下任一命令来包含草稿内容

hugo server --buildDrafts
hugo server -D
hugo server -D --disableFastRender

#由于本主题使用了 Hugo 中的 .Store 来实现一些特性， 非常建议你为 hugo server 命令添加 --disableFastRender 参数来实时预览你正在编辑的文章页面。
#当对新内容感到满意时，将 front matter 中的 `draft` 值更改为 `false`，然后保存文件。
```





### 1.3发布网站

在此步骤中，你将发布你的网站，但不会 *部署* 它。

当你发布站点时，Hugo 在项目根目录的 `public` 目录中创建整个静态站点。这包括 HTML 文件以及图像、CSS 文件和 JavaScript 文件等资源。

当你发布网站时，你通常不希望包含 [草稿、未来或过期的内容](https://gohugo.io/getting-started/usage/#draft-future-and-expired-content)，命令很简单。

```
hugo
```

###  1.4文档指南

强烈建议你花少量时间完整阅读 FixIt 主题的文档，以便你更好地了解如何使用它。

1. [安装篇](https://pre.fixit.lruihao.cn/zh-cn/documentation/installation/)
2. [入门篇](https://pre.fixit.lruihao.cn/zh-cn/documentation/getting-started/)
3. [内容管理](https://pre.fixit.lruihao.cn/zh-cn/documentation/content-management/)
4. [进阶篇](https://pre.fixit.lruihao.cn/zh-cn/documentation/advanced/)

### 1.5一个全局toml模板：

```toml
baseURL = 'https://jiang1bo.github.io/'
title = 'Fermi'
#languageCode = 'en-us' #en-us
theme = 'FixIt' 
defaultContentLanguage = 'zh-CN' 

[languages.zh-CN]
languageCode = 'zh-CN'
weight = 1

#title
#map 页面头部导航栏标题配置。
#logo: string LOGO 的 URL。
#name: string 标题名称。
#pre: string 在名称之前添加其他信息。
#post: string 在名称之后添加其他信息。
#typeit: bool 是否为标题显示打字机动画。
    [params.header.title]
      logo = ""
      name = "Fermi's Blog"
      pre = ""
      post = ""
      typeit = false
#页面头部导航栏副标题配置。
#name: string 副标题名称。
#typeit: bool 是否为副标题显示打字机动画。
    [params.header.subtitle]
      name = ""
      typeit = false

#作者设置
  [params.author]
    name = "fermi"
    email = "3044483124@qq.com"
    link = "https://jiangbos.top/"
    avatar = "./web-app-manifest-512x512.png"


#主页设置
#enable: bool 是否显示个人信息，默认：false。
#gravatarEmail: stringGravatar 邮箱，用于优先在主页显示的头像。
#avatarURL: string 主页显示头像的 URL。FixIt 0.2.17 | 新增
#avatarMenu: string 头像菜单链接的 identifier。
#title: string 主页显示的网站标题（支持 HTML 格式）。
#subtitle: string 主页显示的网站副标题。
#typeit: bool 是否为副标题显示打字机动画，默认：true。
#social: bool 是否显示社交账号，默认：true。
#disclaimer: string 免责声明（支持 HTML 格式）。
#posts
#map 主页文章列表配置。
#enable: bool 是否显示文章列表，默认：true。
#paginate: int 主页每页显示文章数量，默认：6。
[params.home]
    paginate = 10
    [params.home.profile]
      enable = true
      gravatarEmail = ""
      avatarURL = "favicon-96x96.png"
      avatarMenu = ""
      title = "你好,这里是ferm，一名python & linux开发工程师， 热爱技术和分享。喜欢探索新技术，解决实际问题。在这个博客中，我分享我的学习笔记、项目经验和生活感悟。"
      subtitle = "“如果你来访我，我不在，请和我门外的花坐一会儿，它们很温暖，我注视它们很多很多日子了。” ——汪曾祺《人间草木》"
      typeit = true
      social = true
      disclaimer = ""
    [params.home.posts]
      enable = true
      paginate = 6

#typeit map 打字机动画配置。
#speed int 打字速度，默认：100。
#cursorSpeed int 光标速度，默认：1000。
#cursorChar string 光标字符，默认：|。
#duration int 动画持续时间，默认：-1。
#loop FixIt 0.2.18 | 新增 bool 是否循环播放，默认：false。

  [params.typeit]
    speed = 100
    cursorSpeed = 1000
    cursorChar = "|"
    duration = -1
    loop = false


# LoveIt 新增 | 0.2.0 应用图标配置
  [params.app]
    # 当添加到 iOS 主屏幕或者 Android 启动器时的标题, 覆盖默认标题
    title = "Fermi's Blog"
    # 是否隐藏网站图标资源链接
    noFavicon = false
    # 更现代的 SVG 网站图标, 可替代旧的 .png 和 .ico 文件
    svgFavicon = "favicon.svg"
    # Android 浏览器主题色
    themeColor = "#ffffff"
    # Safari 图标颜色
    iconColor = "#5bbad5"
    # Windows v8-10磁贴颜色
    tileColor = "#da532c"


#可以参考位于 themes/FixIt/assets/data/social.yaml 的默认数据来配置你的社交链接。
#你可以直接配置你的社交 ID 来生成一个默认社交链接和图标：
#  Mastodon = "@xxxx"
#生成的社交链接是 https://mastodon.social/@xxxx。
#或者你可以通过一个字典来设置更多的选项：
#[params.social]
#  [params.social.Mastodon]
#    # 排列图标时的权重（权重越大，图标的位置越靠后）
#    weight = 0
#    # 你的社交 ID
#   id = "@xxxx"
#    # 你的社交链接的前缀
#    prefix = "https://mastodon.gal/"
#    # 当鼠标停留在图标上时的提示内容
#     title = "Mastodon" 

# custom social links like the following
# [params.social.twitter]
#   id = "lruihao"
#   weight = 3
#   prefix = "https://twitter.com/"
#   Title = "Twitter"
#   [social.twitter.icon]
#     class = "fa-brands fa-x-twitter fa-fw"

# FixIt 0.2.16 | CHANGED Social config about the author
  [params.social]
    GitHub = "jiang1bo"    
    Wordpress = "jiangbos.top"       
    CSDN = "?spm=1038.2274.3001.4476"
    Email = "ktamdz@163.com"

    QQ = ""
    QQGroup = "" # https://qun.qq.com/join.html
    Feishu = ""
    Douyin = ""
    TikTok = ""
    Phone = ""

    Bilibili = "" 
    Weibo = ""    
    Zhihu = ""
    Paypal = ""    
    Steam = ""
    Twitch = ""   
    Whatsapp = ""
    Twitter = ""
    Instagram = ""
    Facebook = ""
    Telegram = ""
    Gitlab = ""
    Youtubelegacy = ""
    Youtubecustom = ""
    Youtubechannel = ""
    Reddit = ""
    RSS = false
    Discord = ""
    DiscordInvite = ""


#[params.social]
#    [params.social.csdn]
#    weight = 0
#    id = ""
#    prefix = ""
#    title = "CSDN"
#    [params.social.github]
#    weight = 2
#    id = "jiang1bo"
#    prefix = "https://github.com/"
#    title = "GitHub"
#    [params.social.bilibili]
#    weight = 3
#    id = ""
#    prefix = "https://bilibili.com/"
#    title = "bilibili"


# FixIt 0.2.17 | 新增 赞赏设置
    [params.page.reward]
      enable = true
      animation = true
      # 相对于页脚的位置，可选值：["before", "after"]
      position = "after"
      comment = "Buy me a coffee"
      # FixIt 0.2.18 | 新增 二维码图片展示模式，可选值：["static", "fixed"]，默认：`static`
      mode = "static"
      [params.page.reward.ways]
        wechatpay = "./web-app-manifest-512x512.png"
        # alipay = "/images/alipay.png"
        # paypal = "/images/paypal.png"
        # bitcoin = "/images/bitcoin.png"

# 文章页面的分享信息设置
    [params.page.share]
      enable = true
      Twitter = true
      Facebook = true
      Linkedin = false
      Whatsapp = false
      Pinterest = false
      Tumblr = false
      HackerNews = false
      Reddit = false
      VK = false
      Buffer = false
      Xing = false
      Line = false
      Instapaper = false
      Pocket = false
      Flipboard = false
      Weibo = true
      Myspace = false
      Blogger = false
      Baidu = false
      Odnoklassniki = false
      Evernote = false
      Skype = false
      Trello = false
      Mix = false
#githubCorner FixIt 0.2.14 | 新增map 在左上角或者右上角显示 GitHub 开源链接。
[params.githubCorner]
    enable = true
    permalink = "https://github.com/jiang1bo"
    title = "View source on GitHub"
    position = "right"




# FixIt 0.2.15 | 更改 评论系统设置
    [params.page.comment]
      enable = true
      # FixIt 0.1.1 | 新增 Gitalk 评论系统设置 (https://github.com/gitalk/gitalk)
      #[params.page.comment.gitalk]
      #  enable = true
      #  owner = "jiang1bo"
      #  repo = "jiang1bo"
      #  clientId = "xxx3li4c6tbHMoc0ABQD"
      #  clientSecret = "xxxf028b6a13cb0577027197179a83c97486b6b6"
      

      #  <script src="https://giscus.app/client.js"
      #  data-repo="jiang1bo/jiang1bo"
      #  data-repo-id="R_kgDOQgMS0A"
      #  data-category="General"
      #  data-category-id="DIC_kwDOQgMS0M4CzSCX"
      #  data-mapping="pathname"
      #  data-strict="0"
      #  data-reactions-enabled="1"
      #  data-emit-metadata="0"
      #  data-input-position="top"
      #  data-theme="preferred_color_scheme"
      #  data-lang="zh-CN"
      #  crossorigin="anonymous"
      #  async>
      #  </script>
      #origin = "https://giscus.app" # FixIt 0.3.7 | 新增 Or set it to your self-hosted domain

      # FixIt 0.2.14 | 新增 Giscus 评论系统设置
      [params.page.comment.giscus]
        enable = true
        repo = "jiang1bo/jiang1bo"
        repoId = "xxxgDOQgMS0A"
        category = "General"
        categoryId = "xxx_kwDOQgxxx0M4CzSCX"
        mapping = "pathname"
        origin = "https://giscus.app" # FixIt 0.3.7 | 新增 Or set it to your self-hosted domain
        strict = "0" # FixIt 0.2.18 | 新增
        term = ""
        reactionsEnabled = "1"
        emitMetadata = "0"
        inputPosition = "bottom" # ["top", "bottom"]
        lightTheme = "light"
        darkTheme = "dark"
        lazyLoad = true
        lang = "zh-CN"  # 设置语言


#verification map 网站验证代码，用于 Google/Bing/Yandex/Pinterest/Baidu/360/Sogou。
[params.verification]
    google = ""
    bing = ""
    yandex = ""
    pinterest = ""
    baidu = ""
    so = ""
    sogou = ""


#seo map SEO 配置。
[params.seo]
    image = ""
    thumbnailUrl = ""

#compatibility map 兼容性设置。
[params.compatibility]
    polyfill = false
    objectFit = false


#backToTop FixIt 0.2.16 | 新增 map 返回顶部按钮配置。
[params.backToTop]
    enable = true
    scrollpercent = true

#readingProgress FixIt 0.2.16 | 新增 map 阅读进度条配置。
[params.readingProgress]
    enable = false
    start = "left"
    position = "top"
    reversed = false
    light = ""
    dark = ""
    height = "2px"

#cacheRemotebool 是否缓存远程图片以获得更好的优化效果，默认：false。
#optimisebool 是否对图片进行缩放和优化，默认：false。
[params.image]
  cacheRemote = false
  optimise = false

#recentlyUpdated FixIt 0.3.13 | 新增 map 最近更新文章设置。
[params.recentlyUpdated]
    archives = true
    section = true
    list = true
    days = 30
    maxCount = 10

#busuanzi FixIt 0.3.10 | 新增 map 不蒜子计数器配置。
[params.busuanzi]
    enable = true
    source = "https://vercount.one/js"
    siteViews = true
    pageViews = true




#搜索设置
  [params.search]
    enable = true
    type = "fuse"
    contentLength = 4000
    placeholder = ""
    maxResultLength = 10
    snippetLength = 30
    highlightTag = "em"
    absoluteURL = false
    
    [params.search.fuse]
      isCaseSensitive = true
      minMatchCharLength = 2
      findAllMatches = false
      location = 0
      threshold = 0.3
      distance = 100
      ignoreLocation = false
      useExtendedSearch = true
      ignoreFieldNorm = false

#页面底部信息设置
[params.footer]
    enable = true
    copyright = true
    author = true
    since = ""
    gov = "china"
    icp = "20251129"
    license = ""
    [params.footer.powered]
      enable = true
      hugoLogo = true
      themeLogo = true
    [params.footer.siteTime]
      enable = true
      animate = true
      icon = "fa-solid fa-heartbeat"
      pre = ""
      value = "2021-11-11"
    [params.footer.order]
      powered = 0
      copyright = 0
      statistics = 0
      visitor = 0
      beian = 0

#自定义输出设置
[mediaTypes]
  [mediaTypes."text/markdown"]
    suffixes = ["md"]

[outputFormats]
  #  用于输出 /archives/index.html 文件的设置
  [outputFormats.archives]
    path = "archives"
    baseName = "index"
    mediaType = "text/html"
    isPlainText = false
    isHTML = true
    permalinkable = true
  #  用于输出 /offline/index.html 文件的设置
  [outputFormats.offline]
    path = "offline"
    baseName = "index"
    mediaType = "text/html"
    isPlainText = false
    isHTML = true
    permalinkable = true
  #  用于输出 readme.md 文件的设置
  [outputFormats.readme]
    baseName = "readme"
    mediaType = "text/markdown"
    isPlainText = true
    isHTML = false
  #  用于输出 baidu_urls.txt 文件的设置
  [outputFormats.baidu_urls]
    baseName = "baidu_urls"
    mediaType = "text/plain"
    isPlainText = true
    isHTML = false
  #  用于输出 search.json 文件的设置
  [outputFormats.search]
    baseName = "search"
    mediaType = "application/json"
    rel = "search"
    isPlainText = true
    isHTML = false
    permalinkable = true

    
[markup]
  _merge = "shallow"
[outputs]
  _merge = "shallow"
[taxonomies]
  _merge = "shallow"

[params]
description = 'welcome to femris的博客网站!'
keywords = ["Hugo", "FixIt"]
#是否显示纯文本摘要，默认：false
summaryPlainify = 'true'


#header map 页面头部导航栏配置。
[params.header]
#桌面端导航栏模式，可选值：sticky、normal、auto, 默认：sticky。
    desktopMode = "sticky"
#移动端导航栏模式，可选值：sticky、normal、auto, 默认：auto。
    mobileMode = "auto"



[params.page]
# FixIt 0.2.18 | 新增 是否启用文章作者头像
    authorAvatar = true
# 代码块包装器配置
    [params.page.code]
      # FixIt 0.3.9 | 新增 是否启用代码块包装器
      enable = true
      # 是否显示代码块包装器的复制按钮
      copy = true
      # FixIt 0.2.13 | 新增 是否显示代码块包装器的编辑按钮
      edit = false
      # 默认展开显示的代码行数
      maxShownLines = 10
```



## 3.GitHub 仓库准备

接下来创建两个 GitHub 仓库：

1. GitHub Pages 仓库：仓库名前缀必须使用用户名，格式为<username>.github.io，必须为公开仓库，用于博客部署。
2. 博客仓库：可随意命名如 myblog，私有仓库，用于存储博客源代码，包括文章和配置。

**当blog博客仓库的代码提交到 main 分支时，会触发 GitHub Actions 将 Hugo 构建好的静态站点文件部署到 GitHub Pages 仓库，用户即可通过 GitHub Pages 域名进行博客访问。**

博客仓库将主题以 下载后手动导入，通过配置文件指定使用的主题；

2个仓库中，只有 GitHub pages 仓库是必须的，如果不在意博客内容和源代码隐私性，可以去掉 myblog 仓库，将博客源代码存储到 <username>.github.io 仓库中，GitHub Pages 支持使用特定分支（默认是 gh-pages）部署。

### 这里需要两个授权

**（1）为了使 `r1源仓库` 中的 Actions 工作流有权限启动构建，你需要一个 Token**。

**（2）为了源仓库r1有权限向目标仓库r2推送，你需要一个 SSH密钥对， 进行授权。**

### 1.获取并配置部署令牌 (Personal Access Token)

**1. 获取并配置部署令牌 (Personal Access Token)**

**为了使 `源r1仓库` 中的 Actions 工作流有权限启动构建，你需要一个 Token**。

- **生成 Token**：在你的 GitHub 主页，点击头像 -> **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**。
- **关键设置**：为 Token 设置一个描述性名称（如 `Deploy to My Pages`），**过期时间建议选 `No expiration`**，并在权限中**务必勾选完整的 `repo`**。生成后**务必立即复制保存**。
- **在 r1 中添加 Secret**：进入私有仓库 `r1` 的 **Settings** -> **Secrets and variables** -> **Actions**。点击 `New repository secret`。
  - **Name**: `PERSONAL_TOKEN` (必须与工作流中名称一致)。
  - **Value**: 粘贴你刚才复制的 Token。

**2. 为目标仓库 r2 启用 GitHub Pages**
进入公共仓库 `r2` 的 **Settings** -> **Pages**。在 **Source** 部分，选择 **GitHub Actions**。



### 2.获取并配置**SSH 部署密钥 (Deploy Key)**

这个工作流使用 **SSH密钥对** 而非 Personal Access Token (PAT) 进行授权，这是更安全、更推荐的做法。你需要按照以下步骤进行配置：

1. **生成密钥对**：在本地使用 `ssh-keygen -t rsa -b 4096 -C “your_email@example.com” -f gh-pages-deploy-key` 生成一对新的SSH密钥（私钥 `gh-pages-deploy-key` 和公钥 `gh-pages-deploy-key.pub`）。
2. **在目标仓库 (r2) 添加公钥**：
   - 进入你的公共 Pages 仓库 (r2) 的 **Settings**。
   - 找到 **Deploy keys** 部分，点击 **Add deploy key**。
   - **Title** 可填 `Actions Deploy Key`，将公钥文件 (`gh-pages-deploy-key.pub`) 的内容粘贴到 **Key** 框中。
   - **务必勾选 `Allow write access`**，否则只有读取权限，无法推送。
3. **在源仓库 (r1) 添加私钥**：
   - 进入你的私有博客源码仓库 (r1) 的 **Settings**。
   - 找到 **Secrets and variables** 下的 **Actions**，点击 **New repository secret**。
   - **Name** 必须填 `PAGES_DEPLOY_KEY`（与工作流中 `secrets.PAGES_DEPLOY_KEY` 对应）。
   - 将私钥文件 (`gh-pages-deploy-key`) 的**全部内容**（包括 `-----BEGIN OPENSSH PRIVATE KEY-----` 和 `-----END OPENSSH PRIVATE KEY-----`）粘贴到 **Value** 框中。

## 4.详细配置步骤

### 1.1. **源仓库（源码仓库）配置**
- **位置**：存放 Hugo 源代码的仓库
- **需要设置的 Secrets**：
  - `ACCESS_TOKEN`：具有 `repo` 权限的 Personal Access Token，用于访问私有子模块
  - `PAGES_DEPLOY_KEY`：SSH 私钥，用于推送到目标仓库

### 1.2. **目标仓库配置**
- **位置**：GitHub Pages 托管的仓库
- **需要设置**：
  - 添加对应的 SSH 公钥为 **Deploy Key**
  - Settings → Deploy keys → 添加新密钥，勾选 **Allow write access**

### 1.3. **创建 Secrets**

**创建 Personal Access Token：**

1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. 生成新 token，至少选择：
   - `repo`（完全访问仓库）
   - `workflow`（如果需要）
3. 复制 token 值

**创建 SSH 密钥对：**

```bash
# 生成 SSH 密钥对
ssh-keygen -t rsa -b 4096 -C "your-email@example.com" -f github_pages_deploy_key

# 会生成两个文件：
# - github_pages_deploy_key（私钥）
# - github_pages_deploy_key.pub（公钥）
```

### 1.4. **配置源仓库 Secrets**
在源仓库的 Settings → Secrets and variables → Actions：
- 添加 `ACCESS_TOKEN`：填入刚才创建的 PAT
- 添加 `PAGES_DEPLOY_KEY`：填入生成的私钥内容

### 1.5. **配置目标仓库 Deploy Key**
在目标仓库的 Settings → Deploy keys：
- 添加新密钥，名称随意
- 将公钥内容粘贴进去
- **必须勾选** "Allow write access"

### 1.6. **检查配置**
确保以下配置与实际匹配：
```yaml
# 检查这些值是否符合你的项目
HUGO_VERSION: 0.152.2        # 必须与本地 Hugo 版本一致
DART_SASS_VERSION: 1.93.2
external_repository: jiang1bo/jiang1bo.github.io  # 目标仓库
publish_branch: main         # 目标仓库的分支
```

### 1.7. **权限设置**
在源仓库的 Settings → Actions → General：
- 确保 "Workflow permissions" 设置为：
  - ✅ "Read and write permissions"
  - ✅ "Allow GitHub Actions to create and approve pull requests"

### 1.8验证工作流

1. **手动触发测试**：
   - 在 Actions 页面点击 "Run workflow"
   - 选择 "workflow_dispatch"

2. **查看日志**：
   - 检查每一步是否成功
   - 特别注意缓存和部署步骤

### 1.9.故障排除

1. **权限问题**：
   
   ```
   Error: Permission to jiang1bo/jiang1bo.github.io denied
   ```
   - 检查 Deploy Key 是否允许写权限
   - 检查 SSH 密钥是否正确
   
2. **Hugo 构建失败**：
   - 确认 Hugo 版本与本地一致
   - 检查主题子模块是否配置正确

3. **子模块问题**：
   
   - 确保主题或其他子模块使用 HTTPS 而非 SSH URL
   - 在 `.gitmodules` 中验证

**这个工作流配置完成后，每次推送到 `main` 分支都会自动构建并部署网站到 GitHub Pages。**

### 1.10一个完整的workflow工作流

2仓库结合,,一个私密构建,一个GitHub pages 公开仓库。

```shell
name: Deploy to Github Pages
on:
  push:
    branches: [ main ]
  workflow_dispatch:
# 权限配置 (已修正，删除无效的 metadata 和 packages)
permissions:
  contents: read  # checkout 代码需要读权限 (如果使用 PAT 认证，此项非必须但建议保留)
  pages: write
  id-token: write
  
jobs:
  build:
    runs-on: ubuntu-latest  # 更新为最新 Ubuntu 版本以获取更好的兼容性
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    env:
      # ==================== 版本定义（根据参考配置更新）====================
      DART_SASS_VERSION: 1.93.2   # Dart Sass 版本
      GO_VERSION: 1.25.3          # Go 语言版本
      HUGO_VERSION: 0.152.2       # Hugo 版本（必须与参考配置及本地版本一致）
      NODE_VERSION: 22.20.0       # Node.js 版本
      TZ: Europe/Oslo             # 设置构建环境时区
      # ==================== 版本定义结束 ====================
    steps:
       # 步骤 1: 检出源代码 (已添加 token 认证)
      - name: Checkout Source Code
        uses: actions/checkout@v5
        with:
          submodules: recursive
          fetch-depth: 0
          # 使用你创建的 Personal Access Token 来访问私有仓库
          token: ${{ secrets.ACCESS_TOKEN }} # 请确保在仓库 Secrets 中已添加 ACCESS_TOKEN
      # 步骤 2: 设置 Go 环境（更新至 v5 版本）
      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: false
      - run: go version
      # 步骤 3: 设置 Node.js 环境（保持 v4，与参考配置一致）
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      # 步骤 4 & 5: 创建本地目录并安装 Dart Sass（优化安装方式）
      - name: Create directory for user-specific executable files
        run: mkdir -p "${HOME}/.local"
      - name: Install Dart Sass
        run: |
          curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "${HOME}/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}" # 将路径加入系统 PATH
      # 步骤 6: 安装 Hugo 扩展版（优化安装方式，并改为本地下载）
      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"
      # 步骤 7: 验证所有安装（新增步骤，参考参考配置）
      - name: Verify installations
        run: |
          echo "Dart Sass: $(sass --version)"
          echo "Go: $(go version)"
          echo "Hugo: $(hugo version)"
          echo "Node.js: $(node --version)"
      # 步骤 8: 安装 Node.js 依赖（新增步骤，参考参考配置）
      - name: Install Node.js dependencies
        run: |
          [[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true
      # 步骤 9: 配置 Git（新增步骤，参考参考配置）
      - name: Configure Git
        run: |
          git config core.quotepath false
      # 步骤 10 & 11 & 12: 缓存管理（新增步骤，大幅提升后续构建速度）
      - name: Cache restore
        id: cache-restore
        uses: actions/cache/restore@v4
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys:
            hugo-
      # 使用与参考配置一致的构建参数，并添加缓存目录
      # 新增缓存参数
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          
          hugo \
            --gc \
            --minify \
            --cacheDir "${{ runner.temp }}/hugo_cache" 
      - name: Cache save
        id: cache-save
        uses: actions/cache/save@v4
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}
      # 步骤 13: 部署 （核心部署步骤保持不变）
      # 从源仓库 (r1) Secrets 读取 SSH 私钥
      #  需要修改为你自己的目标仓库 (r2)
      # 或 gh-pages，需与目标仓库 Pages 设置匹配
      - name: Deploy 
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.PAGES_DEPLOY_KEY }} 
          external_repository: jiang1bo/jiang1bo.github.io 
          publish_dir: ./public
          publish_branch: main 
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          full_commit_message: ${{ github.event.head_commit.message }}

```



使用 CI/CD 工作流程部署他们的网站，通过推送 [1](https://pre.fixit.lruihao.cn/zh-cn/documentation/getting-started/quick-start/#fn:1)到他们的 GitHub 或 GitLab 存储库会触发构建和部署。流行的提供商包括 [Vercel](https://vercel.com/)[2](https://pre.fixit.lruihao.cn/zh-cn/documentation/getting-started/quick-start/#fn:2)、[Netlify](https://www.netlify.com/)[3](https://pre.fixit.lruihao.cn/zh-cn/documentation/getting-started/quick-start/#fn:3)、[AWS Amplify](https://aws.amazon.com/amplify/)、[CloudCannon](https://cloudcannon.com/)、[Cloudflare Pages](https://pages.cloudflare.com/)、 [GitHub pages](https://pages.github.com/) 和 [GitLab pages](https://docs.gitlab.com/ee/user/project/pages/)。

要了解如何部署站点，请参阅 [托管和部署](https://gohugo.io/hosting-and-deployment/) 部分。



# 二，建站总结：

## 网站建站流程

1.买域名，（国内：速度快，需要实名，卖服务器icp备案。国外：速度较慢，不需要实名，服务器，域名都不需要备案）

2.买服务器，域名解析，备案

3.代码编程

4.云上部署

**因为备案和域名没有关系，只和主机是否在国内的有关系。**

国外域名+国外主机=不用备案

国外域名+国内主机=备案

国内域名+国外主机=不用备案

国内域名+国内主机=备案





## 建站总结

**根据中国法律 ，使用境外服务器建立的网站，如果服务对象为中国内地 用户，则需要备案。**

**即使 网站域名是国外的，如果网站内容或服务涉及中国内地用户，也需要备案。 **

**因此，如果您使用腾讯云中国香港服务器建立的网站服务对象为中国内地用户，您需要进行备案。**

**建议您咨询相关部门了解更多细节和要求**



- 使用 TCP 隧道时，如果您穿透的 协议 是 HTTP，必须使用海外节点。HTTPS 协议 不受影响
- 使用 HTTP(S) 隧道时，必须完成 实名认证。如果使用国内节点进行穿透，必须使用 有备案 的域名

来自 https://doc.natfrp.com/app/http.html#choose-tunnel-type

限制过大，可以购买国外服务器，国外建站,较为方便,

国内域名备案比较繁琐。需要money，购买云服务器，icp服务器注册码





## github pages建站

[Fermi](https://jiang1bo.github.io/)

**当建站github仓库名字为name.github.io时(name是github的用户名),网站域名不会有后缀**

功能不完善，限制很大，只能部署静态网页，

经济，简单，稳定



## vps+域名+宝塔+wordpress建站

https://jiangbos.top/

功能强大，技术成熟，方便快捷

耗费有点大

 

# 三，遇到的问题

## **全文加密**

**前置参数**

FixIt 主题提供了两个前置参数用于全文加密。

- **password**: *[必需]* 加密页面内容的密码
- **message**: *[可选]* 加密提示信息

例如，本文的前置参数如下：

```toml
 title: 主题文档 - 内容加密
date: 2022-05-28T11:51:41+08:00
author:
  name: jiangbo
  link: 
description: 了解如何在 FixIt 主题中加密内容。
password: 1212
message: 密码是 1212
resources:
  - name: featured-image
    src: featured-image.png
tags:
  - Encryption
categories:
  - Documentation

```

**信息**

1. 每次输入正确密码后，会在用户本地缓存密码 hash 值，一天之内再次访问时，将自动解锁文章
2. 文章最后提供有一个 “重新加密” 的按钮，点击即可立即忘记密码，并重新加密内容
3. 加密文章已从搜索中隐藏
4. 加密文章的 Markdown 输出已禁用，为了防止密码泄漏，**请勿将加密文章以任何形式公开**



## 图片配置

图片配置，ico需要parm.app，svgfavicon配置控制

- 对于 **static** 和 **assets** 这两个文件夹，Hugo 在构建时会直接将 **static** 文件夹下的文件复制到网站的根目录下，而 **assets** 文件夹下的文件会被编译后得到一个链接。
- 例如，我的网站图标是 **static/favicon.webp**，在最终上线后，可以直接访问 https://www.trrw.tech/favicon.webp 来打开这个图片。     而我首页的头像图片则是 **assets/img/avatar.jpg**，他的链接是 https://www.trrw.tech/img/avatar_hu_b50a61df1acbb930.jpg
- 但是在配置中，这两个文件夹下的文件使用方式是一样的，上个两个文件的使用方式分别是：favicon.webp 和 img/avatar.jpg。

来自 <[https://www.trrw.tech/p/hugo-stack%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE/](https://www.trrw.tech/p/hugo-stack主题配置/)> 

## 文档网页渲染实现pdf内嵌

**Makedown文档本身无法实现pdf文档预览的功能，但是可以通过embed,网页渲染实现pdf文档预览**

Go语言圣经 (Alan A. A. Donovan, Brian W. Kernighan etc.) (Z-Library).pdf

```html
<embed src="https://example.com/document.pdf" type="application/pdf" width="100%" height="800px">
```





# 内容参考：

1. [快速上手 | FixIt](https://fixit.lruihao.cn/zh-cn/documentation/getting-started/quick-start/)

2. [Hugo主题Reimu中giscus.app拒绝连接问题的分析与解决-CSDN博客](https://blog.csdn.net/gitblog_07048/article/details/148377292)

3. [配置 FixIt | FixIt](https://pre.fixit.lruihao.cn/zh-cn/documentation/getting-started/configuration/#page)

4. [使用 Hugo 和 FixIt 主题创建个人博客 - 程序员水王](https://shuiwang.online/posts/create-blog-with-hugo-and-theme-fixit/)

5. [Hugo 添加友链卡片及页面 - Stilig's blog](https://stilig.me/posts/add-friend/)

6. [博客搭建和配置(Github+Cloudflare+FixIt) - Z3n1th Blog](https://z3n1th1.com/2024/12/博客搭建和配置github-cloudflare-fixit/)

7. [利用 Favicon 为 Hugo 静态站点添加图标 - Bright's Blog](https://ibrights.github.io/post/blog20210527/)

8. [Hugo Stack主题配置](https://www.trrw.tech/p/hugo-stack主题配置/)

9. [Windows下使用hugo和Github Pages配置博客 | Leehow的小站](https://www.haoyep.com/posts/windows-hugo-blog-github/#上传页面)

10. [使用Github Actions将构建产物自动推送到另一个仓库 - 贤狼赫萝](https://heluo520.github.io/posts/notes/deploy)





---

> 作者: <no value>  
> URL: http://localhost:1313/posts/70685c3/  

