# Epanl+wordpress网站配置以及迁移指南


<!--more-->

<!-- Place resource files in the current article directory and reference them using relative paths, like this: `![alt](images/screenshot.jpg)`. -->

## **将wordpress源文件上传解压到wwwroot目录下**

之后点击上方导航栏的“首页”，找到域名绑定

![img](https://pica.zhimg.com/v2-6f352e12decdfc087e0ebc577fde9db8_1440w.jpg)

来到雨云的首页，进入域名服务

![img](https://pic4.zhimg.com/v2-b4261eb1c7b0f1bf205dd8b18b319d85_1440w.jpg)

之后类型选择CNAME，子域名填写自己喜欢的，主域名可以随便选，云产品ID选择刚刚创建的虚拟主机，完成后点击立即创建

完成之后可以在下面找到这条记录（前面的是记录ID，每个人都不一样）

把后面的域名复制下来，在这个教程中就是rain.ates.top

回到虚拟主机管理面板，把复制的内容填进去：

上面的红字提示不用管，在雨云创建子域名的时候系统已经帮我们完成了，这里点击确定即可

之后我们需要一个[SSL证书](https://zhida.zhihu.com/search?content_id=233295315&content_type=Article&match_order=1&q=SSL证书&zhida_source=entity)，不然访问网站会提示“不安全”

由于面板的特殊性，我们需要使用第三方SSL机构来申请证书，先回到雨云的虚拟主机面板

在右侧的域名管理中选择“绑定域名”

![img](https://pic4.zhimg.com/v2-abb5c59741123674bc0d4aca5659678f_1440w.jpg)

暂时不要勾选开启HTTPS支持，确认创建后点击访问测试一下有无问题

加载完成后会出现这个页面，代表成功了，这时候浏览器左上角，也就是地址栏的左边会提示不安全

![img](https://pic1.zhimg.com/v2-35f22eb9c796456e22049e9f910e3ab4_1440w.jpg)

如果要像这样给你的网站“加锁”，那就必须配置SSL证书：

访问这里：[SSL Management | GoGetSSL®](https://link.zhihu.com/?target=https%3A//my.gogetssl.com/en/user/)，注册账户，点击New Order

![img](https://pica.zhimg.com/v2-2aebca209638b1422e5718f2ce5daa5e_1440w.jpg)

我们选择90天免费的

点击后面的Order

![img](https://picx.zhimg.com/v2-91c14edfecadd4429a8d37dc5312d7d3_1440w.jpg)

保持默认，最下面选择那个0.00$的，点击Next Step

![img](https://picx.zhimg.com/v2-23e8ce06d3d236a6323deee983e4dd17_1440w.jpg)

点绿色的Complete Order

在左侧选择[SSL Certificates](https://link.zhihu.com/?target=https%3A//my.gogetssl.com/en/user/sslcerts/all/)，点击刚刚生成的证书后面的View

![img](https://pic4.zhimg.com/v2-50acbf51448ba8cfceef1e7da72dbd9b_1440w.jpg)

点第一个齿轮标志的，在这里[SSL Management | GoGetSSL®](https://link.zhihu.com/?target=https%3A//my.gogetssl.com/en/user/csr/generate/)填写你网站的信息，第一个输入框填写你的域名，我这里直接用自动填充了

之后把生成的东西复制下来：

只需要复制Certificate Signing Request(CSR)的内容就行，从—–BEGIN CERTIFICATE REQUEST—–复制到—–END CERTIFICATE REQUEST—–即可，**页面不要关闭，之后还要用到！**

将复制的内容填写到申请页面里：

![img](https://pic2.zhimg.com/v2-a2a4f6f90a5034a159122212eb02209f_1440w.jpg)

点击下面的验证即可

![img](https://pic3.zhimg.com/v2-6fea9a3d1cac4da5d7b6689d52e6159e_1440w.jpg)



验证方式选择HTTP，点击Next Step

之后填写一些你的信息，滑倒最下面

点击*Complete Generation*

之后过大概一分钟刷新一下

点击上面的Domain Validation

点击Validation File，回到EP面板，在文件管理器中进入wwwroot目录，点击创建目录（也就是创建文件夹的意思），创建一个名为“.well-known”的文件夹

![img](https://pic3.zhimg.com/v2-e0517e778513300855e64fe076a5e0b2_1440w.jpg)

**进入这个目录后**再点击创建目录，名为“pki-validation”，创建完成后进入

之后点击上传文件，把下载的TXT文件上传到这个目录中

![img](https://pic4.zhimg.com/v2-d62caea711787a7b5db1503532f92e3b_1440w.jpg)

这样子即可

之后点击GoGetSSL的重新验证

![img](https://pic3.zhimg.com/v2-ba481924964d33a1fb60d2e228e2c192_1440w.jpg)

过一会邮箱就可以收到证书信息了，证书内容**千万不要**透露给别人！

![img](https://pic3.zhimg.com/v2-277b83a28d8461df26168e2ee56af250_1440w.jpg)

之后回到申请CSR的页面，复制Your Private Server Key的内容，从—–BEGIN PRIVATE KEY—–到—–END PRIVATE KEY—–

回到EP面板，在首页-SSL证书中填写相关信息

![img](https://pic3.zhimg.com/v2-e874f9c4edba772feed571f7789d6dae_1440w.jpg)

![img](https://pic3.zhimg.com/v2-54dc2e2d4e04236b4bd0356ba0c0aa00_1440w.jpg)

证书内容是邮箱收到的那一大段，密钥内容是刚刚复制的Your Private Server Key，点击“提交”

![img](https://pic1.zhimg.com/v2-aa49675f26e8e6b096fe1e73be16f29a_1440w.jpg)

状态变为成功

之后在雨云的SSL证书里点击上传证书

![img](https://pic2.zhimg.com/v2-43247da1f4ac2e531a3329059aeec4c7_1440w.jpg)

![img](https://pica.zhimg.com/v2-6ebc9712e62db35a7812b2c5c263fb08_1440w.jpg)

点击保存证书，回到雨云虚拟主机的管理面板，删掉临时用的域名，之后重新绑定，这里要勾选开启HTTPS支持，选择刚刚上传的证书，打开强制SSL跳转

![img](https://picx.zhimg.com/v2-decff4e037bf66b17190cc1633569395_1440w.jpg)

过一分钟以后再次访问，就可以看到网站已经被“加锁”了

![img](https://pic2.zhimg.com/v2-96f710bc35a0ba2fd18380f07c85238d_1440w.jpg)

接下来安装WordPress，这个页面就是选择语言的，拉到最下面可以选择中文，点击继续

![img](https://pic2.zhimg.com/v2-afc372c8612032c8e907a0c3e01cc757_1440w.jpg)

点击现在就开始，之后会要求填写数据库信息，可以在EP面板里下滑找到

![img](https://pic4.zhimg.com/v2-e684b8c3cd14f16e314de6047d6eceb9_1440w.jpg)

密码可以在雨云虚拟主机处下滑找到

![img](https://pic4.zhimg.com/v2-283188fb5841a704ec4cd0103218105d_1440w.jpg)

点击复制即可，将所有信息填写完成后点击提交，如果你填写的没问题那么会出现以下界面，点击“运行安装程序”

![img](https://pic2.zhimg.com/v2-a89c4261f7b04790ad14307caac4ce43_1440w.jpg)

填写站点的基本信息后点击安装即可

![img](https://pic2.zhimg.com/v2-c152016dcabc00c64b3344f8a85a8427_1440w.jpg)

之后等待一会便可以看到安装成功

![img](https://pic4.zhimg.com/v2-f7f96c00b547db605b2fa52460606dfb_1440w.jpg)

点击登录即可登录到后台

这时网站就已经安装完成了，接下来我们需要做一些优化

回到EP面板，找到伪静态选项修改规则为“wordpress”

![img](https://pic2.zhimg.com/v2-38ca9c2129ee77ca0b594f2ac9e362b3_1440w.jpg)

点击保存，之后修改一下wordpress的固定链接

![img](https://pic3.zhimg.com/v2-c2d1d848d3376b89da379a6c8382e38e_1440w.jpg)

在设置-固定链接，可以用自定义结构或者在上面的模板中选择一个

之后配置一下缓存，回到EP，找到缓存设置

![img](https://pica.zhimg.com/v2-5b041122ac70c0c314e10c56e7d40960_1440w.jpg)

至此，基本的优化结束，你可以通过外观-主题来选择你喜欢的网站样式，也可以在插件-安装插件为你的网站添加更多的功能

![img](https://pic3.zhimg.com/v2-8ec338cdf48aae5e40cf6a5e9be8e84a_1440w.jpg)

插件不是越多越好，越多的插件网站加载时间就越慢！

## 前言

WordPress 作为全球最流行的博客系统，使用简单，功能丰富，用它来建站的用户非常多。对于站长们来说，网站搬家也是少不了的，有时我们需要更换主机空间，把网站从一个服务器迁移到另一个服务器上，或者更换域名。

本文就详细介绍下 WordPress 网站搬家的几种方法，我们可以通过手动备份搬家，也可以使用插件工具一键完成网站迁移。还是很简单的。

## 准备

WordPress 网站搬家分两种情况，更换域名和不更换域名。搬家过程相同，更换域名的话需要多一步域名替换的操作。

**注意：**搬家前一定要备份网站文件和数据库，搬家过程中如果出现问题也好恢复！

下面分为两种方法进行介绍：插件法和手动法，插件法更简单，手动法更通用；更换域名的方法单独写在最后，可以独立使用。

## 方法一、使用 WordPress 搬家插件迁移网站

推荐使用 [All-in-One WP Migration](https://link.zhihu.com/?target=https%3A//wordpress.org/plugins/all-in-one-wp-migration/) 搬家插件，直接打包全站数据一键迁移，非常方便。

## **# 1. 导出网站文件**

登陆 WordPress 后台，搜索并安装插件 All-in-One WP Migration。

![img](https://pica.zhimg.com/v2-8b73bfc1b9148ca9fa658996bfbcf220_1440w.jpg)

启用后从左边栏进入该插件 -> 导出站点 -> 导出到文件。（高级选项默认或按需要勾选）

![img](https://pic3.zhimg.com/v2-67b5e03eea0cc6f8b9f977c800c3b636_1440w.jpg)

等待文件打包完成，下载到本地。得到一个后缀名为 .wpress 的文件。

![img](https://pica.zhimg.com/v2-aac9c58bd6febfecc373b18e03e6042a_1440w.jpg)

## **# 2. 导入网站文件到新主机空间**

将域名解析到新主机空间。在新主机空间上安装好 WordPress，进入后台安装 All-in-One WP Migration。

然后进入 [这里](https://link.zhihu.com/?target=https%3A//import.wp-migration.com/) 下载 Basic 版本辅助插件 all-in-one-wp-migration-file-extension.zip，手动上传安装该插件，以解除文件上传大小限制。（<512M）

导入刚刚下载到本地的网站文件进行恢复。

![img](https://pic2.zhimg.com/v2-64b0e533edcaaff04b769f2721b7c0e1_1440w.jpg)

导入需要一定时间，耐心等待。导入后会询问是否覆盖文件，确定。然后修改固定链接设置并保存两次。

![img](https://pic4.zhimg.com/v2-bc8c1596a0c1826191513e3469dec073_1440w.jpg)

**搬家完成！**

## 方法二、WordPress 手动搬家

## **# 1. 备份网站文件和数据库**

进入原网站服务器/主机后台，打包下载网站根目录下的所有文件到本地。

进入 [phpMyAdmin](https://zhida.zhihu.com/search?content_id=10350831&content_type=Article&match_order=1&q=phpMyAdmin&zhida_source=entity)，选择网站数据库，导出 sql 文件到本地。

![img](https://pic1.zhimg.com/v2-d406719f4161d27bef726a6973b4b62a_1440w.jpg)

## **# 2. 上传网站文件和导入数据库**

上传网站文件到新主机的网站目录并解压。

在 phpMyAdmin 中创建一个新数据库，并导入上面的网站数据库 sql 文件。

## **# 3. 修改 [wp-config.php](https://zhida.zhihu.com/search?content_id=10350831&content_type=Article&match_order=1&q=wp-config.php&zhida_source=entity)**

编辑 wp-config.php 文件，修改数据库名称、用户名、密码，连接新主机。

```php
/** 数据库名称 */
define('DB_NAME', '名称');

/** 数据库用户名 */
define('DB_USER', '用户名');

/** 数据库密码 */
define('DB_PASSWORD', '密码');
```

将域名解析到新主机空间，进入 WordPress 后台更新固定连接，完成。

## 方法三、使用 WordPress 自带导出/导入功能

WordPress 的工具中自带了导入/导出功能。只将网站的内容进行迁移，因此搬的不完整，不太推荐使用。

![img](https://pic3.zhimg.com/v2-d6cbd71017e5c0b8c284348a2500a1ae_1440w.jpg)

## WordPress 更换域名的方法

进入 phpMyAdmin，选择网站数据库 -> SQL -> 粘贴下面代码 -> 执行。

```sql
UPDATE wp_options SET option_value = REPLACE(option_value, '旧域名', '新域名') WHERE option_name = 'home' OR option_name = 'siteurl';

UPDATE wp_posts SET post_content = REPLACE(post_content, '旧域名' , '新域名');

UPDATE wp_posts SET guid = REPLACE(guid, '旧域名' ,'新域名');
```

![img](https://pic4.zhimg.com/v2-b354f7a4a5cba7ab629f7d123ce8a1d1_1440w.jpg)

## Tips

- **搬家前一定要先备份！备份！备份！**
- All-in-One WP Migration 免费版只支持导入 512M 以内的文件；
- 网站搬家完成后记得更新固定链接，设置 -> 固定链接 -> 保存;
- 搬家前可以先清理下网站里的冗余内容，比如废弃的插件、站内备份文件等，减小网站体积。

## 结语

WordPress 网站搬家还是很简单的，得益于丰富的插件生态，各种搬家、备份插件也很多，可以按安装量排序选择。

如果网站文件不大的话，建议使用插件，方便省事；如果文件较大，那插件导入可能会有点慢，手动上传或许更快些。





您遇到的 `NET::ERR_CERT_AUTHORITY_INVALID` 错误非常明确。问题不在于您的操作，而在于浏览器所信任的证书链。您直接从自己的服务器（`hk.jiangbos.top`）访问时，浏览器收到了一个**CloudFlare Origin SSL证书**，但这个证书**不是由公共信任的根证书机构签发的**，因此浏览器会直接拦截并报错。

您看到的这个 “CloudFlare Origin SSL Certificate” **不是公网证书，而是CloudFlare和您的源服务器之间建立内部加密的专用证书**，它只在CloudFlare的网络内部被信任。

### 🔍 问题核心与解决方案

为了让您的网站能通过 `https` 安全访问，您必须确保两点：
1.  **最终用户**始终通过CloudFlare的CDN节点（即您分配给域名的CloudFlare代理IP）来访问您的网站。
2.  **终结HTTPS连接**的任务交给CloudFlare，并使用它提供的、受浏览器信任的“边缘证书”。

### 🛠️ 请按以下步骤检查和操作

您需要登录到 **CloudFlare控制面板** (dash.cloudflare.com) 进行操作：

| 步骤                     | 操作位置                                                     | 关键操作与说明                                               |
| :----------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **1. 检查DNS记录**       | CloudFlare面板 > **网站** > 您的域名 > **DNS**               | 确保域名（如 `jiangbos.top` 和 `www.jiangbos.top`）的记录**已开启代理（云朵图标为橙色）**。这是最关键的一步，它确保流量经过CloudFlare。 |
| **2. 检查SSL/TLS模式**   | CloudFlare面板 > **网站** > 您的域名 > **SSL/TLS** > **概述** | 将模式设置为 **“完全”** 或 **“完全（严格）”**。这能确保CloudFlare到您源站的连接（即“回源”）也使用加密。 |
| **3. (可选) 验证源证书** | CloudFlare面板 > **网站** > 您的域名 > **SSL/TLS** > **源服务器** | 您可以在这里看到CloudFlare为您的源站生成的证书（就是您贴出的那个）。确认它已安装到您的服务器（ePanel）即可，但**无需公网信任**。 |
| **4. 清理浏览器缓存**    | 本地浏览器                                                   | 完成上述设置后，请**彻底清理浏览器缓存**并使用无痕模式访问您的域名（如 `https://jiangbos.top`）。 |

### 📝 总结与后续

简单来说，您**不应该直接访问源站IP或源站域名**（如 `hk.jiangbos.top`）来测试HTTPS。请始终访问您公开的域名（如 `https://jiangbos.top`）。

完成CloudFlare的DNS和SSL设置后，访问您的公开域名，安全锁就应该能正常显示了。如果完成所有设置后问题依然存在，请检查网站内是否有代码（如数据库中的链接）仍然指向 `http://` 或源站地址，导致“混合内容”问题。

如果方便，请告诉我您在CloudFlare的DNS记录中，“云朵”代理状态是否是开启的（橙色）？以及SSL/TLS模式设置为什么？这能帮助我进一步确认问题。





搭建完wordpress，试着写了一篇博客。文章发布后，首页已经能显示出文章的标题，但是点进去后却提示该页无法显示。

百度一番，先后尝试网上的修改apache配置等方法后依然无效。折腾到最后无意间发现，这个问题是由于wordpress生成的链接中包含中文字符导致的。在wordpress的文章编辑页面中，手动修改链接为纯英文，问题解决。

现在问题解决了，但是每次编辑完文章都要改下链接似乎也不太方便。对于想偷懒的同学，可以安装这个插件：Pinyin Permalink，自动修改文章的链接为拼音，这样即可一劳永逸的解决问题。
————————————————
版权声明：本文为CSDN博主「wayne_l123」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/l1902090/article/details/73610629


---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/851d1b6/  

