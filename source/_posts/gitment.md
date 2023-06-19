---
title: hexo添加gitment评论系统
date: 2018-02-22 22:57:06
tags:
---

曾经使用多说和网易云评论作为博客的评论系统，不幸都相继倒闭后，博客就一直没有评论系统。虽博客的访问量可以忽略不计，但本着折腾和好奇的原则，还是折腾一下gitment。

# 更新hexo-theme-next主题

最新版本的next主题已经默认支持gitment，需要将next主题升级到最新版本。

我的hexo-theme-next使用单独的git项目进行管理，git地址为：https://github.com/kuring/hexo-theme-next。接下来需要将fork出来的git项目跟next的git项目进行同步。

1. 在本地创建名字为upstream的remote，指向地址为：`git remote add upstream https://github.com/theme-next/hexo-theme-next.git`

2. 拉取next项目到本地分支，本地的分支，执行`git fetch upstream`

3. 将upsteam/master分支合并到master分支上

```
git checkout master

# 由于修改了_config.yml文件，存在冲突，合并失败
lvkai@osx:~/blog/kuring/themes/hexo-theme-next% git merge upstream/master                                                                                                128 ↵
Removing source/css/_common/components/third-party/gentie.styl
Removing layout/_third-party/comments/gentie.swig
Auto-merging _config.yml
CONFLICT (content): Merge conflict in _config.yml
Removing README.en.md
Automatic merge failed; fix conflicts and then commit the result.
```

4. 解决冲突后提交并将master分支push到github仓库

# 注册gitment

前往：https://github.com/settings/profile

Developer settings -> Register a new application

在界面中输入如下内容：

![image](/images/gitment-1.png)

获取到Client ID和Client Secret.

# 新建github repo

创建新的github项目：https://github.com/kuring/gitment-comments

# 在next主题中设置gitment

next主题的配置文件为theme/next/_config.yml，修改其中的gitment设置如下，

1. client_id为在github中注册所获取到的client id
2. client_secret为在github中注册所获取到的client secret
3. github_repo为上面新创建的github repo名称

```
# Gitment
# Introduction: https://imsun.net/posts/gitment-introduction/
gitment:
  enable: true
  mint: true # RECOMMEND, A mint on Gitment, to support count, language and proxy_gateway
  count: true # Show comments count in post meta area
  lazy: false # Comments lazy loading with a button
  cleanly: false # Hide 'Powered by ...' on footer, and more
  language: # Force language, or auto switch by theme
  github_user: kuring # MUST HAVE, Your Github ID
  github_repo: gitment-comments # MUST HAVE, The repo you use to store Gitment comments
  client_id: xxx # MUST HAVE, Github client id for the Gitment
  client_secret: xxxx # EITHER this or proxy_gateway, Github access secret token for the Gitment
  proxy_gateway: # Address of api proxy, See: https://github.com/aimingoo/intersect
  redirect_protocol: # Protocol of redirect_uri with force_redirect_protocol when mint enabled
```

执行`hexo clean && hexo g && hexo s`重新生成页面并在本地运行，可以看到gitment组件已经可以显示了，但是提示`Error: Comments Not Initialized`错误，点击login，然后允许认证，即可消除该错误。

在界面上添加评论后，可以在github repo的issuse中看到，整个搭建完毕。
