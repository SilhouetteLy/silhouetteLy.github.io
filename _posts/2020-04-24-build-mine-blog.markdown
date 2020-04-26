---
layout:     post
title:      "基于github pages + jekyll搭建个人博客网站"
subtitle:   "Hello World, My First Blog"
date:       2020-04-24 18:44:46
author:     "silhouetteLy"
header-img: "img/images/blog/01/00-post-bg.jpg"
tags:
    - Blog
---

> “你去爬山被告知的第一件事，就是别去看顶峰。而是专注于在爬的路，一步一个脚印地，耐心攀爬，如果你不断看山顶，就会泄气。   --黑泽明 ”

## 前言

一直苦于Typora上的书写的Markdown文件不能支持移动端浏览，所以老有搭建私人博客的冲动，因朋友推荐的搭建博客方法，感觉简单而且个性化十足，正好赶上是周末，所以花了一天的时间试试，下面是我的搭建过程及遇到的问题。

## 一、基于github pages + jekyll搭建个人博客网站

### 1.1 基于JeKyll创建静态网站

#### 1.1.1 关于Jekyll

`jekyll是一个简单的免费的Blog生成工具`，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

Jekyll 的核心其实是一个文本转换引擎。它的概念其实就是： 你用你最喜欢的标记语言来写文章，可以是 Markdown，也可以是 Textile,或者就是简单的 HTML, 然后 Jekyll 就会帮你套入一个或一系列的布局中。在整个过程中你可以设置URL路径, 你的文本在布局中的显示样式等等。这些都可以通过纯文本编辑来实现，最终生成的静态页面就是你的成品了。[参考文献](https://jekyllrb.com/)

#### 1.1.2 Jekyll的安装

1. 安装Homebrew

   * Homebrew是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。

   * 因为我已安装过，所以安装步骤省略。[参考文档](https://www.jianshu.com/p/de6f1d2d37bf)

     ![01_homebrew](/img/images/blog/01/01_homebrew.png)

2. 安装Ruby

   * jekyll本身基于Ruby开发，因此，想要在本地构建一个测试环境需要具有Ruby的开发和运行环境。

   * 因为我已安装过，所以安装步骤省略，[参考文档](https://jekyllrb.com/docs/installation/macos/)

     <img src="/img/images/blog/01/02_ruby.png" alt="02_ruby" />

3. 安装Jekyll【[参考文档1](https://jekyllrb.com/docs/),[参考文档2](https://www.jianshu.com/p/8f22ed56da67)】

   1. 安装打包器和jekyll

      ```shell
      $ gem install --user-install bundler jekyll
      ```

   2. 创建一个新的Jekyll网站

      ```shell
      $ jekyll new myblog_name
      ```

      >注意：若提示 `zsh: command not found: jekyll` ，需要把gem路径配置到PATH里面

      * 需要把gem路径配置到PATH里面

        ```shell
        # 1.修改文件，加入相关配置代码
        	$ vim .bash_profile
           # jekyll相关配置
           $ export JEKYLL_HOME=/Users/silhouette/.gem/ruby/2.6.0
           $ export PATH=$PATH:$JEKYLL_HOME/bin
        # 让配置文件生效
        	$ source ~/.bash_profile
        # 测试配置是否起效
        	$ jekyll -v
        ```

   3. 创建一个新的Jekyll网站

      ```shell
      # Create a new Jekyll site at ./myblog.
      $ jekyll new myblog
      ```

   4. 编译网站并启动本地服务

      ```shell
      # Change into your new directory.
      $ cd myblog_name
      # Build the site and make it available on a local server.
      $ bundle exec jekyll serve
      ```

   5. 访问[http://localhost:4000/]()查看效果

      <img src="/img/images/blog/01/03_Jekyll效果图.png" alt="03_Jekyll效果图" style="zoom:50%;" />

### 1.2 GitHub Pages

####1.2.1 GitHub Pages介绍

GitHub Pages是依托于Github仓库的展示你或者你的项目的静态网站。[参考文档](https://pages.github.com/)

#### 1.2.2 基于GitHub Pages托管网站

1. 前往[GitHUb]()创建一个新的仓库，仓库名称为*username.github.io*，其中*username*是你的GitHub用户名或者组织名称。

   > 注意：仓库的名字必须为：<user_name>(or organization name).github.io

2. 将上一步创建的仓库克隆到本地

   ```shell
   $ git clone https://github.com/username/silhouetteLy.github.io
   ```

3. 创建第一个页面并推送到远程仓库

   ```shell
   $ echo "Hello World" > index.html
   $ git add --all
   $ git commit -m "初始提交"
   $ git push -u origin master
   ```

4. 访问[*https://username.github.io*]()查看效果

>注：[参考文档](https://www.jianshu.com/p/8f22ed56da67)

###1.3 github pages整合 jekyll

#### 1.3.1 整合部署

1. 将Jekyll生成的静态网站复制到username.github.io仓库并提交推送到GitHub
2. 访问[*https://username.github.io*]()查看效果

>注：[更多内容](https://help.github.com/en/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll)

## 二、个性化博客网站

### 2.1 目录结构分析

#### 2.1.1 目录结构

一个基本的 Jekyll 网站的目录结构一般是像这样的：

```txt
.
├── _config.yml   
├── _data
|   └── members.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.md
|   └── on-simplicity-in-technology.md
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
|   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
|   ├── _base.scss
|   └── _layout.scss
├── _site
├── .jekyll-metadata
└── index.html # 也可以是带 front matter 的 'index.md' 文件
```

####2.1.2 每个文件的功能

| **文件 / 目录**                                           | **描述**                                                     |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| `_config.yml`                                             | 存储的是 [配置](https://www.jekyll.com.cn/docs/configuration/) 信息。其中许多 参数是可以在命令行中指定的，但是 在此文件中进行设置会更容易些，并且你不必记住它们。 |
| `_drafts`                                                 | 草稿（Drafts）是未发布的文章（posts）。这些文章的文件名中没有包含 日期： `title.MARKUP`。了解如何 [使用草稿功能](https://www.jekyll.com.cn/docs/posts/#drafts)。 |
| `_includes`                                               | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用打括号加百分号前后包裹的标签 来把 `_includes`目录下的文件 包含进来。 |
| `_layouts`                                                | layouts（布局）是包裹在文章外部的模板。布局可以在 [YAML 头信息](http://jekyllcn.com/docs/frontmatter/)中根据不同文章进行选择。 这将在下一个部分进行介绍。标签 `{{ content }}` 可以将content插入页面中。 |
| `_posts`                                                  | 这里放的就是你的文章了。文件格式很重要，必须要符合: `YEAR-MONTH-DAY-title.MARKUP`。 [永久链接](http://jekyllcn.com/docs/permalinks/) 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。 |
| `_data`                                                   | 格式化的站点数据应当放在此目录下。Jekyll 将自动加载此目录下的所有数据文件（支持 `.yml`、 `.yaml`、`.json`、`.csv` 或 `.tsv` 格式和扩展名）， 然后就可以通过 `site.data` 来访问了。假定在 此目录下有一个文件名为 `members.yml` 的文件，那么你可以通过 `site.data.members` 变量来访问该文件中的内容。 |
| `_sass`                                                   | 这些事可以导入（import）到 `main.scss` 文件中的 sass 代码片段， `main.scss` 文件最终经过处理之后形成一个独立的样式文件 `main.css` 并被用于整个站点。 |
| `_site`                                                   | 在 Jekyll 转换完所有的文件之后，将在此目录下放置生成的站点（默认情况下）。 最好将此目录添加到 `.gitignore` 文件中。 |
| `.jekyll-metadata`                                        | 该文件帮助 Jekyll 跟踪哪些文件从上次建立站点开始到现在没有被修改，哪些文件需要在下一次站点建立时重新生成。该文件不会被包含在生成的站点中。将它加入到你的 `.gitignore` 文件可能是一个好注意。 |
| `index.html` or `index.md` and other HTML, Markdown files | 如果这些文件中包含 [YAML 头信息](http://jekyllcn.com/docs/frontmatter/) 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 `.html`, `.markdown`, `.md`, 或者 `.textile` 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。 |
| Other Files/Folders                                       | 其他一些未被提及的目录和文件如 `css` 还有 `images` 文件夹， `favicon.ico` 等文件都将被完全拷贝到生成的 site 中。这里有一些[使用 Jekyll 的站点](http://jekyllcn.com/docs/sites/)，如果你感兴趣就来看看吧。 |

> 注：[参考文档](https://www.jekyll.com.cn/docs/structure/)

### 2.2 开始写第一篇博客

####2.2.1 开始搭建

通用修改 `_config.yml`文件来轻松的开始搭建自己的博客:Jekyll官方网站还有很多的参数可以调，具体请参考[官方文档](http://jekyllcn.com/).

####2.2.2 写一篇博文

要发表的文章一般以markdown的格式放在这里`_posts/`，你只要看看这篇模板里的文章你就立刻明白该如何设置。

yaml 头文件(SEO)长这样:

```txt
---
layout:     post
title:      "Welcome to Jekyll!"
subtitle:   "Hello World, Hello Blog"
date:       2019-01-01 00:00:00
author:     "silhouetteLy"
header-img: "img/post-bg-2019.jpg"
tags:
    - Life
---
```

