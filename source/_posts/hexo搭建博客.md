---
title: Hexo 博客搭建
date: 2018-03-15 17:30:01
tags:
    - Hexo
categories:
    - Hexo
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 
src="//music.163.com/outchain/player?type=2&id=30967201&auto=1&height=66"></iframe>
{% asset_img myBlog.jpg 我的博客首页 %}


#### 1. 安装Git
直接下载安装就可以。下载地址：[Git官网](https://gitforwindows.org/)
或者，有安装教程：{% post_link Git安装教程 %}

#### 2. 安装NodeJS
Hexo是基于nodeJS环境的静态博客，里面的npm工具很有用。而且命令特别简单。
* [下载地址](https://nodejs.org/en/)(说明：LTS为长期支持版，Current为当前最新版)
* 基本上一路上next就行。注意：Custom Setup中记得选择`ADD To Path`。这样就不用自己配置环境变量了。不然就和java一样去配置环境变量。
* 查看版本
    * `node -v`
    
    {% asset_img node1.png node版本 %}
    
#### 3. 安装Hexo
* 创建一个文件夹，用于存放Blog的所有文件。
* Shift+鼠标右键。在该目录打开命令行，然后执行命令：`npm i -g hexo`
* 安装完成之后查看版本：`hexo -v`

    {% asset_img hexo1.png hexo版本 %}

* 初始化。`hexo init`;运行成功，打开文件夹可看到以下文件。

    {% asset_img h1.png 初始化 %}
    
* 解释一下
    * node_modules：是依赖包
    * public：存放的是生成的页面
    * scaffolds：命令生成文章等的模板
    * source：用命令创建的各种文章
    * themes：主题
    * _config.yml：整个博客的配置
    * db.json：source解析所得到的
    * package.json：项目所需模块项目的配置信息
    
#### 4. 将项目在 Github 配置

* 添加一个repo，名称为`yourname.github.io`,其中yourname是你的github用户名。必须这个规则创建。
* 开启`GitHub Pages`。在Setting配置页中，找到这个项，点击Automatic page generator，Github将会自动替你创建出一个gh-pages的页面。 如果你的配置没有问题，那么大约15分钟之后，yourname.github.io这个网址就可以正常访问了~ 如果yourname.github.io已经可以正常访问了，那么Github一侧的配置已经全部结束了。

     {% asset_img h2.png 生成成功 %}
 
* 然后就是Blog的一些设置了。

#### 5. Hexo 怎么玩。
    
* 命令行自动部署blog。此处必须依赖插件。需先安装一波 `npm install hexo-deployer-git --save`。然后配置_config.xml文件。
    ```text
    deploy:
      type: git
      repository:
        github: git@github.com:YangXHero/yangkai.github.io.git
        coding: git@git.coding.net:yangkai92/yangkai92.git
      branch: master
    ```
* 自定义站点文章搜索。需先安装一波 `npm install hexo-generator-search --save`。然后配置文件。
    ```text
    search:
      path: search.xml
      # 如只想索引文章，可设置为post
      field: all
    ```
    
* 添加RSS。在主题配置文件中有NexT默认的RSS设置，默认为留空，这时使用 Hexo 生成的 Feed 链接，需要先安装 hexo-generator-feed插件。
  
  在站点根目录打开git bash，安装插件：
  ```class
    $ npm install --save hexo-generator-feed
  ```
  在站点配置文件加以下代码：
  ```properties
    # 配置RSS
    feed:
      #feed 类型 (atom/rss2)
      type: atom
      #rss 路径
      path: atom.xml
      #在 rss 中最多生成的文章数(0显示所有)
      limit: 0
  ```
  修改主题配置文件如下：
  ```properties
  # Set rss to false to disable feed link.
  # Leave rss as empty to use site's feed link.
  # Set rss to specific value if you have burned your feed already.
  rss: /atom.xml
  ```
* 在文章中添加图片信息。

  ```text
  把主页配置文件_config.yml 里的post_asset_folder:这个选项设置为true
    
  在你的hexo目录下执行这样一句话npm install hexo-asset-image --save，
  这是下载安装一个可以上传本地图片的插件，来自dalao：dalao的git
    
  等待一小段时间后，再运行hexo n "xxxx"来生成md博文时，
  /source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹

  文章中引用。  `{% asset_img 图片名称 图片描述 %}`
  ```
  
* 常用命令
  ```text
  1.hexo new 文章名称
  2.hexo g  生成静态文件
  3.hexo d  部署，发布。
  4.hexo clean 清除上次生成的静态文件。
  ```
    
* 新建标签和分类页面
  ```text
    hexo new page tags    
    hexo new page categories
  
    在页面对应index.md中加相应的type类型。
    ---
    title: tags
    date: 2018-08-20 17:31:44
    type: "tags"
    ---

    ---
    title: categories
    date: 2018-08-20 17:31:53
    type: "categories"
    ---
  ```
