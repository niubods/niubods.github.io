---
layout: post
title: 在GitHub上搭建Jekyll个人博客
category: 生命在于折腾
tags: [GitHub, Jekyll, Bootstrap]
excerpt: >
  长久以来都想搭建一个技术博客，当年玩NDS自制软件的时候，就想要有个写学习总结和整理文档的地方。尝试过很多公共博客，都不适合写技术类文章，因为我想要贴高亮代码。当时就很红的Wordpress可以安装代码高亮插件，但是搭建个人博客需要购买域名和php空间，我没有任何网上支付手段，于是只好用新浪博客凑合了一阵子...
---
长久以来都想搭建一个技术博客，当年玩NDS自制软件的时候，就想要有个写学习总结和整理文档的地方。尝试过很多公共博客，都不适合写技术类文章，因为我想要贴高亮代码。当时就很红的Wordpress可以安装代码高亮插件，但是搭建个人博客需要购买域名和php空间，我没有任何网上支付手段，于是只好用新浪博客凑合了一阵子。

某些得不到满足的需求会周期性的出现。后来用GitHub的时候发现居然可以在上面托管静态网站，也支持直接上传jekyll项目。折腾了一阵子，终于搞出了个还过得去的个人博客。在GitHub上写博客，最大的门槛是git。当时还没有GitHub for Windows，所有操作都要用命令行，并且由于某个广为人知的原因，GitHub也老是遭殃，推送一篇文章到GitHub也要半天，还不一定成功……于是这个博客根本就没有真正用起来。

搭建博客的想法再一次周期性出现的时候，我已经很习惯在Evernote中记录自己的想法了。但Evernot还是有让人不满意的地方，比如我不能用喜欢的Markdown来写，书写体验不佳，插入高亮代码也不方便，还有笔记主要还是给自己看的，人总有将思想传播出去的欲望。对于书写体验不佳的问题，我发现了[Draft](https://draftin.com)这么个应用，表面上功能很简单，致力于改善书写体验，并且支持Markdown语法。试用之后我很快喜欢上了这个简约而不简单的应用，然后在支持的发布列表中发现了[Tumblr](http://www.tumblr.com/)……

Tumblr是一个轻博客，风格清新，讨人喜欢。选一个漂亮的主题，用自定义css设置一下字体，修改几处不满意的地方，就打造出了一个个性十足的博客。虽然自带的编辑框很小，不适合写太多文字，但是好在有Draft这款神器。惊喜的是Tumblr居然支持Markdown，另外可以通过gist贴高亮代码，真是太完美了。虽然打心底里喜欢，但是不得不承认Tumblr也还是不符合我的要求。Tumblr是一个定位在社交领域的轻博客，不适合发布太长的文章是一方面，另外我几乎没发现有其他的中文用户啊亲！

好吧又扯远了，还是来说说用Jekyll搭建博客吧。Jekyll是用ruby写的静态网站生成系统，GitHub自家出品。[Jekyll 官方网站](http://jekyllrb.com/)上有详细的文档。最好在Linux上安装Jekyll，当时在windows上运行Jekyll真是各种坑啊。幸好现在我有Ubuntu Server虚拟机。Ubuntu发行版应该已经自带ruby1.8，直接`$sudo gem install jekyll`安装就可以。我就是开始没注意，误装ruby1.9.1，碰上很多麻烦。

Jekyll本身只是一个静态网站生成工具，相当于只有一个平台和一系列工具，靠这些来搭建一个高质量的博客系统难度较高。

那么下一个主角就是[JekyllBootstrap](http://jekyllbootstrap.com/)了。这是一个接近成品的博客系统，把它克隆到本地，稍微修改一下就可以用了。另外它提供了7套主题，不算很丰富但也够用了。我比较喜欢的暗色主题所以选择了“Hooligan”。

首先有个问题，如果我在博客中引用了一段gist，它会自动引入一个用来支持代码高亮的css文件。亮色主题下看着还不错，但是在暗色主题下就悲剧了，不但和整体配色格格不入，并且和主题原本的代码高亮样式叠加后表现非常奇怪。gist本身无法自定义配色，那么只好改自己的css了。我写了一个[gist-darkstrap.css](https://gist.github.com/niubods/5575078)，看上去完全模仿了自带的代码高亮样式。

还有一个问题，GitHub广泛支持它们改进过的Markdown方言，[GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown)，简称GFMD，其中一项很实用的改进就是可以使用3个反引号作为代码的开始和结束标记，并且还可以指定语言。Jekyll也支持代码高亮，不过采用的是另一种用Liquid标签的方式，看上去很糟糕。实际上Jekyll默认使用maruku来转换Markdown文件，而改用redcarpet就可以使用GFMD的语法了。本地运行Jekyll的话可能需要安装redcarpet：`$sudo gem install redcarpet`。然后在_config.yml中添加一项配置:`markdown: redcarpet`就可以了。

最后可以将博客系统托管到GitHub上去了。我们知道，GitHub提供了一项叫做[GitHub Pages](http://pages.github.com/)的服务，你只需要创建一个叫做username.github.io的代码库（username是自己的用户名），之后就可以用`http://username.github.io`来访问自己的站点了，当然如果自己有域名的话也可以设置成用自己的域名来访问。GitHub公布了当前使用的[Jekyll和相关模块的版本](https://help.github.com/articles/using-jekyll-with-pages#troubleshooting)。已经有redcarpet模块了，那么在GitHub Pages上也可以放心使用GFMD语法了。