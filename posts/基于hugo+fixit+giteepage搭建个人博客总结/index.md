# 基于hugo+Fixit+Giteepage搭建个人博客总结


<!--more-->

## 个人博客地址：[Fermi (gitee.io)](https://jiang-zhao-153.gitee.io/fermi/)

![image-20230926205155303](https://gitee.com/jiang-zhao-153/typora_images/raw/master/images/image-20230926205155303.png)

## **环境要求：**

**go语言环境**

[All releases - The Go Programming Language (google.cn)](https://golang.google.cn/dl/)

![image-20230926205433450](https://gitee.com/jiang-zhao-153/typora_images/raw/master/images/image-20230926205433450.png)

**hugo框架**

[gohugoio/hugo: The world’s fastest framework for building websites. (github.com)](https://github.com/gohugoio/hugo/releases/tag/v0.118.2)

![image-20230926205504460](https://gitee.com/jiang-zhao-153/typora_images/raw/master/images/image-20230926205504460.png)

**为hugo添加环境变量，查看hugo版本**

![image-20230926210851119](https://gitee.com/jiang-zhao-153/typora_images/raw/master/images/image-20230926210851119.png)

![image-20230926205554033](https://gitee.com/jiang-zhao-153/typora_images/raw/master/images/image-20230926205554033.png)

## **使用hugo搭建个人网站**

建站步骤：[FixIt (lruihao.cn)](https://fixit.lruihao.cn/zh-cn/)

Fixit_gitee主题：[FixIt: 🔧 A clean, elegant but advanced blog theme for Hugo 一个简洁、优雅且高效的 Hugo 主题 (gitee.com)](https://gitee.com/lruihao/FixIt)

**创建网站**

**1.Hugo 提供了一个 new命令来创建一个新的网站：**

| 1     | hugo new site  my_website |
| :---- | :------------------------ |
| **2** | **cd my_website**         |

**2.将** [**FixIt**](https://github.com/hugo-fixit/FixIt) **主题克隆到** **themes** **目录中，将其作为** [**Git 子模块**](https://git-scm.com/book/en/v2/Git-Tools-Submodules) **添加到您的项目中。**

| 1     | git init                                                     |
| ----- | :----------------------------------------------------------- |
| **2** | **git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt** |

**3.以下是创建第一篇文章的方法：**

| hugo new posts/first_post.md |
| ---------------------------- |

**默认情况下，所有文章和页面均作为草稿创建。如果想要渲染这些页面，请从元数据中删除属性 draft: true设置属性 draft: false 或者在以下步骤中为hugo命令添加    -D   --buildDrafts参数。****

**4.启动网站保存文件后，使用以下命令在本地启动网站：**

| 1    | hugo server |
| ---- | ----------- |

当你运行 hugo server 时，当文件内容更改时，页面会随着更改自动刷新。

**5.去查看 http://localhost:1313**



**6.当你准备好部署你的网站时，运行以下命令：**

| 1    | hugo |
| ---- | ---- |

会生成一个 public 目录，其中包含你网站的所有静态内容和资源。现在可以将其部署在任何 Web 服务器上。

我们的大多数用户使用 CI/CD 工作流程部署他们的网站，通过推送 [1](https://fixit.lruihao.cn/zh-cn/documentation/getting-started/#fn:1) 到他们的 GitHub 或 GitLab 存储库会触发构建和部署。流行的提供商包括 [Vercel](https://vercel.com/)[2](https://fixit.lruihao.cn/zh-cn/documentation/getting-started/#fn:2)、[Netlify](https://www.netlify.com/)[3](https://fixit.lruihao.cn/zh-cn/documentation/getting-started/#fn:3)、[AWS Amplify](https://aws.amazon.com/cn/amplify/)、[CloudCannon](https://cloudcannon.com/)、[Cloudflare Pages](https://pages.cloudflare.com/)、 [GitHub pages](https://pages.github.com/) 和 [GitLab pages](https://docs.gitlab.com/ee/user/project/pages/)。



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

1. 每次输入正确密码后，会在用户本地缓存密码     hash 值，一天之内再次访问时，将自动解锁文章
2. 文章最后提供有一个 “重新加密”     的按钮，点击即可立即忘记密码，并重新加密内容
3. 加密文章已从搜索中隐藏
4. 加密文章的 Markdown     输出已禁用，为了防止密码泄漏，**请勿将加密文章以任何形式公开**

## **md模板**

```
title: 内容加密

date: 2022-05-28T11:51:41+08:00

author:

 name: jiangbo

 link: https://gitee.com/jiang-zhao-153

description: 了解如何在 FixIt 主题中加密内容。

password: 1234

message: 密码是 1234       //可填可不填

resources:

 -name: featured-image

 -src: featured-image.png

tags:

 - Encryption              //标签

categories:

 - Documentation         //分类

<!--more-->
```

## **Gitee page** **部署**

新建仓库，上传public内的代码，启动部署

**当仓库名为gitee账户名时，会以“gitee.io”结尾，不会带“/”**

Github 也可以为其配置域名

[部署到Github Pages上的博客，自定义域名，和免费域名如何申请 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/142185293)


---

> 作者: [jiangbo](https://gitee.com/jiang-zhao-153)  
> URL: https://jiang-zhao-153.gitee.io/fermi/posts/%E5%9F%BA%E4%BA%8Ehugo+fixit+giteepage%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%80%BB%E7%BB%93/  

