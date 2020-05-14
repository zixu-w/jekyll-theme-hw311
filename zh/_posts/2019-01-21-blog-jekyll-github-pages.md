---
layout: post
title:  "使用Jekyll和GitHub Pages搭建博客"
date:   2019-01-21 21:20:39
author: Zixu Wang
categories: Jekyll
tags: blog 博客
lang: zh
ref: blog-jekyll-github-pages
---

作为一名穷酸CS青年，我也时常有写一些博客的想法，一来可以把自己学习生活中的体会感悟记录下来方便日后复习参考；二来也可以把知识分享出去交流学习。然而之前搭建博客的尝试均以失败告终，没能坚持下来，简单总结一下，这些失败的尝试无非是走向了两个极端：要么使用现成的免费内容管理平台比如CSDN或者知乎专栏，但是这些平台限制太多，结构和界面也不在自己的控制下，不够简洁美观；要么自己租用服务器和域名，从头开始自己搭建一个独立博客，但是搭建和调整的过程、发布文章的格式过于繁琐，无法把精力集中在内容创作上。

前几天一个朋友开始写了[GitBook](https://jizhuoran.gitbook.io){:target='_blank'}，又勾起了我搭建博客的三分钟热度。为了吸取之前的教训，我总结了对平台选择的需求：

1. 减小开销，尽量不使用付费平台或服务器；
2. 提供最大程度的自由，能自己设计界面和功能；
3. 暴露最小程度的细节，不必耗费精力管理网站，把内容创作和站点管理分离开，能够专心创作文章。

经过一些研究，我发现 [Jekyll](https://jekyllrb.com/){:target='_blank'} + [GitHub Pages](https://pages.github.com/){:target='_blank'} 最贴合我的需求，并且建成了你正在浏览的这个网站。作为这个博客第一篇实际上的文章，我决定把建站的方法和过程分享出来，也希望能帮助到跟我有同样需求的人。

## GitHub Pages
<img src="/assets/imgs/Octocat.png" title="GitHub" alt="GitHub" class="img-as-is"/>

[GitHub Pages](https://pages.github.com/){:target='_blank'}是GitHub提供的直接从代码仓库生成**静态**网站的托管服务， 本意是方便GitHub上的项目、个人或组织建立文档和帮助网站。[^1] 使用GitHub Pages搭建网站非常简单，只需要创建一个名为 `<username>.github.io` 的代码仓库（用你的GitHub用户名替换 `<username>`），这个仓库里的文件就会被GitHub发布到 `https://<username>.github.io` [^2]

比如我们创建文件 `index.html`：
{% highlight html %}
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Hello GitHub Pages!</title>
  </head>
  <body>
    <h1>Hello GitHub Pages!</h1>
    <p>This page is hosted on GitHub Pages.</p>
  </body>
</html>
{% endhighlight %}
并上传到刚刚建立的 `<username>.github.io` 仓库里，打开浏览器访问 `https://<username>.github.io`，就能看到我们刚刚写的网页了：

<img src="/assets/imgs/index-example.png" title="示例网页" alt="示例网页" class="img-as-is"/>

可以看出，GitHub Pages管理了所有网站后台的细节，我们只需要自己编写前端网页并上传就可以实现任意的静态网站，非常符合我们的要求。当然，这项服务的使用也有一些限制，比如网站大小不能大于1 GB、每月流量有100GB的软限制、以及每个小时最多生成十次等等。[^3] 当然，违反法律或者使用协议的内容也是不允许的。同时，GitHub也不允许、不建议GitHub Pages被用于商业用途，不过我们用来搭建个人博客是绰绰有余了。

总结一下，GitHub Pages给我们提供了免费的静态网站空间和域名，提供了方便的内容管理方式，文章更新和网站的迭代还可以通过<tt>git</tt>来管理版本。如果有需要，还可以自定义自己购买的域名。[^4] 现在虽然GitHub Pages帮我们照顾了整个网站的后台，但是我们仍然需要自己实现整个前端，如果想要一个美观的界面的话，这绝对是个浩大的工程（我试过，放弃了 :smiley:，但是如果读者有能力并且有兴趣这样做的话，是绝对可以实现的）。而且每一篇文章都得用一个完整的HTML来组织格式，这违背了我们分离内容创作和站点管理的要求。这时候我们就需要Jekyll了。

## Jekyll
<img src="/assets/imgs/jekyll-logo-light-transparent.png" title="Jekyll" alt="Jekyll" class="img-as-is"/>

[Jekyll](https://jekyllrb.com/){:target='_blank'}是一个**静态**网站生成器，可以将使用标记语言编写的内容文件生成网页。[^5] 下面是一个典型的Jekyll项目结构：
{% highlight html %}
{% raw %}
├── 404.html                          # 404页面
├── _config.yml                       # 配置文件
├── _data                             # 数据文件目录
├── _includes                         # 包含文件目录
├── index.html                        # 主页
├── _layouts                          # 布局目录
│   ├── default.html                  # 缺省页面布局文件
│   └── post.html                     # 文章页面布局文件
├── _drafts                           # 稿件目录
│   └── blog-jekyll-github-pages.md   # 未发布草稿
└── _posts                            # 已发布文章目录
    └── 2019-01-19-hello-world.md     # 已发布文章
{% endraw %}
{% endhighlight %}
我们可以把Jekyll看成一个从标记语言（Markdown、HTML等等）到静态网站的编译器，它把我们写的内容（比如 `_posts/2019-01-19-hello-world.md`）转换成相应的网页，然后根据定义的布局把这些网页放在一起，组成一个网站。这大大简化了我们创作内容的难度，因为Markdown的语法比HTML简单许多，不熟悉的读者可以学习一下。[^6] 页面模板的使用也通过代码复用很好地隔离了内容与格式，我们只需要写一次HTML/CSS模板（比如 `_layouts/post.html`），之后的内容创造就可以套用模板，专心于内容。下面就是一个模板的例子：
{% highlight html %}
{% raw %}
<!-- _layouts/default.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>{{ page.title }}</title>
  </head>
  <body>
    <div class="page-content">
      {{ content }}
    </div>
  </body>
</html>
{% endraw %}
{% endhighlight %}
这个简单的模板描述了一个HTML页面，其中的双大括号是Jekyll使用的[Liquid模板语言](https://jekyllrb.com/docs/liquid/){:target='_blank'}（了解更多请参考[Liquid官方文档](https://shopify.github.io/liquid/){:target='_blank'}[^7]），如果我们有另一个文件 `hello-jekyll.md` 使用了这个模板：
{% highlight markdown %}
---
layout: default
title: 'Hello Jekyll!'
---

## Hello Jekyll!

This page uses the default layout.
{% endhighlight %}
那么Jekyll就会生成一个相应的HTML文件：
{% highlight html %}
{% raw %}
<!-- hello-jekyll.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Hello Jekyll!</title>
  </head>
  <body>
    <div class="page-content">
      <h2 id="hello-jekyll">Hello Jekyll!</h2>
      <p>This page uses the default layout.</p>
    </div>
  </body>
</html>
{% endraw %}
{% endhighlight %}
Jekyll的README里介绍了其设计哲学[^8]，我觉得非常贴近我们开头提到的需求：

> *Jekyll does what you tell it to do -- no more, no less. It doesn't try to outsmart users by making bold assumptions, nor does it burden them with needless complexity and configuration. Put simply, Jekyll gets out of your way and allows you to concentrate on what truly matters: your content.*

更方便的是，Jekyll就是GitHub Pages背后的处理引擎，我们上传到GitHub Pages仓库里的文件都会自动经过Jekyll的处理。[^9] 但是有一点需要注意：GitHub Pages使用的Jekyll和本地使用的不完全相同，比如出于安全考虑，GitHub Pages不允许自定义Jekyll插件[^10]，只有[一个列表](https://pages.github.com/versions/){:target='_blank'}里的插件可以使用[^11]，当然这都是后话了。

## 本地安装使用Jekyll

在网站迭代和写文章的过程中，我们也希望能在本地实时查看并调整效果，这一节就以Ubuntu为例，介绍一下本地使用Jekyll的方法。

首先，Jekyll是由Ruby编写的，我们需要安装Ruby的运行环境和RubyGem，Ruby的包管理器。
{% highlight bash %}
 sudo apt update
 sudo apt install ruby-full
 sudo apt install gem
 sudo gem update --system
{% endhighlight %}
然后安装Jekyll和Bundler
{% highlight bash %}
 sudo gem install jekyll bundler
{% endhighlight %}
安装成功后，可以使用Jekyll新建一个博客
{% highlight bash %}
 jekyll new myblog
{% endhighlight %}
这个命令会在 `./myblog/` 目录下初始化一个基础的Jekyll项目
{% highlight html %}
├── 404.html
├── about.md
├── _config.yml
├── Gemfile
├── Gemfile.lock
├── index.md
└── _posts
    └── 2019-01-21-welcome-to-jekyll.markdown
{% endhighlight %}
在 `./myblog/` 目录下可以使用Jekyll来生成网站并在本地服务器上架设。运行
{% highlight bash %}
 cd myblog
 bundle exec jekyll serve
{% endhighlight %}
然后访问 `localhost:4000` 就可以看到这个网站了。

<img src="/assets/imgs/jekyll-example.png" title="myblog" alt="myblog" class="img-as-is"/>

不熟悉Jekyll的读者阅读至此可以先参考[官方文档](https://jekyllrb.com/docs/){:target='_blank'}[^5] 在这个最简单的项目里多试一试Jekyll的不同特性、模板的使用、网站结构的配置等等。将项目push到之前新建的GitHub Pages仓库里就可以发布网站了。

## 使用现有的主题

如果不想自己编写模板和样式，网上有非常多的开源主题可以直接使用，比如[JekyllThemes.org](http://jekyllthemes.org/){:target='_blank'}和[JekyllThemes.io](https://jekyllthemes.io/){:target='_blank'}. 我的这个网站就是基于[Centrarium主题](https://github.com/bencentra/centrarium){:target='_blank'}修改而成。鉴于我做了很多修改，也自己加了不少功能，我也整理了一个主题发布到了GitHub上：[Jekyll-Theme-HW311](https://github.com/zixu-w/jekyll-theme-hw311){:target='_blank'}，喜欢的欢迎fork和star。主要的特性有：

- 文章搜索
- 简单的多语言支持
- 可自定义字体和颜色
- 主页和文章支持题图
- 基于类别的文章归档
- 代码高亮
- 使用Disqus的文章评论
- Google Analytics分析

要使用这个模板，先在GitHub上fork一份我的主题仓库（[Jekyll-Theme-HW311](https://github.com/zixu-w/jekyll-theme-hw311){:target='_blank'}）到你自己的GitHub账户下，然后把重命名成 `<username>.github.io`（用你的GitHub用户名替换 `<username>`）。

在使用这个主题前，还需要编辑 `_config.yml` 来进行配置。需要编辑的条目有：
{% highlight yaml %}
title: '<website title>'              # 网站的标题
subtitle: '<website subtitle>'        # 网站的副标题
email: '<your email>'                 # 你的email
name: '<your name>'                   # 你的名字
description: '<website description>'  # 网站描述
url: '<website url>'                  # 网站URL，比如https://<username>.github.io
cover: '<path to cover image>'        # 网站封面图片
logo: '<path to logo>'                # 网站logo
disqus_shortname: '<disqus handle>'   # Disqus上设置的shortname，用于连接Disqus服务
ga_tracking_id: '<ga tracking id>'    # Google Analytics追踪ID
social:
  - ...                               # 社交网站和分享设置
protocols:
  ...                                 # OS protocol设置
{% endhighlight %}
完成这些设置并同步到GitHub上之后，就可以访问 `<username>.github.io` 看到你的博客了！:tada:

写文章可以在 `zh/_posts/` 或 `en/_posts/` 目录下新建 `YYYY-MM-DD-文章标题.md` 文件。你可以在这些目录下找到示例文件以供参考。

多语言翻译支持、文章类别等数据配置文件可以在 `_data/` 目录下找到。

Jekyll功能的使用请参考[官方文档](https://jekyllrb.com/docs/){:target='_blank'}[^5]，关于这个主题的具体问题也欢迎在[下面评论区](#disqus_thread)或者[GitHub](https://github.com/zixu-w/jekyll-theme-hw311){:target='_blank'}评论。

## 使用自定义域名

如果不想使用GitHub提供的 `github.io` 次级域名，我们也可以自行购买域名并指向我们的GitHub Pages，请参考[GitHub提供的教程](https://help.github.com/articles/using-a-custom-domain-with-github-pages/){:target='_blank'}。[^4]

首先在你的GitHub Pages代码仓库里配置自定义域名，可以通过 `Settings -> GitHub Pages -> Custom domain` 进行设置；也可以直接在项目根目录下添加一个 `CNAME` 文件，写入自定义域名（使用不带协议名的裸域名，比如 `hw311.me`，而不是 <code>https://<span></span>hw311.me</code>）。

然后，去你的域名提供商网站设置A记录，把域名的IP解析成以下四条[^12]
{% highlight html %}
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
{% endhighlight %}
这四条是本文写成时，[GitHub Pages官方文档](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider){:target='_blank'}[^12]上提供的IP地址，实际配置时建议再次查看文档以获取最新的地址配置。

全部正确配置之后，等待一段时间之后就可以在自定义域名上看到自己的博客了。

## 后记

至此，我们已经使用Jekyll和GitHub Pages搭建出一个独立的个人博客了。这个方案虽说也有很多不足和缺陷，比如只能搭建静态网站、插件使用被GitHub限制等等，但还是完美贴合了我们一开始的需求。只想专注于内容的读者已经可以开始使用模板专心创作了，如果你像我一样也喜欢自己折腾折腾建站，希望本文用到的参考资料可以帮助到你。

最后也希望自己这一次能坚持下去吧。（笑）

<hr>
## 参考资料
<ul>
  <li><p>
    <a target="_blank" href="http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html" title="搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门">搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门</a>
  </p></li>
  <li><p>
    <a target="_blank" href="https://www.jianshu.com/p/88c9e72978b4" title="用Jekyll搭建的Github Pages个人博客">用Jekyll搭建的Github Pages个人博客</a>
  </p></li>
</ul>
[^1]: [What is GitHub Pages?](https://help.github.com/articles/what-is-github-pages/){:target='_blank'}
[^2]: [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/){:target='_blank'}
[^3]: [GitHub Pages - Usage limits](https://help.github.com/articles/what-is-github-pages/#usage-limits){:target='_blank'}
[^4]: [Using a custom domain with GitHub Pages](https://help.github.com/articles/using-a-custom-domain-with-github-pages/){:target='_blank'}
[^5]: [Jekyll Documentation](https://jekyllrb.com/docs/){:target='_blank'}
[^6]: [The Markdown Guide](https://www.markdownguide.org/){:target='_blank'}
[^7]: [Liquid: Safe, customer-facing template language for flexible web apps](https://shopify.github.io/liquid/){:target='_blank'}
[^8]: [Jekyll - Philosophy](https://github.com/jekyll/jekyll#philosophy){:target='_blank'}
[^9]: [Using Jekyll as a static site generator with GitHub Pages](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/){:target='_blank'}
[^10]: [Adding Jekyll plugins to a GitHub Pages site](https://help.github.com/articles/adding-jekyll-plugins-to-a-github-pages-site/){:target='_blank'}
[^11]: [List of gems used by GitHub Pages](https://pages.github.com/versions/){:target='_blank'}
[^12]: [GitHub Pages - Configuring A records with your DNS provider](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider){:target='_blank'}
