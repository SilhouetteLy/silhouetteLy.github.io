---
layout:     post
title:      "个人博客功能完善"
subtitle:   "Jekyll添加相关功能按钮"
date:       2020-04-29 23:44:46
author:     "silhouette"
header-img: "img/images/blog/02/00-post-bg.jpg"
tags:
    - Blog
---

## 前言

想对刚搭建好的博客添加一些常用的功能，如`返回顶部`、`全文搜索`等。以下内容参考资料有：

* [Jekyll博文内链和返回顶部按钮](https://www.smslit.top/2015/10/28/backToTop-Jekyll/)
* [Jekyll search组件](http://www.codeboy.me/2016/01/18/jekyll-search-component/)

### 添加返回按钮块

只需两步就可以在页面添加一个固定的返回顶部按钮，这里没有加JS代码，没什么漂亮的效果，只是功能的实现。

1. 在 `footer.html`文件中添加样式

   ```html
   <div id="backtop">
      <a href="#">TOP</a>
   </div> 
   ```

2. 添加按钮的css样式到 css 文件中

   ```css
   #backtop a { /* back to top button */
       text-align: center;
       line-height: 50px;
       font-size: 16px;
       width:50px;
       height: 50px;
       position: fixed;
       bottom: 10px; /* 小按钮到浏览器底边的距离 */
       right: 60px; /* 小按钮到浏览器右边框的距离 */
       color: rgb(64,120,192); /* 小按钮中文字的颜色 */
       z-index: 1000;
       background: #fff; /* 小按钮底色 */
       padding: auto; /* 小按钮中文字到按钮边缘的距离 */
       border-radius: 50px; /* 小按钮圆角的弯曲程度（半径）*/
       -moz-border-radius: 50px;
       -webkit-border-radius: 50px;
       font-weight: bold; /* 小按钮中文字的粗细 */
       text-decoration: none !important;
       box-shadow:0 1px 2px rgba(0,0,0,.15), 0 1px 0 #ffffff inset;
   }
   #backtop a:hover { /* 小按钮上有鼠标悬停时 */
       background: rgba(64,120,192,0.8); /* 小按钮的底色 */
       color: #fff; /* 文字颜色 */
   }
   ```

3. 在`head.html`文件引入 CSS 文件

   ```html
   <!-- Custom CSS -->
   <link rel="stylesheet" href="{{ "/css/silhouette.css" | prerend: site.baseurl }}">
   ```

### 全局搜索

为了能更快额加入搜索功能，就在网上直接到了一个已经封装好了的单独组件，通过[下载链接](https://github.com/androiddevelop/jekyll-search)将组建代码拉到本地。然后就可以开始加入操作了：

1. 将search目录放至于博客根目录下，其中search目录结构如下:

   ```txt
    search
    ├── cb-footer-add.html
    ├── cb-search.json
    ├── css
    │   └── cb-search.css
    ├── img
    │   ├── cb-close.png
    │   └── cb-search.png
    └── js
        ├── bootstrap3-typeahead.min.js
        └── cb-search.js
   ```

2. 在`_include/footer.html`中的`/footer>`前加入`cb-foot-add.html`中的代码。

   ```html
   <div class="cb-search-tool" style="position: fixed; top: 0px ; bottom: 0px; left: 0px; right:  0px;
         opacity: 0.95; background-color: #111111; z-index: 9999; display: none;">
       <input type="text" class="form-control cb-search-content" id="cb-search-content" style="position: fixed; top: 60px" placeholder="文章标题 日期 >标签" >
   
       <div style="position: fixed; top: 16px; right: 16px;">
           <img src="/search/img/cb-close.png"  id="cb-close-btn"/>
       </div>
   </div>
   
   <div style="position: fixed; right: 16px; bottom: 20px;">
       <img src="/search/img/cb-search.png"  id="cb-search-btn"  title="双击ctrl试一下"/>
   </div>
   
   <link rel="stylesheet" href="{{ "/search/css/cb-search.css" | prepend: site.baseurl }}">
   
   <script src="{{ "/search/js/bootstrap3-typeahead.min.js" | prepend: site.baseurl }}"></script>
   <script src="{{ "/search/js/cb-search.js" | prepend: site.baseurl }}"></script>
   ```

3. 从第二步可知，加入的代码包含了`CSS`和`JS`代码块，为了代码的整洁性，我们将引入`CSS`的代码放入`head.html`中，注意需要事先引入**bootstrap的css文件**，如果没有的话，则得先引入；有的话就放在下面，如下：

   ```html
   <!-- Bootstrap Core CSS -->
   <link rel="stylesheet" href="{{ "/css/bootstrap.min.css" | prepend: site.baseurl }}">
   
   <!-- Custom CSS -->
   <link rel="stylesheet" href="{{ "/css/silhouette.css" | prerend: site.baseurl }}">
   <!-- Pygments Github CSS -->
   <link rel="stylesheet" href="{{ "/css/syntax.css" | prepend: site.baseurl }}">
   
    <!--jekyll Search-->
   <link rel="stylesheet" href="{{ "/search/css/cb-search.css" | prepend: site.baseurl }}">
   ```

4. 将第二步中 copy 的引入JS 的代码除出`<footer>`标签中，统一下面，和其它的 JS 文件放在一起。注意注意需要事先引入**jquery**与**bootstrap的 js 文件)**，并且引入的 jquery的标签得早于贴进去的 `cb-search.js`。如下：

   ```html
   <!-- jQuery -->
   <script src="{{ "/js/jquery.min.js " | prepend: site.baseurl }}"></script>
   
   <!-- Bootstrap Core JavaScript -->
   <script src="{{ "/js/bootstrap.min.js " | prepend: site.baseurl }}"></script>
   
   <!--jekyll search-->
   <script src="{{ "/search/js/bootstrap3-typeahead.min.js" | prepend: site.baseurl }}"></script>
   <script src="{{ "/search/js/cb-search.js" | prepend: site.baseurl }}"></script>
   ```

> *如有任何知识产权、版权问题或理论错误，还请指正。*