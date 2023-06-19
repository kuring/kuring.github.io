title: git 常用操作
date: 2023-03-07 15:36:41
tags:
author:
---
删除当前分支的最后一次更新：git reset --hard HEAD^，使用reset不会留下痕迹

拉取另外一个分支的commit: `git cherry-pick $commit` 其中$commit为另外一个分支的提交

删除远程分支 `git push origin --delete xxx`

强制更新远程分支：`git push --force-with-lease origin feature/statefulset`

比较两个分支: `git diff $branch1 $branch2 --stat`

删除本地分支：`git branch -D dev-yihui`

拉取远程分支并切换分支：`git checkout -b develop origin/kubeone-v2` develop为本地分支，origin/kubeone-v2为远程分支

最近一次修改的commit内容修改：`git commit --amend --author="kuring <kuring.public@gmail.com>"`

配置用户名和密码：`git config user.name kuring`  `git config user.email kuring.public@gmail.com`
