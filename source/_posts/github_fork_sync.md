---
title: 在github上同步fork的项目
Status: public
url: github_fork_sync
date: 2014-11-02
---

在github上可以fork别人的项目成为自己的项目，但是当fork的项目更新后自己fork的项目应该怎么怎么更新呢？我从网上看到了两种方式，一种是采用github的web界面中的操作来实现，具体是通过“Pull Request”功能来实现；另外一种是通过在本地合并代码分支的方式来解决。本文将采用第二种方式，以我最近fork的项目为例来说明。

# 将fork后自己的项目clone到本地

执行`git clone https://github.com/kuring/leetcode.git`即可将自己fork的代码更新到本地。

fork完成后的远程分支和所有分支情况如下：

```
kuring@T420:/data/git/leetcode$ git remote -v
origin	https://github.com/kuring/leetcode.git (fetch)
origin	https://github.com/kuring/leetcode.git (push)
kuring@T420:/data/git/leetcode$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

# 将fork之前的项目clone到本地

将fork之前的项目添加到本地的远程分支haoel中，执行`git remote add haoel https://github.com/haoel/leetcode`。

再查看一下远程分支和所有分支情况：

```
kuring@T420:/data/git/leetcode$ git remote -v
haoel	https://github.com/haoel/leetcode (fetch)
haoel	https://github.com/haoel/leetcode (push)
origin	https://github.com/kuring/leetcode.git (fetch)
origin	https://github.com/kuring/leetcode.git (push)
kuring@T420:/data/git/leetcode$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

# 将远程代码halel分支fetch到本地

执行`git fetch haoel`，此时的所有分支情况如下，可以看出多了一个remotes/haoel/master分支。

```
kuring@T420:/data/git/leetcode$ git branch -a
* master
  remotes/haoel/master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

# 将halel分支merge到本地的分支

执行`git merge remotes/haoel/master`，此时发现有冲突，提示内容如下：

```
uring@T420:/data/git/leetcode$ git merge haoel/master
自动合并 src/reverseInteger/reverseInteger.cpp
冲突（内容）：合并冲突于 src/reverseInteger/reverseInteger.cpp
自动合并失败，修正冲突然后提交修正的结果。
```

之所以出现上述错误，这是由于我在fork之后在本地修正了源代码中的一处bug，而在fork之后到现在的时间间隔内原作者haoel也正好修正了该bug。打开文件后发现存在如下的内容，其实就是代码风格的问题，我这里将错误进行修正。

```
36 <<<<<<< HEAD                                                                                                                                                                
37     while( x != 0 ){
38 =======
39     while( x != 0){
40 >>>>>>> haoel/master
```

如果没有冲突的情况下通过merge命令即会将haoel/master分支合并master分支并执行commit操作。可以通过`git status`命令看到当前冲突的文件和已经修改的文件。执行`git status`命令可以看到如下内容，说明未冲突的文件已经在暂存区，冲突的文件需要修改后执行add操作：

```
kuring@T420:/data/git/leetcode$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。

您有尚未合并的路径。
  （解决冲突并运行 "git commit"）

要提交的变更：

	修改:         src/3Sum/3Sum.cpp
	修改:         src/4Sum/4Sum.cpp
	修改:         src/LRUCache/LRUCache.cpp
	......	// 此处省略了很多重复的
	......

未合并的路径：
  （使用 "git add <file>..." 标记解决方案）

	双方修改：     src/reverseInteger/reverseInteger.cpp
```

解决完冲突后执行add操作后再通过`git status`命令查看的内容如下。通过`git status`命令却看不到已经解决的冲突文件，对于这一点我还是很理解，参考文章中的[Git 分支 - 分支的新建与合并](http://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)是可以看到已经解决的冲突文件的，因为执行`git add`后将解决完成冲突的文件放到了暂存区中。

```
kuring@T420:/data/git/leetcode$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。

所有冲突已解决但您仍处于合并中。
  （使用 "git commit" 结束合并）

要提交的变更：

	修改:         src/3Sum/3Sum.cpp
	修改:         src/4Sum/4Sum.cpp
	修改:         src/LRUCache/LRUCache.cpp
	......	// 此处省略了很多重复的
	......
```

这里冲突后merge操作并没有执行commit操作，需要解决冲突后再手工执行commit操作，此时整个的同步操作就已经完成了。

# 结尾

如果隔一段时间后又需要同步项目了仅需要执行`git fetch haoel`命令以下的操作即可。

# 参考

* [Git 分支 - 分支的新建与合并](http://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)
* [如何在github上fork一个项目来贡献代码以及同步原作者的修改](http://www.cnblogs.com/rubylouvre/archive/2013/01/24/2874694.html)
* [Github上更新自己fork的代码](http://blog.csdn.net/do_it__/article/details/7836513)

由于git命令较多，为了便于查阅增加一处git data transprot commands

![Git图解](http://kuring.qiniudn.com/git_data_transport_commands.png)