---
title: blog从farbox迁移到了hexo
date: 2017-04-06 23:33:47
tags: hexo farbox
---

清明节假期突然想起了我好久不更的blog，看到Farbox官网上的[《2016，终结了几个产品》](https://blog.farbox.com/post/2017)，说明Farbox已经停止更新了。虽然我挺喜欢Farbox这个项目，也见证了Farbox的成长及作者做产品的思考，在这里也向作者致敬。

我当时开始准备启用Farbox之前试用过jekyll，翻遍了整个github，也没找到个合我心意的theme。幸好是Farbox的出现，让我眼前一亮，这就是我想好的blog系统了。Farbox的停更使我不得不考虑重新换个blog，虽然2016的文章数量仅为罕见的个位数，但有可能今年有时间会多写一些。

近几年hexo特别的火，在试看了官方文档了解功能及考虑了blog的迁移成本后，心想，这就是我想要的blog系统了。hexo该有的功能全都有，甚至比Farbox要强大很多。Farbox的很多设计思想跟hexo相仿，但hexo显的更加自由，blog需要自己一手搭建完成。

当然hexo要想使用的好，做一些全面的了解及折腾是必不可少的，毕竟最终利用的Github pages是个静态的系统。早已没有了想当年翻遍整个github上jekyll theme的精力了，我这次的基调是能少折腾就少折腾，毕竟blog我也不是经常写，访问量也更是少的可怜，就当全面了解下当前最火的hexo就好了。

# theme

本着不折腾原则，直接启用了很火的hexo-theme-next，文档比较全，维护比较及时。基本上按照文档走一遍，该配置的就都可以配置上了。

# 代码同步

代码通过git同步是必备技能。

## hexo项目代码同步

hexo采用的是node.js环境，而Github pages是静态的，因此Github pages上仅能存储的是hexo编译后的静态文件，这些静态文件直接通过`hexo d`部署到kuring.github.com仓库中就可以了。

而对于项目中的_config.yml、md文件我直接用git同步到Github上另外一个项目hexo_bak中了。网上还有思路是同步到kuring.github.com上的另外一个分支，我感觉太啰嗦，容易出错，还不如直接分开来的简便。

## theme项目的代码同步

theme项目中也包含了部分自己的配置及修改，我这里选择的同步策略为从github上fork对应的theme项目，然后clone fork下来的项目到本地，然后直接在theme的项目中通过git命令同步到github fork的项目中。

网上也有思路是通过git subtree的方式来解决，我仍然感觉太啰嗦，不采用。

但这样一个blog项目需要多个git仓库，git push起来会比较麻烦，好在theme一般不怎么修改。

# 评论系统

之前用多说的时候也没几个评论的，用起来还不错，至少比被墙了的disqus要好很多，可是多说这么好的项目要关闭了。我直接使用了国内的网易云跟帖来满足评论的需求。

# 站内搜索

站内搜索是必不可少的功能，next主题提供了多种选择，我直接使用了hexo-generator-searchdb通过本地搜索来完成，生成的xml文件目前还比较小，效果还可以。

# 常用命令

* 启用本地server端：`hexo clean && hexo g && hexo s`
* 部署到github：`hexo d`
* 发布文章：`hexo new 文章url`

使用`hexo new draft test`会在source/_drafts目录下创建对应文件，此时文件不会生成页面，用于存放未写完的文章。`hexo publish draft test`命令可将_drafts下的文章移动到_posts目录下，并添加创建时间等信息。

# 收个尾

blog总算迁移完成了，期望今年能多写上几篇。
